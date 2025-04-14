
# 1 Strukturierung der Konfigurationsdatei

Wir konnten bisher sehen, dass der Apache Webserver viele Module und Konfigurationseinstellungen besitzt. Schon alleine die Hauptkonfigurationsdatei _**apache2.conf**_ bietet viele Möglichkeiten, die hier nicht alle behandelt werden sollen.

Damit die apache2.conf in der Struktur lesbarer wird, wurden die Kommentarzeilen entfernt und es wurde eine kleine Umsortierung vorgenommen. Auch wurde anstelle des letzten Eintrags `IncludeOptional sites-enabled/*.conf `die darin enthaltenen Daten direkt in die apache2.conf kopiert und die Kommentare entfernt. Wie im letzten Kapitel schon beschrieben, ist im Verzeichnis `sites-enabled/ `nur ein Link auf eine Datei 000-default.conf im Verzeichnis `sites-available/`. Somit wurden genau genommen die Daten aus der Datei `sites-available/000-default.conf` verwendet.

https://www.a-coding-project.de/ratgeber/apache/grundkonfiguration 

Umstrukturierte apache2.conf und 000-default.conf"
```
#####################################################
# Globale Konfigurationseinstellungen
#####################################################
DefaultRuntimeDir ${APACHE_RUN_DIR}
PidFile ${APACHE_PID_FILE}

Timeout 300
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5

User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}

HostnameLookups Off
Include ports.conf


#####################################################
# Externe Module und Konfigurationen laden
#####################################################
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf
IncludeOptional conf-enabled/*.conf

AccessFileName .htaccess
<FilesMatch "^\.ht">
	Require all denied
</FilesMatch>


#####################################################
# Globale Einstellungen für Logdateien
#####################################################
ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn
LogFormat "%h %l %u %t \"%r\" %>s %O" common


#####################################################
# Globale Einstellungen für Verzeichnisse und Dateien
#####################################################
<Directory />
	Options FollowSymLinks
	AllowOverride None
	Require all denied
</Directory>

<Directory /usr/share>
	AllowOverride None
	Require all granted
</Directory>

<Directory /var/www/>
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>


#####################################################
# Spezielle Einstellungen für Websites laden
#####################################################
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log common
</VirtualHost>

Man könnte nun die Konfiguration noch weiter reduzieren und weitere unnötige Module vermeiden. Der Apache Webserver braucht aber zumindest ein MPM-Modul. Somit ist das Modul mpm_prefork auf jeden Fall notwendig. 
```

# 2 Konfiguration des Moduls MPM prefork

In der Konfigurationsdatei des Moduls mpm-prefork stehen für den Betrieb des Webservers wichtige Einträge. 

```
Datei mpm-prefork.conf ohne Kommentare

<IfModule mpm_prefork_modules>
    StartServers              5
    MinSpareServers           5
    MaxSpareServers          10
    MaxRequestWorkers       150
    MaxConnectionsPerChild    0
</IfModule>
```



Parameter werden in der Apache-Konfiguration als Direktiven bezeichnet.

