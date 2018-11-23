---
title: iText导出pdf
date: 2018-11-23 15:18:25
tags: 工具类
---
# [demo地址](https://github.com/AsummerCat/itextdemo)



# iText
是用于生成PDF文档的一个java类库。通过iText不仅可以生成PDF或rtf的文档，而且可以将XML、Html文件转化为PDF文件。 

# 导入依赖

```
   <!--iext核心包 -->
        <dependency>
            <groupId>com.itextpdf</groupId>
            <artifactId>itextpdf</artifactId>
            <version>5.5.13</version>
        </dependency>

        <!--转换xml到pdf 用来解析html -->
        <dependency>
            <groupId>com.itextpdf.tool</groupId>
            <artifactId>xmlworker</artifactId>
            <version>5.5.13</version>
        </dependency>
    </dependencies>
```

<!--more-->

需要注意的是: 

* 默认不支持中文字体

所以要导入

```
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itext-asian</artifactId>
    <version>5.2.0</version>
</dependency>
```
<font color="red">但是呢 因为这个包很久没有更新了 itext核心包中的相关位置已经改变了 直接导入itext-asian是无法使用的</font>

* 方案一:

 ```
 下载itext-asian.jar 解压 改变位置 然后导入
 相关信息暂不介绍
 
 ```
 ```
 //中文字体
			BaseFont bfChinese = BaseFont.createFont( "STSongStd-Light" ,"UniGB-UCS2-H",BaseFont.NOT_EMBEDDED);
			Font font = new Font(bfChinese, 12,Font.NORMAL);
			PdfPCell cell = new PdfPCell(new Paragraph("测试",bigHoldFont));

 ```

* 方案二:

```
直接使用 自己下载的中文字体
本文方案就是下载自定义字体
```

---


# 其他需要了解的知识

* 根目录地址  

```
 String uploadPath = this.getClass().getClassLoader().getResource("").getPath();
 
 /Users/macpro/Demo/itextdemo/target/classes/
```

* 响应消息

```
//返回消息的类型
 response.setContentType("application/pdf");
 //这里表示用下载的方式
 response.setHeader("Content-Disposition", "attachment;filename=测试.pdf");
```

* 文件分隔符(不分系统)

```
File.separator
```

# 首先呢?要解决中文乱码/不显示问题

* 为什么要实现这个接口呢 因为后面使用tool的XMLWorkerHelper导出文件的时候需要有这个类
* 如果你不需要使用html导出 或者使用其他方式导出  那么可以直接使用下面这个类中的`getChineseFont()`这个方法就可以了

```
package com.linjingc.itext.util;

import com.itextpdf.text.BaseColor;
import com.itextpdf.text.Font;
import com.itextpdf.text.FontProvider;
import com.itextpdf.text.pdf.BaseFont;
import org.springframework.stereotype.Component;

import java.io.File;

/**
 * 解决itext中文不显示
 * html版本
 *
 * @author cxc
 * @date 2018/11/23 01:39
 */
@Component
public class ChineseFontUtil implements FontProvider {
    @Override
    public Font getFont(String s, String s1, boolean b, float v, int i, BaseColor baseColor) {
        return getChineseFont();
    }

    @Override
    public boolean isRegistered(String s) {
        return false;
    }

    public Font getChineseFont() {
        //获取文件根地址
        String uploadPath = this.getClass().getClassLoader().getResource("").getPath() + File.separator;
        //获取字体解决中文乱码问题
        BaseFont baseFont = null;
        String fontPath = uploadPath + "static" + File.separator + "msyh.ttf";
        try {
            baseFont = BaseFont.createFont(fontPath, BaseFont.IDENTITY_H, BaseFont.NOT_EMBEDDED);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return new Font(baseFont);
    }
}

```



# 简单生成pdf

* 写入内容   `document.add(new Paragraph("Hello World哈哈哈", fontChinese));`
*  `new Paragraph():这里表示输入的内容     fontChinese:这个是字体`

