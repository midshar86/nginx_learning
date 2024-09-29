# `nginx`的学习

## 在`window`系统下使用`nginx`部署静态资源(`vue`项目)

- 将`vue`项目的路由模式改为`hash`模式
- 在`vue`项目中执行`npm run build`将完成的项目进行打包成为静态资源
- 项目的文件将被存放在`dist`目录中

### 使用说明

- 在`nginx`官网下载稳定版本的安装包,并且解压到文件夹
- 点击`.exe`可执行文件,当屏幕出现黑色闪屏后表示启动成功
  - 使用`start nginx`开启`nginx`服务
  - ==只有执行`start nginx`后才可以执行以下三个命令==
  - 使用`./nginx.exe -s quit`正常退出
  - 使用`./nginx.exe -s stop`强制退出
  - 使用`./nginx.exe -s reload`重新加载服务,一般用作在`nginx.conf`配置文件修改后
  - 使用`taskkill /f /t /im nginx.exe`利用`window`系统指令关闭`nginx`
- `nginx`的静态资源文件存放在解压后的同级`html`文件夹中,将打包后的`vue`资源`dist`目录中的文件拖放到该文件夹中
- `nginx`的配置文件存放在解压后的同级`conf`文件夹中,`nginx.conf`文件是配置文件
- 修改`nginx.conf`文件
  - 修改`location`

```shell
# 访问网页html资源
location / # 斜杠将会匹配到下方index.html之后,这里也可以配置额外的路径 /other,也会追加到下方的index.html之后
{
  root   html; # 根路径文件夹
  index  index.html; # 网页文件的路径
  try_files $uri $uri/ /index.html; # 配置路由模式下的url重定向
}
location /index/api
{
  proxy_pass  http://api.avatardata.cn/; #当访问中有api时，则访问这个我们需要跨域的地址
}

# 访问静态图片文件
server
{
  listen       888; # 监听的端口号
  server_name  localhost; # 域名
  location /list/ # 改路径用来代替alias设置的路径,当访问http://localhost:888/list/pic.jpg时会替换成为下方的路径
  {
    alias  C:/Users/Vitam/Desktop/nginx-1.26.2/imgs/pic/; # 绝对路径
    autoindex on;
  }
}
```

- - 修改错误页面

```shell
error_page   500 502 503 504  /50x.html;
location = /50x.html # 错误页面的路径
{
    root   html; # 根路径文件夹
}
```

- - 配置访问端口及域名

```shell
listen       80; # 监听端口号
server_name  localhost; # 监听的域名
```

- 配置完成后,执行`./nginx.exe -s reload`重新加载配置文件
- 访问`localhost:80`即可浏览网页
- 如果更改配置文件后刷新网页无法生效,使用`taskkill /f /t /im nginx.exe`关闭后重启`nginx.exe`程序

### 详细配置

- `location`的配置规则
  - `location /`一般的配置规则
  - `location = /`精确匹配到`=`后面的路径或文件,无法匹配到其他路径资源
  - `location ~* \.[GIF|jpg|jpeg|png]`匹配正则表达式,`*`表示不区分大小写,只要路径中存在图片格式的文件就能匹配
    - 如果希望能够匹配成功,目录中需要存在一份对应格式的文件,匹配成功后,会被自动转换成为`.gif`
  - `location ~ \.[GIF|jpg|jpeg|png]`匹配正则表达式,去掉`*`表示匹配确定的文件,不会被转换为小写格式
  - `location ^~ /path/subpath`只能匹配指定目录下的文件
- 在`nginx`中配置解决跨域
  - 在`server`中配置

```shell
# 允许跨域请求
add_header 'Access-Control-Allow-Origin' *;
# 允许带上cookie请求
add_header 'Access-Control-Allow-Credentials' 'true';
# 允许请求的方法,可以设置post,get,fetch,delete,patch
add_header 'Access-Control-Allow-Methods' *;
# 允许请求的header
add_header 'Access-Control-Allow-Headers' *;
```

- 在`nginx`中配置防盗链
  - 在`server`中进行配置

```shell
# 对源站点验证
valid_referers *.yourOwnOrigin.com;
# 非法引用会返回错误状态码
if($invalid_referer){
  return 404/403;
}
```

- 在`nginx`中配置反向代理与负载均衡

```shell
# 配置上游代理服务器
upstream nginx_boot
{
  # 最简单的负载均衡配置,设置权重比,数值越大代表越有可能被该服务器处理
  # 30s内检查心跳发送两次包,未回复就代表该机器宕机,请求分发权重比为1:2
  server 192.168.0.000:8080 weight=100 max_fails=2 fail_timeout=30s;
  server 192.168.0.000:8090 weight=200 max_fails=2 fail_timeout=30s;
  # 这里的IP请配置成你WEB服务所在的机器IP
}

# 配置反向代理
server
{
  listen  80; # 监听浏览器端的端口号
  server_name localhost; # 设置为你的域名
  location ~
  {
    # root   html;
    # 配置一下index的地址，最后加上index.ftl。
    # index  index.html index.htm index.jsp index.ftl;
    # proxy_set_header Host $host;
    # proxy_set_header X-Real-IP $remote_addr;
    # proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # 请求交给名为nginx_boot的upstream上,反向代理
    proxy_pass http://nginx_boot;
  }
}
```
