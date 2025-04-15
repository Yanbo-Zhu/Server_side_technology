
Der Apache ist noch immer der bekannteste Webserver. Doch während er früher "immer und überall" eingesetzt wurde, werden heutzutage vermehrt nginx und node.js verwendet. Alle hier gezeigten Konfigurationsanweisungen beziehen sich auf ein Debian-System (also Linux) und den Apache 2.4.

**Sicherheitshinweis:** Bei jeder Linux-Distribution ist der Apache Webserver dabei und läuft selbstverständlich problemlos. Aber **die mitgelieferten Standard Apache-Konfigurationen dürfen nicht in produktiven Umgebungen eingesetzt werden!** Diese Standard-Konfigurationen sind so erstellt, dass die Webserver unter möglichst vielen Umständen laufen. Sie

- sind nicht abgesichert,
    - haben zu viele Module eingeschaltet,
    - geben zu viele Informationen über sich bekannt,
- sind nicht übersichtlich,
- sind nicht performant.

Daher ist es wichtig, dass Sie sich mit der Konfiguration des Apache-Webservers eingehend beschäftigen, bevor sie diesen produktiv einsetzen.


# 1 **Manager- und Bearbeiterprozesse (=Workerprozesse)**  

Ob der Apache läuft, kann mit _**ps aux | grep apache2**_ ermittelt werden. In der Abbildung sehen wir sechs Prozesse.

![[images/Apache-1grep.png]]

- Der oberste Prozess läuft unter _root_ und ist der Managerprozess. Die Aufgabe des Managerprozesses ist die Bereitstellung von so vielen Bearbeiterprozessen, wie in der Konfigurationsdatei angegeben sind.
- Die anderen fünf Prozesse laufen in diesem Beispiel unter dem Nutzer _www-data_. Dabei handelt es sich um einen nicht-privilegierten Nutzer, der automatisch vom Webserver eingerichtet wurde. Diese fünf Prozesse können Anfragen (Client-Requests) entgegennehmen und bearbeiten.

**Sicherheitshinweis:** Bearbeiterprozesse sollten immer einem nicht-privilegierten Benutzer gehören, der keine anderen Rechte hat, als die Webdokumente auszuliefern.
- Der Managerprozess steht also an oberster Position und sollte normalerweise vom User "_root_" gestartet worden sein. Die Bearbeiterprozesse haben die Rechte eines Users "_www-data_" (auf die Bedeutung Rechte wird später eingegangen).
- Der Bearbeiterprozess bearbeitet die Anfrage und gibt die Rückmeldung zurück an den Client. Da es beim Apache-Webserver für jede Anfrage einen Bearbeiterprozess gibt, der zuvor gestartet wurde, nennt man diese Arbeitsweise auch "Prefork".
	- 处理进程负责处理请求，并将响应返回给客户端。由于 Apache Web 服务器为每一个请求都启动一个处理进程，这种工作方式被称为 **“Prefork（预派生）”** 模式。
- Damit nicht mit jeder Anfrage extra ein neuer Bearbeiterprozess gestartet werden muss, wird mit einer Voreinstellung in der Apache-Konfiguration die Mindestanzahl der immer laufenden Prozesse vorgeben.
	- 为了避免每次请求都额外启动一个新进程，Apache 配置中可以通过预设值指定始终保持运行的最小进程数量。
- Der Managerprozess startet bei Bedarf zusätzliche Bearbeiterprozesse.
- Damit bei einer großen Anzahl von Anfragen der Server nicht überlastet wird, sollte eine maximale Anzahl laufender Prozesse vorgegeben werden.

> Die Einstellung der aktiven und maximalen Bearbeiterprozesse gehört somit zu den notwendigen Grundeinstellungen, um den Webserver effizient und sicher zu betreiben.



# 2 **Starten und Stoppen des Apache (Linux)**  

Mit _**systemctl stop apache2**_ und _**systemctl start apache2**_ können Sie den Apache stoppen und starten. Beim Stoppen werden alle offenen Verbindungen (Client Requests) abgebrochen. Dies ist bei produktiven Systemen natürlich unerwünscht, sodass mit _**systemctl reload apache2**_ die Prozesse ohne Unterbrechung neugestartet werden können.

