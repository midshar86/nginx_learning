# `nginx`的学习

## 在`window`系统下使用`nginx`部署静态资源(`vue`项目)

- 将`vue`项目的路由模式改为`hash`模式
- 在`vue`项目中执行`npm run build`将完成的项目进行打包成为静态资源
- 项目的文件将被存放在`dist`目录中

### 下载安装

- 在`nginx`官网下载稳定版本的安装包,并且解压到文件夹
- 点击`.exe`可执行文件,当屏幕出现黑色闪屏后表示启动成功
  - 使用`start nginx`开启`nginx`服务
  - ==只有执行`start nginx`后才可以执行以下三个命令==
  - 使用`./nginx.exe -s quit`正常退出
  - 使用`./nginx.exe -s stop`强制退出
  - 使用`./nginx.exe -s reload`重新加载服务,一般用作在`nginx.conf`配置文件修改后
- `nginx`的静态资源文件存放在解压后的同级`html`文件夹中,将打包后的`vue`资源`dist`目录中的文件拖放到该文件夹中
- `nginx`的配置文件存放在解压后的同级`conf`文件夹中,`nginx.conf`文件是配置文件
- 修改`nginx.conf`文件
  - 修改`location`

```shell
location /
{
  root   html; # 根路径文件夹
  index  index.html index.htm; # 网页文件的路径
  try_files $uri $uri/ /index.html; # 配置路由模式下的url重定向
}
location /index/api
{
  proxy_pass  http://api.avatardata.cn/; #当访问中有api时，则访问这个我们需要跨域的地址
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
