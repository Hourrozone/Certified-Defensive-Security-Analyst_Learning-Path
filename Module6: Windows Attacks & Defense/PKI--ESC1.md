# 🔐 PKI - ESC1

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

<img width="684" height="856" alt="image" src="https://github.com/user-attachments/assets/f0820995-7448-47a3-ac46-99ee82632104" />

<img width="2785" height="1192" alt="image" src="https://github.com/user-attachments/assets/a7d65351-5d30-438a-b57c-28386fbe2114" />

When checking the 'Vulnerable Certificate Templates' section from the output of Certify, we will see that a single template with plenty of information about it is listed. We can tell that the name of the CA in the environment is PKI.eagle.local\eagle-PKI-CA, and the vulnerable template is named UserCert. The template is vulnerable because:

* All Domain users can request a certificate on this template.
* The flag CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT is present, allowing the requester to specify the SAN (therefore, any user can request a certificate as any other user in the network, including privileged ones).
* Manager approval is not required (the certificate gets issued immediately after the request without approval).
* The certificate can be used for 'Client Authentication' (we can use it for login/authentication).

To abuse this template, we will use Certify and pass the argument request by specifying the full name of the CA, the name of the vulnerable template, and the name of the user, for example, Administrator:

```powershell
PS C:\Users\bob\Downloads> .\Certify.exe request /ca:PKI.eagle.local\eagle-PKI-CA /template:UserCert /altname:Administrator
```

<img width="719" height="879" alt="image" src="https://github.com/user-attachments/assets/b67f0c2b-dcbc-4e0f-9555-80e6e732e09e" />

<img width="1885" height="1107" alt="image" src="https://github.com/user-attachments/assets/fee042fb-4ca9-4c78-8880-147b72a22ec0" />

Once the attack finishes, we will obtain a certificate successfully. The command generates a PEM certificate and displays it as base64. We need to convert the PEM certificate to the PFX format by running the command mentioned in the output of Certify (when asked for the password, press Enter without providing one), however, to be on the safe side, let's first execute the below command to avoid bad formatting of the PEM file.

```shellsession
@htb[/htb]$ sed -i 's/\s\s\+/\n/g' cert.pem
```

Then we can execute the openssl command mentioned in the output of Certify.

```shellsession
@htb[/htb]$ openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

<img width="1172" height="105" alt="image" src="https://github.com/user-attachments/assets/26302357-2b29-4ad9-8b21-ad996e0f0892" />

Now that we have the certificate in a usable PFX format (which Rubeus supports), we can request a Kerberos TGT for the account Administrator and authenticate with the certificate:

```powershell
PS C:\Users\bob\Downloads> .\Rubeus.exe asktgt /domain:eagle.local /user:Administrator /certificate:cert.pfx /dc:dc1.eagle.local /ptt
```

<img width="663" height="826" alt="image" src="https://github.com/user-attachments/assets/f485b3e3-e6e9-41ce-87c6-1dd7d34c1113" />

<img width="2599" height="1530" alt="image" src="https://github.com/user-attachments/assets/40736589-d623-4243-968c-4813bce2db0a" />

After successful authentication, we will be able to list the content of the C$ share on DC1:

```powershell
PS C:\Users\bob\Downloads> dir \\dc1\c$
```

<img width="1437" height="549" alt="image" src="https://github.com/user-attachments/assets/383a3274-a946-4ee9-bc70-e422bb5edac2" />

---

## 🛡️ Prevention

The attack would not be possible if the CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT flag is not enabled in the certificate template. Another method to thwart this attack is to require CA certificate manager approval before issuing certificates; this will ensure that no certificates on potentially dangerous templates are issued without manual approval (which hopefully correlates that the request originated from a legit user).

Because there are many different privilege escalation techniques, it is highly advised to regularly scan the environment with Certify or other similar tools to find potential PKI issues.

In addition, certificate templates should be regularly reviewed to ensure that only appropriate users and groups can enroll for certificates and that unnecessary Client Authentication or Smart Card Logon capabilities are removed.

Certificate templates that allow users to supply the subject or SAN should be reviewed carefully, especially when they can be used for authentication. Certificate Authority permissions and certificate issuance settings should also be audited regularly to identify unintended privilege-escalation paths.

---

## 🔎 Detection

When the CA generates the certificate, two events will be logged, one for the received request and one for the issued certificate, if it succeeds. Those events have the IDs of 4886 and 4887 as shown below:

<img width="972" height="1108" alt="image" src="https://github.com/user-attachments/assets/6fa705fb-56b6-44e9-907e-de877e5fe34c" />

Unfortunately, we can only tell that Bob requested a certificate from WS001; we cannot know if the request specified the SAN.

The CA contains a list of all issued certificates, so if we look there, we will see the request for certificate ID 36 (the one from the attack scenario above):

<img width="1374" height="70" alt="image" src="https://github.com/user-attachments/assets/383ad56e-63a2-40c2-8bc3-cac8f9c26fd4" />

The general overview of the GUI tool does not display the SAN either, but we can tell that a certificate was issued via the vulnerable template. If we want to find the SAN information, we'll need to open the certificate itself:

<img width="470" height="408" alt="image" src="https://github.com/user-attachments/assets/6c2d6f2b-4cf1-4b22-8e93-a6af0fc38d63" />

There is also the possibility to view that programmatically: the command certutil -view will dump everything on the CA with all of the information about each certificate (this can be massive in a large environment):

```cmd
certutil -view
```

<img width="1695" height="968" alt="image" src="https://github.com/user-attachments/assets/13b37892-32ed-4520-ae38-ca048d8f6977" />

With some scripting, we can automate parsing and discovery of abused vulnerable templates by threat agents.

Finally, if you recall, in the attack, we used the obtained certificate for authentication and obtained a TGT; AD will log this request with the event ID 4768, which will specifically have information about the logon attempt with a certificate:

<img width="1457" height="1028" alt="image" src="https://github.com/user-attachments/assets/7088f4ac-2797-4b80-8fce-f712157e7dd5" />

Note that events 4886 and 4887 will be generated on the machine issuing the certificate rather than the domain controller. If GUI access is not available, we can use PSSession to interact with the PKI machine, and the Get-WinEvent cmdlet to search for the events:

```cmd
C:\Users\bob\Downloads>runas /user:eagle\htb-student powershell

Enter the password for eagle\htb-student:
Attempting to start powershell as user "eagle\htb-student" ...
```

```powershell
PS C:\WINDOWS\system32> New-PSSession PKI
```

<img width="608" height="806" alt="image" src="https://github.com/user-attachments/assets/f6530352-6617-494c-8358-589d23a08ddd" />

To view the full audit log of the events, we can pipe the output into Format-List , or save the events in an array and check them individually:

```powershell
[pki]: PS C:\Users\htb-student\Documents> $events = Get-WinEvent -FilterHashtable @{Logname='Security'; ID='4886'}
[pki]: PS C:\Users\htb-student\Documents> $events[0] | Format-List -Property *
```

<img width="624" height="731" alt="image" src="https://github.com/user-attachments/assets/8997b1e6-d22d-4a0a-805a-62424dfedc04" />

---

## 🔑 Key Takeaway

**PKI - ESC1** is an Active Directory Certificate Services misconfiguration where users can request authentication certificates while being able to specify another identity in the certificate.

The key security lesson is that **certificate templates can become an authentication and privilege-escalation path when their enrollment permissions and subject/SAN settings are too permissive**. Regularly auditing certificate templates and monitoring certificate issuance helps reduce this risk.
