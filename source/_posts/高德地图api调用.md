---
title: 高德地图api调用
date: 2019-03-05 19:59:21
tags: 前端
---

# 高德地图api调用

## 首先导入相关js

```javascript
<!-- 经纬度转换-->
     <script type="text/javascript" src="http://webapi.amap.com/maps?v=1.4.6&key=df4aa96bd16299b2ea11c2b572aab88a&plugin=AMap.Geocoder"></script>   
<!--jqery-->
    <script src="http://libs.baidu.com/jquery/2.1.4/jquery.min.js"></script>
```

<!--more-->

## 坐标转换

### 页面

```javascript
<body>
<button  onclick="test()">点击事件</button>
	  <div id="demo"></div>
</body>
```

### js

```javascript
<script>

function test(){   
var geocoder = new AMap.Geocoder({  
    // city 指定进行编码查询的城市，支持传入城市名、adcode 和 citycode  
    city: '全国'  
  });  
  var address = '罗源县公安局(城关派出所)';//这里是需要转化的地址  

  geocoder.getLocation(address, function(status, result) {  
    if (status === 'complete' && result.info === 'OK') {  
	console.log(result);  
      // result为对应的地理位置详细信息  
    }  
  })  
  }  

</script>  
```

