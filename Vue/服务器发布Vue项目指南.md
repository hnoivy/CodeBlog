# 服务器发布Vue项目指南(更新中)
> 正在完善状态，还剩下Nginx。 Apache和Tomcat的配置亲测有效。
下面所有的例子`vue-router`的`history`模式下。

## 1：Apache服务器
1. *修改Apache默认配置*

首先要重新修改`\conf\httpd.conf`文件让文件支持`rewrite`
![引入模块](apacheconfig2.jpg '引入模块')
找到
```bash
// 这一行需要解开注释 引入这个模块
LoadModule rewrite_module modules/mod_rewrite.so
```

然后新增或者修改下面得代码
![修改重写支持](apacheconfig1.jpg '修改重写支持')
```bash
# 重写文件根目录
DocumentRoot "/usr/local/apache/demo"
# 目录
<Directory "/usr/local/apache/demo">
    #
    # Possible values for the Options directive are "None", "All",
    # or any combination of:
    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
    #
    # Note that "MultiViews" must be named *explicitly* --- "Options All"
    # doesn't give it to you.
    Options Indexes FollowSymLinks

    #
    # 修改未允许重写
    AllowOverride all

    #
    # Controls who can get stuff from this server.
    #
    Require all granted
</Directory>
```

2. *新增htacces规则*

上面操作时修改服务器得默认配置让服务器支持Rewrite，下面来创建Rewrite规则

首先在和`index.html`同级得地方新建`.htacces`文件，具体内容可以参照`Vue-Router`官网给得[例子](https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90)

```bash
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```
![.htacces文件](htac.png '.htacces文件')

### Apache开启Gzip压缩
```bash
/usr/local/apache/bin/apxs -i -c -n -a mod_deflate.so
```

## 2：Nginx服务器

## 3：Tomcat下SpringMVC中发布

这个发布到SpringMVC中的配置相对来说就比较简单了。
只需要在发布的文件夹中新增`WEB-INF`配置文件夹中就行。如下图
![tomcat中配置](tomcat.png 'tomcat中文件')

`WEB-INF` 文件夹放在项目中那么`tomcat`会自动扫描文件夹中的`web.xml`然后重写web配置
![tomcat中配置](webxml.png 'xml')

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
  version="3.0"
  metadata-complete="true">
  
  <!-- <display-name>webapp</display-name> -->
  <description>
    webapp
  </description>
  <error-page>  
    <!-- 重写404页面 -->
    <error-code>404</error-code>  
    <location>/index.html</location>
  </error-page>  
