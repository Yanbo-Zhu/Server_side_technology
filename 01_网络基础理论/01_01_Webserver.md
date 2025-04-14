
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


https://blog.csdn.net/qq_33922306/article/details/120801706

## 1.1 WebServer 分类

按照我个人对web服务器的理解，我将其分为两类：

1、一类是以nginx、Apache Http Server为代表的通用的Web Server
2、一类是基于语言的web Server，比如C++的Piatache、Java的Tomcat、Python的gunicorn等

区分的依据主要是，是否可以执行计算任务！

## 1.2 WebServer 执行流程

详细的说，一个web服务器的工作是什么呢？其实就是：

1、监听用户的请求
2、收到请求后对HTTP请求进行解析
3、根据解析的内容，返回数据

## 1.3 WebServer 如何返回数据

前面我说的两种服务器的区别就在第三步，也就是返回的数据，一般是一个HTML的页面。如果是是一个静态页面，换句话说，就是返回一个文本文件，那么就很容易了，直接调用send()系统调用就可以了，这是Nginx等通用WebServer可以搞定的！但是如果需要动态生成HTML呢？比如需要先查询数据库，然后执行一些操作，最终再生成数据然后返回呢？这个时候Nginx就做不到了，或者说没办法直接做到了！因为我的处理方式是java或php写的，这个时候nginx显然是没办法直接拿到结果的！

怎么办呢？
1、使用tomcat，因为他本身就是Java写的，上述webserver的1、2步骤都是由Java实现，当需要第三步动态生成数据的时候他只需要启动一个线程来处理一下就行了
2、php的解决方案，nginx会将这个请求发给PHP-FPM，后者会启动php解析器调用php脚本，然后将结果通过标准输入输出的方式发给nginx，nginx再返回解雇，这个就是FastCGI协议！


# 2 什么是NIO

关于NIO和BIO我在下面的文章中，详细的讨论过，这是我写过的最好的总结，不知道为啥阅读量低的可怜！

Kingdo：对高并发的思考

我在这里再阐述一下NIO：

我们详细的展开上述的webserver的监听请求的过程：
其实就是创建一个套接字，然后监听，这个套接字我们将其命名为listen-fd，是的，它是一个文件描述符！
监听是需要一个线程的。此时一个用户的一个HTTP请求到达，这个时候我们需要和用户建立一个TCP链接，然后创建一个新的套接字与用户通讯，创建的这个套接字我们称之为handle-fd！注意，一个TCP请求，可以传输多个HTTP请求！也就是长连接！

那么BIO和NIO区别就在如何在处理这个handle-fd：
   前者直接启动一个新的线程，来接受并处理用户发来的请求，也就是每个handle-fd对应一个线程，但是由于HTTP通常是长连接的，那么在没有HTTP请求的时候，此线程就被阻塞了！而且我们需要为每个用户的TCP连接创建一个线程，这会使得服务器不敢重负！
   而NIO使用epoll，他会监控handle-fd，当handle-fd可读时(即一个HTTP请求到达)，触发一个事件，我们的线程只需要循环处理事件就可以了！这样的话一个线程可以同时处理多个TCP连接，当其中一个发来了HTTP请求，epoll就会产生一个事件，我们的线程会循环的检查有没有事件产生从而处理他！

这就是NIO的优势！


# 3 Nginx
## 3.1 Nginx和Tomcat是如何使用NIO的？

Nginx

对于Nginx，会启动多个worker进程，所有的进程都会监听listen-fd，当TCP连接到达时，理论上所有的worker都会收到请求，也就是群惊，但是通过一定的方法可以保证只有一个人监听到了TCP的到达！
每个worker进程维护了一个epoll，epoll监听了listen-fd，当他知道TCP到达后就会产生对应的事件，然后会唤醒正在等待事件的线程，为了好理解，我们认为一个worker进程中就一个线程！然后处理这个TCP请请求，也就是调用accept系统调用创建一个handle-fd，然后将其挂到epoll中，当handle-fd接收到HTTP请求，同样epoll会产生事件，线程再处理这个事件！

可以看到线程其实是串行的处理每一个HTTP请求！多个worker实现了并行！这样线程就会一直在工作不会因为http请求没有到达而被阻塞，同时有确保了线程数目不会过多！可以认为其比较充分的利用了CPU资源！

Tomcat
Tomcat我没研究过，我用与之类似的Pistache来说明，我的主页有对于Pistache代码的全面解析，有兴趣的可以去看看！
和Nginx的Master-Worker模式略有不同，但是本质上没区别，有个名字叫Reactor 多线程模型！
首先会启动一个名为Reactor 的线程和多个handler线程，这个Reactor线程的任务就是监听listen-fd，当有TCP链接时，选择其中一个handler线程，然后handler线程创建handle-fd，然后监听handle-fd，也是通过epoll机制，本质上没有任何区别。

## 3.2 为什么通常认为Nginx会更快呢？

