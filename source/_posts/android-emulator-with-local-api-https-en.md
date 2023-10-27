---
title: Setting up Android emulator to request API on localhost via HTTPS through Mitmproxy
date: 2023-10-22
tags:
    - android
    - api
    - https
    - mitmproxy
    - manual
categories:
    - EN
---

Recently I needed to make some manual acceptance testing on Android application which talks to the API I am working on. For this purpose I installed Android Studio and set up Android emulator which makes requests to the API running in docker on local host. 
<!-- more -->

But there was one major problem to solve: Android emulator requires HTTPS connection, with the SSL certificate it trusts. It was not easy to figure out all the "how?" and "why?" during solving this issue, so I decided to document my final solution here.

## General info (very simplistic)
When you run Android emulator, the default IP address to connect to local host is 10.0.2.2. Because if you specify `localhost` or `127.0.0.1` it will lead to the Android device's own local host. 

So, first of all we need to configure our API to support HTTPS. HTTPS connection requires SSL certificates, which guarantee the identities of communicating parties and is used to encrypt requests' payload. These certificates are simple files, granted by Certificate Authorities (CA). CAs are the organisations trusted by all the communicating parties. SSL certificates can be derived from other certificates, i.e. "signed by". In this process, certificate, used to sign by is "root", and the one signed is "leaf". The tree can be more than 1-level deep. The list of trusted top-most root certificates is "hard-coded" in each operating system. So, in order to make your browser or application to trust some SSL certificate, this certificate should be derived from one of these top-most root certificates. 

For use in production, you would need to contact CA to get certificate (usually they are paid). But for development, you can be a CA for yourself - use so-called "self-signed" certificates. The only problem with such certificates is that not all clients will trust them. For example, browsers will show the warning like "Your connection is not secure" when visiting site with self-signed certificate. Android emulator refuses such certificates too, by default.

What we need to do is to create self-signed certificate and tell Android emulator that this certificate should be trusted.

First of all, ensure that Android emulator trusts user-added certificates. If your app is debaggable (i.e. the build supports debug), ensure that there are such lines in the `network_security_config.xml` file in Android Studio:
```xml
<debug-overrides>
    <trust-anchors>
        <!-- Trust user added CAs while debuggable only -->
        <certificates src="user" />
    </trust-anchors>
</debug-overrides>
```
Else, if app is not debaggable, add following lines manually, to the topmost node:
```xml
<base-config>
    <trust-anchors>
        <certificates src="user"/>
    </trust-anchors>
</base-config>
```
Then follow the next steps.

## Simplest setup
### 1. Create self-signed certificate
Note the `subjectAltName` - it is required to allow to use this certificate with connections that do not specify host name. Remember, Android emulator will connect to the raw IP 10.0.2.2.
```bash
openssl req \
    -x509 -nodes -days 365 \
    -newkey rsa:2048 \
    -addext "subjectAltName = IP.1:10.0.2.2" \
    -keyout local_private.pem \
    -out local_signed.pem
```
This will create two files: private and public keys. Usually, you should not expose your private key, but in development you can put both of them to the single file, by just appending the content of certificate with content of private key:
```bash
cat local_private.pem >> local_signed.pem
```
### 2. Configure your local server to use this certificate
For Nginx, add these lines to the `server` block:
```bash
listen 443 ssl;
ssl_certificate /Absolute/path/to/local_signed.pem;
ssl_certificate_key /Absolute/path/to/local_signed.pem;
```
Nginx understands the format when private and public keys are located in single file, and will expose only the private one to clients.

Do not forget to reload nginx:
```bash
nginx -s reload
```

### 3. Copy your local_signed.pem to your virtual Android device
In Android emulator you can just drag-n-drop file to the device. 

### 4. Install your certificate on your device
To install on Android, open settings and search "certificates" -> "Certificate management app" -> "Install a certificate" -> "CA certificate". By the way, on iOS the installation is different: try to open the certificate file and after it says something like "Review profile in settings" go to Settings and select "Profile Downloaded" on the top of the settings' list.

### 5. Test
Now you can request localhost API from Android emulator using https://10.0.2.2. Change the API host in the code (possibly, help of Android developer will be needed), re-build the app, and try to use your application.