</web-app>
```

这就配置完了，可以说是贼简单了。

## Nuxt项目发布指南

发布`SSR`项目的时候官方推荐了两种方式**服务端渲染应用部署** 和 **静态应用部署**。静态应用部署的话基本上就失去了`SSR`的优势，而且部署方式也和上面讲的大同小异。这里着重来讲服务端渲染应用部署。

服务端渲染应用部署的话不同于静态部署，我们同时要在服务器上部署上`Node`环境。

### `Node环境部署`

1. cd到`/usr/local`文件夹并在这个文件夹下新建一个存放安装环境的文件夹`node`后进入`node`文件夹（名字可以随便起，这里做demo就用node了）
```bash
cd /usr/local
mkdir -p node
```
2. 下载`Node`我这里采用的是最新的长期服务版本，你想安装其他的版本，可以在[这里](https://nodejs.org/dist/)查找，还可以使用`nvm`来管理你的`Node`,简单[教程](https://www.jianshu.com/p/d0e0935b150a)
```bash
wget https://nodejs.org/dist/v10.13.0/node-v10.13.0-linux-x64.tar.gz
```
3. 下载完成之后解压并且为文件夹改名
```bash
# 解压到当前文件夹
tar -zxvf node-v10.13.0-linux-x64.tar.gz -C ./
# 改名
mv node-v10.13.0-linux-x64/ ./node10.13.0
```
4. 建立软连接，为`node,npm`注册环境变量(其他的`npm`全局包也要这样注册)
```bash
# 软链接指向到node npm
ln -s /usr/local/node/node10.13.0/bin/node  /usr/local/bin/node
ln -s /usr/local/node/node10.13.0/bin/npm  /usr/local/bin/npm 
```
5. 查看软链接建立是否成功
```bash
ls -al /usr/local/bin
```
显示如下就表示成功了

![node建立链接](./images/nodeinstall1.png 'node建立链接')
接着使用`node -v`
```bash
node -v
# 出现版本号就是安装成功了
v10.13.0
```

到这里为止我们的`Node`环境就安装成功了，接下来进行`nuxt`部署

### `Nuxt`部署
1. 全局安装`nuxt`，然后按照上面的方式建立软连接
```bash
# 安装
npm i nuxt -g
# 建立软连接
ln -s /usr/local/node/node10.13.0/bin/nuxt  /usr/local/bin/nuxt 
```
2. 代码部署

回到`/local`文件夹下，我们建立一个`nuxt`文件用来存放我的`nuxt`项目。然后进入`nuxt`文件夹
```bash
# 回到loacal
cd ../
mkdir nuxt
cd nuxt
```
在本地环境中执行`nuxt build`,然后会生成一个`.nuxt`文件夹。

然后修改`package.json`,为它加上新的内容，`nuxt`应用会根据下面的配置自动配置服务的端口号和地址。
```bash
"config": {
  "nuxt": {
    "host": "0.0.0.0", # 通过IPV4访问
    "port": xxxx
  }
},
```
然后将项目中的`.nuxt文件夹 static文件夹 package.json nuxt.config.js`上传到服务器的`nuxt`文件夹中，

然后在`nuxt`文件夹中执行
```bash
npm i
```
下载完成之后执行`nuxt start`出现下面的日志就表示成功启动了我们的`nuxt`应用

![成功启动](./images/nuxtstart.png '成功启动')

### `pm2`使用

既然使用了`Node`服务，那我们最好是使用[pm2](https://pm2.io/doc/en/runtime/overview/?utm_source=pm2&utm_medium=website&utm_campaign=rebranding)做进程守护，至于`pm2`是什么，以及`pm2`是干什么的，这篇文章我们不过多的赘述，有兴趣可以自己看看。

首先安装`pm2`
```bash
npm i pm2 -g
# 建立软连接
ln -s /usr/local/node/node10.13.0/bin/pm2  /usr/local/bin/pm2 
```

然后在`package.json`中加入`pm`启动指令或者直接像下面这样启动
```bash
# package.json
"scripts": {
    "pm2:nuxt": "pm2 start npm --name 'XXX' -- run nuxt:start", # 启动名字为xxx的进程
    "nuxt:start": "PORT=xxxx nuxt start",
    "start": "nuxt start",
    "generate": "nuxt generate",
    "pm2:stop:all": "pm2 stop all" # 停止所有进程

  }
# 直接启动命令
pm2 start npm --name 'XXX' -- run nuxt:start
```
出现这样的日志就表示成功

![成功启动](./images/pm2.png '成功启动')

下面是一些常用的`pm2`命令
```bash
pm2 start 0        # 启动 id为 0的指定应用程序
pm2 restart 0      # 重启 id为 0的指定应用程序
pm2 stop 0         # 停止 id为 0的指定应用程序
pm2 delete 0       # 删除 id为 0的指定应用程序

pm2 list           # 查看当前正在运行的进程
pm2 start all      # 启动所有应用
pm2 restart all    # 重启所有应用
pm2 stop all       # 停止所有的应用程序
pm2 delete all     # 关闭并删除所有应用
pm2 logs           # 控制台显示所有日志
 
pm2 logs 0         # 控制台显示编号为0的日志
pm2 show 0         # 查看执行编号为0的进程
pm2 monit xxx      # 监控名称为xxxx的进程
```

至此，一个`Nuxt`应用就算是部署完成了。

### 出现过的问题

1. 启动`koa server.js`的时候外网无法访问。
   
   这个问题的具体出现原因我暂时还没找到，只是后来换了`nuxt start`外网才可以顺利访问。这算是我自己提的一个`issue`。

2. 权限不足使用`sudo su`找不到`node npm`等命令
   
   这个很简单， 我们再重新建立一边软链接就行了
  ```bash
  sudo ln -s /usr/local/bin/node /usr/bin/node
  sudo ln -s /usr/local/lib/node /usr/lib/node
  sudo ln -s /usr/local/bin/npm /usr/bin/npm
  ```
