[使用 docker-compose 构建你的项目](https://juejin.im/post/5e10163ce51d45410706539f#heading-8)

## 原文中出现的问题：

### 命令错误

```javascript
// 原来命令
docker--compose inspect nginx-node
// 改为
docker inspect nginx-node
```
### 启动多个容器 名字占用问题

运行docker-compose up -d --scale node=5 

不能指定这个名字了
container_name: node-server-1 # 容器名称

### 练习代码地址

最后附上地址：https://gitee.com/liangshaojie/docker-compose-test.git