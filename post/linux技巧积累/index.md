

# 1 系统运维服务器故障排查

__统计服务器tcp连接状态数__


```bash
ss -ant|awk '{a[$1]++}END{for(i in a)print a[i],i}'
# 记住a为awk关联数组 索引为$1 值为数量，for(i in a)用索引遍历数组 i为索引a[i]为 i出现的次数
```
<!--more-->

__查看nginx日志path访问前20名__
```bash
cat /usr/local/nginx/logs/hugo.log|awk '{print $7}'|awk -F? '{a[$1]++}END{for(i in a)print i,a[i]}'|sort -k 2 -nr|head -n 20
```


__lvm给根扩容 /dev/sdb 盘加到根__

```bash
pvcreate /dev/sdb
vgextend vg_templet /dev/sdb
lvextend -l +100%FREE /dev/vg_templet/lv_root
resize2fs /dev/vg_templet/lv_root
```

随机密码生成
```bash
pwgen -y10 16 200

openssl rand -hex 10
```
ps 自定义格式 psr意思为进程运行在运行在哪个cpu上
```bash
ps -eo %cpu o%mem opsr,pid,ppid,start_time,user,time,cmd --sort %cpu
```

高级linux分区parted
```bash
parted /dev/vdb mklabel GPT
parted /dev/vdb mkpart vdb1 1 100%

```
查看进程号$pid打开的文件
```bash
lsof -p $pid
```

系统资源监控sysstat包
```bash
sar -n DEV 1 5 监控网卡
sar -u 1 5 监控cpu
sar -d 1 3 监控块设备
sar -r 1 3 监控内存
```

---

# 2 服务器安全
iptables防护常用配置

```bash
iptables
```

# 3 脚本技巧

`xargs`

`exec`
`set`

# 4 qcow2的虚拟机镜像扩容
直接扩展磁盘镜像

先关机
```bash
qemu-img resize centos6.8.xm +500G
qemu-img info centos6.8.xm
```
然后启动系统,下面的看情况分区lvm的例子如下
```bash
fdisk -l
fdisk /dev/sda #给磁盘新加的分区
如果原来有/dev/sda1 /dev/sda2 就分区3
n
3
w
```
需要重启
```bash
vgextend vggroup /dev/sda3 #分区3加入名字为vggroup的vg组
lvextend -l +100%FREE /dev/vggroup/vl_root
resize2fs /dev/vggroup/vl_root #重置文件系统

```



学习资料

[google shell编程风格](http://zh-google-styleguide.readthedocs.io/en/latest/google-shell-styleguide/contents/)












