---
title: 使用Thumbnailator工具压缩图片
date: 2018-09-20 22:02:14
tags: [工具类]
---
### 使用Thumbnailator工具需要引入thumbnailator-0.4.8.jar包，在pom中添加一下代码即可。
<!--more-->
```
<dependency>
     <groupId>net.coobird</groupId>
     <artifactId>thumbnailator</artifactId>
     <version>0.4.8</version>
</dependency>
```



> 可以參考文章 ：https://blog.csdn.net/qiaqia609/article/details/53171149

### TEST.java

```
import net.coobird.thumbnailator.Thumbnails;
 
import java.io.File;
import java.io.IOException;
 
public class Test {
    public static void main(String[] args) throws IOException {
        Thumbnails.of(new File("C:\\Users\\Administrator\\Pictures\\IMG_368845.jpg"))
                .size(160, 160)
                .toFile(new File("C:\\Users\\Administrator\\Pictures\\IMG_368845_min.jpg"));
    }
}
```



例子:

### 从图像文件创建缩略图

```
Thumbnails.of(new File("original.jpg"))
        .size(160, 160)
        .toFile(new File("thumbnail.jpg"));
```

在此示例中，图像来自`original.jpg`调整大小，然后保存到`thumbnail.jpg`。

或者，*Thumbnailator*将接受文件名作为`String`。`File`不需要使用对象指定图像文件：

```
Thumbnails.of("original.jpg")
        .size(160, 160)
        .toFile("thumbnail.jpg");
```

在编写快速原型代码或从脚本语言中使用*Thumbnailator*时，此表单非常有用。

### 使用旋转和水印创建缩略图

```
Thumbnails.of(new File("original.jpg"))
        .size(160, 160)
        .rotate(90)
        .watermark(Positions.BOTTOM_RIGHT, ImageIO.read(new File("watermark.png")), 0.5f)
        .outputQuality(0.8)
        .toFile(new File("image-with-watermark.jpg"));
```

在此示例中，`original.jpg`调整图像大小，然后顺时针旋转90度，然后在右下角放置一个半透明水印，然后`image-with-watermark.jpg`以80％压缩质量设置保存。

### 创建缩略图并写入 `OutputStream`

```
OutputStream os = ...;
		
Thumbnails.of("large-picture.jpg")
        .size(200, 200)
        .outputFormat("png")
        .toOutputStream(os);
```

在此示例中，将文件中的图像`large-picture.jpg`调整为最大尺寸200 x 200（保持原始图像的纵横比），并将其写入指定`OutputStream`的PNG图像。

### 创建固定大小的缩略图

```
BufferedImage originalImage = ImageIO.read(new File("original.png"));

BufferedImage thumbnail = Thumbnails.of(originalImage)
        .size(200, 200)
        .asBufferedImage();
```

上面的代码将图像带入`originalImage`并创建一个200像素乘200像素的缩略图，并将结果存储在其中`thumbnail`。

### 按给定因子缩放图像

```
BufferedImage originalImage = ImageIO.read(new File("original.png"));

BufferedImage thumbnail = Thumbnails.of(originalImage)
        .scale(0.25)
        .asBufferedImage();
```

上面的代码将图像`originalImage`带入并创建一个缩略图，该缩略图是原始图像的25％，并使用默认缩放技术来制作存储在其中的缩略图`thumbnail`。

### 创建缩略图时旋转图像

```
BufferedImage originalImage = ImageIO.read(new File("original.jpg"));

BufferedImage thumbnail = Thumbnails.of(originalImage)
        .size(200, 200)
        .rotate(90)
        .asBufferedImage();
```

上面的代码采用原始图像并创建一个顺时针旋转90度的缩略图。

### 使用水印创建缩略图

```
BufferedImage originalImage = ImageIO.read(new File("original.jpg"));
BufferedImage watermarkImage = ImageIO.read(new File("watermark.png"));

BufferedImage thumbnail = Thumbnails.of(originalImage)
        .size(200, 200)
        .watermark(Positions.BOTTOM_RIGHT, watermarkImage, 0.5f)
        .asBufferedImage();
```

如图所示，可以通过调用该`watermark`方法将水印添加到缩略图。

可以从[`Positions`](http://thumbnailator.googlecode.com/hg/javadoc/net/coobird/thumbnailator/geometry/Positions.html)枚举中选择定位。

缩略图的不透明度（或相反地，透明度）可以通过改变最后一个参数来调整，其中`0.0f`缩略图是完全透明的，并且`1.0f`水印是完全不透明的。

### 将缩略图写入特定目录

```
File destinationDir = new File("path/to/output");

Thumbnails.of("apple.jpg", "banana.jpg", "cherry.jpg")
        .size(200, 200)
        .toFiles(destinationDir, Rename.PREFIX_DOT_THUMBNAIL);
```

此示例将获取源图像，并将缩略图作为文件写入`destinationDir`（`path/to/output`目录），同时将其重命名`thumbnail.`为文件名。

因此，缩略图将被写为以下文件：

- `path/to/output/thumbnail.apple.jpg`
- `path/to/output/thumbnail.banana.jpg`
- `path/to/output/thumbnail.cherry.jpg`

写入指定目录时也可以保留原始文件名：

```
File destinationDir = new File("path/to/output");

Thumbnails.of("apple.jpg", "banana.jpg", "cherry.jpg")
        .size(200, 200)
        .toFiles(destinationDir, Rename.NO_CHANGE);
```

在上面的代码中，缩略图将写入：

- `path/to/output/apple.jpg`
- `path/to/output/banana.jpg`
- `path/to/output/cherry.jpg`