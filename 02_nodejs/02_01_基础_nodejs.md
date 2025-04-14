

# 1 Die Javascript-Laufzeitumgebung in node.js


**node.js** ist kein Webserver im eigentlichen Sinn, sondern vielmehr eine Laufzeitumgebung für **serverseitiges JavaScript**, die als Webserver eingesetzt werden kann.

Node.js ist eine asynchrone, eventbasierte Laufzeitumgebung und wurde für die Entwicklung von skalierbaren Netzwerkanwendungen entworfen. 
Ein großer Unterschied zwischen Node.js und üblichen Server-Technologien ist, dass Node.js auf einem "Non Blocking"-Laufzeitmodell basiert. Im Vergleich zu Laufzeitmodellen, die Nebenläufigkeit über parallele Threads umsetzen, wie z.B. der Apache-Webserver, ==verwendet Node.js nur einen einzigen Thread, der eine Schleife ausführt==. Diese, die sogenannte **"Event-Loop"**, arbeitet der Reihe nach alle eingehenden Events ab. 

Events können dabei z.B. eingehende Web-Anfragen über das Netzwerk, Laden von Daten aus dem Dateisystem oder Antworten von selbstgestellten Anfragen an andere Web-Services sein. 
Das "Geheimnis" dabei ist, dass (fast) alle Events in Node.js durch Callbacks abgearbeitet werden. Ein ähnliches Prinzip kennen Sie schon, wenn Sie bereits mit AJAX im Web-Browser gearbeitet haben. Wenn Sie im Browser asynchrone Daten anfordern, übergeben Sie in der Regel eine Funktion, die ausgeführt wird, wenn die angeforderten Daten geladen wurden. Genau dieses Programmiermuster nennt man einen Callback.



![](images/Pasted%20image%2020250116213521.png)


Node.js ist eine serverseitige Plattform, die es ermöglicht, JavaScript außerhalb des Browsers auszuführen. Es basiert auf der V8-JavaScript-Engine von Google Chrome und ist geeignet für die Erstellung skalierbarer, ereignisgesteuerter und nicht blockierender Serveranwendungen.

Grundkonzepte von Node.js
1. Ereignisgesteuerte Architektur: Node.js verwendet ein Event-Loop-Modell, wodurch es skalierbar und effizient ist.
2. Nicht-blockierende I/O: Aufgaben wie Dateioperationen oder Netzwerkanfragen blockieren nicht den Haupt-Thread.
3. Modularität: Node.js verwendet das CommonJS-Modulsystem, sodass Funktionen und Module einfach wiederverwendet werden können.


## 1.1 **Arbeitsweise der Event-Loop**  

Vereinfacht kann man sich vorstellen, dass sich Node.js für alle Events, die behandelt werden sollen, einen Event-Handler mit der Callback-Implementierung vorhält. Tritt nun ein Event auf, wird es in eine Liste (Event-Queue) hinzugefügt. Die Event-Loop arbeitet nun alle Callbacks zu den Events der Reihe nach ab.

