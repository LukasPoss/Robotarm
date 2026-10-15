# Roboterarm – CAD, 3D-Druck, Elektronik und Software

Ein selbst konstruierter Roboterarm, bei dem Mechanik, Elektronik und Software zusammenkommen. Das Projekt ist in Siemens NX entstanden und wird Schritt für Schritt um eigene Steuerung, Positionsrückmeldung und eine Weboberfläche erweitert.

Der Roboterarm ist ein laufendes Eigenbauprojekt. Nicht alles ist bereits fertig – ein Teil der Arbeit besteht gerade darin, die Mechanik im realen Aufbau zu testen und daraus die nächsten Schritte für die Steuerung abzuleiten.

## Warum ich das Projekt angefangen habe

Die Idee hatte ich schon während meines Maschinenbaustudiums. Im Technikum des Umwelt-Campus Birkenfeld stand ein Roboterarm, den ich irgendwann selbst bewegen durfte. Danach wollte ich einen eigenen Arm konstruieren und als Semesterprojekt umsetzen. Eine Betreuung dafür kam damals leider nicht zustande.

Nach meiner Ausbildung zum Fachinformatiker für Anwendungsentwicklung habe ich die Idee wieder aufgegriffen und angefangen, das Projekt eigenständig umzusetzen. So kann ich mehrere Bereiche verbinden, die mich interessieren: CAD und Konstruktion, 3D-Druck, Elektronik und Embedded-Programmierung.

Ich konstruiere gerne Bauteile so, dass sie zusammenpassen und sich möglichst sinnvoll wiederverwenden lassen. Deshalb ist der Arm nicht nur eine Sammlung einzelner Teile, sondern auch eine Art Baukastensystem. Komponenten und Ideen aus früheren Projekten fließen dabei ebenfalls ein.

## Projektstatus

**Status: in Entwicklung**

- [x] Mechanische Konstruktion in Siemens NX 10
- [x] Eigene Bauteile für den 3D-Druck konstruiert
- [x] Mechanischer Aufbau mit sechs MG90S-Servos
- [x] Servos für kontinuierliche Rotation umgebaut
- [x] ESP32-S3-WROOM-1 als zentrale Steuerung ausgewählt
- [x] Raspberry Pi als Host für Website und API eingerichtet
- [x] Weboberfläche und Kamerastream in die Website eingebunden
- [ ] Positionspotentiometer vollständig montieren und kalibrieren
- [ ] Mechanik und Motorbewegungen systematisch vermessen
- [ ] Positionsrückmeldung und Regelung umsetzen
- [ ] Weboberfläche vollständig mit der Robotersteuerung verbinden
- [ ] Bauanleitung und Montage-Dokumentation veröffentlichen

Die Liste bildet den aktuellen Entwicklungsstand ab und wird im Laufe des Projekts angepasst. Eine vorhandene Weboberfläche bedeutet noch nicht, dass bereits alle Achsen zuverlässig darüber geregelt werden können.

## Mechanischer Aufbau

Die Konstruktion wurde von mir selbst in **Siemens NX 10** erstellt. Die Bauteile sind für den eigenen Aufbau ausgelegt und werden überwiegend im 3D-Druck gefertigt. Für die Achsen kommen unter anderem **8-mm-Aluminiumhülsen** zum Einsatz. Sie dienen als mechanische Verbindung und ermöglichen gleichzeitig, Kabel durch die Konstruktion zu führen.

Ursprünglich hatte ich SG90-Servos vorgesehen. Bei der Reichweite und den zu erwartenden Belastungen des Arms waren mir deren Kunststoffgetriebe jedoch zu wenig robust. Außerdem war der verfügbare Bewegungsbereich für die geplante Mechanik nicht ausreichend. Deshalb bin ich auf MG90S-Servos mit Metallgetriebe umgestiegen.

Für das Projekt habe ich die MG90S-Servos auf kontinuierliche Rotation umgebaut. Dadurch lässt sich die Bewegung nicht mehr einfach über eine vorgegebene Servo-Winkelposition einstellen. Die tatsächliche Position muss daher über eine zusätzliche Rückmeldung erfasst werden.

### Getriebe und Positionsrückmeldung

Für den Antrieb und die Positionsmessung werden zwei unterschiedliche Zahnradübersetzungen verwendet:

- **Antrieb:** 16 Zähne am Motor und 48 Zähne am Gelenk. Das ergibt eine Untersetzung von 3:1. Das Gelenk dreht sich damit theoretisch mit einem Drittel der Motordrehzahl; das Drehmoment wird idealisiert entsprechend erhöht. Reibung und andere Verluste sind dabei nicht berücksichtigt.
- **Positionsmessung:** 16 Zähne am Gelenk und 24 Zähne am Potentiometer. Das Potentiometer dreht sich damit mit zwei Dritteln des Gelenkwinkels. Bei 270° Gelenkbewegung wären das theoretisch 180° am Potentiometer.

Das Potentiometer soll also nicht direkt die gesamte Gelenkbewegung mitmachen. Die Übersetzung hilft dabei, seinen mechanischen Drehbereich sinnvoll zu nutzen und zu vermeiden, dass es bei der vorgesehenen Gelenkbewegung an seinen Anschlag kommt.

Die Rückmeldung muss noch mechanisch und elektrisch abgestimmt sowie kalibriert werden. Spiel in den Zahnrädern, mechanische Toleranzen und die tatsächliche Bewegung der umgebauten Servos müssen bei der späteren Regelung berücksichtigt werden.

