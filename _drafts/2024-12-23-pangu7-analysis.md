---
title: Pangu7 Analysis
category: Blog
tags: ['iOS Exploit', 'Case Study']
date: 2024-12-23 14:35:00 +0900
author: insp3ct0r_x
---

## TL; DR;

blabla

## Structure

몇 번 훑어보니 이전 evasi0n7과 비슷하게 `__objc_cons1` 부터 `__objc_cons11` 영역에 exploit payload 를 숨겨두고 이를 추출해서 실행하는 것으로 보임.

gunzip 호출하는 로직도 여러 곳 존재하였음.

- `1000069e9    void -[pgMainViewController initJB](struct pgMainViewController* self, SEL sel)`

아예 여기서 대놓고 저런 식으로 호출하네

```objc
  31 @ 100006ae3  rax_5 = sub_100008ea1(rdi_9, "__objc_cons1")
  32 @ 100006ae8  rdi_10 = rax_5
  33 @ 100006aeb  rax_6 = _objc_retainAutoreleasedReturnValue(obj: rdi_10)
  34 @ 100006af0  var_38 = rax_6
  35 @ 100006b02  rdi_11 = rax_6
  36 @ 100006b05  rax_7 = -[4326891992 (gun) gunzip](self: rdi_11, sel: "gunzip")
  37 @ 100006b0a  rdi_12 = rax_7
  38 @ 100006b0d  obj = _objc_retainAutoreleasedReturnValue(obj: rdi_12)
  39 @ 100006b12  rbx_2 = obj
  40 @ 100006b1c  rdi_13 = r13
  41 @ 100006b1f  rax_8 = sub_100008ea1(rdi_13, "__objc_cons2")
  42 @ 100006b24  rdi_14 = rax_8
  43 @ 100006b27  obj_1 = _objc_retainAutoreleasedReturnValue(obj: rdi_14)
  44 @ 100006b2c  var_40 = obj_1
  45 @ 100006b37  rdi_15 = obj_1
  46 @ 100006b3a  rax_9 = -[4326891992 (gun) gunzip](self: rdi_15, sel: "gunzip")
  47 @ 100006b3d  rdi_16 = rax_9
  48 @ 100006b40  obj_2 = _objc_retainAutoreleasedReturnValue(obj: rdi_16)
  49 @ 100006b45  var_70 = obj_2
  50 @ 100006b50  rdi_17 = r13
  51 @ 100006b53  rax_10 = sub_100008ea1(rdi_17, "__objc_cons3")
```

호출되는 지점이 여러 곳이라 다 xref 찍어보니 살짝 위치가 다름.

```
-[pgMainViewController loadView]
=> __objc_cons9

-[pgMainViewController startJailbreak]
=> __objc_cons4

-[pgMainViewController initJB]
=> __objc_cons1
=> __objc_cons2
=> __objc_cons3
=> __objc_cons5
=> __objc_cons6
=> __objc_cons7

-[pgLocalization init]
=> __objc_cons8
```

10번 11번의 경우 `sub_100005438` 에서 조건에 따라 둘 중 하나가 선택되는 방식이었는데 상위 함수를 보면 `-[pgMainViewController showAdjustProfileDate]` 였고 NSConcreteStackBlock 으로 호출되는 걸 보니 아마 주기적으로 돌면서 뭘 체크하는거든 그런 거인 듯..?

아마 중국어냐 아니냐에 따라 보여지는 화면이 다른 것 같음.

---

## something note 

Pesudo ObjC 처럼 뭔가 깔끔하게 번역해주는 거 힘드려나.. 한 눈에 들어오진 않는 듯 아직?

암만봐도 msgSend 불러야 할 위치에서 그냥 `call(r14)` 하는 건 왜 그럴까..?
