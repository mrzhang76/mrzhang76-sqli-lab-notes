# Less25a **GET Trick with OR & AND Blind**
TAG：GET型注入、双写突破
## 漏洞利用  
此less为less25的变种，它关闭了报错信息的输出，在确定参数包裹方式上存在一定的障碍  
![less25a_1](images\less25a_1.png)    
   
```id=1```时可以正常显示，```id=1'```与```id=1"```都无法正常显示,可知参数为整形  
在判断出参数类型后，其余的操作与less25基本相同  