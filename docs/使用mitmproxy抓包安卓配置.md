## 使用mitmproxy抓包安卓配置
### 1. 连接手机

查看已连接设备  

```plain
adb devices
```



 进入交互式 Shell  

```plain
adb shell
```

1.重命名你的证书文件

```plain
openssl x509 -inform DER -in your_certificate.crt -out charles_ca.pem
```

在 Android 系统分区（通常是 `/system` 或 `/system_root/system`）下，可信任 CA 证书被存放在 `**/system/etc/security/cacerts/**` 目录中（部分 ROM 可能放在 `/system/etc/security` 或 `/system/etc/security/cacerts_google`，需要根据手机及 ROM 版本来确认）。

**系统证书需要采用特定的文件命名方式（一般是由证书主题散列值取前 8 位 + **`**.0**`**）**，并设置合适的权限。最常见的做法是：

1. 先用 `openssl x509 -subject_hash_old -in charles_ca.pem | head -1` 命令计算出证书的“旧版哈希”值（通常是 8 位十六进制字符串），得到比如 `abcdef12` 之类的值；
2. 将文件命名为 `abcdef12.0`（注意中间的点和后缀是 `.0`）。
3. 若要验证文件名是否正确，可以再次执行 `openssl x509 -subject_hash_old -in abcdef12.0 -noout -issuer_hash`，如果返回值正好是 `abcdef12`，说明文件名与证书哈希匹配



计算出证书的名字，将你的证书文件改为计算出的名字

```plain
openssl x509 -subject_hash_old -in charles_ca.pem | head -1
```

 以 root 身份将 `abcdef12.0` 文件复制到 `/system/etc/security/cacerts/`

```plain
cp /sdcard/Download/abcdef12.0 /system/etc/security/cacerts/
```

 设置文件权限  

```plain
chmod 644 /system/etc/security/cacerts/abcdef12.0
chown root:root /system/etc/security/cacerts/abcdef12.0

```

###  2. 在 Magisk 下创建自定义模块并放置证书  


```plain
su
mkdir -p /data/adb/modules/add_ca_certs
```

文件目录

```plain
/data/adb/modules/
└── add_charles_ca/               (模块文件夹 - 名字可自定义)
    ├── module.prop               (模块描述文件)
    └── system/
        └── etc/
            └── security/
                └── cacerts/

```

 创建并编辑 `module.prop`

+ `id`: 模块ID（唯一识别符，不能含空格）
+ `name`: 模块名称
+ `version`: 模块版本文字
+ `versionCode`: 模块版本号（整数）
+ `author`: 作者名
+ `description`: 模块简要描述

```plain
cat <<EOF > /data/adb/modules/add_charles_ca/module.prop
id=add_charles_ca
name=Add Charles CA
version=1.0
versionCode=1
author=Me
description=Install Charles cert systemlessly
EOF

```

### 3. 准备和复制证书文件
1. **导出 Charles 证书**
+ 在电脑端 Charles → `Help -> SSL Proxying -> Install Charles Root Certificate`

导出 `.crt` 或 `.pem` 文件。

2. **转换为 PEM 并计算哈希（如需）**
+ 如果你导出是 `.crt`，就执行： 

```plain
openssl x509 -inform DER -in charles.crt -out charles.pem
```

获取旧版哈希值（Android 仍多用 subject_hash_old）： 

```plain

openssl x509 -subject_hash_old -in charles.pem | head -1
```

 假设结果是 `abcdef12`；

重命名证书： 

```plain
mv charles.pem abcdef12.0
```

3. **把证书拷贝到手机**

```plain
adb push abcdef12.0 /sdcard/Download/
```

例如放到 `/sdcard/Download/abcdef12.0`；

回到 root shell，复制到模块目录： 

```plain
cp /sdcard/Download/abcdef12.0 /data/adb/modules/add_charles_ca/system/etc/security/cacerts/
```

设置权限为 `644`

```plain
chmod 644 /data/adb/modules/add_charles_ca/system/etc/security/cacerts/abcdef12.0
```

一般拥有者默认就是 `root:root`，无需再改，但如果要手动指定可用： 

```plain
root:root /data/adb/modules/add_charles_ca/system/etc/security/cacerts/abcdef12.0
```

重启电脑

```plain
reboot
```

4. **把证书拷贝到手机**

[https://github.com/Fuzion24/JustTrustMe/releases/tag/v.2](https://github.com/Fuzion24/JustTrustMe/releases/tag/v.2)



### 4. 安装magic


```plain
adb reboot recovery
```

```plain
adb push Magisk-v20.2.zip /
```

