

# 1 总览 

In diesem Kapitel werden wir drei unterschiedliche Webserver behandeln:
- den Apache Webserver,
- den nginx Webserver,
- und den node.js Webserver.

Es gibt **unterschiedliche Webserver** für ganz unterschiedliche Zwecke. Beispielsweise gibt es in vielen Druckern einen Webserver, um den Druckstatus über einen Browser (oder ein Desktop-Programm) abrufen zu können. Oder auch Heizungsthermostate, die über einen Webserver gesteuert werden. An diese Webserver werden natürlich ganz andere Anforderungen gestellt, als an Hochleistungs-Webserver für bekannte Internet-Auftritte.

In diesem Kapitel behandeln wir drei sehr bekannte Webserver für "normale Internetauftritte":

- **Apache Webserver**: Das "Urgestein". Läuft sehr stabil und kann für alle möglichen Anwendungs- und Spezialfälle konfiguriert werden, ist also **sehr flexibel** einsetzbar. Der Apache Webserver ist "workerprozessbasiert". Somit bekommt jeder Client-Request einen eigenen Workerprozess zugewiesen - und damit die gesamte Mächtigkeit und Schwerfälligkeit des Apache. Es wird für einen Client-Request, bei dem ein Bild angefordert wird, beispielsweise auch das PHP-Modul des Webservers, so wie alle anderen Module im Workerprozess, geladen. Je nach Hardwareausstattung des Servers und nach Anzahl der (gleichzeitigen) Client-Requests, kann dies zu einem Performance-Problem führen.
- **nginx Webserver**: Ein schlanker, eventbasierter Webserver mit viel geringeren Konfigurationsmöglichkeiten und ohne die schwergewichtigen Workerprozesse. Somit ist der nginx **sehr performant**. Oftmals werden nginx und Apache-Webserver auch gemeinsam eingesetzt. Dabei übernimmt der nginx auf einem Server alle statischen Ressourcen (Bilder, CSS-Dateien etc.) und leitet nur die "Spezialanfragen" an einen anderen Server weiter (als Load-Balancer), auf dem dann ein Apache läuft. So kann man in einer "Server-Farm" die hohe Performance des nginx und die gute Flexibilität des Apache nutzen.
- **node.js Webserver**: Ein neuer Webserver, der in **JavaScript** geschrieben ist und ebenfalls eventbasiert arbeitet. Interessant ist der node.js, da er nicht nur ein Web-, sondern auch eine Art **"Datenstrom"-Server** ist. So kann er auch als TCP-Server arbeiten, oder für den asynchronen Netzwerkzugriff eingesetzt werden.


**Prinzipielle Funktionsweise**  
Jeder Server hat eine eindeutige IP-Adresse bzw. eine Domain oder URI, die auf diese IP-Adresse zeigt. Über die IP-Adresse kommt die Anfrage bis zum Server. Auf dem Server können unterschiedliche Serverprozesse laufen, die jeweils ein ganz bestimmtes Protokoll erwarten und als "Hintergrundprozess" (engl. deamon) auf eine Anfrage warten. Diese Anfragen "lauschen" auf einem Port. Beim Protokoll HTTP ist dies, standardmäßig voreingestellt, der Port 80.

Ablauf einer Client-Anfrage:
- Dem Webserver wird eine feste IP-Nummer zugeordnet, sodass der Client den gewünschten Server ansprechen kann.
- Auf dem Server wiederum werden die verschiedenen Anfragen über Portnummern an das entsprechende Programm weitergeleitet.
- Der Standard-Port für den Webserver ist Port 80.
- Jede Anfrage auf Port 80 wird an das Server-Programm weitergeleitet und dort von einem Prozess bearbeitet.


# 2 NGINX versus Node.js


NGINX
Überblick und Eigenschaften
- Modular aufgebauter Webserver mit umfangreichen Managementfunktionen
- Prinzipien: Ereignisbasiert und Single-Threaded
- Hohe Leistung und gute Konfigurierbarkeit
- Häufiger Einsatz als Reverse Proxy

