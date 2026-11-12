# Detecting Password Spraying

## Password Spraying

Unlike traditional brute-force attacks, where an attacker tries numerous passwords for a single user account, password spraying distributes the attack across multiple accounts using a limited set of commonly used or easily guessable passwords. The primary goal is to evade account lockout policies typically instituted by organizations. These policies usually lock an account after a specified number of unsuccessful login attempts to thwart brute-force attacks on individual accounts. However, password spraying lowers the chance of triggering account lockouts, as each user account receives only a few password attempts, making the attack less noticeable.

An example of password spraying using the Spray tool can be seen below

<img width="1051" height="573" alt="image" src="https://github.com/user-attachments/assets/b8c6f449-4852-4b3d-9652-4701fc29f51e" />

## Password Spraying Detection Opportunities

Detecting password spraying through Windows logs involves the analysis and monitoring of specific event logs to identify patterns and anomalies indicative of such an attack. A common pattern is multiple failed logon attempts with Event ID `4625` - Failed Logon from different user accounts but originating from the same source IP address within a short time frame.

Other event logs that may aid in password spraying detection include:

| Event ID | Description |
|---|---|
| `4768` and ErrorCode `0x6` | Kerberos Invalid Users |
| `4768` and ErrorCode `0x12` | Kerberos Disabled Users |
| `4776` and ErrorCode `0xC0000064` | NTLM Invalid Users |
| `4776` and ErrorCode `0xC000006A` | NTLM Wrong Password |
| `4648` | Authenticate Using Explicit Credentials |
| `4771` | Kerberos Pre-Authentication Failed |

## Detecting Password Spraying With Splunk

Now let's explore how we can identify password spraying attempts, using Splunk.

**Timeframe:** `earliest=1690280680 latest=1690289489`

<img width="722" height="120" alt="image" src="https://github.com/user-attachments/assets/94e76cde-fd61-47a3-95c9-e00c3bfb7626" />

<img width="1900" height="1264" alt="image" src="https://github.com/user-attachments/assets/4f8ad167-f939-4ee3-a884-61d601552cc9" />

### Search Breakdown

1. **Filtering by Index, Source, and EventCode:** The search starts by selecting events from the main index where the source is `WinEventLog:Security` and the `EventCode` is `4625`. This EventCode represents failed logon attempts in the Windows Security Event Log.

2. **Time Range Filter:** The search restricts the time range of events to those occurring between the Unix timestamps `1690280680` and `1690289489`. These timestamps represent the earliest and latest times in which the events occurred.

3. **Time Binning:** The `bin` command is used to create time buckets of 15 minutes duration for each event based on the `_time` field. This step groups the events into 15-minute intervals, which can be useful for analyzing patterns or trends over time.

4. **Statistics:** The `stats` command is used to aggregate events based on the fields `src`, `Source_Network_Address`, `dest`, `EventCode`, and `Failure_Reason`. For each unique combination of these fields, the search calculates the following statistics:
   - `values(user) as Users`: All unique values of the `user` field within each group.
   - `dc(user) as dc_user`: The distinct count of unique values of the `user` field within each group. This represents the number of different users associated with the failed logon attempts in each group.

---

## 🔑 Key Takeaway

Password spraying attempts to avoid account lockouts by testing a small number of passwords across multiple user accounts. Detection involves identifying patterns of failed authentication events, particularly Event ID `4625`, where multiple accounts experience failures from the same source IP within a short time frame. Splunk can help correlate these events using time-based grouping and distinct user counts to identify potential password spraying activity.
