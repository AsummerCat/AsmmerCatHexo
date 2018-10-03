---

title: 上传文件前端图片压缩后上传(base64)
date: 2018-09-20 22:02:14
tags: [前端,java]

---
 ** 之前做项目,有涉及到多张大图片上传效率很慢 有考虑使用 
 1.多线程上传 结果发现 数据还是需要先进入servce层进行 处理  没啥用
 2.使用第三方图片保存文件 结果可能图片还是有点大 没保存成功 
 3.最后选择了用图片压缩成base64 然后上传服务器 大概能3M转到30兆 
 4.(当然 有的方法比这个更好 反正我只是记录下操作历史)
 <!--more-->
 5.比如 腾讯好像有出一个图片压缩的开源 叫啥忘记了 总知有很多方式啦 
 **

----

主要内容是:

```
1.创建画布(压缩的成品) ,不想显示就隐藏(但是我们需要这个) 
	<canvas id="图片回显Id" style=display:none"></canvas>

2.获取前端图片内容:
	var businessLicense=document.getElementById("图片ID").files[0];

3.创建一个前端文件流 进行处理
var reader=new FileReader();   //前端文件流
    reader.readAsDataURL(businessLicense);
    reader.onload=function(e){
        createCanvas(this.result,"图片回显Id","图片.jpg",businessLicense.name)  //这里处理数据
    }
    
4.createCanvas ();具体处理数据

4.1 设置画布基本大小
	var canvas=docment.getElementById(页面上的画布Id);
	var cxt=canvas.getContext('2d'); //画布大小
	canvas.width=640; //手动设置画布大小
	canvas.height=400;
	// width=Img.width, //确保canvas的尺寸和图片一样
     //height=Img.height; 
     //canvas.width=width;
      //canvas.height=height;
	
4.2 获取生成的图片set到画布上去
var img=new Image(); //生成的图片
img.src=src; //这里的src是指之前上传文件的图片流
    img.onload=function(){
        cxt.drawImage(img,0,0,640,400);//生成了    后两个属性 宽 高
        $("#"+showImg).show().attr('src',canvas.toDataURL("image/jpeg",0.9));//加载到页面显示
        }
  
```



----

## 以下是例子:

 ## <font color="red">前端部分:</font>

```html
    PC:<input type="file" id="idCard">
    APP:<input type="file" id="idCard" accept="image/**">
    显示图片:<canvas id="图片回显Id" style=display:none"></canvas>
```
---

##  <font color="red">js部分:</font>

```javascript

$().ready(function(){
        #(“图片ID”).on(‘change’,function(event){
    var businessLicense=document.getElementById("图片ID").files[0];//图片文件内容
   if(!checkImgType(businessLicense)||!checkImgSize(businessLicense)){//判断文件是否符合要求
return false
   }
var reader=new FileReader();   //前端文件流
    reader.readAsDataURL(businessLicense);
    reader.onload=function(e){
        createCanvas(this.result,"图片回显Id","图片.jpg",businessLicense.name)  //这里处理数据
    }
        })
    }
)
```

#### 图片校验类型

```javascript
//图片校验类型
function checkImgType(file){
var fileName=file.name;
var flag=true;
var imgType=fileName.substring(fileName.lastIndexOf('.')+1,fileName.length).toLocaleLowerCase();
    if(imgType !='jpg' && imgType!='bmp' && imgType!='jpeg'){
        layer.msg(fileName+"文件大小超过2M",{time:1500,shift:6});
        return false
    }
    return flag;
}

//图片校验文件大小
function checkImgSize(file){
    var flag=true
    if(file.size>1024*1024*2){
        layer.msg(fileName+"只能上传jpg,bmp,png,jpeg类型的图片",{time:1500,shift:6});
        return false
    }
    return flag
}

```

#### //上传图片内容到后台

```javascript
//上传图片内容到后台
//src  图片流
//showImg  图片回显Id
//fileName 提交到后台的名称
//oldFileName 页面文件原来的名称
function createCanvas(src,showImg,fileName,oldFileName){
var token =$("需要提交其他的参数").val();
var canvas=docment.getElementById(showImg);
var cxt=canvas.getContext('2d'); //画布大小
canvas.width=640;
canvas.height=400;
var img=new Image(); //生成的图片
img.src=src;
    img.onload=function(){
        cxt.drawImage(img,0,0,640,400);//生成了
        $("#"+showImg).show().attr('src',canvas.toDataURL("image/jpeg",0.9));//加载到页面显示
            $.ajax({
            url :'请求路径',
            data:{
                "data":canvas.toDataURL("image/jpeg",0.9).split(',')[1],
                "其他参数":token,
                "fileName":fileName
                },
            type:'post',
            cache:false,
            success:function(data){
                if(data=true){
                layer.msg(oldFileName+"上传成功",{time:1500,shift:6});
                }else{
                layer.msg(oldFileName+"上传失败",{time:1500,shift:6});
                }
            }
        })
    }
}
```



##  <font color="red">后端部分:</font>

```java
@RequsestMapping(value="路径",method=RequstMethod.POST)
@ResponseBody
public boolean uploadImg(String token,String fileName,String data)throws Exception{
        boolean flag=true;
        BASE64Decoder decoder=new BASE64Decoder();
        byte[] image=decoder.decodeBuffer(data);

        //创建文件
        try{
        //含文件名的全路径
        String path=路径地址 + File.separator + fileName;

        File file=new File(path);
        if(!file.getParentFile().exists()){ //如果父目录不存在,创建父目录
            file.getParentFile().mkdirs();
        }
        if(file.exists()){    //如果已存在,删除旧文件
            file.delete();
        }
        InputStream is =new ByteArrayInputStream(files); //图片流
        BufferedImage image=ImageIO.read(is);
        flag=ImageIO.write(image,"jpg",new File(Path));
    }catch(Exception e){
        flag=false;
        e.printStackTrace();
    }
    return flag;
}

```

