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

CMD npm run dev
```
### 构建Docker镜像
```
docker image build -t lai/hello-dk:1.0.0
```
### 运行容器
```
docker run -d -it -p 30010:30010 shaunlai/hello-dk
```
### 进入容器操作
```
docker exec -it  f69b2d0a6f22 /bin/sh
docker exec -it  f69b2d0a6f22 bash
docker exec -it  f69b2d0a6f22 sh
```