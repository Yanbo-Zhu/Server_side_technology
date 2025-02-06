
# 1 Eine Dokumentenorientierte Datenbank

![](images/Pasted%20image%2020250206220244.png)

Merkmale
- Dokumentenorientierte Datenbank, d.h. Daten werden in Dokumenten gespeichert und in Kollektionen verwaltet
- Dokumente entsprechen Zeilen einer Tabelle in einer relationalen Datenbank
- Kollektionen entsprechen Tabellen in einer relationalen Datenbank
- MongoDB ist schemafrei - Dokumente einer Kollektion müssen nicht einem bestimmten, vorgegebenen Schema folgen

BSON
- _**Binary JSON**_: binäre codierte Serialisierung von JSON
- Zugrunde liegendes Dokumentenformat
- Unterstützung der JSON-Datenformate, plus zusätzlicher Formate (z.B. `**date**` oder `**byte**` array)

# 2 Lokale MongoDB versus Cloud MongoDB


![](images/Pasted%20image%2020250206221208.png)




# 3 Einführende Beispiele


Beispiel für Schreiboperationen
![](images/Pasted%20image%2020250206221421.png)


Beispiel für Leseoperationen
![](images/Pasted%20image%2020250206221436.png)

![](images/Pasted%20image%2020250206221443.png)


# 4 CRUD – Einfügen (CReate) neuer Dokumente (I/II)

```js
db.posts.insert({
  "title": "Second Post",
  "user": "Alice"
})


db.posts.update({
  "user": "Alice"
}, {
  "title": "Second Post", 
  "user": "Alice"});
}, {
  upsert: true
})


db.posts.save({
  "title": "Second Post",
  "user": "Alice"
})


db.posts.save({
  "_id": Object("45c230a123d45efab2340"),
  "title": "Second Post",
  "user": "Alice"
})

```

- `**insert(document)**`: häufigste Methode zum Einfügen neuer Dokumente in eine Kollektion
- `**document**` als einziges Argument der Methode
- `**_id**` wird automatisch generiert und dem Dokument zugewiesen
- Verursacht ggfs. das Anlegen einer neuen Kollektion, wenn die angegebene nicht existiert
- `**update(selectionCriteria, updateStatement, options)**`: alternative Methode zum Einfügen von Dokumenten
- `**upsert**` ist `**true**` wenn ein neues Dokument angelegt werden soll wenn kein Dokument mit den Suchkriterien in der Kollektion vorhanden ist
- `**save(document)**`: weitere Alternative, die optional die Definition einer eigenen `**_id**` erlaubt


# 5 CRUD – Lesen (READ) von Dokumenten 

- `**find(selectionCriteria)**`: Methode zum Finden und Lesen von Dokumenten einer Kollektion
- `**selectionCriteria**` enthält einfache oder zusammengesetzte Anfrageselektoren
- Beachte: alle MongoDB-Operatoren werden mit einem `**$**` eingeleitet

```js

db.posts.find() //alle Dokumente einer Kollektion

db.posts.find({}) //alle Dokumente einer Kollektion

db.posts.find({ "user": "Alice"})

db.posts.find(
  {"user": {$in: ["Alice", "Bob"]}}
) //alle Dokumente mit user=Alice oder Bob

db.posts.find({
  $or: [{"user": "Alice"}, {"user": "Bob"}]}
) // //alle Dokumente mit user=Alice oder Bob

db.posts.find({
  "user": "Alice",
  "commentsCount": {$gt: 10}
}) //alle Dokumente mit user=Alice und commentsCount>10

```


# 6 CRUD – Aktualisieren (Update) und Löschen (DELETE)

```js
db.posts.update({
  "user": "Alice"
}, {
  $set: {
    "title": "Second Post", 
}}, {
  multi: true
})

db.posts.remove()
//Entfernt alle Dokumente der Kollektion


db.posts.remove({"user": "Alice"})
//Entfernt alle Dokumente mit user=Alice

db.posts.remove({"user": "Alice"}, true)
//Entfernt ein Dokument mit user=Alice
```

- `**update(selectionCriteria, updateStatement, options)**`: Methode zum Aktualisieren von Dokumenten einer Kollektion (oder zum Einfügen eines neuen Dokuments falls `**selectionCriteria**` durch kein existierendes Dokument erfüllt werden)
- `**selectionCriteria**`: enthält einfache oder zusammengesetzte Anfrageselektoren
- `**updateStatement**`: spezifiziert die zu aktualisierenden Felder
- `**options**`: verschiedene Optionen, zum Beispiel ob ein oder mehrere Dokumente aktualisiert werden sollen

- `**remove(selectionCriteria, justOne)**`: Methode zum Löschen von Kollektionen oder einer oder mehrerer Dokumente
- `**selectionCriteria**`: s.o.
- `**justOne**`: zeigt an, dass bei mehreren Dokumenten, die `**selectionCriteria**` erfüllen, nur eines gelöscht wird

