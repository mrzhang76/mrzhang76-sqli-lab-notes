# Less-29 **GET Protection with WAF**
TAG：GET型注入、WAF
## 漏洞利用  
less29、less30、less31都是架设了WAF的页面，需要额外的JSP环境来满足WAF功能的运行  
因在window上部署较为容易，选择了使用jspstudy与phpstudy来搭建环境  
本题的真正入口在目录下的tomcat-files文件夹中的less，必须携带参数才能正常进入，否则会报错    
![less29_2](images\less29_2.png)  
输入一个单引号后跳转到了waf提示界面  
![less29_3](images\less29_3.png)  

那就意味着传输给页面的数据被架设在tomcat上的jsp服务器进行了检测，只有不触发WAF才能将恶意语句传输给apache服务器上的页面  
![less29_1](images\less29_1.png)  
这时候需要掌握不同网页服务器进行参数获取与解析的知识  
![less29_4](images\less29_4.png)  
JSP/TOMCAT会获取参数列表中第一个参数，PHP/APACHE会获得参数列表中最后一个参数  
  
根据这种特性，可以构造双参数绕过架设在JSP/TOMCAT上的WAF，直接将恶意语句提交给PHP/APACHE服务器  
Payload:```http://192.168.232.170:8080/tomcat-files/sqli-labs/Less-29/?id=1&id='```  
![less29_5](images\less29_5.png)  
从页面的回显可知，这种构造方式成功的绕过了WAF的审查  
  
获取回显点payload:```?id=1&id=0' union select 1,2,'3```  
![less29_6](images\less29_6.png)  
  
获取数据库名payload:```?id=1&id=0' union select 1,database(),'3```  
![less29_7](images\less29_7.png)  
   
## 源码分析
### less29源代码  
index.jsp:(WAF页面)
```
 <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd" > 

<%@ page import="java.io.*" %>
<%@ page import="java.net.*" %>

<HTML>
<HEAD>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <TITLE>Less-29 WAF PROTECT</TITLE>
</HEAD>
<body bgcolor="#000000">

<%
	String id = request.getParameter("id");
	String qs = request.getQueryString();
		
	if(id!=null)
	{
		if(id!="")
		{
			try
			{
				String rex = "^\\d+$";
				Boolean match=id.matches(rex);
				if(match == true)
				{
					URL sqli_labs = new URL("http://localhost/sqli-labs/Less-29/index.php?"+ qs);
			        URLConnection sqli_labs_connection = sqli_labs.openConnection();
			        BufferedReader in = new BufferedReader(
			                                new InputStreamReader(
			                                sqli_labs_connection.getInputStream()));
			        String inputLine;
			        while ((inputLine = in.readLine()) != null) 
			            out.print(inputLine);
			        in.close();
				}
				else
				{
					response.sendRedirect("hacked.jsp");
				}
			} 
			catch (Exception ex)
			{
				out.print("<font color= '#FFFF00'>");
				out.println(ex);
				out.print("</font>");				
			}
			finally
			{
				
			}
		}
		
	}
	else
	{
		URL sqli_labs = new URL("http://localhost/sqli-labs/Less-29/index.php");
        URLConnection sqli_labs_connection = sqli_labs.openConnection();
        BufferedReader in = new BufferedReader(
                                new InputStreamReader(
                                sqli_labs_connection.getInputStream()));
        String inputLine;
        while ((inputLine = in.readLine()) != null) 
            out.print(inputLine);
        in.close();
	}
%>
</font> </div><center>

<font size='4' color= "#33FFFF">
<br>
<br>
Thanks to "int3rf3r3nc3" 
<a href="http://sectree.wordpress.com/">sectree.wordpress.com</a> for coding this page
<br><br>
</font>
<font size='3' color= '#99FF00'>
Refrences:
<a href="https://www.owasp.org/images/b/ba/AppsecEU09_CarettoniDiPaola_v0.8.pdf">AppsecEU09_CarettoniDiPaola_v0.8.pdf</a><br>
<a href="https://community.qualys.com/servlet/JiveServlet/download/38-10665/Protocol-Level Evasion of Web Application Firewalls v1.1 (18 July 2012).pdf">https://community.qualys.com/servlet/JiveServlet/download/38-10665/Protocol-Level Evasion of Web Application Firewalls v1.1 (18 July 2012).pdf</a>
</font>
</center>
 </BODY> 
</html>

```
### 漏洞点分析  
```
...
if(id!=null)
	{
		if(id!="")
		{
			try
			{
				String rex = "^\\d+$";
				Boolean match=id.matches(rex);
				if(match == true)
				{
					URL sqli_labs = new URL("http://localhost/sqli-labs/Less-29/index.php?"+ qs);
			        URLConnection sqli_labs_connection = sqli_labs.openConnection();
...
```
通过审阅WAF页面对于参数的处理部分，发现它只会处理第一个参数，只要同时传递多个函数就可以突破WAF的封锁  