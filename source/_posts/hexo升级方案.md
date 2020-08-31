---
title: hexo升级方案
date: 2018-11-23 11:11:45
tags: [hexo]
---

最好在hexo文件夹路径下执行
# 升级
1、全局升级`hexo-cli`，先`hexo version`查看当前版本，然后`npm i hexo-cli -g`，再次`hexo version`查看是否升级成功。

2、使用`npm install -g npm-check`和`npm-check`，检查系统中的插件是否有升级的，可以看到自己前面都安装了那些插件

3、使用`npm install -g npm-upgrade`和`npm-upgrade`，升级系统中的插件

4、使用`npm update -g`和`npm update --save`
<!--more-->
PS：第四步遇到了错误，错误提示如下：
```
> fsevents@1.2.11 install /Users/lanvnal/Files/blog/node_modules/hexo/node_modules/fsevents
> node-gyp rebuild

xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance

xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance

No receipt for 'com.apple.pkg.CLTools_Executables' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLILeo' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLI' found at '/'.
gyp: No Xcode or CLT version detected!
```
其实已经安装过了xcode cli，但是这里还是报错了，估计和苹果新系统有关，重装就好了，操作如下：

如果像以前一样执行`xcode-select --install`会有如下报错：
```
xcode-select: error: command line tools are already installed, use "Software Update" to install updates
```
解决办法：
```
sudo rm -rf /Library/Developer/CommandLineTools
xcode-select --install
```

然后再执行第四步，完美升级。