
# 1 Sitzungen und Sitzungsmanagement

- **_Sitzung_** (**_Session_**): Verbindung zwischen Client und Server, die mehrere Anfragen umfasst
- Sitzungen werden durch **_Session Identifiers_** (**_SessionIds_**) repräsentiert
- Problem: HTTP ist zustandslos, d.h. Zustand wird nach Bearbeitung einer HTTP-Anfrage gelöscht
- **_Sitzungsmanagement_**: Verwaltung, Speicherung und Wiederherstellen des Zustandes einer Sitzung im Backend
- Drei Optionen für das Sitzungsmanagement
    - Client-basiertes Sitzungsmanagement
    - In-Memory-Sitzungsmanagement
    - Datenbank-basiertes Sitzungsmanagement


# 2 Arten des Sitzungsmanagements

![](images/Pasted%20image%2020250206211402.png)

Client-basiertes Sitzungsmanagement
- Sitzungszustand wird in einem oder mehreren Cookies speichert - keine SessionId
- Übertragung des Cookies bei jeder Anfrage an den Server
- Aktualisierung des Cookies mit neuem Sitzungszustand bei jeder Antwort vom Server an den Client

In-Memory Sitzungsmanagement
- Sitzungszustand wird im Hauptspeicher des Servers gespeichert
- Cookies transportieren die SessionId
- Schnell, aber schlecht skalierbar da kein Load Balancing

Datenbank-basiertes Sitzungsmanagement
- Sitzungszustand wird in einem externen Session Store gespeichert
- Cookies transportieren die SessionId
- Beispiel: MongoDB

# 3 Basic Authentication 

![](images/Pasted%20image%2020250206211743.png)

- Zugang zu geschützten Bereichen eines Webservers mittels Nutzername und Passwort zur Authentifizierung des Nutzers
- Verschiedene Möglichkeiten der Authentifizierung: Basic Authentication, Digest Access Authentication,...


Basic Authentication
- Webserver zeigt dem Client die Notwendigkeit einer Authentifzierung mittels Status Code 401 sowie der Authentifizierungsart und den geschützten Bereich an
- Nutzername/Passwort (Credentials) und Art der Authentifzierung gelten innerhalb des Realm
- Credentials werden vom Client an den Server Base64-codiert übertragen
- Base64: Codierung von Daten durch die Zeichen A-Z, a-z, 0-9, + und / sowie = am Ende
- Achtung: Base64 ist eine Codierung, keine Verschlüsselung
- Basic Authentication nur mit HTTPS verwenden


# 4 Session IDs mit Cookies

![](images/Pasted%20image%2020250206212034.png)


IN-Memory Sitzungsmanagement mit Express-Session
- Express-Session-Middleware ermöglicht Sicherung des Sitzungszustands im Hauptspeicher des Servers
- Middleware erzeugt ein Objekt session innerhalb des `**req**`-Objektes
- `**session**`-Objekt kann genutzt werden, um Parameter des Sitzungszustandes (z.B. Bestückung eines Einkaufswagens im eCommerce) zu sichern
- Middleware erkennt wiederkehrenden Nutzer und fügt das nutzerspezifische `**session**`-Objekt dem `**req**`-Objekt hinzu
- Middleware verlangt einen Schlüssel zum Signieren der im Cookie transportierten `**SessionId**` (strittig und fragwürdig!!!)

# 5 Das REQUEST-Objekt und seine Bestandteile

![](images/Pasted%20image%2020250206212150.png)

# 6 Der Aufbau des Request-Objekts beim Middleware-Durchlauf

![](images/Pasted%20image%2020250206213429.png)