## Hardware

| Komponente | Verwendung |
|---|---|
| 6 × MG90S | Antriebe, für kontinuierliche Rotation umgebaut |
| ESP32-S3-WROOM-1 | Zentrale Steuerung des Roboterarms |
| Externe Potentiometer | Positionsrückmeldung der Gelenke |
| Raspberry Pi 4 Model B (2 GB) | Host für Website, API und Netzwerkdienste |
| ESP32-S3-CAM mit OV3660 | Kamerabild für die Weboberfläche |
| 5 V / 90 W Netzteil | Versorgung des Aufbaus; tatsächliche Stromaufnahme muss bei Bedarf gemessen werden |
| 8-mm-Aluminiumhülsen | Achsen und Kabeldurchführungen |
| 3D-gedruckte Bauteile | Individuell konstruierte mechanische Teile |

Die Leistungsangabe des Netzteils beschreibt dessen maximale Nennleistung, nicht den tatsächlichen Verbrauch des Arms. Die Stromaufnahme hängt unter anderem von Bewegung und mechanischer Belastung ab. Falls es im Betrieb zu Spannungseinbrüchen kommt, sind Messungen und gegebenenfalls Anpassungen der Versorgung sinnvoll.

## Software und Systemarchitektur

Die Weboberfläche, die API und die Mikrocontroller gehören zu einem gemeinsamen Projekt. Die Aufgaben sind dabei auf mehrere Komponenten verteilt:

1. **Weboberfläche:** Stellt die Bedienoberfläche und den Kamerastream im Browser bereit.
2. **Nginx:** Liefert die Website aus und leitet API- und Kameraanfragen weiter.
3. **FastAPI:** Stellt die API auf dem Raspberry Pi bereit. Bereits vorhandene Status-Endpunkte liefern Informationen zum Roboter und zur Kamera.
4. **ESP32-S3:** Ist als Controller für die eigentliche Motorsteuerung und das Einlesen der Positionssensoren vorgesehen.
5. **ESP32-S3-CAM:** Stellt den Kamerastream für die Website bereit.

Der Raspberry Pi betreibt die Web- und API-Komponenten. Für den Roboter ist ein separates WLAN vorgesehen, über das der Controller und die Kamera erreichbar sind. Die Netzwerkaufteilung soll die Kommunikation innerhalb des Aufbaus übersichtlich halten.

Die spätere Regelung soll möglichst lokal auf dem ESP32-S3 stattfinden. Dort können Sollposition und gemessene Position miteinander verglichen und die Motoren entsprechend angesteuert werden. Der Raspberry Pi soll dabei nicht jeden einzelnen zeitkritischen Motorschritt übernehmen müssen.

### Entwicklungsstand der Steuerung

Die Motoren werden zunächst direkt an ihren tatsächlichen Einbaupositionen getestet. Dazu nutze ich aktuell einen einfachen Testaufbau mit einem Arduino Uno und zwei Tastern, über die sich die Motorbewegung schrittweise in beide Richtungen untersuchen lässt.

Die Testintervalle sind dabei zunächst nur Steuerbefehle und keine garantierten Gelenkwinkel. Ziel ist es, das Bewegungsverhalten der jeweiligen Kombination aus Motor, Getriebe und Mechanik zu erfassen und die realen Bewegungsgrenzen zu bestimmen. Diese Erkenntnisse sind die Grundlage für die spätere Kalibrierung und Positionsregelung.

Die Weboberfläche und der Kamerastream sind bereits eingebunden. Die vollständige, zuverlässige Steuerung des Arms über die Weboberfläche ist dagegen noch in Arbeit.

## Bauanleitung und Dokumentation

Eine ausführliche Bauanleitung wird **extern veröffentlicht** und ist derzeit noch in Vorbereitung. Sie soll den mechanischen Aufbau, die Montage und ergänzende Hinweise nachvollziehbar dokumentieren.

Auch die CAD-Dateien werden im Zuge der weiteren Aufbereitung sortiert und ergänzt. Bis dahin ist dieses Repository vor allem die zentrale Anlaufstelle für den Projektstand und die zugehörigen Softwarebestandteile.

## Geplante nächste Schritte

- Mechanische Teile und CAD-Dateien aufräumen und sinnvoll gruppieren
- Bewegungsbereiche der einzelnen Gelenke vermessen
- Positionssensoren montieren und kalibrieren
- Steuerung mit Positionsregelung entwickeln
- Weboberfläche mit der Steuerung verbinden und das Verhalten unter realer Last testen
- Versorgung und Stromaufnahme einmessen und bei Bedarf um Elektrolyth Kondensatoren ergänzen
- Bauanleitung, CAD-Dokumentation und gegebenenfalls ein Montagevideo ergänzen

## Lizenz

Dieses Projekt steht unter der **GNU Affero General Public License v3.0 (AGPL-3.0)**. Die vollständigen Lizenzbedingungen stehen in der Datei [`LICENSE`](LICENSE).

## Hinweis

Der Roboterarm ist ein privates Entwicklungsprojekt und befindet sich noch im Aufbau. Funktionen, Anschlussdetails und Dokumentation können sich während der Entwicklung ändern. Bitte prüfe die jeweilige Hardware- und Softwarekonfiguration, bevor du Komponenten nachbaust oder den Aufbau in Betrieb nimmst.
