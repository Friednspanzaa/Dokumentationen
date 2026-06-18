# Container Schulung
mit Thomas Hanke & Jasmin Fikus

- Part 1: Podman & Docker
- Part 2: Container As A Service Kubernetes)

### Verwendete Links

| Link | Beschreibung |
| ----------- | ----------- |
| [Hub.Docker](https://hub.docker.com/) | Hier holt sich Docker seine Images |
| [Sematic Versioning](https://semver.org/lang/de/) | Standartisierung bei Versionierungen |
| [GitIgnore](https://github.com/github/gitignore) | Sammlung an git.ignore-Dateien.

## WörterWolke

Docker - Podman
Kubernetes - Open Shift - Swarm
Image - Container
YAML - Namespace - Skalierung
"Security"

Docker läuft als Root-Daemon, wenn ich also Docker aufrufe, werde ich Root. Podman ist "rootless". Es läuft also mit den Rechten, mit denen ich starte.

## Fork Exec Model

Bei Fork wird das Programm in einem neuem Prozess gestartet. Sollte es fertig sein, oder beendet werden, übernimt der Parent-Prozess.
Bei Fork wird das Programm in einem neuem Prozess gestartet. Sollte das Programm dann geschlossen werden, gibt es kein Parent-Prozess der übernemen könnte.

## Skalierung

- vertikale Skalierung: Mehr CPU Power, mehr Memory
- horizontale Skalierung: Mehr Rechner, mehr Server

## Sicherheit

- CVE : Common Vunerable Exposiure - BEKANNTE Epxloits und Bugs
- CVSS: Common Vunerable Scoring System - Bewertung dieser Fehler.

## Versionierung

[Sematic Versioning](https://semver.org/lang/de/) dient der eindeutigen Versionierung von Software.

- `Major` - Änderungen die inkompatibilität zu früheren Versionen erzeugen könnten
- `Minor` -  Neue Funktionen die aber Kompatibel zu früheren Versionen sind.
- `Patch` - Bugfixes

### Linux Kommando Aufbau

- Kommando (`ls`)
- Optionen (`-a`)
- Argumente (`/etc/`)

Podman läuft ein wenig anders, mit Sub und oder auch Sub Sub Kommandos.

`podman container attach --help`

- Kommando: `podman`
- Subkommando: `container`
- SubSubkommando: `attach --help`

## Unser erster Container

`podman run hello-world`

Podman versucht direkt den Container runterzuladen und ihn auszuführen. Bei erneutem Ausführen des Containers ist das Image bereits runtergeladen und muss nicht erneut runtergeladen werden. Podman versucht den Alias "hello-world" aufzulösen und greift dabei auf die Datei `/etc/containers/registries.conf.d/000-shortnames.conf` zu.

- `FGCN` Fully Qualified Container Namen
  - `quay.io/podman/hello:latest`

| Befehl | Wirkung |
| ----------- | ----------- |
| `cat /home/User/.local/share/containers` | Hier speichert Podman seine Images |
| `podman container ls` | Zeigt den status aller Container die installiert/runtergeladen sind. |
| `podman container ls -a` | Infos über ALLE Verwendeten Container an. Auch die, die geschlossen sind. |

> *Red Hat Open Shift Local* ist ein kostenloses Programm in dem man Open-Shift Lokal ausprobieren kann.

## Wir bauen einen Container

Wir starten ein interaktives Alpine-Image in einem neuem Pseudoterminal
- `podman run -it alpine`
  - `-i`: Interative, Ein und Ausgabe übertragen
  - `-t`: Terminal, Pseudoterminal um die Ausgabe irgendwo anzuzeigen
  
 ---

Wir verknüpfen unser bash mit dem Container. Wir springen quasi hinein und können diese auch Beenden.
- `podman attach $NAME`

- `podman pull $IMAGE-NAME` - Ein Image einfach nur Runterladen

## Namespaces & Controll-Groups

 Namespaces nutzen wir um eindeutige Interpretationen zu ermöglichen. Ein Container erhält seinen eigenen Namespace in welcherm er arbeiten kann.

- `man namespaces`

Mit cgroups (Controll-Groups) könnte ich einschränken wie viel der Container z.b. auf ein Prozessor zugreifen könnte.

Es gibt auch Control-Groups die z.b. Block-Devices steuern bzw. den Zugriff steuern.

Es sollte immer eine Begrenzung des Memorys und der sonstigen Hardware geben, damit der Container nicht den kompletten Host übernimmt (Fehlprogrammierung)

> Ein Container ist nur eine Hülle. Arbeiten selbt tut der "Prozess". Also Docker oder Podman.

- `podman info`
  - Zum Nachschauen welcher Variablen und Einstellungen im Podman hinterlegt sind.

## Registrys

Wir haben über Registrys geredet :-)

## Logs

Jeder Container hinterlässt Logs. Diese können im nachhinein beobeachtet werden, aber auch zur Laufzeit.
z.B. mit:

- `podman logs $Contianer-Name`
  - `-f` für follow

## Den Container bedienen

Den Container-Name mit `podman container ls` rausfinden.

![Docker Container Lifecyle Management](images/docker_lifecyle.jpg)

- `Strg` + `p` + `q` - Zum Beenden eines Containers.

- `man podman-run | grep Detached`
  - Gibt alle Man Pages für "podman run" an und filtert nach dem Keyword "detached"

- `podman exec -it $Container-Name ls`

- `podman rm $Container-Name` Um den Container zu löschen inklusive Logs.

### Container Löschen

- `podman run --rm -it alpine` - Führt den Container erst aus, löscht ihn aber, nachdem er beendet wurde.
- `podman container prune` - Alle Container (die nicht laufen) löschen
- `podman image rm nginx:latest` - Löscht das Image von "nginx:latest"

### Container (um)benennen

- `podman run --name Robins_Container -it alpine` - Startet den Container mit einem eigenenm Namen
- `podman rename $Container-Name $Neuer-Container-Name` - Laufenden Container umbenenen.

> Ein Container-Name muss immer einzigartig sein. Wenn ich versuche ein zusätzlichen Container mit selben Name versuche zu *erzeugen* (run) kommt es zum Fehler. Einfaches starten (start) hingegen klappt.

## Cattle Pet - Prinzip

Die Grundlagen der Virtualisierung
>Pets werden gepflegt, Cattle wird "weggeschmissen und neu geholt".

## Volumes, Binds und tmpfs

### Volumes

> Volumes sind Speicher auf die verschiedene Container zugreifen können

- `podman volume create $VOLUME-NAME` - Das Volume erzeugen
- `podman run --volume robins_volume:/mnt --rm --name volume_test -it alpine`

| Kommando | Wirkung |
| ----------- | ----------- |
| `podman run` | Kommando für Podman |
| `--volume robins_volume:/mnt` | Benutze das erzeugte Volume, und binde es unter /mnt ein. |
| `--rm` | LÖSCHE den Container, nachdem er beendet wurde |
| `--name volume_test` | Benenne den Container (unnötig) |
| `-it alpine` | interaktiv, pseudoterminal, Alpine-Image |

### Binds

`podman run --volume /home/User/meinUserVerzeichnis/:/mnt:z --rm --name volume_test -it alpine`

| Kommando | Wirkung |
| ----------- | ----------- |
| `podman run` | Kommando für Podman |
| `--volume /home/User/meinUserVerzeichnis/:/mnt:z` | Benutze das erzeugte Volume, und binde es unter /mnt ein. Das z-Flag erlaubt die Nutzung auch unter SE-Linux und das der Container auf das Verzeichniss zugreifen darf. |
| `--rm` | LÖSCHE den Container, nachdem er beendet wurde |
| `--name volume_test` | Benenne den Container (unnötig) |
| `-it alpine` | interaktiv, pseudoterminal, Alpine-Image |

## Netze & Ports

`podman run -it --rm nginx`

> Wir können einen nginx Starten, der geöffnete Port (80) ist allerdings nicht erreichbar. Wir wollen den Freigegebenen Port vom Host aufrufen können.

`podman run -ditp 8080:80 --rm nginx`

| Kommando | Wirkung |
| ----------- | ----------- |
| `podman run` | Kommando für Podman |
| `-dp 8080:80` | Detached, Port öffnen: Host 8080 auf Container 80 |
| `--rm` | Lösche den Container dannach wieder |
| `nginx` | Nutze das Image nginx |

[Zum Testen](http://localhost:8080)

> podman exec -it cid sh

## Work

`podman run -dp 8080:80 -v /home/User/meinUserVerzeichnis:/usr/share/nginx/html --name Robins_Port_Test nginx`

| Kommando | Wirkung |
| ----------- | ----------- |
| `podman run` | Kommando für Podman |
| `-dp 8080:80` | Detached, Port öffnen: Host 8080 auf Container 80 |
| `-v /home/User/meinUserVerzeichnis:/usr/share/nginx/html` | Binde den Ordner MeinUserVerzeichnis auf /usr/share/nginx/html/ |
| `--name Robins_Port_Test` | Vergib einen schönen, sauberen Namen |
| `nginx` | Nutze das Image nginx |

## Umgebung Aufräumen

- `podman container prune`
- `podman volume prune`

## MYSQL und Word-Press

- `podman run -it --rm mysql`
- `podman image inspect mysql | less`

``` bash
  "Volumes": {
      "/var/lib/mysql": {}
  },
```

> wir finden raus, dass das Image ein volume BRAUCHT. Defaultmäßig würde der Container dies selber erzeugen.

- `podman volume create mysql_volume` - Wir erzeugen ein eigenes Volume
- `podman run -it --volume dww:/var/lib/mysql mysql`

``` bash
    You need to specify one of the following as an environment variable:
  - MYSQL_ROOT_PASSWORD
  - MYSQL_ALLOW_EMPTY_PASSWORD
  - MYSQL_RANDOM_ROOT_PASSWORD
```

- `podman run -it --volume dww:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQLALLOW_EMPTY_PASSWORD=true -e MYSQL_RANDOM_ROOT_PASSWORD=root mysql`

- `podman exec -it -l sh` Rein in die Maschine und Shell. podman exec -it -l sh

---

`podman network create net_wordpress`

- Ein Netzwerk erzeugen womit Container untereinander kommunizieren.

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

---

``` bash
podman run -d --name pma \
--network net_wordpress \
-p 8080:80 \
-e PMA_HOST=mariadb-test \
phpmyadmin
```

> `podman run -it --rm alpine ls -l`
>
> ls -l wird als Default übertragen. In dem Image ist ein "entry-Point" definiert. (Cmd)

``` bash
  "Config": {
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "Cmd": [
            "/bin/sh"
        ],
        "WorkingDir": "/"
  }
```

## Raus aus Bash -> Rein in die Beschreibung

> Mit `podman-compose` können wir über eine Composer-Datei ein komplettes Setup aufbauen. Datenbanken, Appserver und Webserver inkl. Netzwerk, Volumes und mehr.

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

> Secrets und Senstitive Daten sollten ausgelagert werden. Z.B. in eine .env Datei. Diese Datei wird dann nicht in der Versionierung (z.B. Git) aufgenommen.

``` BASH
DB_NAME= wordpress
MYSQL_USER= wpuser
MYSQL_PASSWORD= wppassword
MYSQL_ROOT_PASSWORD= safe
```

*.env* - Datei

>podman-compose muss seperat installiert werden das es nicht im standardmäßigem Podman-Paket vorhanden ist

#### Verwendete Befehle

| Kommando | Wirkung |
| ----------- | ----------- |
| `sudo dnf install podman-compose` | Podman-Compose installieren (Debian basiertes OS) |
| `podman-compose up` | Podman anhand der Compose-Datei starten. Der Befehl muss im selben Ornder ausgeführt werden wie die composer.yml ist. |
| `podman-compose down` | Die Sitzung wieder runterfahren. |
| `podman-compose stats` | Statistiken über alle Composierten Session auslesen. |
| `podman-compose --env-file prod` | Wir laden eine alternative Env.File (prod), Default: .env |
| `config` | Wir schauen uns die Configurations-Datei (Composer) damit an. Variabelen werden dabei schon aufgelöst. |

## Unser eigenener Container

Wir erzeugene eine neue Datei

``` BASH
FROM docker.io/library/nginx:alpine # Grundlegendes Image
COPY index.html /usr/share/nginx/html/index.html # Kopiere die index.html in den nginx Ornder
EXPOSE 80 # Öffne Port 80 (wir müssen diesen trotzdem beim aufrufen des Containers angeben)
```

*Containerfile* - Datei

- `podman image ls` - Zum einsehen aller Images (auch die, die wir selbst hinterlegt haben).
- `podman build -t localhost/itzbund/mein_image .`

| Kommando | Wirkung |
| ----------- | ----------- |
| `podman build` | Podman soll ein Image "bauen" |
| `-t [FQCN]/mein_image` | Wir hinterlegen einen Tag für unser neues Image damit wir dieses später auch benennen und starten können |
| `.` | Configurationsumgebung. (. = Aktueller Ordner) |

- `podman run -itp 8080:80 robins_erster_container:latest`

Um größere Images zu bauen können wir Teile aus bereits bestehenden Images "kopieren".
Dazu können wir z.B. das "FROM" oder "COPY" Kommando in der Containerfile vewenden. Mehr Infos dazu im Buch ab Seite 97.


Diese Doku senden an:
- Lars.Hennlein
- Iwana.Jäger
- Frederike
