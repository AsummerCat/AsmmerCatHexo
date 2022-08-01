---
title: Solr基本语法
date: 2019-03-05 20:12:44
tags: [solr]
---

# Solr基本语法

[Apache Solr入门教程(初学者之旅)](https://www.cnblogs.com/cblogs/p/solr-tutorial.html)
[手把手教你如何玩转Solr（包含项目实战）](https://blog.csdn.net/cs_hnu_scw/article/details/79388080)
[solr初探-安装使用](http://www.kailing.pub/article/index/arcid/149.html)



<!--more-->

# 语法常用

- q - 查询字符串，这个是必须的。如果查询所有*:* ，根据指定字段查询（Name:张三 AND Address:北京）`  
- `fq - （filter query）过虑查询，作用：在q查询符合结果中同时是fq查询符合的，例如：q=Name:张三&fq=CreateDate:[20081001 TO 20091031],找关键字mm，并且CreateDate是20081001`
- ` start - 返回第一条记录在完整找到结果中的偏移位置，0开始，一般分页用。 `  
- ` rows - 指定返回结果最多有多少条记录，配合start来实现分页。 ` 
- ` sort - 排序，格式：sort=<field name>+<desc|asc>[,<field name>+<desc|asc>]… 。示例：（score desc, price asc）表示先 “score” 降序, 再 “price” 升序，默认是相关性降序。 `  
- `wt - (writer type)指定输出格式，可以有 xml, json, php, phps。`
- `fl表示索引显示那些field( *表示所有field,如果想查询指定字段用逗号或空格隔开（如：Name,SKU,ShortDescription或Name SKU ShortDescription【注：字段是严格区分大小写的】）) `

# 基础查询

1. 最普通的查询，比如查询姓张的人（ Name:张）,如果是精准性搜索相当于SQL SERVER中的LIKE搜索这需要带引号（""）,比如查询含有北京的（Address:"北京"） 
2. 多条件查询，注：如果是针对单个字段进行搜索的可以用（Name:搜索条件加运算符(OR、AND、NOT) Name：搜索条件）,比如模糊查询（ Name:张 OR Name:李 ）单个字段多条件搜索不建议这样写，一般建议是在单个字段里进行条件筛选，如（ Name:张 OR 李），多个字段查询（Name:张 + Address:北京 ）

# 查询语法

### 精确查找

```script
 q= 为查询  name:精确查找  
 http://localhost:8983/solr/jcg/select?q=name:A Clash of Kings
```

# 实例

## 查询所有

```script
http://localhost:8080/solr/primary/select?q=*:*
```

### 限定返回字段

```script
http://localhost:8080/solr/primary/select?q=*:*&fl=productId  
表示：查询所有记录，只返回productId字段
```

### 分页

```
http://localhost:8080/solr/primary/select?q=*:*&fl=productId&rows=6&start=0  
表示：查询前六条记录，只返回productId字段
```

### 增加限定条件

```script
http://localhost:8080/solr/primary/select?q=*:*&fl=productId&rows=6&start=0&fq=category:2002&fq=namespace:d&fl=productId+category&fq=en_US_city_i:1101  
表示：查询category=2002、en_US_city_i=110以及namespace=d的前六条记录，只返回productId和category字段
```

### 添加排序

```
http://localhost:8080/solr/primary/select?q=*:*&fl=productId&rows=6&start=0&fq=category:2002&fq=namespace:d&sort=category_2002_sort_i+asc  
表示：查询category=2002以及namespace=d并按category_2002_sort_i升序排序的前六条记录，只返回productId字段
```

# java 使用

## Solr进行数据的增删改查的处理

```
package com.hnu.scw.solr;
import java.io.IOException;
import java.util.List;
import java.util.Map;
import org.apache.solr.client.solrj.SolrQuery;
import org.apache.solr.client.solrj.SolrServer;
import org.apache.solr.client.solrj.SolrServerException;
import org.apache.solr.client.solrj.SolrQuery.ORDER;
import org.apache.solr.client.solrj.impl.HttpSolrServer;
import org.apache.solr.client.solrj.response.QueryResponse;
import org.apache.solr.common.SolrDocument;
import org.apache.solr.common.SolrDocumentList;
import org.apache.solr.common.SolrInputDocument;
import org.junit.Test;
/**
 * solrj的相关开发
 * @author scw
 *2018-02-26
 */
public class SolrManager {
    /**
	 * 添加文档数据到solr服务器中
	 * @throws Exception
	 */
    @Test
    public void addContent() throws Exception{
        //设置solr服务器的路径，默认是使用第一个collection库，所以路径最后可以不加collection1
        String baseURL = "http://localhost:8080/solr";
        //如果要使用第二个collection库，那么就使用下面的链接
        //String baseURL2 = "http://localhost:8080/solr/collection2";		
        //创建服务器连接对象
        HttpSolrServer httpSolrServer = new HttpSolrServer(baseURL);
        //创建新的文档对象
        SolrInputDocument solrInputDocument = new SolrInputDocument();
        //设置文档的域
        solrInputDocument.setField("id", "haha222");
        solrInputDocument.setField("name", "佘超伟123");
        //进行添加
        httpSolrServer.add(solrInputDocument);
        //进行手动提交，否则无法进行添加
        httpSolrServer.commit();
    }
    /**
     * 进行删除文档操作
     * @throws SolrServerException
     * @throws IOException
     */
    @Test
    public void deleteContent() throws Exception{
        String baseURL = "http://localhost:8080/solr";
        SolrServer httpSolrServer = new HttpSolrServer(baseURL);
        //删除全部，第一个参数是设置需要删除的数据的域和值，第二个是执行后多久进行删除操作
        //httpSolrServer.deleteByQuery("*:*",1000);
        //删除某个特定域的特定值的数据
        httpSolrServer.deleteByQuery("id:haha",1000);
    }

    /**
	 * 修改文档内容
	 * 修改其实和添加是一样的，因为只要添加的ID是一样的，那么就会把原来的删除了，然后再添加一个
	 * @throws IOException 
	 * @throws SolrServerException 
     */
    @Test
    public void updateContent() throws SolrServerException, IOException{
        String baseURL = "http://localhost:8080/solr";
        SolrServer httpSolrServer = new HttpSolrServer(baseURL);
        //创建新的文档对象
        SolrInputDocument solrInputDocument = new SolrInputDocument();
        //设置文档的域
        solrInputDocument.setField("id", "haha123");
        solrInputDocument.setField("name", "哈哈123");
        httpSolrServer.add(solrInputDocument);
    }
     
    /**
	 * 查询数据（多功能的显示处理）
	 * @throws Exception 
     */
    @Test
    public void queryContent() throws Exception{
        String baseURL = "http://localhost:8080/solr";
        SolrServer httpSolrServer = new HttpSolrServer(baseURL);
        //创建查询数据对象（便于设置查询条件）
        SolrQuery solrQuery = new SolrQuery();
        //设置查询的域和值，这个在之后的项目中可以用于动态
        //方法一：参数q就代表query查询
        //solrQuery.set("q","name:佘超伟123");
        //方法二：(一般使用该方法)
        solrQuery.setQuery("name:佘超伟");
        //方法三：通过设置默认域
        //solrQuery.set("df", "name");
        //solrQuery.setQuery("佘超伟");
        //设置查询过滤条件(可以设置多个，只要域和值有改变就可以了)
        //solrQuery.set("fq", "id:haha123");
        //添加排序方式（可选内容）
        //solrQuery.addSort("需要排序的域",ORDER.asc);//升序
        //solrQuery.addSort("需要排序的域",ORDER.desc);//降序
        //设置分页处理(比如这是设置每次显示5个)
        solrQuery.setStart(0);
        solrQuery.setRows(5);
        //设置只查询显示指定的域和值(第二个参数可以是多个，之间用“逗号”分割)
        //solrQuery.set("fl", "name");
        //设置某域进行高亮显示
        solrQuery.setHighlight(true);
        solrQuery.addHighlightField("name");
        //设置高亮显示格式的前后缀
        solrQuery.setHighlightSimplePre("<span style='color:red'>");
        solrQuery.setHighlightSimplePost("</span");	
        
        //执行查询，获得查询结果对象
        QueryResponse query = httpSolrServer.query(solrQuery);
        //获取查询的结果集
        SolrDocumentList results = query.getResults();
        //获取高亮显示的查询结果
        //注意点：因为高亮的结果和正常的查询结果是不一样的，所以要进行特别的处理
        Map<String, Map<String, List<String>>> highlighting = query.getHighlighting();
        //遍历结果集
        for (SolrDocument solrDocument : results) {
            String idStr = (String) solrDocument.get("id");
            System.out.println("id----------------" + idStr);
            String nameStr = (String) solrDocument.get("name");
            System.out.println("name----------------" + nameStr);
            System.out.println("===========高亮显示=====================");
            Map<String, List<String>> map = highlighting.get(idStr);
            List<String> list = map.get("name");
            String resultString = list.get(0);
            System.out.println("高亮结果为：-----" + resultString);
        }		
    }
}
```

