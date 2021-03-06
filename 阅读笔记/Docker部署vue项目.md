- [Docker部署vue项目(原文地址)](https://juejin.im/post/5cce4b1cf265da0373719819)

- [win10如何安装docker](https://www.runoob.com/docker/windows-docker-install.html) 

docker常用命令

```javascript
// 查看当前镜像列表
docker image ls 
// 删除容器
docker rmi ed9c93747fe1  //ed9c93747fe1 镜像的ID
// 查看当前所有容器
docker ps
// 删除容器命令
docker rm 3e0d8f771e90  //3e0d8f771e90 容器的ID
// 进入一个容器
docker exec -it 561e6a1657b8 /bin/bash
// 查看当前容器的ip
cat /etc/hosts   // 或者  docker inspect 02277acc3efc

// 停用全部运行中的容器:
docker stop $(docker ps -q)
// 删除全部容器
docker rm $(docker ps -aq)
```

运行脚本：

```javascript
// 项目目录在这里 win10系统 E:\github\vue-demo

// 创建两个后端服务
docker run -p 5000:8080 -d --name nodeserver nodewebserver

docker run -p 5001:8080 -d --name nodeserver1 nodewebserver

// 静态资源容器  dist目录挂在，以及nginx目录挂在
docker run -p 3000:80 -d --name vuenginxnew --mount type=bind,source=/e/github/vue-demo/nginx,target=/etc/nginx/conf.d --mount type=bind,source=/e/github/vue-demo/dist,target=/usr/share/nginx/html nginx

```

nginx配置如下

```javascript
// 负载均衡服务器
upstream backend {
    server 172.17.0.4:8080;
    server 172.17.0.3:8080;
}
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    access_log  /var/log/nginx/host.access.log  main;
    error_log  /var/log/nginx/error.log  error;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    location /api/ {
        rewrite  /api/(.*)  /$1  break;
        proxy_pass http://backend;   // 代理到负载均衡
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

最后附上源码地址：https://gitee.com/liangshaojie/dockerVueDemo