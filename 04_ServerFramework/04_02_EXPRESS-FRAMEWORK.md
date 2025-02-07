

# 1 Erweiterung von Connect

![](images/Pasted%20image%2020250203101257.png)



Route Path
Route Method
Routing
Route Handler
Template Engine

- Connect: Einbindung von Middleware-Funktionen für Infrastrukturaufgaben
- Express: Erweiterung von Connect um Mechanismen zur Erledigung von Infrastrukturaufgaben
- Express kapselt Connect, d.h. Objekte und Methoden von Connect stehen auch in Express zur Verfügung


- Route Path: Endpunkt einer Webanwendung (Pfadnamen einer URI)
- Route Method: HTTP-Methode für die eine Route definiert ist
- Routing
    - Bearbeitung einer HTTP-Anfrage und Erstellung der Antwort . 
    - Genauer: Durchleiten einer Anfrage durch Middleware-Funktionen und verschiedene Module der Webanwendung und Zurückleitung der Antwort
- Route Handler: Callback-Funktion einer Middleware-Funktion bzw. eines Moduls der Webanwendung
- Template Engine: Erstellung von dynamischen HTML-Ausgaben basierend auf Templates

# 2 Integrierte Module


Anfragen und Antworten 

| Modul                | Beschreibung                                                                                                                                                                                          |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `**basicAuth**`      | - Fügt einer Anwendung Unterstützung für die http-Authentifizierung hinzu  <br>- Authentifizierung erfolgt über einen Callback, so dass beliebige Strategien implementiert werden können              |
| `**compress**`       | - Komprimierung von ausgehenden Antworten auf Basis von `gzip` und `deflate`  <br>- Modul prüft eigenständig, ob der vom Benutzer verwendete Webbrowser komprimierte Daten akzeptiert                 |
| `**csrf**`           | - Schützt Webanwendungen vor Cross Site Request Forgery  <br>- Generiert für jeden Benutzer ein Token, welches bei jeder Anfrage validiert wird  <br>Benötigt die Module `cookieParser` und `session` |
| `**limit**`          | - Begrenzt das vom Webserver akzeptierte Datenvolumen von eingehenden Anfragen  <br>- Überschreitet die Größe einer Anfrage das zulässige Datenvolumen, wird diese abgebrochen                        |
| `**methodOverride**` | - Fügt einer Anwendung Unterstützung für unechte http-Methoden hinzu                                                                                                                                  |
| **`responseTime`**   | - Berechnet die Zeit, die zur Beantwortung einer eingehenden Anfrage benötigt wird, und sendet diese als Header in der ausgehenden Antwort an den Webbrowser zurück                                   |

---


Parser

| Modul              | Beschreibung                                                                                                                                                           |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `**bodyParser**`   | - Verarbeitet den Body von eingehenden Anfragen  <br>- Unterstützt die Formate `json`, `urlencoded` und `multipart`                                                    |
| `**cookieParser**` | - Analysiert die in einer eingehenden Anfrage enthaltenen Cookies  <br>- Stellt Cookies im Objekt `req.cookies` zur Verfügung                                          |
| `**json**`         | - Verarbeitet den Body von eigehenden Anfragen  <br>- Unterstützt das Format `json`  <br>- Wird intern vom `bodyParser`-Modul verwendet                                |
| `**multipart**`    | - Verarbeitet den Körper von eigehenden Anfragen  <br>- Unterstützt das Format `multipart/form-data`  <br>- Wird intern vom `bodyParser`-Modul verwendet               |
| `**urlencoded**`   | - Verarbeitet den Körper von eigehenden Anfragen  <br>- Unterstützt das Format `application/x-www-form-urlencoded`  <br>- Wird intern vom `bodyParser`-Modul verwendet |


----


 Webserver und Sessions

