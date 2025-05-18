---
title: Root is no more used in Palera1n
category: Blog
tags: ['Jailbreak', 'Tips']
date: 2024-08-23 14:40:00 +0900
author: ji9umi
---

## Intro

In this year, i moved the defend forward team from the pentesting team and spent most of time to analyze the IoT system or code audit. These task also funny and intreseting but it’s far from a jailbreak what i loved so i didn't know trend like what’s the best tools or update list of the major tools.

A few days ago, i had to jailbreake the phone for extract the target application and the palera1n was a lot of things is changed. That's why i wirte this post. Let's dig into the palera1n!

## Remote connection with SSH

The one of the powerful feature of jailbreak is make possible to ssh connection your phone. For this, 'root' user is used with 'alpine' which is their default password. I don’t remember what version i used however it was very close to first release version and their internal was so complicate to light user. From my memory, palera1n also used root account to connect ssh at this version.

But now, everything is changed. While you jailbreak with palera1n, it requires new password and it will be used by the 'mobile' account password. From my memory, palera1n also used root account to connect ssh at this version.

```
# In the past:

ssh root@<iphone-ip-address>
// Enter default password if you not changed

# Currently:

ssh mobile@<iphone-ip-address>
// Enter your own password
```

Dopamine seems to use same mechanism to login to ssh.

---

## Reference

- [How to login ssh](https://medium.com/@hunterid/not-alpine-ssh-password-for-jailbroken-iphone-meowbrek2-3bc3bd66428a)
