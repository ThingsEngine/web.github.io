---
layout: post
title: "Sublime Text 3 安装插件究极猪皮解决方案"
categories: [安装]
tags: [Sublime Text]
published: True
---

## 本文的目标

2.1 使用 Sublime Text 3 进行插件的安装？

```md
// Given
当你安装插件遭到科学上网也解决不了的问题的时候，手动安装插件之后没有效果的时候
```


## 手动安装 Package Control 

首先去这个[链接](https://github.com/wbond/sublime_package_control)下载 Package Control 之后把下载得包 解压 重命名为 Package Control 复制到sublime存放插件的目录。
![](https://ws3.sinaimg.cn/large/005BYqpggy1g1p656cynqj30h20chabj.jpg)
现在你已经手动安装了 Package Control 你可以在 Preferences -> Package Control中看到他


### 配置用户代理

多数时我们都是不能直接去下载插件的，需要设置一下科学上网
Preferences -> Settings -> 在右侧的 User 里面设置代理
````
{
  "http_proxy": "代理地址:代理端口",
  "https_proxy": "代理地址:代理端口",
  "ignored_packages":
  [
    "Markdown",
    "Vintage"
  ]
}
````
这样你的 Sublime Text 3 的代理就设置好了



## 配置 Package Control 代理
Package Control 的代理设置方法 Preferences -> Package Settings -> Package Control -> Setting-User 然后粘贴进去
````
{
  "bootstrapped": true,
  "channels":
  [
    "https://raw.githubusercontent.com/wilon/sublime/master/download/channel_v3.json"
  ],
  "http_proxy": "代理地址:代理端口",
  "https_proxy": "代理地址:代理端口",
  "in_process_packages":
  [
  ],
  "installed_packages":
  [
    这里是你本地安装过的插件，无需复制
  ]
}

````
现在你的插件已经可以正常安装了～

## 总结一下

主要是代理的设置问题，跟着步骤去做就没问题
