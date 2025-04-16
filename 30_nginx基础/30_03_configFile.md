

Die Konfiguration des nginx ist insgesamt etwas übersichtlicher als die Konfiguration des Apache Webservers. Zusätzlich gibt es auch eine Reihe von [Standardkonfigurationen für häufig vorkommende Anwendungsszenarien](https://www.nginx.com/resources/wiki/start/) und [sehr gute Tipps, auch zum Thema Sicherheit](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/).

# 1 **Verzeichnisstruktur**  

Die Verzeichnisstruktur ist ähnlich der [Apache Verzeichnisstruktur](https://isp.eduloop.de/loop/Die_Verzeichnisstruktur_des_apache2 "Die Verzeichnisstruktur des apache2"). Die Hauptkonfigurationsdatei heißt _**nginx.conf**_ und befindet sich im Verzeichnis _**/etc/nginx**_. Die Debian-Verzeichnisstruktur ist hier sehr gut erläutert: [https://wiki.debian.org/Nginx/DirectoryStructure](https://wiki.debian.org/Nginx/DirectoryStructure)

[![Apache10-1nginx.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/b/b1/Apache10-1nginx.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/b/b1/Apache10-1nginx.png)


# 2 Syntax 


## 2.1 location block 

在 Nginx 中，location 块是用来匹配客户端请求的 URI并定义如何处理这些请求的。

```
location [匹配类型] 匹配规则 {
    # 处理规则，比如 proxy_pass、root、rewrite、fastcgi_pass 等
}
```

| 匹配类型                | 描述                            |
| ------------------- | ----------------------------- |
| `location = /uri`   | **严格匹配**请求的 URI（完全相等）         |
| `location /uri`     | 默认匹配（前缀匹配），如果没有其他更精确的匹配，就使用这个 |
| `location ^~ /uri`  | 如果前缀匹配成功，就**立即使用**这个，不再检查正则匹配 |
| `location ~ regex`  | 使用**正则表达式**匹配（大小写敏感）          |
| `location ~* regex` | 使用**正则表达式**匹配（不区分大小写）         |

优先级规则（从高到低）

1. `=` 精确匹配    
2. `^~` 前缀匹配
3. `~` 和 `~*` 正则匹配（按顺序） 
4. 普通前缀匹配（最后一个匹配到的）

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html;

    # 精确匹配
    location = / {
        return 200 "This is the homepage";
    }

    # 前缀匹配
    location /images/ {
        autoindex on;
    }

    # 忽略正则，优先这个匹配
    location ^~ /static/ {
        root /var/www/assets;
    }

    # 区分大小写的正则匹配
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    # 忽略大小写的正则匹配
    location ~* \.(jpg|png|gif|css|js)$ {
        expires 30d;
        access_log off;
    }
}
```

# 3 /etc/nginx/nginx.conf

nginx.conf 是 Nginx 的主配置文件，它位于 /etc/nginx/nginx.conf，控制着整个 Nginx 服务的行为，比如进程数量、日志、包含哪些子配置、HTTP/Stream 配置等。

|区块|说明|
|---|---|
|`user`|指定运行 Nginx 的系统用户（通常为 `www-data`）|
|`worker_processes`|设置工作进程数量（`auto` 表示自动根据 CPU 核心数）|
|`events`|定义与连接处理相关的设置（如最大连接数）|
|`http`|HTTP 服务的核心设置区域|
|`include`|引入其他配置文件（如 `sites-enabled` 目录下的虚拟主机配置）|
|`access_log`, `error_log`|指定日志路径|
|`gzip`|启用响应压缩，提高传输效率|


Original Hauptkonfigurationsdatei nginx.conf

```nginx
# 全局设置
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 1024;  # 每个工作进程最大连接数
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # 日志定义
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # 压缩设置
    gzip on;

    # 引入其他虚拟主机配置
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```


```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;
	gzip_disable "msie6";

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json ...;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```


# 4 /etc/nginx/sites-available and /etc/nginx/sites-enabled

/etc/nginx/sites-available 这是一个 可用站点配置文件夹。
每个网站/服务的配置通常放在一个独立的文件中，比如：
```
/etc/nginx/sites-available/example.com
/etc/nginx/sites-available/myapp.local
```
这些文件本身不会生效，除非被链接到另一个目录 /etc/nginx/sites-enabled。



 /etc/nginx/sites-enabled 这是**启用的站点配置目录**。
- 这个目录中的每个配置文件通常是 `sites-available` 中文件的**符号链接**（symbolic link）。
	- sudo ln -s /etc/nginx/sites-available/my_site /etc/nginx/sites-enabled/
- Nginx 只加载 `sites-enabled` 中的配置文件。
- 禁用一个站点的方法
	- sudo rm /etc/nginx/sites-enabled/my_site and sudo systemctl reload nginx

开发者或服务器管理员通常会：
1. 为每个站点单独创建配置文件（在 `sites-available` 中）
2. 通过 `ln -s` 命令控制启用哪些站点
3. 保持配置结构清晰、便于管理



## 4.1 sites-available/default 


Ein Hauptteil der Konfiguration befindet sich in der Datei _**sites-enabled/default**_. Wie beim Apache schon beschrieben, ist im Verzeichnis _sites-enabled/_ nur ein Link auf eine Datei _**default**_ im Verzeichnis _sites-available/_ vorhanden. Somit wurden genau genommen die Daten aus der Datei _sites-available/default_ verwendet.


```
# sites-available/default
# Default server configuration
#
server {   # 这是一个服务器块（server block），用于定义一个网站（虚拟主机）的配置。
	listen 80 default_server;  # 指定这个虚拟主机监听 80 端口（默认 HTTP 端口）。
	listen [::]:80 default_server;

	# SSL configuration
	#
	# listen 443 ssl default_server;
	# listen [::]:443 ssl default_server;
	#
	# Note: You should disable gzip for SSL traffic.
	# See: https://bugs.debian.org/773332
	#
	# Read up on ssl_ciphers to ensure a secure configuration.
	# See: https://bugs.debian.org/765782
	#
	# Self signed certs generated by the ssl-cert package
	# Don't use them in a production server!
	#
	# include snippets/snakeoil.conf;

	root /var/www/html;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}

	# pass PHP scripts to FastCGI server
	#
	#location ~ \.php$ {
	#	include snippets/fastcgi-php.conf;
	#
	#	# With php-fpm (or other unix sockets):
	#	fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
	#}

	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	#location ~ /\.ht {
	#	deny all;
	#}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#	listen 80;
#	listen [::]:80;
#
#	server_name example.com;
#
#	root /var/www/example.com;
#	index index.html;
#
#	location / {
#		try_files $uri $uri/ =404;
#	}
#}
```


### 4.1.1 解释 



```
server {  # 这是一个服务器块（server block），用于定义一个网站（虚拟主机）的配置。
    listen 80;   # 指定这个虚拟主机监听 **80 端口**（默认 HTTP 端口）。
    server_name localhost;   # 指定这个服务器响应的域名，这里是 localhost，表示只响应本地请求。

    root /var/www/html;   # 设置网站根目录，网页文件（如 `index.html`、`index.php`）会从这个目录中查找。
    index index.php index.html;   # 设置默认的首页文件，优先顺序是：`index.php` → `index.html`。


	# try_files：按顺序尝试访问
	# $uri：请求的路径（如 /about.html）
	# $uri/：目录（如 /about/）
	# 如果都找不到，则返回 404 Not Found
    location / {
        try_files $uri $uri/ =404;
    }

    # PHP 支持部分
    location ~ \.php$ {   # ~ \.php$：正则匹配所有以 .php 结尾的请求
        include snippets/fastcgi-php.conf;   # 引入 FastCGI 相关配置，比如 SCRIPT_FILENAME 等
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;  # 注意 PHP 版本.  把请求交给 PHP-FPM 处理. 注意这里的路径 /run/php/php8.1-fpm.sock 是 PHP-FPM 提供的 Socket 文件（用于 Nginx 与 PHP 通信）
    }

    location ~ /\.ht {  
        deny all; # 阻止访问 .htaccess 或其他以 .ht 开头的敏感文件，防止泄露服务器配置。
    }
}
```

# 5 docker image nignx 

```sh
 ⚡ 🦄  docker exec -it nginx-container /bin/bash
root@c72ce2a429fb:/# cat /etc/nignx/nignx.conf
cat: /etc/nignx/nignx.conf: No such file or directory

root@c72ce2a429fb:/# cd /etc/nginx/

root@c72ce2a429fb:/etc/nginx# ll
bash: ll: command not found

root@c72ce2a429fb:/etc/nginx# ls
conf.d  fastcgi_params  mime.types  modules  nginx.conf  scgi_params  uwsgi_params

root@c72ce2a429fb:/etc/nginx# cat nginx.conf

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}





```



```sh
root@docker-desktop:/# cd /etc/nginx/conf.d/
root@docker-desktop:/etc/nginx/conf.d# ls
my_site.conf
root@docker-desktop:/etc/nginx/conf.d# cat my_site.conf 
server {
    listen       80;
    server_name  localhost;

    location /blog_board_frontend {
        proxy_pass http://localhost:9999;
    }

    location /blog_board_backend {
        proxy_pass http://localhost:8080;
    }

}root@docker-desktop:/etc/nginx/conf.d# 
```


```sh
root@9f3456b26a85:/etc/nginx/conf.d# cat default.conf 
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
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
```
# 6 例子 


## 6.1 proxy_pass


![](image/Pasted%20image%2020250312141221.png)

Die Anwendung funktioniert wie folgt:
1. Um die Anwendung zu nutzen, verbinden sich Clients mit dem nginx Webserver, welcher auf dem HTTP-Standardport 80 läuft und mittels proxy_pass auf die Startseite des Live-Servers mit der Vue-Anwendung weiterleitet.
2. Durch Klick auf den Button „Get Flights“ wird eine GET Anfrage mit den gewählten Suchparametern an den nginx Webserver geschickt. Diese Anfrage wird mittels proxy_pass an den Node.js Anwendungsserver weitergeleitet.
3. Der Anwendungsserver extrahiert die Suchparameter aus der Anfrage und schickt entsprechende Anfragen an die in den Parametern spezifizierten Provider.
4. Die Antworten der Provider werden vom Anwendungsserver aggregiert und über den nginx Reverse Proxy an den Client zurückgegeben, woraufhin die Ergebnisse im Browser angezeigt werden.


vue 中写code
```js
fetch(/flights/..)   就会直接转发到 http://localhost:3000 去

function getFlights() {
    let params = '';
    let url = '/flights';
    
    for(const airline of airSelected.value) {
        params += (airline.toLowerCase() + '=true&');
    }
    
    params += 'dep=' + depSelected.value;
    url += '?' + params;
    
    console.log('Fetching', url);

    //TODO: GET the results from the application server and pass them to the parent component
    fetch(url)
        .then(res => res.json())
        .then(res => emit('results', res))  // emit 对应的 funtion 为 newResults => results = newResults  这个 lambda 表达式.  newResults 为 function 的 参数, results 为 function 的返回值
        .catch(() => emit('results', []));
}
```



---

Die Zeile listen 80; in der Datei bedeutet, dass der Proxy-Server auf Port 80 läuft.
Die URL im Browser von http://localhost:8080 auf http://localhost:80 oder http://localhost ändern! (location / { proxy_pass http://localhost:8080 })
In der nächsten Aufgabe werden Anfragen an /flights gesendet. Mit dem Block location /flights { proxy_pass http://localhost:3000; } wird dem Proxy-Server mitgeteilt, dass er sich in diesem Fall mit einem Server auf Port 3000 kommunizieren soll.


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
      listen 80;

      location / {
       # Vue-based Frontend
        proxy_pass http://localhost:8080;
      }

      location /flights {
       # Node-based Backend
        proxy_pass http://localhost:3000;
      }
    }  

    include servers/*;
}
```



或者 

Starten Sie nun den nginx-Server und testen Sie ihre Anwendung, indem Sie mittels Postman Anfragen an die URL http://localhost:8080 senden. Sie sollten nun die selben Antworten wie zuvor unter Port 3000 erhalten.

```
server {
    listen 8080;
    location / {
        proxy_pass http://localhost:3000;
    }
}
```






