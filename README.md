# penvpn-ipv6

## Key&Cert(服务器侧)

### 安装easy-rsa
最新版本的OpenVPN不带easy-rsa，因此需要手动安装。
```
wget https://cloud.github.com/downloads/OpenVPN/easy-rsa/easy-rsa-2.2.0_master.tar.gz
tar -zxvf easy-rsa-2.2.0_master.tar.gz
cp -R easy-rsa-2.2.0_master/easy-rsa/ /etc/openvpn/
```

### 配置easy-rsa
编辑/etc/openvpn/easy-rsa/2.0目录下的vars文件。vim /etc/openvpn/easy-rsa/2.0/vars
可以自定义以下几项：
```
export KEY_COUNTRY=”CN”
export KEY_PROVINCE=”JiangSu”
export KEY_CITY=”NanJing”
export KEY_ORG=”GreenStudio”
export KEY_EMAIL=”admin@njut.asia“
```

之后使vars设置生效
```
source ./vars
./clean-all
```

### 生成证书
创建证书颁发机构（全部回车）
```
./build-ca server
创建服务器证书（全部回车，最后两个输入y）

./build-key-server server
创建客户端证书（全部回车，最后两个输入y，可创建多个名字不一样的客户端证书）

./build-key client
创建Hellman参数

./build-dh
```

## Server端配置
编辑/etc/openvpn/server/server.conf

```
; 本地ip，更改此处
local 192.168.0.79
port 1194
proto tcp

; 此处设置dev为tap方式，使用二层方式，以便新建interface自动生成mac地址
dev tap

; 使用已生成的openssl密钥进行配置
ca easy-rsa/pki/ca.crt
cert easy-rsa/pki/issued/server.crt
key easy-rsa/pki/private/server.key
dh easy-rsa/pki/dh.pem
tls-auth easy-rsa/ta.key
verify-client-cert

; 配置dhcp自动分配地址段
server 10.48.0.0 255.255.255.0
server-ipv6 2001:db8:f00:bebe::/64

push "route-ipv6 ::/0"
push "route-metric 2000"

client-to-client
duplicate-cn

cipher AES-256-CBC
comp-lzo

persist-key
persist-tun

explicit-exit-notify 0
```

## 客户端配置
客户端安装openvpn client，配置如下：

```
client
dev tap
proto tcp
remote <远端openvpn server地址> 1194
resolv-retry infinite

nobind
persist-key
persist-tun

ca /storage/emulated/0/Download/OpenVPN/cert/ca.crt
cert /storage/emulated/0/Download/OpenVPN/cert/client1.crt
key /storage/emulated/0/Download/OpenVPN/cert/client1.key
tls-auth cert/ta.key 1

remote-cert-tls server
cipher AES-256-CBC
comp-lzo
```