```js
let app=express();
let counter=0;


let logger=function(req, res, next) { 
  counter++;
  req.counter=counter;
  console.log(req.counter, '\tLogger:', req.method, req.url);
  res.on('finish', () => {
    console.log(req.counter, '\tLogger:', res.statusCode);
  });
  next();
}
app.use(logger);

app.use(express.static(path.join(__dirname, '../../yasn-frontend/dist')));


app.use(bodyParser.urlencoded({extended: true}));
app.use(bodyParser.json());

app.use(session({
  secret: 'webtechnologien',
  resave: 'true',
  saveUninitialized: 'true',
  cookie: {maxAge: 60*60*24*365*1000}
}));

passport.use(new localStrategy(
  function(username, password, done) {
    console.log('\tInside Passport: '+username+" "+password);
    if (username==='jamesbond' && password==='abc') {
      console.log('\tInside Passport: >>>Login success');
      return done(null, {id: '123', username: 'jamesbond'});
    }
    else {
      console.log('\tInside Passport: >>>Login failure');
      return done('401', false);
    }
  }
));

passport.serializeUser(function(user, done) {
  console.log('\tInside callback of serializeUser: '+user.id);
  done(null, user.id);
});

passport.deserializeUser(function(id, done) {
  console.log('\tInside callback of deserializeUser: '+id);
  done(null, {id: '123', username: 'jamesbond', firstname: 'James', lastname: 'Bond'});
});
    
app.use(passport.initialize());
    
app.use(passport.session());

app.post('/signin', passport.authenticate('local'), function(req,res,next) {
  console.log(req.counter, '\tPOST /signin: req.session.passport.user: '+JSON.stringify(req.session.passport.user));
  console.log(req.counter, '\tPOST /signin: req.user: '+JSON.stringify(req.user));
  if (req.isAuthenticated()) {
    console.log(req.counter, '\tPOST /signin sucessfull');
    return res.status(200).send('OK');
  } else {
    console.log(req.counter, '\tPOST /signin failed');
    return res.status(401).send('Unauthorized');
  } 
});

app.use(function(req, res, next) {
  if (req.isAuthenticated()) {
    console.log(req.counter, '\tPassword Gate passed');
    return next();
  }
  console.log(req.counter, '\tPasword Gate not passed');
  return res.status(401).send('Unauthorized');
});

app.use(express.static(path.join(__dirname, '../../yasn-frontend')));

app.get('/signout', function(req, res, next) {
  req.logout(function(err) {
    if (err) { 
      console.log(req.counter, '\tGET /signout failed')
      return next(err); 
    }
    console.log(req.counter, '\tGET /signout successfull')
    return res.status(200).send('OK');
  });
});

require('../app/routes/timeline.server.routes.js')(app);

return app;
};
```


```js
app.post('/signin', passport.authenticate('local'), function(req,res,next) {
  console.log(req.counter, '\tPOST /signin: req.session.passport.user: '+JSON.stringify(req.session.passport.user));
  console.log(req.counter, '\tPOST /signin: req.user: '+JSON.stringify(req.user));
  if (req.isAuthenticated()) {
    console.log(req.counter, '\tPOST /signin sucessfull');
    return res.status(200).send('OK');
  } else {
    console.log(req.counter, '\tPOST /signin failed');
    return res.status(401).send('Unauthorized');
  } 
});

app.use(function(req, res, next) {
  if (req.isAuthenticated()) {
    console.log(req.counter, '\tPassword Gate passed');
    return next();
  }
  console.log(req.counter, '\tPasword Gate not passed');
  return res.status(401).send('Unauthorized');
});

app.use(express.static(path.join(__dirname, '../../yasn-frontend')));

app.get('/signout', function(req, res, next) {
  req.logout(function(err) {
    if (err) { 
      console.log(req.counter, '\tGET /signout failed')
      return next(err); 
    }
    console.log(req.counter, '\tGET /signout successfull')
    return res.status(200).send('OK');
  });
});

require('../app/routes/timeline.server.routes.js')(app);

return app;
};
```


## 6.1 Step 1: Instanziierung eines leeren Request-Objekts

![](images/Pasted%20image%2020250206214013.png)

## 6.2 Step 2: Logger

