<!--

author:   Konstantin Kirchheim

email:    konstantin.kirchheim@ovgu.de

version:  1.0.0

language: de_DE

narrator:  Deutsch Female

-->

# Einführung

Willkommen zurück im eLearning-System *eLab*.

--{{1}}--
Während der Fokus der vorhergehenden Aufgaben auf den aktorischen Hardwarekomponenten unserer Roboterplattform lag, wollen wir in dieser Aufgabe, die ersten Sensoren in Betrieb nehmen.

--{{2}}--
Mit ihnen wird es uns möglich sein, bestimmte Eigenschaften der Umgebung unseres Roboters zu überwachen. Die erhaltenen Umgebungsinformationen können wir in der nächsten Aufgabe nutzen, um eine intelligente Bewegungsstrategie umzusetzen.

--{{3}}--
Bei den Sensoren, die wir in dieser Aufgabe ansteuern und auslesen werden, handelt es sich um zwei infrarot Distanzsensoren, sowie einer intertialen Messeinheit (engl. *inertial measurement unit (IMU)*) 

--{{4}}--
Mit Hilfe der Distanzsensoren können wir den Nah- und Fernbereich der Vorderseite unseres Roboters überwachen. Diese Informationen sind z.B. zur Implementierung einer Kollisionsvermeidung hilfreich. 

--{{5}}--
Mit den Informationen der IMU, die sich aus einem 3-Achsen-Gyroskp, 3-Achsen-Beschleunigungssensor, 3-Achsen-Kompass und einem Temperatursensor zusammensetzt, kann eine Überwachung der Positions- und Orientierungsänderungen unseres Roboters umgesetzt werden. Wir können also, mit einer gewissen Unsicherheit, diese Informationen nutzen um die aktuelle Position des Roboters relativ zu seiner intertialen Position zu bestimmen.

--{{5}}--
Bevor ihr diese *high-level* Informationen jedoch generieren könnt, implementiert ihr in dieser Aufgabe zunächst die Funktionalität zum Auslesen der Sensorwerte. Anschließend nutzt ihr die zuvor implementierte Ansteuerung der Motoren, um euren Roboter wiederkehrend in seine Ausgangsposition zu versetzen.

## Themen und Ziele

--{{1}}--
Entsprechend der bereits angesprochenen Sensorik (IR Distanzsensoren, IMU) bilden deren Messverfahren und speziellen Ausführungen ein Kernthema dieser Aufgabe.

--{{2}}--
Allerdings unterscheidet sich je nach Sensorik, dessen Ausführung und Anbindung an den Mikrocontroller das Vorgehen zum Auslesen aktueller Sensorwerte. 

--{{3}}--
Im Allgemeinen geben Sensoren zunächst analoge Spannungswerte aus, die mit dem Wert der gemessenen Umgebungsgröße funktional korrelieren. 

--{{4}}--
Diese müssen mit Hilfe eines Analog-Digital-Wandlers *(engl. analog-digital converter (ADC))* zunächst in digitale Spannungswerte umgesetzt werden. Dies kann, je nach Ausführung, sowohl innerhalb des Sensors implementiert sein, oder durch die Anwendung eines externen ADCs geschehen.

--{{5}}--
Anschließend an die Digitalisierung der Spannungswerte, müssen diese noch entsprechend der Sensorkennlinie in die eigentlich zu messende Umgebungsgröße umgerechnet werden. 

--{{6}}--
Mit Hilfe dieses Vorgehens und der angesprochenen Sensorik, ist das Ziel dieser Aufgabe, Entfernungen vor dem Roboter, sowie dessen Positions- und Orientierungsänderungen zu messen. Ausgewählte Werte sollen durch die 8-Segment-Anzeigen ausgegeben werden.

--{{7}}--
Im finalen Schritt bringt ihr eurem Roboter mit Hilfe der gewonnenen Sensorwerte bei, selbstständig zu seiner Ausgangsposition zurückzukehren.

**Themen:**

* Infrarot Distanzsensoren
* Inertiale Messeinheit (engl. *inertial measurement unit (IMU)*)
* Analog-Digital-Converter (ADC) zum umwandeln Analoger in digitale Signale 
* Interpretation von Spannungswerten anhand von Kennlinien 


**Ziel(e):**

* Messen von Entfernungen durch Infratorsensorik
* Messen von Positions- und Orientierungsänderungen
* Sensorbasierte Bewegungsstrategie


## Weitere Informationen

--{{1}}--
Vertiefend sind die Arbeitsweisen und genutzten physikalischen Prinzipien der einzelnen Sensoren interessant. 

--{{2}}--
Darüber hinaus sind auch die verschiedenen Varianten von Analog-Digital-Wandler von Interesse.

