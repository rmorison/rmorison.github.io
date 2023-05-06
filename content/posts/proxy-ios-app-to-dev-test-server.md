+++
title = "Proxy an iOS App to a Dev or Test Server"
author = ["Rod Morison"]
date = 2023-05-06T14:02:00-07:00
draft = false
topics = ["QA & Test", "Backend Dev", "Networking"]
description = "How to get a mobile app with hard wired API urls to talk to a dev or test server with mitmproxy"
+++

{{< figure src="/ox-hugo/grayscale_photo_of_street_sign-scopio-7b8bd990-d4d6-4895-9673-d80afd80a853-cropped.jpg" >}}


## DISCLAIMER {#disclaimer}

This procedure describes a configuration that carries some security risk. In fact, most local developer environments host a variety of security risks: locally stored passwords and keys, production data, proprietary source code and secrets, etc. If you're working on a proprietary project you should confirm with your IT or InfoSec department that running a local Man-in-the-Middle proxy is allowed. Read [Man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) to understand why all this mitm stuff keeps [CISOs](https://en.wikipedia.org/wiki/Chief_information_security_officer) up at night.

Finally, while the setup can be very useful and is widely shared among developers, you should absolutely not run the proxy when not in use and remove any added CA certs when there is no longer a need.

More formally:

This Man-in-the-Middle (MITM) proxy setup is provided for educational, testing, and debugging purposes only. It is not intended for any unauthorized, malicious, or unethical activities. By using this tool, you agree to take full responsibility for your actions and any consequences that may arise from using this software. The developers and distributors of this setup disclaim any liability or responsibility for any misuse, harm, or damages resulting from the use of a MITM proxy tool. Use of this tool implies that you understand and accept these terms and will not use it in any manner that violates applicable laws, regulations, or ethical standards.


## The Scenario {#the-scenario}

