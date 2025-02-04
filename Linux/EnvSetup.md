# Environment Setup

## Ubuntu

### 设置root密码
```bash
sudo passwd
```

### 用户sudo免输密码
```bash
# 使用root账户登录
passwd jack
chmod u+w /etc/sudoers
echo "liushaojie ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
chmod u-w /etc/sudoers
```

### 替换apt源
```bash
cp /etc/apt/sources.list /etc/apt/sources.list.bak
sed -i "s@http://.*archive.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
sed -i "s@http://.*security.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list

sudo apt update
sudo apt upgrade -y
```

### 开发工具
```bash
sudo apt-get install -y build-essential gdb yasm nasm cmake python3 python3-pip git vim pkgconf zip
```

### 替换pip源
```bash
python3 -m pip install --upgrade pip
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip3 config set global.trusted-host pypi.tuna.tsinghua.edu.cn
```

### 安装python库
```bash
pip3 install matplotlib numpy pandas plotly plotly-express scipy ipykernel jupyter
```

### git 配置
```bash
git config --global user.name "nowhere-man"
git config --global user.email "liu.shaojie@outlook.com"
```
