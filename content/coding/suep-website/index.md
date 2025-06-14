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

登陆功能通过POST一个表单到`https://ids.shiep.edu.cn/authserver/login`实现，表单数据如下：

- `username` - 用户名
- `password` - 密码
- `captchaResponse` - 验证码（不总存在）
- `rememberMe` - 一周内免登陆，若勾选则设置为`on`，否则不包含在请求中
- `lt` - 未知（隐藏）
- `dllt` - 未知（隐藏，总是为`userNamePasswordLogin`）
- `execution` - 未知（隐藏，总是为`eMsN`，`M`、`N`为正整数）
- `_eventId` - 未知（隐藏，总是为`submit`）
- `rmShown` - 未知（隐藏，总是为`1`）

“隐藏”指`input`元素有`type="hidden"`的属性。它们的值必须通过GET `https://ids.shiep.edu.cn/authserver/login`获取。

验证码不总出现，需要GET `https://ids.shiep.edu.cn/authserver/needCaptcha.html?username=<用户名>&_=<当前时间戳>`。若响应的内容中包含`true`字符串，则代表需要输入验证码。

若要获取验证码图片，需要GET `https://ids.shiep.edu.cn/authserver/captcha.html?ts=<当前时间戳>`。响应的内容包含一幅JPEG图像。

若用户名和密码无误，在POST后会跳转至特定网页并设置`iPlanetDirectoryPro`和`CASTGC`两个cookies；若登陆失败则状态码被设置为200。

若想退出登陆，只需GET `https://ids.shiep.edu.cn/authserver/logout`即可。

如果在之后想要检测用户是否登陆，可查看返回的HTML中有无`div.auth_page_wrapper`的元素。

## 学生事务及管理系统
上海电力大学学生事务及管理系统（<https://estudent.shiep.edu.cn>）为我们提供了一些看似有用实则没用的小功能。

该系统只要成功登陆了统一身份认证平台就能访问。

该系统通过从服务器获取HTML（不是JSON）再在弹窗上渲染的方式来与用户交互。例如GET `https://estudent.shiep.edu.cn/GeRCZ/JiBXX.aspx`即可获得学生的基本信息。

## 上电DeepSeek
2025年4月3日，AI智能助手（<https://ai.shiep.edu.cn>，需要VPN）正式上线运行。在上线的第一时间我便进行了尝试，评价结果为**一坨狗屎**。

接下来我将详细讲述如何登陆及使用这堆**狗屎**。

该系统在成功登陆了统一身份认证平台之外，还要再GET `http://10.166.40.12?url=https://ai.shiep.edu.cn`，这会跳转至`https://ai.shiep.edu.cn?authToken=<token>`。请将`<token>`保存至变量，之后的所有请求都要用到。

接下来我只会讲述如何与大语言模型交流，因为这是这堆**狗屎**中唯一有用的内容。

为了与大语言模型交流，需要GET `https://ai.shiep.edu.cn/api/data/ai/ve-api`并附带用如下键值对组成的查询字符串：

- `prompt`：提示词
- `model`：`deepseek-v3`或`deepseek-r1`之一
- `chatSessionId`：总是为`fetching...`
- `authToken`：上文提到过的

接下来，我将以服务器发送事件（SSE）的格式讲述应该如何解读响应（因为响应头`Content-Type`被设置成了`text/event-stream`）：

- `chatSessionId`事件：一个UUID
- `rmessage`事件：一个JSON，其`data`键对应的值是思维链的一部分
- `cmessage`事件：一个JSON，其`data`键对应的值是非思维链的一部分
- `statistics`事件：统计信息组成的JSON
- `close`事件：关闭连接

由此可见：我们每次只能和DeepSeek对话一次，而且因为提示词是通过URL向服务器发送的，能传递的内容非常有限。信息办，你创造的东西不是**一坨狗屎**是什么？

## 一站式办事大厅
一站式办事大厅（<https://ehall.shiep.edu.cn>）提供了一些稍微有用的功能，我将分章节叙述。

### 一卡通服务平台
一卡通服务平台（<https://ecard.shiep.edu.cn>，需要VPN）的登陆过程非常抽象，只登陆统一身份认证平台还不够，我将向您展示具体的HTML来说明：
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

通过GET `http://10.168.103.76/accountcardUser.action`，我们可以获得卡余额、卡状态等有用的信息（使用正则表达式提取）。

要想获取用户的账户列表，可GET `http://10.168.103.76/accounttodayTrjn.action`或GET `http://10.168.103.76/accounthisTrjn.action`并遍历`select#account>option`元素。元素的`value`属性即是用户的账号（一般情况下只会有一个）。

POST到`http://10.168.103.76/accounttodatTrjnObject.action`可查询当日流水。POST的表单如下：

