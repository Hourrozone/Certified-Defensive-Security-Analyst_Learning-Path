# Overview

In this module, we will explore several different **Active Directory attacks**. For each attack, the goal is to understand not only how the attack works, but also how defenders can prevent and detect it.

## 🎯 Module Objectives

For each attack, we will:

* **Describe the attack** and explain the underlying concept.
* **Walk through the attack** so you can understand how it can be carried out in the lab.
* **Discuss preventive techniques** and compensating controls that can reduce the risk.
* **Discuss detection capabilities** and how defenders can identify suspicious activity.
* **Discuss the honeypot approach**, where applicable, and how deliberately placed decoy resources can help detect attackers.

## 🔎 Attacks Covered in This Module

The following attacks are covered throughout the module:

* 🔴 **Kerberoasting**
* 🔴 **AS-REProasting**
* 🔴 **GPP Passwords**
* ⚠️ **Misconfigured GPO Permissions** (or GPO-deployed files)
* 🔎 **Credentials in Network Shares**
* 🔎 **Credentials in User Attributes**
* 🔴 **DCSync**
* 🔴 **Kerberos Golden Ticket**
* 🔴 **Kerberos Constrained Delegation attack**
* 🔴 **Print Spooler & NTLM Relaying**
* 🔴 **Coercing attacks & Kerberos Unconstrained Delegation**
* 🔐 **Object ACLs**
* 🔴 **PKI Misconfigurations - ESC1**
* 🔴 **PKI Misconfigurations - ESC8 (Coercing + Certificates)**

---

# 🧪 Lab Environment

Along with the explanations, this module provides a **playground environment** where you can practice the attacks yourself.

The lab allows you to follow the provided walkthroughs and see how the different attacks work in a controlled environment.

> ⚠️ **Important:** The walkthroughs are intended to demonstrate the security problem and show the general attack process. They are **not intended to explain every attack in depth**. Other modules on the platform already cover these attacks in much greater detail.

## 🖥️ Attacker Perspective

The attacks in this module are performed from two machines:

* **Windows 10 — WS001**
* **Kali Linux**

The lab assumes that the attacker has already obtained some form of **remote code execution** on the Windows 10 machine, WS001.

The compromised user is:

* **User:** Bob
* **Role:** Regular Active Directory user
* **Special permissions:** None

This means that we begin from the perspective of a normal compromised user rather than an administrator.

---

## 🌐 Lab Network

The environment contains the following machines:

| Machine        | IP Address             |
| -------------- | ---------------------- |
| **DC1**        | `172.16.18.3`          |
| **DC2**        | `172.16.18.4`          |
| **Server01**   | `172.16.18.10`         |
| **PKI**        | `172.16.18.15`         |
| **WS001**      | DHCP or `172.16.18.25` |
| **Kali Linux** | DHCP or `172.16.18.20` |

> 📌 **Note:** The IP addresses for WS001 and Kali Linux can depend on the section of the module. In some sections, these machines use DHCP addresses.

### 🧠 Lab Mental Model

The overall setup can be viewed like this:

```text
                    Active Directory Environment
                              │
             ┌────────────────┼────────────────┐
             │                │                │
           DC1              DC2            Server01
       172.16.18.3      172.16.18.4      172.16.18.10
             │
             │
            PKI
       172.16.18.15
             │
             │
       ┌─────┴─────┐
       │           │
     WS001       Kali
  172.16.18.25  172.16.18.20
       │           │
       └──── VPN ──┘
```

The important idea is that **WS001 and Kali act as the attacker-side machines**, while the other systems form the Active Directory environment being tested.

---

# 🔐 Connecting to the Lab Environment

Most of the hosts mentioned above are vulnerable to several attacks and are located inside an **isolated network**.

The network can be accessed through the **VPN**.

Once connected to the VPN, students can directly access:

* **WS001**
* **Kali Linux**

Depending on the section, either WS001, Kali, or both machines will be used during the attack scenarios.

These machines serve as the **initial foothold and attacker devices** throughout the exercises.

---

# 📝 Connecting from a Linux Host

The following sections explain how to:

1. Connect to the Windows machine **WS001**.
2. Connect to the **Kali Linux** machine.
3. Transfer files between WS001 and the Linux attacking machine.

---

# 🖥️ Connect to WS001 via RDP

Once connected to the VPN, you can access the Windows machine using **Remote Desktop Protocol (RDP)**.

Most Linux distributions provide an RDP client. One option is **`xfreerdp`**.

For this lab, we use the **Bob** account to connect to WS001.

### Credentials

```text
Username: eagle\bob
Password: Slavi123
```

To connect, execute:

```shellsession
xfreerdp /u:eagle\\bob /p:Slavi123 /v:TARGET_IP /dynamic-resolution
```

<img width="1668" height="1304" alt="image" src="https://github.com/user-attachments/assets/b6774228-939f-4230-a957-a337eeb02693" />

If the connection is successful, a new window containing the **WS001 desktop** will appear.

<img width="1027" height="836" alt="image" src="https://github.com/user-attachments/assets/4d5b6bc4-322d-4e02-a969-73e9f50b0166" />

---

# 🐧 Connect to Kali via SSH

Once connected to the VPN, you can also access the Kali Linux machine through **SSH**.

The credentials for the machine are the default Kali credentials:

```text
Username: kali
Password: kali
```

To connect using SSH:

```shellsession
ssh kali@TARGET_IP
```

<img width="1323" height="772" alt="image" src="https://github.com/user-attachments/assets/4d6438d6-6e2d-4e0e-86f5-5a33134c11ed" />

You can also access Kali through RDP:

```shellsession
xfreerdp /v:TARGET_IP /u:kali /p:kali /dynamic-resolution
```

---

# 📁 Moving Files Between WS001 and Your Linux Attacking Machine

During the exercises, you may need to move files between **WS001** and your Linux attacking machine.

To make this easier, the lab provides a **shared folder on WS001** that can be accessed through **SMB**.

<img width="973" height="805" alt="image" src="https://github.com/user-attachments/assets/05dd1cbe-f41a-4eeb-a780-3d1dbe378a9c" />

From the Kali machine, you can access this shared folder using the `smbclient` command.

Accessing the folder requires authentication, so you will need to provide valid credentials.

In this lab, the folder can be accessed using the Administrator account:

```shellsession
smbclient \\\\TARGET_IP\\Share -U eagle/administrator%Slavi123
```

<img width="739" height="218" alt="image" src="https://github.com/user-attachments/assets/b286e973-ece2-41cf-b864-875981dd346a" />

Once connected to the SMB share, you can use:

* `put` — upload a file to the shared folder.
* `get` — download a file from the shared folder.

For example, the basic workflow is:

```text
Kali/Linux
   │
   │  SMB
   ▼
WS001 Shared Folder
   │
   ├── put → Upload files
   │
   └── get → Download files
```

---

# 🔑 Key Takeaways

Before starting the individual attack sections, remember the basic lab setup:

* 🎯 **WS001** is the Windows machine used as the initial foothold in the scenarios.
* 🐧 **Kali Linux** acts as the Linux-based attacking machine.
* 🌐 The machines are located inside an isolated network accessible through the VPN.
* 👤 **Bob** is the assumed compromised user and starts as a regular Active Directory user.
* 🖥️ **RDP** can be used to access WS001.
* 🔐 **SSH** can be used to access Kali.
* 📁 **SMB** is used to transfer files between the systems.
* 🧪 The environment is designed as a controlled playground for understanding and practicing the attacks covered in this module.

With the lab environment understood, we can move on to the individual Active Directory attacks and examine how they work, how defenders can prevent them, and how suspicious activity can be detected.
