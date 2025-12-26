---
title: 'app spider'
layout: post
tags:
  - spider
category: spider
---
桌面应用内容获取分析

<!--more-->
# 一、桌面应用DOM
## inspect.exe / FlaUInspect
客户端获取桌面应用的所有属性信息
![微信截图_20231215165659](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信截图_20231215165659.png)

## WinAppDriver
* 服务端获取桌面应用的所有属性信息
```
https://github.com/Microsoft/WinAppDriver/releases
```
* 接入方式
```
https://github.com/microsoft/WinAppDriver/tree/master/Samples
```

# 二、Appium、Selenium、Playwright自动化测试工具
* appium支持最多平台驱动（win、安装、浏览器等）
* selenium在浏览器平台中更加高效
* playwright支持浏览器平台、且支持更多语言

# 三、Demo（公众号）
## 1、公众号存在唯一ID，内部用__biz标识，
例如：__biz=Mzg5MjYyNDY5MQ==

![微信截图_20231113173454](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信截图_20231113173454.png)

在线获取历史文章工具地址：```https://wx.okhelp.xyz/```

![微信截图_20231113173345](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信截图_20231113173345.png)

## 2、公众号存在一个url地址可以查询到历史文件列表，在浏览器中不可打开，仅支持在客户端PC打开，或者手机端打开（至20231215有效）
链接：
```
https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=Mzg5MjYyNDY5MQ==
```
![微信图片_20231215171811](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信图片_20231215171811.png)
## 3、客户端请求使用https，代理需要证书才可以获取到https的请求信息

## 4、客户端打开后会生成，更多安全认证信息拼接到地址再打开，即认证码在url上传递，若能人工生成认证码，或在认证码有效期中获取该链接也可在浏览器中打开
链接：
```
https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=MzI2NDI5NzkwNA==&uin=OTA4NzMzNTIx&key=c62732d4126d447abb23f34e03130e0cc8576cd51e2b228410ecb6fa7c9ddf2725b25a65e428232900deacadd664c7b1ced5bbb41781bf32620cdebbb06977c0ec50ae676dbbbc5c705c7fe0a89d6bdd9bdf1f91cfd9b046a7975fd747037a324fcca2cf57476daf4c0cea7e24f571ead18c1189d4cd7333f6a6d9c1d0f8f5fb&devicetype=Windows+10+x64&version=6309080f&lang=zh_CN&a8scene=0&acctmode=0&pass_ticket=nfGYUqkK2k74zzEYLaKCbEFTtaLut8APn6DCUbO6HEpUmCcLK8T0xlfLP%2FBgDHCpU5jUVleIyLAbsfFEh2LAlg%3D%3D&wx_header=1
```
![微信截图_20231113174040](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信截图_20231113174040.png)

## 5、若获取部分认证也可在浏览器打开，但是无法进行翻页
例如在cookie中加上wap_sid2
![微信图片_20231113173848](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信图片_20231113173848.png)
![微信截图_20231113174543](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信截图_20231113174543.png)

## 6、客户端打开链接方式是内部浏览器，使用inspect.exe等可以直观看出其地址转化后的结果
![微信图片编辑_20231215172238](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信图片编辑_20231215172238.jpg)

## 7、下拉刷新文章列表方式为调用api，即传递认证即可拿到下一页的文章列表，一直下拉即可拿到所有文章
其api为：
```
https://mp.weixin.qq.com/mp/profile_ext?action=getmsg&__biz=MzI2NDI5NzkwNA==&f=json&offset=10&count=10&is_ok=1&scene=&uin=OTA4NzMzNTIx&key=bdb15eb8b0eb27ca63cacffa8cd9033d8745e43106ea5fc3eb70b5c6a67a5ba99ebbe5ea705d55e468134cce5d4f8c37455e0bf55183be8b64f257d10a99f6539219d494391fb4b62fec384b4857dd75aae4646119fbd2f162fe7d72e743825d35f3418011983d500bdbf798686a576399fae3dd841611693357d5b4deb6df03&pass_ticket=Le5qqlqxWLXK5fmOrzvO%26amp%3Bnbsp%3BnIhwA6fzw1WQR%2FFxIt4Xgr0qZy5A2Ulktlvbfed1RnRrb6RJtgPXBV%26amp%3Bnbsp%3BLtiaPGpL0g%3D%3D&wxtoken=&appmsg_token=1243_aauo9%252F46eSnMi90toDXPXNsIMTbv7F1A8t8DFw~~&x5=0&f=json
```
## 8、api分析
```
https://mp.weixin.qq.com/mp/profile_ext
?action=getmsg
&__biz=MzI2NDI5NzkwNA==
&f=json
&offset=10
&count=10
&is_ok=1
&scene=
&uin=OTA4NzMzNTIx
&key=bdb15eb8b0eb27ca63cacffa8cd9033d8745e43106ea5fc3eb70b5c6a67a5ba99ebbe5ea705d55e468134cce5d4f8c37455e0bf55183be8b64f257d10a99f6539219d494391fb4b62fec384b4857dd75aae4646119fbd2f162fe7d72e743825d35f3418011983d500bdbf798686a576399fae3dd841611693357d5b4deb6df03
&pass_ticket=Le5qqlqxWLXK5fmOrzvO%26amp%3Bnbsp%3BnIhwA6fzw1WQR%2FFxIt4Xgr0qZy5A2Ulktlvbfed1RnRrb6RJtgPXBV%26amp%3Bnbsp%3BLtiaPGpL0g%3D%3D
&wxtoken=
&appmsg_token=1243_aauo9%252F46eSnMi90toDXPXNsIMTbv7F1A8t8DFw~~
&x5=0
&f=json
```
1. __biz为公众号id，已知
2. uin为用户id
3. key、pass_ticket、appmsg_token都是在文章认证的列表接口中返回（必须是携带了完整认证码的接口、即能下拉翻页）
![微信图片_20231215174047](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信图片_20231215174047.png)
4. offset为当前页，count为每页数（固定为10不可更改），其中offset并非为0、10、20..的固定序列，而是根据该页获取的文章后，会在结果中返回，即：offset=0&count=10获取时，返回10条最后发布的信息，信息中总共包含12篇文章，因此在结果中还会返回nextOffset=13，因此下一页为offset=13&count=10才可，并且获取默认按发布消息时间降序
![微信截图_20231215180509](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信截图_20231215180509.png)

## 【总结】获取到公众号唯一标识后，既能拼接出固定的文章列表地址，使用apium操作客户端打开地址，并用appWinDriver获取到dom中的已认证地址，并在有效期内提取出认证信息，即可调用api正常访问文章列表，最后只需使用文章地址即可访问文章内容
![微信截图_20231215182050](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/微信截图_20231215182050.png)
