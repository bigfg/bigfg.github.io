---
layout:     post
title:      fuchsia
subtitle:   zircon代码结构
date:       2020-02-12
author:     BY
header-img: img/post-bg-fuchsia.svg
catalog: true
tags:
    - fuchsia
---

> source code [https://fuchsia.googlesource.com/fuchsia/]

### zircon代码结构

- zircon:
    - bootloader    : a UEFI boot shim for Zircon that can load images vis chaining iPXE, filesystem, local disk partitions
    - experimental  : nothing
    - kernel        : core impementation of Zircon
    - prebuilt      : nothing
    - public        : GN integration for Zircon
    - script        : scripts
    - system        :  
    - third_party   : third_party
    - tools         : some useful tools
    - vdso          : virtual dynamic shared object

### bootloader

### experimental

### kernel

### prebuilt
