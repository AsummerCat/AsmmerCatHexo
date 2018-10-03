----

title:  jquery.validate前端验证字段
date: 2018-09-20 22:02:14
tags: 前端

----

### 官网地址

jQuery校验官网地址：<http://bassistance.de/jquery-plugins/jquery-plugin-validation>

---

<!-- more -->

#### 需要引入JS

```html
        <script type="text/javascript" src="<%=request.getContextPath()%>/validate/jquery-1.6.2.min.js"></script>  
        <script type="text/javascript" src="<%=request.getContextPath()%>/validate/jquery.validate.min.js"></script>
        <script type="text/javascript" src="<%=request.getContextPath()%>/validate/jquery.metadata.min.js"></script> //使用class验证方式
```

---



### 基础语法:

```html
$("#signupForm").validate({
  2.         rules: {
  3.   firstname: "required",
  4.   email: {
  5.     required: true,
  6.     email: true
  7.   },
```

---

### 默认规则:

```
(1)、required:true               必输字段
(2)、remote:"remote-valid.jsp"   使用ajax方法调用remote-valid.jsp验证输入值
(3)、email:true                  必须输入正确格式的电子邮件
(4)、url:true                    必须输入正确格式的网址
(5)、date:true                   必须输入正确格式的日期，日期校验ie6出错，慎用
(6)、dateISO:true                必须输入正确格式的日期(ISO)，例如：2009-06-23，1998/01/22 只验证格式，不验证有效性
(7)、number:true                 必须输入合法的数字(负数，小数)
(8)、digits:true                 必须输入整数
(9)、creditcard:true             必须输入合法的信用卡号
(10)、equalTo:"#password"        输入值必须和#password相同
(11)、accept:                    输入拥有合法后缀名的字符串（上传文件的后缀）
(12)、maxlength:5                输入长度最多是5的字符串(汉字算一个字符)
(13)、minlength:10               输入长度最小是10的字符串(汉字算一个字符)
(14)、rangelength:[5,10]         输入长度必须介于 5 和 10 之间的字符串")(汉字算一个字符)
(15)、range:[5,10]               输入值必须介于 5 和 10 之间
(16)、max:5                      输入值不能大于5
(17)、min:10                     输入值不能小于10
```

---

### 默认的错误提示:

```html
messages: {
required: "This field is required.",
remote: "Please fix this field.",
email: "Please enter a valid email address.",
url: "Please enter a valid URL.",
date: "Please enter a valid date.",
dateISO: "Please enter a valid date (ISO).",
dateDE: "Bitte geben Sie ein g眉ltiges Datum ein.",
number: "Please enter a valid number.",
numberDE: "Bitte geben Sie eine Nummer ein.",
digits: "Please enter only digits",
creditcard: "Please enter a valid credit card number.",
equalTo: "Please enter the same value again.",
accept: "Please enter a value with a valid extension.",
maxlength: $.validator.format("Please enter no more than {0} characters."),
minlength: $.validator.format("Please enter at least {0} characters."),
rangelength: $.validator.format("Please enter a value between {0} and {1} characters long."),
range: $.validator.format("Please enter a value between {0} and {1}."),
max: $.validator.format("Please enter a value less than or equal to {0}."),
min: $.validator.format("Please enter a value greater than or equal to {0}.")
},
```

如果需要修改,可在js中写入:

```
$.extend($.validator.messages, {
    required: "必选字段",
    remote: "请修正该字段",
    email: "请输入正确格式的电子邮件",
    url: "请输入合法的网址",
    date: "请输入合法的日期",
    dateISO: "请输入合法的日期 (ISO).",
    number: "请输入合法的数字",
    digits: "只能输入整数",
    creditcard: "请输入合法的信用卡号",
    equalTo: "请再次输入相同的值",
    accept: "请输入拥有合法后缀名的字符串",
    maxlength: $.validator.format("请输入一个长度最多是 {0} 的字符串"),
    minlength: $.validator.format("请输入一个长度最少是 {0} 的字符串"),
    rangelength: $.validator.format("请输入一个长度介于 {0} 和 {1} 之间的字符串"),
    range: $.validator.format("请输入一个介于 {0} 和 {1} 之间的值"),
    max: $.validator.format("请输入一个最大为 {0} 的值"),
    min: $.validator.format("请输入一个最小为 {0} 的值")
});
```

推荐做法，将此文件放入messages_cn.js中，在页面中引入

```html
<script type="text/javascript" src="<%=path %>/validate/messages_cn.js"></script>
```



----

### 使用方式

#### 初始化：

一般来说是根据name 来确认验证

```html
初始化 并且写入js校验 
<script type="text/javascript">
        $(function(){
            var validate = $("#myform").validate({
                debug: true, //调试模式取消submit的默认提交功能   
                //errorClass: "label.error", //默认为错误的样式类为：error   
                focusInvalid: false, //当为false时，验证无效时，没有焦点响应  
                onkeyup: false,   
                submitHandler: function(form){   //表单提交句柄,为一回调函数，带一个参数：form   
                    alert("提交表单");   
                    form.submit();   //提交表单   
                },   
                
                rules:{
                    myname:{
                        required:true
                    },
                    email:{
                        required:true,
                        email:true
                    },
                    password:{
                        required:true,
                        rangelength:[3,10]
                    },
                    confirm_password:{
                        equalTo:"#password"
                    }                    
                },
                messages:{
                    myname:{
                        required:"必填"
                    },
                    email:{
                        required:"必填",
                        email:"E-Mail格式不正确"
                    },
                    password:{
                        required: "不能为空",
                        rangelength: $.format("密码最小长度:{0}, 最大长度:{1}。")
                    },
                    confirm_password:{
                        equalTo:"两次密码输入不一致"
                    }                                    
                }
                          
            });    
    
        });
        </script>
```



