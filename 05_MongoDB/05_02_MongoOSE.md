
![](images/Pasted%20image%2020250206223704.png)


# 1 Treiber für die Anbindung einer MongoDB

Node.js Modul für die Anbindung einer MongoDB an eine Express-Webanwendung

server.js
```js
const express=require('./config/express');
const mongoose=require('./config/mongoose.js');

(async () => {
  try {
    let dbConnection = await mongoose();
    console.log("Zurück von mongoose");
    const app=express();
    app.listen(3000, '127.0.0.1');
    module.exports=app;
    console.log(`Hello Node.js v${process.versions.node}!`);
  } catch (err) {
    console.error('Fehler beim Herstellen der Datenbankverbindung:', err);
  }
})();
```


**/config/mongoose.js**
```js
const mongoose=require('mongoose');
const dbUrl='mongodb+srv://username:passwort@cluster0.87gnnnv.mongodb.net/yasn?retryWrites=true&w=majority';
const clientOptions={serverApi: {version: '1', deprecationErrors:true}};

module.exports = async () => {
  try {
    await mongoose.connect(dbUrl, clientOptions);
    console.log('MongoDB connected...');
    return mongoose.connection;
  } catch (err) {
    console.error('MongoDB connection error:', err);
    if (mongoose.connection) {
      mongoose.connection.disconnect();
    }
    throw err; // Weiterleitung des Fehlers an den aufrufenden Code}
  }
}
```



# 2 Definition von Datenmodellen gemäß MVC-Pattern (I/II)

/app/models/user.server.model.js
```js
const mongoose=require('mongoose'), Schema=mongoose.Schema;

let userSchema=new Schema({
    title: String,
    firstname: String,
    lastname: String,
    email: String,
    sex: String,
    dateOfBrith: Date,
    username: String,
    password: String,
    thumbnail: String,
    followed: [{
        firstname: String,
        lastname: String,
        username: String,
        thumbnail: String
    }],
    followedBy: [{
        firstname: String,
        lastname: String,
        username: String,
        thumbnail: String
    }]
});

module.exports=mongoose.model('User', userSchema);
```


/app/models/message.server.model.js
```js
const mongoose=require('mongoose'), Schema=mongoose.Schema;

const commentSchema=new Schema({
    username: String,
    author: String,
    thumbnail: String,
    time: {
        type: Date,
        default: Date.now
    },
    headline: String,
    content: String
});

let messageSchema=new Schema({
    username: String,
    author: String,
    thumbnail: String,
    time: {
        type: Date,
        default: Date.now
        },
    headline: String,
    content: String,
    comments: [commentSchema]
});

module.exports=mongoose.model('Message', messageSchema);
```


- Definition von Schemata als Grundlage für MVC-Modelle
- MongoDB ist eigentlich schemafrei, erlaubt jedoch die Definition von Schemata als Grundlage für Modelle im MVC-Pattern
- Modelle werden beim Start von MongoDB eingebunden und stehen dann der gesamten Webanwendung zur Verfügung


# 3 Anlegen eines neuen Nutzers in der Datenbank

**/config/express.js**
```js
...
app.post('/signup', async function(req, res, next) {
  let salt=bcrypt.genSaltSync(10);
  let user = new User(req.body);
  user.password = bcrypt.hashSync(req.body.password, salt);
  user.thumbnail = user.username + '.jpg';  
  try {
    await user.save();
    return res.status(200).send('OK');
  } catch (err) {
    return res.status(500).json({error: err.message});
  }
});
...
```


**Datenstruktur im Body der HTTP-Anfrage**
```js
{
  "firstName": "Max",
  "lastName": "Mustermann",
  "email": "max@mustermann.de",
  "username": "maxi",
  "password": "ghb344&!er"
}
```

- Controller holt sich zunächst eine Referenz auf das Modell
- Definition einer Callback-Funktion, die bei eingehenden POST-Anfragen auf `**/signup**` aufgerufen wird
- Controller erzeugt eine neue Instanz des Modells `**User**` (dessen Grundalge das Schema **`userSchema`** bildet)
- Bei der Instanziierung werden die zu schreibenden Daten als JSON-Objekt übergeben
- Hier: JSON-Objekt ist im Body der HTTP-Anfrage enthalten
- Struktur der Daten im Body muss dem Schema entsprechen
- Asynchrones Speichern der Daten in der Datenbank durch Aufruf von `**user.save()**`
- Im Fehlerfall: Fehlermeldung wird der nächsten Middleware-Funktion übergeben
- Im Erfolgsfall: gelesene Daten werden zur Bestätigung an die Antwort angehängt

