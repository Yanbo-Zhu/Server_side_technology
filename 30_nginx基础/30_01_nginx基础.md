
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