```
package com.linjingc.itext.service;

import com.itextpdf.text.*;
import com.itextpdf.text.pdf.PdfWriter;
import com.linjingc.itext.util.ChineseFontUtil;
import lombok.extern.log4j.Log4j2;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * itext测试demo
 *
 * @author cxc
 * @date 2018/11/22 23:29
 */
@Service
@Log4j2
public class HelloService {

    @Autowired
    private ChineseFontUtil chineseFontUtil;

    /**
     * 测试导出Pdf
     */
    public void exportPdf() {

        //创建一个临时目录接收
        String uploadPath = this.getClass().getClassLoader().getResource("").getPath() + File.separator;

        try {
            //1、创建文档对象实例
            Document document = new Document();

            //获取字体解决中文乱码问题
            Font fontChinese = chineseFontUtil.getChineseFont();

            //页面大小
            Rectangle rect = new Rectangle(PageSize.B4.rotate());
            //页面背景色
            rect.setBackgroundColor(BaseColor.ORANGE);

            //创建一个pdf
            PdfWriter.getInstance(document, new FileOutputStream(uploadPath + "一只写Bug的猫普通的导出Pdf.pdf"));
            //打开pdf
            document.open();
            //写入内容
            for (int i = 0; i < 5; i++) {
                document.add(new Paragraph("Hello World哈哈哈", fontChinese));
            }
            //关闭pdf
            document.close();

        } catch (IOException | DocumentException e) {
            log.info("导出错误{}", e);
        }
    }

}

```


# 使用字节流 把html的内容 直接导出pdf

##  html的内容

```
   /**
     * 拼写html字符串代码
     */
    private static String getHtml() {
        StringBuffer html = new StringBuffer();
        html.append("<div>一只写Bug的猫</div>");
        html.append("<div>一只写Bug的猫</div>");
        html.append("<div>一只写Bug的猫</div>");
        html.append("<div>一只写Bug的猫</div>");
        html.append("<div><img src='http://img.chuansong.me/mmbiz_jpg/qIsbibEfba7ibBibLMkia5ia3CR6nSGMwCq70mDeasBePwdmg8G4icOMuiblKOFZlLHOric5oCnX361k0cibibfbd9K7yUQA/0?wx_fmt=jpeg'/></div>");
        return html.toString();
    }
```

## 方法

```
 /**
     * 页面直接写入输出
     */
    public void exportHtmlToPdf() {

        //创建一个临时目录接收
        String uploadPath = this.getClass().getClassLoader().getResource("").getPath() + File.separator;

        try {
            //1、创建文档对象实例
            Document document = new Document();
            //页面大小
            Rectangle rect = new Rectangle(PageSize.B4.rotate());
            //页面背景色
            rect.setBackgroundColor(BaseColor.ORANGE);

            //创建一个pdf
            PdfWriter pdfWriter = PdfWriter.getInstance(document, new FileOutputStream(uploadPath + "一只写Bug的猫普通的导出htmlToPdf.pdf"));
            //打开pdf
            document.open();
            //写入内容
            //模板内容
            String html = getHtml();
            ByteArrayInputStream bin = new ByteArrayInputStream(html.getBytes("UTF-8"));
            XMLWorkerHelper.getInstance().parseXHtml(pdfWriter, document, bin, null, Charset.forName("UTF-8"), chineseFontUtil);
            //关闭pdf
            document.close();

        } catch (IOException e) {
            log.info("io异常");
        } catch (DocumentException e) {

            log.info("iText异常{}", e);
        }
    }
```

# html模板导出pdf

## templatesToPdf.html

```
<body>
<div>测试导出Pdf</div>
<div><img src='http://img.chuansong.me/mmbiz_jpg/qIsbibEfba7ibBibLMkia5ia3CR6nSGMwCq70mDeasBePwdmg8G4icOMuiblKOFZlLHOric5oCnX361k0cibibfbd9K7yUQA/0?wx_fmt=jpeg'/></div>
</body>
```
## 获取缓存流中的html模板

