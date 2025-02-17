
# 1 Teile und Herrsche

![](images/Pasted%20image%2020250206204419.png)

- Problem: Komplexität moderner Webanwendungen
- Unterschiedliche Aufgaben
    - Anwendungslogik und Verarbeitung von Daten
    - Persistente Datenspeicherung
    - Präsentation und Darstellung
    - Kommunikation


MVC Pattern
- **_MVC_**: **_Model-View-Controller_**
- Funktionale Aufteilung einer Webanwendung in getrennte Module
- **_View_**: Nutzerinterface, d.h. Darstellung einer Webanwendung und Interaktion mit dem Nutzer
- **_Model_**: Datenstrukturen einer Webanwendung und assoziierte Aufgaben, d.h. Lesen und Schreiben von Daten sowie Persistenz
- **_Controller_**: Bearbeitung von Anfragen, Sitzungsmanagement sowie Verknüpfung von Model und View



# 2 SPA – Verschiebung des View in den Client


![](images/Pasted%20image%2020250206204531.png)



# 3 Verzeichnisse

![](images/Pasted%20image%2020250206204613.png)

- `**app**`: Express-Anwendungslogik
    - `**controllers**`: Controllers der Webanwendung
    - `**models**`: Models der Webanwendung
    - `**routes**`: Routing-Middleware der Webanwendung
    - `**views**`: Templates der Webanwendung (nicht erforderlich bei SPA)
- `**config**`: Konfigurationsdateien der Webanwendung für unterschiedliche Umgebungen (Development, Testing, Production)
- `**public**`: Aufbewahrung der statischen Dateien zur Auslieferung an den Webbrowser vor der Passwortschranke
    - `**css**`: Style-Sheet-Dateien
    - `**img**`: Bilddateien
    - `**js**`: Client-seitige JavaScripts
- non-public. Aufbewahrung statischer Dateien hinter der Passwortschranke


# 4 Architektur des Backend

![](images/Pasted%20image%2020250206205636.png)





