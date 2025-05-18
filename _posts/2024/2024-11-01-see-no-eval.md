---
title: "See No Eval: Runtime Dynamic Code Execution in Objective-C"
category: Blog
tags: ['iOS N-day Exploit', 'Conference']
date: 2024-11-01 16:00:00 +0900
author: ji9umi
---

## 0x01. Intro

This presentation is dedicated to NSPredicate, which was referenced in another presentation covering its role in iOS and macOS exploits. You can watch it on [YouTube](https://youtu.be/dvvFWa3Nm2M?si=rGhAKKm2GmSGSLi4) or read more in the [blog post](https://codecolor.ist/2021/01/16/see-no-eval-runtime-code-execution-objc/).

In 2019, a RealworldCTF challenge required exploting an iPhone XR on iOS 12.4.1 to be solved.

## 0x02. Analysis

The challenge was a simple calculator application written in Swift, which received input via the `icalc://` URL scheme. The input was used as arguments for NSExpression, and the calculation result was displayed on the screen.

You need to understand NSPredicate before NSExpression. Let's break it down.

### NSPredicate

```objc
NSPredicate *predicate = [NSPredicate predicateWithFormat: @"name == %@", keyword];
NSArray *results = [filtered filteredArrayUsingPredicate:predicate];
```

This sample code was shown in the presentation to illsutrate how NSPredicate works. The key part is the condition `name == %@`, where `%@` is a format specifier in Objective-C used to represent any object that inherits from NSObject, such as NSString.

If we pass 'Apple' as the keyword, the expression becomes `name == "Apple"`. The left side of the operator, *name*, is an **NSFunctionExpression**, and the right side, 'Apple', is an **NSConstantValueExpression**, and the operator is an **NSEqualityPredicateOperator**.

### NSExpression

NSExpression can be used on its own, without NSPredicate. This [part](https://www.youtube.com/watch?v=dvvFWa3Nm2M&t=376s) explain its standalone usage with sample code, and also goes into great detail about converting Objective-C code to an abstract syntax tree(AST). I highly recommend watching it if you're interested in the topic.

### Function Expressions

As is well known from the title, `eval()` enables code execution in scripting languages. Objective-C is a compiled language, so this doesn't apply in the same way - but simliar behavior is still possible with **function expressions**.

Until OS X 10.4, NSExpression only supported a limited set of predefined functions, and these functions could be accessed using custom keywords in predicate syntax.

However, starting with OS X 10.5, NSExpression also supports **arbitrary method invocation**, making arbitrary code execution possible. This is possible by using the `FUNCTION(receiver, selector, arguments, ...)` expression in NSPredicate.

String or number literals are commonly used in expressions, but they return NSString or NSNumber objects. To achieve arbitrary code execution, the Class must be placed as the receiver, and `CAST()` can be used to convert the literal to the desired class.

According to the [official documentation](https://developer.apple.com/documentation/foundation/nsexpression?language=objc#Function-Expressions), you can convert a value to an NSDate using syntax like `CAST(####, "NSDate")`. However, when using `CAST(val, "Class")`, it effectively behaves like a call to **NSClassFromString**.

## 0x03. Exploit

Not only is arbitrary code execution possible by calling Objective-C methods, but classic ROP-style techniques-such as PC control-are also achieveable using `-[NSInvocation invokeUsingIMP:]`. Since PC control ins't necessary for this challenge, the video only explains how the trigger is executed.

They read the flag file, encoded its contents in Base64, and sent it via a URL request.

## 0x04. Limitation

The device used in the CTF was an iPhone Xr with an A12 chip, but the app installed via the App Store didn't have PAC applied. The challenge didn't required a PAC bypass, which was a bit of a missed opportunity.

---

## 0xFF. References

- [YouTube - See No Eval](https://youtu.be/dvvFWa3Nm2M?si=rGhAKKm2GmSGSLi4)
- [Blog post - See No Eval](https://codecolor.ist/2021/01/16/see-no-eval-runtime-code-execution-objc/)
- [Apple Official Documentation - NSExpression](https://developer.apple.com/documentation/foundation/nsexpression?language=objc#Function-Expressions)
