## docker部署node服务
### Dockerfile文件
```
FROM node:10.0-alpine

RUN apk --update add tzdata \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && apk del tzdata

RUN mkdir -p /usr/src/nodejs/

WORKDIR /usr/src/nodejs/

# add npm package
COPY package.json /usr/src/nodejs/package.json
RUN cd /usr/src/nodejs/
RUN npm i

# copy code
COPY . /usr/src/nodejs/

EXPOSE 30010

# CMD npm run dev
# 定义变量，构建可通过--build-arg 在docker build传递给构造器 ARG node_env
ARG node_env 
ENV NODE_ENV=$node_env
CMD npm run ${NODE_ENV}
```
### 构建Docker镜像
```
docker image build -t lai/hello-dk:1.0.0
docker image build --build-arg node_env=dev -t shaunlai/hello-dk:1.0.1 .
```
### 镜像上传仓库
```
docker push shaunlai/hello-dk:1.0.1
```
![镜像上传](http://innomind-zj.smartbx.top/DK3.png)
### 运行容器
```
docker run -d -it -p 30010:30010 shaunlai/hello-dk
```
> -t 让 Docker 分配一个伪终端，并绑定到容器的标准输入上  
> -i 让容器的标准输入持续打开  
> -d 让容器在后台，以守护进程的方式执行，这也是在工作中最常用到的

![动态设置环境变量](http://innomind-zj.smartbx.top/DK2.png)

![docker 命令](http://innomind-zj.smartbx.top/DK1.png)
### 进入容器操作
```
docker exec -it  f69b2d0a6f22 /bin/sh
docker exec -it  f69b2d0a6f22 bash
docker exec -it  f69b2d0a6f22 sh
```

![docker CMD](http://innomind-zj.smartbx.top/dk.png)