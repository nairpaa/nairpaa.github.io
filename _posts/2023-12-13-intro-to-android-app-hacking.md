---
title: '01 - Introduction to Android App Hacking'
date: 2023-12-13 16:42:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'introduction']     # TAG names should always be lowercase
author: nairpaa
---

Dalam kesempatan ini, saya mencoba untuk mengumpulkan catatan-catatan mengenai *penetration test* pada aplikasi Android.

Catatan ini mencakup berbagai teknik yang digunakan untuk melakukan eksploitasi pada aplikasi Android, termasuk rekomendasi teknis yang dapat diimplementasikan. Selain itu, saya juga membahas solusi untuk berbagai tantangan CTF yang terkait.

## 0x1 - Table of Content

### A. Basic Programming

- [Basic Kotlin](https://www.youtube.com/playlist?list=PL-CtdCApEFH_hja5vRJgQOXylCiQud7Qa)
- [Basic Gradle](https://www.youtube.com/watch?v=bu26Us8m8Cw&list=PL-CtdCApEFH8yGJzfU_gners0ybO4MlrV)
- [Basic Android](https://www.youtube.com/watch?v=pUTz5IOkBtE&list=PL-CtdCApEFH9kC-xh_g7WeCB33lHXxWFc&index=1)
- [Basic Flutter](https://www.youtube.com/watch?v=SoX3cel4LRM&list=PLZQbl9Jhl-VACm40h5t6QMDB92WlopQmV)

### B. MASVS Resilience

#### 1. Reverse Engineering 

- REApp 0x1: RE with jadx
- REApp 0x2: RE with DEX Bytecode
- REApp 0x3: RE with Native Library
- REApp 0x4:- Obfuscation
- Obfuscating Android Code

#### 2. Root Detection

- [Rooting Device Realme 3 Pro](https://nairpaa.github.io/posts/rooting-realme-3-pro/)
- Detecting Root and Bypassing Anti-Root Android Flutter App
- Detecting Root and Bypassing Anti-Root Android Kotlin App

#### 3. Integrity Check

- [Decompile & Patching APK](/posts/patching-apk/)
- Implementing Integrity Checks in Android Apps

#### 4. Frida

- [Frida-Labs 0x1: Frida setup, Hooking a method](https://nairpaa.github.io/posts/frida-labs-challenge-0x1/)
- [Frida-Labs 0x2: Calling a static method](https://nairpaa.github.io/posts/frida-labs-challenge-0x2/)
- [Frida-Labs 0x3: Changing the value of a variable](https://nairpaa.github.io/posts/frida-labs-challenge-0x3/)
- [Frida-Labs 0x4: Creating a class instance](https://nairpaa.github.io/posts/frida-labs-challenge-0x4/)
- [Frida-Labs 0x5: Invoking methods on an existing instance](https://nairpaa.github.io/posts/frida-labs-challenge-0x5/)
- [Frida-Labs 0x6: Invoking a method with an object argument](https://nairpaa.github.io/posts/frida-labs-challenge-0x6/)
- [Frida-Labs 0x7: Hooking the constructor](https://nairpaa.github.io/posts/frida-labs-challenge-0x7/)
- [Frida-Labs 0x8: Introduction to native hooking](https://nairpaa.github.io/posts/frida-labs-challenge-0x8/)
- [Frida-Labs 0x9: Changing the return value of a native function](https://nairpaa.github.io/posts/frida-labs-challenge-0x9/)
- [Frida-Labs 0xA: Calling a native function](https://nairpaa.github.io/posts/frida-labs-challenge-0xa/)
- [Frida-Labs 0xB: Patching instructions using X86Writer and ARM64Writer](https://nairpaa.github.io/posts/frida-labs-challenge-0xb/)
- Detecting Frida and Bypassing Anti-Frida Android Flutter App
- Detecting Frida and Bypassing Anti-Frida Android Kotlin App

### C. Capture the Flag

**1337UP 2023 - Intigriti**:
- Memdump
- Fatcher
