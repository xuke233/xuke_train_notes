# nginx 学习笔记
- nginx 介绍
    1. Nginx (“engine x”) 是一个高性能的 HTTP 和 反向代理 服务器，也是一个 IMAP/POP3/SMTP 代理服务器。
        Nginx特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好。
        Nginx由内核和模块组成，其中，内核的设计非常微小和简洁，完成的工作也非常简单，
        仅仅通过查找配置文件将客户端请求映射到一个location block（location是Nginx配置中的一个指令，用于URL匹配），
        而在这个location中所配置的每个指令将会启动不同的模块去完成相应的工作

- nginx windows安装
    1. 下载软件
       在[nginx官网](http://nginx.org/en/download.html)下载nginx的稳定版本1.18.0(截至2021-02-15)，下载下来为zip压缩包
       解压压缩包，将解压后的压缩包放置你需要的文件夹下面
    2. 启动nginx
       执行nginx-1.18.0文件夹下面的nginx.exe文件
       或者"win+R",输入"cmd"，点击回车，切换文件夹至nginx-1.18.0文件夹，输入"start nginx"
    3. 测试是否安装成功
        打开浏览器，输入"http:localhost:80",出现"Welcome to nginx"的首页，即为安装成功

- nginx 常用指令
  | 指令 | 功能 |
  |:--:|:--:|
  | nginx -v | 查看当前nginx版本 |
  | nginx -V | 查看nginx版本详细信息 |
  | start nginx | 启动nginx |
  | nginx -s stop | 强制关闭nginx（不推荐） |
  | nginx -s quit | 正常关闭nginx（推荐） |
  | nginx -s reload | 重新加载配置文件 |
  | nginx -c XX.conf | 指定配置文件 |
  
- nginx.conf文件解读
    1. nginx.conf文件
       ```
        #user  nobody;
        worker_processes  1;
        
        #error_log  logs/error.log;
        #error_log  logs/error.log  notice;
        #error_log  logs/error.log  info;
        
        #pid        logs/nginx.pid;

        events {
        worker_connections  1024;
        }
        
        http {
        include       mime.types;
        default_type  application/octet-stream;

            #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
            #                  '$status $body_bytes_sent "$http_referer" '
            #                  '"$http_user_agent" "$http_x_forwarded_for"';
        
            #access_log  logs/access.log  main;
        
            sendfile        on;
            #tcp_nopush     on;
        
            #keepalive_timeout  0;
            keepalive_timeout  65;
        
            #gzip  on;
        
            server {
                listen       80;
                server_name  localhost;
        
                #charset koi8-r;
        
                #access_log  logs/host.access.log  main;
        
                location / {
                    root   html;
                    index  index.html index.htm;
                }
        
                #error_page  404              /404.html;
        
                # redirect server error pages to the static page /50x.html
                #
                error_page   500 502 503 504  /50x.html;
                location = /50x.html {
                    root   html;
                }
        
                # proxy the PHP scripts to Apache listening on 127.0.0.1:80
                #
                #location ~ \.php$ {
                #    proxy_pass   http://127.0.0.1;
                #}
        
                # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
                #
                #location ~ \.php$ {
                #    root           html;
                #    fastcgi_pass   127.0.0.1:9000;
                #    fastcgi_index  index.php;
                #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
                #    include        fastcgi_params;
                #}
        
                # deny access to .htaccess files, if Apache's document root
                # concurs with nginx's one
                #
                #location ~ /\.ht {
                #    deny  all;
                #}
            }
        
            # another virtual host using mix of IP-, name-, and port-based configuration
            #
            #server {
            #    listen       8000;
            #    listen       somename:8080;
            #    server_name  somename  alias  another.alias;
        
            #    location / {
            #        root   html;
            #        index  index.html index.htm;
            #    }
            #}
        
            # HTTPS server
            #
            #server {
            #    listen       443 ssl;
            #    server_name  localhost;
        
            #    ssl_certificate      cert.pem;
            #    ssl_certificate_key  cert.key;
        
            #    ssl_session_cache    shared:SSL:1m;
            #    ssl_session_timeout  5m;
        
            #    ssl_ciphers  HIGH:!aNULL:!MD5;
            #    ssl_prefer_server_ciphers  on;
        
            #    location / {
            #        root   html;
            #        index  index.html index.htm;
            #    }
            #}
        
        }
       ```
       
    2. '#' 开头的表示注释内容，我们去掉所有以'#'开头的段落，精简之后的内容如下：
       ```
       worker_processes  1;

        events {
        worker_connections  1024;
        }
        
        http {
        include       mime.types;
        default_type  application/octet-stream;
        
        
            sendfile        on;
        
            keepalive_timeout  65;
        
            server {
                listen       80;
                server_name  localhost;
        
                location / {
                    root   html;
                    index  index.html index.htm;
                }
        
                error_page   500 502 503 504  /50x.html;
                location = /50x.html {
                    root   html;
                }
        
            }
        
        }
       ```
       
    3. 全局块
       从配置文件开始到 events 块之间的内容，主要会设置一些影响nginx 服务器整体运行的配置指令，
       主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。
       例如第一行配置的：
       ```
       worker_processes  1;
       ```
       这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约，这个后面会详细介绍。
       
    4. events块
       events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，
       是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。
       例如：
       ```
       events {
          worker_connections  1024;
       }
       ```
       表示每个 work process 支持的最大连接数为 1024
       
    5. http块
       这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。
       http 块包括 http全局块、server 块
       
      1. http 全局块   
       http全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。
       
      2. server 块   
       这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。
       每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。
  
    6. server配置
       | 语句 | 解读 |
       | :--: | :--: |
       | listen 80 | 监听端口为80，可以自定义其他端口，也可以加上IP地址，如，listen 127.0.0.1:8080 |
       | server_name  localhost | 定义网站域名，可以写多个，用空格分隔 |
       | charset koi8-r | 定义网站的字符集，一般不设置，而是在网页代码中设置 |
       | access_log  logs/host.access.log  main | 定义访问日志，可以针对每一个server（即每一个站点）设置它们自己的访问日志 |
       | error_page  404  /404.html | 定义404.html |
       | error_page   500 502 503 504  /50x.html | 当状态码为500、502、503、504时，则访问50x.html |
       | location = /50x.html { root   html;  //定义50x.html所在路径 } | 定义50x.html所在路径 |
    7. location配置
       通过指定模式来与客户端请求的URI相匹配，基本语法如下：location [=|~|~*|^~|@] pattern{……}
       
       1. "没有修饰符"表示：必须以指定模式开始
          例如：
          ```
            location /abc{
            ......
          }
          ```
          那么，如下是对的：    
          http://baidu.com/abc   http://baidu.com/abc?p1   
          http://baidu.com/abc/   http://baidu.com/abcde
       2. "="表示：必须与指定的模式精确匹配 
       3. "~"表示：指定的正则表达式要区分大小写
       4. "~*"表示：指定的正则表达式不区分大小写
       5. "^~"类似于无修饰符的行为，也是以指定模式开始，不同的是，如果模式匹配 
       6. @：定义命名location区段，这些区段客户段不能访问，只可以由内部产生的请
       查找顺序和优先级   
       1. 带有“=“的精确匹配优先   
       2. 没有修饰符的精确匹配   
       3. 正则表达式按照他们在配置文件中定义的顺序   
       4. 带有“^~”修饰符的，开头匹配   
       5. 带有“~” 或“~*” 修饰符的，如果正则表达式与URI匹配   
       6. 没有修饰符的，如果指定字符串与URI开头匹配
      
    8. root 、alias指令区别
        ```
       location /img/ {
            alias /var/www/image/;
       }
        ```
       若按照上述配置的话，则访问/img/目录里面的文件时，nginx会自动去/var/www/image/目录找文件
       ```
        location /img/ {
            root /var/www/image;
        }
        ``` 
       若按照这种配置的话，则访问/img/目录下的文件时，nginx会去/var/www/image/img/目录下找文件
       alias是一个目录别名的定义，root则是最上层目录的定义   
       还有一个重要的区别是alias后面必须要用“/”结束，否则会找不到文件的。。。而root则可有可无~~

- nginx 搭建文件服务器
    1. 添加server块
       ```
        server {
            listen       8080 default_server;
            listen       127.0.0.1:8080 default_server;
            server_name  _;    
            charset      utf-8; # 中文名的文件不乱码
            root         C:\Users\kxg65\Desktop\upload; # 保存文件的路径
        }
        ```
    2. 启动nginx，打开浏览器，输入localhost:8080
   