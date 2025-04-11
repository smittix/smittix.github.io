---
title: "Gigastone Cve"
date: 2025-04-11T11:49:34+01:00
draft: true
toc: false
images:
tags:
  - untagged
---

[toc]
# Introduction

So, I got sucked into the vortex of social media the other night and noticed these quirky little red thingies a few people were using to boost their WiFi range. After some detective work (thanks, Amazon!), I stumbled upon these tiny travel routers from Gigastone, and guess what? They were a steal, just £18.99 for a pair!

{{< image src="/images/gigastone1.png" alt="" position="center" style="display: block; margin: 0 auto; max-width: 100%; height: auto;" >}}

Naturally, I couldn't resist snagging a couple. I mean, they practically screamed "hack me" for a tear-down post.

## Let’s open it up!

![image.png](:/63e73860bfe0413ebdcc90e4fcfd1f54)

Upon opening the device which was really simple with a pry tool and took near to no effort I was happy to see a Serial Peripheral Interface (SPI) NOR Flash chip (Red Box) and what looked to be Universal Asynchronous Receiver / Transmitter **(**UART) connectivity (Yellow Box).

## UART

I managed to get a console via the UART connection using some dupont wires and a TTL USB to UART cable. However, no joy there as the shell that was provided was very limited and attempts to dump the firmware image was futile.

![image.png](:/347b4f499eb14526b4da42904adc2742)

## SPI

Let’s move on to the SPI chip and see if we can dump the firmware using my trusty BusPirate 

![](:/d2a169ee87b6462dbe381cfe4755b011)

After a little Google-fu I found the datasheet for this particular chip which had the pin-outs which enabled me to hook it up to the BusPirate which dumped the firmware successfully now I could have a rummage through the file-system.

![image.png](:/5411ca2370354ea7b6ccf9ee1db56381)

I spent a few evenings going through things which turned out quite fruitful. The first one being that the router was using 202.96.209.5 for DNS which is owned by China Telecom and that just may be a concern to those of us that are privacy minded.

![china_dns.png](:/9da5cde831d94c2687e00b75b8f973b6)

Secondly, there were a handful of scripts/binaries lurking around some in plain-text some needing a little persuasion by Ghidra.

My initial infrastructure assessment of the device revealed nothing of significance except a web-server and DNS which is standard for a router. 

However, after running one of the scripts, the device unexpectedly rebooted, leaving me puzzled at first. It wasn’t until I investigated further that I discovered Telnetd had been enabled, confirmed by a quick port scan. How did that happen?

Well, the following wget command was launched:

`wget --load-cookies cookies.txt "<http://192.168.16.254/cgi-bin/tr1.cgi?cgi=2&mode=0&ssid=YHRlbG5ldGQgLWwvYmluL3NoYA==&ht=0&channel=0&dhcp=50&static=0&ip=0.0.0.0&subnet=0.0.0.0&gw=0.0.0.0&pppmode=0&pppusr=bnVsbA==&ppppwd=bnVsbA==&pwd=>"`

The '`wget`' command appears to be being used to send a request to the device's web server to execute a specific CGI script (tr1.cgi) with various parameters. This command seems to be exploiting a feature or a bug in the CGI script to turn on telnet. Let's break down the command to understand how it's achieving this:

# Breaking Down the Command

### wget --load-cookies cookies.txt:

This part of the command uses wget to send an HTTP request. The --load-cookies cookies.txt option tells wget to use the cookies stored in cookies.txt for the request. This might be necessary for authentication if the web server requires login credentials.

## URL Breakdown:

### Base URL: [http://192.168.16.254/cgi-bin/tr1.cgi](http://192.168.16.254/cgi-bin/tr1.cgi)

This is the URL of the CGI script being accessed on the device's web server.

### Query Parameters:

- `cgi=2`: This parameter likely specifies the type of action to be taken by the CGI script.
- `mode=0`: This parameter specifies a mode, possibly indicating a specific operation within the CGI script.
- `ssid=YHRlbG5ldGQgLWwvYmluL3NoYA==`:This parameter is Base64 encoded which we will go into shortly.

Other parameters (`ht=0`, `channel=0`, `dhcp=50`, etc.) likely configure other settings, but they don't seem directly relevant to this vulnerability.

