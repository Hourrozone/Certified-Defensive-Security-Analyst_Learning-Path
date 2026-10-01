# Introduction and Terminology

---

# What is Active Directory?

## 🧠 Active Directory in Simple Terms

**Active Directory (AD)** is Microsoft's directory service for Windows enterprise environments. Microsoft officially released it in 2000 with Windows Server 2000, and it has been continuously improved with newer versions of Windows Server.

At its simplest, you can think of Active Directory as the **central identity and management system for a Windows organization**.

Instead of managing every computer, user, group, and permission separately, an organization can manage them centrally through Active Directory.

AD can manage resources such as:

* 👤 Users
* 🖥️ Computers
* 👥 Groups
* 🌐 Network devices
* 📁 File shares
* ⚙️ Group Policies
* 🔐 Permissions
* 🤝 Trust relationships
* 🛠️ Services and applications

Active Directory is based on concepts and protocols that existed before it, including **X.500** and **LDAP (Lightweight Directory Access Protocol)**.

Because AD is hierarchical and distributed, organizations can structure their environment into domains, trees, forests, organizational units, and other objects.

---

## 🔐 Why Is Active Directory So Important?

Active Directory provides several critical security and management functions:

* **Authentication** — determining who a user or computer is.
* **Authorization** — determining what that user or computer is allowed to access.
* **Accounting** — tracking activities and access within the environment.
* **Centralized management** — allowing administrators to manage users, computers, policies, and permissions.

Active Directory is extremely widespread in enterprise environments, and many enterprise applications integrate directly with it.

🚨 This makes Active Directory a high-value target.

If an attacker compromises important parts of an Active Directory environment, the impact can extend far beyond a single computer. The attacker may gain access to users, computers, applications, servers, and sensitive data across the organization.

This directly affects the **CIA triad**:

* **Confidentiality** — attackers may gain access to sensitive information.
* **Integrity** — attackers may modify systems or data.
* **Availability** — attackers may disrupt or encrypt systems and data.

For example, ransomware operators may compromise an organization's environment and encrypt its data, preventing legitimate users from accessing it.

However, the bigger lesson is this:

> 🚨 An Active Directory compromise can become an organization-wide compromise rather than a compromise of a single machine.

---

## 🩹 Active Directory Vulnerabilities and Patch Management

Researchers continuously discover vulnerabilities affecting Microsoft products and Active Directory-related technologies.

One of the most important defensive practices is therefore **Patch Management**.

Proper patch management helps reduce the number of known vulnerabilities attackers can exploit.

For this module, however, we will assume that patch management is being handled correctly.

Instead, we will focus on another major category of Active Directory security problems:

* Misconfigurations
* Excessive permissions
* Dangerous default settings
* Abuse of legitimate Active Directory features
* Common attack paths
* Weak security architecture

These problems are particularly important because some of them are difficult to completely eliminate in real-world environments.

🛡️ This is where concepts such as **Defense in Depth** and **Network Segmentation** become important.

The goal is not simply to prevent every possible attack.

The goal is to make sure that when one security control fails, additional controls prevent the attacker from moving freely through the environment.

---

## 📚 Recommended Background

If this is your first time learning about Active Directory or these terms are unfamiliar, check out the **Intro to Active Directory** module for a more detailed explanation of:

* AD structure
* AD objects
* Domains
* Forests
* Organizational Units
* Groups
* Policies
* Other Active Directory concepts

You can also review **Active Directory - Enumeration and Attacks** to strengthen your understanding of Active Directory enumeration and common attack techniques.

---

# Refresher

🧠 Before continuing, let's review some of the basic Active Directory concepts.

---

## 🌐 Domain

A **domain** is a group of objects that share the same Active Directory database.

These objects can include:

* Users
* Computers
* Groups
* Other AD objects

For example:

```text
test.local
```

could represent a domain containing the organization's users and computers.

---

## 🌳 Tree

A **tree** is a collection of one or more domains that are related through a common namespace.

For example:

```text
test.local
├── staging.test.local
└── preprod.test.local
```

Here, the domains are organized under the `test.local` domain.

Multiple trees can exist within an Active Directory environment.

---

## 🌲 Forest

A **forest** is a collection of multiple Active Directory trees.

You can think of the hierarchy like this:

```text
Forest
│
├── Tree
│   ├── Domain
│   ├── Domain
│   └── Domain
│
└── Tree
    ├── Domain
    └── Domain
```

🌲 The **forest** represents the highest-level logical structure in Active Directory.