Aufgabe: Stoppen Sie den Webserver und schauen Sie mit _ps aux | grep apache_ nach, ob dies erfolgreich war.

**Nach jeder Änderung an der Konfigurationsdatei des Webservers, muss der Webserver neugestartet werden.** Es ist ein sehr häufig auftretender (Anfänger-)Fehler, dass nach einer Konfigurationsänderung vergessen wird, den Webserver neuzustarten.



# 3 Direkte Programmoptionen für apache2


Der Webserver wird normalerweise über das Startskript (also _systemctl start apache2_) gestartet. Im Startskript selbst steht selbstverständlich der Aufruf des Webserver-Programms _apache2._ Das Webserver-Programm kennt verschiedene Optionen.

Hilfe zu den möglichen Programmoptionen erhält man mit _apache2 –h_ oder _man apache2_.

|Optionen|Bedeutung|
|---|---|
|_-D name_|define a name for use in _<IfDefine name>_ directives|
|_-d directory_|specify an alternate initial ServerRoot|
|_**-f file**_|**specify an alternate ServerConfigFile**|
|_-C directive_|process directive before reading config files|
|_-c directive_|process directive after reading config files|
|_-e level_|show startup errors of level (see LogLevel)|
|_-E file_|log startup errors to file|
|_**-v**_|**show version number**|
|-V|show compile settings|
|_-h_|list available command line options (this page)|
|_**-l**_|**Compiled-in modules**|
|_-L_|list available configuration directives|
|_-S_|show parsed settings (currently only vhost settings)|
|_-t_|run syntax check for config files (with docroot check)|

Wichtig ist die Ermittlung der Serverversion, damit Module und andere Ergänzungen auch zu der entsprechenden Serversoftware passen. Die Serverversion lässt sich demnach ermitteln mit: _**/usr/sbin/apache2 -v**_

Tipp: Falls die Daten nicht in den gezeigten Verzeichnissen vorhanden sind, können Sie mit dem Linux-Befehl "_whereis apache2_" suchen.




# 4 Modularer Aufbau des Apache


Bevor wir uns die Konfigurationsdateien ansehen, schauen wir uns hier das generelle Modulkonzept des Apache-Webservers an.

Um den Apache auf unterschiedlichen Plattformen anzubieten, wurde eine =="Apache Portable Runtime (APR)"== entwickelt, die direkt an die unterschiedlichen Betriebssysteme angepasst ist. Je nach Betriebssystem gibt es also eine spezielle APR. Über die APR hat der Kern somit Zugriff auf einige grundlegende Funktionen des Betriebssystems. Dazu gehören:

- Ein- und Ausgabe von Dateien
- Netzwerkfunktionen
- Thread- und Prozessverwaltung
- Speicherverwaltung
- Laden dynamischen Codes

Die APR ist ein eigenständiges Projekt und lässt sich als Bibliothek unter der Apache-Lizenz auch für andere Anwendungen einsetzen.

