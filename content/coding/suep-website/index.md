+++
title = "论上海电力大学校园网"
date = 2024-09-08T20:08:24+08:00
toc = true
tags = ["校园网", "闲话"]
+++

在这篇博客中，我将介绍上海电力大学校园网一部分功能的前端逻辑以及对校园网设计模式的思考。

<!--more-->

本文仅仅是将我的[suep-toolkit](https://github.com/zhengxyz123/suep-toolkit)项目中的Python代码用文字描述并添加了些许分析。

## 统一身份认证平台
统一身份认证平台（<https://ids.shiep.edu.cn>）提供了登陆上海电力大学各校园网服务的功能。

登陆功能通过POST一个表单到`https://ids.shiep.edu.cn/authserver/login?service=<服务的URL>`实现，表单数据如下：

- `username` - 用户名
- `password` - 密码
- `captchaResponse` - 验证码（不总存在）
- `rememberMe` - 一周内免登陆，若勾选则设置为`on`，否则不包含在请求中
- `lt` - 未知（隐藏）
- `dllt` - 未知（隐藏，总是为`userNamePasswordLogin`）
- `execution` - 未知（隐藏，总是为`eNsM`，`N`、`M`为正整数，`N-1`可能代表页面刷新的次数，`M-1`可能表示登陆失败的次数）
- `_eventId` - 未知（隐藏，总是为`submit`）
- `rmShown` - 未知（隐藏，总是为`1`）

“隐藏”指`input`元素有`type="hidden"`的属性。

验证码不总出现，需要GET `https://ids.shiep.edu.cn/authserver/needCaptcha.html?username=<用户名>&_=<当前时间戳>`。若响应的内容中包含`true`字符串，则代表需要输入验证码。

若要获取验证码图片，需要GET `https://ids.shiep.edu.cn/authserver/captcha.html?ts=<当前时间戳>`。响应的内容包含一幅JPEG图像。

若用户名和密码无误，在POST后会跳转至特定网页并设置`iPlanetDirectoryPro`和`CASTGC`两个cookies；若登陆失败则状态码被设置为200。

若想退出登陆，只需GET一下`https://ids.shiep.edu.cn/authserver/logout`即可。

如果在之后想要检测用户是否登陆，可查看返回的HTML中有无`div.auth_page_wrapper`的元素。

## 学生事务及管理系统
上海电力大学学生事务及管理系统（<https://estudent.shiep.edu.cn>）为我们提供了一些看似有用实则没用的小功能。

该系统只要成功登陆了统一身份认证平台就能访问。

该系统通过从服务器获取HTML（不是JSON）再在弹窗上渲染的方式来与用户交互。例如GET `https://estudent.shiep.edu.cn/GeRCZ/JiBXX.aspx`即可获得学生的基本信息。

## 一站式办事大厅
一站式办事大厅（<https://ehall.shiep.edu.cn>）提供了一些稍微有用的功能，我将分章节叙述。

### 一卡通服务平台
一卡通服务平台（需要VPN）的登陆过程非常抽象，只登陆统一身份认证平台还不够，我将向您展示具体的HTML来说明：
```html
<form id="loginForm" action="" method="post" style="display: hidden">
    <input type="hidden" name="errorcode" value="1" />
    <input type="hidden" name="continueurl" value="http://10.168.103.76/sfrzwhlgportalHome.action" />
    <input type="hidden" name="ssoticketid" value="<学号>" />
</form>
<script>
    document.getElementById('loginForm').action = 'http://10.168.103.76/sfrzwhlgportalHome.action';
    document.getElementById('loginForm').submit();
</script>
```

即我们要POST一个表单到`http://10.168.103.76/sfrzwhlgportalHome.action`，表单数据如下：

- `errorcode` - 总是为`1`
- `continueurl` - 总是为`http://10.168.103.76/sfrzwhlgportalHome.action`
- `ssoticketid` - 学号

经过尝试，POST后的响应即是一卡通服务平台的主页。

## 教务系统
教学管理信息系统（<https://jw.shiep.edu.cn/eams/index.action>，需要VPN）在本文提到的几个功能中还是相对重要的，它能提供每周的课表、大学四年的培养计划，以及选课功能等等。

教务系统提供了两种登陆方式：“统一身份认证”和“直接登陆教务系统”。对于前者，只要成功登陆了统一身份认证平台就能通过GET `https://jw.shiep.edu.cn/eams/login.action`进入教务系统主页；若选择后者，需要输入验证码，不是特别推荐。

如果有获取当前教学周的需求，可以访问教务处主页（<https://jwc.shiep.edu.cn>）获取。

在`div#semester_start`和`div#semester_end`中分别以`YYYY-MM-DD`的形式存储着学期开始和截止的日期。

如果某日期小于学期开始日期或大于学期截止日期，且该月大于5月，则为暑假；否则为寒假。

## 能源管理
能源管理系统（<http://10.50.2.206>，需要VPN）为学生提供了宿舍电费充值及查看电表参数等功能。

每次访问能源管理系统都会要求用户重新登陆，参照登陆统一身份认证平台的步骤并POST数据到`https://ids.shiep.edu.cn/authserver/login?service=http://10.50.2.206:80/&renew=true`即可。

通过GET `http://10.50.2.206/api/charge/query?_dc=<当前时间戳>`可以获取电表参数，响应是一个JSON，其结构如下：

```javascript
{
    "success": true, // 成功与否
    "info": [
        "mid": 2, // 未知
        "type": 0, // 未知
        "recharges": 1, // 充值次数
        "reskwh": 100.0, // 剩余电量
        "P": 1, // 功率
        "U": 1, // 电压
        "I": 0, // 未知
        "FP": 1.0, // 功率因数
        "limit": 29700, // 功率限制
        "state": 0, // 电表状态
        "room": "C1-A101" // 公寓房间号
    ]
}
```

通过GET `http://10.50.2.206/api/charge/GetRoom?_dc=<当前时间戳>`可以获取用户所住的学生公寓号及房间号，响应是一个JSON，其结构如下：

```javascript
{
    "success": true, // 成功与否
    "info": [
        "building": "C1", // 公寓号
        "room": "A101", // 房间号
        "kwh": 100 // 63.6 元
    ]
}
```

通过POST到`http://10.50.2.206/api/charge/Submit?_dc=<当前时间戳>`可以充值电费，POST的表单如下：

- `building` - 公寓号
- `room` - 房间号
- `kwh` - 充值电量

其响应是一个JSON，其结构：

```javascript
{
    "success": true, // 成功与否
    "info": "交易成功" // 显示在界面上的信息
}
```

如果充值电量是一个负数或一个超大整数，`info`项会被设置为`"请输入正整数"`；若充值电量不是数字，则`info`被设置为`"错误的充值电量"`。

从校园卡中扣除的钱是实际费用四舍五入到两位小数的结果。因此，分50次买100度电比一次性买100度电会便宜0.1元。

**请大家不要在浏览器之外尝试充值电费的功能，如果充值成功的话就会扣除校园卡里面的钱！**
{style="width: 90%; margin: 0 auto; text-align: center"}

通过GET `http://10.50.2.206/api/charge/user_account?_dc=<当前时间戳>&page=<一个自然数>&start=<一个自然数>&limit=<一个自然数>`可以看见自己的充值情况(`page`、`start`等参数似乎没有意义)，响应是一个JSON，其结构如下：

```javascript
{
    "success": true, // 成功与否
    "info": [
        {
            "oid": 1234567, // 流水号
            "type": "电费", // 项目名
            "money": 0.64, // 付款金额
            "room": "C1-A101", // 房间号
            "quantity": 1, // 购电量
            "datetime": "2024-01-01 00:00:00" // 日期
        },
        {
            ...
        }
    ]
}
```

## 上电云盘
上电云盘（<https://pan.shiep.edu.cn>，需要VPN）给每个学生都提供了50GB的存储空间，也可以用来交作业，但是老师偏偏就喜欢用其它的软件来达到相同的目的。

登陆云盘需要一些额外的步骤，请仔细阅读。

首先，请GET `https://ids.shiep.edu.cn/authserver/login?service=https://pan.shiep.edu.cn/sso`，这会自动跳转到`https://pan.shiep.edu.cn/sso?ticket=<一个需要记住的字符串>`，请复制那个需要记住的字符串或将其存入变量中。

云盘的API非常复杂，这里不适合叙述。但是在[这个页面](https://pan.shiep.edu.cn/#/developers)可以下载API的文档。

## 后记
这篇博客的大部分内容是我大一刚刚开学后的军训期间（14天）完成的。最开始我只是想研究校园网是如何登陆的，后来我就想把上电学生常用的功能都研究一遍，便诞生了[suep-toolkit](https://github.com/zhengxyz123/suep-toolkit)项目以及本博客。

在研究的过程中，我需要经常查看网页的HTML以及JavaScript代码来了解前端逻辑。对于JavaScript我没有什么想说的，但是HTML里面的槽点实在是太多了，这里只举一例：校园网的一部分网页通过嵌入另一个HTML页面来显示侧边栏（一个单独的HTML文件）、顶部的横幅（一个单独的HTML文件）和主体内容（还是一个单独的HTML文件）。这是非常不推荐的，因为原本仅需1个请求就可以搞定的事情现在被分成了4个请求。而且网站还选用`frame`标签来嵌入HTML，这是已经弃用的标签，现在开发的网站应该不会使用了。

我校的校园网还是会使用AJAX功能的，不过大部分时候它返回的既不是JSON也不是XML，而是一整个HTML，还是会直接显示到屏幕上的那种。鄙人认为这校园网前后端混在一起了，是一个典型的错误示范。

总之，这一整个校园网（除了忽悠新生的“[门面网站](https://shiep.edu.cn/)”）就是一坨十年前（2010年左右）的屎山。IE的时代已经过去了，现在是所有浏览器都遵循同一个标准的时代，也是网站开发理论、技术和框架都非常成熟的时代。希望上海电力大学的领导们能够认识到时代的变化，抓紧时间启动校园网翻新的进程。

愿这篇博客中讲述的大部分内容不再适用于几年之后的上海电力大学校园网。