|                                                                                                            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [<IfModule mpm_prefork_modules>](http://httpd.apache.org/docs/2.4/en/mod/core.html#ifmodule)               | Hiermit wird überprüft, ob das entsprechende Modul geladen wird. Die Direktiven würden ansonsten zu Fehlermeldungen führen, da diese nur für das entsprechende Modul gelten.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| [StartServers](http://httpd.apache.org/docs/2.4/en/mod/mpm_common.html#startservers) 5                     | Anzahl der Bearbeiterprozesse des Servers, die beim Start erstellt werden. Wir hatten im Kapitel [Der Apache Webserver](https://isp.eduloop.de/loop/Der_Apache_Webserver "Der Apache Webserver") gesehen, dass beim Aufruf von _ps aux_ fünf Bearbeiterprozesse zur Verfügung stehen. Hinweis: der Default-Wert muss bei produktiven Systemen erhöht werden.                                                                                                                                                                                                                                                                                                                                                                                                       |
| [MinSpareServers](http://httpd.apache.org/docs/2.4/en/mod/prefork.html#minspareservers) 5                  | Minimale Anzahl der unbeschäftigten Bearbeiterprozesse des Servers. Wenn viele gleichzeitige Client Requests erfolgen, erhöht der Managerprozess die Anzahl der offenen Bearbeiterprozesse, damit sofort Kapazität für die Beantwortung weiterer Client Requests vorhanden ist. Hinweis: der Wert kann gleich dem Wert von _StartServers_ sein.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| [MaxSpareServers](http://httpd.apache.org/docs/2.4/en/mod/prefork.html#maxspareservers) 10                 | Maximale Anzahl der unbeschäftigten Bearbeiterprozesse des Servers. Wenn, nach einer hohen Anzahl von gleichzeitigen Client Requests, nicht mehr so viele Anfragen kommen, reduziert der Managerprozess die Anzahl der offenen Bearbeiterprozesse. Hinweis: der Wert sollte doppelt so hoch sein wie _MinSpareServers_.                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| [MaxRequestWorkers](http://httpd.apache.org/docs/2.4/en/mod/mpm_common.html#maxrequestworkers) 150         | Maximale Anzahl der Bearbeiterprozesse, die zur Bedienung gleichzeitiger Client Requests zur Verfügung stehen. Der Wert sollte hoch sein, damit viele Client Requests bearbeitet werden können. Die maximale Größe hängt damit zusammen, wie viele Kindprozesse auf dem Server zugelassen sind. Um mehr als 256 gleichzeitige Bearbeiterprozesse zuzulassen, muss [ServerLimit](http://httpd.apache.org/docs/2.4/en/mod/mpm_common.html#serverlimit) als Direktive verwendet und erhöht werden. Auch muss geprüft werden, ob die Linux-Konfiguration eine hohe Anzahl an Kindprozessen zulässt. Bei einem Wert oberhalb von 200.000, muss der Apache neu kompiliert werden. Hinweis: der voreingestellte Wert von 150 wird bei Produktivsystemen nicht ausreichen. |
| [MaxConnectionsPerChild](http://httpd.apache.org/docs/2.4/en/mod/mpm_common.html#maxconnectionsperchild) 0 | Obergrenze für die Anzahl von Anfragen, die ein einzelner Kindprozess während seiner Laufzeit bearbeitet. Der Wert "0" bedeutet, dass die Kindprozesse nie erneuert werden. Dies kann bei einer schlechten Programmierung oder einer schlechten Konfiguration zur Folge haben, dass durch eine "memory leakage" die Prozesse immer größer werden. Hinweis: ein "memory leakage" kommt zwar selten vor, aber es ist ratsam, den Wert auf eine hohe Ziffer (beispielsweise 10.000) zu setzen, damit die einzelnen Arbeiterprozesse auch mal Feierabend bekommen.                                                                                                                                                                                                     |
| _</IfModule>_                                                                                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|                                                                                                            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |


# 3 Section Direktiven

Direktiv 隶属于某个 section 

Es gibt Direktiven, die in einem bestimmten Abschnitt (Section) ausgeführt werden. Eine Section-Direktive hatten wir bereits kennen gelernt: `<IfModule>`.

Section-Direktive sind sehr wichtig, ==damit Einstellungen z.B. nur für ein bestimmtes Unterverzeichnis unter dem Dokumentenverzeichnis (DocumentRoot) gelten.==

Eine gute Erklärung, wann welche Section-Direktiven verwendet werden, findet sich in der [Apache-Dokumentation](http://httpd.apache.org/docs/2.4/de/sections.html).

Es gibt eine Reihe von Section-Direktiven, die Wichtigsten sind fett dargestellt:

![[Pasted image 20250414154717.png]]


|                                                                                        |                                                                                                                                                                                                                                          |
| -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| _**[<Directory>](http://httpd.apache.org/docs/2.4/de/mod/core.html#directory)**_       | Spezifiziert die Eigenschaften des angegebenen Verzeichnisses und steht in direktem Bezug zu den Ablageorten der Dateien im Dateisystem.                                                                                                 |
| _[<DirectoryMatch>](http://httpd.apache.org/docs/2.4/de/mod/core.html#directorymatch)_ | Verwenden von Regulären Ausdrücken für die Angabe der Verzeichnisse.                                                                                                                                                                     |
| _**[<Files>](http://httpd.apache.org/docs/2.4/de/mod/core.html#files)**_               | Spezifiziert die Eigenschaften der angegebenen Dateien.                                                                                                                                                                                  |
| _[<FilesMatch>](http://httpd.apache.org/docs/2.4/de/mod/core.html#filesmatch)_         | Definieren von Regulären Mustern für die Angabe der Dateien.                                                                                                                                                                             |
| _[<IfDefine>](http://httpd.apache.org/docs/2.4/de/mod/core.html#ifdefine)_             | Schließt Direktiven ein, die nur ausgeführt werden, wenn eine Testbedingung beim Start wahr ist.                                                                                                                                         |
| _**[<IfModule>](http://httpd.apache.org/docs/2.4/de/mod/core.html#ifmodule)**_         | Schließt Direktiven ein, die abhängig vom Vorhandensein oder Fehlen eines speziellen Moduls ausgeführt werden.                                                                                                                           |
| _[<Location>](http://httpd.apache.org/docs/2.4/de/mod/core.html#location)_             | Ähnlich zu <_Directory_>, aber ohne einen direkten Bezug zum Dateisystem. Lesen Sie mehr hierzu unter: [http://httpd.apache.org/docs/2.4/de/sections.html#file-and-web](http://httpd.apache.org/docs/2.4/de/sections.html#file-and-web). |
| _[<LocationMatch>](http://httpd.apache.org/docs/2.4/de/mod/core.html#locationmatch)_   | Ähnlich zu <_DirectoryMatch_>, aber ohne einen direkten Bezug zum Dateisystem.                                                                                                                                                           |
| _**[<VirtualHost>](http://httpd.apache.org/docs/2.4/de/mod/core.html#virtualhost)**_   | Angabe eines Bereiches für eine Domain.                                                                                                                                                                                                  |



Man sollte in der Konfigurationsdatei immer die obersten Verzeichnisse zuerst spezifizieren. Die Reihenfolge der Abarbeitung im Apache ist `_<Directory...>,_ _<DirectoryMatch...>_, _<Files>_, _<FilesMatch...>_, _<Location...>_ sowie _<LocationMatch...>_`.

Die verzeichnis- oder dateibezogenen Einstellungen werden in entsprechenden Bereichen angegeben. Die eingestellten Eigenschaften werden an die Unterverzeichnisse vererbt.

Es muss sehr genau darauf geachtet werden, bis zu welcher Stelle in der Konfigurations-Datei die einzelnen Abschnitte gültig sind.


# 4 Direktiven und Modulzuordnung

Beim Modul _**mpm-prefork**_ haben wir erkennen können, dass Direktiven immer einem, oder mehreren Modulen zugeordnet sind. Die Zuordnung kann man der Apache-Dokumentation entnehmen. Soll eine Direktive in einer Konfigurationsdatei verwendet werden, so muss das dazugehörige Modul eingebunden (inkludiert) sein.

```
Direktiven der Konfigdatei mpm-prefork.conf

<IfModule mpm_prefork_modules>
    StartServers              5
    MinSpareServers           5
    MaxSpareServers          10
    MaxRequestWorkers       150
    MaxConnectionsPerChild    0
</IfModule>
```

Für die Direktive [_**StartServers**_](https://httpd.apache.org/docs/2.4/de/mod/mpm_common.html#startservers) gibt es in der Apache-Dokumentation nachfolgende Abbildung:

[![Apache8-1a-StartServers.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/1/10/Apache8-1a-StartServers.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/1/10/Apache8-1a-StartServers.png)

 Abb. 40: Dokumentation ''StartServers''

Hieraus kann man erkennen, dass die Direktive _StartServers_ in unterschiedlichen Modulen eingesetzt werden kann.


---


Für die Direktive [_**MinSpareServers**_](https://httpd.apache.org/docs/2.4/de/mod/prefork.html#minspareservers) gibt es in der Apache-Dokumentation nachfolgende Abbildung:

[![Apache8-2a-MinSpareServers.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/8/8f/Apache8-2a-MinSpareServers.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/8/8f/Apache8-2a-MinSpareServers.png)



---

Für die Direktive [_**<IfModule ...> ... </IfModule>**_](https://httpd.apache.org/docs/2.4/de/mod/core.html#ifmodule) gibt es in der Apache-Dokumentation nachfolgende Abbildung:

[![Apache8-3a-ifModule.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/b/b9/Apache8-3a-ifModule.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/b/b9/Apache8-3a-ifModule.png)

 Abb. 42: Dokumentation ''IfModule''

Hieraus kann man erkennen, dass die Direktive _<IfModule ...> ... </IfModule>_ zum _core_-Modul gehört.

- Man hat also die Möglichkeit über die [Liste der Direktiven](https://httpd.apache.org/docs/2.4/de/mod/directives.html) das zugehörige Modul zu ermitteln.

- Man kann auch über die [Liste der Module](https://httpd.apache.org/docs/2.4/de/mod/) die hierzu passenden Direktiven erfahren.

Beide Listen sind ein wichtiges Handwerkszeug für die Konfiguration des Apache Webservers.


# 5 Beispiel

## 5.1 ServerTokens und ServerSignature

Am Beispiel der Direktive _**ServerTokens**_ kann das Zusammenspiel zwischen der Apache-Konfiguration und ==den Auswirkungen im HTTP-Header== sehr gut nachvollzogen werden.


---

Beispiel
Auf dem hier verwendeten Debian9-System befindet sich die Direktive _**ServerTokens**_ in _/etc/apache2/conf-available/security.conf_.
Im HTTP-Response-Header werden normalerweise Informationen zum Server mitgeliefert. Über [ServerTokens kann bestimmt werden, wie aussagekräftig diese Informationen sind](https://httpd.apache.org/docs/2.4/de/mod/core.html#servertokens). Diese Daten werden nur aus dem Grund gesendet, damit die Nutzungshäufigkeit des Apache-Webservers ermittelt werden kann.

**Empfehlung: ServerTokens Prod**

---


Aufgabe

Stellen Sie _**ServerTokens Full**_ ein und ermitteln Sie die Angaben im Server Response des HTTP-Headers. Sie können dann sehr gut erkennen, wie die Konfiguration des Webservers den HTTP-Header beeinflusst.

Anschließend stellen Sie _**ServerTokens Prod**_ ein und sollten einen deutlichen Unterschied erkennen können.

Hinweis: Wenn kein Unterschied zu erkennen ist, haben Sie entweder den Webserver nach den Konfigurationsänderungen nicht neugestartet, oder die Direktive wird in einer anderen config-Datei überschrieben.


Sie können die Auswirkungen der Einstellungen auch gut an Fehlerseiten erkennen.

[![Apache9-serverTokensOS.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/1/1b/Apache9-serverTokensOS.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/1/1b/Apache9-serverTokensOS.png)

 Abb. 45: ServerTokens OS

  

[![Apache9-serverTokensProd.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/7/78/Apache9-serverTokensProd.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/7/78/Apache9-serverTokensProd.png)

 Abb. 46: ServerTokens Prod

  
Mit [_**ServerSignature Off**_](https://httpd.apache.org/docs/2.4/de/mod/core.html#serversignature) werden keine Informationen mehr angezeigt.

[![Apache9-serverSignatureOff.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/c/c7/Apache9-serverSignatureOff.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/c/c7/Apache9-serverSignatureOff.png)

 Abb. 47: ServerSignature Off


---

Aufgabe

Ermitteln Sie, ob mit _**ServerSignature Off**_ noch Angaben über den Server im Server Response des HTTP-Headers stehen.

## 5.2 LimitRequest


Aus dem Kapitel HTTP wissen wir, dass ein Client Request einen HTTP-Body enthalten kann. Es gibt einige Apache-Direktiven, die im direkten Zusammenhang mit der Zulässigkeit dieses HTTP-Bodys stehen und die Sicherheit vor Denial-of-Service-Attacken (DoS) erhöhen können (sofern noch weitere Maßnahmen im Netzwerk getroffen werden). 

|   |   |
|---|---|
|[LimitRequestBody](https://httpd.apache.org/docs/2.4/de/mod/core.html#limitrequestbody) 0|Die Direktive gibt die Anzahl der Bytes zwischen 0 (unbegrenzt) und 2147483647 (2GB) an, die im Request-Body (Datenteil der Anfrage) erlaubt sind.  <br><br>**Empfehlung:** Setzen Sie den Wert unbedingt, da die Default-Einstellung _unbegrenzt_ Denial-of-Service-Attacken leicht ermöglicht, z.B. LimitRequestBody 1000000.|
|[LimitRequestFields](https://httpd.apache.org/docs/2.4/de/mod/core.html#limitrequestfields) 100|Begrenzt die Anzahl der erlaubten Zeilen eines Client Requests.  <br><br>**Empfehlung:** Auch wenn die Einschränkung der Headerzeilen nicht so entscheidend ist, kann der Wert problemlos auf 30 gesetzt werden. Also LimitRequestFields 30|
|[LimitRequestFieldsize](https://httpd.apache.org/docs/2.4/de/mod/core.html#limitrequestfieldsize) 8190|Begrenzt die Länge des vom Client gesendeten HTTP Client-Request-Headers.  <br><br>**Empfehlung:** Der Wert kann so beibehalten werden. Bei sehr speziellen Konfigurationen und Anwendungen könnte der Wert sogar noch zu niedrig sein.|
|[LimitRequestLine](https://httpd.apache.org/docs/2.4/de/mod/core.html#limitrequestline) 8190|Begrenzt die Länge der Request Line (also der ersten Zeile im Client Request mit der GET-Anfrage).  <br><br>**Empfehlung:** Der Wert kann so beibehalten werden. Wenn Sie ausprobieren möchten, welchen Effekt die Werte haben, dann können sie gerne mal LimitRequestLine 5 probieren.|
|[LimitXMLRequestBody](https://httpd.apache.org/docs/2.4/de/mod/core.html#limitxmlrequestbody) 1000000|Begrenzt die Größe eines XML-basierten Request-Bodys.  <br><br>**Empfehlung:** Da hier ja ein Default-Wert angegeben ist, muss keine Änderung erfolgen.|


## 5.3 Header 

In dieser Übung können Sie zeigen, dass Sie den Apache konfigurieren können. Die Übung vereint viele unterschiedliche Kenntnisse. Bevor Sie die Lösung ansehen, sollten Sie zumindest überlegen, wie Sie vorgehen würden. Wer mit Linux-Systemen vertraut ist, sollte die Übung ohne einen Blick in die Lösung probieren.

Schreiben sie Ihren Namen in den HTTP-Header des Server Response. Hilfestellung: die zugehörige Direktive heißt Header. 

Losung: 

Zur Direktive [_**Header**_](http://httpd.apache.org/docs/2.4/en/mod/mod_headers.html#header) gehört das Modul _**mod_headers**_. Die Direktive kann im Kontext _server config, virtual host, directory, .htaccess_ eingebunden werden.

Zuerst muss nachgesehen werden, ob das Modul _mod_headers_ in _/etc/apache2/mods-enabled_ vorhanden und somit schon in die apache2.conf inkludiert ist (dort wird es ohne _mods__ geschrieben, also nur _headers_ - und es ist vermutlich nicht vorhanden).

Falls es unter mods-enabled nicht vorhanden ist, sollte man also nachschauen, ob es unter /etc/apache2/mods-available zu finden ist. Dort sollte es vorhanden sein.

Somit muss nun mit dem Linux-Komando _**ln -s QUELLE ZIEL**_ ein Softlink erstellt werden. Wenn man im Verzeichnis _/etc/apache2/mods-enabled_ ist, dann lautet die Syntax _**ln -s ../mods-available/headers.load ./headers.load**_

- _**-s**_ der Parameter -s legt einen Softlink an.
- _**../mods-available/headers.load**_ weist den Weg zur Quelle des Links. Die _**..**_ stehen in Linux für das Verzeichnis über dem jetzigen Verzeichnis. Es geht im Dateibaum also zuerst ein Verzeichnis hoch und dann in das Verzeichnis _mods-available_.
- _**./headers.load**_ weist den Weg zum Ziel. Der einzelne Punkt _**.**_ steht in Linux für das aktuelle Verzeichnis.

Nun in /etc/apache2/mods-enabled mit _**ls -al**_ nachsehen, ob die Erstellung des Softlinks geklappt hat.

Falls ja, dann in die _apache2.conf_ die Direktive einfügen. Die Direktive MUSS irgendwo im _general header_ nach dem Inkludieren der Module, also nach der Zeile _IncludeOptional mods-enabled/*.load_, erfolgen.

Nun folgende Zeile einfügen: _**Header add X-Name "Ihr NAME OHNE UMLAUTE"**_. Das _X-_ wurde gewählt, da damit "eigene" Einträge im HTTP-Header gekennzeichnet werden.

Dann den Apache Webserver neu starten via _systemctl reload apache2_.

Und abschließend das Ergebnis im HTTP-Header des Server Response (siehe [HTTP-Request und HTTP-Response](https://isp.eduloop.de/loop/HTTP-Request_und_HTTP-Response "HTTP-Request und HTTP-Response") kontrollieren.

## 5.4 Zugangsschutz

Ein typisches Beispiel, um Ihren Server in einfacher Weise zu schützen, ist die Konfiguration eines passwortbasierten Zugangsschutzes. Dieser Schutz ist nicht sehr sicher, aber er verhindert, dass "robots" auf die Seite kommen und Schwachstellen (z.B. in Formularfeldern) ausprobieren können.

> **Bitte konfigurieren Sie Ihren Server mit Zugangsschutz, damit die kommende PHP-Programmierung gefahrlos durchgeführt werden kann.**

Für den Apache gibt es [viele Arten des Zugangsschutzes](http://httpd.apache.org/docs/2.4/en/howto/auth.html). Hier wird ein einfach zu realisierender Zugangsschutz gewählt. Doch zuerst muss überlegt werden, welche Dokumente der Apache-Webserver überhaupt an einen Client ausliefern kann.

Der Zugangsschutz soll für alle Dokumente gelten, die über den Apache zur Verfügung gestellt werden. Mit der Direktive _**DocumentRoot**_ wird das "Root-Verzeichnis" angegeben. Unterhalb dieses Verzeichnisses können alle Dokumente per Client Request angefragt werden. 
Aber beim Apache sind Links in andere Verzeichnisse möglich, sofern diese mit der Direktive _**Options FollowSymLinks**_ erlaubt werden. Es reicht also nicht, wenn man den Zugangsschutz auf der Ebene _DocumentRoot_ einrichtet und gleichzeigt _FollowSymLinks_ zulässt. Hier kommt nun eine Anleitung zum Einrichten eines Basis-Zugangsschutzes.

  
**[Konfigurationsanweisungen in der apache2.conf](https://isp.eduloop.de/loop/Strukturierung_der_Konfigurationsdatei "Strukturierung der Konfigurationsdatei")**  
In der apache2.conf steht folgender Abschnitt, der sich auf alle Verzeichnisse des Servers bezieht und sehr sinnvoll ist:

```
<Directory />
    Options FollowSymLinks
    AllowOverride None
    Require all denied
</Directory>
```

- Durch [_**AllowOverride None**_](http://httpd.apache.org/docs/2.4/en/mod/core.html#allowoverride) werden _.htaccess_-Dateien verboten (sofern sie in Unterverzeichnissen nicht explizit wieder zugelassen werden).
- Durch [_**Require all denied**_](https://httpd.apache.org/docs/2.4/de/mod/mod_authz_core.html#require) wird der Zugriff komplett gesperrt.

  
Anschließend gibt es in der apache2.conf Direktiven, die sich auf ein Unterverzeichnis beziehen und damit die oben genannten Regeln aufheben.

```
<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

- Durch [_**AllowOverride None**_](http://httpd.apache.org/docs/2.4/en/mod/core.html#allowoverride) werden auch hier alle _.htaccess_-Dateien verboten, was an dieser Stelle überflüssig ist, da dieses Verbot bereits auf einer höheren Verzeichnisebene erfolgte.
- Durch [_**Require all granted**_](https://httpd.apache.org/docs/2.4/de/mod/mod_authz_core.html#require) wird der Zugriff komplett geöffnet. Es sind also prinzipiell alle Dokumente unterhalb von _/var/www/_ abrufbar.  
    **Hier muss also der Zugangsschutz ansetzen**.

---

**Zugangsschutz einrichten**  
Der neue Abschnitt mit dem Zugansschutz soll wie folgt aussehen:

```
<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride None
    AuthType Basic
    AuthName "Mein Zugangsschutz"
    AuthUserFile "/etc/apache2/htpass"
    Require user IhrUserName
</Directory>
```

- [_**AuthType**_ (benötigt das Modul mod_authn_core)](http://httpd.apache.org/docs/2.4/en/mod/mod_authn_core.html#authtype) gibt die generelle Art des Schutzes an. Wenn die Seite aufgerufen wird, erzeugt der Browser ein Fenster für eine Passwortabfrage.
- [_**AuthName**_ (benötigt das Modul mod_authn_core)](http://httpd.apache.org/docs/2.4/en/mod/mod_authn_core.html#authname) ist für den Text zuständig, der in dem Fenster für die Passwortabfrage angezeigt wird. Hier können Sie (fast) beliebigen Text eingeben.
- [_**AuthUserFile**_ (benötigt das Modul mod_authn_file)](http://httpd.apache.org/docs/2.4/en/mod/mod_authn_file.html#authuserfile) enthält die Angabe der Passwortdatei.
- [_**Require**_ (benötigt das Modul mod_authz_core)](https://httpd.apache.org/docs/2.4/de/mod/mod_authz_core.html#require) wird nun, gegenüber den ursprünglichen Einstellungen _granted_, geändert in _user_ und, mit der Eingabe eines Nutzernamens (bitte ändern Sie _IhrUserName_ in einen sinnvollen Nutzernamen), wird der Zugriff entsprechend eingeschränkt. Wird anstelle eines Nutzernamens _valid-user_ angegeben, so haben alle Nutzer, die in der Passwortdatei eingetragen sind, Zugriffsrechte mit ihrem eigenen Passwort.

Somit haben wir den passenden Abschnitt, der in die apache2.conf geschrieben werden muss. Selbstverständlich müssen Sie nachsehen, ob alle notwendigen Module vorhanden sind.

---

**Passwortdatei anlegen und Passwort festlegen**  
Nun muss ein Passwort für den angegebenen Nutzer erstellt werden. Hierzu gibt es das Linux-Kommando [_**htpasswd -cb** passwdfile username_](http://httpd.apache.org/docs/2.4/en/programs/htpasswd.html). Also in der Kommandozeile in das Verzeichnis `/etc/apache2` wechseln und dort den Befehl eingeben:

`htpasswd -cb htpass IhrUserName (Passwort)`

Anschließend den Webserver neu starten und die Webseite aufrufen. Dann sollte ein Fenster für die Eingabe des Passwortes sichtbar werden.


### 5.4.1 .htaccess 和 htpasswd 在 Apache 中设置基础访问控制

好的，以下是如何通过 .htaccess 和 htpasswd 在 Apache 中设置基础访问控制（用户名 + 密码保护）的详细步骤：


第一步：确保 Apache 允许 `.htaccess` 文件控制
Apache 需要在对应的配置文件中启用 `.htaccess` 的使用（通常是 `apache2.conf` 或某个虚拟主机配置文件）。
假设你的网站根目录是 `/var/www/html`，请确保在对应 `<Directory>` 配置块中包含以下内容：
```
<Directory /var/www/html>
    AllowOverride All
</Directory>
```

然后运行以下命令重新加载 Apache 配置：
```
sudo systemctl reload apache2
```


第二步：创建密码文件 .htpasswd

使用 htpasswd 工具来创建保存用户名和加密密码的文件。

sudo apt install apache2-utils  # 如果还没有安装 htpasswd 工具
sudo htpasswd -c /etc/apache2/.htpasswd yourusername
- `-c` 代表创建文件（只在第一次使用时加）
- 输入你要设置的密码

`sudo htpasswd -c /etc/apache2/.htpasswd alice`


第三步：在网站目录下创建 `.htaccess` 文件

在网站根目录（例如 `/var/www/html`）中添加一个 `.htaccess` 文件，写入以下内容：
```
AuthType Basic
AuthName "Geschützter Bereich"  # 这是登录弹窗中显示的名称
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```


第四步：测试访问控制

1. 在浏览器中访问你的网站页面，比如 http://your-server-ip/index.html    
2. 应该会弹出用户名/密码登录框
3. 输入 `.htpasswd` 文件中设置的用户名和密码即可访问



可选增强：限制某些目录
如果你只想保护某个子目录（比如 `/var/www/html/secret`），就在该子目录下单独设置 `.htaccess` 文件即可，其他目录不受影响。
