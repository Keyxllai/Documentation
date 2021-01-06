## 树莓派node环境

### 安装node
1. 查看系统信息
```
pi@raspberrypi:~ $ uname -a
Linux raspberrypi 5.4.79-v7+ #1373 SMP Mon Nov 23 13:22:33 GMT 2020 armv7l GNU/Linux
```
上面系统显示32位，ARM V7架构，基于这些信息去官网(https://nodejs.org/en/download/)下载相应的安装包
2 下载nodejs预编译安装包
```
wget https://nodejs.org/dist/v14.15.4/node-v14.15.4-linux-armv7l.tar.xz
```
解压安装包
```
tar -xvf node-v14.15.4-linux-armv7l.tar.xz
```
3 配置node和npm软连接
```
root@raspberrypi:/home/pi/node-v14.15.4-linux-armv7l/bin# sudo ln /home/pi/node-v14.15.4-linux-armv7l/bin/node /usr/local/bin/node

root@raspberrypi:/home/pi/node-v14.15.4-linux-armv7l/bin# sudo ln -s /home/pi/node-v14.15.4-linux-armv7l/bin/npm /usr/local/bin/npm
```
### 树莓派文件共享（samba）
树莓派和Windows直接进行文件传输，通过samba实现在Windows网上邻居访问树莓派即可。  

1 安装samba
```
root@raspberrypi:/# sudo apt-get install samba samba-common-bin
```
2 修改配置文件/etc/samba/smb.conf， [homes]节点中把read only修改为no
3 重启samba服务
```
root@raspberrypi:/home/pi# chmod 777 /etc/samba/smb.conf 
root@raspberrypi:/home/pi# sudo /etc/init.d/smbd restart
[ ok ] Restarting smbd (via systemctl): smbd.service.
```
4 添加默认用户pi到samba
```
root@raspberrypi:/home/pi# sudo smbpasswd -a pi
New SMB password:
Retype new SMB password:
Added user pi.
```