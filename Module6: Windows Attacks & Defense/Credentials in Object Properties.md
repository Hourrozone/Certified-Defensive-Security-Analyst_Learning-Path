# 🔐 Credentials in Object Properties

## 📖 Description

Objects in Active Directory have a plethora of different properties; for example, a user object can contain properties that contain information such as:

* Is the account active
* When does the account expire
* When was the last password change
* What is the name of the account
* Office location for the employee and phone number

When administrators create accounts, they fill in those properties. A common practice in the past was to add the user's (or service account's) password in the Description or Info properties, thinking that administrative rights in AD are needed to view these properties. However, every domain user can read most properties of an object (including Description and Info).

---

# 🔴 Attack

A simple PowerShell script can query the entire domain by looking for specific search terms/strings in the Description or Info fields:

```powershell
Function SearchUserClearTextInformation
{
    Param (
        [Parameter(Mandatory=$true)]
        [Array] $Terms,

        [Parameter(Mandatory=$false)]
        [String] $Domain
    )

    if ([string]::IsNullOrEmpty($Domain)) {
        $dc = (Get-ADDomain).RIDMaster
    } else {
        $dc = (Get-ADDomain $Domain).RIDMaster
    }

    $list = @()

    foreach ($t in $Terms)
    {
        $list += "(`$_.Description -like `"*$t*`")"
        $list += "(`$_.Info -like `"*$t*`")"
    }

    Get-ADUser -Filter * -Server $dc -Properties Enabled,Description,Info,PasswordNeverExpires,PasswordLastSet |
        Where { Invoke-Expression ($list -join ' -OR ') } | 
        Select SamAccountName,Enabled,Description,Info,PasswordNeverExpires,PasswordLastSet | 
        fl
}
```

We will run the script to hunt for the string pass, to find the password Slavi123 in the Description property of the user bonni:

<img width="603" height="175" alt="image" src="https://github.com/user-attachments/assets/b4fd013e-9d15-4f59-ab5b-859deb593dc9" />

<img width="1383" height="340" alt="image" src="https://github.com/user-attachments/assets/0085db37-b435-4640-8061-b937b226de17" />

---

# 🛡️ Prevention

We have many options to prevent this attack/misconfiguration:

* Perform continuous assessments to detect the problem of storing credentials in properties of objects.
* Educate employees with high privileges to avoid storing credentials in properties of objects.
* Automate as much as possible of the user creation process to ensure that administrators don't handle the accounts manually, reducing the risk of introducing hardcoded credentials in user objects.

---

# 🔎 Detection

Baselining users' behavior is the best technique for detecting abuse of exposed credentials in properties of objects. Although this can be tricky for regular user accounts, triggering an alert for administrators/service accounts whose behavior can be understood and baselined is easier. Automated tools that monitor user behavior have shown increased success in detecting abnormal logons. In the example above, assuming that the provided credentials are up to date, we would expect events with event ID 4624/4625 (failed and successful logon) and 4768 (Kerberos TGT requested). Below is an example of event ID 4768:

<img width="1365" height="1080" alt="image" src="https://github.com/user-attachments/assets/6261f9d3-201a-45a6-94a4-d7fa4d502d88" />

Unfortunately, the event ID 4738 generated when a user object is modified does not show the specific property that was altered, nor does it provide the new values of properties. Therefore, we cannot use this event to detect if administrators add credentials to the properties of objects.

---

# 🪤 Honeypot

Storing credentials in properties of objects is an excellent honeypot technique for not-very-mature environments. If struggling with basic cyber hygiene, then it is more likely expected to have such issues (storing credentials in properties of objects) in an AD environment. For setting up a honeypot user, we need to ensure the followings:

* The password/credential is configured in the Description field, as it's the easiest to pick up by any adversary.
* The provided password is fake/incorrect.
* The account is enabled and has recent login attempts.
* While we can use a regular user or a service account, service accounts are more likely to have this exposed as administrators tend to create them manually. In contrast, automated HR systems often make employee accounts (and the employees have likely changed the password already).
* The account has the last password configured 2+ years ago (makes it more believable that the password will likely work).

Because the provided password is wrong, we would primarily expect failed logon attempts; three event IDs (4625, 4771, and 4776) can indicate this. Here is how they look in our playground environment if an attacker is attempting to authenticate with the account svc-iis and a wrong password:

### 4625

<img width="1515" height="1117" alt="image" src="https://github.com/user-attachments/assets/78180eac-1a87-496f-b66d-6e3984c46f62" />

### 4771

<img width="1660" height="1125" alt="image" src="https://github.com/user-attachments/assets/29b2cab7-d5f6-4d6f-ab1f-ae393839e136" />

### 4776

<img width="1040" height="478" alt="image" src="https://github.com/user-attachments/assets/4fc84310-b086-4f83-bddc-d43649b0e3bd" />

---

# 🎯 What Does “Credentials in Object Properties” Actually Mean?

**“Credentials in Object Properties”** means that sensitive information, such as passwords, has been stored inside Active Directory object properties like **Description** or **Info**.

This can be dangerous because these properties are generally readable by normal domain users, so credentials that were intended to be hidden may become accessible to anyone who knows where to look.

However, the problem is **not the Active Directory properties themselves**. These properties are useful for storing legitimate information about users and accounts. The security issue happens when administrators place sensitive credentials inside them.

The main lesson is to **never store passwords or other sensitive credentials in readable AD object properties**. Active Directory objects and their properties are useful for managing an organization, but sensitive authentication information should be stored using appropriate secure mechanisms instead.
