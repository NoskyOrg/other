
# install shadowsocks & serverspeeder on CentOS 7

## 安装 影梭 & 锐速

### yum 源安装依赖

```bash
yum -y install xorg-x11-xauth m2crypto python python-devel python-setuptools openssl openssl-devel curl wget unzip gcc automake autoconf make libtool net-tools
yum -y update
reboot
```

### 编译安装依赖

```bash
cd /root
wget https://github.com/jedisct1/libsodium/releases/download/1.0.16/libsodium-1.0.16.tar.gz
tar xf libsodium-1.0.16.tar.gz 
cd libsodium-1.0.16
./configure && make -j2
make install

echo "include ld.so.conf.d/*.conf" > /etc/ld.so.conf
echo "/lib" >> /etc/ld.so.conf
echo "/usr/lib64" >> /etc/ld.so.conf
echo "/usr/local/lib" >> /etc/ld.so.conf
ldconfig
```

### 更换内核 & 关闭 防火墙 SELinux

因为安装`锐速破解版`需要指定的内核版本

```bash
rpm -i --force https://buildlogs.cdn.centos.org/c7.01.u/kernel/20150327030147/3.10.0-229.1.2.el7.x86_64/kernel-3.10.0-229.1.2.el7.x86_64.rpm
reboot

# wget https://buildlogs.cdn.centos.org/c7.01.u/kernel/20150327030147/3.10.0-229.1.2.el7.x86_64/kernel-3.10.0-229.1.2.el7.x86_64.rpm
# rpm -i --force kernel-3.10.0-229.1.2.el7.x86_64.rpm

systemctl stop firewalld
systemctl disable firewalld
getenforce

# 开启一个端口
firewall-cmd --zone=public --add-port=80/tcp --permanent    
# --permanent永久生效，没有此参数重启后失效
firewall-cmd --reload
# 刷新服务生效

# 关闭一个端口
firewall-cmd --zone=public --remove-port=2222/tcp --permanent
firewall-cmd --reload

# 查看开放的端口
firewall-cmd --zone=public --list-ports
firewall-cmd --reload
```

### 安装 & 配置 & 启动 shadowsocks

```bash
yum -y install python-setuptools && easy_install pip
pip install shadowsocks

ip a

# 以下的 server 更换为真实IP，port 自己随意更换，推荐大于 10000
cat << eof > /etc/shadowsocks.json
{
    "server":"45.0.0.1",
    "server_port":10000,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"bigPass",
    "timeout":300,
    "method":"salsa20",
    "fast_open": true
}
eof

ssserver -c /etc/shadowsocks.json -d start
ps aux | grep ssserver
netstat -lnpt | grep python
```

### 安装 锐速破解版

```bash
wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh
bash serverspeeder.sh

ps -elf | grep ssserver 
netstat -lnpt | grep python
ps aux | grep ssserver | wc -l
```

### 计划任务 配置开机自启

```bash
cd /root

vi chkSS.sh
# ----------------------
#!/bin/bash

num=`ps aux | grep ssserver | wc -l`

if ((num < 2))
then
    ssserver -c /etc/shadowsocks.json -d start
fi
# ----------------------


$ systemctl start crond
$ systemctl enable crond
cat << eof >> /var/spool/cron/root
*  *  *  *  * bash /root/chkSS.sh        
*  *  *  *  * sleep 10; bash /root/chkSS.sh 
*  *  *  *  * sleep 20; bash /root/chkSS.sh 
*  *  *  *  * sleep 30; bash /root/chkSS.sh 
*  *  *  *  * sleep 40; bash /root/chkSS.sh
*  *  *  *  * sleep 50; bash /root/chkSS.sh 
eof
crontab -l


cat chkSS.sh 
chmod +x chkSS.sh 
bash -x chkSS.sh 
```

## 完
