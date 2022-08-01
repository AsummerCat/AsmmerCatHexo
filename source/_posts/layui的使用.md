---
title:  layui的使用
date: 2018-09-27 14:32:31
tags:  [前端]
---
## 关闭当前窗口

```script
function clearALL(){
  		  //获取当前窗体索引
        var index=parent.layer.getFrameIndex(window.name); 
    	  //执行关闭  
   		 parent.layer.close(index); 
    }
```

<!--more-->

----



## 弹出层

```script
 layer.open({
        type: 2,
        title: ['添加显示城市', 'font-size:18px;'],
        content: ctx+ "/city/listAllAreaShow.do",
        //这里content是一个URL，如果你不想让iframe出现滚动条，你还可以content: ['url', 'no']
        area: ['25%', '75%'],
        scrollbar: false,
        shadeClose: false,
        btn: ['确定','关闭'],
        yes: function(index,layero){
            //当点击‘确定’按钮的时候，获取弹出层返回的值
            // var body = layer.getChildFrame('body', index);
            var iframeWin = window[layero.find('iframe')[0]['name']];
           var id= iframeWin.findAllSelectId();
           if(id==null||id==undefined||id==''){
               id='';
           }
           var text= iframeWin.findAllSelectText();
            if(text==null||text==undefined||text==''){
                text='';
            }
          $("#showCity").val(text);
          $("#showCityId").val(id);
            //最后关闭弹出层
            layer.close(index);
        },
        cancel: function(){
            //右上角关闭回调
        }
    })

```

---

## 父页面调用子页面JS方法

```script
//获取子页面窗口
var iframeWin = window[layero.find('iframe')[0]['name']];  
//调用子页面的js方法        
var id= iframeWin.findAllSelectId();  

```

---

## 父页面获取子页面元素

```script
//获取子页面的body		
var body = layer.getChildFrame('body', index); 
//判断不为空 再进行追加操作 不然会重复添加
            if(body.find('#city').val()){    
                body.find('#city').find("option:selected").text().trim())+",";
                body.find('#city').val())+",";
            }

```

