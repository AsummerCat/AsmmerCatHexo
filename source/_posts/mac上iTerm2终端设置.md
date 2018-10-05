---
title: mac上iTerm2终端设置
date: 2018-10-04 15:30:15
tags: mac
---

# 1.参考

>1.[Mac OS 终端利器 iTerm2](https://www.cnblogs.com/xishuai/p/mac-iterm2.html)  
>2.[mac攻略(八) -- 神器zsh和iterm2的配置](https://www.cnblogs.com/redirect/p/6429731.html)


---

<!--more-->

# 2.安装 iTerm2

下载地址：https://www.iterm2.com/downloads.html

下载的是压缩文件，解压后是执行程序文件，你可以直接双击，或者直接将它拖到 Applications 目录下。

或者你可以直接使用 Homebrew 进行安装：

`$ brew cask install iterm2`

---

# 3. 配置 iTerm2 主题

iTerm2 最常用的主题是 Solarized Dark theme，下载地址：http://ethanschoonover.com/solarized

下载的是压缩文件，你先解压一下，然后打开` iTerm2，按Command + ,`键，打开 `Preferences` 配置界面，然后`Profiles -> Colors -> Color Presets -> Import`，选择刚才解压的`solarized->iterm2-colors-solarized->Solarized Dark.itermcolors`文件，导入成功，最后选择 Solarized Dark 主题，就可以了。  

![iTerm2配置](/img/2018-10-4/iTerm2Config.png)

---

# 4.配置 Oh My Zsh

Oh My Zsh 是对主题的进一步扩展，地址：https://github.com/robbyrussell/oh-my-zsh

一键安装：

`$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
安装好之后，需要把 Zsh 设置为当前用户的默认 Shell（这样新建标签的时候才会使用 Zsh）：

`$ chsh -s /bin/zsh`
然后，我们编辑`vim ~/.zshrc`文件，将主题配置修改为`ZSH_THEME="agnoster"`。

![](/img/2018-10-4/oh-my-zshConfig1.png)

`agnoster`是比较常用的 zsh 主题之一，你可以挑选你喜欢的主题，zsh 主题列表：https://github.com/robbyrussell/oh-my-zsh/wiki/themes

效果如下（配置了声明高亮）：  
![](/img/2018-10-4/oh-my-zshConfig2.png)

---

# 5.配置 Meslo 字体

使用上面的主题，需要 Meslo 字体支持，要不然会出现乱码的情况，字体下载地址：Meslo LG M Regular for Powerline.ttf

下载好之后，直接在 Mac OS 中安装即可。

然后打开 iTerm2，按`Command + ,`键，打开 Preferences 配置界面，然后`Profiles -> Text -> Font -> Chanage Font`，选择 Meslo LG M Regular for Powerline 字体。  
![](/img/2018-10-4/oh-my-zshConfig3.png)  
当然，如果你觉得默认的12px字体大小不合适，可以自己进行修改。

另外，VS Code 的终端字体，也需要进行配置，打开 VS Code，按`Command + ,`键，打开用户配置，搜索fontFamily，然后将右边的配置增加`"terminal.integrated.fontFamily": "Meslo LG M for Powerline"`，示例：  
![](/img/2018-10-4/oh-my-zshConfig4.png)  

---

# 6.声明高亮

效果就是上面截图的那样，特殊命令和错误命令，会有高亮显示。

使用 Homebrew 安装：

`$ brew install zsh-syntax-highlighting`
安装成功之后，编辑`vim ~/.zshrc`文件，在最后一行增加下面配置：

`source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh`  
![](/img/2018-10-4/oh-my-zshConfig5.png)  

# 7.自动建议填充

这个功能是非常实用的，可以方便我们快速的敲命令。

配置步骤，先克隆`zsh-autosuggestions`项目，到指定目录：

`$ git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions`
然后编辑`vim ~/.zshrc`文件，找到`plugins`配置，增加`zsh-autosuggestions`插件。  
![](/img/2018-10-4/oh-my-zshConfig6.png)  
注：上面声明高亮，如果配置不生效的话，在plugins配置，再增加`zsh-syntax-highlighting`插件试试。

有时候因为自动填充的颜色和背景颜色很相似，以至于自动填充没有效果，我们可以手动更改下自动填充的颜色配置，我修改的颜色值为：`586e75`，示例：   
![](/img/2018-10-4/oh-my-zshConfig7.png)   
效果：  
![](/img/2018-10-4/oh-my-zshConfig8.png)  

---

# 8.左右键跳转

主要是按住`option + → or ←`键，在命令的开始和结尾跳转切换，原本是不生效的，需要手动开启下。

打开 iTerm2，按`Command + ,`键，打开 `Preferences` 配置界面，然后`Profiles → Keys → Load Preset... → Natural Text Editing`，就可以了。

---

# 9.iTerm2 快速隐藏和显示

这个功能也非常使用，就是通过快捷键，可以快速的隐藏和打开 iTerm2，示例配置（`Commond + .`）：  
![](/img/2018-10-4/oh-my-zshConfig9.png)   

---

# 10.11. iTerm2 快捷命令

快捷命令说明：

```

命令	说明
command + t	新建标签
command + w	关闭标签
command + 数字 command + 左右方向键	切换标签
command + enter	切换全屏
command + f	查找
command + d	垂直分屏
command + shift + d	水平分屏
command + option + 方向键 command + [ 或 command + ]	切换屏幕
command + ;	查看历史命令
command + shift + h	查看剪贴板历史
ctrl + u	清除当前行
ctrl + l	清屏
ctrl + a	到行首
ctrl + e	到行尾
ctrl + f/b	前进后退
ctrl + p	上一条命令
ctrl + r	搜索命令历史


```

# 11.扩展
 
shell 就是和上面这些系统内核指令打交道的一座桥梁,我们通过键盘输入一种自己容易记忆识别的符号标识(shell 命令)
其实 zsh 也是一种 shell ,但是并不是我们系统默认的 shell ,unix 衍生系统的默认shell 都是 bash
 
```
chsh命令用于修改你的登录shell。
1 查看安装了哪些shell
cat /etc/shells
 
2 查看正在使用的shell
echo $SHELL
 
3 将shell改成zsh！
chsh -s /bin/zsh
然后重启你的shell,修改成功
 
 
cask常用命令：
brew cask search #列出所有可以被安装的软件
brew cask search php #查找所有和php相关的应用
brew cask list #列出所有通过cask安装的软件
brew cask info phpstorm #查看 phpstorm 的信息
brew cask uninstall qq #卸载 QQ
```


# 12.远程连接
 打开`Preferences` -> `Profiles` -> `左下角加号点击添加`
  
  ![iTerm2配置](/img/2018-10-4/iTerm2Config1.png)
  
  以后就可以直接在 `Profiles` 中登录远程服务器