- `account` - 账号
- `inputObject` - 交易类型（一般选择`all`，即“查询全部”）

其响应是一个HTML，我们可以在元素`tr.bl>td>div[align=center]`处找到当日流水有多少页（使用正则表达式提取），并再POST到该URL即可使用循环读取当日流水的每一页。POST的表单如下：

- `pageVo.pageNum` - 当前页（从1开始）
- `inputObject` - 之前选择的交易类型
- `account` - 之前选择的账号

若想获得历史流水，可先POST以下表单到`http://10.168.103.76/accounthisTrjn1.action`：

- `account` - 账号
- `inputObject` - 交易类型（一般选择`all`，即“查询全部”）

再POST想要查询的时间段到`http://10.168.103.76/accounthisTrjn2.action`：

- `inputStartDate` - 以`YYYYMMDD`编码的开始日期
- `inputEndDate` - 以`YYYYMMDD`编码的结束日期

接下来浏览器先等待一秒再执行后续操作，不过不等待一秒也可以。

接下来POST一个空的表单到`http://10.168.103.76/accounthisTrjn3.action`即可查询当日流水。

其响应是一个HTML，我们可以在元素`tr.bl>td>div[align=center]`处找到历史流水有多少页（使用正则表达式提取）。然后POST到`http://10.168.103.76/accountconsubBrows.action`即可使用循环读取历史流水的每一页。POST的表单如下：

- `inputStartDate` - 之前选择的开始日期（`YYYYMMDD`格式）
- `inputStartDate` - 之前选择的结束日期（`YYYYMMDD`格式）
- `pageNum` - 当前页（从1开始）

由于一卡通服务的限制，最多只能查询间隔为30天的历史流水（即开始日期与结束日期之间的间隔小于30天，而不是开始日期与当前日期的间隔小于30天）。

## 教务系统
教学管理信息系统（<https://jw.shiep.edu.cn/eams/index.action>，需要VPN）在本文提到的几个功能中还是相对重要的，它能提供每周的课表、大学四年的培养计划，以及选课功能等等。

教务系统提供了两种登陆方式：“统一身份认证”和“直接登陆教务系统”。对于前者，只要成功登陆了统一身份认证平台就能通过GET `https://jw.shiep.edu.cn/eams/login.action`进入教务系统主页；若选择后者，需要输入验证码，不是特别推荐。

