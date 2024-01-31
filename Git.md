# Git

配置代理 使用以下命令配置代理地址和端口号：

~~~
git config --global http.proxy http://代理地址:端口号
~~~

如果你的代理需要账号密码验证，可以使用以下命令：

~~~
git config --global http.proxy http://用户名:密码@代理地址:端口号
~~~

同时配置 HTTP 和 HTTPS 代理，可以使用以下命令：

~~~
git config --global http.proxy http://代理地址:端口号

git config --global https.proxy https://代理地址:端口号
~~~

查看代理：

~~~
git config --global --get http.proxy
git config --global --get https.proxy

~~~

取消代理 使用以下命令取消代理：

~~~
git config --global --unset http.proxy

git config --global --unset https.proxy
~~~

查看全局配置

~~~
 git config --global --list  
~~~

