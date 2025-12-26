---
title: 'Emscripten安装'
layout: post
tags:
  - wasm
category: 框架
---
Emscripten安装.

<!--more-->

# 一、Emscripten安装

## 1、安装python、git

## 2、安装

```shell
# 1.克隆emsdk
git clone https://github.com/juj/emsdk.git

# 2.进入emsdk文件夹
cd emsdk

# 3.更新emsdk 这里使用是git所以运行时会提示使用"git pull"
emsdk update
git pull

# 4.安装最新的emsdk 并配置全局的环境变量
emsdk install --global latest

# 5.激活
emsdk activate latest

# 6.应用环境变量
emsdk_env.bat
```

## 3、验证安装

```shell
emcc -v
emcc --clear-cache

```

## 4、使用

```shell
#编写一份cpp文件/下载一份
git clone git@github.com:codenoid/md5-cpp.git

#编译test.c文件
emcc -O3 -s WASM=1 -s EXTRA_EXPORTED_RUNTIME_METHODS='["cwrap"]’ test.c
```

- -O3: 优化级别，O3 是最高优化级别。
- -s WASM=1：生成 wasm 代码，而不是 asm.js 代码。
- -s EXTRA_EXPORTED_RUNTIME_METHODS=‘[“cwrap”]‘：在 JavaScript 中使用 cwrap 函数引用导出函数。
