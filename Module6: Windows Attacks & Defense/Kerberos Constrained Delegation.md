# 🎫 Kerberos Constrained Delegation

## 📖 Description

Kerberos Delegation enables an application to access resources hosted on a different server; for example, instead of giving the service account running the web server access to the database directly, we can allow the account to be delegated to the SQL server service. Once a user logs into the website, the web server service account will request access to the SQL server service on behalf of that user, allowing the user to get access to the content in the database that they’ve been provisioned to without having to assign any access to the web server service account itself.

We can configure three types of delegations in Active Directory:

* **Unconstrained Delegation (most permissive/broad)**
* **Constrained Delegation**
* **Resource-based Delegation**

Knowing and understanding that any type of delegation is a possible security risk is paramount, and we should avoid it unless necessary.

As the name suggests, unconstrained delegation is the most permissive, allowing an account to delegate to any service. In constrained delegation, a user account will have its properties configured to specify which service(s) they can delegate. For resource-based delegation, the configuration is within the computer object to whom delegation occurs. In that case, the computer is configured as I trust only this/these accounts. It is rare to see Resource-based delegation configured by an Administrator in production environments ( threat agents often abuse it to compromise devices). However, Unconstrained and Constrained delegations are commonly encountered in production environments.

---

# 🔴 Attack

We will only showcase the abuse of constrained delegation; when an account is trusted for delegation, the account sends a request to the KDC stating, "Give me a Kerberos ticket for user YYYY because I am trusted to delegate this user to service ZZZZ", and a Kerberos ticket is generated for user YYYY (without supplying the password of user YYYY). It is also possible to delegate to another service, even if not configured in the user properties. For example, if we are trusted to delegate for LDAP, we can perform protocol transition and be entrusted to any other service such as CIFS or HTTP.

To demonstrate the attack, we assume that the user web_service is trusted for delegation and has been compromised. The password of this account is Slavi123. To begin, we will use the Get-NetUser function from PowerView to enumerate user accounts that are trusted for constrained delegation in the domain:

> ⚠️ **Note:** Throughout the exercise, please use the PowerView-main.ps1 located in `C:\Users\bob\Downloads` when enumerating with the `-TrustedToAuth` parameter.

```powershell id="v4s9g2"
PS C:\Users\bob\Downloads> Get-NetUser -TrustedToAuth
```

<img width="632" height="758" alt="image" src="https://github.com/user-attachments/assets/b0637b25-b75f-478f-a660-c888e5c79dcb" />

<img width="1201" height="563" alt="image" src="https://github.com/user-attachments/assets/d52a77e1-a908-4459-ac18-f09bd39d4aec" />

We can see that the user web_service is configured for delegating the HTTP service to the Domain Controller DC1. The HTTP service provides the ability to execute PowerShell Remoting. Therefore, any threat actor gaining control over web_service can request a Kerberos ticket for any user in Active Directory and use it to connect to DC1 over PowerShell Remoting.

Before we request a ticket with Rubeus (which expects a password hash instead of cleartext for the `/rc4` argument used subsequently), we need to use it to convert the plaintext password Slavi123 into its NTLM hash equivalent:

```powershell id="0y1s6h"
PS C:\Users\bob\Downloads> .\Rubeus.exe hash /password:Slavi123
```

<img width="658" height="349" alt="image" src="https://github.com/user-attachments/assets/b09e3101-38b5-4884-b5e0-80e808ac127d" />

<img width="991" height="370" alt="image" src="https://github.com/user-attachments/assets/71024381-7c39-464c-9ab6-ef97a2348413" />

Then, we will use Rubeus to get a ticket for the Administrator account:

```powershell id="n7x4cp"
PS C:\Users\bob\Downloads> .\Rubeus.exe s4u /user:webservice /rc4:FCDC65703DD2B0BD789977F1F3EEAECF /domain:eagle.local /impersonateuser:Administrator /msdsspn:"http/dc1" /dc:dc1.eagle.local /ptt
```

<img width="628" height="525" alt="image" src="https://github.com/user-attachments/assets/e4333e19-3f62-4531-8aae-ed986d7f27e2" />

<img width="1423" height="366" alt="image" src="https://github.com/user-attachments/assets/8caa86c1-ae94-42b3-9ec1-f0bb71ac5e0b" />

