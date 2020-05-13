#### 部署Gitlab
```shell
docker run -d -p 20080:80 -p 20022:22 \
  --name gitlab --restart=unless-stopped \
  --hostname gitlab.frank.com \
  -v /home/frank/volumes/gitlab/config:/etc/gitlab \
  -v /home/frank/volumes/gitlab/logs:/var/log/gitlab \
  -v /home/frank/volumes/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:rc
```
配置好hosts，即可访问gitlab.frankcom

#### go get 获取代码
#### 生成证书
为了让go get可以获取代码，我们需要使用https，自签证书部分查看参考资料[《实现局域网https域名访问内网服务》](https://www.ucloud.cn/yun/6832.html)

##### 配置Nginx https代理
```
server {
  listen    80;
  listen       443 ssl;
  server_name  [gitlab.frank.com;](http://gitlab.frank.com;/)
  charset utf-8;
  access_log  /var/log/nginx/gitlab.access.log;
  error_log  /var/log/nginx/gitlab.error.log;
  ssl on;
  ssl_certificate         /etc/nginx/ssl/server.crt;
  ssl_certificate_key     /etc/nginx/ssl/server.key;
  ssl_session_timeout     10m;
  ssl_session_cache       shared:SSL:10m;

  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_pass    http://gitlab.frank.com:20080;
  }

  location ~.*\.(js|css|png)$ {
      proxy_pass  http://gitlab.frank.com:20080;
  }
}

```

##### 其他方案
上述部署成功以后，go get 便可以访问到gitlab.frank.com中的代码了。还有一种方案是使用git config 将https替换为ssh，我尝试下git clone 没有问题，但是go get依赖去访问https的方案，所以只能老老实实自签http是证书。