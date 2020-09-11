# POST-Error based-string-Stacked Blind 
TAG：POST型注入、堆叠注入、盲注  
## 漏洞利用  
![less45_1](images\less45_1.png)  
此less与less44相似，都没有报错信息显示，需要使用相同的方式进行参数包裹方式的测试  
![less45_2](images\less45_2.png)  
经过测试，本less的参数包裹方式为单引号+小括号  
构造建立用户的语句  
Payload:```1');insert into users(id,username,password) values ('24','less45','hello')--+```  
![less45_3](images\less45_3.png)  
成功建立用户  
![less45_4](images\less45_4.png)  