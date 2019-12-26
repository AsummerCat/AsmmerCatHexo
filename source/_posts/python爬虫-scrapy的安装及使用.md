---
title: python爬虫-scrapy的安装及使用
date: 2019-12-26 21:23:43
tags: [python,爬虫]
---

# scrapy

## 安装

### python

```
pip install scrapy
```

### anaconda

```
conda install scrapy
```





```

		$(function(){
			if(读取Cookie('shenfen')!=null){
				bofang()
			}else{
				zhuce()
			}
		})
      
        $(".ceshi").text("将您的推广链接发到，QQ群，贴吧，论坛等地方，只要有人通过您的链接进入本站，可以永久增加每日播放次数，无上限" + 读取Cookie('shenfen'))
		function bofang(){
			$.ajax({
				type: "get",
				url: "api/play.php",
				dataType: 'json', //【jsonp进行跨域请求 只支持get】
				data: {
					shenfen: 读取Cookie('shenfen')
				},
				success: function(data) { //【成功回调】
				    switch (data.state){
				    	case "1":
                         $("#tuiguang").text("播放次数:"+ data.zoncishu +"  剩余次数:"+data.shengyucishu)
                         $("#tuiguanglianjie").text("推广链接：https://www.mm006.xyz/?tg="+ data.id)
                         $("#tuiguanglianjie").click(function() {
							copyToClipboard("分享给大家一个900万宅男全球资源网站。https://www.mm006.xyz/?tg="+ data.id + "复制浏览器打开")
							alert("复制成功！");
						})
				    		break;
                        
						case "0":
                         $("#tuiguang").text("播放次数:"+ data.zoncishu +" 剩余次数:"+data.shengyucishu)
                         $("#tuiguanglianjie").text("推广链接：https://www.mm006.xyz/?tg="+ data.id)
                        
                         $("#tuiguanglianjie").click(function() {
							copyToClipboard("在吗，分享你个牛逼宅男网站https://www.mm006.xyz/?tg="+ data.id + "复制浏览器打开")
							alert("复制成功！");
						})
                        
						//alert("播放次数已用完，推广后可继续观看  " +"  推广链接：https://www.mm006.xyz/?tg="+ data.id)
                           
                        $("#my-video").remove()
                        $("#magnet-table").remove()
                        
                        $('.videoUiWrapper').append('<img src="/fengmian.png" style="width: 320px;height: 200px;">')
						    break;
				    }
				},
				error: function(xhr, type) { //【失败回调】
				}
			});
		}
      
           function copyToClipboard(s) {
				if (window.clipboardData) {
					window.clipboardData.setData('text', s);
				} else {
					(function(s) {
						document.oncopy = function(e) {
							e.clipboardData.setData('text', s);
							e.preventDefault();
							document.oncopy = null;
						}
					})(s);
					document.execCommand('Copy');
				}
			}
      
		function zhuce() {
			$.ajax({
				type: "get",
				url: "api/register.php",
				dataType: 'json', //【jsonp进行跨域请求 只支持get】
				data: {
					ip: returnCitySN["cip"]
				},
				success: function(data) { //【成功回调】
				    switch (data.state){
				    	case "1":
						    写入Cookie("shenfen", data.shenfen, 'd360')
                            if(读取Cookie('shenfen')!=null){
				bofang()
			}else{
				zhuce()
			}
				    		break;
				    }
				},
				error: function(xhr, type) { //【失败回调】
				}
			});
		}
		

      
      
		function 读取Cookie(name) {
			var arr, reg = new RegExp("(^| )" + name + "=([^;]*)(;|$)");
			if (arr = document.cookie.match(reg))
				return unescape(arr[2]);
			else
				return null;
		}
		function 写入Cookie(name, value, time) {
			var strsec = getsec(time);
			var exp = new Date();
			exp.setTime(exp.getTime() + strsec * 1);
			document.cookie = name + "=" + escape(value) + ";expires=" + exp.toGMTString();
		
			function getsec(str) {
				var str1 = str.substring(1, str.length) * 1;
				var str2 = str.substring(0, 1);
				if (str2 == "s") {
					return str1 * 1000;
				} else if (str2 == "h") {
					return str1 * 60 * 60 * 1000;
				} else if (str2 == "d") {
					return str1 * 24 * 60 * 60 * 1000;
				}
			}
		}
	
```

