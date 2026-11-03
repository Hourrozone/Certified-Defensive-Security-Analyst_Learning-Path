# Strange HTTP Headers

**Related PCAP File(s):**

- `CRLF_and_host_header_manipulation.pcapng`

We might not notice anything like fuzzing right away when analyzing our web server's traffic. However, this does not always indicate that nothing bad is happening. Instead, we can always look a little bit deeper. In order to do so, we might look for strange behavior among HTTP requests. Some of which are weird headers like:

- Weird Hosts (Host: )
- Unusual HTTP Verbs
- Changed User Agents

## Finding Strange Host Headers

In order to start, as we would normally do, we can limit our view in Wireshark to only http replies and requests.

```text
http
```

<img width="979" height="409" alt="image" src="https://github.com/user-attachments/assets/9f7be242-d241-4baa-ae5c-addfdffe099c" />

Then, we can find any irregular Host headers with the following command. We specify our web server's real IP address to exclude any entries which use this real header. If we were to do this for an external web server, we could specify the domain name here.

```text
http.request and (!(http.host == "192.168.10.7"))
```

<img width="940" height="342" alt="image" src="https://github.com/user-attachments/assets/7ac1af19-2362-4b62-be48-1f44aa3c7c54" />

Suppose we noticed that this filter returned some results, we could dig into these HTTP requests a little deeper to find out what hosts these bad actors might have tried to use. We might commonly notice `127.0.0.1`.

<img width="1045" height="313" alt="image" src="https://github.com/user-attachments/assets/c7b6160f-c703-4d78-95a0-1bae8547ec65" />

Or instead something like `admin`.

<img width="1047" height="315" alt="image" src="https://github.com/user-attachments/assets/1e2f4096-e065-4635-b267-c1f7b1ae8b38" />

Attackers will attempt to use different host headers to gain levels of access they would not normally achieve through the legitimate host. They may use proxy tools like burp suite or others to modify these before sending them to the server. In order to prevent successful exploitation beyond only detecting these events, we should always do the following.

- Ensure that our virtualhosts or access configurations are setup correctly to prevent this form of access.
- Ensure that our web server is up to date.

## Analyzing Code 400s and Request Smuggling

We might also notice some bad responses from our web server, like code 400s. These codes indicate a bad request from the client, so they can be a good place to start when detecting malicious actions via http/https. In order to filter for these, we can use the following:

```text
http.response.code == 400
```

<img width="896" height="162" alt="image" src="https://github.com/user-attachments/assets/465ee864-c04a-4367-a346-c2a05e6a2702" />

Suppose we were to follow one of these HTTP streams, we might notice the following from the client.

<img width="1249" height="111" alt="image" src="https://github.com/user-attachments/assets/b5507fd6-6675-453f-8f79-da6b55438871" />

This is commonly referred to as HTTP request smuggling or CRLF (Carriage Return Line Feed). Essentially, an attacker will try the following.

```text
GET%20%2flogin.php%3fid%3d1%20HTTP%2f1.1%0d%0aHost%3a%20192.168.10.5%0d%0a%0d%0aGET%20%2fuploads%2fcmd2.php%20HTTP%2f1.1%0d%0aHost%3a%20127.0.0.1%3a8080%0d%0a%0d%0a%20HTTP%2f1.1 Host: 192.168.10.5
```

Which will be decoded by our server like this.

```text
GET /login.php?id=1 HTTP/1.1
Host: 192.168.10.5

GET /uploads/cmd2.php HTTP/1.1
Host: 127.0.0.1:8080

 HTTP/1.1
Host: 192.168.10.5
```

Essentially, in cases where our configurations are vulnerable, the first request will go through, and the second request will as well shortly after. This can give an attacker levels of access that we would normally prohibit. This occurs due to our configuration looking like the following.

### Apache Configuration

```apache
<VirtualHost *:80>

    RewriteEngine on
    RewriteRule "^/categories/(.*)" "http://192.168.10.100:8080/categories.php?id=$1" [P]
    ProxyPassReverse "/categories/" "http://192.168.10.100:8080/"

</VirtualHost>
```

**CVE-2023-25690**

As such watching for these code 400s can give clear indication to adversarial actions during our traffic analysis efforts. Additionally, we would notice if an attacker is successful with this attack by finding the code 200 (success) in response to one of the requests which look like this.

## 🔑 Key Takeaway

Strange HTTP headers, unexpected Host values, and unusual request patterns can indicate attempts to manipulate web server behavior. Monitoring irregular Host headers and HTTP 400 responses can help identify potential HTTP request smuggling or CRLF attacks. Proper virtual host and access configurations, along with keeping web servers up to date, can help reduce the risk of exploitation.