```
 /**
     * 字节缓冲流读取模板文件
     */
    private BufferedInputStream getHtmlTemplates() {
        //获取文件根地址
        String uploadPath = this.getClass().getClassLoader().getResource("").getPath() + File.separator;
        BufferedInputStream bufferedInputStream = null;
        String resPath = "templates" + File.separator + "templatesToPdf.html";
        try {
            bufferedInputStream = new BufferedInputStream(new FileInputStream(uploadPath + resPath));
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        return bufferedInputStream;
    }
```

##  导出方法

```
 /**
     * html模板导出PDF
     */
    public void exportHtmlTemplateToPdf() throws IOException {
        BufferedInputStream htmlTemplates = null;
        //创建一个临时目录接收
        String uploadPath = this.getClass().getClassLoader().getResource("").getPath() + File.separator;

        try {
            //1、创建文档对象实例
            Document document = new Document();
            //页面大小
            Rectangle rect = new Rectangle(PageSize.B4.rotate());
            //页面背景色
            rect.setBackgroundColor(BaseColor.ORANGE);

            //创建一个pdf
            PdfWriter pdfWriter = PdfWriter.getInstance(document, new FileOutputStream(uploadPath + "一只写Bug的猫html模板导出Pdf.pdf"));
            //打开pdf
            document.open();
            //写入内容
            //模板内容
            htmlTemplates = getHtmlTemplates();
            XMLWorkerHelper.getInstance().parseXHtml(pdfWriter, document, htmlTemplates, null, Charset.forName("UTF-8"), chineseFontUtil);

            //关闭pdf
            document.close();

        } catch (IOException e) {
            log.info("io异常");
        } catch (DocumentException e) {

            log.info("iText异常{}", e);
        } finally {
            if (htmlTemplates != null) {
                htmlTemplates.close();
            }
        }
    }
```

# Html模板页面填充内容输出PDF

* 首先呢  如何做标记 渲染?
* 原本是想有freemark做模板的
* 然后抱着学习的想法 直接使用html直接用占位符 自定义方法 解析占位符

## 工具类

* 这里主要用正则表达式抓取出字符串中 我们使用占位符中的标记 然后用map的key 对应做渲染

```
package com.linjingc.itext.util;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * 用来字符中查找占位符的内容
 *
 * @author cxc
 * @date 2018/11/23 10:44
 */
public class OffestUtil {
    /**
     * 获取占位符中的内容
     *
     * @param str
     * @return
     */
    public static List<String> match(String str) {
        List<String> results = new ArrayList<String>();
        Pattern p = Pattern.compile("\\{([\\w]*)\\}");
        Matcher m = p.matcher(str);
        while (m.find()) {
            results.add(m.group(1));
        }
        return results;
    }

    /**
     * 将String中需要填充的占位符 用Map填充
     *
     * @return
     */
    public static String fullStringByMap(String str, Map map) {
        String res = str;
        List<String> list = match(str);
        for (String data : list) {
            res = res.replace("{" + data + "}", (String) map.get(data));
        }
        return res;
    }


    //测试
    public static void main(String[] args) {
        String req = "adfwe{abc}defg{def}gju{ght}dfdf";
        List<String> list = match(req);
        for (String res : list) {
            System.out.println(res);
        }
    }
}


```


## 根据字符流读取文件 一行行填充内容

* 这样的目的就是 可以拿到模板的每一行数据 
* 然后经过上面的工具类处理下 就可以把标记的内容 替换为我们需要的数据了

