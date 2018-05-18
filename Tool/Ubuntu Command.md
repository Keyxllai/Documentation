## 问题
安装遇到异常 Could not get lock /var/lib/dpkg/lock - open (11: Resource temporarily unavailable)
Unable to lock the administration directory (/var/lib/dpkg), is another process using it?
- `ps -A | grep apt` 能够输出每个apt-get或者apt进程
- `sudo kill -9 1345【进程ID】`或者`sudo kill -SIGKILL 1345【进程ID】`杀掉每个进程