|Modul|Beschreibung|
|---|---|
|`**cookieSession**`|- Erstellt Sitzungen (Sessions) auf Basis von Cookies  <br>- Gesamte Session wird in ein Cookie serialisiert und von dort wieder serialisiert|
|`**directory**`|- Formatierte Ausgabe einer Dateiliste für ein angegebenes Verzeichnis  <br>- Verwendung von Filtern möglich  <br>- Möglichkeit der Ausgabe versteckter Dateien|
|`**favicon**`|- Sendet die Datei `favicon`  <br>- favicon: Kofferwort von favorite und icon (d.h. Favoritensymbol)  <br>- Verwendung eines Caches, so dass die Datei nicht für jede Anfrage aus dem Dateisystem gelesen werden muss|
|`**session**`|- Fügt einer Webanwendung Unterstützung für Sessions hinzu  <br>- Sessions werden durch `SessionID` repräsentiert, die durch Cookies transportiert werden  <br>- Zustand einer Session wird im Arbeitsspeicher gehalten  <br>- Alternative Einbindung persistenter Speicherlösungen möglich  <br>- Erfordert `cookieParser`-Modul|
|`**static**`|- Senden von statischen Dateien aus dem Dateisystem  <br>- Kein Caching|
|**`staticCache`**|- Cache für das static-Modul  <br>- Umsetzung einer `Least-Recently-Used`-Strategie  <br>- Begrenzung der Anzahl und der Größe der gecachten Dateien|

---

Werkzeuge 

| Modul              | Beschreibung                                                                                                                      |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| `**errorHandler**` | - Vorgefertigtes Fehler-Modul  <br>- Liefert im Falle eines Fehlers die entsprechende Fehlermeldung als Antwort an den Webbrowser |
| `**logger**`       | - Protokolliert alle eingehenden Anfragen in einem übergebenen Stream                                                             |


# 3 Wichtige Methoden und Objekte im Überblick


Methoden des Application-Objekts

|Methoden|Beschreibung|
|---|---|
|`**app.set(name, value)**`|Setzt eine Umgebungsvariable zur Konfiguration von Express|
|`**app.get(name)**`|Liest den Wert einer Umgebungsvariablen|
|`**app.use([path], callback)**`|Fügt eine Middleware-Funktion in Form eines Callbacks in die Route der Webanwendung ein und bindet sie optional in ein bestimmtes Verzeichnis|
|`**app.VERB(path, [callback...], callback)**`|Fügt eine oder mehrere Module der Webanwendung hinzu und bindet sie an ein bestimmtes Verzeichnis und eine bestimmte HTTP-Methode|



Eigenschaften des Request-Objekts

|Methoden und Objekte|Beschreibung|
|---|---|
|`**req.query**`|Enthält die Query-String-Parameter einer aufgerufenen URL|
|`**req.params**`|Enthält die Routing-Parameter einer URL|
|`**req.body**`|Enthält den Body-Teil einer Anfrage, unterstützt durch bodyParser|
|`**req.param(name)**`|Liefert den Wert eines bestimmten Anfrageparameters|
|`**req.path**`, `**.host**` und `**.ip**`|Liefern Pfad, Hostname und IP-Adresse|
|`**req.cookies**`|Liefern Cookies einer Anfrage, bestimmt durch das cookieParser-Modul|


Eigenschaften des Response-Objekts

| Methoden und Objekte                       | Beschreibung                                                                                                                   |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| `**res.status(code)**`                     | Setzt den Status-Code einer HTTP-Antwort                                                                                       |
| `**res.set(field, [value])**`              | Setzt Parameter des Antwort-Headers                                                                                            |
| `**res.cookie(name, value, [options])**`   | Setzt Antwort-Cookie                                                                                                           |
| `**res.redirect([status], url)**`          | Anweisung eines Redirects zur der angegebenen URL                                                                              |
| `**res.send([body \| status], [body])**`   | Zusammenstellung der HTTP-Antwort und automatische Definition wichtiger Header-Parameter, z.B. Content-Type und Content-Length |
| `**res.render(view, [locals], callback)**` | Zusammenstellung einer HTML-Seite basierend auf einem Template                                                                 |



# 4 Route Handlers


- `**app.VERB**` mit `**VERB=get**`, `**post**`, `**put**` usw. übergibt Route Handler (Callback-Funktion) für die Beantwortung von HTTP-Anfragen einer bestimmten Methode an einen bestimmten Pfad
- `**app.all**` bindet alle Methoden an einen Route Handler, d.h. Ausführung geschieht unabhängig von der verwendeten HTTP Methode

```js
let express=require('express');
let app=express();

app.get('/', function(req, res) {
  res.send('GET request to the homepage');
});

app.post('/', function(req, res) {
  res.send('POST request to the homepage')
});

app.all('/secret', function(req, res, next) {
  console.log('Betreten des geschützten Bereichs');
  next();
});
```



# 5 Route Paths