### 1.class方式

#### 2.将校验规则写到js代码中



----

### 异步验证

```html
remote: {
    url: "check-email.php",     //后台处理程序
    type: "post",               //数据发送方式
    dataType: "json",           //接受数据格式   
    data: {                     //要传递的数据
        username: function() {
            return $("#username").val();
        }
    }
}
远程地址只能输出 "true" 或 "false"，不能有其他输出。
```

#### 例:

前端验证字段   (1)

```html
//$("#name").focus();
            $("#inputForm").validate({
                rules: {
       title: {remote: "${ctx}/article/zjkArticle/check?oldtitle=" + encodeURIComponent('${zjkArticle.title}')}
                },
                messages: {
                    title: {remote: "标题已存在"},
                },
                submitHandler: function(form){
                    loading('正在提交，请稍等...');
                    form.submit();
                },
```

2. 前端验证字段   (2):

```

idCard:
 {remote: {
url: "${ctxapp}/checkIdCard", 
type: "Post", 
data: {oldidCard:function(){ return "${user.idCard}";},idCard:function(){ return $("#idCard").val();}}}}
```



----



### 常用方法及注意问题
#### 1、用其他方式替代默认的submit

```html
$(function(){
   $("#signupForm").validate({
        submitHandler:function(form){
            alert("submit!");   
            form.submit();
        }    
    });
});

可以设置validate的默认值，写法如下：
$.validator.setDefaults({
submitHandler: function(form) { alert("submit!"); form.submit(); }
});
```

#### 2. ignore：忽略某些元素不验证

```html
ignore: ".ignore"
```





-----



### Jquery Validate 自定义验证规则

```html
addMethod(name,method,message)方法
参数 name 是添加的方法的名字。
参数 method 是一个函数，接收三个参数 (value,element,param) 。
value 是元素的值，element 是元素本身，param 是参数。
```

#### 例子:

  #### 身份证号码验证

```
jQuery.validator.addMethod(“idcardno”, function(value, element) {
            return this.optional(element) || isIdCardNo(value);
        }, “请正确输入身份证号码”)
```

#### 字母数字

```
jQuery.validator.addMethod(“alnum”, function(value, element) {
            return this.optional(element) || /^[a-zA-Z0-9]+$/.test(value);
        }, “只能包括英文字母和数字”);
```

 #### 邮政编码验证

```
jQuery.validator.addMethod(“zipcode”, function(value, element) {
            var tel = /^[0-9]{6}$/;
            return this.optional(element) || (tel.test(value));
        }, “请正确填写邮政编码”);
```

#### 汉字

```
jQuery.validator.addMethod(“chcharacter”, function(value, element) {
            var tel = /^[u4e00-u9fa5]+$/;
            return this.optional(element) || (tel.test(value));
        }, “请输入汉字”);
```

#### 字符最小长度验证（一个中文字符长度为2）

```
jQuery.validator.addMethod(“stringMinLength”, function(value, element, param) {
            var length = value.length;
            for(var i = 0; i < value.length; i++) {
                if(value.charCodeAt(i) > 127) {
                    length++;
                }
            }
            return this.optional(element) || (length >= param);
        }, $.validator.format(“长度不能小于 { 0 }!”));
```

#### 字符验证

```
jQuery.validator.addMethod(“string”, function(value, element) {
            return this.optional(element) || /^[u0391-uFFE5w]+$/.test(value);
        }, “不允许包含特殊符号!”);
```

#### 手机号码验证

```
jQuery.validator.addMethod(“mobile”, function(value, element) {
            var length = value.length;
            return this.optional(element) || (length == 11 && /^(((13[0-9]{1})|(15[0-9]{1}))+d{8})$/.test(value));
        }, “手机号码格式错误!”);
```

#### 电话号码验证

```
jQuery.validator.addMethod(“phone”, function(value, element) {
            var tel = /^(d{3,4}-?)?d{7,9}$/g;
            return this.optional(element) || (tel.test(value));
        }, “电话号码格式错误!”);
```

#### 必须以特定字符串开头验证

```
jQuery.validator.addMethod(“begin”, function(value, element, param) {
            var begin = new RegExp(“ ^ ”+param);
            return this.optional(element) || (begin.test(value));
        }, $.validator.format(“必须以 { 0 } 开头!”));
```

 #### 验证两次输入值是否不相同

```
jQuery.validator.addMethod(“notEqualTo”, function(value, element, param) {
            return value != $(param).val();
        }, $.validator.format(“两次输入不能相同!”));
```

 #### 验证值不允许与特定值等于

```
jQuery.validator.addMethod(“notEqual”, function(value, element, param) {
            return value != param;
        }, $.validator.format(“输入值不允许为 { 0 }!”));
```

#### 验证值必须大于特定值(不能等于)

```
jQuery.validator.addMethod(“gt”, function(value, element, param) {
            return value > param;
        }, $.validator.format(“输入值必须大于 { 0 }!”));
```

#### 小数点前最多9位，小数点后最多6位

```
jQuery.validator.addMethod("decimal", function (value, element) {
    return this.optional(element) || /^([1-9]\d{0,8}|0)(\.\d{1,6})?$/.test(value);
}, "小数点前最多9位，小数点后最多6位^_^");
```

#### 结束时间不能小于开始时间

```
jQuery.validator.addMethod("laterTo", function (value, element, param) {
    return $(param).val().split("-").join("") < $(element).val().split("-").join("");
}, "结束时间不能小于开始时间^_^");
```

 