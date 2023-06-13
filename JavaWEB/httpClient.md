1 HTTP允许传输任意类型的数据对象，正在传输的类型由Content-Type加以标记

```
text/html：HTML格式（默认格式）
text/plain：纯文本格式
text/xml：XML格式
image/gif：gif图片格式
image/jpeg：jpg图片格式
image/png：png图片格式

application/xhtml+xml：XHTML格式
application/xm：XML数据格式
application/atom+xml：ATOM XML聚合格式
application/json：JSON数据格式
application/pdf：PDF格式
application/msword：Word文档格式
application/octet-stream：二进制数据流（如常见的文件下载）
application/x-www-form-urlencoded：（<form encType="">中默认的encType，form表单数据被编码为key/value格式发送到服务器，这是表单默认的提交数据的格式）
multipart/form-data：需要在表单中进行文件上传时，就需要使用该格式
```

# get请求

```
// ① 加请求头
// 模拟浏览器
    httpGet.addHeader("User-Agent","Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36");
// 防盗链
    httpGet.addHeader("Referer","https://www.baidu.com");

// ② url编码
    address=URLEncoder.encode(address,StandardCharsets.UTF_8.name());
```

# 异步连接池

