# [钉钉群机器人开发接口](https://www.cnblogs.com/tjp40922/p/11299023.html)

# 获取自定义机器人webhook

步骤一，在机器人管理页面选择“自定义”机器人，输入机器人名字并选择要发送消息的群。如果需要的话，可以为机器人设置一个头像。点击“完成添加”，完成后会生成Hook地址，如下图：

![img](http://img01.taobaocdn.com/top/i1/LB1lIUlPFXXXXbGXFXXXXXXXXXX#align=left&display=inline&height=294&originHeight=1372&originWidth=2088&status=done&width=447)

步骤二，点击“复制”按钮，即可获得这个机器人对应的Webhook地址，其格式如下：

```
https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxx
```

# 使用自定义机器人

（1）获取到Webhook地址后，用户可以向这个地址发起HTTP POST 请求，即可实现给该钉钉群发送消息。注意，发起POST请求时，必须将字符集编码设置成UTF-8。

（2）当前自定义机器人支持文本 (text)、链接 (link)、markdown(markdown)、ActionCard、FeedCard消息类型，大家可以根据自己的使用场景选择合适的消息类型，达到最好的展示样式。

（3）自定义机器人发送消息时，可以通过手机号码指定“被@人列表”。在“被@人列表”里面的人员收到该消息时，会有@消息提醒(免打扰会话仍然通知提醒，首屏出现“有人@你”)。

（4）当前机器人尚不支持应答机制 (该机制指的是群里成员在聊天@机器人的时候，钉钉回调指定的服务地址，即Outgoing机器人)。

SDK ：

可以下载[SDK](https://ding-doc.dingtalk.com/doc#/faquestions/vzbp02)，简化调用方式。

消息发送频率限制：

每个机器人每分钟最多发送20条。消息发送太频繁会严重影响群成员的使用体验，大量发消息的场景 (譬如系统监控报警) 可以将这些信息进行整合，通过markdown消息以摘要的形式发送到群里。

# 测试自定义机器人

> 通过下面方法，可以快速验证自定义机器人是否可以正常工作：

使用命令行工具curl（最新版本：`7.29.0`）。

为避免出错，请将以下命令直接复制到命令行，再将xxxxxxxx替换为真实access_token；若测试出错，请检查复制的命令是否和测试命令一致，多特殊字符会报错

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
curl 'https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxx' \
   -H 'Content-Type: application/json' \
   -d '{"msgtype": "text", 
        "text": {
             "content": "我就是我, 是不一样的烟火"
        }
      }'
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　python示例：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
#author tom
import requests
import json

def dingTalk():
    headers={
        "Content-Type": "application/json"
            }
    data={"msgtype": "text",
            "text": {
                 "content": "我就是我, 是不一样的烟火"
            }
          }
    json_data=json.dumps(data)
    requests.post(url='https://oapi.dingtalk.com/robot/send?access_token=35fd4b08dea143f19921121f0a6282dcb014ebb11dae72114ed569c9effe8e5e',data=json_data,headers=headers)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

# 消息类型及数据格式

## text类型

```
{
    "msgtype": "text", 
    "text": {
        "content": "我就是我, 是不一样的烟火@156xxxx8827"
    }, 
    "at": {
        "atMobiles": [
            "156xxxx8827", 
            "189xxxx8325"
        ], 
        "isAtAll": false
    }
}
```

| 参数      | 参数类型 | 必须 | 说明                                      |
| --------- | -------- | ---- | ----------------------------------------- |
| msgtype   | String   | 是   | 消息类型，此时固定为：text                |
| content   | String   | 是   | 消息内容                                  |
| atMobiles | Array    | 否   | 被@人的手机号(在content里添加@人的手机号) |
| isAtAll   | bool     | 否   | @所有人时：true，否则为：false            |

![img](https://img.alicdn.com/tfs/TB1jFpqaRxRMKJjy0FdXXaifFXa-497-133.png#align=left&display=inline&height=112&originHeight=133&originWidth=497&status=done&width=418)

## link类型

```
{
    "msgtype": "link", 
    "link": {
        "text": "这个即将发布的新版本，创始人陈航（花名“无招”）称它为“红树林”。
而在此之前，每当面临重大升级，产品经理们都会取一个应景的代号，这一次，为什么是“红树林”？", 
        "title": "时代的火车向前开", 
        "picUrl": "", 
        "messageUrl": "https://www.dingtalk.com/s?__biz=MzA4NjMwMTA2Ng==&mid=2650316842&idx=1&sn=60da3ea2b29f1dcc43a7c8e4a7c97a16&scene=2&srcid=09189AnRJEdIiWVaKltFzNTw&from=timeline&isappinstalled=0&key=&ascene=2&uin=&devicetype=android-23&version=26031933&nettype=WIFI"
    }
}
```

| 参数       | 参数类型 | 必须 | 说明                           |
| ---------- | -------- | ---- | ------------------------------ |
| msgtype    | String   | 是   | 消息类型，此时固定为：link     |
| title      | String   | 是   | 消息标题                       |
| text       | String   | 是   | 消息内容。如果太长只会部分展示 |
| messageUrl | String   | 是   | 点击消息跳转的URL              |
| picUrl     | String   | 否   | 图片URL                        |

![img](https://img.alicdn.com/tfs/TB1VfZtaUgQMeJjy0FeXXXOEVXa-498-193.png#align=left&display=inline&height=138&originHeight=193&originWidth=498&status=done&width=355)

## markdown类型

```
{
     "msgtype": "markdown",
     "markdown": {
         "title":"杭州天气",
         "text": "#### 杭州天气 @156xxxx8827\n" +
                 "> 9度，西北风1级，空气良89，相对温度73%\n\n" +
                 "> ![screenshot](https://gw.alicdn.com/tfs/TB1ut3xxbsrBKNjSZFpXXcXhFXa-846-786.png)\n"  +
                 "> ###### 10点20分发布 [天气](http://www.thinkpage.cn/) \n"
     },
    "at": {
        "atMobiles": [
            "156xxxx8827", 
            "189xxxx8325"
        ], 
        "isAtAll": false
    }
 }
```

| 参数      | 类型   | 必选 | 说明                                   |
| --------- | ------ | ---- | -------------------------------------- |
| msgtype   | String | 是   | 此消息类型为固定markdown               |
| title     | String | 是   | 首屏会话透出的展示内容                 |
| text      | String | 是   | markdown格式的消息                     |
| atMobiles | Array  | 否   | 被@人的手机号(在text内容里要有@手机号) |
| isAtAll   | bool   | 否   | @所有人时：true，否则为：false         |

![img](https://img.alicdn.com/tfs/TB1yL3taUgQMeJjy0FeXXXOEVXa-492-380.png#align=left&display=inline&height=241&originHeight=380&originWidth=492&status=done&width=312)

说明：目前只支持md语法的子集，具体支持的元素如下：

```
标题
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题

引用
> A man who stands for nothing will fall for anything.

文字加粗、斜体
**bold**
*italic*

链接
[this is a link](http://name.com)

图片
![](http://name.com/pic.jpg)

无序列表
- item1
- item2

有序列表
1. item1
2. item2
```

## 整体跳转ActionCard类型

```
{
    "actionCard": {
        "title": "乔布斯 20 年前想打造一间苹果咖啡厅，而它正是 Apple Store 的前身", 
        "text": "![screenshot](@lADOpwk3K80C0M0FoA) 
 ### 乔布斯 20 年前想打造的苹果咖啡厅 
 Apple Store 的设计正从原来满满的科技感走向生活化，而其生活化的走向其实可以追溯到 20 年前苹果一个建立咖啡馆的计划", 
        "hideAvatar": "0", 
        "btnOrientation": "0", 
        "singleTitle" : "阅读全文",
        "singleURL" : "https://www.dingtalk.com/"
    }, 
    "msgtype": "actionCard"
}
```

| 参数           | 类型   | 必选  | 说明                                            |
| -------------- | ------ | ----- | ----------------------------------------------- |
| msgtype        | string | true  | 此消息类型为固定actionCard                      |
| title          | string | true  | 首屏会话透出的展示内容                          |
| text           | string | true  | markdown格式的消息                              |
| singleTitle    | string | true  | 单个按钮的方案。(设置此项和singleURL后btns无效) |
| singleURL      | string | true  | 点击singleTitle按钮触发的URL                    |
| btnOrientation | string | false | 0-按钮竖直排列，1-按钮横向排列                  |
| hideAvatar     | string | false | 0-正常发消息者头像，1-隐藏发消息者头像          |

通过整体跳转ActionCard类型消息发出的消息样式如下：

![img](https://img.alicdn.com/tfs/TB1ehWCiBfH8KJjy1XbXXbLdXXa-334-218.png#align=left&display=inline&height=159&originHeight=218&originWidth=334&status=done&width=244)![img](https://img.alicdn.com/tfs/TB1nhWCiBfH8KJjy1XbXXbLdXXa-547-379.png#align=left&display=inline&height=236&originHeight=379&originWidth=547&status=done&width=340)

## 独立跳转ActionCard类型

```
{
    "actionCard": {
        "title": "乔布斯 20 年前想打造一间苹果咖啡厅，而它正是 Apple Store 的前身", 
        "text": "![screenshot](@lADOpwk3K80C0M0FoA) 
 ### 乔布斯 20 年前想打造的苹果咖啡厅 
 Apple Store 的设计正从原来满满的科技感走向生活化，而其生活化的走向其实可以追溯到 20 年前苹果一个建立咖啡馆的计划", 
        "hideAvatar": "0", 
        "btnOrientation": "0", 
        "btns": [
            {
                "title": "内容不错", 
                "actionURL": "https://www.dingtalk.com/"
            }, 
            {
                "title": "不感兴趣", 
                "actionURL": "https://www.dingtalk.com/"
            }
        ]
    }, 
    "msgtype": "actionCard"
}
```

| 参数           | 类型   | 必选  | 说明                                                    |
| -------------- | ------ | ----- | ------------------------------------------------------- |
| msgtype        | string | true  | 此消息类型为固定actionCard                              |
| title          | string | true  | 首屏会话透出的展示内容                                  |
| text           | string | true  | markdown格式的消息                                      |
| btns           | array  | true  | 按钮的信息：title-按钮方案，actionURL-点击按钮触发的URL |
| btnOrientation | string | false | 0-按钮竖直排列，1-按钮横向排列                          |
| hideAvatar     | string | false | 0-正常发消息者头像，1-隐藏发消息者头像                  |

通过独立跳转ActionCard类型消息发出的消息样式如下：

![img](http://img01.taobaocdn.com/top/i1/LB1GgOFQVXXXXXnaXXXXXXXXXXX#align=left&display=inline&height=205&originHeight=556&originWidth=1998&status=done&width=737)

## FeedCard类型

```
{
    "feedCard": {
        "links": [
            {
                "title": "时代的火车向前开", 
                "messageURL": "https://www.dingtalk.com/s?__biz=MzA4NjMwMTA2Ng==&mid=2650316842&idx=1&sn=60da3ea2b29f1dcc43a7c8e4a7c97a16&scene=2&srcid=09189AnRJEdIiWVaKltFzNTw&from=timeline&isappinstalled=0&key=&ascene=2&uin=&devicetype=android-23&version=26031933&nettype=WIFI", 
                "picURL": "https://www.dingtalk.com/"
            },
            {
                "title": "时代的火车向前开2", 
                "messageURL": "https://www.dingtalk.com/s?__biz=MzA4NjMwMTA2Ng==&mid=2650316842&idx=1&sn=60da3ea2b29f1dcc43a7c8e4a7c97a16&scene=2&srcid=09189AnRJEdIiWVaKltFzNTw&from=timeline&isappinstalled=0&key=&ascene=2&uin=&devicetype=android-23&version=26031933&nettype=WIFI", 
                "picURL": "https://www.dingtalk.com/"
            }
        ]
    }, 
    "msgtype": "feedCard"
}
```

| 参数       | 类型   | 必选 | 说明                     |
| ---------- | ------ | ---- | ------------------------ |
| msgtype    | string | true | 此消息类型为固定feedCard |
| title      | string | true | 单条信息文本             |
| messageURL | string | true | 点击单条信息到跳转链接   |
| picURL     | string | true | 单条信息后面图片的URL    |

通过FeedCard类型消息发出的消息样式如下：
