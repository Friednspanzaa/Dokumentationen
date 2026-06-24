# Grundlagen der Virtualisierung

mit Thomas Hanke\
Author: Robin Heydkamp, [robin.heydkamp@itzbund.de](robin.heydkamp@itzbund.de)

- Part 1: Podman & Docker
- Part 2: Container As A Service (Kubernetes & Open Shift)

## Übersicht

- [Grundlagen der Virtualisierung](#grundlagen-der-virtualisierung)
  - [Übersicht](#übersicht)
  - [Links](#links)
  - [Einführung](#einführung)
    - [WörterWolke](#wörterwolke)
    - [Docker vs Podman](#docker-vs-podman)
    - [Fork Exec - Model](#fork-exec---model)
    - [Skalierung](#skalierung)
    - [Sicherheit](#sicherheit)
    - [Versionierung](#versionierung)
    - [Linux Kommando Aufbau](#linux-kommando-aufbau)
  - [Unser erster Container](#unser-erster-container)
  - [Wir bauen einen Container (Vorlage mit alpine)](#wir-bauen-einen-container-vorlage-mit-alpine)
  - [Namespaces \& Control-Groups](#namespaces--control-groups)
  - [Registrys](#registrys)
  - [Logs](#logs)
  - [Der Container Lifecyle](#der-container-lifecyle)
    - [Container Löschen](#container-löschen)
    - [Container (um)benennen](#container-umbenennen)
    - [Cattle Pet - Prinzip](#cattle-pet---prinzip)
  - [Volumes, Binds und tmpfs](#volumes-binds-und-tmpfs)
    - [Volumes](#volumes)
    - [Binds](#binds)
    - [tmpfs](#tmpfs)
  - [Netze \& Ports](#netze--ports)
  - [Container in der Praxis](#container-in-der-praxis)
  - [Umgebung Aufräumen](#umgebung-aufräumen)
  - [MYSQL und Word-Press](#mysql-und-word-press)
    - [Netzwerke](#netzwerke)
      - [Verwendung](#verwendung)
        - [Kommando Erster Container](#kommando-erster-container)
        - [Kommando Zweiter Container](#kommando-zweiter-container)
  - [Raus aus Bash -\> Rein in die Beschreibung](#raus-aus-bash---rein-in-die-beschreibung)
    - [Verwendete Befehle](#verwendete-befehle)
  - [Unser eigener Container](#unser-eigener-container)

---

## Links

| Link | Beschreibung |
| ----------- | ----------- |
| [Hub.Docker](https://hub.docker.com/) | Hier holt sich Docker seine Images |
| [Sematic Versioning](https://semver.org/lang/de/) | Standartisierung bei Versionierungen |
| [GitIgnore](https://github.com/github/gitignore) | Sammlung an git.ignore-Dateien |
| [*Red Hat Open Shift Local*](https://developers.redhat.com/products/openshift-local) | Open Shift lokal ausprobieren. Developer Account benötigt |

## Einführung

### WörterWolke

``` sting
Docker - Podman
Kubernetes - Open Shift - Swarm
Image - Container
YAML - Namespace - Skalierung
"Security"
````

### Docker vs Podman

Docker läuft als Root-Daemon, wenn ich also Docker aufrufe, läuft der Container als Root. Podman ist "rootless". Die Container laufen also mit den Rechten, mit denen ich sie starte.

### Fork Exec - Model

Bei Fork wird das Programm in einem neuem Prozess gestartet. Sollte es fertig sein, oder beendet werden, übernimt der "Parent"-Prozess.
Bei Exec wird das Programm in gleichen Prozess gestartet. Sollte das Programm dann geschlossen werden, gibt es kein "Parent"-Prozess der übernemen könnte.

### Skalierung

- vertikale Skalierung: Mehr CPU Power, mehr Memory
- horizontale Skalierung: Mehr Rechner, mehr Server

### Sicherheit

- CVE : Common Vulnerability Exposure - Bekannte Exploits und Bugs
- CVSS: Common Vulnerability Scoring System - Bewertung dieser Fehler.

### Versionierung

[Sematic Versioning](https://semver.org/lang/de/) dient der eindeutigen Versionierung von Software.

- `Major` - Änderungen die inkompatibilität zu früheren Versionen erzeugen könnten
- `Minor` -  Neue Funktionen die aber Kompatibel zu früheren Versionen sind.
- `Patch` - Bugfixes

### Linux Kommando Aufbau

Beispiel: `ls -a /etc/`

- Kommando (`ls`)
- Optionen (`-a`)
- Argumente (`/etc/`)

Podman läuft ein wenig anders, mit Sub und oder auch Sub Sub Kommandos.

Beispiel: `podman container attach --help`

- Kommando: `podman`
- Subkommando: `container`
- SubSubkommando: `attach --help`

## Unser erster Container

Kommando: `podman run hello-world`

Podman versucht direkt das Image runterzuladen und es als Container auszuführen. Bei erneutem Ausführen des Containers ist das Image bereits runtergeladen und muss nicht erneut runtergeladen werden. Podman versucht den Alias "hello-world" aufzulösen und greift dabei z.B. auf die Datei `/etc/containers/registries.conf.d/000-shortnames.conf` zu.

- `FQCN` Fully Qualified Container Name
  - `quay.io/podman/hello:latest`

| Kommando | Wirkung |
| ----------- | ----------- |
| `cat /home/User/.local/share/containers` | Hier speichert Podman seine Images |
| `podman container ls` | Zeigt den Status aller Container die aktiv sind. |
| `podman container ls -a` | Infos über ALLE Verwendeten Container an. Auch die, die geschlossen sind. |

> [*Red Hat Open Shift Local*](https://developers.redhat.com/products/openshift-local) ist ein kostenloses Programm in dem man Open-Shift Lokal ausprobieren kann.

## Wir bauen einen Container (Vorlage mit alpine)

Wir starten ein interaktives Alpine-Image in einem neuem Pseudoterminal

- `podman run -it alpine`
  
| (Sub) Kommando | Wirkung |
| ----------- | ----------- |
| `podman run` | ein Podman Image starten |
| `-i` | interative, Ein und Ausgabe übertragen |
| `-t` | terminal, Pseudoterminal um die Ausgabe irgendwo anzuzeigen |
| `alpine` | Nutze das Image alpine |

 ---

Wir verknüpfen unser bash mit dem Container. Wir springen quasi hinein und können diese auch Beenden. Der Container-Name lässt sich mit `podman container ls` rausfinden.

- `podman attach $CONTAINER-NAME` - Sich an ein laufenden Container "hängen" bzw. reinspringen.

- `podman pull $IMAGE-NAME` - Ein Image einfach nur Runterladen

- `podman run -it alpine ls -l` - ls -l wird als "Default" übertragen. In dem Image ist ein "entry-Point" definiert, wo der Befehl ausgeführt wird. In diesem Falle die Kommandozeile (Cmd)

## Namespaces & Control-Groups

 Namespaces nutzen wir um eindeutige Interpretationen bzw. Definitionen zu ermöglichen. Ein Container erhält seinen eigenen Namespace in welchem er arbeiten kann. Im Namespace "Schmied" steht der Befehl "baue ein Schloss" für eine abschließbare Kette. Im Namespace "Königin" erstellt der gleiche Befehle ein Königliches Schloss mit Ballsaal und Thron.

- `man namespaces`

Mit cgroups (Control-Groups) lässt sich einschränken auf wie viele Ressourcen ein Container zugreifen darf. Zum Beispiel Prozessorleistung, Festplattenspeicher bis hin zu Lese/Schreibgeschwindigkeiten und verwendung von Inodes.

Es sollte immer eine Begrenzung des Arbeitsspeicher und der sonstigen Hardware geben, damit der Container nicht den kompletten Host übernimmt.

> Fehlprogrammierung ist immer möglich

Ein Container ist nur eine Hülle. Arbeiten selbt tut der "Prozess". Also Docker oder Podman.

- `podman info`
  - Zum Nachschauen welcher Variablen und Einstellungen im Podman hinterlegt sind.

## Registrys

Wir haben über Registrys geredet :-)

## Logs

Jeder Container hinterlässt Logs. Diese können im nachhinein beobeachtet werden, aber auch zur Laufzeit.
z.B. mit:

- `podman logs $CONTAINER-NAME`
  - `-f` für follow

## Der Container Lifecyle

Wir haben bereits einen Container erzeugt und gestartet (run) und können ihn über "`podman start`" und "`podman stop`" starten und stoppen.

![Docker Container Lifecyle Management](images/docker_lifecyle.jpg)

- `Strg` + `p` `q` - Zum Beenden eines Containers.
- Den Container-Name mit `podman container ls` rausfinden.

| Kommando | Wirkung |
| ----------- | ----------- |
| `man podman-run \| grep detached` | Gibt alle Man Pages für "podman run" an und filtert nach dem Keyword "detached" |
| `podman exec -it $CONTAINER-NAME ls` | Springt in den Container (interaktiv, terminal) und fürt das Kommando `ls` aus |
| `podman rm $CONTAINER-NAME` | Entfernt **gestoppte** Container inkl. aller Logs |

### Container Löschen

| Kommando | Wirkung |
| ----------- | ----------- |
| `podman run --rm -it alpine` | Führt den Container erst aus, löscht ihn aber, nachdem er beendet wurde. |
| `podman container prune` | Alle Container (die nicht laufen) löschen |
| `podman image rm nginx:latest` | Löscht das Image von "`nginx:latest`" |

### Container (um)benennen

| Kommando | Wirkung |
| ----------- | ----------- |
| `podman run --name $CONTAINER-NAME -it alpine` | Startet einen Container und vergibt einen eigenen Namen |
| `podman rename $CONTAINER-NAME $NEW-CONTAINER-NAME` | einen laufenden Container umbenenen. |

> Ein Container-Name muss immer einzigartig sein. Wenn ich versuche ein zusätzlichen Container mit selben Name versuche zu *erzeugen* (`podman run`) kommt es zum Fehler.

### Cattle Pet - Prinzip

Die Grundlagen der Virtualisierung.
Pets werden gepflegt, Cattle wird "weggeschmissen und neu geholt".

## Volumes, Binds und tmpfs

### Volumes

Volumes sind Speicher auf die verschiedene Container zugreifen können

- `podman volume create $VOLUME-NAME` - Das Volume erzeugen

Benutzung: `podman run --volume $VOLUME-NAME:/mnt --rm --name volume_test -it alpine`

| (Sub) Kommando | Wirkung |
| ----------- | ----------- |
| `podman run` | Kommando für Podman |
| `--volume $VOLUME-NAME:/mnt` | Benutze das erzeugte Volume, und binde es unter /mnt ein. |
| `--rm` | LÖSCHE den Container, nachdem er beendet wurde |
| `--name volume_test` | Benenne den Container 'volume_test' |
| `-it` | interaktiv, pseudoterminal |
| `alpine` | Nutze das Image alpine |

### Binds

Binds sind Speicher, die, im gegensatz zu Volumes, vom Host gemanaged werden und damit etwas "interaktiver" sind. Sie werden fast identisch eingebunden. Statt dem Volume-Namen verwenden wir den Ordner-Namen

Benutzung: `podman run --volume /home/User/Ordner1/:/mnt:z --rm --name volume_test -it alpine`

| (Sub) Kommando | Wirkung |
| ----------- | ----------- |
| `podman run` | Kommando für Podman |
| `--volume /home/User/Ordner1/:/mnt:z` | Binde das designierte Verzeichniss 'Ordner1' in dem Container unter /mnt ein. Das z-Flag erlaubt die Nutzung auch unter SE-Linux und das der Container auf das Verzeichniss zugreifen darf. |
| `--rm` | LÖSCHE den Container, nachdem er beendet wurde |
| `--name volume_test` | Benenne den Container 'volume_test' |
| `-it alpine` | interaktiv, pseudoterminal, Alpine-Image |

### tmpfs

Wir tmpfs mit einem halben Satz angesprochen... ( ͡° ͜ʖ ͡°)_/¯

## Netze & Ports

Wir können ein nginx-Image starten, der geöffnete Port (80) wäre allerdings nicht erreichbar. Wir wollen den Freigegebenen Port vom Host aufrufen können.

`podman run -dit -p 8080:80 nginx`

| Kommando | Wirkung |
| ----------- | ----------- |
| `podman run` | Kommando für Podman |
| `-dit` | detached, interative, terminal |
| `-p 8080:80` | Port öffnen: Host 8080 auf Container 80 |
| `nginx` | Nutze das Image nginx |

[Zum Testen](http://localhost:8080)

> Wir nutzen Port 8080 da die ersten 1000 Ports nur vom System vergeben werden sollen/dürfen.

## Container in der Praxis

Beispiel: `podman run -dp 8080:80 -v /home/User/Ordner1:/usr/share/nginx/html --name Port_Test_Container nginx`

| (Sub) Kommando | Wirkung |
| ----------- | ----------- |
| `podman run` | Kommando für Podman |
| `-dp 8080:80` | detached, Port öffnen: Host 8080 auf Container 80 |
| `-v /home/User/Ordner1:/usr/share/nginx/html` | Binde den Ordner Ordner1 auf /usr/share/nginx/html/ ein |
| `--name Port_Test_Container` | Vergib einen schönen, sauberen Namen |
| `nginx` | Nutze das Image nginx |

## Umgebung Aufräumen

- `podman container prune`
- `podman volume prune`
- `podman network prune`

## MYSQL und Word-Press

- `podman pull mysql` - Ein Mysql Image runterladen
- `podman image inspect mysql | less` - Das Image untersuchen

``` bash
  "Volumes": {
      "/var/lib/mysql": {}
  },
```

wir finden raus, dass das Image ein Volume **braucht**. Defaultmäßig würde der Container dies selber erzeugen. Wir können dies aber auch selber machen:
 `podman volume create mysql_volume` - und anschließend dem Container mitgeben mit: `podman run -it --volume mysql_volume:/var/lib/mysql mysql`
Trotzdem bekommen wir die Fehlermeldung:

``` bash
    You need to specify one of the following as an environment variable:
  - MYSQL_ROOT_PASSWORD
  - MYSQL_ALLOW_EMPTY_PASSWORD
  - MYSQL_RANDOM_ROOT_PASSWORD
```

Manche Container benötigen zusätzliche Variablen. Jede Variabele muss seperat beim Aufruf hinzugefügt werden.

- `podman run -it --volume mysql_volume:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_ALLOW_EMPTY_PASSWORD=true -e MYSQL_RANDOM_ROOT_PASSWORD=root mysql`

| (Sub) Kommando | Wirkung |
| ----------- | ----------- |
| `podman run` | Kommando für Podman |
| -it | interactive, pseudoterminal |
| --volume mysql_volume:/var/lib/mysql | Verwende das (vorher angelegte) Volume "mysqsl_volume" und binde es unter "/var/lib/mysql ein |
| -e MYSQL_ROOT_PASSWORD=root | Umgebungsvariable für den Container |
| -e MYSQLALLOW_EMPTY_PASSWORD=true | Umgebungsvariable für den Container |
| -e MYSQL_RANDOM_ROOT_PASSWORD=root | Umgebungsvariable für den Container |
| mysql | Nutze das Image mysql |

- `podman exec -it -l sh` - Rein in die Maschine und Shell starten. `-l` ist ein Indikator für den letzten verwendeten Container. Der Name oder die ID wären ebenfalls möglich.

### Netzwerke

Über ein Netzwerk können Container untereinander kommunizieren und sich gegenseitig als "Server" wahrnehmen.

`podman network create net_wordpress` - Erzeugt ein Containernetzwerk mmit der Bezeichnung: "net_wordpress"

#### Verwendung

##### Kommando Erster Container

``` bash
podman run -d \
--name mariadb-test \
--network net_wordpress \
-e MYSQL_DATABASE=wp \
-e MYSQL_USER=wpuser \
-e MYSQL_PASSWORD=geheim \
-e MYSQL_ROOT_PASSWORD=root \
-v vol_wordpress:/var/lib/mysql \
mariadb
```

##### Kommando Zweiter Container

``` bash
podman run -d --name pma \
--network net_wordpress \
-p 8080:80 \
-e PMA_HOST=mariadb-test \
phpmyadmin
```

## Raus aus Bash -> Rein in die Beschreibung

Mit `podman-compose` können wir über eine Composer-Datei ein komplettes Setup aufbauen. Datenbanken, Appserver und Webserver inkl. Netzwerk, Volumes und mehr.

>`podman-compose` muss i.d.R. nachinstalliert werden da es nicht im standardmäßigem Podman-Paket vorhanden ist. Z.B. mit `sudo dnf install podman-compose`

``` YAML
version: '3.8'

services:
  db:
    image: docker.io/library/mysql:8.0
    container_name: wordpress-db
    # restart: always ## Funktioniert nur unter Docker, da Docker einen Deamon benutzt.
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql 

  wordpress:
    image: docker.io/library/wordpress:latest
    container_name: wordpress-app
    environment:
      WORDPRESS_DB_HOST: db:3306 # Name des Services (Zeile 4) + Port
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
      WORDPRESS_DB_NAME: ${DB_NAME}
    ports:
      - "8080:80"
    depends_on:
      - db

volumes:
  db_data: # Volumen für die Datenbank, Default reicht aus.
```

*compose.yml* - Datei

> Secrets und Senstitive Daten (Hier als Variable mit ${XXX} gekennzeichnet) sollten ausgelagert werden. Z.B. in eine .env Datei. Diese Datei wird dann nicht in der Versionierung (z.B. Git) aufgenommen und/oder bekommt spezielle Zugriffsberechtigungen.

``` BASH
DB_NAME= wordpress
MYSQL_USER= wpuser
MYSQL_PASSWORD= wppassword
MYSQL_ROOT_PASSWORD= safe
```

*.env* - Datei

### Verwendete Befehle

| Kommando | Wirkung |
| ----------- | ----------- |
| `sudo dnf install podman-compose` | Podman-Compose installieren (Debian basiertes OS) |
| `podman-compose up` | Podman anhand der Compose-Datei starten. Der Befehl muss im selben Ornder ausgeführt werden wie die composer.yml ist. |
| `podman-compose down` | Die Sitzung wieder runterfahren. |
| `podman-compose stats` | Statistiken über alle composierten Session auslesen. |
| `podman-compose --env-file prod` | Wir laden eine alternative env.File mit dem Namen "prod", Default: .env |
| `config` | Wir schauen uns die Configurations-Datei (composer.yml) damit an. Variabelen werden dabei schon aufgelöst. |

## Unser eigener Container

Wir erzeugen eine neue Datei

``` BASH
FROM docker.io/library/nginx:alpine # Grundlegendes Image
COPY index.html /usr/share/nginx/html/index.html # Kopiere die index.html in den nginx Ornder
EXPOSE 80 # Öffne Port 80 (wir müssen diesen trotzdem beim aufrufen des Containers angeben)
```

*containerfile* - Datei

Kommando: `podman build -t [FQCN]/mein_image .`

| (Sub) Kommando | Wirkung |
| ----------- | ----------- |
| `podman build` | Podman soll ein Image "bauen" |
| `-t [FQCN]/mein_image` | Wir hinterlegen einen Tag für unser neues Image damit wir dieses später auch benennen und starten können |
| `.` | Configurationsumgebung. Woher sollen Variablen gezogen werden. (. = Aktueller Ordner) |

Kommando: `podman run -itp 8080:80 mein_image:latest`

| Kommando | Wirkung |
| ----------- | ----------- |

| `podman run` | Podman soll ein Image "bauen" |
| `-itp` | interactive, pseudoterminal, sende Port 80 (Container) auf 8080 (Host) |
| `mein_image:latest` | welches Image soll verwendet werden inklusive der Version |

Um größere Images zu bauen können wir Teile aus bereits bestehenden Images "kopieren".
Dazu können wir z.B. das "`FROM`" oder "`COPY`" Kommando in der Containerfile verwenden. Mehr Infos dazu im Buch ab Seite 97.

- `podman image ls` - Zum einsehen aller Images (auch die, die wir selbst hinterlegt haben)
