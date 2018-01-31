---
layout:     post
title:      "OSI七层模型"
category : basictheory
date:       2018-01-26
author:     "Max"
header-img: "img/post-2018.jpg"
catalog:    true
tags:
    - 
---

# 1. 概述

开放系统互连模型（Open Systems Interconnection model，OSI 模型）是国际标准化组织制定的一种抽象概念，用以在多种通信系统及标准协议间界定与标准化其通信功能与交互操作。原始版本的 OSI 模型分为七层，每一层服务于上层，并由下层提供服务。

![OSI](/img/in-post/osi/tcpip.gif)

# 2. 详述

## 2.1 物理层 Physical Layer

物理层是 OSI 分层结构体系中最重要、最基础的一层，它建立在传输媒介基础上，起建立、维护和取消物理连接作用，实现设备之间的物理接口。物理层之接收和发送一串比特(bit)流
，不考虑信息的意义和信息结构。