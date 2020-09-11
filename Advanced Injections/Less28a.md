# Less-28a **GET Trick with SELECT & UNION**  
## 漏洞利用
此less为less28关闭错误回显信息的变种版本，可以使用之前的探测方法来获取参数的包裹方式  
![less2a_1](images\less28a_1.png) 
使用less27a的方法获取参数类型为字符型带小括号  

根据参数类型可以直接使用less28的payload来执行恶意攻击    
获取回显点payload:```?id=0')%a0union%a0select%a01,2,3;%00```
![less2a_2](images\less28a_2.png) 