Reverse Proxy
- Intermediärer Webserver zwischen Client und Backend
- Abdeckung des gesamten TCP/IP-Protokollstapels 
- Load Balancing: Gleichmäßige Verteilung von Anfragen auf die Web- oder Anwendungsserver eines Clusters
- Komprimierung von Webinhalten
- SSL Endpoint: Durchführung von Ver- und Entschlüsselung  zur Entlastung von Web- und Anwendungsservern
- Sicherheit und Anonymität: Erkennung von Angriffen durch Kombination mit Intrusion Detection Systemen (IDS)

---

Anforderungen moderner Webanwendungen
- Bessere Skalierbarkeit zur Lösung C10k-Problems: 10.000 Clients müssen quasi gleichzeitig bedient werden können
- Webanwendungen dürfen weder in Komfort noch Reaktivität nativen Anwendungen nachstehen
- Echtzeitfähigkeit zur schnellstmöglichen Versorgung des Benutzers mit neuen Informationen

Lösung Node.js
- Seit 2005 entwickelt von Ryan Dahl, 2009 erstmals veröffentlicht auf der jsconf.eu-Konferenz in Berlin
- Basiert auf JavaScript und der JavaScript-Laufzeitumgebung V8, ursprünglich entwickelt für Google Chrome
- Ereignisgesteuerte, ressourcensparende Architektur, die eine Großzahl gleichzeitig bestehender Netzverbindungen erlaubt
- Integrierte Module für HTTP-Client- und Serverfunktionalitäten, Zugriff auf das Dateisystem etc.
- Beliebig erweiterbar durch zahlreiche externe Module

---


Backend-Konfigurationen mit Node.js und NGINX

![](images/Pasted%20image%2020250116212542.png)

# 3 Multi- versus Single-Thread Server
## 3.1 MULTI-THREAD (ODER SYNCHRONER) SERVER 

![](images/Pasted%20image%2020250116212630.png)

- Erzeugung eines dedizierten Threads pro TCP-Verbindung
- Threads haben einen exklusiven Speicherbereich
- Hoher CPU-Aufwand für das Thread-Management, d.h. für die ständig wechselnde Zuweisung von CPU-Ressourcen zu Threads   
- Jeder Thread bearbeitet exklusiv einen Anfrage/Antwort-Zyklus der Verbindung
- Lebenszeit eines Threads entspricht der Verbindungsdauer, d.h. mehrere HTTP-Anfragen/Antworten (siehe keep-alive von HTTP)
- Maximal sechs gleichzeitige Verbindungen pro Client
- Synchrone Arbeitsweise: Operationen auf externen Ressourcen (Datenbanken, Dateisysteme, Kommunikation zu anderen Anwendungsservern) verursachen Blockierung des jeweiligen Threads
- Hoher Speicheraufwand und CPU-Verbrauch führen zu mäßiger Skalierbarkeit


 Request/Respoinse und Rollenverteilung (II/II)
 - Synchrones Protokoll der Anwendungsschicht für die Client/Server-Kommunikation
- HTTP Request wird immer durch Client, HTTP Response wird immer durch Server gesendet
- Synchroner Ablauf: Strikte Einhaltung des Request/Response-Zyklus, d.h. Client sendet Request und wartet (d.h. ist blockiert bis) bis Antwort eintrifft
- HTTP 1.0: erste offizieller Standard des Protokolls, veröffentlicht 1996
- HTTP 1.1: gegenwärtige Version, optimiert um Keepalive und Caching Mechanismen, veröffentlicht im Januar 1997
- HTTP/2: Verbesserungen bzgl. Ladegeschwindigkeit (SPDY) und Sicherheit, Verabschiedung des Standards in Vorbereitung
- SPDY: "Zwischenstandard" von Google mit Mechanismen zur Komprimierung, Multiplexing, Priorisierung



## 3.2 SINGLE-THREAD (ODER ASYNCHRONER) SERVER

![](images/Pasted%20image%2020250116213003.png)

