---
title: SpringMVC接口抓取自参数新增接口参数
date: 2024-05-04 15:50:54
tags: [SpringMvc]
---
# andlerMethodArgumentResolver

继承 HandlerMethodArgumentResolver 新增参数使用
<!--more-->
```

/**
 * HandlerMethodArgumentResolver是SpringMVC中，给控制层方法的参数赋值的
 * 先根据参数类型寻找参数，如果有该类型，就赋值给这个参数
 */
@Service
public class UserArgumentResolver implements HandlerMethodArgumentResolver {
    /**
     * 定义要赋值的参数类型
     *
     * @param methodParameter
     * @return
     */
    @Override
    public boolean supportsParameter(MethodParameter methodParameter) {
        Class<?> clazz = methodParameter.getParameterType();
        return clazz == JSONObject.class;
    }

    /**
     * 具体执行赋值的方法
     *
     * @param methodParameter
     * @param modelAndViewContainer
     * @param nativeWebRequest
     * @param webDataBinderFactory
     * @return
     * @throws Exception
     */
    @Override
    public Object resolveArgument(MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer, NativeWebRequest nativeWebRequest, WebDataBinderFactory webDataBinderFactory) throws Exception {
        //获取request和response
        HttpServletRequest request = nativeWebRequest.getNativeRequest(HttpServletRequest.class);
        HttpServletResponse response = nativeWebRequest.getNativeResponse(HttpServletResponse.class);
        String authorization = request.getHeader("Authorization");
        String paramStr = getAllParam(request);
        //如果是get请求则获取url的参数
        if ("GET".equals(request.getMethod())) {
            paramStr = JSONObject.toJSONString(getUrlParam(request.getRequestURL().toString()));
        }
        return paramStr;
    }


    /**
     * 获取url上的参数
     *
     * @param url
     * @return
     */
    private Map<String, String> getUrlParam(String url) {
        if (url == null) {
            return null;
        }
        int index = url.indexOf("?");
        String param = url.substring(index + 1);

        String[] params = param.split("&");

        Map<String, String> map = new HashMap<>();

        for (String item : params) {
            String[] kv = item.split("=");
            if (kv.length == 1) {
                map.put(kv[0], "");
            } else {

                map.put(kv[0], kv[1]);
            }

        }
        return map;
    }


    /**
     * 获取整个表单的参数
     *
     * @param request
     * @return
     * @throws IOException
     */

    private String getAllParam(HttpServletRequest request) throws IOException {
        String paramStr;
//        request.getContentType()
        //获取表单参数
        if (request.getContentType() != null && (request.getContentType().indexOf("multipart/form-data") >= 0 || request.getContentType().indexOf("application/x-www-form-urlencoded") >= 0)) {
            Map form_map = new HashMap();
            Enumeration<String> en = request.getParameterNames();
            while (en.hasMoreElements()) {
                String parameterName = en.nextElement();
                form_map.put(parameterName, request.getParameter(parameterName));
            }
            JSONObject fromJson = new JSONObject(form_map);
            paramStr = fromJson.toString();
        } else {
            MyRequestWrapper requestWrapper = new MyRequestWrapper(request);
            paramStr = requestWrapper.getBody();
            if (paramStr.equals("")) {
                paramStr = "{}";
            }
            JSONObject jsonObject = JSONObject.parseObject(paramStr);
            paramStr = jsonObject.toString();
        }
        return paramStr;
    }
}
```

# 指定返回值类型处理

    @ControllerAdvice
    public class MyResponseBodyAdvice implements ResponseBodyAdvice {
        @Override
        public boolean supports(MethodParameter returnType, Class converterType) {
            //针对某个返回值类型进行处理
            return returnType.hasMethodAnnotation(JsonView.class);
        }

        @Override
        public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
            JSONObject jsonObject = JSONObject.parseObject((String) body);
            // 将返回结果封装成ResponseResult对象统一返回。
            jsonObject.put("message", "请求成功");
            jsonObject.put("param", "null");
            jsonObject.put("code", 200);
            return jsonObject;
        }
    }

# 进行MVC配置文件处理

    @Configuration
    public class WebConfig extends WebMvcConfigurationSupport {
      @Autowired
        UserArgumentResolver userArgumentResolver;
        @Autowired
        TokenArgumentResolver tokenArgumentResolver;

        /**
         * 添加我们到处理器到配置中
         * @param argumentResolvers
         */
        @Override
        public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
            argumentResolvers.add(userArgumentResolver);
            argumentResolvers.add(tokenArgumentResolver);
        }
    }

