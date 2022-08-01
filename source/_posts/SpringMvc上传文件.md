---
title: SpringMvc上传文件
date: 2018-09-20 22:02:14
tags: [SpringMvc]
---

## 第一种方式(推荐)

```
<bean id="multipartResolver"    
        class="org.springframework.web.multipart.commons.CommonsMultipartResolver">    
        <property name="defaultEncoding" value="UTF-8" />    
    
        <property name="maxUploadSize" value="2000000000" />    
    </bean>    


public JsonResult upload(@RequestParam(value = "img") List<MultipartFile> list,HttpServletRequest request) {
     String fileimages="";
    String saveFilePath = request.getServletContext().getRealPath("/upload/file");  //存放地址
    try {

        if(list.size()<=9){ //判断文件的个数
            for (MultipartFile file : list) {
                String fileName = file.getOriginalFilename();
                String saveName =  generateFileName(fileName);
                File localFile = new File(saveFilePath + File.separator + saveName);//新文件
                String ext = localFile.getPath().substring(
                        localFile.getPath().lastIndexOf(".") + 1);
                // 检查文件类型
                checkAllowFiles (ext);
                File dirPath = new File(saveFilePath);
                if (!dirPath.exists()) {//文件目录不存在
                    dirPath.mkdirs();
                }
                file.transferTo(localFile);//复制到新文件中
                fileimages += localFile.getName()+",";
            }
        }else{
            return JsonResult.failed("上传图片请小于等于9张");
        }
    } catch (IllegalStateException e) {
        return JsonResult.failed(e.getMessage());
    }catch (MaxUploadSizeExceededException e){
        return JsonResult.failed("请上传小于10m的图片");
    }
    catch (Exception e) {
        return JsonResult.failed(e.getMessage());
    }
    return JsonResult.success(fileimages);
}


/**
 * 得到文件名
 */
private static String getFileExt(String name) {
    int pos = name.lastIndexOf(".");
    if (pos > 0) {
        return name.substring(pos);
    } else {
        return name;
    }
}

/**
*校验文件类型
**/
private static void checkAllowFiles(String ext) throws IOException {
            String allowFiles="png,jpg,jpeg";
    if (allowFiles != null) {
        if (!allowFiles.contains(ext.toLowerCase())) {
            throw new IOException("暂不支持上传该类型的文件！");
        }
    }
}

```

---

<!--more-->

## 第二种方式

```
/*
     * 通过流的方式上传文件
     * @RequestParam("file") 将name=file控件得到的文件封装成CommonsMultipartFile 对象
     */
    @RequestMapping("fileUpload")
    public String  fileUpload(@RequestParam("file") CommonsMultipartFile file) throws IOException {
         
        System.out.println("fileName："+file.getOriginalFilename());
         
        try {
            //获取输出流
            OutputStream os=new FileOutputStream("E:/"+new Date().getTime()+file.getOriginalFilename());
            //获取输入流 CommonsMultipartFile 中可以直接得到文件的流
            InputStream is=file.getInputStream();
            int temp;
            //一个一个字节的读取并写入
            while((temp=is.read())!=(-1))
            {
                os.write(temp);
            }
           os.flush();
           os.close();
           is.close();
         
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return "/success"; 
    }
```

## 第三种方式(推荐)

```
/*
     * 采用file.Transto 来保存上传的文件
     */
    @RequestMapping("fileUpload2")
    public String  fileUpload2(@RequestParam("file") CommonsMultipartFile file) throws IOException {
        System.out.println("fileName："+file.getOriginalFilename());
        String path="E:/"+new Date().getTime()+file.getOriginalFilename();
         
        File newFile=new File(path);
        //通过CommonsMultipartFile的方法直接写文件（注意这个时候）
        file.transferTo(newFile);
        long  endTime=System.currentTimeMillis();
        return "/success"; 
    }
```

## 第四种方式

```
/*
     *采用spring提供的上传文件的方法
     */
    @RequestMapping("springUpload")
    public String  springUpload(HttpServletRequest request) throws IllegalStateException, IOException
    {
         //将当前上下文初始化给  CommonsMutipartResolver （多部分解析器）
        CommonsMultipartResolver multipartResolver=new CommonsMultipartResolver(
                request.getSession().getServletContext());
        //检查form中是否有enctype="multipart/form-data"
        if(multipartResolver.isMultipart(request))
        {
            //将request变成多部分request
            MultipartHttpServletRequest multiRequest=(MultipartHttpServletRequest)request;
           //获取multiRequest 中所有的文件名
            Iterator iter=multiRequest.getFileNames();
             
            while(iter.hasNext())
            {
                //一次遍历所有文件
                MultipartFile file=multiRequest.getFile(iter.next().toString());
                if(file!=null)
                {
                    String path="E:/springUpload"+file.getOriginalFilename();
                    //上传
                    file.transferTo(new File(path));
                }
                 
            }
           
        }
    return "/success"; 
    }
```

