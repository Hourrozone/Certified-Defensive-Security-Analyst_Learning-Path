# HTTP Requests

## HTTP Requests

In the previous section, we found out that the `secret.js` main function is sending an empty POST request to `/serial.php`. In this section, we will attempt to do the same using cURL to send a POST request to `/serial.php`. To learn more about cURL and web requests, you can check out the Web Requests module.

## cURL

cURL is a powerful command-line tool used in Linux distributions, macOS, and even the latest Windows PowerShell versions. We can request any website by simply providing its URL, and we would get it in text-format, as follows:

<img width="748" height="470" alt="image" src="https://github.com/user-attachments/assets/0e82ef5d-7011-4016-b8a0-94bb62926d7d" />

This is the same HTML we went through when we checked the source code in the first section.

## POST Request

To send a POST request, we should add the `-X POST` flag to our command, and it should send a POST request:

```shellsession
@htb[/htb]$ curl -s http://SERVER_IP:PORT/ -X POST
```

**Tip:** We add the `"-s"` flag to reduce cluttering the response with unnecessary data.

However, POST request usually contains POST data. To send data, we can use the `"-d "param1=sample""` flag and include our data for each parameter, as follows:

```shellsession
@htb[/htb]$ curl -s http://SERVER_IP:PORT/ -X POST -d "param1=sample"
```

Now that we know how to use cURL to send basic POST requests, in the next section, we will utilize this to replicate what server.js is doing to understand its purpose better.

## 🔑 Key Takeaway

cURL can be used to replicate HTTP requests made by JavaScript functions. The `-X POST` flag sends a POST request, while `-d` includes POST data in the request.