想要获取课表，可以参考[《NEU新版教务处课程表》](https://gist.github.com/whoisnian/32b832bd55978fefa042d7c76f9d76c3)中的步骤。

接下来我将要介绍选课的详细逻辑。

我们首先需要GET `https://jw.shiep.edu.cn/eams/stdElectCourse.action`，再通过正则表达式`electionProfile.id=(\d+)`获取其中的数字字符串（可能不止有一个）。

然后，通过GET `https://jw.shiep.edu.cn/eams/stdElectCourse!data.action?profileId=<刚刚获得的数字字符串>`来获取所有可选课程。其返回的是一个JavaScript脚本，部分内容如下：
```javascript
var lessonJSONs = [
    ...
    {
        "id": 718711,
        "no": "2700154.01",
        "name": "创新中国",
        "code": "2700154",
        "credits": 1.0,
        "courseId": 33592,
        "stdCount": 10,
        "limitCount": 10.0,
        "startWeek": 5,
        "endWeek": 15,
        "courseTypeId": 1216,
        "courseTypeName": "人文社科类",
        "courseTypeCode": "217",
        "lessonTypeId": 1,
        "lessonTypeName": "正常",
        "lessonTypeCode": "02",
        "scheduled": true,
        "hasTextBook": false,
        "period": 16,
        "weekHour": 1,
        "withdrawable": true,
        "textbooks": "null",
        "teachers": "线上教师（超星）",
        "campusCode": "",
        "campusName": "",
        "remark": "超星线上课程",
        "arrangeInfo": [
            {
                "weekDay": 7,
                "weekState": "00000111111111110000000000000000000000000000000000000",
                "startUnit": 13,
                "endUnit": 13,
                "weekStateDigest": "5-15",
                "rooms": ""
            }
        ]
    }
    ...
];
```

我们只需要将该字符串转换为合法的JSON字符串即可被几乎所有的编程语言读取。

下面以Python为例，简单介绍转换方法：
```python
import re

# original_str 为请求的内容，需要做切片处理
# legal_json_str 为合法的 JSON 字符串
legal_json_str = re.sub(r"(,|{)(\w+):", r'\1"\2":', original_str).replace("'", '"')
```

想要选课，需要POST到`https://jw.shiep.edu.cn/eams/stdElectCourse!batchOperator.action`，并附带如下数据：
```javascript
{
    "profileId": "<第一步就获取到的数字字符串>",
    "optype": true,
    "operator0": "<课程 id>:true:0"
}
```

想要退课，需要POST到与选课操作相同的URL，并附带如下数据：
```javascript
{
    "profileId": "<第一步就获取到的数字字符串>",
    "optype": false,
    "operator0": "<课程 id>:false"
}
```

响应都是一个HTML，如果HTML中含有“成功”二字即视为选（退）课成功。

对于Linux系统来说，访问教务系统时SSL会报`unable to get local issuer certificate`错误。你可能需要做一些额外的工作来访问教务系统（例如使用`requests`库发送请求时添加`verify=False`参数）。

要获取当前教学周，需要GET `https://jwc.shiep.edu.cn`。

在`div#semester_start`和`div#semester_end`中分别以`YYYY-MM-DD`的形式存储着学期开始和截止的日期。

如果某日期小于学期开始日期或大于学期截止日期，且该月大于5月，则该日期属于暑假；否则属于寒假。

## 能源管理
能源管理系统（<http://10.50.2.206>，需要VPN）为学生提供了宿舍电费充值及查看电表参数等功能。

每次访问能源管理系统都会要求用户重新登陆，参照登陆统一身份认证平台的步骤并POST数据到`https://ids.shiep.edu.cn/authserver/login?service=http://10.50.2.206:80/&renew=true`即可。

通过GET `http://10.50.2.206/api/charge/query?_dc=<当前时间戳>`可以获取电表参数。响应是一个JSON，其结构如下：

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

通过GET `http://10.50.2.206/api/charge/GetRoom?_dc=<当前时间戳>`可以获取用户所住的学生公寓号及房间号。响应是一个JSON，其结构如下：

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

通过POST到`http://10.50.2.206/api/charge/Submit?_dc=<当前时间戳>`可以充值电费。POST的表单如下：

- `building` - 公寓号
- `room` - 房间号
- `kwh` - 充值电量

其响应是一个JSON，结构如下：

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

通过GET `http://10.50.2.206/api/charge/user_account?_dc=<当前时间戳>&page=<一个自然数>&start=<一个自然数>&limit=<一个自然数>`可以看见自己的充值情况(`page`、`start`等参数似乎没有意义)。响应是一个JSON，其结构如下：

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

若要登陆云盘，请GET `https://ids.shiep.edu.cn/authserver/login?service=https://pan.shiep.edu.cn/sso`，这会自动跳转到`https://pan.shiep.edu.cn/sso?ticket=<ticket>`，请将`<ticket>`存入变量`ticket`。

然后，POST到`https://pan.shiep.edu.cn/api/v1/auth1?method=getbythirdparty`并附带如下数据：
```javascript
{
    "deviceinfo": {
        "ostype": 6
    },
    "params" : {
        "ticket": ticket // 刚刚提到的变量
    },
    "thirdpartyid": "dlxy-as"
}
```

其响应是一个JSON，结构如下（这里的UUID都是随机生成的）：
```javascript
{
    "expires": 3600,
    "tokenid": "1927593b-75a8-4975-a3d5-ad203b2db48b",
    "userid": "824f2df1-e685-475b-95ee-dc07c727e69d"
}
```

在之后的所有API调用中，都需要附加`&tokenid=<tokenid>`到URL之后。

本云盘基于AnyShare，您可自行搜索其API。

## 后记
这篇博客的一部分内容是我大一刚刚开学后的军训期间（14天）完成的。最开始我只是想研究校园网是如何登陆的，后来我就想把上电学生常用的功能都研究一遍，便诞生了[suep-toolkit](https://github.com/zhengxyz123/suep-toolkit)项目以及本博客。

相信大家可以看到：校园网的一部分功能是为浏览器专门设计的，并没有相应的API可以调用；有些页面使用了已经弃用的技术（比如说用`frame`标签来嵌入一个HTML页面）。这与现在常用的前、后端分离的开发模式相悖，也不符合HTML标准。

同时，我在大一上学期遇到了两次服务器无法访问的问题：第一次是一卡通停止服务超过一周，第二次是选课期间的教务系统服务器故障。学校各部门（尤其是信息办）都没有及时就两次服务器故障做出响应，教务处也仅仅在事后轻描淡写了一句“选课因服务器故障暂停”。

综上，我们发现校园网不仅技术较落后，同时大部分系统也缺乏有效的管理。

我在此恳求上海电力大学能够拨出一笔少得可怜的钱来改进一下校园网的业务逻辑，同时让相关人员做好服务器的维护工作，并且做好服务器故障的应急处理工作。

以上，是我个人小小的努力，希望大家喜欢。