```
 /**
     * 根据字符流读取文件 一行行填充内容
     *
     * @return
     * @throws IOException
     */
    private String getHtmlTemplatesFill(Map map) throws IOException {

        //获取文件根地址
        String uploadPath = this.getClass().getClassLoader().getResource("").getPath() + File.separator;
        BufferedReader br = null;
        StringBuffer str = new StringBuffer();

        String resPath = "templates" + File.separator + "templatesFullToPdf.html";
        try {
            br = new BufferedReader(new InputStreamReader(new FileInputStream(uploadPath + resPath), "UTF-8"), 60 * 1024 * 1024);
            String lineTxt;
            while ((lineTxt = br.readLine()) != null) {
                //用来获取是否有占位符的内容
                String fullString = OffestUtil.fullStringByMap(lineTxt, map);
                str.append(fullString);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (br != null) {
                br.close();
            }
        }
        return str.toString();
    }
```

## 导出方法

```
 /**
     * 页面填充内容输出PDF
     */
    public void exportHtmlFillToPdf() {

        //创建一个临时目录接收
        String uploadPath = this.getClass().getClassLoader().getResource("").getPath() + File.separator;

        try {
            //1、创建文档对象实例
            Document document = new Document();
            //页面大小
            Rectangle rect = new Rectangle(PageSize.B4.rotate());
            //页面背景色
            rect.setBackgroundColor(BaseColor.ORANGE);

            //创建一个pdf
            PdfWriter pdfWriter = PdfWriter.getInstance(document, new FileOutputStream(uploadPath + "一只写Bug的猫普通的导出html模板Pdf.pdf"));
            //打开pdf
            document.open();
            //写入内容
            Map<String, Object> map = new HashMap<String, Object>(4);
            map.put("1", "哈哈哈");
            //模板内容
            String html = getHtmlTemplatesFill(map);
            ByteArrayInputStream bin = new ByteArrayInputStream(html.getBytes("UTF-8"));
            XMLWorkerHelper.getInstance().parseXHtml(pdfWriter, document, bin, null, Charset.forName("UTF-8"), chineseFontUtil);
            //关闭pdf
            document.close();

        } catch (IOException | DocumentException e) {
            log.info("iText异常{}", e);
        }
    }
```


# 页面填充内容输出PDF 导出到浏览器

* 这里就没上面东西可以介绍了
* 主要核心点就是 生成pdf文件那边 输出response的响应流中;


```
//创建一个pdf 输出到浏览器
            response.setContentType("application/pdf");
            //这里表示直接返回下载
            //response.setHeader("Content-Disposition", "attachment;filename=测试.pdf");
            
            //生成文件
            PdfWriter pdfWriter = PdfWriter.getInstance(document, response.getOutputStream());
```

## 完整导出方法

```
 /**
     * 页面填充内容输出PDF 导出到浏览器
     */
    public void exportHtmlFillPdfToBrowser(HttpServletResponse response) {
        try {
            //1、创建文档对象实例
            Document document = new Document();
            //页面大小
            Rectangle rect = new Rectangle(PageSize.B4.rotate());
            //页面背景色
            rect.setBackgroundColor(BaseColor.ORANGE);

            //创建一个pdf 输出到浏览器
            response.setContentType("application/pdf");
            //这里表示直接返回下载
            //response.setHeader("Content-Disposition", "attachment;filename=测试.pdf");

            //生成文件
            PdfWriter pdfWriter = PdfWriter.getInstance(document, response.getOutputStream());
            //打开pdf
            document.open();
            //写入内容
            Map<String, Object> map = new HashMap<String, Object>(4);
            map.put("1", "哈哈哈");
            //模板内容
            String html = getHtmlTemplatesFill(map);
            ByteArrayInputStream bin = new ByteArrayInputStream(html.getBytes("UTF-8"));
            XMLWorkerHelper.getInstance().parseXHtml(pdfWriter, document, bin, null, Charset.forName("UTF-8"), chineseFontUtil);

            //关闭pdf
            document.close();

        } catch (IOException | DocumentException e) {
            log.info("iText异常{}", e);
        }
    }


```