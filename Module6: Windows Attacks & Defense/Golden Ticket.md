# 🎫 Golden Ticket

## 📖 Description

The Kerberos Golden Ticket is an attack in which threat agents can create/generate tickets for any user in the Domain, therefore effectively acting as a Domain Controller.

When a Domain is created, the unique user account krbtgt is created by default; krbtgt is a disabled account that cannot be deleted, renamed, or enabled. The Domain Controller's KDC service will use the password of krbtgt to derive a key with which it signs all Kerberos tickets. This password's hash is the most trusted object in the entire Domain because it is how objects guarantee that the environment's Domain issued Kerberos tickets.

Therefore, any user possessing the password's hash of krbtgt can create valid Kerberos TGTs. Because krbtgt signs them, forged TGTs are considered valid tickets within an environment. Previously, it was even possible to create TGTs for inexistent users and assign any privileges to their accounts. Because the password's hash of krbtgt signs these tickets, the entire domain blindly trusts them, behaving as if the user(s) existed and possessed the privileges inscribed in the ticket.

The Golden Ticket attack allows us to escalate rights from any child domain to the parent in the same forest. Therefore, we can escalate to the production domain from any test domain we may have, as the domain is not a security boundary.

This attack provides means for elevated persistence in the domain. It occurs after an adversary has gained Domain Admin (or similar) privileges.

---

# 🔴 Attack

To perform the Golden Ticket attack, we can use Mimikatz with the following arguments:

* `/domain`: The domain's name.
* `/sid`: The domain's SID value.
* `/rc4`: The password's hash of krbtgt.
* `/user`: The username for which Mimikatz will issue the ticket (Windows 2019 blocks tickets if they are for inexistent users.)
* `/id`: Relative ID (last part of SID) for the user for whom Mimikatz will issue the ticket.

Additionally, advanced threat agents mostly will specify values for the `/renewmax` and `/endin` arguments, as otherwise, Mimikatz will generate the ticket(s) with a lifetime of 10 years, making it very easy to detect by EDRs:

* `/renewmax`: The maximum number of days the ticket can be renewed.
* `/endin`: End-of-life for the ticket.

First, we need to obtain the password's hash of krbtgt and the SID value of the Domain. We can utilize DCSync with Rocky's account from the previous attack to obtain the hash:

```cmd
C:\WINDOWS\system32>cd ../../../

C:\>cd Mimikatz

C:\Mimikatz>mimikatz.exe
```

<img width="661" height="797" alt="image" src="https://github.com/user-attachments/assets/19930abb-69a5-40a9-81de-7d56aec54e3f" />

<img width="667" height="771" alt="image" src="https://github.com/user-attachments/assets/a145d424-2087-4ccb-a3c3-79e9251f966b" />

We will use the Get-DomainSID function from PowerView to obtain the SID value of the Domain:

```powershell
PS C:\Users\bob\Downloads> powershell -exec bypass

Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\Users\bob\Downloads> . .\PowerView.ps1
PS C:\Users\bob\Downloads> Get-DomainSID
S-1-5-21-1518138621-4282902758-752445584
```

<img width="744" height="209" alt="image" src="https://github.com/user-attachments/assets/42939ef2-8f13-4e0a-8837-aacf8cc30c67" />

Now, armed with all the required information, we can use Mimikatz to create a ticket for the account Administrator. The `/ptt` argument makes Mimikatz pass the ticket into the current session:

```cmd
C:\Mimikatz>mimikatz.exe
```

<img width="644" height="579" alt="image" src="https://github.com/user-attachments/assets/9a6ec822-791a-4264-a526-f2bb5b5f270d" />

<img width="1073" height="544" alt="image" src="https://github.com/user-attachments/assets/76c4d9d8-f3e7-484a-8a7b-156b4fa0d1ca" />

The output shows that Mimikatz injected the ticket in the current session, and we can verify that by running the command klist (after exiting from Mimikatz):

```cmd
mimikatz # exit

Bye!

C:\Mimikatz>klist
```

<img width="634" height="311" alt="image" src="https://github.com/user-attachments/assets/21f798a4-5390-4e38-8f3a-efd8f4f086d1" />

<img width="694" height="381" alt="image" src="https://github.com/user-attachments/assets/18f3885f-decf-471f-8f34-f4b873ec4e6f" />

hat the ticket is working, we can list the content of the C$ share of DC1 using it:

```cmd
C:\Mimikatz>dir \\dc1\c$
```

<img width="598" height="341" alt="image" src="https://github.com/user-attachments/assets/5287cc21-14df-43ce-93c5-77de4e889996" />

<img width="510" height="270" alt="image" src="https://github.com/user-attachments/assets/2dfe69a1-d523-4d37-ae8a-116bcfe6ed11" />

---

# 🛡️ Prevention

Preventing the creation of forged tickets is difficult as the KDC generates valid tickets using the same procedure. Therefore, once an attacker has all the required information, they can forge a ticket. Nonetheless, there are a few things we can and should do:

