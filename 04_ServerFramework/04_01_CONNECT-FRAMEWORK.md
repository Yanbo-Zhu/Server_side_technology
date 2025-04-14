

![](images/Pasted%20image%2020250202201849.png)

FUNKTIONALITÄTEN EINER ANWENDUNG
- Kernfunktionalitäten: primäre Aufgaben zur Umsetzung einer domänenspezifischen Geschäftslogik
- Infrastrukturaufgaben: sekundäre, anwendungsunabhängige Managementaufgaben


Middleware
- Verschiedene Middleware-Funktionen zur Wahrnehmung unterschiedlicher Aufgaben
- Beispiele: Logging, Authentifizierung, Sitzungsmanagement, Internationalisierung
- Sequentielle Ausführung von Funktionen bevor eine Anfrage die Anwendung erreicht


Connect-Framework
- Middleware für Node.js
- Ziel: komponentenorientierte Entwicklung und Integration von Infrastrukturcode
- Zwei Arten von Middleware-Funktionen
    - **_Filter_** werten eingehende Anfrage (ohne Veränderung) aus und leiten sie an das nächste Modul weiter
    - **_Provider_** fungieren als Stellvertreter der Webanwendung und leiten Anfrage nach Bearbeitung nicht weiter

# 1 Struktur einer Connect-Anwendung


![](images/Pasted%20image%2020250202203943.png)

```js
//run node server01.js to start the server
let connect=require('connect');
let app=connect();
 
let helloWorld=function(request, response, next) {
  response.setHeader('Content-Type', 'text/plain');
  response.end('Hello World from Connect');
};
 
app.use(helloWorld);
app.listen(8080);
```


Grundstruktur einer Connect-Anwendung
- Import des Connect-Moduls
- Aufruf der **`connect`**-Methode liefert einen Server, der die jeweilige Anwendung repräsentiert
- Ersetzt die bisherige `**createServer**`-Methode aus dem `**http**`/`**https**`-Modul


Middleware-Funktionen
- Middleware-Funktionen sind Callback-Funktionen mit der Signatur `**function(req, res, next)**`
    - `**req**`: Objekt der HTTP-Anfrage
    - `**res**`: Objekt der HTTP-Antwort
    - `**next**`: nächste auszuführende Middleware-Funktion
- Hinzufügen der Middleware-Funktion mit `**use**`-Methode


# 2 Sequentialisierung und Struktur

```js
let connect=require('connect');
let app=connect();

let logger=function(request, response, next) {
  console.log("Auf dem Hinweg zu Geschäftslogik");
  console.log(request.method, request.url);
  next();
  console.log("Auf dem Rückweg von der Geschäftslogik");
};

let helloWorld=function(request, response) {
  console.log("Bei der Geschäftslogik angekommen");
  response.setHeader('Content-Type', 'text/plain');
  response.end('Hello World!');
};

app.use(logger);
app.use(helloWorld);
app.listen(8080);
```


Sequentialisierung von Middleware-Funktionen
- Sequentialisierung durch wiederholte Ausführung der `**use**`-Methode
- Achtung: First-in-First-Out, d.h. Callbacks werden bei eingehenden Anfragen in der Reihenfolge abgearbeitet, in der sie mit `**use**` hinzugefügt wurden
- Weitergabe einer Anfrage zwischen Middleware-Funktionen durch Aufruf von `**next**` (ohne Übergabe von `**req**` und `**res**`)

1. **Sequential Execution via `use` Method:**
    - Middleware functions are added using `app.use()`.
    - The order in which they are added determines their execution sequence.
2. **FIFO Processing:**
    - Incoming requests pass through middleware functions **in the order they were registered**.
    - Each middleware can process or modify the request before passing it to the next function.
3. **Passing Control with `next()`**
    - Calling `next()` moves the request to the next middleware in the stack.
    - The request (`req`) and response (`res`) objects are **automatically forwarded**, so there's no need to pass them explicitly.


