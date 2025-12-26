---
title: 'Aspose.PSD for Java'
layout: post
tags:
  - psd
category: 框架
---

专业文档处理组件aspose：Aspose.PSD for Java操作PSD工具包

<!--more-->

# 一、Maven依赖
官方地址：https://repository.aspose.com/repo/

官方demo：https://github.com/aspose-psd/Aspose.PSD-for-Java

```

<dependency>
    <groupId>com.aspose</groupId>
    <artifactId>aspose-psd</artifactId>
    <version>20.5</version>
    <classifier>javadoc</classifier>
</dependency>

<dependency>
    <groupId>com.aspose</groupId>
    <artifactId>aspose-psd</artifactId>
    <version>20.5</version>
    <classifier>jdk16</classifier>
</dependency>


```

# 二、修改jar

反编译工具：JbyteModify

class操作组件：javassist
https://www.jianshu.com/p/7bed2ad09230


```
ClassPool.getDefault().insertClassPath("xxx.jar");
CtClass c = ClassPool.getDefault().getCtClass("xxx.class");
CtMethod[] as = c.getDeclaredMethods("method");
as[2].insertAfter("{return 0;}");
c.writeFile("D:\\");

```

# 三、鉴权

xml文件
```
<License>
    <Data>
        <Products>
            <Product>Aspose.Total for Java</Product>
            <Product>Aspose.Words for Java</Product>
        </Products>
        <EditionType>Enterprise</EditionType>
        <SubscriptionExpiry>20991231</SubscriptionExpiry>
        <LicenseExpiry>20991231</LicenseExpiry>
        <SerialNumber>8bfe198c-7f0c-4ef8-8ff0-acc3237bf0d7</SerialNumber>
    </Data>
    <Signature>
        sNLLKGMUdF0r8O1kKilWAGdgfs2BvJb/2Xp8p5iuDVfZXmhppo+d0Ran1P9TKdjV4ABwAgKXxJ3jcQTqE/2IRfqwnPf8itN8aFZlV3TJPYeD3yWE7IT55Gz6EijUpC7aKeoohTb4w2fpox58wWoF3SNp6sK6jDfiAUGEHYJ9pjU=
    </Signature>
</License>

```

java代码
```
#全路径
new License().setLicense(new FileInputStream("...license.xml"));
```

# 四、使用