注意：
nginx转发后，会丢失host头，造成网关不知道原host。在进行网关匹配的时候需要将头信息添加上。（网关匹配是优先匹配）
```
  location / {
        proxy_pass http://gulimall;
        proxy_set_header Host $host;
    }
```