# 4 Hashen und Salzen von Passwörtern mit bcrypt (I/III)

![](images/Pasted%20image%2020250206224152.png)
- Passwörter dürfen nicht im Klartext in der Datenbank gespeichert werden
- Statt Klartext werden Passwörter gehasht (nicht umkehrbar)
- Problem 1: Verwendung desselben Passworts durch mehrere Nutzer führt zu denselben Hashwerten
- Problem 2: Sammlungen von Hashwerten (Brute Force, Dictionaries, Regenbogentabellen) werden von Angreifern für die Herleitung der Klartext-Passwörter verwendet
- Lösung: Konkatenation des Passwortes mit einem individuellen Zufallswert (Salt) vor dem Hashen
- Jeder Nutzer erhält eigenen Salt

- bcrypt ist eine Express-Bibiliothek für das Hashen und Salzen von Passwörten
- Generierung eines Salts mit   `**bcrypt.genSaltSync(10);**`
- Parameter gibt Schwierigkeitsgrad des Salts an
- Berechnung des gesalzenen Hashs mit `**bcrypt.hashSync(req.body.password, salt);**`
- Überprüfung eines präsentierten Klartext-Passworts mit `**bcrypt.compareSync(password, user.password))**`


# 5 Passwort-Strategie mit Anbindung über Mongoose 

![](images/Pasted%20image%2020250206224329.png)

**/config/express.js**
```js
let User=require('../app/models/user.server.model');
...
passport.use(new localStrategy(
  async (username, password, done) => {
    console.log('Passport: localStrategy');
    try {
      let user = await User.findOne({ username: username });
      if (!user) {
        return done(null, false, { message: 'Incorrect username.' });
      }
      if (!bcrypt.compareSync(password, user.password)) {
        return done(null, false, { message: 'Incorrect password.' });
      }
      return done(null, user);
    } catch (err) {
      return done(err);
    }
  }
));
...
```


# 6 Speichern von Nachrichten und Kommentaren in YASN

**/app/routes/messages.server.routes.js**
```js
let message=require('../controllers/messages.server.controller');

module.exports=function(app) {    
    app.post('/messages', message.writeMessage);
    app.post('/messages/:messageId', message.writeComment);
}
```

**/app/controllers/message.server.controller.js**
```js
let Message = require('mongoose').model('Message');

exports.writeMessage = async function(req, res, next) {
  let message = new Message(req.body);
  message.username = req.user.username;
  message.author = req.user.firstname + ' ' + req.user.lastname;
  message.thumbnail = req.user.username + '.jpg';
  try {
    await message.save();
    res.location('/message/' + message._id).json(null);
  } catch (error) {
    res.status(500).json({error: "Es ist ein Datenbankfehler aufgetreten"});
  }
};
...
```


```js
...
exports.writeComment=async function(req, res) {
  try {
    const updatedMessage=await Message.findByIdAndUpdate(
      req.params.messageId, 
      {
        $push: {
          comments: {
            $each: [{
              username: req.user.username, 
              author: req.user.firstname+' '+req.user.lastname,
              thumbnail: req.user.username+'.jpg',
              headline: req.body.headline,
              content: req.body.comment
            }],
            $position: 0 
          }
        }
      },
      {new: true}
    );
    if (!updatedMessage) {
      return res.status(422).json({error: "Referenzierte Nachricht existiert nicht"});
    }
    res.status(200).json({updatedMessage});
  } catch (error) {
    res.status(500).json({error: "Es ist ein Datenbankfehler aufgetreten"});
  }
};
```



# 7 Lesen der Timeline in YASN

**/app/routes/messages.server.routes.js**
```js
module.exports=function(app) {
    let timeline=require('../controllers/timeline.server.controller');
    app.get('/timeline', timeline.read);
}
```


**/app/controllers/timeline.server.controller.js**
```js
let Message=require('../models/message.server.model');

exports.read = async function(req, res) {

// Suchanfragenobjekt für Nachrichten aufbauen
let queryObject = {
  $or: [{ username: req.user.username }]
};

for (let i = 0; i < req.user.followed.length; i++) {
  queryObject.$or.push({ username: req.user.followed[i].username });
}
    
// Nachrichten aus Datenbank holen
try {
  const messages = await Message.find(queryObject).sort('-time').exec();
  res.status(200).json(messages);
} catch (err) {
  return res.status(204).json({error: "Keine Nachrichten in der Timeline"});
}
};
```