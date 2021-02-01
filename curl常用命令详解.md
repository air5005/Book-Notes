[TOC]

# curl官网书籍
https://ec.haxx.se/

# 使用curl设置cookie
```
curl --cookie "PHPSESSID=%00%7f%7%7%73%74%65" "http://6.6.6.2/1.txt"
```

# 使用curl 发送 post xml攻击内容到服务器
1. 把攻击内容存放到文本里面
```
vim content.txt 

<?xml version=""1.0"" encoding=""utf-8""?>
<!DOCTYPE test[
<!ELEMENT test ANY>
<!ENTITY x "xxx">
<!ENTITY y "&x;">
<!ENTITY x  "&y;">
]>
<user><username>&flag;</username><password>admin</password></user>
```
2 下发命令
```
[ych@dynamic-005-005-005-002 ~]$ curl -X POST -d@content.txt "http://6.6.6.2/"
curl: (56) Recv failure: Connection reset by peer
```

# curl指定http头部信息
常见的媒体格式类型如下：

text/html ： HTML格式
text/plain ：纯文本格式
text/xml ： XML格式
image/gif ：gif图片格式
image/jpeg ：jpg图片格式
image/png：png图片格式
以application开头的媒体格式类型：

application/xhtml+xml ：XHTML格式
application/xml： XML数据格式
application/atom+xml ：Atom XML聚合格式
application/json： JSON数据格式
application/pdf：pdf格式
application/msword ： Word文档格式
application/octet-stream ： 二进制流数据（如常见的文件下载）
application/x-www-form-urlencoded ： <form encType="">中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）
另外一种常见的媒体格式是上传文件之时使用的：


1. 使用 post方式发送HTML格式内容
```
curl -X POST --header "Content-Type:text/html; charset=utf-8"  --data 'ych test' "http://6.6.6.2/"
```
2. 使用 post方式发送JSON数据格式内容
```
curl -X POST --header "Content-Type:application/json"  --data "{\"a\":\"123\"}" "http://6.6.6.2/"
```
POST 指定请求方式
–header 指定请求头部信息
–data 指定json请求体数据内容
