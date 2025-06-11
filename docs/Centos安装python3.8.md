# Centos安装python3.8

### 使用 yum 命令安装必要的依赖项
	
	sudo yum install gcc openssl-devel bzip2-devel libffi-devel
### 下载 Python 3.8 的源代码包
```
cd ~
wget https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tgz
```

### 解压源代码包并进入解压后的目录
```
tar xzf Python-3.8.0.tgz
cd Python-3.8.0
```
### 配置 Python 安装选项
	./configure --enable-optimizations
### 编译 Python
	make
### 安装 Python
	sudo make altinstall
安装完成后，验证 Python 是否已成功更新

	python3.8 --version

### 更新pip

	 python3.8 -m pip install --upgrade pip
## 配置虚拟环境

### 安装virtualenv
	pip install virtualenv

### 创建虚拟环境
	virtualenv -p /usr/local/bin/python3.8 myenv
### 激活虚拟环境
	source myenv/bin/activate

### 退出虚拟环境
	deactivate      #---任何目录下，都可以


