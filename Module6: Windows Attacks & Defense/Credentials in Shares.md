<img width="1917" height="303" alt="image" src="https://github.com/user-attachments/assets/dee7c9a5-f7bc-4f6e-b488-ae1849913036" /># 🔐 Credentials in Shares

## 📖 Description

Credentials exposed in network shares are (probably) the most encountered misconfiguration in Active Directory to date. Any medium/large enterprises will undoubtedly have exposed credentials, although it may also happen in small businesses. It almost feels like we are moving from "Don't leave your password on a post-it note on your screen" to "Don't leave unencrypted credentials and authorization tokens scattered everywhere".

We often find credentials in network shares within scripts and configuration files (batch, cmd, PowerShell, conf, ini, and config). In contrast, credentials on a user's local machine primarily reside in text files, Excel sheets, or Word documents. The main difference between the storage of credentials on shares and machines is that the former poses a significantly higher risk, as it may be accessible by every user. A network share may be accessible by every user for four main reasons:

1. One admin user initially creates the shares with properly locked down access but ultimately opens it to everyone. Another admin of the server could also be the culprit. Nonetheless, the share eventually becomes open to Everyone or Users, and recall that a server's Users group contains Domain users as its member in Active Directory environments. Therefore every domain user will have at least read access (it is wrongly assumed that adding 'Users' will give access to only those local to the server or Administrators).

2. The administrator adding scripts with credentials to a share is unaware it is a shared folder. Many admins test their scripts in a scripts folder in the C:\ drive; however, if the folder is shared (for example, with Users), then the data within the scripts is also exposed on the network.

3. Another example is purposely creating an open share to move data to a server (for example, an application or some other files) and forgetting to close it later.

4. Finally, in the case of hidden shares (folders whose name ends with a dollar sign $), there is a misconception that users cannot find the folder unless they know where it exists; the misunderstanding comes from the fact that Explorer in Windows does not display files or folders whose name end with a $, however, any other tool will show it.

---

# 🔴 Attack

The first step is identifying what shares exist in a domain. There are plenty of tools available that can achieve this, such as PowerView's Invoke-ShareFinder. This function allows specifying that default shares should be filtered out (such as c$ and IPC$) and also check if the invoking user has access to the rest of the shares it finds. The final output contains a list of non-default shares that the current user account has at least read access to:

```powershell
PS C:\Users\bob\Downloads> Invoke-ShareFinder -domain eagle.local -ExcludeStandard -CheckShareAccess
```

<img width="714" height="222" alt="image" src="https://github.com/user-attachments/assets/9cc05d79-c86f-4296-b95f-023e62c75e15" />

<img width="1917" height="303" alt="image" src="https://github.com/user-attachments/assets/ab8ce996-f80c-4865-b7ea-e4197c7cb123" />

The playground environment has few shares in the domain, however, in production environments, the number can reach thousands (which requires long periods to parse).

The image above shows a share with the name dev$. Because of the dollar sign, if we were to browse the server which contains the share using Windows Explorer, we would be presented with an empty list (shares such as C$ and IPC$ even though available by default, Explorer does not display them because of the dollar sign):

<img width="1678" height="419" alt="image" src="https://github.com/user-attachments/assets/1c183b37-a528-493c-b8d2-c3048d1d7576" />

However, since we have the UNC path from the output, if we browse to it, we will be able to see the contents inside the share:

<img width="1641" height="421" alt="image" src="https://github.com/user-attachments/assets/1292d522-b390-4f95-a0f6-816f29ad9066" />

A few automated tools exist, such as SauronEye, which can parse a collection of files and pick up matching words. However, because there are few shares in the playground, we will take a more manual approach (Living Off the Land) and use the built-in command findstr for this attack. When running findstr, we will specify the following arguments:

* `/s` forces to search the current directory and all subdirectories
* `/i` ignores case in the search term
* `/m` shows only the filename for a file that matches the term. We highly need this in real production environments because of the huge amounts of text that get returned. For example, this can be thousands of lines in PowerShell scripts that contain the PassThru parameter when matching for the string pass.
* The term that defines what we are looking for. Good candidates include pass, pw, and the NETBIOS name of the domain. In the playground environment, it is eagle. Attractive targets for this search would be file types such as .bat, .cmd, .ps1, .conf, .config, and .ini. Here's how findstr can be executed to display the path of the files with a match that contains pass relative to the current location:

