# Less-36 **GET Bypass MySQL Real Escape String**  
TAG：GET型注入、```mysql_real_escape_string()```
## 漏洞利用  
![less36_1](images\less36_1.png)  
  
可以使用less32一样的突破payload:```0%df' union select 1,database(),3--+```
![less36_2](images\less36_2.png)  
   
通过查询此less名称中提示的```mysql real escape string```
![less36_3](images\less36_3.png)  
发现这是一个php中用于转义特殊字符的函数，在某些非安全设置下可被突破  
参照：https://www.dazhuanlan.com/2019/12/10/5deeb6a7f18e6/