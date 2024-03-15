---
title: __ctype_b_loc() in C language
categories: ["Computer Science", Programming]
tags: ["language specific"]
date: 2023-12-29 19:16 +0900
author: jun
---

## Introduce

In today, i met the intresting function in C language so write this article. As you can see in title, the name of function is `__ctype_b_loc()` and follow code is example of this:

```c
0000133e          if ((zx.d((*__ctype_b_loc())[sx.q(rax_26)]) & 0x200) != 0)
0000135f              *(sx.q(var_24) + arg2) = *(&var_88 + sx.q(sx.d(rax_26) - 0x61))
00001386          else if ((zx.d((*__ctype_b_loc())[sx.q(rax_26)]) & 0x100) != 0)
000013aa              *(sx.q(var_24) + arg2) = *(&var_a8 + sx.q(sx.d(rax_26) - 0x41))
000013d1          else if ((zx.d((*__ctype_b_loc())[sx.q(rax_26)]) & 0x800) == 0)
0000144d              *(sx.q(var_24) + arg2) = rax_26
```

## ctype.h

The `__ctype_b_loc()` is an internal function used by `isUpper()`, `isLower()` and other function that related with character and it's defined in `ctype.h`{: .filepath}.

```c
#ifndef _ISbit
/* These are all the characteristics of characters.
   If there get to be more than 16 distinct characteristics,
   many things must be changed that use `unsigned short int's.

   The characteristics are stored always in network byte order (big
   endian).  We define the bit value interpretations here dependent on the
   machine's byte order.  */

# include <bits/endian.h>
# if __BYTE_ORDER == __BIG_ENDIAN
#  define _ISbit(bit)   (1 << (bit))
# else /* __BYTE_ORDER == __LITTLE_ENDIAN */
#  define _ISbit(bit)   ((bit) < 8 ? ((1 << (bit)) << 8) : ((1 << (bit)) >> 8))
# endif

enum
{
  _ISupper = _ISbit (0),        /* UPPERCASE.  */
  _ISlower = _ISbit (1),        /* lowercase.  */
  _ISalpha = _ISbit (2),        /* Alphabetic.  */
  _ISdigit = _ISbit (3),        /* Numeric.  */
  _ISxdigit = _ISbit (4),       /* Hexadecimal numeric.  */
  _ISspace = _ISbit (5),        /* Whitespace.  */
  _ISprint = _ISbit (6),        /* Printing.  */
  _ISgraph = _ISbit (7),        /* Graphical.  */
  _ISblank = _ISbit (8),        /* Blank (usually SPC and TAB).  */
  _IScntrl = _ISbit (9),        /* Control character.  */
  _ISpunct = _ISbit (10),       /* Punctuation.  */
  _ISalnum = _ISbit (11)        /* Alphanumeric.  */
};
#endif /* ! _ISbit  */
```
{: file='/usr/include/ctype.h'}

### enum

Each character has the attributes. For example 'A', is upper case and alphabetic also included in alphanumeric. This is the reason why `isUpper('A')` result is 'True'. By the way, decompile result that mentioned in introduce section is compare like '0x200' with result of `__ctype_b_loc()` function. To understand difference we need to know how `ISbit()` macro is defined.

### table

Let's see the detail.

```c
# include <bits/endian.h>
# if __BYTE_ORDER == __BIG_ENDIAN
#  define _ISbit(bit)   (1 << (bit))
# else /* __BYTE_ORDER == __LITTLE_ENDIAN */
#  define _ISbit(bit)   ((bit) < 8 ? ((1 << (bit)) << 8) : ((1 << (bit)) >> 8))
# endif
```

When system is big endian, `_ISbit()` is just shift once but system is little endian shift once again that's why sample binary is compare with '0x200'.

If passed character is 'lower case' then internal function trying to set `ISbit(1)` on the table. Regarding the source code, big endian is return `1 << 1` and it's **0x2** as hexdecimal. However little endian is return `1 << 1 << 8` and it's **0x200** as hexdecimal.

> We also can be applied to detect the system endian!
{: .prompt-tip}

## References

- [Stackoverflow](https://stackoverflow.com/questions/37702434/ctype-b-loc-what-is-its-purpose)
- [Braincoke's blog](https://braincoke.fr/blog/2018/05/what-is-ctype-b-loc/#reading-an-entry-of-__ctype_b_loc)