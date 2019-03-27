---
title: 使用openOffice预览word全家桶
date: 2019-03-27 21:07:54
tags: [java]
---

# 使用openOffice预览word全家桶

[外部调用第三方在线预览网站](https://products.office.com/zh-CN/office-online/view-office-documents-online)  

## 安装openOffice

[下载地址](http://www.openoffice.org/)  
根据系统类型分别下载  

## 注意内容

<font color="red"> 因为转换的pdf是标准的A4大小 如果xlex列太多 可能会导致换行溢出</font>

最好是 分开转换 或者搭配itext 

<!--more-->

## 安装相关jar

```
  <!-- openoffice -->
        <openoffice.version>3.2.1</openoffice.version>
        <jodconverter.version>3.0-beta-4</jodconverter.version>


 <dependency>
            <groupId>org.openoffice</groupId>
            <artifactId>juh</artifactId>
            <version>${openoffice.version}</version>
        </dependency>
        <dependency>
            <groupId>org.openoffice</groupId>
            <artifactId>unoil</artifactId>
            <version>${openoffice.version}</version>
        </dependency>
        <dependency>
            <groupId>org.openoffice</groupId>
            <artifactId>jurt</artifactId>
            <version>${openoffice.version}</version>
        </dependency>


  <dependency>
            <groupId>org.artofsolving</groupId>
            <artifactId>jodconverter</artifactId>
            <version>${jodconverter.version}</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/src/main/webapp/WEB-INF/lib/jodconverter-3.0-beta-4-20170917.jar</systemPath>
        </dependency>

```

## 在配置文件中添加安装地址

```
 ResourceUtil.getSessionattachmenttitle("office_home");  

office_home=C:\\Program Files (x86)\\OpenOffice 4  

```

## 添加工具类

```
//连接池类似
private static OfficeManager officeManager;

/** OpenOffice安装根目录 */  
  private static String OFFICE_HOME = ConStant.OFFICE_HOME;  
  private static int[] port = { 8100 };  
```

## 首先每次启动都要开启服务 处理结束 然后关闭服务

### 开启服务 

```
public static void startService() {  
   DefaultOfficeManagerConfiguration configuration = new DefaultOfficeManagerConfiguration();  
   try {  
      // 准备启动服务  
      configuration.setOfficeHome(OFFICE_HOME);// 设置OpenOffice.org安装目录  
      // 设置转换端口，默认为8100  
      configuration.setPortNumbers(port);  
      // 设置任务执行超时为5分钟  
      configuration.setTaskExecutionTimeout(1000 * 60 * 5L);  
      // 设置任务队列超时为24小时  
      configuration.setTaskQueueTimeout(1000 * 60 * 60 * 24L);  

      officeManager = configuration.buildOfficeManager();  
      officeManager.start(); // 启动服务  
      log.info("office转换服务启动成功!");  
    } catch (Exception ce) {  
      log.info("office转换服务启动失败!详细信息:" + ce);  
    }  
  } 
```

### 关闭服务

```
	public static void stopService() {  
   log.info("关闭office转换服务....");  
    if (officeManager != null) {  
      officeManager.stop();  
   }  
    log.info("关闭office转换成功!");  
  }

```

## 转换

```
public void convert2PDF(String inputFile, String extend) {    
   String pdfFile = FileUtils.getFilePrefix2(inputFile) + ".pdf";    
      startService();  
      log.info("进行文档转换转换:" + inputFile + " --> " + pdfFile);  
    OfficeDocumentConverter converter = new OfficeDocumentConverter(officeManager);  
    try {  
      converter.convert(new File(inputFile), new File(pdfFile));  
   } catch (Exception e) {  
      e.printStackTrace();    
      log.info(e.getMessage());  
   }  
   
   stopService();  
      log.info("进行文档转换转换---- 结束----");  
}  

```

## 调用

```
  OpenOfficePDFConverter openOfficePDFConverter = new OpenOfficePDFConverter();  
                openOfficePDFConverter.convert2PDF(fileAddress, null);  

```

# 完整代码

```
package org.jeecgframework.core.extend.swftools;

import java.io.File;

import org.artofsolving.jodconverter.OfficeDocumentConverter;
import org.artofsolving.jodconverter.office.DefaultOfficeManagerConfiguration;
import org.artofsolving.jodconverter.office.OfficeManager;
import org.jeecgframework.core.util.FileUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


/**
 * OFFICE文档转换服务类
 * 
 */
public class OpenOfficePDFConverter{
  private static final Logger log = LoggerFactory.getLogger(OpenOfficePDFConverter.class);

  private static OfficeManager officeManager;
  /** OpenOffice安装根目录 */
  private static String OFFICE_HOME = ConStant.OFFICE_HOME;
  private static int[] port = { 8100 };

  public void convert2PDF(String inputFile, String pdfFile, String extend) {
    startService();
    log.info("进行文档转换转换:" + inputFile + " --> " + pdfFile);
    OfficeDocumentConverter converter = new OfficeDocumentConverter(officeManager);
    try {
      converter.convert(new File(inputFile), new File(pdfFile));
      } catch (Exception e) {
      e.printStackTrace();
      log.info(e.getMessage());
  }
    
    stopService();
      log.info("进行文档转换转换---- 结束----");
  }

  public void convert2PDF(String inputFile, String extend) {
    String pdfFile = FileUtils.getFilePrefix2(inputFile) + ".pdf";
    convert2PDF(inputFile, pdfFile, extend);
	}

  public static void startService() {
   DefaultOfficeManagerConfiguration configuration = new DefaultOfficeManagerConfiguration();
    try {
      // 准备启动服务
      configuration.setOfficeHome(OFFICE_HOME);// 设置OpenOffice.org安装目录
      // 设置转换端口，默认为8100
      configuration.setPortNumbers(port);
      // 设置任务执行超时为5分钟
      configuration.setTaskExecutionTimeout(1000 * 60 * 5L);
      // 设置任务队列超时为24小时
      configuration.setTaskQueueTimeout(1000 * 60 * 60 * 24L);

      officeManager = configuration.buildOfficeManager();
      officeManager.start(); // 启动服务
      log.info("office转换服务启动成功!");
    } catch (Exception ce) {
      log.info("office转换服务启动失败!详细信息:" + ce);
    }
  }

  public static void stopService() {
    log.info("关闭office转换服务....");
    if (officeManager != null) {
      officeManager.stop();
    }
    log.info("关闭office转换成功!");
  }
}

```

# 搭配iext 实现 预览全格式 不能预览的就直接下载

```
 /**
     * itext预览
     */
    @RequestMapping(params = "openITextViewFile")  
    public void openITextViewFile(HttpServletRequest request, HttpServletResponse response) throws Exception {  
        String fileid = request.getParameter("fileid");  
        String subclassname = oConvertUtils.getString(request.getParameter("subclassname"), "org.jeecgframework.web.system.pojo.base.TSAttachment");  
        Class<?> fileClass = MyClassLoader.getClassByScn(subclassname);// 附件的实际类  
        Object fileobj = systemService.getEntity(fileClass, fileid);  
        ReflectHelper reflectHelper = new ReflectHelper(fileobj);  
        String ctxPath = ResourceUtil.getConfigByName("webUploadpath");//demo中设置为D://upFiles,实际项目应因事制宜  
        String realpath = oConvertUtils.getString(reflectHelper.getMethodValue("realpath"));  
        //地址全路径  
        String fileAddress = ctxPath + File.separator + realpath;  
        //后缀  
        String extend = oConvertUtils.getString(reflectHelper.getMethodValue("extend"));  
        //输出格式  
        response.setContentType("application/pdf");  

        OutputStream out = response.getOutputStream();  
        InputStream in = null;  
        //需要导出的名称  
        String outFileName;  
 
        //如果是pdf单独处理  
        if ("pdf".equals(extend)) {  
            outFileName = fileAddress;  
        } else if ("doc".equals(extend) || "docx".equals(extend) || "ppt".equals(extend) || "pptx".equals(extend) || FileUtils.isPicture(extend)) {  
            String pdfFile = FileUtils.getFilePrefix2(fileAddress) + ".pdf";  
            File file = new File(pdfFile);  
            if (!file.exists()) {  
                OpenOfficePDFConverter openOfficePDFConverter = new  OpenOfficePDFConverter();  
                openOfficePDFConverter.convert2PDF(fileAddress, null);  
            }  
            outFileName = pdfFile;  
        } else if ("xls".equals(extend) || "xlsx".equals(extend) || "txt".equals(extend)) {  
            outFileName = fileAddress;  
            File file = new File(outFileName);  
            response.setContentType("multipart/form-data");  
            response.setHeader("Content-Disposition", "attachment;fileName=" + java.net.URLEncoder.encode(file.getName(), "UTF-8"));  

        } else {  
            response.setContentType("text/html;charset=utf-8");  
            response.setHeader("Content-type", "text/html;charset=UTF-8");  
            out.write("暂不支持该格式".getBytes("UTF-8"));  
            return;  
        }  

        try {  
            //获取要下载的文件输入流  
            in = new FileInputStream(outFileName);  
            int len = 0;  
            //创建数据缓冲区    
            byte[] buffer = new byte[1024];  
            //通过response对象获取OutputStream流  
            //将FileInputStream流写入到buffer缓冲区  
            while ((len = in.read(buffer)) > 0) {  
                //使用OutputStream将缓冲区的数据输出到客户端浏览器  
                out.write(buffer, 0, len);  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
            response.setContentType("text/html;charset=utf-8");  
            response.setHeader("Content-type", "text/html;charset=UTF-8");  
            out.write("无法找到该文件".getBytes("UTF-8"));  
        } finally {  
            in.close();  
        }  
    }  
```