---
title: SpringMvc上传文件
date: 2018-09-20 22:02:14
tags: java
---

## 第一种方式
<!--more-->
```java
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

## 第二种方式

```

```

