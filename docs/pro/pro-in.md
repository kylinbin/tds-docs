---
id: pro-in
title: 接入准备
sidebar_label: 接入准备
---
import {Highlight} from '../component';

## 一、开启 "TapTap 登录" 功能

1. 在开发者中心进入已经通过审核的应用，在菜单中找到「TapTap 登录」功能；
2. 选择适用地区；
3. 点击开启，获得相应的 Client ID ，即可开始接入流程。

![](/img/tap_taplogin.png)

## 二、配置平台信息
开启功能后，需要配置与应用包体一致的平台信息。一个平台可以设置多条信息以供不同场景使用。如开发者后台配置的信息与实际应用使用的信息不一致，则 TapTap 登录功能无法生效。配置步骤如下：

![](/img/tap_tapconfig.png)

## 三、集成 SDK 到你的应用

下载 TapTap SDK 集成到应用中，再按照技术对接文档调用 API，即可实现 TapTap 登录功能。

TapTap SDK 支持唤起 TapTap 原生应用授权登录。用户未安装 TapTap 客户端时，采用内嵌 Webview 授权登录的方式。  
SDK 集成步骤请参考 [快速开始](/sdk/tap-unity)。

<!-- ## 四、开始测试
如需要测试SDK功能，可以[点击下载](/res/TapSDK测试用例.xlsx)测试用例 -->