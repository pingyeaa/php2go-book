# Nginx反向代理负载均衡的容器化部署

首先，在`home`目录创建`microservices`目录，开启第一篇章。

```bash
cd ~ && mkdir microservices && cd microservices
```

创建`nginx`目录，在目录下分别创建三个节点目录：`nginx01`、`nginx02`、`nginx03`，目的是使`nginx01`作为反向代理服务器，将请求均衡转发到`nginx02`、`nginx03`。

```bash
mkdir -p ./nginx/nginx01 ./nginx/nginx02 ./nginx/nginx03
```

展示效果如下所示。

```
nginx
├── nginx01
└── nginx02
└── nginx03
```

将nginx镜像中的配置文件拷贝到各子目录中，以便做挂载，方法是创建一个临时容器，将配置文件拷贝至宿主机目录，再删除临时容器。

```bash
docker run --name tmpnginx -d nginx:latest
docker cp tmpnginx:/etc/nginx/nginx.conf ~/microservices/nginx/nginx01
docker cp tmpnginx:/etc/nginx/nginx.conf ~/microservices/nginx/nginx02
docker cp tmpnginx:/etc/nginx/nginx.conf ~/microservices/nginx/nginx03
docker cp tmpnginx:/etc/nginx/conf.d ~/microservices/nginx/nginx01
docker cp tmpnginx:/etc/nginx/conf.d ~/microservices/nginx/nginx02
docker cp tmpnginx:/etc/nginx/conf.d ~/microservices/nginx/nginx03
docker rm -f tmpnginx
```

此时nginx目录如下所示。

```
nginx
├── nginx01
│   ├── conf.d
│   │   └── default.conf
│   └── nginx.conf
├── nginx02
│   ├── conf.d
│   │   └── default.conf
│   └── nginx.conf
└── nginx03
    ├── conf.d
    │   └── default.conf
    └── nginx.conf
```

在根目录创建文件`docker-compose.yml`，创建三个web服务，配置文件分别映射到容器中的对应文件。

```yml
version: '3'

services:
  web01:  #服务名称
    image: nginx:latest #镜像
    container_name: web01 #容器名称
    ports:  #映射端口号，前者宿主机端口，后者容器端口
      - 8080:80
    volumes: #映射的目录或文件，前者宿主机目录，后者容器目录
      - ./nginx/nginx01/nginx.conf:/etc/nginx/nginx.conf #配置文件
      - ./nginx/nginx01/conf.d:/etc/nginx/conf.d #扩展配置目录
      - ./nginx/html:/usr/share/nginx/html #html存放目录

  web02:
    image: nginx:latest
    container_name: web02
    volumes:
      - ./nginx/nginx02/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/nginx02/conf.d:/etc/nginx/conf.d
      - ./nginx/html:/usr/share/nginx/html

  web03:
    image: nginx:latest
    container_name: web03
    volumes:
      - ./nginx/nginx03/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/nginx03/conf.d:/etc/nginx/conf.d
      - ./nginx/html:/usr/share/nginx/html
```

打开`nginx/nginx01/conf.d/default.conf`，在文章顶部加入`upstream`配置，web02与web03是`docker-compose.yml`中定义的容器名称`container_name`。

```bash
upstream backend {
    server web02:80;
    server web03:80;
}
```

在`location /`中加入`proxy_pass`以便将请求转发给`backend`。

```bash
location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    proxy_pass http://backend;  #追加该行
}
```

配置完成后，执行以下命令将容器跑起来。

```bash
cd ~/microservices
docker-compose up
```

提示以下内容即成功。

```bash
Recreating microservices_web01_1 ... done
Recreating microservices_web02_1 ... done
Recreating microservices_web03_1 ... done
Attaching to web02, web01, web03
```

此时`microservices`目录结构如下，nginx目录下多出了一个`html`文件夹，可以在`html`目录下创建一个`index.html`，输入`Hello world!`，重新跑一下。

```bash
microservices
├── docker-compose.yml
└── nginx
    ├── html
    │   └── index.html
    ├── nginx01
    │   ├── conf.d
    │   │   └── default.conf
    │   └── nginx.conf
    ├── nginx02
    │   ├── conf.d
    │   │   └── default.conf
    │   └── nginx.conf
    └── nginx03
        ├── conf.d
        │   └── default.conf
        └── nginx.conf
```

现在做个测试，在浏览器中访问`localhost:8080`，观察终端打印的日志。

```bash
web01    | 172.24.0.1 - - [26/Jun/2019:01:48:28 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36" "-"
web02    | 172.24.0.2 - - [26/Jun/2019:01:48:28 +0000] "GET / HTTP/1.0" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36" "-"
```

上述内容表示本次请求通过web01转发到了web02。

```bash
web01    | 172.24.0.1 - - [26/Jun/2019:04:42:36 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36" "-"
web03    | 172.24.0.2 - - [26/Jun/2019:04:42:36 +0000] "GET / HTTP/1.0" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36" "-"
```

再次刷新，可以看到请求通过web01转发到了web03，到目前为止，基本的负载均衡部署就已经完成了，上述的web01是将请求均衡转发到web02、web03的，这种方法叫轮询法，下篇文章介绍几种其他的负载算法。

---
如果你是PHP从业者，又对Golang感兴趣
欢迎加入开源书籍项目，共同完成《PHPer玩转Golang》
Github入口: [EnochZg/php2go-book](https://github.com/EnochZg/php2go-book)