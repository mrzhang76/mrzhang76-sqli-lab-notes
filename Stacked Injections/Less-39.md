# GET-Stacked Query Injection-Intiger based  
TAG：堆叠注入、GET型注入
## 漏洞利用  
![less39_1](images\less39_1.png) 
  
输入一个单引号获取报错信息，可知参数为整形  
![less39_2](images\less39_2.png) 
  
此less与less38原理相似，都使用了```mysqli_multi_query()```函数，可以使用堆叠注入  
  
Payload:```?id=1;insert into users(id,username,password) values ('18','less39','hello')--+```  
![less39_3](images\less39_3.png) 
