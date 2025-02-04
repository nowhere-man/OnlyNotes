# core dump
## 设置core dump文件大小
查看core dump文件大小
```bash
ulimit -a
```
可以看到：
```shell
core file size              (blocks, -c) unlimited
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 1544026
max locked memory           (kbytes, -l) 65536
max memory size             (kbytes, -m) unlimited
open files                          (-n) 1024
pipe size                (512 bytes, -p) 8
POSIX message queues         (bytes, -q) 819200
real-time priority                  (-r) 0
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) 1544026
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited
```

永久设置core文件大小：

修改/etc/security/limits.conf文件， 添加：
```shell
#<domain>      <type>  <item>         <value>
#
*     soft   core unlimited

*     hard  core unlimited
```
## 设置core dump路径
修改 /proc/sys/kernel/core_uses_pid 文件内容为1：
```shell
echo "1" > /proc/sys/kernel/core_uses_pid
```

控制core文件保存位置和文件名格式： 
```shell
echo "/corefile/core-%e-%p-%t" > /proc/sys/kernel/core_pattern
```

```shell
%p - insert pid into filename 添加pid(进程id)
%u - insert current uid into filename 添加当前uid(用户id)
%g - insert current gid into filename 添加当前gid(用户组id)
%s - insert signal that caused the coredump into the filename 添加导致产生core的信号
%t - insert UNIX time that the coredump occurred into filename 添加core文件生成时的unix时间
%h - insert hostname where the coredump happened into filename 添加主机名
%e - insert coredumping executable name into filename 添加导致产生core的命令名
```