# 7 Replikation 

![](images/Pasted%20image%2020250206222810.png)

- **_Replikation_**: Vorhalten mehrfacher Kopien von Daten auf verschiedenen Servern
- Replikation ermöglicht Datenredundanz und erhöht Datenverfügbarkeit
- Voraussetzung für Fehlertoleranz: Schutz vor Datenverlust und Nichtverfügbarkeit bei Ausfall eines Datenbank-Servers
- Erhöhung der Lesekapazität: Leseoperation können von verschiedenen Datenbank-Servern ausgeführt werden
- Verminderung von Latenzen durch _**Lokalitätsprinzip**_: Speicherung von Kopien der Daten in geographisch verteilten Rechenzentren in Kombination mit Geo-Routing der Leseanfragen


![](images/Pasted%20image%2020250206222946.png)

- **_Replica Set_**: Gruppe von MongoDB-Instanzen mit den gleichen Datensätzen
- _**Primary Node**_: MongoDB-Instanz eines Replica Set, welche Schreiboperationen entgegennimmt
- _**Secondary Nodes**_: ein oder mehre MongoDB-Instanzen eines Replica Set, welche Datensätze regelmäßig vom Primary Node replizieren und optional Leseanfragen entgegennehmen
- Bei Ausfall des Primary Node wird aus der Gruppe der verbleibenden Secondary Nodes ein neuer Primary gewählt


# 8 Skalierbarkeit 

![](images/Pasted%20image%2020250206223040.png)


Vertikale Skalierbarkeit
- _**Skalierbarkeit**_: Fähigkeit eines Systems seine Leistung durch Hinzufügen von Ressourcen zu steigern
- Steigerung der Leistung eines Servers durch Hinzufügen oder den Austausch zusätzlicher, leistungsfähigerer Hardware-Ressourcen (CPU, Hauptspeicher, Bandbreite,...)
- Unabhängig von der Softwarearchitektur und Implementierung
- Überproportionale Hardware-Kosten ab einer bestimmten Ausbaustufe
- Begrenzt durch die Leistungsfähigkeit verfügbarer Hardware


Horizontale Skalierbarkeit
- Steigerung der Leistung eines Systems durch Hinzufügen zusätzlicher Server
- Erfordert verteilte, auf Skalierbarkeit ausgelegte Softwarearchitektur
- Theoretisch unbegrenzte Skalierbarkeit

----


![](images/Pasted%20image%2020250206223217.png)

- _**Sharding**_: Partitionierung der Datenmenge und Verteilung auf mehrere Server zur Umsetzung horizontaler Skalierbarkeit
- _**Range-based Partitioning**_ versus _**Hash-based Partitioning**_
- Sharding reduziert die Datenbankoperationen, die ein Server ausführen muss, und reduziert das Datenvolumen pro Server

Shards
- Speichern jeweils eine Teilmenge der gesamten Daten
- Realisierung als einzelne Instanz oder als Replica Set

Routers
- Bestandteil des MongoDB-Treibers bei der Anwendung
- Leiten Anfragen der Anwendung an einen oder mehrere zuständige Shards basierend auf Routing-Informationen
- Horizontale Skalierbarkeit durch mehrere Router

Shared Key
- Indiziertes Feld in jedem Dokument der Datenbank
- Grundlage für die Partitionierung der Datenmenge in Chunks
- Gleichmäßige Verteilung der Chunks auf verfügbare Shards

**Config Server**
- Verwaltung der Routing-Informationen basierend auf Zuweisung der Chunks zu Shards


# 9 Indexierung 

![](images/Pasted%20image%2020250206223425.png)

```js
{
  "_id": ObjectId("52d02240eb01d67d71ad577"),
  "title": "First Blog Post",
  "comments": [
    ...., ....., .....
  ],
  "commentsCount": 12
}

db.posts.find({"commentsCount": {$gt: 10}})
```

- Datenbankindex: von der Datenstruktur getrennte Indexstruktur in einer Datenbank, welche die Suche und das Sortieren nach bestimmten Datenbankfeldern beschleunigt
- Anstelle alle Felder eines Dokumentes bei einer Anfrage zu überprüfen, liefert ein Index eine schnelle Referenz auf alle Dokumente, welche die Bedingungen der Anfrage erfüllen

- MongoDB-Datenbank zur Verwaltung von Einträgen eines Blogs: pro Dokument werden ein Blogeintrag und alle dazu eingegangenen Kommentare gespeichert
- Dediziertes Feld für den schnellen Zugriff auf die Anzahl der eingegangenen Kommentare
- Index auf dem Feld `**commentsCount**` macht Dokumente schnelle auffindbar
