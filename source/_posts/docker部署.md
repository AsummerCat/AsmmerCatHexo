---
title: docker部署
date: 2020-04-08 21:28:31
tags: [docker]
---

# docker部署

dockerfile

```
FROM tomcat:8.5.53-jdk8

ADD zoe-optimus-web-api/target/zoe-optimus-web-api.war /usr/local/tomcat/webapps/

CMD ["catalina.sh", "run"]
```

