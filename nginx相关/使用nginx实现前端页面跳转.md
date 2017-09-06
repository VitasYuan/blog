## 1.安装Brew
在Mac系统下使用软件包管理工具Brew安装nginx，首先打开brew官网首页 <a>https://brew.sh/</a>，复制安装Brew命令在终端执行。
如下图所示：
![](https://github.com/VitasYuan/Blog/blob/master/pictures/nginx-1-2.png)
安装完成后，在终端输入brew命令，如控制台打印出以下内容，则安装成功：

    Example usage:
      brew search [TEXT|/REGEX/]
      brew (info|home|options) [FORMULA...]
      brew install FORMULA...
      brew update
      brew upgrade [FORMULA...]
      brew uninstall FORMULA...
      brew list [FORMULA...]

    Troubleshooting:
      brew config
      brew doctor
      brew install -vd FORMULA

## 2.使用Brew安装nginx
在终端输入：brew install nginx命令，如下图所示：
![](https://github.com/VitasYuan/Blog/blob/master/pictures/nginx-1-1.png)
控制台提示安装成功后，完成安装。所有nginx安装文件在  

    /usr/local/etc/nginx
目录下。

## 3.nginx常用命令

启动 nginx -c [configpath]
-c后面是配置文件地址，如使用默认配置文件可以直接使用nginx启动。

停止：
1.查询nginx进程，然后停止此进程使用  

    ps -ef|grep nginx

命令查询nginx进程id，如下图所示：
![](https://github.com/VitasYuan/Blog/blob/master/pictures/nginx-1-3.png)
2.使用kill命令杀死此进程
使用以下三种命令可以将此进程停止：  

    kill -QUIT 39729
    kill -TERM 39729
    pkill -9 nginx
如下图所示：  
![](https://github.com/VitasYuan/Blog/blob/master/pictures/nginx-1-4.png)

3.重启
进入nginx目录：输入nginx -s reload命令即可重启，如下图所示：
![](https://github.com/VitasYuan/Blog/blob/master/pictures/nginx-1-6.png)

## 4.nginx配置
使用brew安装nginx后默认的配置文件在：

    /usr/local/etc/nginx
切换到此目录，使用vim nginx.conf命令，可以看到默认的配置如下：  

    server {
      listen       8080;
      server_name  localhost;
      location / {
         root   html;
         index  index.html index.htm;
      }

通过此配置文件可以看到，nginx监听端口为8080，首页为index.html
在浏览器中输入localhost:8080可以看到如下页面，则说明，nginx已经正确安装并启动。
![]()
通过修改location中的配置为我们前端工程的目录，即可通过nginx来访问前端页面。
TODO：前端工程配置测试待添加....