---

## 📦 Organizational Units (OU)

**Organizational Units (OUs)** are containers within Active Directory.

They can contain:

* Users
* Groups
* Computers
* Other OUs

OUs are commonly used to organize objects and apply management policies.

For example:

```text
Company
│
├── Users
├── Computers
├── Servers
└── IT
```

---

## 🤝 Trust

A **trust** allows resources in one domain to establish a relationship with resources in another domain.

In simple terms, a trust can allow users or systems from one domain to access resources in another domain, depending on the permissions assigned.

---

## 🖥️ Domain Controller

A **Domain Controller (DC)** is a server that provides critical Active Directory services.

The Domain Controller is responsible for functions such as:

* Authentication
* Authorization
* Managing Active Directory information
* Processing directory requests
* Supporting domain security

🚨 Because Domain Controllers are central to Active Directory, compromising one can have a major impact on the entire domain.

---

# Active Directory Data Store

The **Active Directory Data Store** contains the database files and processes used to store and manage directory information for users, services, and applications.

One of the most important files in an Active Directory environment is:

```text
NTDS.DIT
```

The `NTDS.DIT` file contains the Active Directory database.

By default, Domain Controllers store it in:

```text
%SystemRoot%\NTDS
```

🚨 Because `NTDS.DIT` contains critical Active Directory information, it is one of the most important files in an AD environment.

---

# What Can a Normal AD User Enumerate?

An important Active Directory security concept is that a regular domain user does not necessarily need administrative privileges to discover a significant amount of information.

A regular AD user account with no additional privileges can enumerate much of the information contained within Active Directory, including:

* Domain Computers
* Domain Users
* Domain Group Information
* Default Domain Policy
* Domain Functional Levels
* Password Policy
* Group Policy Objects (GPOs)
* Kerberos Delegation
* Domain Trusts
* Access Control Lists (ACLs)

⚠️ This does **not** automatically mean that a normal user can modify all of these objects.

It means that Active Directory's default permissions can expose a considerable amount of information to authenticated users.

Although these default behaviors can be modified or restricted, changing them incorrectly can break applications, services, and Active Directory functionality itself.

🧠 From an attacker's perspective, this information can be extremely valuable during the enumeration phase.

---

# LDAP

**LDAP (Lightweight Directory Access Protocol)** is one of the primary protocols used to communicate with Active Directory.

Domain Controllers run LDAP services and listen for LDAP requests from the network.

For example, a system can use LDAP to request information such as:

* User information
* Group membership
* Computer information
* Organizational Units
* Other directory objects

In simple terms:

```text
Client
   │
   │ LDAP Request
   ▼
Domain Controller
   │
   │ Directory Information
   ▼
Client
```

🔎 When attackers enumerate Active Directory, LDAP is therefore one of the important protocols they interact with.

---

# Authentication in Windows Environments

Windows environments support several authentication mechanisms.

Understanding these mechanisms is important because attackers frequently target credentials and authentication material.

## 🔐 Username and Password

Traditional Windows authentication can involve usernames and password-derived hashes such as:

* LM
* NTLM
* NetNTLMv1
* NetNTLMv2

Passwords are generally not transmitted directly as plain text during these authentication processes.

---

## 🎫 Kerberos

**Kerberos** is Microsoft's primary authentication protocol for Active Directory environments.

Kerberos works with the Domain Controller as a trusted third party.

Instead of repeatedly sending a username and password to every service, Kerberos uses **tickets** as cryptographic proof of identity.

A simplified workflow looks like this:

```text
User
 │
 │ Authentication Request
 ▼
Domain Controller / KDC
 │
 │ TGT
 ▼
User
 │
 │ Request service ticket
 ▼
KDC
 │
 │ TGS
 ▼
User
 │
 │ Access service
 ▼
Service
```

🎫 Kerberos authentication revolves around tickets that prove the identity of the client.

---

## 🔐 Authentication Over LDAP

LDAP can also be used for authentication.

Authentication may involve:

* Username and password
* User certificates
* Computer certificates

---

# Key Distribution Center (KDC)

The **Key Distribution Center (KDC)** is the Kerberos service installed on a Domain Controller.

The KDC is responsible for creating Kerberos tickets.

The KDC contains two major components:

* **Authentication Server (AS)**
* **Ticket-Granting Server (TGS)**

The simplified idea is:

```text
Authentication Server
        │
        │ TGT
        ▼
Ticket-Granting Server
        │
        │ TGS
        ▼
Service
```

---