Auf der APR kommt in der nächsten Schicht ein [Multi-Processing-Modul (MPM)](http://httpd.apache.org/docs/2.4/en/mpm.html). Es stehen unterschiedlich arbeitende MPMs für unterschiedliche Einsatzzwecke zur Verfügung. Für Linux gibt es beispielsweise die MPMs ["Event"](http://httpd.apache.org/docs/2.4/en/mod/event.html), ["Worker"](http://httpd.apache.org/docs/2.4/en/mod/worker.html) und ["Prefork"](http://httpd.apache.org/docs/2.4/en/mod/prefork.html). Bei den meisten Linux-Distributionen ist das MPM "Prefork" voreingestellt. Hierbei handelt es sich um einen im Voraus "forkenden" Webserver ohne Thread-Unterstützung, der auch hier in allen Beispielen verwendet wird.

Wichtig ist, dass an dem MPM sehr viele Module angeschlossen werden können, z.B. ein PHP-Modul zur Ausführung von PHP-Code.

[![Apache3-generelleModule.jpg](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/a/a7/Apache3-generelleModule.jpg)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/a/a7/Apache3-generelleModule.jpg)

 Abb. 35: Modularer Aufbau des Apache2.x

Es gibt also beim Apache Module auf unterschiedlichen Ebenen. Die mit _apache2 -l_ ermittelten Module befinden sich demnach auf unterschiedlichen "Ebenen". So sind _**core.c**_ und _**http_core.c**_ Module für die "Apache Portable Runtime (APR)".

Das Modul _**mod_so.c**_ hat eine Sonderstellung, denn es ermöglicht, andere Module als "Shared Object" dynamisch einzubinden. Hierzu genügt es, dass die entsprechenden Module in der Konfigurationsdatei aufgerufen werden.


# 5 Apache Module

Neben den zwingend notwendigen, gibt es eine ganze Reihe zusätzlicher Module für spezielle Anwendungsfälle. Eine Liste der Standardmodule findet sich unter [https://httpd.apache.org/docs/2.4/en/mod/](https://httpd.apache.org/docs/2.4/en/mod/). Zusätzlich gibt es noch "externe" Module, zu denen beispielsweise auch das PHP-Modul gehört. Diese finden sich unter [http://modules.apache.org/](http://modules.apache.org/).

Jedes dieser zahlreichen Module hat eigene Konfigurationsparameter. Somit ist es möglich, den Webserver sehr speziell zu konfigurieren, da unüberschaubar viele Konfigurationsparameter zur Verfügung stehen.

|   |   |
|---|---|
|**mod_access**|Provides access control based on client hostname, IP address, or other characteristics of the client request.|
|**mod_alias**|Provides for mapping different parts of the host filesystem in the document tree and for [URL](https://isp.eduloop.de/loop/URL "URL") redirection.|
|**mod_asis**|Sends files that contain their own HTTP headers.|
|**mod_auth**|User authentication using text files.|
|**mod_autoindex**|Generates directory indexes, automatically, similar to the Unix ls command or the Win32 dir shell command.|
|**mod_deflate**|Compresses content before it is delivered to the client.|
|**mod_dir**|Provides for "trailing slash" redirects and serving directory index files.|
|**mod_env**|Modifies the environment which is passed to [CGI](https://isp.eduloop.de/loop/CGI "CGI") scripts and SSI pages.|
|**mod_expires**|Generation of Expires and Cache-Control HTTP headers according to user-specified criteria.|
|**mod_headers**|Customization of HTTP request and response headers.|
|**mod_log_config**|Logging of the requests made to the server.|
|**mod_mime**|Associates the requested filename's extensions with the file's behavior (handlers and filters) and content (mime-type, language, character set and encoding).|
|**mod_mime_magic**|Determines the MIME type of a file by looking at a few bytes of its contents.|


|   |   |
|---|---|
|**mod_speling**|Attempts to correct mistaken URLs that users might have entered by ignoring capitalization and by allowing up to one misspelling.|
|**mod_ratelimit**|Bandwidth Rate Limiting for Clients|
|**mod_usertrack**|Clickstream|

# 6 Die Verzeichnisstruktur des apache2


Die Verzeichnisstruktur ist auf jedem Betriebssystem etwas anders, jedoch ist der prinzipielle Aufbau gleich.

Die Hauptkonfigurationsdatei befindet sich in Linux-Systemen im Verzeichnis _**/etc/apache2**_ und heißt oftmals _**apache2.conf**_. In dieser Datei kann man normalerweise auch erkennen, wo sich die weiteren Konfigurationsdateien befinden (siehe Abbildung). Die Struktur ist für erfahrene Betreiber von Webservern gut, für "Anfänger" jedoch unübersichtlich. Dabei gibt es zwei Probleme:

- Durch die Unübersichtlichkeit entstehen oft Sicherheitslücken, da unbedacht zu viele Module und deren Konfigurationen eingebunden werden.
- Konfigurationsanweisungen können sich gegenseitig überschreiben. In der Regel gilt die zuletzt geladene Anweisung (Direktive). Dies kann jedoch eine Direktive sein, die irgendwo in irgendeiner Konfigurationsdatei steht. Daher sollten die Konfigurationen der Module in eigenen Dateien erfolgen, um zumindest ansatzweise eine Struktur abzubilden.


Die Aufgabe ist also, dass man möglichst viele Module ausschaltet und die zugehörigen Konfigurationsdateien genau analysiert.


![[images/Apache4-Konfigurationsbaum.png]]

## 6.1 _**/etc/apache2**_  
Im Verzeichnis _**/etc/apache2**_ gibt es die Hauptkonfigurationsdatei _**apache2.conf**_ und eine Datei _**ports.conf**_, in der die Standardports festgelegt werden. Dies sind Port 80 für HTTP und Port 443 für die gesicherte Verbindung HTTPS. In der Abbildung sieht man eine kleine Konfigurationsdatei.

![[images/Apache5-ports.conf.png]]


## 6.2 _**mods-enabled**_  
Es gibt ein Unterverzeichnis _**mods-enabled**_, in dem alle eingeschalteten Module mit ihren Konfigurationsdateien vorhanden sind. Die Module selbst werden über die _**.load**_-Dateien geladen. Zu vielen Modulen gibt es entsprechende Konfigurationsdateien _**.conf**_. Nun sollte man für jedes Modul prüfen, ob dieses wirklich benötigt wird.

[![Apache6a-enabled.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/b/b5/Apache6a-enabled.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/b/b5/Apache6a-enabled.png)

 Abb. 38: Dateien im Verzeichnis ''mods-enabled''

Das Modul [_**negotiation.conf**_](https://httpd.apache.org/docs/2.4/mod/mod_negotiation.html) wird beispielsweise nicht benötigt. Entsprechend können die Einträge _negotiation.load_ und _negotiation.conf_ gelöscht werden. Alle Einträge in dem Verzeichnis _**mods-enabled**_ sind nur Verweise (Softlinks) auf die Dateien im Verzeichnis _**mods-available**_. Somit können die Links gefahrlos gelöscht und bei Bedarf wieder hergestellt werden.


Schauen Sie nach, wie viele Module im Verzeichnis _**mods-available**_ auf Ihrem Server prinzipiell zur Verfügung stehen. Sie werden staunen, wie viele es sind.

  
## 6.3 _**conf-enabled**_  
Es gibt ein Unterverzeichnis _**conf-enabled**_, aus dem weitere Konfigurationsdateien geladen werden.

[![Apache-7conf-enabled.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/1/1b/Apache-7conf-enabled.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/1/1b/Apache-7conf-enabled.png)

 Abb. 39: Dateien im Verzeichnis ''mods-enabled''

Hierin gibt es beispielsweise die Datei _**security.conf**_, die sehr sinnvoll ist, doch in den Default-Einstellungen die Sicherheit nahezu komplett ausgeschaltet hat.

Aufgabe

Ändern Sie die Einstellungen in der Datei _**security.conf**_ so, dass alle dort beschriebenen Einstellungen auf dem bestmöglichen, angegebenen Sicherheitsniveau stehen. Ihr Server wird danach trotzdem wie gewohnt laufen. Nach der Änderung in der Datei bitte nicht vergessen, den Server mit _**systemctl reload apache2**_ neu zu starten.

  
## 6.4 _**sites-enabled**_  
Der Apache Webserver eignet sich sehr gut, um viele verschiedene Websites mit unterschiedlichen Anforderungen auf einem Webserver zu betreiben. Wenn man verschiedene Websites hat, dann ist es sinnvoll, die speziellen Konfigurationen im Unterverzeichnis _**sites-enabled**_ abzulegen.== In diesem Unterverzeichnis gibt es nur einen Link auf eine Datei in einem Verzeichnis _**sites-available**_. ==

Wir werden an späterer Stelle auf diese Datei noch genauer eingehen, da wir hier nur mit einer Site arbeiten werden.


==ist im Verzeichnis _sites-enabled/_ nur ein Link auf eine Datei 000-default.conf im Verzeichnis _sites-available/_==



