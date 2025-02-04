## scp

## Windows

### Linux复制到Windows

```bash
# 在windows mingw64 bash执行
scp ubuntu@10.44.20.58:/home/src/data.yuv /d/dst
```

### Windows复制到Linux

```bash
# 在windows mingw64 bash执行
scp /d/src/data.yuv ubuntu@10.44.20.58:/home/dst
```

## Linux

### 从本地复制到远程

```bash
# -r代表复制整个目录
scp -r /home/src ubuntu@10.44.20.58:/home/dst
```

### 从远程复制到本地

```bash
scp -r ubuntu@10.44.20.58:/home/src /home/dst
```

