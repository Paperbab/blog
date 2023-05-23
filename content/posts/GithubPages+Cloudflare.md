---
date: "2022-12-18T19:15:22+08:00"
draft: false
title: Github Pages+Cloudflare部署指南
---
# 前言
前段时间自己用Github Pages搭了个Hexo博客，别问我为啥不用gitee，因为要实名，一直对要实名的东西不大信任，而且我有许多项目都放在github。。。我们家是电信网，天生对Github不友好，就连上Github都需要想尽办法，我的博客自然也很难连上，终于我忍不住啦！！！用cloudflare成功加速了我的博客

# 准备
准备一个域名，我用的是在freenom上注册的免费域名，每年需要续一次费，有点心烦
![zbrrFI.png](https://s1.ax1x.com/2022/12/18/zbrrFI.png)

然后设置一下Pages
![zbrGJx.png](https://s1.ax1x.com/2022/12/18/zbrGJx.png)

这一步你可能会看到这个错误
![zbrUyD.png](https://s1.ax1x.com/2022/12/18/zbrUyD.png)

先不管他，我们等会再解决

# 设置域名解析
打开cloudflare，添加站点，输入你自己的域名，而不是你的github pages域名
![zbrfmQ.png](https://s1.ax1x.com/2022/12/18/zbrfmQ.png)

接下来你可能需要添加NS服务器，我们按照cloudflare的提示找到NS
![zbr0wd.png](https://s1.ax1x.com/2022/12/18/zbr0wd.png)

复制到域名提供商提供的修改NameServer入口（需要注意，部分域名提供商不会允许你自定义NS）
![zbr8F1.png](https://s1.ax1x.com/2022/12/18/zbr8F1.png)

然后你需要ping一下你的Github Pages，得到Github Pages的IP
![zbr4Ts.png](https://s1.ax1x.com/2022/12/18/zbr4Ts.png)

接下来在Cloudflare的“DNS”栏里添加DNS记录，推荐使用A记录，像我这样填就可以
![zbrgl8.png](https://s1.ax1x.com/2022/12/18/zbrgl8.png)

然后转到Github重新验证一下名称服务器（NameServer）就行了！

# 需要注意的
在github内点开Enforce HTTPS
然后再cloudflare中将加密模式改成完全或完全（严格），不然会导致重定向过多而打不开网页！
![zbrj0J.png](https://s1.ax1x.com/2022/12/18/zbrj0J.png)