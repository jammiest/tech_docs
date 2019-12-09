#

## 文档在线预览

 - <http://tech.03pl.com/#/>
 - <https://jammiest.github.io/tech_docs/#/>


## 文档系统本地部署

参考：<https://docsify.js.org/#/zh-cn/quickstart?id=初始化项目>

```shell
# 进入目录

# 安装工具
npm i docsify-cli -g

# 开启服务
docsify serve
```

本地访问 <http://localhost:3000>

## 服务器部署

参考：<https://docsify.js.org/#/zh-cn/deploy>

nginx 的配置

```nginx
server {
  listen 80;
  server_name  your.domain.com;

  location / {
    alias /path/to/dir/of/docs;
    index index.html;
  }
}
```