---

Struktur einer Middleware-Funktion
- Anweisungen vor `**next**` werden bei der "Durchleitung" der Anfrage bearbeitet
- Anweisungen nach `**next**` werden beim Zurückspielen der Antwort bearbeitet, d.h. nach Bearbeitung der Anfrage durch die Webanwendung

- **Instructions before `next()`**
    - These execute while the request is being processed **on the way to the final handler**.
    - Example: Logging, authentication, request modifications.
- **Instructions after `next()`**
    - These execute when the response is being sent **back to the client**.
    - Example: Response modifications, cleanup, error handling.


---

**Example: Middleware in Express.js**

```js
const express = require('express');
const app = express();

// First middleware: Logs request details
app.use((req, res, next) => {
    console.log(`Request received: ${req.method} ${req.url}`);
    next(); // Pass control to the next middleware
});

// Second middleware: Adds a custom header
app.use((req, res, next) => {
    res.setHeader('X-Custom-Header', 'Middleware Test');
    next();
});

// Route handler
app.get('/', (req, res) => {
    res.send('Hello, Middleware!');
});

app.listen(3000, () => console.log('Server running on port 3000'));

```

**Execution Flow**
4. **Client makes a request (`GET /`)**
5. **Middleware 1 logs the request and calls `next()`**
6. **Middleware 2 adds a custom response header and calls `next()`**
7. **The final route handler sends the response (`Hello, Middleware!`)**


# 3 Mounting 

```js
let connect=require('connect');
let app=connect();

let logger=function(request, response, next) {
  console.log("Auf dem Hinweg zu Geschäftslogik");
  console.log(request.method, request.url);
  next();
  console.log("Auf dem Rückweg von der Geschäftslogik");
};

let helloWorld=function(request, response) {
  console.log("Bei helloWorld angekommen");
  response.setHeader('Content-Type', 'text/plain');
  response.end('Hello World!');
};

let goodbyeWorld=function(request, response) {
  console.log("Bei goodbyeWorld angekommen");
  response.setHeader('Content-Type', 'text/plain');
  response.end('Goodbye World');
}

app.use(logger);
app.use('/hello', helloWorld);
app.use('/goodbye', goodbyeWorld);
app.listen(8080);
```

- Vorherige Beispiele: Middleware-Funktionen werden bei jedem HTTP-Aufruf ausgeführt
- **_Mounting_**: Middleware-Funktion wird in ein bestimmtes Verzeichnis des Webservers "eingehängt"
- Ausführung der "gemounteten" Middleware-Funktion nur dann, wenn die aufgerufene Ressource unterhalb des Verzeichnisses liegt, in dem die Middleware-Funktion eingehängt wurde
- Bedingung zur Ausführung der Middleware-Funktion: Mount-Verzeichnis muss ein Präfix des Pfadnamens der aufgerufenen URL sein
- Beispiel: Zunächst Ausführung der Funktion `**logger**` bei jeder Anfrage, dann Ausführung von `**helloWorld**` und `**goodbyeWorld**` wenn Ziel der Anfrage im Verzeichnis `**hello**` bzw. `**goodbye**` liegt bzw. deren Unterverzeichnissen
- Beispiel: Ausführung der Middleware-Funktion `**goodbyeWorld**` bei Aufruf von `**xyz.de/goodbye**`, `**xyz.de/goodbye/alex**`, `**xyz.de/goodbye/berta**`, `**xyz.de/goodbye/visitors/charly**`, usw

# 4 Unterschiedlich Anfrage/Kontrollflüsse

write() 不会发信息给 client ,  end() 才会把信息给client 

Identische Anfrage-/Kontrollflüsse: Erste Middleware versendet Response
![](images/Pasted%20image%2020250202205759.png)\

Unterschiedliche Anfrage-/Kontrollflüsse: Provider versendet Response
![](images/Pasted%20image%2020250202205822.png)



