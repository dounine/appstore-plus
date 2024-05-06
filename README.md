# ipa-app
> ipa-app 是一个基于网页在线安装ipa的工具，支持iOS 9.0以上版本，支持ipa文件的在线安装，可分享，可扫码安装，支持黑白主题。


# 安装引导
> 依赖`docker-compose`进行管理，操作前请确保已安装[docker](https://docs.docker.com/engine/install/)和[docker-compose](https://docs.docker.com/compose/install/linux)。

依赖 [docker-compose.yml](./docker-compose.yml) 文件
> 请根据文件内容自行修改配置，如数据库密码、端口等。
```shell
```yaml
version: '3.3'
services:
  db:
    image: postgres:16.2-alpine3.19
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=ipa-db
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=ROOTAbc123
  api:
    image: dounine/ipa-api:latest
    depends_on:
      - db
    environment:
      - database_url=postgresql://root:ROOTAbc123@db:5432/ipa-db #帐号密码请与上面db服务一致
      - domain=http://localhost:3000  # 前端地址，例如：https://app.ipadump.com
      - release=true  # 是否为生产环境
      - log=error   # 日志级别
      - locale=zh-CN   # 默认语言
      - admin_username=admin   # 管理员账号(PS:请修改默认账号密码)
      - admin_password=admin   # 管理员密码(PS:请修改默认账号密码)
      - form_body_limit=10mb  # post body 限制(不建议修改)
      - file_upload_block_size=5mb  # 文件上传分片大小(不建议修改)
      - ipa_file_limit=1gb    # ipa文件大小限制
      - clean_interval=1m    # 定时清理任务时间(不建议修改)
      - ipa_sign_file_expire=360m  # ipa签名过期时间,m分钟
      - ipa_file_save_time=60m   # ipa保存时间,无关联的ipa文件将被删除
      - ipa_file_temp_expire=60m  # ipa上传临时文件,超时将被删除
      - ipa_sign_storage_limit=10gb  # ipa签名存储总大小
      - ipa_sign_plugin_limit=10mb  # ipa签名插件限制大小
      - ipa_sign_plugin_count_limit=10  # ipa签名插件数量限制
      - ipa_icon_limit=5mb   # ipa图标大小限制
    volumes:
      - ./data/app_file:/app/file
      - ./data/app_logs:/app/logs
  node:
    image: dounine/ipa-node:latest
    depends_on:
      - api
    ports:
      - "3000:3000"  # 请根据实际情况修改冒号左边的端口，如有占用，请使用其它未使用的端口
```

创建`data`目录，用于存放数据库和ipa文件数据。
> 目录的位置自行决定，不可手动删除，否则程序可能会异常。
```shell
mkdir ./data
```
运行`docker-compose`启动服务
```shell
docker-compose up -d
```
访问`http://localhost:3000`即可访问应用，或者对应的域名加端口。

**服务命令行反向代理配置**
```shell
server {
    listen       443 ssl;
    server_name  sign.demo.com;
    ssl_certificate /etc/nginx/conf.d/ssl/sign.demo.com.pem;
    ssl_certificate_key /etc/nginx/conf.d/ssl/sign.demo.com.key;

    location / {
	proxy_read_timeout 1000;
        proxy_pass http://127.0.0.1:3000;
        add_header 'Access-Control-Allow-Origin' '*';
    	add_header 'Access-Control-Allow-Methods' 'GET,POST,PUT,DELETE,PATCH,OPTIONS';
    	add_header 'Access-Control-Allow-Headers' '*';
    	add_header 'Access-Control-Allow-Credentials' 'true';
 	client_max_body_size 50m;
        proxy_set_header  X-Real-IP  $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
}
server {
    listen 80;
    server_name sign.demo.com;
    rewrite ^(.*)$ https://$host$1 permanent;
}
```
或者使用宝塔自行将域名映射到3000端口即可。

## 更新引导
> 不同版本的更新可能会有不同的更新步骤，请根据实际情况进行操作。

停止服务
```shell
docker-compose stop
```
拉取最新镜像
```shell
docker-compose pull
```
删除旧容器
```shell
docker-compose rm
```
启动服务
```shell
docker-compose up -d
```

