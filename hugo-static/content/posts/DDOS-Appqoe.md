---
title: "Block DDOS Attacks with Netscaler AppQoE"
date: 2021-05-06T10:58:12+02:00
draft: true
---

To enable HCIResponse you first need to execute on the shell of netscaler
```
nsapimgr -ys appqos_captcha=1
```

 To make sure this is persistent of reboots of netscaler you need to add it to the rc.netscaler file to make sure it is executed everytime netscaler starts.
