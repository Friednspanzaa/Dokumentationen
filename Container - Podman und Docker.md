# Container Schulung 
mit Thomas Hanke & Jasmin Fikus

- Part 1: Podman & Docker
- Part 2: Container As A Service Kubernetes)

### Verwendete Links

| Link | Beschreibung |
| ----------- | ----------- |
| [Hub.Docker](https://hub.docker.com/) | Hier holt sich Docker seine Images |
| [Sematic Versioning](https://semver.org/lang/de/) | Standartisierung bei Versionierungen | 




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


## Podman:

### Linux Kommando Aufbau:
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

Wir starten ein interaktives Alpine-Image in einem neuem Pseudoterminal ()
- `podman run -it alpine`
  - `-i`: Interative, Ein und Ausgabe übertragen
  - `-t`: Terminal, Pseudoterminal um die Ausgabe irgendwo anzuzeigen
 ---

Wir verknüpfen unser bash mit dem Container. Wir springen quasi hinein und können diese auch Beenden.
- `podman attach $NAME`
  

## Namespaces & Controll-Groups

 Namespaces nutzen wir um eindeutige Interpretationen zu ermöglichen. Ein Container erhält seinen eigenen Namespace in welcherm er arbeiten kann.

- `man namespaces`

Mit cgroups (Controll-Groups) könnte ich einschränken wie viel der Container z.b. auf ein Prozessor zugreifen könnte.

Es gibt auch Control-Groups die z.b. Block-Devices steuern bzw. den Zugriff steuern.

Es sollte immer eine Begrenzung des Memorys und der sonstigen Hardware geben, damit der Container nicht den kompletten Host übernimmt (Fehlprogrammierung)

> Ein Container ist nur eine Hülle. Arbeiten selbt tut der "Prozess". Also Docker oder Podman. 

- `podman info` 
  - Zum Nachschauen welcher Variablen und Einstellungen im Podman hinterlegt sind.

## Registries

Wir haben über Registries geredet :-)

## Logs

Jeder Container hinterlässt Logs. Diese können im nachhinein beobeachtet werden, aber auch zur Laufzeit.
z.B. mit: 
- `podman logs $Contianer-Name`
  - `-f` für follow


## Den Container bedienen

> Den Container-Name mit `podman container ls` rausfinden.

![Docker Container Lifecyle Management](images/docker_lifecyle.jpg)


- `Strg` + `p` + `q` - Zum Beenden eines Containers.


- `man podman-run | grep Detached`
  - Gibt alle Man Pages für "podman run" an und filtert nach dem Keyword "detached"

- `podman exec -it $Container-Name ls`

- `podman rm $Container-Name` Um den Container zu löschen inklusive Logs.

### Container Löschen

- `podman run --rm -it alpine`
- #TODO: Was machte der "Pune" Befehl?

### Container (um)benennen

- `podman run --name Robins_Container -it alpine` - Startet den Container mit einem eigenenm Namen
- `podman rename $Container-Name $Neuer-Container-Name` - Laufenden Container umbenenen. 

> Ein Container-Name muss immer einzigartig sein. Wenn ich versuche ein zusätzlichen Container mit selben Name versuche zu *erzeugen* (run) kommt es zum Fehler. Einfaches starten (start) hingegen klappt.

## Cattle Pet - Prinzip

Die Grundlagen der Virtualisierung
>Pets werden gepflegt, Cattle wird "weggeschmissen und neu geholt". 