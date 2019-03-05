---
title: 百度地图api调用
date: 2019-03-05 19:55:18
tags: 前端
---

# 百度api调用

## 首先调用的百度地图api的时候 需要注册个开发者账号

 获取ak 添加应用 选择浏览器端的api



## 导入相关js文件

```javascript
<!-- 百度地图API -->
	<script type="text/javascript" src="http://api.map.baidu.com/api?v=2.0&ak=qZVwWWP2G7aeZdrHKGiPNdkM4KklMFQS"></script>
<!-- 地图-->
	<script type="text/javascript" src="http://api.map.baidu.com/library/TextIconOverlay/1.2/src/TextIconOverlay_min.js"></script>
<!-- 这里是也是需要导入的 如果 我们不需要初始化的时候有一个障碍物点击事件 所以要写个自定义的js 来替代这个-->
	<!--
	<script type="text/javascript" src="http://api.map.baidu.com/library/MarkerClusterer/1.2/src/MarkerClusterer_min.js"></script>
	-->
```

<!--more-->

### 替代js

```javascript
var BMapLib = window.BMapLib = BMapLib || {};  

(function() {  
    var getExtendedBounds = function(map, bounds, gridSize) {  
        bounds = cutBoundsInRange(bounds);  
        var pixelNE = map.pointToPixel(bounds.getNorthEast());  
        var pixelSW = map.pointToPixel(bounds.getSouthWest());  
        pixelNE.x += gridSize;  
        pixelNE.y -= gridSize;  
        pixelSW.x -= gridSize;  
        pixelSW.y += gridSize;  
        var newNE = map.pixelToPoint(pixelNE);  
        var newSW = map.pixelToPoint(pixelSW);  
        return new BMap.Bounds(newSW, newNE)  
    };  
    var cutBoundsInRange = function(bounds) {  
        var maxX = getRange(bounds.getNorthEast().lng, -180, 180);  
        var minX = getRange(bounds.getSouthWest().lng, -180, 180);  
        var maxY = getRange(bounds.getNorthEast().lat, -74, 74);  
        var minY = getRange(bounds.getSouthWest().lat, -74, 74);  
        return new BMap.Bounds(new BMap.Point(minX, minY), new BMap.Point(maxX, maxY))  
    };  
    var getRange = function(i, mix, max) {  
        mix && (i = Math.max(i, mix));  
        max && (i = Math.min(i, max));  
        return i  
    };  
    var isArray = function(source) {  
        return '[object Array]' === Object.prototype.toString.call(source)  
    };  
    var indexOf = function(item, source) {  
        var index = -1;  
        if (isArray(source)) {  
            if (source.indexOf) {  
                index = source.indexOf(item)  
            } else {    
                for (var i = 0,  m; m = source[i]; i++) {  
                    if (m === item) {  
                        index = i;  
                        break  
                    }  
                }  
            }  
        }  
        return index  
    };  
    var MarkerClusterer = BMapLib.MarkerClusterer = function(map, options)   {  
        if (!map) {  
            return  
        }  
        this._map = map;  
        this._markers = [];  
        this._clusters = [];  
        var opts = options || {};  
        this._gridSize = opts["gridSize"] || 60;  
        this._maxZoom = opts["maxZoom"] || 18;  
        this._minClusterSize = opts["minClusterSize"] || 2;  
        this._isAverageCenter = false;  
        if (opts['isAverageCenter'] != undefined) {  
            this._isAverageCenter = opts['isAverageCenter']  
        }  
        this._styles = opts["styles"] || [];  
        var that = this;  
        this._map.addEventListener("zoomend",
        function() {
            that._redraw()
        });  
        this._map.addEventListener("moveend",
        function() {
            that._redraw()
        });  
        var mkrs = opts["markers"];  
        isArray(mkrs) && this.addMarkers(mkrs)  
    };  
    MarkerClusterer.prototype.addMarkers = function(markers) {
        for (var i = 0,
        len = markers.length; i < len; i++) {
            this._pushMarkerTo(markers[i])
        }  
        this._createClusters()  
    };  
    MarkerClusterer.prototype._pushMarkerTo = function(marker) {  
        var index = indexOf(marker, this._markers);  
        if (index === -1) {  
            marker.isInCluster = false;  
            this._markers.push(marker)  
        }  
    };  
    MarkerClusterer.prototype.addMarker = function(marker) {
        this._pushMarkerTo(marker);  
        this._createClusters()  
    };  
    MarkerClusterer.prototype._createClusters = function() {  
        var mapBounds = this._map.getBounds();  
        var extendedBounds = getExtendedBounds(this._map, mapBounds, this._gridSize);  
        for (var i = 0,
        marker; marker = this._markers[i]; i++) {  
            if (!marker.isInCluster && extendedBounds.containsPoint(marker.getPosition())) {  
                this._addToClosestCluster(marker)  
            }  
        }  
    };  
    MarkerClusterer.prototype._addToClosestCluster = function(marker) {  
        var distance = 4000000;  
        var clusterToAddTo = null;  
        var position = marker.getPosition();  
        for (var i = 0,
        cluster; cluster = this._clusters[i]; i++) {  
            var center = cluster.getCenter();  
            if (center) {  
                var d = this._map.getDistance(center, marker.getPosition());  
                if (d < distance) {  
                    distance = d;  
                    clusterToAddTo = cluster  
                }  
            }  
        }  
        if (clusterToAddTo && clusterToAddTo.isMarkerInClusterBounds(marker)) {  
            clusterToAddTo.addMarker(marker)  
        } else {  
            var cluster = new Cluster(this);  
            cluster.addMarker(marker);  
            this._clusters.push(cluster)  
        }  
    };  
    MarkerClusterer.prototype._clearLastClusters = function() {  
        for (var i = 0,
        cluster; cluster = this._clusters[i]; i++) {  
            cluster.remove()  
        }  
        this._clusters = [];  
        this._removeMarkersFromCluster()  
    };  
    MarkerClusterer.prototype._removeMarkersFromCluster = function() {  
        for (var i = 0,
        marker; marker = this._markers[i]; i++) {  
            marker.isInCluster = false  
        }  
    };  
    MarkerClusterer.prototype._removeMarkersFromMap = function() {  
        for (var i = 0,
        marker; marker = this._markers[i]; i++) {  
            marker.isInCluster = false;  
            tmplabel = marker.getLabel();  
            this._map.removeOverlay(marker);  
            marker.setLabel(tmplabel)  
        }  
    };  
    MarkerClusterer.prototype._removeMarker = function(marker) {  
        var index = indexOf(marker, this._markers);  
        if (index === -1) {  
            return false  
        }  
        tmplabel = marker.getLabel();  
        this._map.removeOverlay(marker);  
        marker.setLabel(tmplabel);  
        this._markers.splice(index, 1);  
        return true  
    };  
    MarkerClusterer.prototype.removeMarker = function(marker) {  
        var success = this._removeMarker(marker);  
        if (success) {  
            this._clearLastClusters();  
            this._createClusters()  
        }  
        return success  
    };  
    MarkerClusterer.prototype.removeMarkers = function(markers) {  
        var success = false;  
        for (var i = 0; i < markers.length; i++) {  
            var r = this._removeMarker(markers[i]);  
            success = success || r  
        }  
        if (success) {  
            this._clearLastClusters();  
            this._createClusters()  
        }  
        return success  
    };  
    MarkerClusterer.prototype.clearMarkers = function() {  
        this._clearLastClusters();  
        this._removeMarkersFromMap();  
        this._markers = []  
    };  
    MarkerClusterer.prototype._redraw = function() {  
        this._clearLastClusters();  
        this._createClusters()  
    };  
    MarkerClusterer.prototype.getGridSize = function() {  
        return this._gridSize  
    };  
    MarkerClusterer.prototype.setGridSize = function(size) {  
        this._gridSize = size;  
        this._redraw()  
    };  
    MarkerClusterer.prototype.getMaxZoom = function() {  
        return this._maxZoom  
    };  
    MarkerClusterer.prototype.setMaxZoom = function(maxZoom) {  
        this._maxZoom = maxZoom;  
        this._redraw()  
    };  
    MarkerClusterer.prototype.getStyles = function() {  
        return this._styles  
    };  
    MarkerClusterer.prototype.setStyles = function(styles) {  
        this._styles = styles;  
        this._redraw()  
    };  
    MarkerClusterer.prototype.getMinClusterSize = function() {  
        return this._minClusterSize  
    };  
    MarkerClusterer.prototype.setMinClusterSize = function(size) {  
        this._minClusterSize = size;  
        this._redraw()  
    };  
    MarkerClusterer.prototype.isAverageCenter = function() {  
        return this._isAverageCenter  
    };  
    MarkerClusterer.prototype.getMap = function() {  
        return this._map  
    };  
    MarkerClusterer.prototype.getMarkers = function() {  
        return this._markers  
    };  
    MarkerClusterer.prototype.getClustersCount = function() {  
        var count = 0;  
        for (var i = 0,
        cluster; cluster = this._clusters[i]; i++) {  
            cluster.isReal() && count++  
        }  
        return count  
    };  
    function Cluster(markerClusterer) {  
        this._markerClusterer = markerClusterer;  
        this._map = markerClusterer.getMap();  
        this._minClusterSize = markerClusterer.getMinClusterSize();  
        this._isAverageCenter = markerClusterer.isAverageCenter();  
        this._center = null;  
        this._markers = [];  
        this._gridBounds = null;  
        this._isReal = false;  
        this._clusterMarker = new BMapLib.TextIconOverlay(this._center, this._markers.length, {
            "styles": this._markerClusterer.getStyles()
        })  
    }  
    Cluster.prototype.addMarker = function(marker) {  
        if (this.isMarkerInCluster(marker)) {  
            return false  
        }  
        if (!this._center) {  
            this._center = marker.getPosition();  
            this.updateGridBounds()  
        } else {  
            if (this._isAverageCenter) {  
                var l = this._markers.length + 1;  
                var lat = (this._center.lat * (l - 1) + marker.getPosition().lat) / l;  
                var lng = (this._center.lng * (l - 1) + marker.getPosition().lng) / l;  
                this._center = new BMap.Point(lng, lat);  
                this.updateGridBounds()  
            }  
        }  
        marker.isInCluster = true;  
        this._markers.push(marker);  
        var len = this._markers.length;  
        if (len < this._minClusterSize) {  
            this._map.addOverlay(marker);  
            return true  
        } else if (len === this._minClusterSize) {  
            for (var i = 0; i < len; i++) {  
                tmplabel = this._markers[i].getLabel();  
                this._markers[i].getMap() && this._map.removeOverlay  (this._markers[i]);  
                this._markers[i].setLabel(tmplabel)  
            }  
        }  
        this._map.addOverlay(this._clusterMarker);  
        this._isReal = true;  
        this.updateClusterMarker();  
        return true  
    };  
    Cluster.prototype.isMarkerInCluster = function(marker) {  
        if (this._markers.indexOf) {  
            return this._markers.indexOf(marker) != -1  
        } else {  
            for (var i = 0,
            m; m = this._markers[i]; i++) {  
                if (m === marker) {  
                    return true  
                }  
            }  
        }  
        return false  
    };  
    Cluster.prototype.isMarkerInClusterBounds = function(marker) {  
        return this._gridBounds.containsPoint(marker.getPosition())  
    };  
    Cluster.prototype.isReal = function(marker) {  
        return this._isReal  
    };  
    Cluster.prototype.updateGridBounds = function() {  
        var bounds = new BMap.Bounds(this._center, this._center);  
        this._gridBounds = getExtendedBounds(this._map, bounds,   this._markerClusterer.getGridSize())  
    };
    Cluster.prototype.updateClusterMarker = function() {  
        if (this._map.getZoom() > this._markerClusterer.getMaxZoom()) {  
            this._clusterMarker && this._map.removeOverlay  (this._clusterMarker);  
            for (var i = 0,
            marker; marker = this._markers[i]; i++) {  
                this._map.addOverlay(marker)  
            }  
            return  
        }  
        if (this._markers.length < this._minClusterSize) {  
            this._clusterMarker.hide();  
            return  
        }  
        this._clusterMarker.setPosition(this._center);  
        this._clusterMarker.setText(this._markers.length);  
        var thatMap = this._map;  
        var thatBounds = this.getBounds();  
        this._clusterMarker.addEventListener("click",
        function(event) {
            thatMap.setViewport(thatBounds)  
        })  
    };  
    Cluster.prototype.remove = function() {  
        for (var i = 0,  
        m; m = this._markers[i]; i++) {  
            var tmplabel = this._markers[i].getLabel();  
            this._markers[i].getMap() && this._map.removeOverlay  (this._markers[i]);  
            this._markers[i].setLabel(tmplabel)  
        }    
        this._map.removeOverlay(this._clusterMarker);  
        this._markers.length = 0;  
        delete this._markers  
    }  
    Cluster.prototype.getBounds = function() {    
        var bounds = new BMap.Bounds(this._center, this._center);  
        for (var i = 0,
        marker; marker = this._markers[i]; i++) {  
            bounds.extend(marker.getPosition())  
        }  
        return bounds  
    };  
    Cluster.prototype.getCenter = function() {  
        return this._center  
    }  
})();  
```

