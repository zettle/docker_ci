# docker

## 在本地window运行看

启动后台服务器
1. 安装mongo，创建名为 `taro` 的数据库
2. 修改 `/backend/models/conf.js` 的配置，链接win本地数据库`mongodb://127.0.0.1:27017`
3. 启动后台服务 `node server.js`，端口3000

启动前端服务
1. 前端是用taro开发，一直启动不成功，就先用里面dist编译好的代码用
2. 前端请求后台接口，是用`/api/xxx`的方式请求的。所以在前端nginx上做好代理，指到3000端口上
3. 商品图片是放在 `/static` 里面，所以nginx再做个代理，把 `jpg/png/jpeg` 等格式映射到 `/static` 文件里面
4. 启动服务
- 如果用了http-server工具，可以执行命令`http-server --proxy http://127.0.0.1:3000`来看前端实现的效果
- 如果win上有nginx，可以用下面配置
```conf
server {
    listen       80;

    location /api/ {
        proxy_pass     http://127.0.0.1:3000/api/;
        proxy_redirect     off;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
  
    location / {
        root  E:/workspace/项目路径/frontend/dist;
        index  index.html index.htm;
    }

    location ~ \.(gif|jpg|png)$ {
        root E:/workspace/项目路径/static;
        index index.html index.htm;
    }
}
```

![](./readmeImg/xiaoguo.png)


## 代码上传到linux服务器
把整个项目上传到centos服务器的 `/root/svr/docker-cli` 路径下（node_modules）的都不要删除，其他原封不动都上传上去。


## 编写docker任务
1. 把nginx的配置保存到`/nginx/conf.d/docker.conf`，内容如下
```conf
server {
    listen       80;
  
    location / {
        root  /var/www/html;
        index  index.html index.htm;
    }

    location ~ \.(gif|jpg|png)$ {
        root /static;
        index index.html index.htm;
    }
}
```

在`/docker-compose.yml`编写nginx的docker任务，内容如下
```yml
version: '3.1'
services:
  nginx:
    restart: always
    image: nginx
    ports:
      - 8091:80
    volumes:
      - ./nginx/conf.d/:/etc/nginx/conf.d
      - ./frontend/dist:/var/www/html/
      - ./static/:/static/
```