### Decoding the `ssid` Parameter

The most crucial part of the command is the `ssid` parameter. Let's decode it:

Input:
`echo "YHRlbG5ldGQgLWwvYmluL3NoYA==" | base64 --decode`

Output:
`telnetd -l /bin/sh`

This decoded string is a command to start the telnet daemon (`telnetd`) with the login shell set to `/bin/sh`. By including this command in the `ssid` parameter, the CGI script is tricked into executing it, thereby enabling telnet access.

### How It Works

1. **CGI Script Vulnerability**: The CGI script `tr1.cgi` seems to accept the `ssid` parameter and might execute it or use it in a way that leads to command execution.
2. **Base64 Encoding**: By encoding the command in Base64, it bypasses potential input validation checks.
3. **Command Execution**: When the web server processes the CGI script with the provided parameters, it decodes the `ssid` parameter and executes the embedded command, enabling telnet with `/bin/sh`.

### Cookie ingredients:

### cookies.txt

Contents:
`192.168.16.254 TRUE / FALSE 1590440637 TR1_ADMIN admin`

### Breakdown of Cookie Fields

1. **`192.168.16.254`**:
**Domain**: This indicates the domain for which the cookie is valid. In this case, it's `192.168.16.254`, which is likely the IP address of the device's web server.
2. **`TRUE`**:
**Include subdomains**: This field specifies whether the cookie should be sent to subdomains of the domain. `TRUE` means it should be sent to subdomains.
3. **`/`**:
**Path**: This indicates the URL path that must exist in the requested URL for the browser to send the Cookie header. `/` means the cookie is valid for the entire domain.
4. **`FALSE`**:
**Secure**: This field specifies whether the cookie should only be sent over secure connections (HTTPS). `FALSE` means the cookie can be sent over both HTTP and HTTPS.
5. **`1590440637`**:
**Expiration time**: This field represents the expiration time of the cookie in Unix time (seconds since January 1, 1970). `1590440637` corresponds to a specific date and time when the cookie will expire.
6. **`TR1_ADMIN`**:
**Name**: This is the name of the cookie. In this case, it’s `TR1_ADMIN`.
7. **`admin`**:
**Value**: This is the value of the cookie. In this case, it’s `admin`.

### Conclusion

The `wget` command is exploiting a vulnerability/feature in the CGI script of the device's web server to execute a command that starts the telnet daemon. This provides shell access to the device over the network.

## Seeing it in Action

Ok, so I have the travel router on and ready to go, lets give it a quick port scan to see what ports are open before we execute the command.

![nmap_before_vuln.png](:/4f7900bfcc494eed817b3adc694eeb4e)

As you can see no Telnet service is listening (Port 23)

So let’s run the command detailed earlier in the article and show what happens

![launching_vuln.png](:/2108095560b14335b08a183e3a9c5d8d)

The router has just rebooted itself, lets re-run that port scan

![nmap_after_vuln.png](:/288f57996ba14da19e8d75db3a422cd0)

Et Voila, the Telnet service is now magically listening on its default port of ‘23’

Let’s connect to the Telnet service

![busybox_login_telnet_initial.png](:/ea793a78121c43b8b0a91d5a7d13b163)

We’re now connected without any user credentials having to be supplied. Let’s take a look around

![initial_telnet_passwd.png](:/d1eeb624390f498aacdd0179642ee6a6)

I’ll send that off to the cracking rig along with the shadow file (which I won’t show) especially with how poor the password actually was. Users Admin and root had the same hash.

![root_login.png](:/f42d7bf573024023b77e0421ac9c0df5)

We’ve successfully logged in as root!

I have sat here rummaging around the file-system and I’ve come up with many ways this vulnerability could be maliciously used which is why I have omitted certain pieces of detail within this article as I cannot foresee a firmware update being released any time soon.

# Responsible Disclosure

Gigastone were contacted multiple times via email to various departments none of which seemed interested in having a conversation around my research. After failing to respond to my last email I gave it a few weeks until I responded with a timeline for disclosure which I also didn’t have a response to. Since then Gigastone have discontinued this product from their website however you can still purchase these devices from well known online vendors.