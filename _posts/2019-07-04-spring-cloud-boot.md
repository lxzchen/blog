---
layout: post
title: Spring Cloud和Spring Boot版本对应关系
date: 2019-07-04
Author: 态度大师
categories: 
tags: [spring cloud, spring boot]
comments: true
---

当我们在使用Spring Cloud和Spring Boot的时候，往往对它们的版本对应关系比较模糊。特别在做版本升级的时候，升了Spring Boot，但不知道对应的Spring Cloud是否还需要升级，或者直接凑合着用。但这样往往就是一些未知的错误包含在了项目里面，不知道合适爆发。这篇文章，就试着简单捋捋，它们之间的关系，帮助大家在升级的时候做到心里有谱，下手不慌。

根据官网介绍，Spring Cloud是一个整体性项目，它由许多个独立的、规范的和不同版本号的小项目组成。为了方便管理整套内容的发布，避免跟其他的其他项目版本出现让使用者困惑的局面，发布的版本号采用名称而非传统版本号的命名方式。发布的版本名称采用伦敦铁路站的名称，并按照字母顺序发布使用。类似**名称.SRX**。SR表示，如service release的简称，X使用数字，代表小的版本号。

以下简单的Spring Cloud和Spring Boot版本对应关系

Spring Cloud | Spring Boot
- | -
Greenwich | 2.1.x
Finchley | 2.0.x
Edgware | 1.5.x
Dalston | 1.5.x
