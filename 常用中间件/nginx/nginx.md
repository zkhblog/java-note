# 基础
① nginx默认采用多进程工作方式，nginx启动后，会运行一个master进程和多个worker进程。其中master充当整个进程组与用户的交互接口，同时对进程进行监护，
管理worker进程来实现重启服务、平滑升级、更换日志文件、配置文件实时生效等功能。worker进程用来处理基本的网络事件，worker之间是平等的，
他们共同竞争处理来自客户端的请求

② 项目上线之后，用户在访问域名的时候，访问的应该是nginx的ip，而不是服务的ip。请求到了nginx之后，如果是静态资源I（/static/**）
直接在指定路径中找到静态资源然后返回。如果不是静态资源（/），nginx把他upstream转交给另外一个ip，这个ip所对应的进程是网关gateway。
到达网关之后，通过url信息断言应该转发给注册中心中的哪个微服务。在给微服务之前，也可以重写url  
在upstream的过程中要注意配置```proxy_set_header Host $host;```，避免nginx丢失host后，在网关中没法根据host进行断言

③ nginx有一个master，有```worker_processes```个worker，每个worker支持最大的连接数1024，支持的最大并发数是  
普通的静态访问最大并发数是：```worker_connectionsworker_processes/2```  
如果是http作为反向代理来说，最大并发数量应该是```worker_connectionsworker_processes/4```

########### 每个指令必须有分号结束。#################
########### 全局块 #################
user administrator administrators;                                                      #配置用户或者组，默认为nobody nobody。
worker_processes 2;                                                                     #允许生成的进程数，默认为1
pid /nginx/pid/nginx.pid;                                                               #指定nginx进程运行文件存放地址
error_log log/error.log debug;                                                          #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
include /usr/share/nginx/modules/*.conf;                                                #配置文件的引入

########### events块（影响Nginx服务器与用户的网络连接） #################
events {
    accept_mutex on;                                                                    #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;                                                                    #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;                                                                         #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;                                                           #最大连接数，默认为512
}

########### http块 #################
http {
    include       mime.types;                                                           #文件扩展名与文件类型映射表
    default_type  application/octet-stream;                                             #默认文件类型，默认为text/plain
    #access_log off;                                                                    #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;                                                 #combined为日志格式的默认值
    sendfile on;                                                                        #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;                                                            #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;                                                               #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;                                                #热备
    }

    error_page 404 https://www.baidu.com;                                               #错误页
    server {
        keepalive_requests 120;                                                         #单连接请求上限次数。
        listen       4545;                                                              #监听端口
        server_name  127.0.0.1;                                                         #监听地址       

        proxy_connect_timeout 12000;                                                    # 调大超时时间，避免nginx重试导致请求重发
        proxy_send_timeout 12000;
        proxy_read_timeout 12000;

        location  ~*^.+$ {                                                              #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;                                                                  #根目录
           #index vv.txt;                                                               #设置默认页
           proxy_pass  http://mysvr;                                                    #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;                                                              #拒绝的ip
           allow 172.18.5.54;                                                           #允许的ip           
        } 
    }
}