- Proprietäre Middleware-Funktion (nicht Bestandteil von express)
- Middleware zum Protokollieren von Requests und Responses
- Einhängen einer `**counter**`-Variable in `**req**` für die Zuordnung von Kontrollausgaben nachgelagerter Middleware-Funktionen
- Protokollierung von Responses bei deren Versendung in nachgelagerten Middleware-Funktionen durch Callback (weil Response nicht durch die Middleware-Chain zurückgeleitet wird)

```js
let app=express();
let counter=0;


let logger=function(req, res, next) { 
  counter++;
  req.counter=counter;
  console.log(req.counter, '\tLogger:', req.method, req.url);
  res.on('finish', () => {
    console.log(req.counter, '\tLogger:', res.statusCode);
  });
  next();
}
app.use(logger);
```


## 6.3 Step 3: Auslieferung öffentlicher Dateien

![](images/Pasted%20image%2020250206214300.png)


```js
app.use(express.static(path.join(__dirname, '../../yasn-frontend/dist')));
```

- `**express.static**`: Middleware zur Auslieferung statischer Dateien aus dem öffentlichen Verzeichnis `**/yasn-frontend/dist**` des Webservers
- Nach dem Holen der angefragten Datei aus dem Verzeichnis wird der Dateiinhalt in einer Response verpackt und verschickt, d.h. die Middleware-Chain endet hier

## 6.4 Step 4: Auslesen von Query- und Body-Daten

![](images/Pasted%20image%2020250206214420.png)

```js
app.use(bodyParser.urlencoded({extended: true}));
app.use(bodyParser.json());
```


- `**bodyParser.urlencoded**` holen Query-Parameter aus der Anfrage, die vom Browser mit `**application/x-www-form-urlencoded**` codiert wurden
- `**bodyParser.json**` holt Daten aus dem Body der Anfrage, die mit `**application/json**` codiert wurden
- Generierung eines `**body**`-Objektes und Einhängen in `**req**`
- Anschliessendes Einfügen der Daten aus der Anfrage in `**body**`
- Daten sind anschließend mit `**req.body.property_name**` gelesen werden


## 6.5 Step 5: Session-Objekt für Daten der Sitzung

![](images/Pasted%20image%2020250206214619.png)

- Prüft, ob Anfrage ein Session-Cookie enthält
- Wenn ja, dann wird `**req**` das im Speicher befindliche `**session**`-Objekt der letzten Anfrage angehängt und das Cookie in der Antwort ggfs. erneuert
- Wenn nicht, dann wird `**req**` ein neues, leeres Session-Objekt angehängt und in der Antwort ein neues Cookie gesetzt


```js
app.use(session({
  secret: 'webtechnologien',
  resave: 'true',
  saveUninitialized: 'true',
  cookie: {maxAge: 60*60*24*365*1000}
}));
```


## 6.6 Step 6: Festlegen einer Passport-Strategie

![](images/Pasted%20image%2020250206214700.png)

- Festlegen einer Passwort-Strategie und Konfiguration der Middleware mit dieser Strategie, die für alle Nutzer während des Lebenszyklus des Servers angewendet wird
- Beispiel zeigt eine einfache lokale Strategie basierend auf Nutzername und Passwort
- Später: Anbindung an Datenbank zur Überprüfung der übermittelten Credentials

```js
passport.use(new localStrategy(
  function(username, password, done) {
    console.log('\tInside Passport: '+username+" "+password);
    if (username==='jamesbond' && password==='abc') {
      console.log('\tInside Passport: >>>Login success');
      return done(null, {id: '123', username: 'jamesbond'});
    }
    else {
      console.log('\tInside Passport: >>>Login failure');
      return done('401', false);
    }
  }
));
```

## 6.7 Step 6: Initialisierung von Passport

![](images/Pasted%20image%2020250206214910.png)