# Kerberos Tickets

Kerberos tickets are tokens that serve as proof of identity.

They are created by the KDC.

## 🎫 TGT

The **Ticket Granting Ticket (TGT)** is proof that the client successfully authenticated to the KDC.

The client can then use the TGT to request service tickets.

---

## 🎫 TGS

The **Ticket Granting Service (TGS)** ticket is created for a specific service that the client wants to access.

For example, a user might request access to a service running on a server.

The KDC issues the appropriate service ticket.

---

## 🔑 KDC Key and KRBTGT

The KDC uses a key to protect the Kerberos authentication process.

Active Directory derives this key from the hashed password of the **KRBTGT account**.

The `KRBTGT` account is created when an Active Directory domain is created.

Although it is a disabled user account, it has an extremely important purpose.

The account contains secrets represented by randomly generated password hashes.

The actual password value is not something administrators normally know or manually configure.

🧠 This is why the `KRBTGT` account becomes extremely important when discussing advanced Kerberos attacks.

---

# Privileged Groups in Active Directory

Each domain contains highly privileged groups, including:

* **Domain Admins**
* **Administrators**

By default, members of **Domain Admins** are also administrators on domain-joined machines.

This provides them with broad administrative access across the domain.

The **Administrators** group of the domain can manage Active Directory objects and, by default, can log on to Domain Controllers.

A forest also contains another highly privileged group:

* **Enterprise Admins**

🚨 **Enterprise Admins** has permissions across the domains within the forest.

This means that compromising highly privileged groups can have consequences that extend beyond a single computer or even a single domain.

---

# ⚠️ Default Groups and Excessive Privileges

Active Directory contains many default groups that have significant privileges.

These groups can introduce hidden escalation paths when users are added to them without carefully considering the required permissions.

Consider the **Account Operators** group.

An organization might assign users to Account Operators because the Service Desk needs to reset user passwords.

At first, this may appear convenient.

However, giving users membership in a highly privileged built-in group violates the principle of least privilege if the users only require a small subset of those permissions.

A safer design would be to create a dedicated group and delegate only the required permissions to the appropriate Organizational Units.

🧠 The principle of least privilege means:

> Give users only the permissions they actually need.

This reduces the number of ways an attacker can move from a lower-privileged account to a highly privileged account.

One possible escalation path involves the `MSOL_` accounts created by Azure AD Connect.

These accounts are placed in the default `Users` container, where Account Operators can modify user objects.

🚨 This demonstrates why simply asking, "Who is a Domain Admin?" is not enough.

Security teams also need to understand:

```text
Who can modify whom?
Who can modify groups?
Who can modify privileged accounts?
Who can reset passwords?
Who can modify service accounts?
Who can indirectly reach Domain Admin privileges?
```

---

# Windows Logon Types

Windows has multiple **logon types**.

A logon type describes how a user or account authenticated to a system.

Examples include:

* Interactive logon
* Remote logon through RDP
* Network logon

Understanding logon types is important because different authentication methods can leave different traces and authentication material on the systems involved.

⚠️ As a general rule, logon types other than **Network Logon, Type 3** can leave credentials or credential material on the system being accessed.

Microsoft provides a complete list of Windows logon types.

🔎 For defenders, understanding these logon types is important when investigating:

* Suspicious authentication
* Lateral movement
* Credential theft
* Remote access
* Abnormal account usage

---

# Interacting With Active Directory

Active Directory lives primarily on Domain Controllers.

To communicate with Active Directory programmatically, systems can use **LDAP**.

For example, an LDAP query can request:

```text
User information
Group membership
Computer information
Organizational Units
Permissions
```

The problem is that LDAP itself is not designed to be a friendly interface for humans.

Microsoft therefore created graphical administration tools that translate administrator actions into the appropriate directory operations.

---

# Remote Server Administration Tools (RSAT)

Microsoft's **Remote Server Administration Tools (RSAT)** allow administrators to interact with Active Directory locally or remotely.

Some of the most commonly used tools include:

## 🖥️ Active Directory Users and Computers

This tool provides a graphical interface for managing objects such as:

* Users
* Groups
* Computers
* Organizational Units

Administrators can view, move, edit, and create these objects.

---

## ⚙️ Group Policy Management

Group Policy Management allows administrators to create and modify **Group Policy Objects (GPOs)**.

GPOs can control many aspects of Windows systems throughout an organization.

---

# Important Network Ports

🧠 Memorizing the important Windows and Active Directory ports is extremely useful for both defenders and penetration testers.

