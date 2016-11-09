---
title: Maven 使用记录
date: 2016-10-28 14:09:12
tags:
---

### Maven生成项目命令
```
mvn archetype:generate 
-DgroupId=com.mycompany.app
-DartifactId=myWebApp 
-DarchetypeArtifactId=maven-archetype-webapp 
-DinteractiveMode=false 
-DarchetypeCatalog=local
```


spring 引入session需要javax.servlet-api

freemarker 需要spring-context-super