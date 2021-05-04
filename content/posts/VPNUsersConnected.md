---
title: "Find User that has logged in to a specific Citrix Gateway"
date: 2021-03-24T15:15:58+01:00
draft: false
---
Today I wanted to turn off a specific Citrix Gateway on Netscaler cause I thought it would not be needed anymore. On the first view there were still a lot of users connected to the vpn vserver so simply turning it off would through them all out and was therefore not an option.

On the first sight the GUI you can see how many users are connected.

![Citrix Gateway vserver view](/devmaryb/media/VPNLog-1.png)

But when looking on ICA Connections/DTLS ICA Connections/Active Sessions you see the Backend Server the Client is connected to or  the Intranet IP and Application.
The only way to get the usernames of the users from  Netscaler is from the log files. They ones we are looking at are at /var/log/

There is the actual one: ns.log and the older ones are already zipped.

![Shell Directory View](/devmaryb/media/VPNLog-2.png)

So I search for the IP address of the Citrix Gateway and for the string "LOGIN" in these files. The IP address is the one configured directly on the Netscaler not the public IP if there is a NAT configuration in place.

In case of the .gz files the command will be the following:

```
gzcat ns.log*gz | grep "ip of vpn vserver" | grep LOGIN
```

If you simply want to get a list of users you can do this by using a regex in another stacked grep.

```
gzcat ns.log*gz | grep "ip of vpn vserver" | grep LOGIN | grep -o "User[ ][a-z]*"
```

Assuming that your usernames contain only characters, the regex can be adapted depending on the username format.
