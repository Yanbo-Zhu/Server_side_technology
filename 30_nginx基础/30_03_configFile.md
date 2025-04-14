

# 1 docker image nignx 

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
# 2 例子 


## 2.1 proxy_pass


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






