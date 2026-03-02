---
title: nginx安装第三方插件之自定义头消息
date: 2022-08-31 14:00:34
tags: [nginx]
---
# nginx自定义头消息
内置的nginx没有这个功能,需要加载`headers-more-nginx-module`模块使用

## 1.下载
```
# 举例目录/tools
cd /tools/
#下载插件
wget https://github.com/openresty/headers-more-nginx-module/archive/v0.34.tar.gz
#解压
tar -zxvf v0.34.tar.gz
```
<!--more-->

## 2.加载模块
```
# 查询nginx路径
whereis nginx
# 查看安装参数命令(取出：configure arguments:)
/app/nginx/sbin/nginx -V
# 在nginx资源目录编译
cd /app/nginx-1.12.2/
# 将上面取出的configure arguments后面追加 
./configure --prefix=/usr/share/nginx --add-module=/nginx-module/headers-more-nginx-module-0.34
# 编辑，切记没有make install
make
# 备份
cp /app/nginx112/sbin/nginx /app/nginx112/sbin/nginx.bak 
# 覆盖(覆盖提示输入y)
cp -f /nginx源文件/objs/nginx /usr/sbin/nginx
```

## 3.修改配置
```
vim /app/nginx112/conf/nginx.conf
# 添加配置(在http模块)
more_clear_headers 'Server';
```
参考配置:
```
            if ($request_uri ~* "^/$"){
               more_clear_input_headers 'Referer';
            }    
```
文档参考:https://github.com/openresty/headers-more-nginx-module
## 4. 重启nginx
```
systemctl stop nginx
systemctl start nginx
```


# 如果是yum安装的nginx
## 1.查看 nginx 版本，并下载同版本的安装包
在进行替换/usr/sbin/nginx。切记：/usr/sbin/nginx 要进行备份。
```
nginx -v
nginx version: nginx/1.20.2
wget http://nginx.org/download/nginx-1.20.2.tar.gz

tar zxf nginx-1.20.2.tar.gz 
cd nginx-1.20.2

```
## 2.查看 nginx 编译命令
```
# 用大写的V就可以看到命令和版本

nginx -V

nginx version: nginx/1.20.2
built by gcc 8.5.0 20210514 (Red Hat 8.5.0-4) (GCC) 
built with OpenSSL 1.1.1k  FIPS 25 Mar 2021
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```

## 3.增加扩展，如增加 geoip
```
# 在最后添加 --with-http_geoip_module
./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-compat --with-debug --with-google_perftools_module --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_degradation_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_mp4_module --with-http_perl_module=dynamic --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_xslt_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-stream_ssl_preread_module --with-threads --with-http_geoip_module

```
或者末尾加上`--add-module=/xx/nginx-module/headers-more-nginx-module-0.34`


## 问题处理

#### 报错提示./configure: error: C compiler cc is not found
```
yum -y install gcc gcc-c++ autoconf automake make
```

#### 报错提示 ./configure: error: the invalid value in --with-ld-opt="-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E"
```
移除 ./configure中的  --with-ld-opt="-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E"
```

#### 报错提示 ./configure: no supported file AIO was found Currently file AIO is supported
```
去除./configure 中的 --with-file-aio
```

#### 报错提示 ./configure: error: can not detect int size
```
去掉--with-cc-opt=''的内容
```
#### 报错提示
```
/configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.
```
修复:
```
yum -y install pcre-devel
```

#### 报错提示 ./configure: error: SSL modules require the OpenSSL library.
```
yum -y install openssl openssl-devel make zlib zlib-devel gcc gcc-c++ libtool    pcre pcre-devel
```

#### 报错提示 ./configure: error: the HTTP XSLT module requires the libxml2/libxslt
```
sudo yum install -y libxml2-devel.x86_64
sudo yum install -y libxslt-devel.x86_64
```

#### 报错提示 ./configure: error: the HTTP image filter module requires the GD library.
```
yum install gd gd-devel
```

#### 报错提示 ./configure: error: perl module ExtUtils::Embed is required
```
yum -y install perl-devel perl-ExtUtils-Embed
```
#### 报错提示 ./configure: error: the Google perftools module requires the Google perftools
```
yum -y install gperftools
```
