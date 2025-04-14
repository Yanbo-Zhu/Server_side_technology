
# Strukturierung der Konfigurationsdatei

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

# Konfiguration des Moduls MPM prefork

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


# Section Direktiven

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


# Direktiven und Modulzuordnung

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


# Beispiel

## ServerTokens und ServerSignature

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