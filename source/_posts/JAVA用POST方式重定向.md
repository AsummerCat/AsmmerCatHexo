---
title: JAVA用POST方式重定向
date: 2022-12-13 16:06:24
tags: [java,工具类]
---
```

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.HashMap;
import java.util.Map;

/**
 * 用POST方式 重定向
 */
public class RedirectWithPost {
    Map<String, String> parameter = new HashMap<String, String>();
    HttpServletResponse response;

    public RedirectWithPost(HttpServletResponse response) {
        this.response = response;
    }

    public void setParameter(String key, String value) {
        this.parameter.put(key, value);
    }

    public void sendByPost(String url) throws IOException {
        this.response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = this.response.getWriter();
        out.println("<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.01 Transitional//EN\">");
        out.println("<HTML>");
        out.println(" <HEAD>");
        out.println(" <TITLE>loading</TITLE>");
        out.println(" </HEAD>");
        out.println(" <BODY>");
        out.println("<form name=\"submitForm\" action=\"" + url + "\" method=\"post\">");
        out.println(" <input type=\"text\" name=\"clientID\" value=" + parameter.get("clientID") + " style=\"display: none;\">");
        out.println(" <input type=\"text\" name=\"userId\" value=" + parameter.get("userId") + " style=\"display: none;\">");
        out.println(" <input type=\"text\" name=\"phone\" value=" + parameter.get("phone") + " style=\"display: none;\">");
        out.println(" <input type=\"text\" name=\"timestamp\" value=" + parameter.get("timestamp") + " style=\"display: none;\">");
        out.println(" <input type=\"text\" name=\"userdata\" value style=\"display: none;\">");
        out.println(" <input type=\"text\" name=\"sign\" value=" + parameter.get("sign") + " style=\"display: none;\">");
        out.println(" <input type=\"text\" name=\"userName\" value style=\"display: none;\">");
        out.println("<input type=\"submit\" value=\"提交\">");
        out.println("</from>");
//        out.println("<script>window.document.submitForm.submit();</script> ");
        out.println(" </BODY>");
        out.println("</HTML>");
        out.flush();
        out.close();
    }

    public static void main(String[] args) throws IOException {
        HttpServletResponse response = null;
        //java重定向 返回post请求
        RedirectWithPost redirectWithPost = new RedirectWithPost(response);
        redirectWithPost.setParameter("clientID", "1");
        redirectWithPost.setParameter("userId", "1");
        redirectWithPost.setParameter("phone", "1");
        redirectWithPost.setParameter("timestamp", "1");
        redirectWithPost.setParameter("sign", "1");
        //发送
        redirectWithPost.sendByPost("www.baidu.com");
    }
}
```