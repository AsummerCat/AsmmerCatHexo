---
title: SpringBoot整合mybatis-plus实现分页功能
date: 2019-07-16 17:17:05
tags: [SpringBoot,mybatis]
---

# SpringBoot整合mybatis-plus实现分页功能

这边的话 mybatis使用的注解的

demo地址: [mybatis-plus-demo](https://github.com/AsummerCat/mybatis-demo/tree/master/mybaitis-plus-demo)

# 老规矩 导入pom

```
  <!--mybatis plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.1.2</version>
        </dependency>
```

<!--more-->

# 创建一个配置类 加载 mybatis-plus

```
/**
 * mybatis配置文件 加入mybatis-plus
 */
@Configuration
//@MapperScan("com.neo.mapper")
public class MybatisConfig {
    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

剩下都是的常规写法 

# controller Server  entity常规写法

```
/**
     * 分页查询
     * @param page 页数
     * @param pagesize 单页数量
     * @return
     */
    @RequestMapping("/pageFindUser/{page}/{pagesize}")
    @ResponseBody
    public String pageFindUser(@PathVariable Long page,@PathVariable Long pagesize) {
        Page<BasicUser> basicUserPage = new Page<>();
        //页数 0和1的效果一样 从1选择
        basicUserPage.setCurrent(page);
        //单页显示数量
        basicUserPage.setSize(pagesize);

        //排序操作
        OrderItem orderItem=new OrderItem().setAsc(true).setColumn("age");
        List<OrderItem> list=new ArrayList<OrderItem>();
        list.add(orderItem);
        basicUserPage.setOrders(list);
        List<BasicUser> basicUsers = userService.pageFindUser(basicUserPage);
        return basicUsers.toString();
    }
```

```
 /**
     * 分页查询
     */
    public List<BasicUser> pageFindUser(Page<BasicUser> page) {
        return userMapper.pageFindUser(page);
    }
```

# mybatis-plus

## dao需要继承 extends BaseMapper<T> 

需要分页的时候需要带入Page<T> page 放入第一个参数

### Page<T> page

```
 Page<BasicUser> basicUserPage = new Page<>();
        //页数 0和1的效果一样 从1选择
        basicUserPage.setCurrent(page);
        //单页显示数量
        basicUserPage.setSize(pagesize);

         //排序操作
        OrderItem orderItem=new OrderItem().setAsc(true).setColumn("age");
        List<OrderItem> list=new ArrayList<OrderItem>();
        list.add(orderItem);
        basicUserPage.setOrders(list);
```

## dao中需要传入 Page<T> page

不带入到sql中 自动分页

例如:

```
 /**
     * 分页查询
     * 分页对象,xml中可以从里面进行取值,传递参数 Page 即自动分页,必须放在第一位(你可以继承Page实现自己的分页对象)
     * @param page
     * @return
     */
    @Select("SELECT * FROM USER")
    List<BasicUser> pageFindUser(Page<BasicUser> page);
```

## 完整

```
@Mapper
public interface UserMapper extends BaseMapper<BasicUser> {
    @Select("SELECT * FROM USER WHERE NAME = #{name}")
    BasicUser findUser(String name);

    @Insert("INSERT INTO USER(NAME, PASSWORD, AGE) VALUES(#{name}, #{password}, #{age})")
    int insert(@Param("name") String name, @Param("password") String password, @Param("age") Integer age);


    /**
     * 分页查询
     * 分页对象,xml中可以从里面进行取值,传递参数 Page 即自动分页,必须放在第一位(你可以继承Page实现自己的分页对象)
     * @param page
     * @return
     */
    @Select("SELECT * FROM USER")
    List<BasicUser> pageFindUser(Page<BasicUser> page);

    /**
     * 需要注意的内容
     * #{name} 和${name}的区别    #{}代表自动拼接``  ${}表示需要手动添加``
     */
}
```