终于到了问题了！从上面的分析看Niginx，在设计上，并不存在任何优势，而且Nginx仅仅能处理静态的资源！
我认为问题就恰恰出在了这里，那就是如何将静态资源返回给用户。

其实我是不知道Nginx怎么实现的，我可以猜，我觉得是这样的：

直接通过sendfile(）系统调用把静态文件的数据发从出去，这过程用C语言是很容易的。

而我们用Java，特别是使用高级的web开发框架，他的过程一般是这样的：
1、read()读文件到内存
2、修改其中的部分内容，对于静态文件则不需要改
3、调用send()发送出去

看上去好像差不多，因为都要先把数据读到内存再发送。

区别在于底层的实现，这需要一定的OS的知识，sendfile的过程，是先把读文件，OS只要你读文件，会首先把文件读pagecache中，也就是页面缓存，然后数据直接从页面缓存复制到TCP的发送缓冲中，整个过程不需要在用户空间执行任何代码！

然而你用read()的话，数据先到pagecache，然后被copy到用户空间的缓冲区，然后再赋值给TCP缓冲区，这一上一下就会浪费很多的时间！

当然你可能会疑问，这个开销和从磁盘加载文件比，岂不是小巫见大巫？的确是的，但是pagecache被加载一次之后，除非内存告急被回收，否则会一直存在，这样这个部分的时间开销两者就都没有了！

其实Nginx采用的其实就是所谓的”零拷贝“方式！甚至，可以绕过TCP缓冲区，直接从pagecache的中读数据！

至此，我必须作一个免责声明，以上都是我之前研究并发的时候对一系列的问题的调研，以及我对相关问题的理解，我没有翻阅过Nginx的源码，也没深入的了解过零拷贝，我只是从理论的角度分析，他的优势！但是对于Pistache、epoll以及pagecache我还是比较有把握的。


## 3.3 为什么Nginx最擅长反向代理

作为问题的结果，我想解释这个问题，其实这和NIO、Epoll是息息相关的！毕竟Nginx的最广泛应用就是反向代理。
所谓反向代理就是，请求发给Nginx，然后再把请求转发出去，等待结果的，然后再将结果返回！

Epoll即是实现NIO的核心，但是我们需要知道，Epoll最擅长的是监听socket和eventfd！也即是最擅长监听网络文件描述符，对于本地的文件，Epoll是没办法的，因此要想实现对文件的异步读写，还是得需要多线程，这也是为什么nodejs的libuv使用了两种方法实现异步，一种是Epoll，一种是多线程！
而实现反向代理，恰好就是访问网络IO，也就是Nginx创建一个client-fd，与要转发的服务器进行通讯，当目标服务器把数据写回的时候，被Nginx的epoll监听到，然后再处理这个事件！

是不是也是在NIO呢？
当然了和php通讯也是类似的，只不过并没有用网络套接字，而是用标准输入输出，这也是一个特殊的fd，只要php-fpm将准备好的数据写道标准输出，然后被Nginx感知到！也是异步IO。



# 4 Nginx、Gunicorn在服务器中分别起什么作用
https://docs.pingcode.com/ask/77123.html

Gunicorn是一个unix上被广泛使用的高性能的Python WSGI（Web Server Gateway Interface） UNIX HTTP Server。和大多数的web框架(flask)兼容，并具有实现简单，轻量级，高性能等特点。
Gunicorn用来解析HTTP请求的网关服务。是一种通用的接口规范，规定了web server（如Apache、Nginx）和web app（或web app框架）之间的标准


开头段落：

在Web服务器配置领域，**Nginx** 和 **Gunicorn** 是常用的组件，它们的角色和功能互补。**Nginx** 主要作为**反向代理服务器**和**静态资源服务器**，负责处理来自客户端的HTTP请求、执行缓存、提供加密以及负载均衡等功能，大大提高网站的并发处理能力和安全性。**Gunicorn** 则是一个**WSGI HTTP服务器**，专门用于运行Python Web应用程序，它作为Nginx与应用间的桥梁，处理Python应用的动态请求。Nginx可以有效地管理和分配网络请求、静态内容，而Gunicorn则确保Python应用顺畅处理动态内容。

对于**Nginx**的详细描述，其为目前互联网上广泛使用的开源Web服务器。由于其轻量级和高性能的特点，在大型网站中尤为受欢迎。它可以作为反向代理，帮助保护应用服务器不被直接暴露于外网风险中，并可以通过缓存内容来提升访问速度，此外，它还支持SSL/TLS加密，保障数据传输的安全。


## 4.1 NGINX的核心作用

Nginx作为一个高效的Web服务器，具备多种功能，但其核心作用可以概括为**反向代理**和**静态资源的高速处理**。作为反向代理，Nginx接收客户端的请求，并将这些请求转发到内部服务器（如Gunicorn）。通过这种方式，可以增强后端服务器的安全性、提升系统的伸缩性和可用性。此外，Nginx利用其高效的I/O模型在处理静态资源（如图片、CSS和JavaScript文件）时，能够支持高并发的连接，大幅减少页面载入时间。

为了承载高流量的访问，Nginx依赖于其**事件驱动架构**，该架构可以更好地管理多个网络连接，而不会因为创建大量的进程或线程而导致资源消耗过大。这种架构在维持同一时间数以千计的活跃连接中显示出了其出色的性能。

## 4.2 GUNICORN的核心作用

Gunicorn担任的则是应用程序和Web服务器之间的中介角色。准确来说，**Gunicorn** 是WSGI服务器的一种实现，WSGI（Web Server Gateway Interface）是Python定义的一个标准接口，允许Web服务器与应用程序进行通信。当Nginx将用户动态请求转发后，Gunicorn接手请求并调用相应的Python代码进行处理，最终将应用程序产生的响应返回给Nginx

由于Gunicorn是专门为运行Python程序设计的，它高度**优化了Python WSGI应用程序**的执行。Gunicorn通过生成多个工作进程来处理请求，充分利用CPU资源，提升并发处理能力。这些工作进程由一个或多个工作线程来监控，确保效率与稳定性。

## 4.3 NGINX AND GUNICORN的协同工作

在一个典型的Python Web应用部署中，**Nginx和Gunicorn协同工作**，将各自的优势结合起来。Nginx首先作为网站的门面，负责接收用户请求。在这种架构中，Nginx通常处理所有的静态文件请求，这些请求可以直接由服务器快速响应，而不必占用应用程序服务器的资源。而对于那些需要查询数据库或涉及应用程序逻辑的动态请求，则由Nginx代理给Gunicorn，由Gunicorn处理这些请求并调用Python代码。

通过这一层层的代理和处理，**加强了安全性**也提高了效率。Nginx与Gunicorn之间通常通过一个socket连接进行通信，这样的设置保证了数据传输的安全性和速度。

## 4.4 **四、性能优化与安全策略**

在这一服务架构中，为了进一步**提高性能并确保安全**，开发人员和运维工程师可以采取多种优化策略。例如，配置Nginx的缓存机制可以显著提高网站加载速度，使用限速限制可防止流量洪峰对服务器造成过大压力。同时，可以实现SSL/TLS加密，确保数据在传输过程中的安全。

Gunicorn方面，通过调整工作模式和工作进程数量，可以使服务器更加精确地满足应用需求。在资源利用上，合理设置超时时间和负载均衡策略，可以防止单点过载并提高系统整体的稳定性与响应速度。

综上所述，**Nginx负责高效地处理静态资源和管理客户端请求**，而**Gunicorn则确保Python应用能够良好地运行**。二者的结合成为了部署Python Web应用服务器的黄金搭档，该搭档被广泛认为可以提供强大、灵活和高效的Web服务解决方案。

## 4.5 **相关问答FAQs

**Nginx和Gunicorn在服务器中有什么不同作用？**
Nginx是一种高性能的Web服务器，常用于作为反向代理服务器来处理客户端的请求。它可以有效地分发请求到后端的应用服务器，如Gunicorn。Nginx还可以处理静态文件的请求，让应用服务器专注于动态内容的生成，提高整体的网站性能。

**Gunicorn在服务器中起到什么作用？**
Gunicorn是一个支持WSGI协议的Python应用服务器，用于运行Python web应用程序。它能够调用应用程序提供的WSGI接口，处理来自Nginx等反向代理服务器的请求，将动态内容生成并返回给客户端。通过Gunicorn，Python web应用程序可以实现并发处理请求，提高网站的性能和稳定性。

**为什么在服务器中需要同时使用Nginx和Gunicorn？**
在生产环境中，通常会同时使用Nginx和Gunicorn来搭建Python web应用程序的服务架构。Nginx作为反向代理服务器负责接收和分发请求，处理静态文件以及负载均衡，而Gunicorn作为应用服务器负责运行Python应用程序并处理动态内容的生成。这样的组合可以有效提高网站的性能、安全性和稳定性。



# 5 NGINX versus Node.js


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

![](../01_网络基础理论/image/Pasted%20image%2020250116212542.png)

# 6 Multi- versus Single-Thread Server
## 6.1 MULTI-THREAD (ODER SYNCHRONER) SERVER 

![](../01_网络基础理论/image/Pasted%20image%2020250116212630.png)

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



## 6.2 SINGLE-THREAD (ODER ASYNCHRONER) SERVER

![](../01_网络基础理论/image/Pasted%20image%2020250116213003.png)

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


# 7 Multi- versus Single-Thread Server

![](../01_网络基础理论/image/Pasted%20image%2020250116213248.png)

- **_Apache_**: Multi-Thread-Server mit synchroner Verarbeitung und blockierenden Threads
- _**nginx**_: Single-Thread-Server mit asynchroner Verarbeitung und nicht-blockierende Threads (vergleichbar zu Node.js)


# 8 Prinzipien der Erstellung dynamischer Webseiten (I/III)

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


