---
title: ARM Secure World
categories: ["Computer Science", Hardware]
tags: ["arm basic"]
date: 2023-10-18 21:26 +0900
author: jun
img_path: /assets/img/arm-secure-world/
---

## TL; DR;

At first, To understand more detail of mitigation we have to know security features that **Trust Zone** and **Exception Levels**. Both features is implemented by ARM so these is processor's features.

## Exception Levels

A kind of iOS mitigation article is mention `EL<n>` likes "EL0, EL1...". I search what meaning of "EL" and finally i knew it's aarch64 features that called **"Exception Levels"**[^1]. It restrict their privileges to prevent abuse look likes **[protection ring](https://en.wikipedia.org/wiki/Protection_ring)**[^2].

There are implemented 4 exception levels on aarch64 likes:
![Figure 1](./exception-level-layer.png)*source: https://blog.csdn.net/luolaihua2018/article/details/131501298*

> Note: EL3 is can using only AArch64.
{: .prompt-tip}

As you can see above figure privilege levels be higher from EL0 to EL3.

- EL0

=> EL0 is the lowest execution level and also called the **unprivileged level** of execution. In this level normal applications run in userspace.

- EL1

=> Operating System kernel usually run at this level. 

- EL2

=> EL2 is used by a hypervison.

- EL3

=> EL3 is used by firmware and highest execution level.

## Trust Zone

A trusted execution environment shortly called as **[TEE](https://en.wikipedia.org/wiki/Trusted_execution_environment)**[^3] is secure area of main processor and ARM also support this features as called **[Trust Zone](https://en.wikipedia.org/wiki/ARM_architecture_family#Security_extensions)**[^4]. In a ARMv8-A series it has two security status "secure world" and "normal world" that seperated by **trust zone**. 

![figure 2](./exception-level-with-trust-zone.png)
*source: https://blog.csdn.net/luolaihua2018/article/details/131501298*

The main operating system actually runs in the normal world and EL3 is gateway to communicate with secure world.

---

## References

- https://blog.matteyeux.com/posts/a-propos-de-kpp/

[^1]: https://developer.arm.com/documentation/102412/0103/Privilege-and-Exception-levels/Exception-levels
[^2]: https://en.wikipedia.org/wiki/Protection_ring
[^3]: https://en.wikipedia.org/wiki/Trusted_execution_environment
[^4]: https://en.wikipedia.org/wiki/ARM_architecture_family#Security_extensions