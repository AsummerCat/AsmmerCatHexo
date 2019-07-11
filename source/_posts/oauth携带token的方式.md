---
title: oauth携带token的方式
date: 2019-07-11 22:23:10
tags: [Security,springCloud,Oauth2]
---

# oauth携带token的方式 

## 在Header中携带 

`http://localhost:8080/order/1 `
Header： 
```Authentication：Bearer f732723d-af7f-41bb-bd06-2636ab2be135```

## 拼接在url中作为requestParam 

```http://localhost:8080/order/1?access_token=f732723d-af7f-41bb-bd06-2636ab2be135```

## 在form表单中携带 

```http://localhost:8080/order/1 
form param： 
access_token=f732723d-af7f-41bb-bd06-2636ab2be135
```

