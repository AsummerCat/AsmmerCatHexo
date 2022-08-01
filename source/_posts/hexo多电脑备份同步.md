---
title: hexo多电脑备份同步
date: 2018-10-05 09:33:31
tags: [hexo]
---

## hexo多电脑同步
很多时候人要在两台电脑上操作hexo随时发表文章，我们将整个hexo目录作为一个私有git仓库，push到http://git.oschina.net.  这里没有采用不同branch的管理方式，而是将其作为一个完全独立的私有仓库，托管到码云上.
<!--more-->


在hexo目录下执行git init,然后 
注意:因为theme目录下的next也是一个git仓库，而根目录下的git仓库是管不了子仓库的。后边需要对这个next theme作大量定制，因此用submodule的方式管理不合适。我们直接把next下的.git文件夹删除即可，这样就可以被外边的git仓库管理了 
然后进行一次`git add .`,然后`git commit -m "add:first commit`".之后push到你在oschina上建的仓库.

这样在另外一台电脑第一次操作时，先把hexo整个仓库git clone下来，然后在hexo目录下执行  
`npm install hexo --save `
再执行一次  ` npm install`  然后


```
hexo clean
hexo g
hexo s
```

在本地测试下。每次在不同电脑上发表博文时，先git pull一次，然后再写文章。

---


##  上传文章后 执行直接备份

```
git add .
git commit -m "更新hexo源文件"
git push origin master
```

---

## 当远程仓库有更新时，执行以下命令，即可同步hexo源文件到本地。

`git pull origin master`

---
