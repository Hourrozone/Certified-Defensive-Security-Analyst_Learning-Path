# 🔄 DCSync

## 📖 Description

DCSync is an attack that threat agents utilize to impersonate a Domain Controller and perform replication with a targeted Domain Controller to extract password hashes from Active Directory. The attack can be performed both from the perspective of a user account or a computer, as long as they have the necessary permissions assigned, which are:

* **Replicating Directory Changes**
* **Replicating Directory Changes All**

---

# 🔴 Attack

We will utilize the user Rocky (whose password is Slavi123) to showcase the DCSync attack. When we check the permissions for Rocky, we see that he has Replicating Directory Changes and Replicating Directory Changes All assigned:

<img width="479" height="561" alt="image" src="https://github.com/user-attachments/assets/97f42b13-dde3-4a49-b564-977d02bde6d2" />

First, we need to start a new command shell running as Rocky:

```cmd
C:\Users\bob\Downloads>runas /user:eagle\rocky cmd.exe

Enter the password for eagle\rocky:
Attempting to start cmd.exe as user "eagle\rocky" ...
```

<img width="691" height="111" alt="image" src="https://github.com/user-attachments/assets/028c3715-c7d2-4333-a036-006a43e6d81c" />

<img width="582" height="195" alt="image" src="https://github.com/user-attachments/assets/0004e381-4caf-4db8-8e2d-11e7baf07958" />

Subsequently, we need to use Mimikatz, one of the tools with an implementation for performing DCSync. We can run it by specifying the username whose password hash we want to obtain if the attack is successful, in this case, the user 'Administrator':

```cmd
C:\Mimikatz>mimikatz.exe

mimikatz # lsadump::dcsync /domain:eagle.local /user:Administrator
```

<img width="653" height="878" alt="image" src="https://github.com/user-attachments/assets/0ad2aeb7-d927-4d89-a440-c7560dc386a5" />

<img width="671" height="425" alt="image" src="https://github.com/user-attachments/assets/03e66a36-6802-4dcb-a6b4-0adacde82619" />

It is possible to specify the /all parameter instead of a specific username, which will dump the hashes of the entire AD environment. We can perform pass-the-hash with the obtained hash and authenticate against any Domain Controller.

---

# 🛡️ Prevention

What DCSync abuses is a common operation in Active Directory environments, as replications happen between Domain Controllers all the time; therefore, preventing DCSync out of the box is not an option. The only prevention technique against this attack is using solutions such as the RPC Firewall, a third-party product that can block or allow specific RPC calls with robust granularity. For example, using RPC Firewall, we can only allow replications from Domain Controllers.

### 🔐 Control Replication Permissions

The most important defensive measure is to carefully control which users and computers have **Replicating Directory Changes** and **Replicating Directory Changes All** permissions. These permissions should only be assigned when there is a legitimate business requirement.

Regularly reviewing these permissions can help identify accounts that have unnecessary replication privileges.

### 🛡️ Protect Privileged Accounts

Accounts with high privileges should be protected carefully because compromising one of these accounts may provide an attacker with the permissions required to perform DCSync.

Organizations should follow the principle of least privilege and avoid giving normal user accounts unnecessary administrative or replication permissions.

### 🔎 Regular Permission Auditing

Active Directory permissions should be reviewed regularly to identify unexpected changes or accounts that have been granted replication rights.

This is especially important because an attacker who gains control of an account with replication permissions may be able to perform DCSync without directly compromising a Domain Controller.

### ⚙️ Monitor Changes to Sensitive Permissions

Changes to permissions involving replication rights should be monitored and investigated when they are unexpected. An attacker may attempt to grant an account the required replication permissions before performing DCSync.

Combining permission monitoring with the **4662** detection described below provides a stronger defensive approach.

---

# 🔎 Detection

Detecting DCSync is easy because each Domain Controller replication generates an event with the ID 4662. We can pick up abnormal requests immediately by monitoring for this event ID and checking whether the initiator account is a Domain Controller. Here's the event generated from earlier when we ran Mimikatz; it serves as a flag that a user account is performing this replication attempt:

<img width="1034" height="1093" alt="image" src="https://github.com/user-attachments/assets/471b60d0-06dc-4e0b-a8d4-4b6c6515205f" />

Since replications occur constantly, we can avoid false positives by ensuring the followings:

* Either the property `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` or `1131f6ad-9c07-11d1-f79f-00c04fc2dcd2` is present in the event.
* Whitelisting systems/accounts with a (valid) business reason for replicating, such as Azure AD Connect (this service constantly replicates Domain Controllers and sends the obtained password hashes to Azure AD).

---

# 🎯 What Does “DCSync” Actually Mean?

**DCSync** is a technique that abuses Active Directory's normal replication functionality to make a user or computer act like a Domain Controller and request password-related data from another Domain Controller.

The important point is that **DCSync does not need to compromise a Domain Controller directly**. If an account has the required replication permissions, it may be able to request this sensitive information.

Active Directory replication itself is necessary for the domain to work, so the goal is not to disable replication. The security focus is on **controlling who has replication permissions and detecting unexpected accounts attempting to perform replication**.

In simple terms, DCSync shows how a legitimate Active Directory feature can become dangerous when **excessive or unnecessary permissions** are assigned to an account.
