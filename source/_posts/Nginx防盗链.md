---
title: nginx防盗链
date: 2019-04-19 07:27:18
tags: [nginx]
---

# nginx防盗链

```
location ~* \.(gif|jpg|png|swf|flv)$ {
root html
valid_referers none blocked *.nginxcn.com;
if ($invalid_referer) {
rewrite ^/ www.nginx.cn
#return 404;
}
}

```

前面的root可以不要如果你在server{}中有设置可以不需要设定

