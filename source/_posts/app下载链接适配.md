# app 下载链接适配

### 背景

- app 的推广页需要有一个下载链接，方便用户点击的时候直接下载 app，但是不同的系统下载的地方不一样；

### 方案

通过 userAgent 判断具体是哪个系统，是否在微信内打开，然后通过调整 respones 解决；

1. 通过 node 服务
2. 通过 Nginx

以上两种方式都可以解决这个问题，基于一些环境原因，选择用 Nginx 的方式去解决

### Nginx

```nginx

server {
    listen    80;
    server_name  www.a.com a.com;
    charset utf-8;


    location /applink {
        #微信浏览器
        if ($http_user_agent ~* "micromessenger" ){
                rewrite ^/.*    https://a.com/ulink/action/  redirect;
        }
        if ($http_user_agent ~* "android"){
                rewrite ^/.*    https://a.com/apk/appname.apk redirect;
        }

        if ($http_user_agent ~* "iphone"){
                rewrite ^/.*    https://apps.apple.com/cn/app/ redirect;
        }
        rewrite ^/.*    http://lizhifm.cn/d  redirect;
    }

}

```
