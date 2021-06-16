---
title: spring boot 中实现策略模式
date: 2021-06-16 10:58:12
tags: spring-boot design pattern strategy spring
---

策略模式是一种行为设计模式， 它能让你定义一系列算法， 并将每种算法分别放入独立的类中， 以使算法的对象能够相互替换。

主要的作用就是避免编写大量的 If-Else 这种 bad smell 代码，提升代码可读性，可维护性。