- Bearbeitung sämtlicher TCP-Verbindungen durch einen einzelnen Thread (**_Single Thread_**)
- Single Thread nimmt HTTP-Anfragen entgegen, bearbeitet sie und erstellt und versendet Antworten
- Asynchrone Arbeitsweise: Operationen auf externen Ressourcen (Datenbanken, Dateisysteme, andere Anwendungsserver) werden als "Aufträge" an einen Pool mit asynchronen Threads (**_Worker Threads_**) delegiert
- Single Threads und Worker Threads blockieren nicht
- **_Event Loop_**: Warteschlange von Ereignissen die durch den Single-Thread nacheinander abgearbeitet werden

- Ereignisse
    - Anfragen für neue TCP-Verbindungen
    - HTTP-Anfragen
    - Fertig bearbeitete I/O-Aufträge externer Ressourcen (Datenbanken, Anwendungsserver, Dateisysteme)
- Einstellen von Ereignissen in die Event Loop mittels Callback-Funktionen
- Single-Thread-Ansatz verursacht wesentlich weniger Overhead (Speicher, CPU,...) als Multi-Thread-Ansatz und skaliert besser
- Voraussetzung: rechenarme Aufgaben mit vielen I/O-Operationen (z.B. Webapplikationen)


# 4 Multi- versus Single-Thread Server

![](images/Pasted%20image%2020250116213248.png)

- **_Apache_**: Multi-Thread-Server mit synchroner Verarbeitung und blockierenden Threads
- _**nginx**_: Single-Thread-Server mit asynchroner Verarbeitung und nicht-blockierende Threads (vergleichbar zu Node.js)


# 5 Prinzipien der Erstellung dynamischer Webseiten (I/III)

1 Möglichkeit: Server-Programm generiert HTML für Multi-Page Applications

```js
const express = require('express');
const app = express();

app.get('/addresses', function(req, res) {
    // Definiere die Adressen
    const addresses = [
        "123 Example St, City, Country",
        "456 Sample Rd, Town, Country",
        "789 Demo Blvd, City, Country"
    ];
    
    // Generiere HTML-Inhalt
    const htmlContent = `
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta http-equiv="X-UA-Compatible" content="IE=edge">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Addresses</title>
        </head>
        <body>
            <h1>Adressen</h1>
            <pre>${JSON.stringify(addresses, null, 2)}</pre>
        </body>
        </html>
    `;
    
    // Setze den Content-Type auf text/html
    res.type('text/html');

    // Sende die HTML-Seite
    res.send(htmlContent);
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, function() {
    console.log(`Server is running on port ${PORT}`);
});

```

---

2 Möglichkeit:  HTML-Templates mit eingebettetem Code für Multi-Page Applications
eingebettetem Code in js script 
```html
<!DOCTYPE html>
<html>
<head>
    <title>EJS Example</title>
</head>
<body>
    <h1>Addresses</h1>
    <% for(var i=0; i<addresses.length; i++) { %>
        <p><%= addresses[i] %></p>
    <% } %>
</body>
</html>
```

```js
const express = require('express');
const app = express();

app.set('view engine', 'ejs');

app.get('/addresses', function(req, res) {
    const addresses = [
      "123 Example St, City, Country",
      "456 Sample Rd, Town, Country",
      "789 Demo Blvd, City, Country"
    ];
    res.render('addresses', {addresses: addresses});
});

app.listen(3000);
```


---


3 Möglichkeit:  Server-Programm generiert JSON für Single Page Applications
```js
const express = require('express');
const app = express();

app.get('/addresses', function(req, res) {
    // Definiere die Adressen
    const addresses = [
        "123 Example St, City, Country",
        "456 Sample Rd, Town, Country",
        "789 Demo Blvd, City, Country"
    ];
    
    // Setze den Content-Type auf application/json
    res.type('application/json');

    // Sende die Adressen als JSON
    res.send(JSON.stringify(addresses));
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, function() {
    console.log(`Server is running on port ${PORT}`);
});
```