Zeichenketten
```js
let express=require('express');
let app=express();

...
app.get('/', function(req, res) {
  res.send('Anfrage an / (root)');
});

app.get('/about', function(req, res) {
  res.send('Anfrage an /about');
});

app.get('/random.txt', function(req, res) {
  res.send('Anfrage an /random.txt');
});
```



Parameter
```js
let express=require('express');
let app=express();

...
app.get('/foo/:id', function(req, res) {
  res.send('Anfrage an /foo/123 oder /foo/456 usw.');
});

app.get('/foo/:id/:subid', function(req, res) {
  res.send('Anfrage an /foo/123/456 oder /foo/456/789 usw.');
});
```



Reguläre Ausrücke
```js

let express=require('express');
let app=express();

...
app.get('/ab?cd', function(req, res) {
  res.send('Anfrage an /acd oder /abcd');
});

app.get('/ab+cd', function(req, res) {
  res.send('Anfrage an /abcd, /abbcd, /abbbcd usw.');
});

app.get('/ab*cd', function(req, res) {
  res.send('Anfrage an /acd, /abcd, /abbcd usw.');
});

app.get('/ab(cd)?e', function(req, res) {
  res.send('Anfragen an /abe und /abcde')
});

app.get('/.*fly$/', function(req, res) {
  res.send('Anfragen an z.B. /butterfly/ oder /dragonfly/ im Pfadnamen, aber nicht /butterflyman/');
});
```


# 6 Registrierung von Route Handlers


Ein Route Handler
```js

let express=require('express');
let app=express();

app.get('/example/a', function(req, res) {
  res.send('Hallo von /example/a');
});
```


Mehrere Route Handler
```js

let express=require('express');
let app=express();

app.get('/example/b', function(req, res, next) 
    {
      res.write('Weiterleitung zur nächsten Middleware');
      next(); 
      res.end();
    }, 
    function(req, res) {
      res.write('Hallo von /example/b');
    }
);
```


Mehrere Route Handler als Array
```js

let express=require('express');
let app=express();

let c0=function(req, res, next) {
  console.log('c0'); next(); 
  res.send('Hallo von /example/c');
};
let c1=function(req, res, next) {
  console.log('c1'); next;
};
let c2=function(req, res) {
  console.log('c2');
};
app.get('/example/c', [c0, c1, c2]);
```



# 7 Konfiguration von Route Handlers mittels Router


![](images/Pasted%20image%2020250206202021.png)


Router – Zusammenstellung einer Middleware

这个文件存在了 `.vorlesung` 下 
```js
let express=require('express');
let router=express.Router();

router.use(function timelog(req, res, next) {
  console.log('Time: '+ Date.now());
  next();
});

router.get('/', function(req, res) {
  res.send('Homepage der Vorlesung');
});

router.get('/about', function(req, res) {
  res.send('Über die Vorlesung');
});

module.exports=router;

```


Einhängen eines Routers in einen Pfad
```js
let express=require('express');
let app=express();

let webtech=require('./vorlesung');
app.use('/webtech', webtech);

```



# 8 HTML Templates und Template Engines

JADE
```html
html
	head
    	title!=title
	body
    	h1!=message

```


Embedded JavaScript Template (EJS)
```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
  </head>
    <body>
      <h1><%= message %></h1>
    </body>
</html>
```

Nutzung von Templates
在 执行一个 get request 的时候, 去执行渲染 一个预定以好的 html template 
```js
...
//Einbindung der Template Engine
app.set('view engine', 'ejs');

//Übergabe des Template-Verzeichnisses
app.set('views', '/app/views');   //  /app/views 这个文件夹下有一个 名字 为 index 的文件 

//Füllen des Templates mit Daten
// 这个是怎么知道 要去 render 那个 template 的 
app.get('/', function(req, res) {
  res.render('index', {title: 'Hey', message: 'Hallo!'});
});
```


- **_Template Engines_** unterstützen die Generierung dynamischer HTML-Seiten mit Hilfe vorgefertigter **_Templates_** (Vorlagen)
- Templates enthalten statischen HTML-Code sowie Parameter, die vor der Auslieferung mit Werten belegt werden (vergleiche PHP oder Java Server Pages)
- Unterstützte Template-Formate bzw. Template Engines
    - Jade
    - Embedded JavaScript (EJS)
    - JavaScript HTML (JsHTML)
- Einbindung einer Template Engine in eine Webanwendung durch Umgebungsvariable `**view engine**`
- Übergabe des Template-Verzeichnisses durch Umgebungsvariable `**views**`