[![Webserver-nodeJs-Event-Loop.png](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/thumb/7/74/Webserver-nodeJs-Event-Loop.png/650px-Webserver-nodeJs-Event-Loop.png)](https://isp.eduloop.de/mediawiki/images/isp.eduloop.de/7/74/Webserver-nodeJs-Event-Loop.png)

Überträgt man die Art der Verarbeitung auf einen einfachen Web-Server mit Datenbank, kann man sich folgendes Beispiel ableiten:

1. Der Server ist gestartet, die Event-Queue ist leer und ein Event-Handler wartet auf eingehende Web-Anfragen.  
2. Trifft eine Anfrage ein, wird ein passendes Event in die Event-Queue gelegt. Da keine anderen Events warten, wird mit der Verarbeitung begonnen.  
3. Für die Verarbeitung der Anfrage werden Daten aus der Datenbank benötigt. Es wird ein Event-Handler inkl. Callback erstellt, der die Verarbeitung fortführt, wenn die Daten aus der Datenbank geladen sind, und die Anfrage an die Datenbank abschickt. Damit ist die Verarbeitung im Moment für die Web-Anfrage abgeschlossen und das Event wird aus der Event-Queue entfernt.  
4. Die in der Zwischenzeit empfangenen Web-Anfragen werden ebenfalls als Event in die Event-Queue abgelegt. Während auf die Daten aus der Datenbank gewartet wird, werden diese der Reihe nach abgearbeitet. Dabei können Web-Anfragen, die ohne Datenbankaufruf auskommen, auch schon komplett bearbeitet werden. Die Anfrage an die Datenbank blockiert also die weitere Verarbeitung nicht (Stichwort: Non Blocking).  
5. Die Daten aus der Datenbank sind angekommen und erstellen ein Event zur Weiterverarbeitung der Anfrage. Dieses Event wird ebenfalls hinten in die Event-Queue hinzugefügt. Ist dieses Event vorne in der Event-Queue angekommen, wird die Verarbeitung mit den Daten der Datenbank fortgesetzt und eine Antwort an den zugehörigen Browser gesendet.

Die oben beschriebenen Events vermischen sich natürlich mit vielen weiteren server-internen Events. Diese funktionieren aber in ähnlicher Weise. Wichtig ist hierbei, dass tatsächlich nur ein Thread aktiv ist, der über die Event-Loop eine fortlaufende Verarbeitung der Events vornimmt.

  

# 2 Integrierte Module (Auswahl)

| Modulname           | Beschreibung                                                     |
| ------------------- | ---------------------------------------------------------------- |
| `**assert**`        | Integriertes Testframework                                       |
| `**buffer**`        | Modul für den Umgang mit binären Rohdaten                        |
| `**child_process**` | Verwaltung von Kindprozessen                                     |
| `**cluster**`       | Prozessverwaltung für Mehrkernprozessoren                        |
| **`console`**       | console-Funktionalität in JavaScript                             |
| `**crypto**`        | Sichere Verbindung und Verschlüsselung                           |
| `**debugger**`      | Mittel zur Laufzeitanalyse von Applikationen                     |
| `**dgram**`         | Kommunikation über UDP                                           |
| `**dns**`           | Namensauflösung unter Node.js                                    |
| `**domain**`        | Sammlung von Exceptions und Fehlern einer Gruppe von Operationen |
| `**events**`        | Basismodul für Objekte, die Events erzeugen                      |
| `**fs**`            | Dateisystemoperationen                                           |
| `**http**`          | HTTP-Client und -Server                                          |
| `**https**`         | HTTPS-Client und -Server                                         |
| `**module**`        | Modulloader von Node.js                                          |
| `**net**`           | Client- und Serverkomponenten für Netzwerkstreams                |
| `**os**`            | Betriebssystemoperationen auslesen                               |

|Modulname|Beschreibung|
|---|---|
|`**path**`|Umgang mit Pfadinformationen|
|`**punycode**`|Codierung und Decodierung von Punycode-Zeichenketten|
|`**querystring**`|Erstellung und Parsing von URL Query Strings|
|`**readline**`|Einlesen von Informationen über einen Stream wie die Standardeingabe|
|`**repl**`|Interaktive Node.js-Shell|
|`**stream**`|Schreib- und lesbare Datenstreams|
|`**string_decoder**`|Umwandlung von Buffer-Objekten in Zeichenketten|
|`**timers**`|Zeitabhängige Funktionen|
|`**tls**`|Verschlüsselte Kommunikation über Datenstreams|
|`**tty**`|Zwischenschicht zum Ansprechen von Standardein- und -ausgabe|
|`**url**`|Funktionalität zum Umgang mit URLs|
|`**util**`|Hilfsfunktionen|
|`**vm**`|Ausführungsumgebung für JavaScript-Code|
|`**zlib**`|Möglichkeit zum Komprimieren und Dekomprimieren von Daten|

# 3 Möglichkeiten der Kommunikation


![](images/Pasted%20image%2020250118132724.png)


![](images/Pasted%20image%2020250118132822.png)

- Traditionelle Kommunikation über das HTTP-Protokoll

- Kommunikation über eine sichere HTTP-Verbindung
- Umsetzung der sicheren Verbindung über Transport Layer Security (TLS) zwischen TCP und HTTP
- Authentifizierung (des besuchten Servers oder gegenseitig), Vertraulichkeit, Integrität

- Websockets: direkte Kommunikation über das Transportprotokoll (TCP oder UDP)


# 4 Einfache Node.js-Server


## 4.1 Rückgabe von Text

Node.js 作为 server, 收到 client 的 anfrage, 然后 答复 这个 Client 某些信息 

1 addListener

```js
// run `node server01.js in the terminal 
console.log(`Hello Node.js v${process.versions.node}!`);
let Server=require('http').Server;  // 从 http 中拿一个 server 
let server=new Server();
server.addListener('request', function(request, response) {
  response.writeHead(200, {
    'content-typ': 'text/plain; charset=utf-8'
  });
  response.write('Hello ');
  response.end('World\n');
});
server.listen(8080);
console.log('Webserver wird ausgeführt.');
```



运行 noder server01.js 

得到 
![](images/Pasted%20image%2020250118133338.png)


console 中得到 
```
hello nodejs v18.2a
Webserver wird ausgeführt
```

- Registrierung der Callback-Funktion mittels `**addListener**`-Methode auf dem `**server**`-Objekt
- `**addListener**` hat gleiche Funktionsweise wie `**addEventListener**` bei JavaScript im Browser
- `**addListener**` ist eine Alias für `**on**`
- Übergabe einer Callback-Funktion bei Erzeugung des Servers, die bei jeder HTTP-Anfrage durch die Laufzeitumgebung aufgerufen wird
- `**request**`: Zugriff auf Header-Felder und Inhalte der HTTP-Anfrage
- `**response**`: Zugriff auf Header-Felder und Inhalte der HTTP-Antwort
- Zusammenstellen einer HTML-Seite mittels **writeHead** (für Header-Parameter) bzw. **write** auf dem **response**-Objekt


---

2 createServer 
Node.js 作为 server, 收到 client 的 anfrage, 然后 答复 这个 Client 某些信息 
- `**createServer**` als alternative (und bevorzugte) Methode zur Erzeugung des Webservers
- Übergabe einer Callback-Funktion, die bei einem eintreffenden Request aufgerufen wird
- Ersetzt `**addEventListener**`

```js
// run `node server02.js in the terminal 
console.log(`Hello Node.js v${process.versions.node}!`);
let http=require('http');
http.createServer(function(request, response) {
  response.writeHead(200, {
    'content-typ': 'text/plain; charset=utf-8'
  });
  response.write('Hello ');
  response.end('World\n');
}).listen(8080);
console.log('Webserver wird ausgeführt.');
```


得到 
![](images/Pasted%20image%2020250118133507.png)


## 4.2 Auslesen von QUERY-Parametern

```js
// run `node server04.js` in the terminal
console.log(`Hello Node.js v${process.versions.node}!`);
let http=require('http');
let url=require('url');
http.createServer(function(request, response) {
  response.writeHead(200, {
    'content-typ': 'text/html; charset=utf-8'
  });
  let urlString=url.parse(request.url,true);  // 用于 读取 ulr string 中的 内容 
  let body='<!DOCTYPE html>'+
  '<html>'+
  '<head>'+
  '<meta charset="utf-8">'+'<title>Node.js-Demo</title>'+
  '</head>'+
  '<body>'+
  '<h1 style="color: green">Hello '+urlString.query.name+'</h1>'+  // urlString.query.name 用来读取 query 中的 name 
  '</body>'+
  '</html>';
  response.end(body);
}).listen(8080);
console.log('Webserver wird ausgeführt.');
```

得到 

![](images/Pasted%20image%2020250118150312.png)

如果在 url 中 不写 ?name=thomas , 则会显示 Hello undefined 


## 4.3 Auslesen des Request Body

一个 request 一般来说 会分批到来 ,  读取每个 packet 中的 data, 
以及 在 包全都到来后, 去做 什么动作 

```js
// run `node server05.js` in the terminal
console.log(`Hello Node.js v${process.versions.node}!`);
let http=require('http');

http.createServer(function(request, response) {
  let body='';
  request.on('data', function(data) {
    body+=data.toString();
  });
  request.on('end', function() {
    console.log(body);
    response.writeHead(200, {'content-type': 'text/plain'});
    response.end(body);
  });
}).listen(8080);

console.log('Webserver wird ausgeführt.');
```


Testen mit:
```sh
curl -d "user=foobar&pass=12345&id=blablabla&ding=submit" http://localhost:8080
```


- Problem: Body einer HTTP-Anfrage kann in mehreren Teilen übertragen werden, d.h. in mehreren TCP-Segmenten
- Betroffen sind `**POST**`- und `PUT`-Nachrichten
- Verschiedene Events informieren Node.js-Server über den Zustand einer HTTP-Anfrage
    - `**data**`: wird ausgelöst, sobald neue Daten geliefert werden
    - `**end**`: wird ausgelöst, sobald die Übermittlung sämtlicher Teile einer Anfrage abgeschlossen ist
    - `**close**`: signalisiert, dass die TCP-Verbindung beendet wurde
- Bearbeitung der Ereignisse durch Callback-Funktionen
- Beispiel: die in einem TCP-Segment übermittelten Daten werden in der Callback-Funktion übergeben und lassen sich in einem Buffer zwischenspeichern
- Beachte: `**end**` und `**close**` werden pro Anfrage nur einmal ausgelöst
- Beachte: tritt `**close**` vor `**end**` auf, ist Übertragung unvollständig
- Beachte: `**on**`-Methode alternativ für `**addListener**`


# 5 Auslieferung statischer Dateien

nodeserver   去读取一个 datei, 把他发给前端 

```js
const http = require('http');
const fs = require('fs');
const url = require('url');
const path = require('path');

const hostname = process.env.HOST || 'localhost';
const port = process.env.PORT || 80;

http.createServer( function (request, response) {

   //Angefragter Pfadname in der URL separieren
   let pathname = url.parse(request.url).pathname;

   //Wen kein Pfadname angegeben wurde, indedx.html per default laden
   if (pathname=='/') pathname='/index.html';

   //Pfadname auf dem Server entspricht dem absoluten Pfad des Webserver (=Current Working Directory, cwd)+Pfadname der URL
   let fullpath = path.join(process.cwd(),pathname);

   console.log("Request for " + pathname + " received.");
   fs.readFile(fullpath, function (err, data) {
      if (err) {
         console.log(err);
         response.writeHead(404, {'Content-Type': 'text/html'});
      } else {

         //Je nach Art des angefragten Inhalts muss im HTTP-Response-Header der Content-Type dementsprechend gesetzt werd
         let fileNameExtension=path.extname(pathname);

         if (fileNameExtension===".html") {
            response.writeHead(200, {'Content-Type': 'text/html'});
         }
         else if (fileNameExtension===".css") {
            response.writeHead(200, {'Content-Type': 'text/css'});
         }
         else if (fileNameExtension===".js") {
            response.writeHead(200, {'Content-Type': 'text/javascript'});
         }
         else if (fileNameExtension===".jpg") {
            response.writeHead(200, {'Content-Type': 'image/jpeg'});
         }
         else if (fileNameExtension===".png") {
            response.writeHead(200, {'Content-Type': 'image/png'});
         }
         else response.writeHead(200, {'Content-Type': 'text/plain'});
         response.write(data);
      }
     response.end();
   });
}).listen(port, hostname, ()=>console.log('Webserver wird ausgeführt unter http://'+hostname+':'+port+'.'));
console.log('Webserver wird gestartet.');
```

- Bisher: dynamische Erzeugung einer HTTP-Antwort
- Hier: Übertragung einer statischen Datei (text oder HTML) aus dem Dateiverzeichnis des Servers
- Benötigte Module: `**http**`, **`url`**, `**fs**`
- `**url**`: Operationen auf URLs, z.B. Extraktion des Pfadnamens
- `**fs**`: Operationen auf dem lokalen Dateisystem, zum Beispiel Lesen oder Schreiben von Dateien
- Beachte: Lesen einer Datei erfolgt asynchron, d.h. Verarbeitung der gelesenen Daten durch Callback-Funktion

# 6 Node.js als Client

Node.js 作为 client, 不是作为 server,   . 他主动发给 别的 机器 一些信息 

就是 server-seite and Client-server 放到一起使用 

![](images/Pasted%20image%2020250118151344.png)


下面两个 都是 发送一个 request , 并且 在同一个 request 函数汇总已经定义好了 response 的 function 

Standardmethode
```js
// run `node client01.js in the terminal 
console.log(`Hello Node.js v${process.versions.node}!`);
let https=require('https');
https.request('https://dummyjson.com/products/1', function(response) {
  response.on('data', function(data) {
    console.log(data.toString());
  });
}).end();
```


Konfiguration der Request-Parameter
```js
// run `node client02.js in the terminal 
console.log(`Hello Node.js v${process.versions.node}!`);
let https=require('https');
https.request({
  host:'dummyjson.com',
  port: 443,
  method: 'GET',
  path:'/products',
  headers: {
    'accept': 'text/json',
    'userAgent': 'node.js'
  },
  agent: false}, function(response) {
    response.on('data', function(data) {
      console.log(data.toString());
  });
}).end();
```



Get-Methode
```js
// run `node client03.js in the terminal 
console.log(`Hello Node.js v${process.versions.node}!`);
let https=require('https');
https.get('https://dummyjson.com/products', function(response) {
  response.on('data', function(data) {
    console.log(data.toString());
  });
}).end();
```



Senden von Daten im Body der Anfrage
就是 request 的 body 中的内容 通过 request.write 
```js
// run `node client04.js in the terminal 
console.log(`Hello Node.js v${process.versions.node}!`);
let https=require('https');
let request=https.request({
  host:'dummyjson.com',
  port: 443,
  method: 'POST',
  path:'/products/add',
  headers: {},
  agent: false}, function(response) {}
);

request.write('{"title":"iPhone 9","description":"An apple mobile which is nothing like apple","price":549,"discountPercentage":12.96,"rating":4.69,"stock":94,"brand":"Apple","category":"smartphones","thumbnail":"https://i.dummyjson.com/data/products/1/thumbnail.jpg","images":["https://i.dummyjson.com/data/products/1/1.jpg","https://i.dummyjson.com/data/products/1/2.jpg","https://i.dummyjson.com/data/products/1/3.jpg","https://i.dummyjson.com/data/products/1/4.jpg","https://i.dummyjson.com/data/products/1/thumbnail.jpg"]}');

request.end();
```


## 6.1 http.get('url', res ⇒ {})

In **Node.js** wird `http.get()` verwendet, um eine **HTTP-GET-Anfrage** zu senden. Es ist eine Methode aus dem **`http`-Modul** und eignet sich besonders für einfache Abrufe von Daten von einer URL.

**Wie funktioniert `http.get()`?**
4. sendet eine GET-Anfrage an die angegebene URL.
5. gibt ein http.ClientRequest-Objekt zurück, mit dem man Fehlerbehandlung durchführen kann.
6. verarbeitet die Antwort (res), die ein http.IncomingMessage-Objekt ist:
    1. res.on('data', callback) – Empfängt die Daten in Chunks.
    2. res.on('end', callback) – Wird aufgerufen, wenn alle Daten eingetroffen sind.

Grundlegende Syntax:
```js
const http = require('http');

http.get('http://irgendeine-url.com', (res) => {
    // Hier kommt der Funktionsbody rein
});
```


Hierbei handelt es sich nicht um dieselbe res wie bei http.createServer((req, res)=>{}) !!


# 7 Der Unterschied zwischen Responses in http.createServer() und in http.get() 

Der Unterschied liegt in der Rolle im HTTP-Kommunikationsprozess.

res in http.createServer((req, res) => {})
- Rolle: Hier ist res das Response-Objekt, mit dem der Node.js-Server eine Antwort an den Client sendet.
- Verwendet für: Das Erstellen und Senden von HTTP-Antworten mit Hilfe von Response-Methoden (res.writeHead, res.write, res.end)

res in http.get('url', res => {})
- Rolle: Hier ist res das Response-Objekt der empfangenen HTTP-Antwort, wenn der Node.js-Server als Client eine Anfrage an eine andere URL sendet.
- Verwendet für: Das Verarbeiten der Antwort von einem externen Server mit Hilfe von Daten-Stream-Methoden (.on('data'), .on('end')).
- Letzte Woche haben wir die Daten-Stream-Methoden genutzt, um Chunks – die Datenblöcke einer Client-Anfrage – zu empfangen, zusammenzusetzen und zu verarbeiten. Diese Woche kommt noch das Erhalten von Daten, die von einem Provider-Server kommen.


Tabellarischer Vergleich:

|   |   |   |
|---|---|---|
|Kontext|res bedeutet...|Was macht es?|
|http.createServer((req, res) => {})|Server-Response (Antwort an den Client)|sendet die Antwort vom Server an den Client.|
|http.get('url', res => {})|Client-Response (Antwort von externem Server)|verarbeitet die Antwort von einem anderen Server.|