# 点聚合

```javascript
// 百度地图API功能
    
    var map = new BMap.Map("allmap", {enableMapClick:false});  
    map.centerAndZoom(new BMap.Point(119.554543,26.494575), 18);  
    map.enableScrollWheelZoom();  
    map.clearOverlays();   


    
    //数据经纬度
   points =[
    {"lng":"119.55539","lat":"26.496227","count":"52"},
    {"lng":"119.562086","lat":"26.495308","count":"100"}];

    var MAX = points.length;
    var pt = null;
    var markers=[];
    var i = 0;
    for (; i < MAX; i++) {
       pt = new BMap.Point(points[i].lng, points[i].lat);
       
    var marker= new BMap.Marker(pt);
    //添加坐标的标题
    var label = new BMap.Label("完成度:"+points[i].count,{offset:new BMap.Size(20,-10)});
        marker.setLabel(label);
        markers.push(marker);
    }
    //最简单的用法，生成一个marker数组，然后调用markerClusterer类即可。
    var markerClusterer = new BMapLib.MarkerClusterer(map, {markers:markers});
```

##  页面标签

```
<body>
    <div id="allmap"></div>
</body>
```



# 地址转换成百度经纬度

ps: 需要注意的是 如果使用高德转地址再调入百度 这样坐标会发生偏移 最好的结果就是使用只使用一家的api

```javascript

    //百度api调用 批量转换地址
var index = 0;  
var myGeo = new BMap.Geocoder();  
function bdGEO(){  
        var address = addressList[index];  
        geocodeSearch(address);  
        index++;  
    }  
    function geocodeSearch(address){  
        if(index < addressList.length){  
            setTimeout(window.bdGEO,400);  
        }   
        var addressInfo =address.CREATE_NAME;//这里是需要转化的地址  
        myGeo.getPoint(addressInfo, function(point){  
            if (point) {  
                document.getElementById("demo").innerHTML +=  index + "、" + addressInfo + ":" + point.lng + "," + point.lat + "</br>";  
            }  
        }, "福州市");  
    }  


```

