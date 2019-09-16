---
title: nginx location匹配规则
key: 20190916
tags: nginx
---

基本的配置语法为：

``` xml
location [=|~|~*|^~]  *uri*  { 
	...
}
```



-  `~` 正则表达式,大小写敏感；
- `~*`正则表达式, 大小写不敏感；
- `^~`可不是正则的意思,后面会有解释；
- `=`  精确匹配

### （I）路径匹配规则简要描述

#### 匹配路径按以下规则进行:

1. 先序遍历所有路径前缀, 把匹配的规则中*最长的那个*记录下来. 然后*按顺序*遍历所有正则, 第一个正则匹配成功之后就停下来. 如果没有正则可以成功匹配, 选用最长长度的前缀路径.

2. 如果*最长的*前缀路径有 ^~ 修饰符, 直接选用这个, 1步骤里面的正则匹配就不用了.

3. 在第一步遍历路径前缀时, 如果前缀路径有 = 修饰符, 则停止继续寻找,直接使用这个.

4. uri前面加一个@, 叫做 named location, 普通的请求不管他, 只用在一些内部的跳转上, 比如下面这样

   ```xml
   location / {
        error_page 404 = @fallback;
    }
   
    location @fallback {
        proxy_pass http://backend;
    }
   ```

5. 如果一个前缀路径最后以/结尾, 而且请求被proxy_pass, fastcgi_pass, uwsgi_pass, scgi_pass, or memcached_pass中的一个处理, 这个情况比较特殊: 请求如果不以/结尾, 会返回一个301的重定向响应, 再最后加上/. 如果不想这样, 就要用 = 修饰符做精确匹配, 像下面这样

   ```
   location /user/ {
        proxy_pass http://user.example.com;
    }
    location = /user {
        proxy_pass http://login.example.com;
    }
   ```

#### proxy_pass中的路径变换

我们用nginx curl python -m SimepleHTTPServer来做测试

- 如果proxy_pass含有URI,那么把标准化之后的Url中匹配location的那部分换成proxy_pass指令中的URI.

  ```
  server {
        listen 8080;
        server_name proxy-pass-test-1.localhost;
  
        location /name/ {
            proxy_pass http://127.0.0.1:8000/remote/;
        }
    }
  
    curl proxy-pass-test-1.localhost:8080/name/abcd/xyz
    127.0.0.1 - - [12/Jun/2017 14:16:32] "GET /remote/abcd/xyz HTTP/1.0" 404 -
  ```

- 如果proxy_pass没有URI, 那么完整的URI全部传递过去.

  ```xml
  server {
        listen 8080;
        server_name proxy-pass-test-2.localhost;
  
        location /name/ {
            proxy_pass http://127.0.0.1:8000;
        }
    }
  
    curl proxy-pass-test-2.localhost:8080/name/abcd/xyz
    127.0.0.1 - - [12/Jun/2017 14:23:18] "GET /name/abcd/xyz HTTP/1.0" 404 -
  ```

有些情况下, 无法判断被替换的部分, 比如:

- location是正则匹配的, 或者是使用的named location

  这种情况下, proxy_path不能使用URI

- URI在内部被rewrite改变了, proxy_pass中的URI被忽略,完整的改变后的URI被传递.

  ```
  server {
        listen 8080;
        server_name proxy-pass-test-3.localhost;
  
        location /name/ {
            rewrite    /name/([^/]+) /users?name=$1 break;
            proxy_pass http://127.0.0.1:8000;
        }
    }
  
    server {
        listen 8080;
        server_name proxy-pass-test-4.localhost;
  
        location /name/ {
            rewrite    /name/([^/]+) /users?name=$1 break;
            proxy_pass http://127.0.0.1:8000/remote/;
        }
    }
  
    curl -i proxy-pass-test-3.localhost:8080/name/childe
    127.0.0.1 - - [12/Jun/2017 14:36:29] "GET /users?name=childe HTTP/1.0" 404 -
  
    curl -i proxy-pass-test-4.localhost:8080/name/childe
    127.0.0.1 - - [12/Jun/2017 14:36:35] "GET /users?name=childe HTTP/1.0" 404 -
  ```

- proxy_pass中使用变量

  ```
  server {
        listen 8080;
        server_name proxy-pass-test-5.localhost;
  
        location /name/ {
            proxy_pass http://127.0.0.1:8080/$request_uri;
        }
    }
    curl -i proxy-pass-test-5.localhost:8080/name/abcd/xyz
    127.0.0.1 - - [12/Jun/2017 14:41:02] "GET /name/abcd/xyz HTTP/1.0" 404 -
  ```

### （II）路径匹配规则详情

#### 一、精确匹配

语法示例：

> location = /static/img/file.jpg {
>
> ...
>
> }

#### 二、前缀匹配

1、普通前缀匹配

语法示例：

> location /static/img/ {
>
> ...
>
> }

2、优先前缀匹配

语法示例：

> location ^~/static/img/ {
>
> ...
>
> }

#### 三、正则匹配

1、区分大小写

语法示例：

> location ~ /static/img/.*\.jpg$ {
>
> ...
>
> }

2、不区分大小写

语法示例：

> location ~* /static/img/.*\.jpg$ {
>
> ...
>
> }

3、区分大小写取反

语法示例：

> location !~ /static/img/.*\.jpg$ {
>
> ...
>
> }

4、不区分大小写取反

语法示例：

> location !~* /static/img/.*\.jpg$ {
>
> ...
>
> }

#### 四、优先级

对于请求： http://example.com/static/img/logo.jpg

1、如果命中精确匹配，例如：

> location = /static/img/logo.jpg {
>
> }

则优先精确匹配，并终止匹配。

2、如果命中多个前缀匹配，例如：

> location /static/ {
>
> }
>
> location /static/img/ {
>
> }

则记住最长的前缀匹配，即上例中的 /static/img/，并继续匹配

3、如果最长的前缀匹配是优先前缀匹配，即：

> location /static/ {
>
> }
>
> location ^~ /static/img/ {
>
> }

则命中此最长的优先前缀匹配，并终止匹配

4、否则，如果命中多个正则匹配，即：

> location /static/ {
>
> }
>
> location /static/img/ {
>
> }
>
> location ~* /static/ {
>
> }
>
> location ~* /static/img/ {
>
> }

则忘记上述 2 中的最长前缀匹配，使用第一个命中的正则匹配，即上例中的 location ~* /static/ ，并终止匹配（命中多个正则匹配，优先使用配置文件中出现次序的第一个）

5、否则，命中上述 2 中记住的最长前缀匹配



### 附录

[Nginx location match tester](https://nginx.viraptor.info/)