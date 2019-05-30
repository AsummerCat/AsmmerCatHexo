---
title: webp格式转换的各种方式
date: 2019-05-30 23:24:10
tags: 前端
---
# 在线转换
智图    http://zhitu.isux.us/  
又拍云：https://www.upyun.com/webphexo 

isparta ：http://isparta.github.io/

github的一个js 旧版本的IE处理: https://github.com/aFarkas/html5shiv

# 判断浏览器是否支持webp

根据header信息判断浏览器是否支持webp

accept头信息会告知浏览器能够识别哪些类型的文件


当前还有一部分客户端并不支持 WebP 格式，可以通过 CDN 层去判断，对于支持的客户端，响应 WebP 格式的图片；不支持的客户端，响应原图。从而实现无缝适配。

<!--more-->
# java

## 方式一

### 导入WEBP的jar包

webp-imageio-0.4.2

### 导入WEBP的工具类
```
public class ImageConverterWebp {

    public static final String WEBP = "webp";
    public static final String WEBP_SUFFIX = ".webp";

    private ImageConverterWebp() {
    }

    /**
     * 1. 传入图片文件路径，返回file对象
     * @param imgFilePath 图片文件路径(比如转换图片为F:/1.png 那么转换后的webp文件为：F:/1.png.webp)
     * @return
     */
    public static File toWebpFile(String imgFilePath) {
        File imgFile = new File(imgFilePath);
        File webpFile = new File(imgFilePath + WEBP_SUFFIX);
        try {
            BufferedImage bufferedImage = ImageIO.read(imgFile);
            ImageIO.write(bufferedImage, WEBP, webpFile);
        }
        catch(Exception e) {
            e.printStackTrace();
        }
        return webpFile;
    }

    /**
     * 2. 传入图片url，返回file对象
     * @param imgUrlPath 图片路径url
     * @param webpFilePath 生成的webp文件路径
     * @return
     */
    public static File toWebpFile(String imgUrlPath, String webpFilePath) {
        File webpFile = new File(webpFilePath);
        try {
            BufferedImage bufferedImage = ImageIO.read(new URL(imgUrlPath));
            ImageIO.write(bufferedImage, WEBP, webpFile);
        }
        catch(Exception e) {
            e.printStackTrace();
        }
        return webpFile;
    }

    /**
     * 3. 传入图片文件路径，返回InputStream
     * @param imgFilePath 图片文件路径(比如转换图片为F:/1.png 那么转换后的webp文件为：F:/1.png.webp)
     * @return
     */
    public static InputStream toWebpStream(String imgFilePath) {
        File imgFile = new File(imgFilePath);
        File webpFile = new File(imgFilePath + WEBP_SUFFIX);
        FileInputStream fis = null;
        try {
            BufferedImage bufferedImage = ImageIO.read(imgFile);
            ImageIO.write(bufferedImage, WEBP, webpFile);
            fis = new FileInputStream(webpFile);
        }
        catch (Exception e) {
            e.printStackTrace();
        }
        finally {
            if(fis != null){
                try {
                    fis.close();
                }
                catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return fis;
    }

    /**
     * 4. 传入图片url，返回InputStream
     * @param imgUrlPath 图片路径url
     * @param webpFilePath 生成的webp文件路径
     * @return
     */
    public static InputStream toWebpStream(String imgUrlPath, String webpFilePath) {
        File webpFile = new File(webpFilePath);
        FileInputStream fis = null;
        try {
            BufferedImage bufferedImage = ImageIO.read(new URL(imgUrlPath));
            ImageIO.write(bufferedImage, WEBP, webpFile);
            fis = new FileInputStream(webpFile);
        }
        catch (Exception e) {
            e.printStackTrace();
        }
        finally {
            if(fis != null){
                try {
                    fis.close();
                }
                catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return fis;
    }

    public static void main(String[] args) {
        String imgUrlPath = "D:\\huangqiming\\jpg\\111.jpg";
        File result = toWebpFile(imgUrlPath);
    }
}
```

### 引入dll的配置文件到JDK上
webp-imageio.dll

## 方式二(推荐 不用导入dll到jdk)

github地址: https://github.com/nintha/webp-imageio-core


### 导入pom.xml
```
<dependency>  
    <groupId>com.github.nintha</groupId>  
    <artifactId>webp-imageio-core</artifactId>  
    <version>{versoin}</version>  
    <scope>system</scope>  
    <systemPath>${project.basedir}/libs/webp-imageio-core-{version}.jar</systemPath>  
</dependency>

```

### Webp编码

```
public static void main(String args[]) throws IOException {
    String inputPngPath = "test_pic/test.png";
    String inputJpgPath = "test_pic/test.jpg";
    String outputWebpPath = "test_pic/test_.webp";

    // Obtain an image to encode from somewhere
    BufferedImage image = ImageIO.read(new File(inputJpgPath));

    // Obtain a WebP ImageWriter instance
    ImageWriter writer = ImageIO.getImageWritersByMIMEType("image/webp").next();

    // Configure encoding parameters
    WebPWriteParam writeParam = new WebPWriteParam(writer.getLocale());
    writeParam.setCompressionMode(WebPWriteParam.MODE_DEFAULT);

    // Configure the output on the ImageWriter
    writer.setOutput(new FileImageOutputStream(new File(outputWebpPath)));

    // Encode
    writer.write(null, new IIOImage(image, null, null), writeParam);
}

```


### web解码

```
public static void main(String args[]) throws IOException {
    String inputWebpPath = "test_pic/test.webp";
    String outputJpgPath = "test_pic/test_.jpg";
    String outputJpegPath = "test_pic/test_.jpeg";
    String outputPngPath = "test_pic/test_.png";

    // Obtain a WebP ImageReader instance
    ImageReader reader = ImageIO.getImageReadersByMIMEType("image/webp").next();

    // Configure decoding parameters
    WebPReadParam readParam = new WebPReadParam();
    readParam.setBypassFiltering(true);

    // Configure the input on the ImageReader
    reader.setInput(new FileImageInputStream(new File(inputWebpPath)));

    // Decode the image
    BufferedImage image = reader.read(0, readParam);

    ImageIO.write(image, "png", new File(outputPngPath));
    ImageIO.write(image, "jpg", new File(outputJpgPath));
    ImageIO.write(image, "jpeg", new File(outputJpegPath));

}
```

### 关于webp-imageio-core项目
这个项目是基于于 qwong/j-webp项目，而 qwong/j-webp 是基于 webp project of Luciad 0.4.2项目。

webp project of Luciad这个项目提供了java上一个关于处理webp的可用实现，但是它需要开发者手动java.library.path中安装对应的动态链接库，非常不方便。qwong/j-webp项目作者为了解决这个问题，改进了对动态链接库的读取方式，把从java.library.path读取改成了从项目resource文件中读取（具体内容见com.luciad.imageio.webp.WebP.loadNativeLibrary方法）。

虽然qwong/j-webp项目解决了动态链接库依赖问题，但是它的作者并未对这些代码提供一个良好封装，毕竟开发者不希望在自己项目里面直接引入第三方包的源码，所以有了webp-imageio-core提供一个可用的jar包，只要导入项目即可使用。