First, did you read the [DISCLAIMER](#disclaimer)? Read it!

You're writing a data API backend for a mobile app. You need to devtest or debug against your locally running backend, but the app doesn't have any developer hidden settings or Easter egg, e.g., to set the API domain. Production mobile apps should **absolutely not** have any test or debug "hidden menus".

The [mitmproxy](https://mitmproxy.org/) app can proxy the mobile app API calls to a local server, handling https with a fake cert and CA. Setup is straightforward, but a little hard to fish out of the docs. This document is a quick how-to and might save you some and trial/error time if you've never setup a man-in-the-middle proxy before.

Note: the [Charles Proxy](https://www.charlesproxy.com/) also supports this setup, but I find its freemium features annoying and its documentation a little scattered, at least for this use case. Furthermore, _mitmproxy_ is open source, carefully built with security in mind, and has a very powerful custom add-on capability. Otoh, I can recommend the [Charles Proxy iOS app](https://apps.apple.com/us/app/charles-proxy/id1134218562), which is quite handy for sniffing traffic on your mobile device.


## How mitmproxy Works {#how-mitmproxy-works}

The [How mitmproxy works](https://docs.mitmproxy.org/stable/concepts-howmitmproxyworks/) page is worth reading, but the essential bit is

> Mitmproxy includes a full CA implementation that generates interception certificates on the fly. To get the client to trust these certificates, we register mitmproxy as a trusted CA with the device manually.


## Proxy Setup {#proxy-setup}

I'll refer to your local computer, where you can run an API service for a mobile app, as _dev host_ below, and your mobile device as _iOS device_.


### Install mitmproxy on your dev host {#install-mitmproxy-on-your-dev-host}

Installation instructions at <https://docs.mitmproxy.org/stable/overview-installation/>


### Get your dev host's local IP address {#get-your-dev-host-s-local-ip-address}

It helps to know your subnet mask here, typically starting with `192.168`, `172.16` or `10.`. Your local system connects to the network with this address. Note that this address can change with time, but not often if the router (DHCP server specifically) does not get reconfigured or replaced. (If you have access, most DHCP services allow your to lock in an address for your MAC, beyond scope here.)


#### OSX and Linux with net-tools installed {#osx-and-linux-with-net-tools-installed}

Open _System Preferences_-&gt;/Network/ and select your network, look for the _IPv4 Address_ entry. Or, in a terminal

```shell
ifconfig | grep "inet " | grep -v 127.0.0.1
```

should show you the address you're looking for.


#### Newer versions of Linux {#newer-versions-of-linux}

```shell
ip -4 addr | grep "inet " | grep -v 127.0.0.1
```


### Install mitmproxy CA certs {#install-mitmproxy-ca-certs}


#### Run mitmproxy {#run-mitmproxy}

First we need to install and trust mitmproxy's fake root certificates on our iOS device and on our dev host. Mitm provides a URL that, when accessed through `mitmproxy`, lets you download those certs. This approach is recommended, but there are [manual installation instructions](https://docs.mitmproxy.org/stable/concepts-certificates/#installing-the-mitmproxy-ca-certificate-manually) available if you need them.

On your dev host open a terminal and run

```shell
mitmproxy --listen-host 0.0.0.0 --listen-port 8080
```

<!--list-separator-->

-  Firewall checkpoint

    If you're running a local firewall you may get an alert asking if traffic to port 8080 should be allowed. "Yes".

    If you're running `ufw` on Linux, that "alert" is probably in `/var/log/auth.log` and you'll need to run

    ```shell
    sudo ufw allow from 192.168.1.0/24 to any port 8080
    ```

    substituting you're local network mask for `192.168.1.0/24`.


#### Point your local system at the mitmproxy {#point-your-local-system-at-the-mitmproxy}

Rather than repeat something well documented on the net, a search for [browser set manual proxy server](https://duckduckgo.com/?q=browser+set+manual+proxy+server&ia=web) should turn up adequate instructions. You should end up at a screen similar to
![](/ox-hugo/network-proxy-setup-ubuntu.png)

Select _Manual_, enter your dev host's IP address from above as the HTTP Proxy, enter 8080 as the port. Don't select any form or authentication (if offered). If there's an _Ignore Hosts_ setting, that is completely optional. We'll only use this proxy setup one time on the dev host, to get certs, and then disable it.


#### Install CA cert on dev host {#install-ca-cert-on-dev-host}

Back at a browser enter the site `http://mitm.it/`, making sure to use `http` here, not `https`. You should get this page.
![](/ox-hugo/install-mitmproxys-ca.png)

Click _Show Instructions_ and then the green _Get mitmproxy-ca-cert.pem_ button for you platform. Follow those instructions.

<!--list-separator-->

-  Security Note

    The fine print at the bottom

    > Other mitmproxy users cannot intercept your connection. This page is served by your local mitmproxy instance. The certificate you are about to install has been uniquely generated on mitmproxy's first run and is not shared between mitmproxy installations.

    is an essential feature that keeps your CA cert unique and prevents potentially malicious hacking by others. Do not let this cert be compromised!


#### Disable the proxy on dev host {#disable-the-proxy-on-dev-host}

Your dev host is setup, disable the proxy from the same place you set it up.


#### Install CA cert on iOS device {#install-ca-cert-on-ios-device}

The iOS device CA cert setup is analogous. I'll refer you to [iOS set manual proxy server](https://duckduckgo.com/?q=ios+set+manual+proxy+server&ia=web) for instructions. Make sure you use the same network that your dev host is on. You should get to this screen and enter the same _Server_ and _Port_ as your dev host.
![](/ox-hugo/network-proxy-setup-ios.png)

Open Safari and again browse to `http://mitm.it/`, download the cert and follow the instructions, making sure you include the ****Important:**** step.
![](/ox-hugo/install-mitmproxys-ca-ios.png)


## Let's Go! {#let-s-go}

Now we're ready to roll!


### Checklist {#checklist}

-   [ ] Your local API service is up and running. I'll assume it's on port `3000`, adjust as needed.
-   [ ] Your iOS device is setup with manual proxy to your dev host on port `8080`. (The dev host does not need to point at the proxy.)
-   [ ] The iOS app you want to test with is installed on your device.


### Map Traffic to Your Dev Host {#map-traffic-to-your-dev-host}

Stop any running `mitmproxy` with a `q` command and start it back up with with the `--map-remote` option, replacing `example.com` with the domain name of the API your want to map to your dev host.

```shell
mitmproxy --map-remote '|https://example.com|http://localhost:3000'
```

If there are multiple APIs, use multiple `--map-remote` options. Or, you can edit the `map_remote` setting while the proxy is running with the `O` command, click or arrow down to `map_remote`, hit `ENTER` to edit, `ESC` to finish, and `q` to return to the options list. (One more `q` to return to the _Flows_ screen.)
![](/ox-hugo/edit-map-remote.png)


### Try It Out {#try-it-out}

Start your app on your iOS device. Requests to the mapped domains should be routing to your local dev host. Verify with request logging.


## Back it Out {#back-it-out}

As mentioned, when not actively needed it is recommended to disable the proxy setup. On your dev host, just don't run `mitmproxy`. For a permanent disable, remove your locally installed CA cert from the system. What you did in [Install mitmproxy on your dev host](#install-mitmproxy-on-your-dev-host), undo that.

On your iOS device un-trust the CA from _Settings_ -&gt; _General_ -&gt; _About_ -&gt; _Certificate Trust Settings_. That's good enough for temporary disablement. Delete the profile for a permanent backout (_Settings_ -&gt; _General_ -&gt; _VPN &amp; Device Management_.)


## Postscript {#postscript}

The [mitmproxy](https://mitmproxy.org/) program can do many other useful things for development and testing. See the [features](https://docs.mitmproxy.org/stable/overview-features/) section for a list of baked in capabilities. And if you need even more customization, there's an [addon](https://docs.mitmproxy.org/stable/addons-overview/) capability; you can write your own Python code for custom effects. The [examples](https://github.com/mitmproxy/mitmproxy/tree/main/examples) dir in the mitmproxy repo is full of interesting stuff. Tools for the toolbox.

{{< figure src="/ox-hugo/black_and_orange_metal_tool-scopio-c9c746e5-e5d0-4d47-aef3-939e85ba0104.jpg" >}}
