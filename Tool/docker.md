## Docker操作
---
### alphine版本ubuntu镜像没有安装vim
> 修改远地址(阿里云)
```
 echo "deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse" >> /etc/apt/sources.list
 echo "deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse" >> /etc/apt/sources.list
 ...
 apt-get update 
 apt-get upgrade
```
> 安装vim
```
apt-get install vim
```
![alphine安装vim](http://innomind-zj.smartbx.top/alpine-vim.png)

### 启动镜像做过修改如何保存，下次镜像启动时候修改还在
```
docker commit -a "Lai" -m "ubuntu-alpine with vim and change source path" f3498aa56515 ubuntu-alpine:1.0.1
```
![保存镜像](http://innomind-zj.smartbx.top/dockersave.png)