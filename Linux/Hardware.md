# 硬件信息

## cpu

```bash
# 详细
cat /proc/cpuinfo
# 概览
lscpu

# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l

# 查看线程数
grep 'processor' /proc/cpuinfo | sort -u | wc -l    

```

## 显卡

```bash
lspci |grep VGA
```

## 内存

```bash
free -m

cat /proc/meminfo | grep MemTotal
```

## 硬盘

```bash
lsblk

sudo fdisk -l |grep Disk

df -h
```

## OS

```bash
uname -a 

cat /etc/*-release
```

