---
title: echarts图表生成工具
date: 2019-03-27 21:09:44
tags: [工具类,前端]
---

# echarts图表生成工具

# 利用echarts快速生成js暂时折线图 或柱形图

## 首先导入js

[在线定制js](https://echarts.baidu.com/builder.html)
这边可以挑选需要的图表生成js 

## 挑选你喜欢的模板 可提供下载js

[模板下载](https://echarts.baidu.com/examples/#)

# 创建demo

## 导入js

```
<script type="text/javascript" src="js/echarts.js"></script>

```

## 准备一个放图表的容器

```
<div id="chartmain" style="width:600px; height: 400px;"></div>

```

<!--more-->

## 设置参数，初始化图表

```
<script type="text/javascript">
        //指定图标的配置和数据  
        var option = {  
            title:{  
                text:'ECharts 数据统计'  
            },  
            tooltip:{},  
            legend:{  
                data:['用户来源']  
            },  
            xAxis:{  
                data:["Android","IOS","PC","Ohter"]  
            },  
            yAxis:{  
 
            },  
            series:[{  
                name:'访问量',  
                type:'line',  
                data:[500,200,360,100]  
            }]  
        };  
        //初始化echarts实例  
        var myChart = echarts.init(document.getElementById('chartmain'));  

        //使用制定的配置项和数据显示图表  
        myChart.setOption(option);  
    </script>
  
```

柱状图其实也很简单，只要修改一个参数就可以了。把series里的type 值修改为"bar"



 饼图和折线图、柱状图有一点区别。主要是在参数和数据绑定上。饼图没有X轴和Y轴的坐标，数据绑定上也是采用value 和name对应的形式。

```
 var option = {  
            title:{    
                text:'ECharts 数据统计'  
            },                
            series:[{  
                name:'访问量',  
                type:'pie',      
                radius:'60%',   
                data:[  
                    {value:500,name:'Android'},  
                    {value:200,name:'IOS'},  
                    {value:360,name:'PC'},  
                    {value:100,name:'Ohter'}   
                ]  
            }]  
        };  

```

## 利用ajax动态生图表数据 (折线图)

### 后台方法

```
    @RequestMapping(params = "doJson")  
    @ResponseBody  
    public AjaxJson doJson() {  
        AjaxJson j = new AjaxJson();  
        List list=new ArrayList();  
  
        Map map1=new HashMap<String,Object>();  
        List list1=new ArrayList();  
        list1.add(100);  
        list1.add(200);  
        list1.add(300);  
        list1.add(400);  
        list1.add(500);  
        map1.put("title","demo");  
        map1.put("name","测试数据");  
        map1.put("data",list1);  
        map1.put("markPoint",1);  
        map1.put("itemStyle",0);  

        Map map2=new HashMap<String,Object>();  
        List list2=new ArrayList();  
        list2.add(450);  
        list2.add(450);  
        list2.add(450);  
        list2.add(450);  
        map2.put("title","demo");  
        map2.put("name","最大值");  
        map2.put("data",list2);  
        map2.put("markPoint",0);  
        map2.put("itemStyle",1);  


        Map map3=new HashMap<String,Object>();  
        List list3=new ArrayList();  
        list3.add(50);  
        list3.add(50);  
        list3.add(50);  
        list3.add(50);  
        map3.put("title","demo");  
        map3.put("name","最小值");  
        map3.put("data",list3);  
        map3.put("markPoint",0);  
        map3.put("itemStyle",1);  



        list.add(map1);  
        list.add(map2);  
        list.add(map3);  

        j.setObj(list);  
        return j;   
    }  
```

### 前台js 控制

```
<script type="text/javascript">  
    $(document).ready(function() {  
  
        $.ajax({  
            type: "GET",  
            url: "pRdrugReviewController.do?doJson",  
            dataType: "json",  
            success: function (result) {  
                console.log(result);  
                var data=result.obj;  
                if(data==null||data==undefined||data==''){  
                    console.log("暂无结果");  
                    return '暂无结果';  
                }  

                     //标题  
                        var  title=""  
                    // 表头  
                        var titleLegend=[];  
                    //数据内容  
                       var seriesData=[];  
                for (var i = 0; i < data.length; i++) {  
                     var infoData=data[i];  
                    title=infoData.title;  
                    titleLegend.push(infoData.name);  

                    var infoSeriesData=infoData.data;  
                    var addData={};  
                    addData.name=infoData.name;  
                    addData.type="line";  
                    var infoSeries=[];  
                    for (var j = 0; j < infoSeriesData.length; j++) {   
                        infoSeries.push(infoSeriesData[j]);  
                    }  
                    addData.data=infoSeries;  
                    //是否需要显示最大最小值  
                    if(infoData.markPoint==1){  
                        var markPoint={};  
                       var markPointData=[];  
                        markPointData.push({type: 'max', name: '最大值'});  
                        markPointData.push({type: 'min', name: '最小值'});  
                        markPoint.data=markPointData;  
                        addData.markPoint=markPoint;  
                    }  
                    //是否需要显示虚线
                    if(infoData.itemStyle==1){  
                        var itemStyle={};  
                        var normal={};  
                        var lineStyle={};  
                        lineStyle.type='dashed';  
                        lineStyle.width=2;  
                        normal.lineStyle=lineStyle;  
                        itemStyle.normal=normal;  
                        addData.itemStyle=itemStyle;  
                    }  
                        seriesData.push(addData);  
                }  

                $('#body').append("<div id="+title+" style='width: 600px;height:400px;'></div>");  

                var myChart = echarts.init(document.getElementById(title));  
                //设置图标配置项
                myChart.setOption({  
                    title:{  
                        text:title  
                    },  
                    tooltip: {  
                        trigger: 'axis'  
                    },  
                    grid: {  
                        left: '3%',  
                        right: '4%',  
                        bottom: '3%',  
                        containLabel: true  
                    },  
                    toolbox: {  
                        feature: {  
                            saveAsImage: {}  
                        }  
                    },  
                    legend:{  
                        data:titleLegend  
                    },  
                    xAxis:{  
                        data:[1,2,3,4,5]  
                    },  
                    yAxis:{},  
                    series:seriesData  
                })    
            }  
        });  
    });  

</script>  
```