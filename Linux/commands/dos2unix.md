# dos2unix

### 修改单个文件
```
dos2unix filename
```

### 修改整个目录

```
find . -type f -exec dos2unix {} \;
```

# unix2dos

### 修改单个文件
```
unix2dos filename
```

### 修改整个目录

```
find . -type f -exec unix2dos {} \;
```
