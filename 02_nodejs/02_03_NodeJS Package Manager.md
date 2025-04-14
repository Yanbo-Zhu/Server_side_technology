
# 1 **NodeJS Package Manager**  

Eine weitere große Stärke von Node.js ist die Vielzahl an fertigen Funktionalitäten. Diese werden in Modulen bereitgestellt und können sehr einfach in ein eigenes Projekt integriert werden. Eine normale Installation von Node.js enthält dazu ein Kommandozeilen-Tool namens npm (Node.js Package Manager). 

Auf [https://www.npmjs.com/](https://www.npmjs.com/) können alle bereitgestellten Module gesucht werden. Neben Funktionalitäten für einen Node.js-Server, werden nahezu alle JavaScript-Frameworks und -Bibliotheken angeboten, sodass sich npm als Standard für professionelle Entwicklung (auch im Frontend) mit Java-Script durchgesetzt hat. 

Um mit npm zu arbeiten, wird im eigenen Projekt eine Datei "package.json" mit Informationen zum Projekt benötigt. Diese Datei kann sehr einfach mit dem Befehl "npm init" erstellt werden. Alle benötigten Informationen werden interaktiv in der Kommandozeile abgefragt.

Beispiel package.json: 
```
{
    "name": "MeinProjekt",
    "version": "1.0.0",
    "description": "Kurze Beschreibung des Projekts",
    "main": "index.js",
    "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
    "author": "Thomas Onken",
    "license": "ISC",
    "dependencies": {
    "express": "^4.16.2"
  }
}
```


Für ein Projekt benötigte Module können innerhalb von "_dependencies_" eingetragen werden. Dadurch können sie, gesammelt mit dem Befehl "_npm install_", im Projektverzeichnis heruntergeladen werden. Modulabhängigkeiten werden aufgelöst und automatisch mit heruntergeladen, z.B. benötigt ein jQuery-Plugin-Modul wie jquery-validation das jquery-Modul und so weiter. Später benötigte Module können mit "_npm install_ <_Modul-Name_> _--save_" hinzugefügt werden. Alle Module werden im Unterverzeichnis _node_modules_ abgelegt.

In einer Node.js Umgebung können die heruntergeladenen Module aus dem node_modules-Verzeichnis mittels der require-Funktion im JavaScript importiert und als Objekt bereitgestellt werden. Beispiel (express-Module):

`var expressModule = require("express");`

Aufbauend auf npm gibt es eine Vielzahl von weiteren Build-Tools, die bei der **Entwicklung von JavaScript-basierter Software** und der **Entwicklung von Web-Frontends** unterstützen.

Hier eine kleine Auswahl von sehr verbreiteten Tools, denen Sie bei der professionellen Entwicklung mit JavaScript und Web-Frontends begegnen werden:

|Tool|Erläuterung|
|---|---|
|[gulp](https://gulpjs.com/)|Build- und Automatisierung|
|[grunt](https://gruntjs.com/)|Build- und Automatisierung|
|[Bower](https://bower.io/)|Frontend Package-Manager|
|[yeoman](http://yeoman.io/)|Scaffolding Tool (Projekt- / Code-Generator)|
|[webpack](https://webpack.js.org/)|Static Asset Bundler|



# 2 npm command 

![](images/Pasted%20image%2020250312124553.png)