| Port              | Service                     |
| ----------------- | --------------------------- |
| **53**            | DNS                         |
| **88**            | Kerberos                    |
| **135**           | WMI/RPC                     |
| **137-139 & 445** | SMB                         |
| **389 & 636**     | LDAP                        |
| **3389**          | RDP                         |
| **5985 & 5986**   | PowerShell Remoting (WinRM) |

🔎 When investigating network traffic, these ports can immediately provide useful context about what type of Windows or Active Directory activity is occurring.

---

# Real-world view

## 🧠 Understanding the Bigger Picture

Every organization that has attempted to improve its security maturity will eventually need to classify its systems according to their importance to the business.

For example:

* ERP systems
* CRM systems
* Backup systems
* Authentication systems
* Critical business applications

Different organizations will classify their systems differently because their business requirements are different.

The same concept applies to Active Directory.

Any additional roles, services, and features added to the standard Active Directory environment should be classified according to their security importance.

The key question is:

> 🚨 If this service is compromised, could an attacker use it to escalate privileges into Active Directory?

If the answer is yes, that service should be treated as a critical part of the organization's security architecture.

---

# Active Directory Attack Surface

Active Directory is massive, complex, and feature-heavy.

That complexity creates many possible escalation paths.

Enterprise organizations commonly integrate additional services with Active Directory, such as:

* DNS
* PKI
* Endpoint Configuration Manager
* Other identity-related services

If an attacker gains administrative access to one of these services, they may potentially find a path toward higher Active Directory privileges.

🧠 This is why defenders cannot focus only on Domain Controllers.

They also need to understand the systems and services that can indirectly influence Active Directory.

We will demonstrate this through several attack paths later in the module.

---

# Active Directory Limitations

Active Directory has limitations that can expand the attack surface.

These limitations can come from:

* **Complexity**
* **Design**
* **Legacy compatibility**

Let's look at an example of each.

---

## 1. Complexity

One simple example is **nested group membership**.

Imagine:

```text
User
  ↓
Group A
  ↓
Group B
  ↓
Group C
  ↓
Domain Admins
```

It can become difficult to determine exactly which users ultimately inherit membership in a highly privileged group.

An organization may have multiple layers of nested groups, making permission analysis difficult.

🚨 In some environments, a user who appears to be an ordinary domain user may indirectly inherit powerful permissions through several layers of group membership.

This is why defenders need to understand **effective permissions**, not just direct group membership.

---

## 2. Design

Active Directory allows administrators to manage Windows machines remotely using **Group Policy Objects (GPOs)**.

Active Directory stores GPO-related files in a special network share called:

```text
SYSVOL
```

Domain-joined devices retrieve their Group Policy settings from Domain Controllers through this shared location.

The process normally occurs periodically, with the default Group Policy refresh interval being approximately 90 minutes.

Because clients need to communicate with Domain Controllers to retrieve these settings, the relevant network protocols must be available.

One of the important protocols involved is:

```text
SMB
```

SMB can also support remote command execution.

🚨 This creates an important security consideration:

If an attacker obtains valid privileged credentials, they may potentially use SMB-based mechanisms to execute commands remotely on systems where those credentials have sufficient privileges.

Because SMB connectivity is commonly available between domain-joined machines and Domain Controllers, privileged credentials combined with exposed remote administration functionality can create a significant attack path.

This is also one reason why traditional Active Directory environments can be difficult to align completely with **zero-trust** principles.

---

## 3. Legacy

Windows has historically been designed with a strong focus on compatibility and working out of the box for Microsoft's customers.

Some older protocols therefore remain available for compatibility purposes.

Two examples are:

* **NetBIOS**
* **LLMNR**

These protocols can provide name-resolution functionality when DNS does not successfully resolve a name.

⚠️ However, their design can create security risks.

Under certain circumstances, attackers can abuse these protocols to capture authentication attempts and credential material from systems on the network.

A simplified attack concept looks like this:

```text
Victim
  │
  │ Name-resolution request
  ▼
Attacker-controlled system
  │
  │ Captures authentication material
  ▼
Attacker
```

🚨 This means an attacker may not always need to exploit a software vulnerability to obtain credentials.

A weak or unnecessary legacy protocol can sometimes provide an opportunity simply because it is enabled and available on the network.

This is an important lesson when securing Active Directory:

> 🛡️ Security problems do not always come from vulnerabilities. They can also come from configuration, architecture, excessive privileges, legacy protocols, and the interaction between otherwise legitimate features.
