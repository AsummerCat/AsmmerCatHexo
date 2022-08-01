---
title: 数据库解决emoji表情无法添加
date: 2018-12-27 14:15:06
tags: [数据库,mysql]
---

## 这边有两种选择方式

* 1.使用String 过滤出特殊的表情   (两种方式 1.自定义工具类过滤 2.导入工具包)
* 2.使用修改字符集的方法 使数据库支持utf8mb4 存入emoji表情 

<!--more-->

# 修改字符集方式

##  修改单表字符集

> ALTER TABLE user CONVERT TO CHARACTER SET utf8mb4

## 查看当前库的所有表

> select table_name from information_schema.`TABLES` where TABLE_SCHEMA = '库名';

## 数据库字符集修改

> ps :由于 uft8 不能不能保存emjo 表情 会报错 所以修改字符集为 utf8mb4
>
> 需要注意的是 如果 `Illegal mix of collations (utf8_general_ci,IMPLICIT) and (utf8mb4_general_ci,COERCIBLE) for operation ‘=‘` 错误：连接字符集使用utf8mb4，但 SELECT/UPDATE where条件有utf8类型的列，且条件右边存在不属于utf8字符，就会触发该异常。表示踩过这个坑。



---



# 使用emoji-java 过滤emoji表情



## 使用github上一个开源的过滤emoji表情的工具

```
<dependency>
   <groupId>com.vdurmont</groupId>
   <artifactId>emoji-java</artifactId>
   <version>4.0.0</version>
  </dependency>
```

## github地址: [emoji-java](https://github.com/vdurmont/emoji-java)

#### 删除表情符号

您可以使用以下方法之一轻松地从字符串中删除表情符号：

- `EmojiParser#removeAllEmojis(String)`：从String中删除所有表情符号
- `EmojiParser#removeAllEmojisExcept(String, Collection<Emoji>)`：从String中删除所有表情符号，但Collection中的表情符号除外
- `EmojiParser#removeEmojis(String, Collection<Emoji>)`：从String中删除Collection中的表情符号



# 使用自定义工具类 过滤emoji表情





```
/**
     * 检测是否有emoji字符
     * @param source
     * @return 一旦含有就抛出
     */
    public static boolean containsEmoji(String source) {
        if (StringUtils.isBlank(source)) {
            return false;
        }

        int len = source.length();

        for (int i = 0; i < len; i++) {
            char codePoint = source.charAt(i);

            if (isEmojiCharacter(codePoint)) {
                //do nothing，判断到了这里表明，确认有表情字符
                return true;
            }
        }

        return false;
    }

    private static boolean isEmojiCharacter(char codePoint) {
        return (codePoint == 0x0) ||
                (codePoint == 0x9) ||
                (codePoint == 0xA) ||
                (codePoint == 0xD) ||
                ((codePoint >= 0x20) && (codePoint <= 0xD7FF)) ||
                ((codePoint >= 0xE000) && (codePoint <= 0xFFFD)) ||
                ((codePoint >= 0x10000) && (codePoint <= 0x10FFFF));
    }

    /**
     * 过滤emoji 或者 其他非文字类型的字符
     * @param source
     * @return
     */
    public static String filterEmoji(String source) {

        if (!containsEmoji(source)) {
            return source;//如果不包含，直接返回
        }
        //到这里铁定包含
        StringBuffer buf = null;

        int len = source.length();

        for (int i = 0; i < len; i++) {
            char codePoint = source.charAt(i);

            //如果是特殊不是符号  进行追加操作
            if (!isEmojiCharacter(codePoint)) {
                 if (buf == null) {
                    buf = new StringBuffer(source.length());
                }
                buf.append(codePoint);
            }
        }
		
        //运算完毕 如果buf==null表示 全是特殊
        if (buf == null) {
            return ''; 
        } else {
            //如果buf的长度=原字符串长度 直接返回原字符串
            if (buf.length() == len) {
                buf = null;
                return source;
            } else {
                //如果buf!=原字符串长度 说明进行处理过了特殊字符了
                return buf.toString();
            }
        }

    }
```



# 结尾



> 使用后两种方式需要修改业务代码 个人觉得 在数据库设计的过程中 指定字符集为 utf8mb4 这种方式 更好