# GPP Passwords

## 📖 Description

SYSVOL is a network share on all Domain Controllers, containing logon scripts, group policy data, and other required domain-wide data. AD stores all group policies in `\\<DOMAIN>\SYSVOL\<DOMAIN>\Policies\`. When Microsoft released it with the Windows Server 2008, Group Policy Preferences (GPP) introduced the ability to store and use credentials in several scenarios, all of which AD stores in the policies directory in SYSVOL.

During engagements, we might encounter scheduled tasks and scripts executed under a particular user and contain the username and an encrypted version of the password in XML policy files. The encryption key that AD uses to encrypt the XML policy files (the same for all Active Directory environments) was released on Microsoft Docs, allowing anyone to decrypt credentials stored in the policy files. Anyone can decrypt the credentials because the SYSVOL folder is accessible to all 'Authenticated Users' in the domain, which includes users and computers. Microsoft published the AES private key on MSDN:

<img width="1514" height="634" alt="image" src="https://github.com/user-attachments/assets/ef979aad-643c-4e9e-a7d9-07d1a443c81a" />


Also, as a reference, this is what an example XML file containing an encrypted password looks like (note that the property is called cpassword):

<img width="2025" height="287" alt="image" src="https://github.com/user-attachments/assets/35cadbe6-b383-4099-a2df-8adca862a9d5" />

---

## 🔴 Attack

To abuse GPP Passwords, we will use the Get-GPPPassword function from PowerSploit, which automatically parses all XML files in the Policies folder in SYSVOL, picking up those with the cpassword property and decrypting them once detected:

```powershell id="n5q8rm"
PS C:\Users\bob\Downloads> Import-Module .\Get-GPPPassword.ps1
PS C:\Users\bob\Downloads> Get-GPPPassword
```

<img width="612" height="259" alt="image" src="https://github.com/user-attachments/assets/f18aa4e2-309d-411c-8f29-2dcb0c7ead16" />

<img width="1766" height="422" alt="image" src="https://github.com/user-attachments/assets/e0f2332c-b194-490c-bdd5-fff5886f0197" />

---

## 🛡️ Prevention

Once the encryption key was made public and started to become abused, Microsoft released a patch (KB2962486) in 2014 to prevent caching credentials in GPP. Therefore, GPP should no longer store passwords in new patched environments. However, unfortunately, there are a multitude of Active Directory environments built after 2015, which for some reason, do contain credentials in SYSVOL. It is therefore highly recommended to continuously assess and review the environment to ensure that no credentials are exposed here.

It is crucial to know that if an organization built its AD environment before 2014, it is likely that its credentials are still cached because the patch does not clear existing stored credentials (only prevents the caching of new ones).

---

## 🔎 Detection

There are two detection techniques for this attack:

### 1. 📄 Accessing the XML File

Accessing the XML file containing the credentials should be a red flag if we are auditing file access; this is more realistic (due to volume otherwise) regarding detection if it is a dummy XML file, not associated with any GPO. In this case, there will be no reason for anyone to touch this file, and any attempt is likely suspicious. As demonstrated by Get-GPPPasswords, it parses all of the XML files in the Policies folder. For auditing, we can generate an event whenever a user reads the file:

<img width="1743" height="1138" alt="image" src="https://github.com/user-attachments/assets/45563f0a-2015-458b-8187-33b4bbc1dd1b" />

Once auditing is enabled, any access to the file will generate an Event with the ID 4663:

<img width="1634" height="817" alt="image" src="https://github.com/user-attachments/assets/6466c269-2b42-4deb-b666-9ca20aeb0acb" />

### 2. 🔐 Logon Attempts

Logon attempts (failed or successful, depending on whether the password is up to date) of the user whose credentials are exposed is another way of detecting the abuse of this attack; this should generate one of the events 4624 (successful logon), 4625 (failed logon), or 4768 (TGT requested). A successful logon with the account from our attack scenario would generate the following event on the Domain Controller:

<img width="1521" height="1078" alt="image" src="https://github.com/user-attachments/assets/9c0dd2f0-6c7d-4ae3-8aa0-4f67586b41ac" />

In the case of a service account, we may correlate logon attempts with the device from which the authentication attempt originates, as this should be easy to detect, assuming we know where certain accounts are used (primarily if the logon originated from a workstation, which is abnormal behavior for a service account).

---

## 🪤 Honeypot

This attack provides an excellent opportunity for setting up a trap: we can use a semi-privileged user with a wrong password. Service accounts provide a more realistic opportunity because:

* The password is usually expected to be old, without recent or regular modifications.
* It is easy to ensure that the last password change is older than when the GPP XML file was last modified. If the user's password is changed after the file was modified, then no adversary will attempt to login with this account (the password is likely no longer valid).
* Schedule the user to perform any dummy task to ensure that there are recent logon attempts.

When we do the above, we can configure an alert that if any successful or failed logon attempts occur with this service account, it must be malicious (assuming that we whitelist the dummy task logon that simulates the logon activity in the alert).

Because the provided password is wrong, we would primarily expect failed logon attempts. Three event IDs (4625, 4771, and 4776) can indicate this; here is how they look for our playground environment if an attacker is attempting to authenticate with a wrong password:

### 4625

<img width="1515" height="1117" alt="image" src="https://github.com/user-attachments/assets/27278b09-8f23-46d2-8cfa-c3accaf3af5c" />

### 4771

<img width="1660" height="1125" alt="image" src="https://github.com/user-attachments/assets/08398a1a-1490-459f-ae89-5337a6d1722b" />

### 4776

<img width="1040" height="478" alt="image" src="https://github.com/user-attachments/assets/240ff0ec-01e3-4f66-b42c-c2d57df374e3" />
