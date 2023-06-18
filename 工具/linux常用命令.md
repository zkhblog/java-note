```
https://www.cnblogs.com/xuxinstyle/p/9609551.html
```

一、先确定日志中是否有该时间点，然后根据日期查询日志  
```
grep '2021-05-14 10:38:42' a.txt
sed -n '/2021-05-14 10:38:42/,/2021-05-14 16:09:26/p' a.txt
```
二、使用more和cat命令分页（每页5行）查看打印的日志，并使用关键词筛选
```
cat -n a.txt | grep "开票完成"|more -5
```
三、实时监控100行日志
```
tail -100f a.txt或者tail -nf 100 a.txt
```
四、按一定格式查找文件
```
find [path ] [-option] [action]
-name表示按名称
-user表示所有者
-size表示目录中大于(+)的文件小于(-)
```
五、启动Linux的定时任务
编辑cronjob任务的的命令是crontab -e，然后编辑任务列表，最后通过service cron start启动
六、权限相关命令
更改文件权限chmod,改变文件所有者是chown

```
centos6和7的区别
① 防火墙：6是iptables，7是firewalld
② 启动服务的命令：6是service，7是systemctl
```

```
netstat -ano | findstr 8090

tasklist|findstr 16016

taskkill /T /F /PID 16016
```