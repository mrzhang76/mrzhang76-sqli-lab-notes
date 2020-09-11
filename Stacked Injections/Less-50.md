# GET -Error based -ORDER BY CLAUSE -numeric -Stacked injection  
TAG：GET型注入、堆叠注入、ORDER BY参数可控  
## 漏洞利用
![less50_1](images\less50_1.png)  
在使用```?sort=1'```尝试进行报错信息的获取时，发现并不能直接的获取报错语句，页面只提示了单引号导致了语句错误，在尝试```?sort=1"```时仍然报错，推测可能是整数型参数  
![less50_2](images\less50_2.png)  
  
![less50_3](images\less50_3.png)  
  
输入```?sort=1 desc```和```?sort=1 asc```  
![less50_4](images\less50_4.png)  
![less50_5](images\less50_5.png)  
发现页面输出的表排序发生了变化，这说明order by后的参数是可控的    
但是报错信息没有回显，没法使用报错注入，可以考虑基于时间的注入或者堆叠注入    
使用堆叠注入添加一个用户,因为页面会返回用户列表，这将很轻松的知道注入是否成功    
Payload:```?sort=1;insert into users(id,username,password) values ('25','less50','hello')--+```  
这个语句和less38的相似  
![less50_6](images\less50_6.png)  
用户添加成功，这意味着可以使用堆叠注入进行高危操作（into outfile注马）  