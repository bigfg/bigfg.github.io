---
layout:     post
title:      fuchsia
subtitle:   代码结构
date:       2020-02-11
author:     BY
header-img: img/post-bg-fuchsia.svg
catalog: true
tags:
    - fuchsia
---

> source code [https://fuchsia.googlesource.com/fuchsia/]

### 代码结构

- fusia:
    - boards    : gn configurations to support different boards
    - build     : shared build configurations for fuchsia
    - bundles   : this directory contains top-level bundles of packages
    - docs      : top-level entry point to the Fuchsia documentation
    - examples  : examples for everything
    - garnet    : build based on zircon, providing low-level functions and some drivers
    - products  : this directory contains definitions for a variety of product configurations
    - scripts   : this repository is for scripts useful when working with the Fuchsia source code
    - sdk       : this directory contains the source code for the core of the Fuchsia SDK
    - src       : build based on zircon, providing all kinds of drivers
    - third_party : this repository contains vendored copies of third party code used in Fuchsia
    - tools     : all kinds of tools
    - zircon    : Zircon is composed of a microkernel as well as a small set of userspace services

