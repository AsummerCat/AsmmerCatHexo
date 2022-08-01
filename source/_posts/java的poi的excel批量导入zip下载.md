---
title: java的poi的excel批量导入zip下载
date: 2019-03-05 20:09:54
tags: [java]
---

# java的poi的excel批量导入zip下载

# 代码

```
//todo 手动导出  
        try {  
            File zipFile = new File(request.getSession().getServletContext  ().getRealPath("") + File.separator + "demo.zip");  
            ZipOutputStream zipOut = null; // 声明压缩流对象  
            zipOut = new ZipOutputStream(new FileOutputStream(zipFile));  
            zipOut.setComment("");  // 设置注释  
            //zip压缩打包  
            List<List<Map<String, Object>>> lists = ListSplitUtils.splitList(list, 10000);  
            for (int i = 0; i < lists.size(); i++) {  
                List<Map<String, Object>> twoList = new ArrayList<Map<String, Object>>();  
                twoList.addAll(lists.get(i));  
                Workbook wb = ExcelExportUtil.exportExcel(params, entityList, twoList);  
                zipOut.putNextEntry(new ZipEntry("demo" + i + ".xlsx"));  
                wb.write(zipOut);  
                zipOut.flush();  
            }  
            zipOut.close();  
            // 以流的形式下载文件。  
            BufferedInputStream fis = new BufferedInputStream(new FileInputStream(zipFile.getPath()));  
            byte[] buffer = new byte[fis.available()];  
            fis.read(buffer);  
            fis.close();  
            // 清空response  
            response.reset();  
            OutputStream toClient = new BufferedOutputStream  (response.getOutputStream());  
            response.setContentType("application/octet-stream");  
            if (zipFile.getName() == null || zipFile.getName() == "") {  
                response.setHeader("Content-Disposition", "attachment;filename=" + new String(zipFile.getName().getBytes("UTF-8"), "ISO-8859-1"));  
            } else {  
                response.setHeader("Content-Disposition", "attachment;filename=" + new String(zipFile.getName().getBytes("UTF-8"), "ISO-8859-1"));  
            }
            toClient.write(buffer);  
            toClient.flush();  
            toClient.close();  
            zipFile.delete();        //是否将生成的服务器端文件删除   
        } catch (IOException ex) {  
            ex.printStackTrace();  
        }  

```

<!--more-->

## 工具类

```
/**
 * 集合工具类
 */
public class ListSplitUtils {  
    //分片  
    public static <T> List<List<T>> splitList(List<T> list, int blockSize) {  
        List<List<T>> lists = new ArrayList<List<T>>();  
        if (blockSize == 1) {  
            lists.add(list);  
            return lists;  
        }  
        if (list != null && blockSize > 0) {  
            int listSize = list.size();  
            if (listSize <= blockSize) {  
                lists.add(list);  
                return lists;  
            }  
            int batchSize = listSize / blockSize;  
            int remain = listSize % blockSize;   
            for (int i = 0; i < batchSize; i++) {  
                int fromIndex = i * blockSize;  
                int toIndex = fromIndex + blockSize;  
                System.out.println("fromIndex=" + fromIndex + ", toIndex=" + toIndex);  
                lists.add(list.subList(fromIndex, toIndex));  
            }  
            if (remain > 0) {  
                System.out.println("fromIndex=" + (listSize - remain) + ", toIndex=" + (listSize));  
                lists.add(list.subList(listSize - remain, listSize));  
            }  
        }  
        return lists;  
    }  
}  
```

