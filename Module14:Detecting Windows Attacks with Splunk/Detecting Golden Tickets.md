# Detecting Golden Tickets

## Introduction

Previously in this section, we covered Golden Tickets. Unfortunately, Zeek lacks the ability to trustworthily identify Golden Tickets. Therefore, we will concentrate our Splunk search on uncovering anomalies in Kerberos ticket creation.

In a Golden Ticket or Pass-the-Ticket attack, the attacker bypasses the usual Kerberos authentication process, which involves the AS-REQ and AS-REP messages.

In a typical Kerberos authentication process, a client begins by sending an AS-REQ (Authentication Service Request) message to the Key Distribution Center (KDC), specifically the Authentication Service (AS), requesting a Ticket Granting Ticket (TGT). The KDC responds with an AS-REP (Authentication Service Response) message, which includes the TGT if the client's credentials are valid. The client can then use the TGT to request service tickets (Ticket Granting Service tickets, or TGS) for specific services on the network.

In a Golden Ticket attack, the attacker generates a forged TGT, which grants them access to any service on the network without having to authenticate with the KDC. Since the attacker has a forged TGT, they can directly request TGS tickets without going through the AS-REQ and AS-REP process.

In a Pass-the-Ticket attack, the attacker steals a valid TGT or TGS ticket from a legitimate user (for example, by compromising their machine) and then uses that ticket to access services on the network as if they were the legitimate user. Again, since the attacker already has a valid ticket, they can bypass the AS-REQ and AS-REP process.

## How Golden Ticket Traffic Looks Like

<img width="929" height="181" alt="image" src="https://github.com/user-attachments/assets/6a65e422-86a1-4adb-840b-9b9b0acc0c76" />

## Related Evidence

- **Related Directory:** `/home/htb-student/module_files/golden_ticket_attack`
- **Related Splunk Index:** `golden_ticket_attack`
- **Related Splunk Sourcetype:** `bro:kerberos:json`

## Detecting Golden Tickets With Splunk & Zeek Logs

Now let's explore how we can identify Golden Tickets, using Splunk and Zeek logs.

```shellsession id="f3k9vx"
index="golden_ticket_attack" sourcetype="bro:kerberos:json"
| where client!="-"
| bin _time span=1m 
| stats values(client), values(request_type) as request_types, dc(request_type) as unique_request_types by _time, id.orig_h, id.resp_h
| where request_types=="TGS" AND unique_request_types==1
```

<img width="1325" height="355" alt="image" src="https://github.com/user-attachments/assets/95789c80-64d9-4f39-9d82-ba9afb825843" />

## Search Breakdown

- **`index="golden_ticket_attack" sourcetype="bro:kerberos:json"`:** This line specifies the data source the query is searching. It's looking for events in the golden_ticket_attack index where the sourcetype (data format) is bro:kerberos:json.

- **`| where client!="-"`:** This line filters out events where the client field is equal to -. This is to remove noise from the data by excluding events where the client information is not available.

- **`| bin _time span=1m`:** This line divides the data into one-minute intervals based on the `_time` field, which is the timestamp of each event. This is used to analyze patterns of activity within each one-minute window.

- **`| stats values(client), values(request_type) as request_types, dc(request_type) as unique_request_types by _time, id.orig_h, id.resp_h`:** This line aggregates the data by the minute, source IP address (`id.orig_h`), and destination IP address (`id.resp_h`). It calculates the following for each combination of these grouping fields:
  - `values(client)`: All the unique client values associated with the events.
  - `values(request_type) as request_types`: All the unique request types associated with the events.
  - `dc(request_type) as unique_request_types`: The distinct count of request types.

- **`| where request_types=="TGS" AND unique_request_types==1`:** This line filters the results to only show those where the only request type is TGS (Ticket Granting Service), and there's only one unique request type.

## 🔑 Key Takeaway

Golden Ticket and Pass-the-Ticket attacks can bypass the normal AS-REQ and AS-REP authentication process because attackers already possess or have forged a valid Kerberos ticket. The Splunk query identifies potential anomalies by finding clients that generate only TGS requests within a one-minute interval, without corresponding AS request activity.
