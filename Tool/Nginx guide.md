## ���
Nginx (engine x) ��һ�������ܵ�HTTP�ͷ�������������Ҳ��һ��IMAP/POP3/SMTP��������
## ��������
Nginx��Igor Sysoev[�����������Ү��] 2002��Ϊ����˹�������ڶ���Rambler.ruվ�㿪����һ��HTTPվ�����������һ�������汾0.1.0������2004��10��4�գ���������ÿ���ܹ�����5�ڸ������ܹ������ڼ������е���������ϵͳ�ϡ�
Nginx�������������������ۼ��ҵ��������**C10K**������Web����������������ʹ������ͻ��Զ�������������Apache�Ƴ�V2.4����������⣬ͬʱҲ��������������������������⣬Lighttpd,Nginx
## �ص�
 -   ֧��Linux epoll;������֧��2/3�򲢷����ӣ�epollģʽ���ƣ�CPUռ���ʵ͡�
 -   �ײ�ʵ�ִ�C;�������ܡ�
 -   ���Ĵ���ȫ�����¼������������
 -   ��ƽ̨
 -  ��Դ��������Ծ�ȸ�
## ��װ
Windows OS���ؽ�ѹ��ָ��Ŀ¼���ɣ�[���ص�ַ](https://nginx.org/en/download.html)
Ubuntu��װ sudo apt-get install nginx
��װ�ɹ�����Nginx�󣬿���ͨ�����������http://localhost����Nginx�Ļ�ӭҳ�棬NiginxĬ�϶˿�Ϊ80�����80�˿��Ѿ���ռ�ã���Ҫ�޸�conf/nginx.conf�ļ���80�˿�Ϊ���ö˿�
## ��������
 -   ����Nginx, �ɹ���Ὺ���������̣���Ȼ��������Windows����ʵ�ֿ�������`start nginx`
 - ���¼�������`nginx -s reload`
 - ��֤�����Ƿ���Ч`nginx -t`
 - ֹͣ����`nginx -s stop` `nginx -s quit`
 - Ubuntu check Nginx status`sudo systemctl status nginx`
 - Ubuntu start Nginx Server`sudo systemctl start nginx`
## ���ù���
 - **��̬������**
 ����Ϊ����sampleվ��
```
server {
  listen 7978;
  server_name 127.0.0.1;
  root dist;
  index index.html;
  location / {
	add_header X-Proxy-server $server_addr; //����Response Header
	error_page 404 = /index.html;           //404����Ĭ��ҳ��
  }
}
```
 - **�������**
 ```
 server {
  listen 7778;
  location / {
	proxy_pass http://baidu.com;
  }
}
 ```
 - **���ؾ���**
 1) RR(Round Robin) [Ĭ��ģʽ]
ÿ��������ʱ��˳����һ���ൽ��ͬ�ĺ�˷����������Խ��max_fails��fail_timeout��weight���ۺ�����
2) weight(Ȩ��)
ָ����ѯ���ʣ�weight�ͷ��ʱ��ʳ����ȣ����ں�˷��������ܲ������
3) ip_hash
ÿ���������IP��Hash������䣬ʹ��ÿ���ͻ��˷���һ����˷�������������ؼ�ȺSessionͬ������[hash���н��һ����]��
4) fair(������)
5) (url_hash)
������URL��Hash�������������ʹÿ��URL����ͬһ����˷���������˷�����Ϊ����ʱ�Ƚ���Ч

����ΪRR�������ã�����http://localhost:6606�ܹ�ѭ������Server1��Server2
```
http {
    ...
    #��upstreamָ���һ�鷴�����/���ؾ����˷�������
    upstream test_svr {
      server localhost:6607;
      server localhost:6608;
    }
    server {
      listen 6606;
      location / {
        proxy_pass http://test_svr;#����ָ�������ķ�������
      }
    }
}
```
����Ϊ��ȨRR������http://localhost:6606�ܹ�ѭ������Server1 ���κ�Server2һ�Ρ�
```
http {
    ...
    upstream test_svr {
      #ip_hash;
      server localhost:6607 weight=2 max_fails=2 fail_timeout=30s;
      server localhost:6608 weight=1 max_fails=2 fail_timeout=30s;
    }
    server {
      listen 6606;
      location / {
        proxy_pass http://test_svr;
        proxy_set_header Host  $host;#���������Header
        proxy_set_header X-Forwarded-For  $remote_addr;
      }
    }
}
```