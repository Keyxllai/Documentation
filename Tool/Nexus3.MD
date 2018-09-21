## Installation
- [Download](https://help.sonatype.com/repomanager3/download/download-archives---repository-manager-3)
## Ubuntu安装
- root用户在/usr/local下创建文件夹nexus `mkdir -p /usr/local/nexus`
- 将安装的压缩包解压到上面的目录,解压后有两个文件夹 `tar -zxf uexus-3.13-0-01-unix.tar.gz -C /usr/local/nexus`
- 创建一个uexus的系统用户 `useradd  nexus --shell /bin/bash` 检查用户是否创建成功`cat /etc/passwd`
- 设置 /usr/local/nexus为nexus的主目录 `chown -R nexus:nexus  /usr/local/nexus`
- 到nexus-3.13.0-01 目录下启动nexus `./bin/nexus run`
- 访问localhost:8081
