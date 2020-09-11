# Less-31 **WAF PROTECT** 
TAG：GET型注入、WAF  
## 漏洞利用  
Less31是less29的变种,它在参数的闭合方式上有所改变   
![less31_1](images\less31_1.png)  
  
当在第二个参数上输入"时，页面显示出了报错信息，同时也暴漏了页面对参数的闭合方式  
![less31_2](images\less31_2.png)  
由报错信息可知，传入页面的参数由("$id")闭合  
  
获取回显点payload:```?id=1&id=0") union select 1,2,("3```  
![less31_3](images\less31_3.png)  