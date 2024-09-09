+++
title = "论上海电力大学校园网"
date = 2024-09-08T20:08:24+08:00
tags = ["网络", "闲话"]
+++

在这篇博客中，我将介绍上海电力大学校园网一部分功能的前端逻辑及对校园网设计模式的思考。

<!--more-->

本文仅仅是将我的[suep-toolkit](https://github.com/zhengxyz123/suep-toolkit)项目中的Python代码用文字描述。

## 统一身份认证平台
统一身份认证平台（<https://ids.shiep.edu.cn>）提供了登陆上海电力大学各校园网服务的功能。

登陆功能通过POST一个id为`casLoginForm`的表单到`https://ids.shiep.edu.cn/authserver/login`实现，表单有如下几个输入元素：

- `username` - 用户名
- `password` - 密码
- `captchaResponse` - 验证码（不总存在）
- `rememberMe` - 一周内免登陆，若勾选则设置为`on`，否则不包含在请求中
- `lt` - 未知（隐藏）
- `dllt` - 未知（隐藏，总是为`userNamePasswordLogin`）
- `execution` - 未知（隐藏，总是为`eNsM`，`N`、`M`为正整数，`N`可能代表页面刷新的次数，`M`可能表示登陆失败的次数）
- `_eventId` - 未知（隐藏，总是为`submit`）
- `rmShown` - 未知（隐藏，总是为`1`）

“隐藏”指`input`元素有`type="hidden"`的属性。

验证码不总出现，需要GET地址为`https://ids.shiep.edu.cn/authserver/needCaptcha.html?username=<用户名>&_=<当前时间戳>`的网页。若响应的内容中包含`true`字符串，则代表需要输入验证码。

若要获取验证码图片，需要GET地址为`https://ids.shiep.edu.cn/authserver/captcha.html?ts=<当前时间戳>`的网页。响应的内容包含一幅JPEG图像。

> 时间戳可能不是必须的，只是为了防止浏览器缓存罢了。

若用户名和密码无误，在POST后会跳转至特定网页并设置`iPlanetDirectoryPro`和`CASTGC`两个cookies；若登陆失败则状态码被设置为200。

## 学生事务及管理系统
上海电力大学学生事务及管理系统（<https://estudent.shiep.edu.cn>）为我们提供了一些看似有用实则没用的小功能。

该系统只要成功登陆了统一身份认证平台就能访问。

该系统以从服务器获取HTML（不是JSON）再在弹窗上渲染来与用户交互。例如GET地址为`https://estudent.shiep.edu.cn/GeRCZ/JiBXX.aspx`的网页即可获得学生的基本信息。

看到这种操作，我只能感叹：这可是十年前的科技啊！
