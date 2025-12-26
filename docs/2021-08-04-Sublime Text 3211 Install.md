---
title: 'Sublime Text 3 Install'
layout: post
tags:
  - software
category: 软件
---
Sublime Text 3 version 3211版本安装以及破解过程。

<!--more-->

## 一、在hosts中添加（成功后可还原）

hosts地址: `C:\Windows\System32\drivers\etc`

```txt
#sublimetext　
127.0.0.1 www.sublimetext.com
127.0.0.1 sublimetext.com
127.0.0.1 sublimehq.com
127.0.0.1 telemetry.sublimehq.com
127.0.0.1 license.sublimehq.com
127.0.0.1 45.55.255.55
127.0.0.1 45.55.41.223
```

## 二、修改编辑 sunlime_text.exe

1. 关闭 Sublime text3，并打开安装目录，备份一下 sublime_text.exe（以防万一）
2. 打开网址：https://hexed.it
3. 点击上面的 Open File，弹出的对话框中选择已安装后的 sublime_text.exe。
4. Ctrl+F 搜索查找 ，输入97 94 0D，然后点击按钮 Search Now。
5. 然后在97 94 0D上面点击，替换为00 00 00即可！
6. 最后点击上面的 Export 按钮导出，将完成后的文件复制到刚才的目录替换掉原来的文件即可

![20210804114314](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/20210804114314.png)



## 三、输入注册码注册

打开 Sublime text，然后点击菜单 Help -> Enter Lisence :

```
----- BEGIN LICENSE -----
TwitterInc
200 User License
EA7E-890007
1D77F72E 390CDD93 4DCBA022 FAF60790
61AA12C0 A37081C5 D0316412 4584D136
94D7F7D4 95BC8C1C 527DA828 560BB037
D1EDDD8C AE7B379F 50C9D69D B35179EF
2FE898C4 8E4277A8 555CE714 E1FB0E43
D5D52613 C3D12E98 BC49967F 7652EED2
9D2D2E61 67610860 6D338B72 5CF95C69
E36B85CC 84991F19 7575D828 470A92AB
------ END LICENSE ------
```