## With Mitmproxy
So far so good, but we could go further and configure the HTTPS proxy for more convenient development. With proxy we will be able to inspect, intercept and modify all the requests made from application. Also, it will be possible to route requests to different servers (for example to QA server, instead of localhost) without changing API URL in application's code and re-building it each time. 

**By the way, with proxy we do not even need to configure local API to support HTTPS, since the secured connection is required only between client and proxy, not between proxy and server.**

The good option for proxy is open-sourced [Mitmproxy](https://mitmproxy.org). After installation (is trivial), here is how to configure it.

### 1. Install Mitmproxy's CA certificate on your device
Mitmproxy goes with its own CA certificate. The file is located at `~/.mitmproxy/mitmproxy-ca.pem`. We need Android device to trust it, so should install it as described [earlier](#4-install-your-certificate-on-your-device).

### 2. Create a certificate to use
Android emulator uses IP adress 10.0.2.2 to connect to host. And the problem is that Mitmproxy's certificate is not associated with this IP (if it was, we could start using our API via proxy at this moment). The solution is to create a new certificate, signed by Mitmproxy's one, and associate it with 10.0.2.2. Create file with options for self-signed certificate:
```bash
echo "subjectAltName = IP.1:10.0.2.2" > android_options.txt
```

### 3. Create a certificate signing request
Creating a "leaf" certificate involves 2 steps - creating signing request and actual signing. To create request:
```bash
openssl req \
    -nodes -new -newkey rsa:2048 \
    -keyout mitm_private.pem \
    -out mitm_req.pem
```
This command will ask you to provide some info about your "company". You can leave all the asked fields empty, or provide some data if you want.

### 4. Create certificate signed by Mitmproxy's CA
Now, create a signed certificate using your request and options in file:
```bash
openssl x509 \
    -req -in mitm_req.pem \
    -days 365 \
    -CA ~/.mitmproxy/mitmproxy-ca.pem \
    -CAkey ~/.mitmproxy/mitmproxy-ca.pem \
    -CAcreateserial -out mitm_signed.pem \
    -extfile ./android_options.txt
```
And append secret key to your certificate file (this is the format acceptable by mitmproxy):
```bash
cat mitm_private.pem >> mitm_signed.pem
```

### 5. Start mitmproxy
Now start your proxy with created certificate. Mitmproxy provides 3 similar tools - `mitmproxy`, `mitmweb` and `mitmdump`. For seeing all requests going through proxy on a page in browser we need `mitmweb`:
```bash
mitmweb --mode reverse:https://your.server.host --certs "*=/Absolute/path/to/mitm_signed.pem"
```
If your destination server does not support HTTPS itself, or has self-signed certificate (which is true in our case), add `--ssl-insecure` option, which tells mitmproxy to not require connection to server to be secure:
```bash
mitmweb --mode reverse:https://your.server.host --ssl-insecure --certs "*=/Absolute/path/to/mitm_signed.pem"
```

### 6. Test
Now you can request localhost API from Android emulator using https://10.0.2.2:8080. By default mitmproxy listens on 8080 port, change if you configured it to listen on another one. All the requests made will be available to inspect in Mitmweb panel at http://127.0.0.1:81.

### 8. Modify all requests automatically
Now you can intercept and modify any request matching pattern (see field "Intercept" in the Mitmweb panel). But if you need to automatically modify all requests, e.g. to add Xdebug trigger to the queries, you can do it using Mitmproxy plugin (simple Python script). For example:
```bash
cat <<EOT > ~/mitmproxy-addons/add-xdebug-session-param.py
def request(flow) -> None:
    flow.request.query["XDEBUG_SESSION_START"] = "1"
EOT
```
And restart mitmproxy with plugin:
```bash
mitmweb --mode reverse:https://your.server.host --certs "*=/Absolute/path/to/mitm_signed.pem" -s ~/mitmproxy-addons/add-xdebug-session-param.py
```

## Conclusion
Configuring Android emulator with local API allows backend developers to understand the application on all the levels, not only on backend. But it requires at least basic understanding of how HTTPS works. Hope this manual will be helpful, please leave comments in case of questions or proposals.