# appstore-plus
> appstore-plus 是一个基于网页在线安装ipa的工具，支持iOS 9.0以上版本，支持ipa文件的在线安装，可分享，可扫码安装，支持黑白主题，高性能多线程异步非阻塞签名打包，全网速度最快，没有之一。

[界面预览](./preview/README.md)

[![QQ群](https://img.shields.io/badge/QQ%E7%BE%A4-628633891-blue)](http://qm.qq.com/cgi-bin/qm/qr?_wv=1027&k=kPEQ7s8Tq_efdpfWI7JicqunBOdCtCKK&authKey=CjmL2v2EbiezEUXGBJFJHSb2mv0jt%2BE4K3Zt0R%2FWturqrVYSmENSiQHLg89xocnW&noverify=0&group_code=628633891)


# 1. 宝塔方式安装
[点击查看](./宝塔安装教程.md)
# 2. 纯命令方式安装
> 依赖`docker-compose`进行管理，操作前请确保已安装[docker](https://docs.docker.com/engine/install/)和[docker-compose](https://docs.docker.com/compose/install/linux)。

依赖 [docker-compose.yml](./docker-compose.yml) 文件
> 请根据文件内容自行修改配置，如数据库密码、端口等。

```yaml
version: '3.3'
services:
  db:
    image: postgres:16.2-alpine3.19
    restart: always
    volumes:
      - /data/db:/var/lib/postgresql/data  # 数据库存储目录，默认创建在/data/postgres保存，请使用绝对位置。
    environment:
      - TZ=Asia/Shanghai  # 时区(其它时区请自行修改)
      - POSTGRES_DB=ipa-db
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=ROOTAbc123
  api:
    image: dounine/ipa-api:latest
    restart: always
    depends_on:
      - db
    environment:
      - database_url=postgresql://root:ROOTAbc123@db:5432/ipa-db #帐号密码请与上面db服务一致，默认不会暴露到外部(安全)，如有需要请自行修改
      - domain=http://localhost:3000  # 对外访问地址，必需修改成自己的https域名，没有端口，例如修改成：https://app.ipadump.com
      - TZ=Asia/Shanghai  # 时区(其它时区请自行修改)
      - log=error   # 日志级别(不要修改)
      - locale=zh-CN   # 默认语言(不要修改)
      - admin_username=admin   # 管理员账号(PS:请修改默认账号密码)
      - admin_password=admin   # 管理员密码(PS:请修改默认账号密码)
      - form_body_limit=10mb  # post body 限制(不要修改)
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
      - /data/app_file:/app/file  # ipa文件存储目录，默认创建在/data/app_file保存，请使用绝对位置。
      - /data/app_logs:/app/logs  # 日志存储目录，默认创建在/data/app_logs保存，请使用绝对位置。
    ports:
      - "3000:3000"  # 请根据实际情况修改冒号左边的端口，如有占用，请使用其它未使用的端口
```
### 必看说明
> domain：参数必须修改成自己的https域名，否则就算能签名也不能将ipa应用安装到ios手机上，例如修改成：https://app.ipadump.com

## 安装步骤
1. 下载上面的配置文件并保存到服务器任意位置
```shell
wget https://raw.githubusercontent.com/dounine/appstore-plus/main/docker-compose.yml
```
运行`docker-compose`
```shell
docker-compose up -d
```
访问`http://你的服务器ip:3000`即可访问网站，或者你设置好后的域名。

反向代理配置参考
```shell
server {
    listen       443 ssl;
    server_name  app.ipadump.com;
    ssl_certificate /etc/nginx/conf.d/ssl/app.ipadump.com.pem;
    ssl_certificate_key /etc/nginx/conf.d/ssl/app.ipadump.com.key;

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
    server_name app.ipadump.com;
    rewrite ^(.*)$ https://$host$1 permanent;
}
```
后台管理地址：[http://你的服务器ip:3000/zh-CN/admin](http://你的服务器ip:3000/zh-CN/admin)

## 版本更新
> 不同版本的更新可能会有不同的更新步骤，请根据实际情况进行操作。

1. 停止服务
```shell
docker-compose stop
```
2. 拉取最新镜像
```shell
docker-compose pull
```
3. 删除旧容器
```shell
docker-compose rm
```
4. 启动服务
```shell
docker-compose up -d
```
