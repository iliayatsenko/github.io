---
title: Setting up real iOS app to request API on localhost via HTTPS through Mitmproxy
date: 2023-10-27
tags:
    - ios
    - api
    - https
    - mitmproxy
    - manual
categories:
    - EN
---

In this article I will continue the story started in [previous one](android-emulator-with-local-api-https-en.md).

Now I will set up real application running on physical iPhone to call API listening on my development laptop in the same local network. Also, I will continue use [Mitmproxy](https://mitmproxy.org) to inspect all the requests made.

### Determine the IP of your laptop in local network.
Each device in a local network has its own IP address, accessible for all other devices in the same local network (connected to the same Wi-Fi, for example). To know this IP, open the details of current Wi-Fi connection. "IP address" field should be somewhere around. Usually it looks like "192.168.*.***".

### Configure connection proxy on mobile device.
We need to tell our mobile device to send all the network traffic through proxy server. This is made quite easy. For example, on iPhone: open details of current Wi-Fi connection and press "Configure Proxy". Then select "Manual", and type the IP of your laptop from previous step as "Server", and port, on which your Mitmproxy will be listening, as "Port". As we are going to use Mitmproxy, the default port is 8080.

### Start Mitmproxy
Now install [Mitmproxy](https://mitmproxy.org) on your laptop. Then start `mitmweb` - a utility of mitmproxy which shows traffic in a browser.
```bash
mitmweb
```
Now your proxy server is listening on port 8080, and you can see all the request coming through it on http://127.0.0.1:8081.

### Install Mitmproxy CA certificate on mobile device.
Now we need to say our iPhone to trust the SSL certificate used by proxy server. Firstly we need to install it, as described in [previous post](android-emulator-with-local-api-https-en.md#1-install-mitmproxys-ca-certificate-on-your-device). On iPhone we need manually change the trust setting for installed certificate: go to the Settings, find "Certificate Trust Settings", and toggle "Enable full trust for root certificates" checkbox near "mitmproxy".

### Test
Open you application on mobile device, or any site in browser. You will need all the HTTP(S) requests made from iPhone in mitmweb panel on the laptop.

### Filter requests
There may be a lot of different request from different apps and utilities, so if you want to show only specific ones, you can add `--allow-hosts` option when running mitmproxy:
```bash
mitmweb --allow-hosts="api.myapp.com"
```
Note that despite this setting all the HTTP requests will appear in the mitmweb panel. This filter only applies to the HTTPS requests.

### Redirect specific requests
For now we can see all the network activity of our application, but what if we want to redirect all the requests to our API on localhost, for deeper debugging? This can be achieved by simple mitmproxy addon. For example, we want all requests to the "api.myapp.com" to be redirected to the "localhost:444":
1) Create a file with following content:
```python
from mitmproxy import ctx

class RedirectFromTo:
    def __init__(self):
        self.num = 0

    def load(self, loader):
        loader.add_option(
            name="redirect_from",
            typespec=str,
            default="",
            help="Host to redirect from",
        )
        loader.add_option(
            name="redirect_to",     
            typespec=str,   
            default="",
            help="Host to redirect to",          
        )

    def request(self, flow):
        if (ctx.options.redirect_from != "") and (ctx.options.redirect_to != "") and (flow.request.host == ctx.options.redirect_from):
            flow.request.host = ctx.options.redirect_to
            flow.request.port = 444

addons = [RedirectFromTo()]
```
This is a simple mitmproxy addon written in Python. It declares 2 options - `redirect_from` and `redirect_to` - and then, if both are passed, and the host of incoming request matches the value of `redirect_from`, it is substituted with `redirect_to`. Also, port is changed (hardcoded for now).

2) Save this file to `~/mitmproxy-addons/redirect-from-to.py`;
3) Run mitmproxy with addon:
```bash
mitmweb -s mitmproxy-addons/redirect-from-to.py --set redirect_from="api.myapp.com" --set redirect_to="localhost" --ssl-insecure
```
If your API on localhost is not using HTTPS, or use self-signed certificate (which is most probably true) - `--ssl-insecure` option is required; else you can omit it. Now all requests to "api.myapp.com" should be redirected to localhost.

### Modifying requests
If you need to modify all requests, you can use another addon script, as described in [previous post](android-emulator-with-local-api-https-en.md#8-modify-all-requests-automatically).

### Conclusion
As you can see, it is not so hard to inspect, redirect and modify all the network interactions of your application. You do not even need to re-build it, you can use the production version. The only thing is that it can have short connection timeouts, and you will not be able to deeply debug your API, because connection will be lost. But you can always replay any request in your mitmweb panel, so this is not a big problem.
Do not forget to reset proxy settings on your mobile device after debugging :).


