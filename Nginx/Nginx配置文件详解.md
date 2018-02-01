### nginx配置文件详解
```
# 运行用户
user    www;

# 启动进程,通常设置成和cpu的数量相等,或设置为auto
worker_processes    1;

# 设置全局错误日志存储位置
# error_log     /var/wwwlogs/error.log;
# error_log     /var/wwwlogs/error.log  notice;
# error_log     /var/wwwlogs/error.log  info;

# 设置pid文件存储位置
# pid   /var/wwwlogs/nginx.pid;

# 工作模式及连接数上限
events {

    # epoll是多路复用IO(I/O Multiplexing)中的一种方式
    use   epoll; 

    # 单个后台worker process进程的最大并发链接数    
    worker_connections  1024;

    # 最大客户连接数，由于浏览器默认使用两个并发连接，注意：此值不是一个nginx参数！！！！！！！！！！！
    # 作为普通HTTP服务器
    # max_clients = worker_processes * worker_connections / 2
    # 作为反向代理服务器
    # max_clients = worker_processes * worker_connections / 4

}


http {

    # 设定mime类型,类型由mime.type文件定义
    include         mime.types;
    default_type    application/octet-stream;

    # 设定日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" ' '$status $body_bytes_sent "$http_referer" ' '"$http_user_agent" "$http_x_forwarded_for"';

    # 日志存储路径与格式
    access_log  /var/wwwlogs/access.log  main;

    # sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，
    # 对于普通应用，必须设为 on,
    # 如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，
    # 以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile        on;
    # tcp_nopush    on;

    # 连接超时时间
    keepalive_timeout   60;
    tcp_nodelay         on;

    # 开启gzip压缩
    gzip                on;
    gzip_min_length     1k;
    gzip_buffers        4 16k;
    gzip_http_version   1.0;
    gzip_comp_level     2;
    gzip_types          text/plain application/x-javascript text/css application/xml;
    gzip_vary           on;

    # 设定请求缓冲
    client_header_buffer_size    128k;
    large_client_header_buffers  4 128k;

    # 设定虚拟主机配置
    server {

        # 监听80端口
        listen    80;

        # 定义域名 www.test.com 多个使用空格隔开
        server_name  www.test.com;

        # 默认网站根目录位置
        root html;

        # 定义首页索引文件的名称
        index index.php index.html index.htm;

        # 设定本虚拟主机的访问日志
        access_log /var/wwwlogs/nginx.access.log    main;

        # 默认请求
        location / {
            # 定义首页索引文件的名称
            index   index.php index.html index.htm;
        }

        # 定义错误提示页面
        error_page  500 502 503 504 /50x.html;
        location = /50x.html {
        }

        # 静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {
            # 过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
            expires 30d;
        }

        # PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
        location ~ .php$ {
            fastcgi_pass    127.0.0.1:9000;
            fastcgi_index   index.php;
            fastcgi_param   SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include         fastcgi_params;
        }

        # 禁止访问 .tpl 文件
        location ~ /.tpl {
            deny    all;
        }

    }
}
```