To confirm that Rubeus injected the ticket in the current session, we can use the klist command:

```powershell id="2d8j5m"
PS C:\Users\bob\Downloads> klist
```

<img width="1211" height="328" alt="image" src="https://github.com/user-attachments/assets/6cd06df0-cf18-4882-b3cb-da1fc4c09e9d" />

With the ticket being available, we can connect to the Domain Controller impersonating the account Administrator:

<img width="637" height="144" alt="image" src="https://github.com/user-attachments/assets/cedaa547-f8ae-4f5e-928c-ec8714cf989f" />

If the last step fails (we may need to do klist purge, obtain new tickets, and try again by rebooting the machine). We can also request tickets for multiple services with the `/altservice` argument, such as LDAP, CFIS, time, and host.

---

# 🛡️ Prevention

Fortunately, when designing Kerberos Delegation, Microsoft implemented several protection mechanisms; however, it did not enable them by default to any user account. There are two direct ways to prevent a ticket from being issued for a user via delegation:

* Configure the property **Account is sensitive and cannot be delegated** for all privileged users.
* Add privileged users to the **Protected Users** group: this membership automatically applies the protection mentioned above (however, it is not recommended to use Protected Users without first understanding its potential implications).

We should treat any account configured for delegation as extremely privileged, regardless of its actual privileges (such as being only a Domain user). Cryptographically secure passwords are a must, as we don't want Kerberoasting giving threat agents an account with delegation privileges.

### 🔐 Limit Delegation

Delegation should only be configured for accounts that genuinely require it. The services and systems to which an account can delegate should also be kept as limited as possible.

This follows the principle of least privilege: an account trusted for delegation should not have more delegation capability than its application requires.

### 🧑‍💻 Protect Delegated Accounts

Accounts configured for delegation should receive additional security attention because compromising them can provide an attacker with capabilities beyond the account's normal privileges.

Strong, unique passwords and regular credential reviews can reduce the risk of an attacker obtaining control of these accounts.

### 🔎 Regularly Review Delegation Configuration

Active Directory should be regularly reviewed for accounts configured with delegation. Administrators should verify that each delegated account still has a legitimate business requirement and remove delegation when it is no longer needed.

### 🚫 Protect Privileged Users

Privileged accounts should be configured as **sensitive and cannot be delegated** where appropriate. This helps prevent delegated services from obtaining tickets that impersonate highly privileged users.

---

# 🔎 Detection

Correlating users' behavior is the best technique to detect constrained delegation abuse. Suppose we know the location and time a user regularly uses to log in. In that case, it will be easy to alert on other (suspicious) behaviors—for example, consider the account 'Administrator' in the attack described above. If a mature organization uses Privileged Access Workstations (PAWs), they should be alert to any privileged users not authenticating from those machines, proactively monitoring events with the ID 4624 (successful logon).

In some occasions, a successful logon attempt with a delegated ticket will contain information about the ticket's issuer under the Transited Services attribute in the events log. This attribute is normally populated if the logon resulted from an S4U (Service For User) logon process.

S4U is a Microsoft extension to the Kerberos protocol that allows an application service to obtain a Kerberos service ticket on behalf of a user; if we recall from the attack flow when utilizing Rubeus, we specified this S4U extension. Here is an example logon event by using the web service to generate a ticket for the user Administrator, which then was used to connect to the Domain Controller (precisely as the attack path above):

<img width="1601" height="1380" alt="image" src="https://github.com/user-attachments/assets/b98de8ab-0ea8-4c63-9722-a724be6ff997" />

---

# 🎯 What Does “Kerberos Constrained Delegation” Actually Mean?

**Kerberos Constrained Delegation** is a legitimate Active Directory feature that allows a service to access another service **on behalf of a user**, without requiring the user's password.

It is useful for applications where one service needs to access another service on behalf of the user. However, if an account configured for delegation is compromised, an attacker may be able to abuse that trust to impersonate other users to the services it is allowed to access.

The important lesson is that **delegation is useful, but it creates additional trust relationships**. Organizations should therefore limit delegation to what is actually required and protect accounts configured for delegation, especially when privileged users could potentially be impersonated.
