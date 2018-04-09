配置下载文件夹
1、在tomcat 安装目录\conf\Catalina\localhost下建立任意文件名xml文件，比如：download.xml,

内容如下：
```

<?xml version="1.0" encoding="UTF-8"?>

<Context  reloadable="true" docBase="D://download" crossContext="true">

</Context>
```

2、配置web.xml（tomcat的配置文件），修改如下配置：
```
<servlet>
        <servlet-name>default</servlet-name>
        <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
        <init-param>
            <param-name>debug</param-name>
            <param-value>0</param-value>
        </init-param>
        <init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
```
要将listings下面一行的false改为true。

Environment variables

ANDROID_HOME
D:\Android\sdk
PYTHON
D:\Python\Python36\


Jenkins Location
本机静态ip地址加端口号加Jenkins
http://192.168.2.25:8888/jenkins/

Shell
 	Shell executable
D:/Program Files/Git/git-bash.exe


SMTP port 25
Charset UTF-8
Default Content
```
<!DOCTYPE html>  
<html>  
<head>  
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>  
</head>  
<body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4"
    offset="0">
    <table width="95%" cellpadding="0" cellspacing="0"
        style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
        <tr>
            <td><br />
            <b><font color="#0B610B">构建信息</font></b>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
        <tr>
            <td>
                <ul>
<li>$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:</li>
<li>(邮件由Jenkins自动发出，请勿回复~)</li>
<li>构建状态：$BUILD_STATUS</li>
<li>触发原因：${CAUSE}</li>
<li>下载地址：<a href="http://192.168.2.25:8888/download">http://192.168.2.25:8888/download   </a></li>
<li>二维码下载：<img src='http:/192.168.2.25:8888/download/androidApk.jpg' width=200px height=200px /></li>
       <li>最近修改：${CHANGES, showPaths=false, format="%a：\"%m\"<br>", pathFormat="\n\t- %p"}</li>

             <li>构建日志： <a href="${BUILD_URL}console">${BUILD_URL}console   </a></li>
                    <li>构建  Url ： <a href="${BUILD_URL}">${BUILD_URL}    </a></li>
                    <li>工作目录 ： <a href="${PROJECT_URL}ws">${PROJECT_URL}ws   </a></li>
                    <li>项目  Url ： <a href="${PROJECT_URL}">${PROJECT_URL}    </a></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td><b><font color="#0B610B">变更集</font></b>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
   <tr>
            <td>${JELLY_SCRIPT,template="html"}<br/>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
    </table>
</body>
</html>

```


Global Tool Configuration


Git
Path to Git executable  D:\Program Files\Git\bin\git.exe

Configure Global Security

Markup Formatter
 	Markup Formatter Safe HTML


Project Recipient List

```
$DEFAULT_RECIPIENTS;$EMAIL
```
Project Reply-To List
```
$DEFAULT_REPLYTO
```
Content Type
HTML (txt/html)

Default Subject
```
$DEFAULT_SUBJECT
```

Default Content

```
$DEFAULT_CONTENT

```
Pre-send Script
```
if (!$SEND_EMAIL) cancel=true;

```
Post-send Script
```
$DEFAULT_POSTSEND_SCRIPT
```