--{{3}}--
Um Daten von peripheren Hardwarkomponenten auf den Mikrocontroller zu übertragen, existieren neben der uns bereits bekannten "PIN-Methode", auch weitere, spezifizierte BUS-System und Protokolle. Hier seien Beispielhaft [I^2^C](https://en.wikipedia.org/wiki/I%C2%B2C) und [SPI](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus) genannt.

--{{3}}--
Letztlich findet ihr wie immer die Datenblätter und Dokumentation für diese Aufgabe unter dem Punkt **PKeS**.

**Sensoren:**

* [Entfernungssensor](https://en.wikipedia.org/wiki/Proximity_detector)
* [Gyroskope](http://sensorwiki.org/doku.php/sensors/gyroscope)
* [Beschleunigungssensoren](http://www.sensorwiki.org/doku.php/sensors/accelerometer)
* [Magnetometer](https://en.wikipedia.org/wiki/Magnetometer)

**ADC:**

* [Analog-Digital-Wandler](https://en.wikipedia.org/wiki/Analog-to-digital_converter)
* Allgemeine Erklärungen zur Analog-Digital-Wandler:
  
  !![ADC](https://www.youtube.com/embed/_gZjBx9cdro)<!--
    width: 560px;
    height: 315px;
  -->

**Serielle Bus-Protokolle:**

* [I^2^C](https://en.wikipedia.org/wiki/I%C2%B2C)
* [SPI](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus)

**Arduino-Projekt:**

* Ein Arduino-Projekt mit einem Sharp Distanzsensor:
  
  !![ArduinoSharp](https://www.youtube.com/embed/DTUUKM--PyQ)<!--
    width: 560px;
    height: 315px;
  -->


**PKeS:**

* [Datenblatt IMU (MPU-9255)](https://stanford.edu/class/ee267/misc/MPU-9255-Datasheet.pdf)
* [Datenblatt infrarot Distanzsensor Sharp GP2Y0A41SK0F](https://www.pololu.com/file/0J713/GP2Y0A41SK0F.pdf)
* [Datenblatt infrarot Distanzsensor Sharp GP2Y0A02YK0F](https://www.sparkfun.com/datasheets/Sensors/Infrared/gp2y0a02yk_e.pdf)
* [Datenblatt Motortreiber](http://www.st.com/content/ccc/resource/technical/document/datasheet/59/d0/ce/41/56/bb/4b/10/DM00034699.pdf/files/DM00034699.pdf/jcr:content/translations/en.DM00034699.pdf)
* [Schaltbelegungsplan](https://github.com/liaScript/PKeS0/blob/master/materials/robubot_stud.pdf?raw=true)
* [Arduinoview](https://github.com/fesselk/Arduinoview/blob/master/doc/Documetation.md)
* [Datenblatt des AVR ATmega32U4](http://www.atmel.com/Images/Atmel-7766-8-bit-AVR-ATmega16U4-32U4_Datasheet.pdf)


# Aufgabe 3

--{{1}}--
Zu Begin dieser Aufgabe, müssen die von den Sensoren gelieferten Werte zunächst ausgelesen und in den Mikrocontroller übertragen werden.

--{{2}}--
Wie bereits angedeutet, liefern einige Sensoren lediglich analoge Spannungswerte als Repräsentation der gemessenen Umgebungsinformation. Dies trifft auf die beiden Sharp-Distanzsensoren zu. Daher müssen in einem ersten Schritt die entsprechenden Analog-Digital-Wandler (einer für jeden Sensor) konfiguriert werden. So erhaltet ihr die digitalisierten Informationen der Spannungswerte.

--{{3}}--
Im Gegensatz zu den Distanzssenoren überliefert die IMU bereits digitalisierte Spannungswerte. Nichtsdestotrotz müssen die Eingänge des Mikrocontrollers entsprechend konfiguriert werden, um diese Werte auszulesen. Dies soll in der zweiten Teilaufgabe implementiert werden.

--{{4}}--
Nachdem sowohl die Sensorwerte der Distanzsensoren als auch der IMU nun auf den Mikrocontroller übertragen werden können, müssen die Werte nun noch in die korrekten Einheiten konvertiert werden. Diese Konvertierung soll in der dritten Teilaufgabe implementiert werden.

--{{5}}--
Letztlich sollen die gemessenen Sensorwerte nun noch durch das Arduinoview-Interface, wie auch durch die 8-Segment-Anzeigen dargestellt werden. Diese Darstellung sollt ihr in der letzten Teilaufgabe implementieren.

**Hinweis:**

**Verwendet bei der Bearbeitung der Aufgabe keine Funktionen aus der Arduino-Bibliothek. Lediglich die Funktionen der `Serial`-Klasse zur Ansteuerung der seriellen Schnittstelle, sowie der Funktion `millis()` zur einfachen Zeitmessung können genutzt werden.**


**Teilaufgaben**

* *3.1:* Konfiguration des ADC zum Auslesen analoger Sensorwerte
* *3.2:* Konfiguration des Mikrocontrollers zum Auslesen der IMU
* *3.3:* Umwandlung der digitalisierten Spannungswerte in die entsprechenden Modalitäten
* *3.4:* Visualisierung der Sensorwerte und Anzeige auf 8-Segment-Display.

Die Aufgabe ist bis zu der Woche vom **18.12. - 22.12.2017** vorzubereiten.


## Aufgabe 3.1


**Ziel:**

--{{1}}--
In der Ersten Teilaufgabe sollt ihr Zunächst den ADC des Microcontrollers Konfigurieren, um Werte von den analogen Eingängen zu lesen. In der zweiten Aufgabe sollen diese Werte interpretiert werden.


**Teilschritte:**

1. Recherche: 
   * Informiert euch grundlegend über ADCs
   * Lest das Kapitel ... im Handbuch des Microcontrollers um herrauszufinden, welche Register wie Konfiguriert werden müssen. 
2. Konfiguriert den ADC und beginnt, die Werte auszulesen. Ihr werdet 10-Bit Nutzdaten erhalten.


## Aufgabe 3.2

**Ziel:**

In dieser Teilaufgabe geht es darum, die ausgelesenen Werte entsprechend der Datenblätter zu interpretieren.
Ihr habt diese Aufgabe erfolgreich absolviert, wenn euer Roboter das folgene kann:

1. Rohe Sensorwerte auslesen, und in sinnvolle Werte umrechnen.
   * Einheit: cm
   * Genauigkeit: +- 2 cm
   * Reichweite lang: 40 - 70 cm
   * Reichweite kurz: 10 - 40 cm
2. Die aktuellen Werte auf dem 8-Segment-Display anzeigen. 

**Teilschritte:**

1. Recherche: informiert euch über die unterschiedlichen Sensortypen und ihre jeweiligen Kennlinien. 
2. Rechnet die erhaltenen 10-Bit-werte in eine konkrete Entfernung um. Dafür gibt es mehrere sinnvolle möglichkeiten, seid kreativ! Wenn ihr mit eurem Ergebnis zufrieden seid, könnte ihr es auf dem 8-Segment-Display ausgeben und testen. 


## Aufgabe 3.3

**Ziel:**

**Teilschritte:**

## Aufgabe 3.4

**Ziel:**

**Teilschritte:**

# Quizze

## Analog-Digital-Wandler

**Welche Vorteile hat ein Flash-ADC gegenüber einem Analog-Digital-Wandler, der nach dem Wägeverfahren arbeitet?**

  [[X]] Er benötigt keinen Digital-Analog-Wandler
  [[ ]] Er ist kostengünstiger
  [[X]] Er benötigt weniger Taktzyklen
  [[ ]] Er hat (im Allgemeinen) geringere Anschaffungskosten

**Welche Schaltung wird zur Umsetzung eines Analog-Digital-Wandler, der nach dem Wägeverfahren arbeitet, benötigt?**

  [( )] Eine H-Brücke
  [( )] Ein Schwingkreis
  [(X)] Eine Sample-And-Hold Schaltung
  [( )] Ein Gleichrichter


## Distanzsensoren

**Welche Vorgehen zur optischen Distanzmessung können unterschieden werden?**

[[X]] Time-of-Flight
[[X]] Laufzeitmessung
[[X]] Triangulation
[[ ]] Quadratur
[[X]] Phasenverschiebung
[[ ]] Parallelverschiebung

**Auf welchem physikalischem Zusammenhang basieren kapazitive Distanzsensoren?**

  [( )] $R = \frac{U}{I}$
  [(X)] $C = \epsilon \cdot \frac{A}{d}$
  [( )] $L = \frac{\Psi}{I}$
  
## Inertiale Messeinheit

**Aus welchen Sensoren besteht eine typische inertiale Messeinheit?**

  [( )] Temperatursensor, Distanzsensor
  [( )] Gyroskop, Beschleunigungssensor, Distanzsensor
  [(X)] Magnetometer, Gyroskop, Beschleunigungssensor, Temperatursensor
  [( )] Magnetometer, Beschleunigungssensor, Distanzsensor
  
**Wie viele *Degrees of freedom* hat die auf dem Roboter verbaute IMU (MPU-9255)?**

  [( )] 6
  [(X)] 9
  [( )] 10
  
**Welche Größen werden für eine Richtungsachse innerhalb der IMU (MPU-9255) gemessen?**

  [(X)] magnetsiche Flussdichte, Beschleunigung, Drehwinkelbeschleunigung
  [( )] magnetische Flussdichte, Geschwindigkeit, Drehwinkelbeschleunigung
  [( )] magnetische Flussdichte, Geschwindigkeit, Orientierung
  [( )] magnetische Feldstärle, Beschleunigung, Orientierung

