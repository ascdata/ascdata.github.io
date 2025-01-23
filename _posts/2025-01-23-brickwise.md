---
title:  "Brickwise: Webscraping auf Immobilienscout24.de"
mathjax: true
layout: post
categories: media
---

**Brickwise** ist mein persönliches Projekt mit dem Ziel eine Plattform zur Sammlung, Speicherung und Analyse von Immobiliendaten zu erstellen. 
Dazu nutze ich **Web Scraping** mit Scrapy, um Daten von [Immobilienscout24.de](https://www.immobilienscout24.de/) zu sammeln.  
Diese werden entsprechend transformiert, in einer **PostgreSQL-Datenbank** gespeichert und anschließend mit **Apache Superset** zu interaktiven Dashboards visualisiert.

---

## Überblick über die Architektur

![brickwise_architektur.jpg](https://raw.githubusercontent.com/ascdata/ascdata.github.io/refs/heads/master/_posts/media/brickwise_architektur.jpg)

Das Projekt besteht aus den folgenden Komponenten:

- **Entwicklungsumgebung:**
  - Hier läuft der **Scrapy Spider**, der Immobiliendaten von Webseiten wie Immobilienscout24 sammelt.
  - Die Daten werden direkt in eine **PostgreSQL-Datenbank** auf einer Docker-VM gespeichert.
  - Die Entwicklungsumgebung umfasst zudem Visual Studio Code, Git und Python
  - Alle Python-Komponenten werden in einer venv (virutelle Entwicklungsumgebung) betrieben

- **Docker-VM:**
  - Hostet die PostgreSQL-Datenbank für die Datenspeicherung.
  - Hostet **Apache Superset** für die Analyse und Visualisierung.

### **Datenfluss**
1. **Scrapy Spider** sammelt Daten.
2. Daten werden über **psycopg2** in **PostgreSQL** geschrieben.
3. **Apache Superset** greift auf die PostgreSQL-Datenbank zu und erstellt Dashboards.

---

## Einrichtung

Zunächst wird ein Projektordner erstellt und eine virtuelle Umgebung eingerichtet. Dazu wird der folgende Befehl verwendet:

```bash
mkdir brickwise && cd brickwise
python -m venv venv
source venv\Scripts\activate 
```

Danach wird eine requirements.txt erstellt, die die benötigten Python-Pakete definiert:

```txt
scrapy
psycopg2
pandas
```

Die Abhängigkeiten können anschließend installiert werden:

```bash
pip install -r requirements.txt
```

### Scrapy

Ein neues Scrapy-Projekt wird mit dem folgenden Befehl erstellt:

```bash
scrapy startproject scrapy_brickwise
```

Die Datei settings.py im Scrapy-Projekt sollte angepasst werden, um den User-Agent und die Cookies zu konfigurieren.
Ich habe festgestellt, dass Seiten wie [Immobilienscout24.de](https://www.immobilienscout24.de/) Scrapy Spiders als [Webcrawler](https://de.wikipedia.org/wiki/Webcrawler) erkennen.
Dies kann umgangen werden, indem man dem Spider User-Agent-Informationen zuteilt; also Informationen wie anfragender Browser, Betriebssystem etc..

Beispiel:
```python
"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36"
```

Zudem müssen Cookie-Informationen mit übertragen werden. Diese können beispielsweise aus einem Browser, welcher [Immobilienscout24.de](https://www.immobilienscout24.de/) bereits aufgerufen hat, exportiert werden.
Am einfachsten geht das mit einem entsprechenden Browser Add-In, mit dem man Cookies exportieren kann.

Anschließend können die Cookie-Informationen dem Spider mitgegeben werden (zensiertes Beispiel):

```python
def start_requests(self):
        cookies = {
            "consent_status": "true",
            "IS24VisitId": "xxx",
            "IS24VisitIdSC": "xxx",
            "longUnreliableState": "xxx",
            "optimizelyUniqueVisitorId": "xxx",
            "utag_main": "xxx",
            "aws-waf-token": "xxx",
            "_dd_s": "xxx",
            "_lr_sampling_rate": "0",
            "ABNTEST": "xxx",
            "AWSALB": "xxx",
            "AWSALBCORS": "xxx",
            "seastate": "xxx",
        }
```

### Docker

Auf der Docker-VM wird eine docker-compose.yml erstellt, um die PostgreSQL-Datenbank und Apache Superset zu konfigurieren:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: postgres
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: brickwise
    ports:
      - "5432:5432"
    volumes:
      - ./postgres_data:/var/lib/postgresql/data

  superset:
    image: apache/superset
    container_name: superset
    environment:
      SUPERSET_WEBSERVER_PORT: 8088
    ports:
      - "8088:8088"
    depends_on:
      - postgres
    volumes:
      - ./superset_home:/app/pythonpath
```

Die Docker-Container werden mit folgendem Befehl gestartet:

```bash
docker-compose up -d
```


Der Scrapy Spider speichert Daten direkt in die PostgreSQL-Datenbank. Die Verbindung erfolgt über psycopg2 mit folgendem Beispielcode:

```python
import psycopg2
connection = psycopg2.connect(
    dbname="brickwise",
    user="admin",
    password="admin",
    host="localhost",
    port="5432"
)
cursor = connection.cursor()
cursor.execute("SQL-Statement"))
connection.commit()
```

Apache Superset wird konfiguriert, um auf die PostgreSQL-Datenbank zuzugreifen. Dazu wird in der Superset-Weboberfläche eine neue Datenbankverbindung mit der SQLAlchemy-URI hinzugefügt:

```txt
postgresql+psycopg2://admin:admin@localhost:5432/brickwise
```

Um Daten zu scrapen, wird der Scrapy Spider wie folgt gestartet:
```bash
scrapy crawl immoscout
```

Ein Parsing der Daten sieht dann beispielsweise so aus:

```python
def parse(self, response):
        # Überprüfe den HTTP-Statuscode
        self.log(f"HTTP Status: {response.status}")
        if response.status == 200:
            self.log("Zugriff erfolgreich! Die Seite wurde geladen.")

            # Extrahiere Immobilien-Daten
            for property_item in response.css("article.result-list-entry"):
                title = property_item.css("h2.result-list-entry__brand-title::text").get(default="N/A").strip()
                price = property_item.css("dd.result-list-entry__primary-criterion::text").get(default="N/A").strip()

                # Ausgabe der Daten im Log
                self.log(f"Immobilie: {title}, Preis: {price}")
        else:
            self.log("Fehler: Zugriff verweigert.")
```
Es wird anhand des Response-Codes 200 geprüft, ob ein Zugriff über den Spider grundsätzlich erfolgreich war und eine entsprechende Meldung wird zurück gegeben.
Die Werte die gecrawl werden sollen, können am einfachsten über die Entwicklertools des Browsers identifiziert werden (F12 im Browser).

![entiwcklertools.png](https://raw.githubusercontent.com/ascdata/ascdata.github.io/refs/heads/master/_posts/media/entiwcklertools.png)
