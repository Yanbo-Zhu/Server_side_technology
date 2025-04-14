
In diesem kleinen Unterkapitel zum _**nginx**_ wird Ihnen vieles aus der Apache-Konfiguration vertraut vorkommen, nur in schlankerer Form. Auf der Debian9-Distribution ist der nginx nicht vorinstalliert, die Installation ist aber sehr einfach. Zuerst sollte der Apache-Webserver mit` _systemctl stop apache2_` gestoppt werden. Mit `apt-get install nginx` wird dann der nginx installiert und gestartet.

Die Dokumentation für den nginx ist unter [https://nginx.org/en/docs/](https://nginx.org/en/docs/) zu finden.


**nginx Webserver**: Ein schlanker, eventbasierter Webserver mit viel geringeren Konfigurationsmöglichkeiten und ohne die schwergewichtigen Workerprozesse. Somit ist der nginx **sehr performant**. Oftmals werden nginx und Apache-Webserver auch gemeinsam eingesetzt. Dabei übernimmt der nginx auf einem Server alle statischen Ressourcen (Bilder, CSS-Dateien etc.) und leitet nur die "Spezialanfragen" an einen anderen Server weiter (als Load-Balancer), auf dem dann ein Apache läuft. So kann man in einer "Server-Farm" die hohe Performance des nginx und die gute Flexibilität des Apache nutzen.


Nginx ist ein leistungsstarker und vielseitiger Webserver, der hauptsächlich für folgende Zwecke eingesetzt wird:

1. Webserver: Er liefert statische Inhalte wie HTML, CSS und Bilder schnell und effizient an die Clients.
2. Reverse Proxy: Nginx leitet Anfragen an andere Server weiter, beispielsweise an Backend-Anwendungen wie Node.js, und fungiert dabei als Vermittler.
3. Load Balancer: Er verteilt eingehende Anfragen gleichmäßig auf mehrere Backend-Server, um Lastspitzen zu bewältigen und die Verfügbarkeit zu erhöhen.
4. Gateway für APIs: Nginx kann verwendet werden, um Anfragen an verschiedene Microservices zu koordinieren.
5. Caching: Er speichert Inhalte zwischengespeichert, um die Serverleistung zu verbessern.


# 1 Einsatzzweck und Unterschiede zum Apache

Der nginx ist die schlanke und performante Alternative zum Apache-Webserver, bei der vieles ähnlich zu sein scheint.

erhält sich der nginx Webserver aufgrund seiner eventbasierten Arbeitsweise

**Sicherheitshinweis**  
Auch für den nginx - wie für jeden Webserver - gilt, dass die Standardkonfiguration aus Sicherheitsgründen nicht im produktiven Betrieb eingesetzt werden darf, sondern entsprechend den Anforderungen sorgfältig angepasst werden muss.


**nginx Webserver starten und stoppen**  
Ob der nginx läuft, kann mit _**ps aux | grep nginx**_ ermittelt werden. [Im Gegensatz zum Apache](https://isp.eduloop.de/loop/Der_Apache_Webserver "Der Apache Webserver") sehen wir dann nur zwei Prozesse: einen Managerprozess und einen Bearbeiterprozesse, da der nginx eventbasiert alle Anfragen mit einem Prozess entgegen nimmt.

Mit _**systemctl start nginx**_ und _**systemctl stop nginx**_ wird der Webserver gestartet und gestoppt.

  
**Direkte Programmoptionen für den nginx**  
So wie beim [Apache Webserver](https://isp.eduloop.de/loop/Direkte_Programmoptionen_f%C3%BCr_apache2 "Direkte Programmoptionen für apache2"), gibt es auch beim nginx Parameter, die direkt in der Linux-Konsole eingegeben werden können:

- _**nginx -v**_ ermittelt die installierte Version (z.B. nginx/1.10.3)
- _**nginx -h**_ zeigt einen Überblick über die Optionen

  
**Module und Direktiven**  
Ebenso, wie beim Apache, gibt es [Module und Direktiven](https://nginx.org/en/docs/). Dabei fällt auf, dass es weniger Module gibt und diese in der Regel mit weniger Direktiven auskommen.

**Verzeichnisstruktur**  
Die Verzeichnisstruktur ist ähnlich der [Apache Verzeichnisstruktur](https://isp.eduloop.de/loop/Die_Verzeichnisstruktur_des_apache2 "Die Verzeichnisstruktur des apache2"). Die Hauptkonfigurationsdatei heißt _**nginx.conf**_ und befindet sich im Verzeichnis _**/etc/nginx**_. Die Debian-Verzeichnisstruktur ist hier sehr gut erläutert: [https://wiki.debian.org/Nginx/DirectoryStructure](https://wiki.debian.org/Nginx/DirectoryStructure)

[![Apache10-1nginx.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/b/b1/Apache10-1nginx.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/b/b1/Apache10-1nginx.png)



Auch beim nginx sind Konfigurationsanweisungen ausgelagert und befinden sich beispielsweise in einem Unterverzeichnis _modules-enabled_.

[![Apache10-2nginx.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/a/aa/Apache10-2nginx.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/a/aa/Apache10-2nginx.png)

Aber auch wenn die Struktur ähnlich aussieht, so verhält sich der nginx Webserver aufgrund seiner eventbasierten Arbeitsweise deutlich anders und ist auch anders zu konfigurieren.


# 2 Konfiguration des nginx

Die Konfiguration des nginx ist insgesamt etwas übersichtlicher als die Konfiguration des Apache Webservers. Zusätzlich gibt es auch eine Reihe von [Standardkonfigurationen für häufig vorkommende Anwendungsszenarien](https://www.nginx.com/resources/wiki/start/) und [sehr gute Tipps, auch zum Thema Sicherheit](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/).



Original Hauptkonfigurationsdatei nginx.conf
```
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


---

sites-available/default 

Ein Hauptteil der Konfiguration befindet sich in der Datei _**sites-enabled/default**_. Wie beim Apache schon beschrieben, ist im Verzeichnis _sites-enabled/_ nur ein Link auf eine Datei _**default**_ im Verzeichnis _sites-available/_ vorhanden. Somit wurden genau genommen die Daten aus der Datei _sites-available/default_ verwendet.


```
# Default server configuration
#
server {
	listen 80 default_server;
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


# 3 Beispiel Zugangsschutz nginx


Wir hatten den [Apache Webserver mit einem einfachen Zugangsschutz](https://isp.eduloop.de/mediawiki/index.php?title=Beispiel_Zugansschutz&action=edit&redlink=1 "Beispiel Zugansschutz (Seite nicht vorhanden)") versehen. Diesen Zugangsschutz wollen wir nun auch beim nginx einreichten. Beim Apache hatten wir hierfür folgende Einträge verwendet:

```
<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride None
    AuthType Basic
    AuthName "Mein ZugangsSchutz"
    AuthUserFile "/etc/apache2/htpass"
    Require user IhrUserName
</Directory>
```

  
Beim nginx erreichen wir einen einfachen Zugangsschutz mit folgenden Zeilen (in der Datei **default**):

```
location / {
    auth_basic           "Mein ZugangsSchutz";
    auth_basic_user_file /etc/nginx/htpass;
}
```

  
Natürlich müssen wir auch beim nginx die Datei _**htpass**_ erst erstellen. Dies machen wir mit einem Editor und schreiben in die Datei:

IhrUserName:IhrUserPW

Es fällt auf, dass das Passwort hier im Klartext in der Datei steht. Aber es ist auch möglich, die [htpasswd-Funktionalität](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html) zu verwenden.