* Block privileged users from authenticating to any device.
* Periodically reset the password of the krbtgt account; the secrecy of this hash value is crucial to Active Directory. When resetting the password of krbtgt (regardless of the password's strength), it will always be overwritten with a new randomly generated and cryptographically secure one. Utilizing Microsoft's script for changing the password of krbtgt KrbtgtKeys.ps1 is highly recommended as it has an audit mode that checks the domain for preventing impacts upon password change. It also forces DC replication across the globe so all Domain Controllers sync the new value instantly, reducing potential business disruptions.
* Enforce SIDHistory filtering between the domains in forests to prevent the escalation from a child domain to a parent domain (because the escalation path involves abusing the SIDHistory property by setting it to that of a privileged group, for example, Enterprise Admins). However, doing this may result in potential issues in migrating domains.

### 🔐 Protect the krbtgt Account

The `krbtgt` account is especially important because its password hash is used to sign Kerberos tickets. Access to this hash should therefore be treated as highly sensitive.

Organizations should carefully monitor privileged access to Domain Controllers and investigate unexpected attempts to obtain credentials or replication data associated with the `krbtgt` account.

### 🔄 Plan krbtgt Password Resets

Resetting the `krbtgt` password should be treated as a planned security operation rather than an emergency action performed without preparation. Domain Controllers need to receive the updated password information, and administrators should consider ticket lifetimes and replication before performing the reset.

### 🧑‍💻 Limit Privileged Authentication

Privileged accounts should be restricted from logging on to ordinary workstations whenever possible. Using dedicated administrative systems such as Privileged Access Workstations (PAWs) reduces the number of machines from which highly privileged credentials can potentially be exposed.

### 🔎 Monitor Privileged Account Activity

Organizations should establish normal authentication patterns for privileged accounts and investigate unexpected authentication from unusual systems, locations, or times. This does not directly prevent ticket forging, but it can reduce the time an attacker can remain undetected after obtaining the required secrets.

---

# 🔎 Detection

Correlating users' behavior is the best technique to detect abuse of forged tickets. Suppose we know the location and time a user regularly uses to log in. In that case, it will be easy to alert on other (suspicious) behaviors—for example, consider the account 'Administrator' in the attack described above. If a mature organization uses Privileged Access Workstations (PAWs), they should be alert to any privileged users not authenticating from those machines, proactively monitoring events with the ID 4624 and 4625 (successful and failed logon).

Domain Controllers will not log events when a threat agent forges a Golden Ticket from a compromised machine. However, when attempting to access another system(s), we will see events for successful logon originating from the compromised machine:

<img width="1555" height="1346" alt="image" src="https://github.com/user-attachments/assets/48155c97-a096-42a1-90b9-c228cdfcf2a2" />

Another detection point could be a TGS service requested for a user without a previous TGT. However, this can be a tedious task due to the sheer volume of tickets (and many other factors). If we go back to the attack scenario, by running `dir \\dc1\c$` at the end, we generated two TGS tickets on the Domain Controller:

### Ticket 1:

<img width="1511" height="857" alt="image" src="https://github.com/user-attachments/assets/be9a0fcc-4162-412f-b6ec-063a427a7f8b" />

### Ticket 2:

<img width="1373" height="862" alt="image" src="https://github.com/user-attachments/assets/3e7c0547-d637-4386-af80-0313a755bb31" />

The only difference between the tickets is the service. However, they are ordinary compared to the same events not associated with the Golden Ticket.

If SID filtering is enabled, we will get alerts with the event ID 4675 during cross-domain escalation.

---

# 📌 Note

If an Active Directory forest has been compromised, we need to reset all users' passwords and revoke all certificates, and for krbtgt, we must reset its password twice (in every domain). The password history value for the krbtgt account is 2. Therefore it stores the two most recent passwords. By resetting the password twice, we effectively clear any old passwords from the history, so there is no way another DC will replicate this DC by using an old password. However, it is recommended that this password reset occur at least 10 hours apart from each other (maximum user ticket lifetime); otherwise, expect some services to break if done in a shorter period.

---

# 🎯 What Does “Golden Ticket” Actually Mean?

A **Golden Ticket** is essentially a forged Kerberos authentication ticket created using the secret associated with the `krbtgt` account. Because the Domain Controller trusts tickets signed with that secret, an attacker who obtains the `krbtgt` hash can create tickets that the domain accepts as legitimate.

This is why protecting the `krbtgt` credentials is extremely important. The attack does not mean Kerberos itself is broken; instead, it abuses the trust that Kerberos places in tickets signed by the domain's `krbtgt` secret.

The main lesson is that **compromise of `krbtgt` can become a domain-wide security problem**, so protecting privileged credentials, controlling access to Domain Controllers, monitoring unusual authentication, and properly resetting `krbtgt` are important parts of defending an Active Directory environment.