<img width="710" height="258" alt="image" src="https://github.com/user-attachments/assets/1340187e-527e-49cb-a865-338e3a7c203d" />

<img width="1951" height="272" alt="image" src="https://github.com/user-attachments/assets/238dc96a-6d70-40d7-bc24-854cc832734c" />

Similarly, we can execute findstr by looking for pw as the search term. Note that if we remove the `/m' argument, it will display the exact line in the file where the match occurred:

<img width="718" height="166" alt="image" src="https://github.com/user-attachments/assets/af5f6cf8-598a-495b-9cf7-3d5e38b67741" />

<img width="1932" height="135" alt="image" src="https://github.com/user-attachments/assets/ba74590b-2a1d-4083-bb88-31312b7e5236" />

One obvious and yet uncommon search term is the NetBIOS name of the domain. Commands such as runas and net take passwords as a positional argument on the command line instead of passing them via pass, pw, or any other term. It is usually defined as /user:DOMAIN\USERNAME PASSWORD. Here's the search executed in the playground domain:

<img width="708" height="181" alt="image" src="https://github.com/user-attachments/assets/ade91e98-b075-4114-a2a8-273e5fe71e4d" />

<img width="1929" height="161" alt="image" src="https://github.com/user-attachments/assets/8f3db025-bd4f-44ba-96a9-c379b7333f76" />


The last command reads the text file and displays its content, including the credentials; this would be the domain's built-in account credentials (not uncommon to find domain admin rights in these script files).

Note that running findstr with the same arguments is recently picked up by Windows Defender as suspicious behavior.

---

# 🛡️ Prevention

The best practice to prevent these attacks is to lock down every share in the domain so there are no loose permissions.

Technically, there is no way to prevent what users leave behind them in scripts or other exposed files, so performing regular scans (e.g., weekly) on AD environments to identify any new open shares or credentials exposed in older ones is necessary.

### 🔐 Share Permissions

* Apply the principle of least privilege to every network share.
* Avoid granting broad access to `Everyone`, `Users`, or other large groups unless there is a specific business requirement.
* Regularly review both **share permissions** and **NTFS permissions**, because effective access is determined by the combination of both.
* Remove obsolete shares and permissions that are no longer required.
* Review hidden shares as well. A `$` at the end of a share name should not be considered a security control.

### 🔑 Credential Storage

* Do not store plaintext passwords, authentication tokens, API keys, connection strings, or other secrets inside scripts and configuration files.
* Avoid hardcoded credentials in `.bat`, `.cmd`, `.ps1`, `.conf`, `.config`, and `.ini` files.
* Where possible, use managed identities, Group Managed Service Accounts (gMSA), or an appropriate enterprise secrets-management solution instead of embedding passwords.
* If credentials must exist temporarily in a file, ensure that access to the file is restricted and that the file is removed when it is no longer required.
* Rotate credentials immediately if they are discovered in an exposed share.

### 🔎 Regular Auditing

* Perform regular scans of network shares to identify newly created shares and unexpected permissions.
* Search existing shares for sensitive information such as passwords, tokens, connection strings, and private keys.
* Review changes to share and NTFS permissions.
* Maintain an inventory of network shares and their intended business purpose.
* Remove or remediate credentials found during periodic scans rather than simply deleting the discovered file without rotating the exposed credential.

### 🧑‍💻 Administrative Practices

* Administrators should understand whether a directory is locally stored or exposed through a network share before placing scripts or configuration files inside it.
* Temporary shares created for file transfers should be removed or locked down immediately after use.
* Avoid using privileged accounts in scripts whenever possible.
* Separate administrative scripts from general-purpose shares.
* Use dedicated service accounts with the minimum required permissions when automation requires an account.
* Ensure that administrators understand that hidden shares are not security boundaries.

### 🚨 Credential Exposure Response

If credentials are discovered in a network share, simply removing the file is not sufficient. The credential should be treated as compromised because another user or attacker may already have accessed or copied it.

A suitable response should include:

1. Remove or restrict access to the exposed file/share.
2. Identify which credentials were exposed.
3. Rotate or disable the affected credentials.
4. Determine whether the exposed account is privileged.
5. Review authentication logs for suspicious use of the account.
6. Investigate whether the exposed credentials were used to access other systems.
7. Search for additional copies of the same credentials elsewhere in the environment.
8. Document the exposure and remediate the underlying permission or administrative issue.

---

# 🔎 Detection

Understanding and analyzing users' behavior is the best detection technique for abusing discovered credentials in shares. Suppose we know the time and location of users' login via data analysis. In that case, it will be effortless to alert on seemingly suspicious behaviors—for example, the discovered account 'Administrator' in the attack described above. If we were a mature organization that used Privileged Access Workstation, we would be alert to privileged users not authenticating from those machines. These would be alerts on event IDs 4624/4625 (failed and successful logon) and 4768 (Kerberos TGT requested).

Below is an example of a successful logon with event ID 4624 for the Administrator account:

<img width="1370" height="1063" alt="image" src="https://github.com/user-attachments/assets/bf700ad0-061c-4bc4-a293-8710c0c83108" />

Similarly, if Kerberos were used for authentication, event ID 4768 would be generated:

<img width="1269" height="1219" alt="image" src="https://github.com/user-attachments/assets/8233c151-a236-443e-a071-3500c72749d6" />

Another detection technique is discovering the one-to-many connections, for example, when Invoke-ShareFinder scans every domain device to obtain a list of its network shares. It would be abnormal for a workstation to connect to 100s or even 1000s of other devices simultaneously.

---

# 🪤 Honeypot

This attack provides another excellent reason for leaving a honeypot user in AD environments: a semi-privileged username with a wrong password. An adversary eventually can detect the honeypot account if the password was changed after the file’s last modification date. Below is a good setup for the account:

* A service account that was created 2+ years ago. The last password change should be at least one year ago.
* The last modification time of the file containing the fake password must be after the last password change of the account. Because it is a fake password, there is no risk of a threat agent compromising the account.
* The account is still active in the environment.
* The script containing the credentials should be realistic. (For example, if we choose an MSSQL service account, a connection string can expose the credentials.)

Because the provided password is wrong, we would primarily expect failed logon attempts. Three event IDs (4625, 4771, and 4776) can indicate this. Here is how they look from our playground environment if an attacker is attempting to authenticate with the account svc-iis and a wrong password:

### 4625

<img width="1515" height="1117" alt="image" src="https://github.com/user-attachments/assets/8ba2032b-6bf1-411c-8066-cee24b12b3f1" />

### 4771

<img width="1660" height="1125" alt="image" src="https://github.com/user-attachments/assets/af1ca481-f449-4aa5-bfe1-05b5a9f344cd" />

### 4776

<img width="1040" height="478" alt="image" src="https://github.com/user-attachments/assets/8630efc9-0a39-4109-9ca4-f1a67ab6616f" />

---

# 🎯 What This Article Wants Students to Understand

The main lesson of this article is that **network shares can unintentionally become a major source of credential exposure in an Active Directory environment**.

Students should understand that:

* 🔎 **Finding shares is only the first step.** A share may expose scripts, configuration files, and other documents containing sensitive information.
* 🔐 **Hidden shares are not secure shares.** A `$` only prevents normal display in Windows Explorer; it does not prevent users or tools from accessing the share when the path is known.
* 🚨 **Credentials inside scripts are effectively exposed credentials.** Passwords, tokens, and connection strings should not be treated as safe simply because they are stored inside a script or configuration file.
* 🛡️ **Permissions are a major part of the security problem.** Broad share permissions can turn a single misplaced credential into a domain-wide security issue.
* 🔎 **Credential exposure and credential abuse are different stages.** An attacker may first discover credentials in a share and later authenticate with those credentials against another system.
* 📊 **Detection should focus on behavior.** Unexpected authentication by privileged accounts, unusual access patterns, and one-to-many network connections can reveal this type of activity.
* 🪤 **Honeypot credentials can provide an additional detection mechanism.** A fake credential placed in a realistic location can generate useful authentication alerts when an attacker attempts to use it.
* 🔄 **Removing the file is not enough after exposure.** Once a credential has been exposed, the credential itself should be considered compromised and rotated.

The overall goal is for students to understand the relationship between **misconfigured shares → exposed credentials → credential abuse → authentication activity → detection and prevention**.

In a real Active Directory environment, the objective is not simply to find exposed credentials. The objective is to understand **why the credentials became accessible, how the exposure could be abused, how the resulting authentication activity could be detected, and how the organization can prevent the same problem from happening again**.
