---
title: Mobile testing over VPN
categories: ['Bug Bounty', Tips]
tags: ['environment']
date: 2023-10-07 12:16:00 +0900
author: jun
img_path: /assets/img/mobile-testing-over-vpn/
---

## Agenda

Some bug bounty platform is required connecting vpn to testing server in this case we need more setting to intercept their packet. In this article we talk about how to network with work VPN.

## Prerequisites

Before you starting to follow this setup you may need:

1. Two NIC
    - At least one **wireless NIC**
2. Macbook
3. VPN client
4. Proxy tools

The **macbook** is not important condition but i tested on macbook because of you should find more resource about this setup. Also i used **Open VPN** as vpn client and **Burpsuite** as proxy tools. 

## Step-by-step

```shell
----------         -------------           ------------
| Mobile | ------> | Proxy(PC) | ========> | Internet |
----------         -------------    VPN    ------------
```
{: file='topology'}

After we complete setup, network connection looks like above topology. A vpn client is only install on your PC we don't need any special setting in your phone.

### network setting on pc

I used one ethernet cable and one macbook default wireless nic. The ethernet is used to connect **public internet** with vpn and wireless is used to **share network** for mobile device.

To create internet share, follow `Preferences -> General -> Sharing`. You can see **Internet Sharing** on 'Accessories & Internet' section. 

![Sharing](sharing.png)
_Preferences -> General -> Sharing_

Click the right of toggle switch button that `i` mark to change internet sharing options. Below images is my setting for share vpn connection.

![Sharing_Options](sharing_options.png)
_Internet Sharing options_

> If you want to change wireless options like ssid or password, you **must** change before ***turn on*** internet sharing.
{: .prompt-tip}

Click the Wi-Fi Options and set your own ssid and other options.

![Wi-Fi_Options](wifi_options.png)
_Wi-Fi-Options_

Finally, Turn on internet sharing and enable Wi-Fi adapter.

![Turn_on_sharing](turn_on_sharing.png)
_Turn-on-sharing_

### test internet sharing

After finished changing options and turn on internet sharing, you can find new Wi-Fi network on your mobile device. You can access public internet by this network but it doesn't through vpn even if pc is connected by vpn to server. I check this result by visiting '**check my public ip**' website.

### make redirection by firewall

In debian based linux dist i frequently used `iptables` to control firewall rule. Few days after search i found [this](https://davidhamann.de/2017/04/19/sharing-vpn-on-macos/) article about **prctl**. Prctl is similar iptables also way to write rules.

```bash
#!/bin/sh

sysctl -w net.inet.ip.forwarding=1
pfctl -f ./nat-rules -e
```
{: file='natvpn.sh'}

```bash
nat on utun6 from bridge100:network to any -> (utun6)
```
{: file='nat-rules'}

I write two script file first is prctl rules file that called `nat-rules`{: .filepath} and other one is enable forwarding and apply to prctl rules that called `natvpn.sh`{: .filepath}. A `utn6` in nat-rules file you must to replace that name of **YOUR VPN INTERFACE** and `bridge100` is **INTERNET SHARING** interface in result of `ifconfig`.

> If you want flush prctl rules, run `prctl -F all` command on your terminal.
{: .prompt-info}

### test firewall rule

After aplying prctl rules, check once again in mobile device. If public ip is same with your pc and mobile device that means firewall rule is successfully work.

---

## References

- [Write prctl rule to share vpn connection](https://davidhamann.de/2017/04/19/sharing-vpn-on-macos/)