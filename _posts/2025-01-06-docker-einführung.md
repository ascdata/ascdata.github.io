![docker](https://raw.githubusercontent.com/ascdata/ascdata.github.io/refs/heads/master/_posts/media/docker.jpg)

## Was ist Docker?
Docker ist ein Open-Source-Tool, mit dem Entwickler Anwendungen in Containern erstellen und betreiben können. 
Container sind kleine, isolierte Umgebungen, die alles enthalten was eine Anwendung braucht um zu laufen. 
Egal ob es der Code, die Bibliotheken oder die Einstellungen sind alles ist dabei. 
Und das Beste: Container laufen überall gleich, egal ob lokal, in der Cloud oder in der Produktion.

## Warum Docker so praktisch ist
Containerisierung gab es schon vor Docker, aber sie war echt kompliziert und hat viel technisches Know-how gebraucht. 
Docker hat das geändert. Mit einem standardisierten Ansatz macht es die Containerisierung einfacher.
Docker bietet eine Möglichkeit alles, was eine Anwendung braucht, in einem sogenannten Docker-Image zu verpacken. 
Dafür benutzt man eine Textdatei namens Dockerfile, die quasi der Bauplan für das Image ist. 
Dieses Image kann dann leicht geteilt und in jeder Umgebung gestartet werden. 
Dank Docker Hub, einem zentralen Online-Repository, kann man fertige Images speichern und mit anderen teilen.

## So funktioniert Docker
Mit Docker kann man Container ganz einfach erstellen, starten oder verwalten. Entweder über das Terminal oder eine grafische Oberfläche wie Docker Desktop. 
Die Anwendung bleibt dabei immer isoliert von anderen Containern und dem Host-System. 
Beispiel: Jeder Container ist wie eine Wohnung in einem Hochhaus. Einzelne Einheiten, aber die gleiche Infrastruktur.


![docker-prozess](https://raw.githubusercontent.com/ascdata/ascdata.github.io/refs/heads/master/_posts/media/docker_prozess.png)

## Die wichtigsten Bausteine von Docker
Damit du verstehst, wie Docker unter der Haube funktioniert, schauen wir uns die Kernbestandteile an.

### 1. Docker-Container
Container sind der Kern von Docker. 
Sie enthalten alles was nötig ist, um eine Anwendung auszuführen und laufen komplett unabhängig. 
Dafür brauchen sie wenig Ressourcen und starten sehr schnell. 

### 2. Docker-Images
Wenn Container die Wohnungen sind, dann sind Docker-Images die Baupläne. 
Ein Docker-Image ist eine Vorlage, die alles definiert was der Container später braucht. 
Images bestehen aus Schichten, die jede Änderung darstellen. Zum Beispiel eine Datei hinzufügen oder ein Paket installieren. 

### 3. Dockerfiles
Ein Dockerfile ist eine Textdatei, in der Schritt für Schritt beschrieben wird, wie ein Image gebaut werden soll.
Man kann ein Basis-Image angeben, Befehle ausführen, Dateien kopieren oder Umgebungsvariablen setzen.

Hier ein simples Beispiel:

```bash
FROM debian:latest
RUN apt update && apt install -y python
COPY programm.py /programme/
WORKDIR /programme
CMD ["python", "programm.py"]
```

Dieses Dockerfile macht Folgendes: 
- Es verwendet das neueste Debian-Image als Grundlage
- Führt ein Update der Paketlisten durch und installiert Python
- Überträgt die Datei programm.py in das Verzeichnis /programme im Container
- Definiert /programme als Arbeitsverzeichnis für nachfolgende Befehle
- Führt programm.py mit Python aus, wenn der Container gestartet wird.

## Welche Vorteile bietet Docker?

### Vereinfachte Entwicklung
Docker erleichtert die Entwicklung. 
Anwendungen werden in Containern verpackt, sodass Entwickler unabhängig an verschiedenen Teilen arbeiten können. 
Das sorgt dafür, dass später alles reibungslos zusammenpasst. 
Auch das Testen wird leichter und mögliche Probleme können frühzeitig entdeckt werden.

### Mehr Portabilität
Docker macht Anwendungen portabel. 
Container laufen überall gleich, egal ob auf deinem Laptop, in einer Testumgebung oder auf einem Server in der Cloud. 
Damit gibt es keine Kompatibilitätsprobleme mehr und das Bereitstellen auf verschiedenen Plattformen wird zum Kinderspiel.

### Höhere Effizienz
Container sind leichtgewichtig und starten schnell. 
Im Vergleich zu virtuellen Maschinen sind sie effizienter, weil sie weniger Ressourcen brauchen. 
Das heißt, man kann mehr Anwendungen mit den gleichen Ressourcen betreiben und spart dabei Zeit.

### Einfache Skalierung
Wenn eine Anwendung wächst, kann sie mit Docker ganz leicht skalieren. 
Mehr Container zu starten, um mehr Nutzer oder Datenverkehr zu bewältigen, ist schnell gemacht. 
So kann man flexibel bleiben, egal wie viel Last auf einem System liegt.

### Testen und Bereitstellen leicht gemacht
Mit Docker ist auch das Testen und Veröffentlichen von Anwendungen entspannter. 
Docker-Images lassen sich einfach versionieren und Änderungen sind leicht nachzuverfolgen. 
Falls etwas schiefgeht, kann man ohne großen Aufwand zurück zu einer funktionierenden Version. 
Außerdem funktioniert Docker perfekt mit CI/CD-Pipelines, um Build- und Bereitstellungsprozesse zu automatisieren.

## Wie benutzt man Docker?

Docker ist einfach zu benutzen, wenn man die Basics draufhat. Folgende Schritte sind grundsätzlich notwendig:

### 1. Docker installieren
Auf der offiziellen [Docker-Webseite](https://www.docker.com/) kann man den Installer herunterladen.
Es gibt Versionen für Windows, macOS und verschiedene Linux-Distributionen.

### 3. Docker-Images nutzen oder selbst bauen
Docker arbeitet mit Images, die als Vorlage für deine Container dienen. Hier gibt es zwei Möglichkeiten:

#### Fertige Images nutzen: 
Auf [Docker Hub](https://hub.docker.com/) gibt es eine Vielzahl von Images, die ggf. bereits alle benötigten Anforderungen erfüllen.
Viele gängige Anwendungen haben offizielle Images, die man direkt nutzen kann. 

Image laden:

```bash
docker pull <image_name>:<version>
```

Ersetze <image_name> und <tag> mit dem Namen und der Version des Images.

#### Eigenes Image erstellen: 
Wenn kein fertiges Image passt, kann man ein eigenes bauen. 
Dazu schreibt man ein Dockerfile, welches beschreibt wie das Image aussehen soll.


```bash
docker build -t mein-container .
```
- "docker build" startet den Prozess ein Docker-Image zu erstellen
- "-t mein-container" gibt dem Container den Namen meincontainer
- "." gibt das Build-Verzeichnis an (in diesem Fall das aktuelle Verzeichnis)

### 3. Einen Container starten
Sobald das Image bereit steht, kann man einen Container mit folgendem Befehl daraus starten.

```bash
docker run -p 8080:80 meincontainer
```
- "docker run" startet einen neuen Container
- "-p 8080:80" legt die Portzuordnung zwischen dem Host-System und dem Container fest (8080=Port auf Host-System, 80=Port im Container)
- "meincontainer" ist der Name des Docker-Images

### 4. Kommunikation zwischen Containern
Container sind erstmal voneinander getrennt. Wenn sie aber miteinander reden sollen, kann man ein eigenes Netzwerk erstellen:

```bash
docker network create mein-netzwerk
```
- "docker network create" erstellt ein neues Netzwerk
- "mein-netzwerk" ist der Name des zu erstellenden Netzwerks

```bash
docker run --name programm --network mein-netzwerk mein-programm
docker run --name datenbank --network mein-netzwerk meine-datenbank
```
- "docker run" startet einen neuen Container
- "--name programm" bzw. "--name datenbank" gibt dem gestarteten Container einen Namen
- "--network mein-netzwerk" verbindet den Container mit dem Netzwerk mein-netzwerk
- "mein-programm" bzw. "meine-datenbank" ist das Docker-Image das als Grundlage für den Container dient. Es wird zuerst lokal gessucht. Wenn es lokal nicht vorhanden ist versucht Docker das Image aus einem Repository wie Docker Hub herunterzuladen.

Jetzt kann das Programm die Datenbank unter dem Hostnamen datenbank erreichen.

### 5. Nützliche Docker-Befehle
Hier ein paar Befehle, die man oft brauchen wird:

- docker pull: Lade ein Image aus Docker Hub
- docker run: Starte einen Container
- docker build: Erstelle ein Image aus einem Dockerfile
- docker ps: Zeige alle laufenden Container an
- docker images: Zeige alle vorhandenen Images
- docker stop: Stoppe einen Container
- docker rm: Lösche einen gestoppten Container
- docker rmi: Lösche ein Image



## Docker Compose

Wenn man Docker immer mehr in den Entwicklungsalltag integriert, sollte man ein Tool kennenlernen das vieles einfacher macht: Docker Compose. 
Damit lassen sich Anwendungen, die aus mehreren Containern bestehen, problemlos verwalten. 
Alles was man dafür braucht ist eine YAML-Datei, in der der gesamte Software-Stack definiert wird.
Somit ist es möglich mehrere Docker-Container als einen zusammenhängenden Dienst zu betreiben. 
Anstatt jeden Container einzeln zu starten und Netzwerke sowie Volumes manuell zu konfigurieren, fasst Docker Compose alles in einer Datei namens docker-compose.yml zusammen.

### Wie verwendet man Docker Compose?

![docker-compose](https://raw.githubusercontent.com/ascdata/ascdata.github.io/refs/heads/master/_posts/media/docker-compose.jpg)

#### docker-compose.yml erstellen
Im ersten Schritt erstellt man eine Datei namens docker-compose.yml im Hauptverzeichnis des Projekts. 
Darin wird festgelegt, welche Dienste die Anwendung benötigt und wie sie miteinander verbunden sind. 
In diesen Beispiel wird ein Tomcat-Webserver und eine PostgreSQL-Datenbank bereitgestellt.

```yaml
version: '3.8'

# Netzwerke definieren
networks:
  meinNetzwerk: # Name des Netzwerks
    driver: bridge

# Dienste definieren
services:
  tomcat: # Name des Dienstes (Tomcat-Webserver)
    image: tomcat:latest # Verwende das offizielle Tomcat-Image
    ports:
      - "8080:8080" # Mappe Port 8080 auf dem Host zu Port 8080 im Container
    networks:
      - meinNetzwerk # Füge den Dienst zum Netzwerk hinzu
    depends_on:
      - postgres # Warte, bis der PostgreSQL-Dienst gestartet ist
    environment:
      - JAVA_OPTS=-Xms512m -Xmx1024m # Optionale Speicheranpassung für die JVM
    volumes:
      - ./webapps:/usr/local/tomcat/webapps # Ordner für Dateien

  postgres: # Name des Dienstes (PostgreSQL-Datenbank)
    image: postgres:latest # Verwende das offizielle PostgreSQL-Image
    environment: # Umgebungsvariablen für die Datenbank
      POSTGRES_USER: benutzer # Benutzername für den Zugriff
      POSTGRES_PASSWORD: passwort123 # Passwort für den Benutzer
      POSTGRES_DB: meine_datenbank # Name der zu erstellenden Datenbank
    volumes:
      - db-daten:/var/lib/postgresql/data # Speicherort der Datenbankdaten
    networks:
      - meinNetzwerk # Füge den Dienst zum Netzwerk hinzu

# Volumes definieren, um Daten persistent zu speichern
volumes:
  db-daten:
    driver: local # Nutzt den Standard-Treiber
````

#### Dienste starten
Im zweiten Schritt können mit dem Befehl
```bash
docker-compose up -d
``` 
alle in der YAML-Datei definierten Dienste gestartet werden. 
Der Parameter -d sorgt dafür, dass die Container im Hintergrund laufen.
Falls es das erste Mal ist oder sich das Dockerfile geändert hat baut Docker Compose automatisch die nötigen Images.

#### Container prüfen
Nach dem Start sollte man sicherstellen, dass die Container korrekt laufen. 
Mit folgendem Befehl können die laufenden Container angezeigt werden:
```bash
docker ps
```

Beispielausgabe:
```bash
CONTAINER ID   IMAGE           COMMAND                  PORTS                    NAMES
a1b2c3d4e5f6   tomcat:latest   "catalina.sh run"        0.0.0.0:8080->8080/tcp   tomcat
g7h8i9j0k1l2   postgres:latest "docker-entrypoint.s…"   5432/tcp                 postgres
```

#### Zugriff testen
Tomcat-Webserver: Öffne http://localhost:8080 im Browser des Hostsystems. 
Hier sollte die Tomcat-Willkommensseite erscheinen.
PostgreSQL-Datenbank: Verbinde dich mit einem Datenbanktool. 
Die Verbindungs-URL könnte so aussehen: 
```bast
jdbc:postgresql://localhost:5432/meine_datenbank
```

Und fertig wäre eine lauffähige Webserver-Umgebung inklusive Datenbank, ohne auswändige Installation und Konfiguration.