- `**serializeUser**` 
    - Konfiguriert `**passport**` mit einer Funktion, die festlegt, welche Daten des Nutzers im `**session**`-Objekt gespeichert werden (üblicherweise nur eine Nutzer-ID)
    - Übergebene Funktion wird nur nach dem `**login**` durchlaufen
    - Speichert einen Teilzustand der Sitzung im Hauptspeicher des Servers
- `**deserializeUser**` 
    - Konfiguriert `**passport**` mit einer Funktion, die festlegt, welche weiteren Daten der Sitzung aus der Datenbank gelesen und dem `**req**` hinzugefügt werden
    - Übergebene Funktion wird bei jeder Anfrage durchlaufen
    - Speichert einen Teilzustand der Sitzung in der Datenbank des Servers
- **passport.initialize** wird bei jeder Anfrage durchlaufen und fügt `**req**` die `**login**`-, `**logout**`- und `**isAuthenticated**`-Methoden hinzu
- passport.session fügt anschließend dem `**session**`-Objekt `**passport.user**` (mit den Daten aus der mit `**serializeUser**` übergebenen Funktion) und dem `**req**`-Objekt `**user**` (mit den Daten aus der mit `**deserializeUser**` übergebenen Funktion) hinzu


```js
passport.serializeUser(function(user, done) {
  console.log('\tInside callback of serializeUser: '+user.id);
  done(null, user.id);
});

passport.deserializeUser(function(id, done) {
  console.log('\tInside callback of deserializeUser: '+id);
  done(null, {id: '123', username: 'jamesbond', firstname: 'James', lastname: 'Bond'});
});
    
app.use(passport.initialize());
    
app.use(passport.session());
```


## 6.8 Step 7: Controller für den Login

![](images/Pasted%20image%2020250206215219.png)


```js
app.post('/signin', passport.authenticate('local'), function(req,res,next) {
  console.log(req.counter, '\tPOST /signin: req.session.passport.user: '+JSON.stringify(req.session.passport.user));

  console.log(req.counter, '\tPOST /signin: req.user: '+JSON.stringify(req.user));

  if (req.isAuthenticated()) {
    console.log(req.counter, '\tPOST /signin sucessfull');
    return res.status(200).send('OK');
  } else {
    console.log(req.counter, '\tPOST /signin failed');
    return res.status(401).send('Unauthorized');
  } 
});
```

- Nutzername und Passwort werden im Body (codiert mit `**application/json**`) von `**POST /signin**` an der Server übertragen
- Controller ruft Passwortstrategie auf, mit der beim Start des Servers `**passport**` konfiguriert wurde 
- Passwortstrategie verifiziert Nutzername und Passwort
- Die an `**serializeUser**` übergebene Funktion hängt `**passport.user**` in das `**session**`-Objekt ein
- Die an `**deserializeUser**` übergebene Funktion beschafft weitere Sitzungsdaten aus Datenbank und hängt sie in `**req**` als `**user**`-Objekt ein


## 6.9 Step 8: Passwortschranke

![](images/Pasted%20image%2020250206215553.png)

- Passwortschranke überprüft mit `**isAuthenticated**`-Methode, welche durch `**passport.initialize**` in `**req**` eingehängt wurde, ob die Anfrage zu einem authentifizierten Nutzer gehört
- Wenn ja, wird Anfrage an nachgelagerte Funktionen mit `**next()**` weitergeleitet
- Wenn nein, wird Middleware-Chain abgebrochen und eine `**401**`-Antwort zurückgeschickt

```js
app.use(function(req, res, next) {
  if (req.isAuthenticated()) {
    console.log(req.counter, '\tPassword Gate passed');
    return next();
  }
  console.log(req.counter, '\tPasword Gate not passed');
  return res.status(401).send('Unauthorized');
});
```


## 6.10 Step 9: Auslieferung nicht-öffentlicher Dateien

```js
app.use(express.static(path.join(__dirname, '../../yasn-frontend/non-public')));
```

![](images/Pasted%20image%2020250206220156.png)


