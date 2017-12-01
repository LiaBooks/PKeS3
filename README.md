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
Bei den Sensoren, die wir in dieser Aufgabe ansteuern und auslesen werden, handelt es sich um zwei infrarot Distanzsensoren, sowie eine intertiale Messeinheit (engl. *inertial measurement unit (IMU)*) 

--{{4}}--
Mit Hilfe der Distanzsensoren können wir den Nah- und Fernbereich der Vorderseite unseres Roboters überwachen. Diese Informationen sind z.B. zur Implementierung einer Kollisionsvermeidung hilfreich. 

--{{5}}--
Mit den Informationen der IMU, die sich aus einem 3-Achsen-Gyroskop, 3-Achsen-Beschleunigungssensor, 3-Achsen-Kompass und einem Temperatursensor zusammensetzt, kann eine Überwachung der Positions- und Orientierungsänderungen unseres Roboters umgesetzt werden. Wir können also, mit einer gewissen Unsicherheit, diese Informationen nutzen um die aktuelle Position des Roboters relativ zu seiner intertialen Position zu bestimmen.

--{{5}}--
Bevor ihr diese *high-level* Informationen jedoch generieren könnt, implementiert ihr in dieser Aufgabe zunächst die Funktionalität zum Auslesen der Sensorwerte. 

## Themen und Ziele

--{{1}}--
Entsprechend der bereits angesprochenen Sensorik (IR-Distanzsensoren, IMU) bilden deren Messverfahren und speziellen Ausführungen ein Kernthema dieser Aufgabe.

--{{2}}--
Allerdings unterscheidet sich je nach Sensorik, dessen Ausführung und Anbindung an den Mikrocontroller das Vorgehen zum Auslesen aktueller Sensorwerte. 

--{{3}}--
Im Allgemeinen geben Sensoren zunächst analoge Spannungswerte aus, die mit dem Wert der gemessenen Umgebungsgröße funktional korrelieren. Diese müssen mit Hilfe eines Analog-Digital-Wandlers *(engl. analog-digital converter (ADC))* zunächst in digitale Spannungswerte umgesetzt werden. In Abhängigkeit von der konkreten Ausführung des Sensors, kann dies bereits innerhalb des Sensors geschehen, oder muss durch einen externen ADC implementiert werden.

--{{4}}--
Anschließend an die Digitalisierung der Spannungswerte müssen diese noch entsprechend der Sensorkennlinie in die eigentlich zu messende Umgebungsgröße umgerechnet werden. 

--{{5}}--
Mit Hilfe dieses Vorgehens und der angesprochenen Sensorik, ist das Ziel dieser Aufgabe, Entfernungen vor dem Roboter, sowie dessen Positions- und Orientierungsänderungen zu messen. Ausgewählte Werte sollen durch die 8-Segment-Anzeigen ausgegeben werden. Letztlich sollen die Informationen so verarbeitet werden, dass ein Positionstracking des Roboters möglich ist.

**Themen:**

* Infrarot Distanzsensoren
* Inertiale Messeinheit (engl. *inertial measurement unit (IMU)*)
* Analog-Digital-Converter (ADC)
* Interpretation von Spannungswerten anhand von Kennlinien 


**Ziel(e):**

* Messen von Entfernungen durch Infrarot-Sensorik
* Messen von Positions- und Orientierungsänderungen
* Interpretation der Sensorwerte über die Zeit (Position, Orientierung)


## Weitere Informationen

--{{1}}--
Wie schon durch die Vorlesung bekannt, sind die physikalischen Prinzipien der einzelnen Sensoren interessant für diese Aufgabe.

--{{2}}--
Darüber hinaus sind auch die verschiedenen Varianten von Analog-Digital-Wandler von Interesse. Informiert euch auch darüber, welche Varianten uns der [AVR ATmega32U4](http://www.atmel.com/Images/Atmel-7766-8-bit-AVR-ATmega16U4-32U4_Datasheet.pdf) bietet.

--{{3}}--
Neben der von uns hauptsächlich genutzten Anbindung peripherer Hardwarkomponenten an die Mikrocontroller, existieren noch leistungsfähigere Bus-Systeme. Hier seien Beispielhaft [I2C](https://en.wikipedia.org/wiki/I%C2%B2C) und [SPI](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus) genannt. Interessant ist vor allem der *I^2^C*-Bus (alternative Schreibweise: *I2C*), da über eine solche Variante die IMU an unseren Mikrocontroller angeschlossen ist. **Hinweise: im Datenblatt des Mikrocontrollers wird hier auch von *TWI* gesprochen!**

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

* [I2C](https://en.wikipedia.org/wiki/I%C2%B2C)
* [SPI](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus)

**Arduino-Projekt:**

* Ein Arduino-Projekt mit einem Sharp Distanzsensor:
  
  !![ArduinoSharp](https://www.youtube.com/embed/DTUUKM--PyQ)<!--
    width: 560px;
    height: 315px;
  -->


**PKeS:**

* [Datenblatt IMU (MPU-9255)](https://stanford.edu/class/ee267/misc/MPU-9255-Datasheet.pdf)
* [Datenblatt IMU (MPU-9255) Register Map](https://stanford.edu/class/ee267/misc/MPU-9255-Register-Map.pdf)
* [Datenblatt infrarot Distanzsensor Sharp GP2Y0A41SK0F](http://sharp-world.com/products/device/lineup/data/pdf/datasheet/gp2y0a41sk_e.pdf)
* [Datenblatt infrarot Distanzsensor Sharp GP2Y0A02YK0F]( http://sharp-world.com/products/device/lineup/data/pdf/datasheet/gp2y0a02yk_e.pdf)
* [Datenblatt Motortreiber](http://www.st.com/content/ccc/resource/technical/document/datasheet/59/d0/ce/41/56/bb/4b/10/DM00034699.pdf/files/DM00034699.pdf/jcr:content/translations/en.DM00034699.pdf)
* [Schaltbelegungsplan](https://github.com/liaScript/PKeS0/blob/master/materials/robubot_stud.pdf?raw=true)
* [Arduinoview](https://github.com/fesselk/Arduinoview/blob/master/doc/Documetation.md)
* [Datenblatt des AVR ATmega32U4](http://www.atmel.com/Images/Atmel-7766-8-bit-AVR-ATmega16U4-32U4_Datasheet.pdf)


# Aufgabe 3

--{{1}}--
Zu Begin dieser Aufgabe müssen die von den Sensoren gelieferten Werte zunächst ausgelesen und in den Mikrocontroller übertragen werden.

--{{2}}--
Wie bereits angedeutet, liefern einige Sensoren lediglich analoge Spannungswerte als Repräsentation der gemessenen Umgebungsinformation. Dies trifft auf die beiden Sharp-Distanzsensoren zu. Daher müssen in einem ersten Schritt die entsprechenden Analog-Digital-Wandler (einer für jeden Sensor) konfiguriert werden. So erhaltet ihr die digitalisierten Informationen der Spannungswerte. Diese müssen dann für jeden der beiden Distanzsensoren individuell in Entfernungswerte konvertiert werden.

--{{3}}--
Im Gegensatz zu den Distanzssensoren überliefert die IMU bereits digitalisierte Spannungswerte. Allerdings ist die IMU über einen I^2^C-Bus angeschlossen, den ihr nutzen müsst, um die IMU entsprechend zu konfigurieren und aktuelle Sensorwerte auszulesen. Dies soll in der zweiten Teilaufgabe implementiert werden. Ähnlich zu den Distanzsensoren müssen auch hier die von dem Sensor gelieferten Werte konvertiert werden.

--{{4}}--
Letztlich sollen die gemessenen Sensorwerte nun noch durch das Arduinoview-Interface, wie auch durch die 8-Segment-Anzeigen, dargestellt werden. Diese Darstellung sollt ihr in der letzten Teilaufgabe implementieren. Darüber hinaus sollen die Informationen der IMU genutzt werden, um ein einfaches Positionstracking zu implementieren.

--{{5}}--
Im Gegensatz zu den bisherigen Aufgaben greifen wir diesesmal auf eure bisherigen Implementierungen zurück. Dies betrifft die Ansteuerung der 8-Segment-Anzeigen, sowie der Motoren. Während die 8-Segment-Anzeige direkt zur Bearbeitung der Aufgabe notwendig ist, könnt ihr die Motoren und den zur Ansteuerung bekannten *Joystick* nutzen, um den Roboter zu bewegen und so eure Sensorik zu testen. 

--{{6}}--
Damit ihr eure Lösungen aus den vorherigen Aufgaben weiter nutzen könnt, müsst ihr sie aus den entsprechenden Projekten kopieren. Wie ihr sicherlich bemerkt habt, befindet sich das eLab noch in einem jungen Stadium, sodass wir noch keine Gelegenheit hatten für einen automatisierten Austausch des Quellcodes zwischen Projekten/Aufgaben zu sorgen. Wir bitten das zu entschuldigen.


**Hinweis:**

**Verwendet bei der Bearbeitung der Aufgabe keine Funktionen aus der Arduino-Bibliothek. Lediglich die Funktionen der `Serial`-Klasse zur Ansteuerung der seriellen Schnittstelle, der Funktion `millis()` zur einfachen Zeitmessung, sowie der Funktionen aus `Wire.h` für die I^2^C Kommunikation können genutzt werden.**


**Teilaufgaben**

* *3.1:* Auslesen der Distanzsensoren und Konvertieren der Sensorwerte.
* *3.2:* Auslesen der IMU und Konvertieren der Sensorwerte.
* *3.4:* Visualisierung der Sensorwerte, Anzeige auf den 8-Segment-Anzeigen und Implementierung eines einfachen Positionstracking

Die Aufgabe ist bis zu der Woche vom **18.12. - 22.12.2017** vorzubereiten.


## Aufgabe 3.1

--{{1}}--
Das Ziel dieser Aufgabe ist es, die Distanzsensoren an der Vorderseite unserer Roboterplattform in Betrieb zu nehmen. Implementiert dazu die Funktionen in `Distance.{h,cpp}`.

--{{2}}--
Der erste Schritt dazu ist, den entsprechenden Analog-Digital-Konverter für die Verwendung mit den Distanzsensoren zu konfigurieren. Dies soll in der Funktion `configADC()` geschehen.

--{{3}}--
Damit das Auslesen der Sensorwerte zyklisch geschehen kann, implementiert ihr im nächsten Schritt die Funktion `readADC(uint8_t channel)`, die den aktuellen, digitalisierten Spannungswert des ausgewählten Sensors liefern soll.

--{{4}}--
Im letzten Schritt muss der digitalisierte Spannungswert entsprechend der Kennlinie des jeweiligen Distanzsensors in Milimeter konvertiert werden. Implementiert dazu die Funktionen `linearizeLR(uint16_t voltage)` und `linearizeSR(uint16_t voltage)` (wobei *LR* für *Long Range* und *SR* für *Short Range* steht). Ihr könnt dazu die entsprechenden Kennlinien aus den Datenblättern der Sensoren nutzen. Beachtet, dass wir die Distanzwerte in Millimeter (mm) darstellen wollen!

**Ziel:**
Implementiert das Auslesen der Sensorwerte beider Distanzsensoren und konvertiert die Spannungswerte in Distanzwerte in Millimeter (mm). 

**Teilschritte:**

1. Konfiguriert Analog-Digital-Wandler für die Verwendung mit den Distanzsensoren.
2. Implementiert Funktion `readADC(uint8_t channel)` zum Auslesen der Distanzsensoren (digitalisierte Spannungswerte).
3. Implementiert Funktionen `linearizeLR(uint16_t voltage)` und `linearizeSR(uint16_t voltage)` zur Transformation der Spannungswerte in Distanzwerte (mm) für beide Sensoren.

## Aufgabe 3.2

--{{1}}--
Nachdem wir nun die Ansteuerung der Distanzsensoren erfolgreich implementiert haben, wollen wir auch die Daten der inertialen Messeinheit auslesen und konvertieren.

--{{2}}--
Die IMU ist, im Gegensatz zu den Distanzsensoren, per I^2^C an den Mikrocontroller angeschlossen. Daher haben wir euch eine Implementierung zur Kommunikation mit der IMU vorgegeben. Um diese zu Initialisieren, müsst ihr zunächst `initializeIMU()` aufrufen.

--{{3}}--
Anschließend könnt ihr mit Hilfe der I^2^C-Kommunikation die aktuellen Sensorwerte aus der IMU auslesen. Dort sind vorallem die Beschleunigungs- und Drehraten für uns von interesse. Daher füllt die Funktion `readIMU(struct IMUdata& d)` die Struktur `IMUdata`.

--{{4}}--
Wie schon bei den Distanzsensoren erhaltet ihr von der IMU lediglich Werte die proportional zu den Beschleunigungen bzw. Drehraten sind. Daher müsst ihr auch diese Werte in $\frac{m}{s^2}$ bzw. $\frac{deg}{s}$ konvertieren. Implementiert dazu die Funktion `linearizeIMU(struct IMUdata& data)`.

**Ziel:**
Implementiert das Auslesen und Transformieren der Beschleunigungs- und  Drehratenwerte der IMU. Beschleunigungen sollen in $\frac{m}{s^2}$ und Drehraten in $\frac{deg}{s}$ repräsentiert werden.

**Teilschritte:**

1. Implementiert das Auslesen und Transformieren der Beschleunigungs- und  Drehratenwerte der IMU.

## Aufgabe 3.3

--{{1}}--
Da wir nun die Umgebung unseres Roboters mit Hilfe unserer Sensoren beobachten können, wollen wir die erhaltenen Werte natürlich noch entsprechend darstellen.

--{{2}}--
Daher besteht das Ziel dieser Aufgabe darin, die Sensorwerte mit Hilfe von Arduinoview zu visualisieren und ein einfaches Positionstracking zu implementieren.

--{{3}}--
Im ersten Schritt solltet ihr eurem Arduinoview ein Diagramm hinzufügen, in dem ihr die aktuellen Werte der Distanzsensoren visualisiert.

--{{4}}--
Äquivalent zum vorhergehen Schritt könnt ihr nun ein weiteres Diagramm dem Arduinoview-Interface hinzufügen, in dem ihr die Beschleunigungswerte der 3-Achsen der IMU darstellt.

--{{5}}--
Da uns die IMU auch Drehraten für die 3 Achsen bereitstellt, sollen auch diese Werte in einem separaten Diagramm dargestellt werden.

--{{6}}--
Da uns durch die Aufgabe 1 nun auch die 8-Segment-Anzeigen zur Darstellung von Zeichen zur Verfügung steht, wollen wir diese natürlich auch nutzen. Gebt daher jeweils den kleinsten der beiden Distanzwerte auf der Anzeige aus. Falls lediglich ein Distanzsensor valide Werte liefert, gebt ihr diesen aus. Falls beide Distanzsensoren keine validen Werte liefern, gebt ein '---' auf dem Display aus. 

--{{7}}--
Zuletzt möchten wir die Daten der IMU für ein einfaches Positionstracking nutzen. Lest dazu sowohl die Beschleunigungs- als auch die Drehraten der IMU über die gesamte Ausführungszeit aus und konvertiert sie so, dass ihr eine Positions- und Orientierungsangabe relativ zu der Startposition des Roboters erhaltet. Gebt letztlich die Position ($x~in~[mm]$ , $y~in~[mm]$) und Orientierung ($\theta~in~[^\circ]$) über das Arduinoview-Interface kontinuierlich aus.

**Ziel:**
Visualisiert die Sensordaten mit Diagrammen in Arduinoview und zeigt die Position ($x$,$y$) und Orientierung ($\theta$) des Roboters relativ zu seiner Startposition an.

**Teilschritte:**

1. Stellt die aktuellen Werte beider Distanzsensoren in einem Diagramm dar.
2. Stellt die aktuellen Beschleunigungswerte aller 3 Achsen in einem Diagramm dar.
3. Stellt die aktuellen Drehraten aller 3 Achsen in einem Diagramm dar.
4. Gebt einen validen Distanzwert auf den 8-Segment-Anzeigen aus

   * Falls die Distanzwerte beider Distanzsensoren valide sind, gebt den kleineren Wert aus.
   * Falls die lediglich ein Distanzwert valide ist, gebt diesen Wert aus.
   * Falls kein Distanzsensor einen validen Wert liefert, gebt '---' auf dem Display aus.
   
5. Konvertiert die Beschleunigungs- und Drehratenwerte in eine Roboterposition und -Orientierung, d.h. $x$,$y$,$\theta$.


# Quizze

--{{1}}--
Wie auch in der letzten Aufgabe haben wir noch ein paar kurze Fragen an euch.

## Analog-Digital-Wandler

**Welche Vorteile hat ein 8-Bit Flash-ADC gegenüber einem 8-Bit Analog-Digital-Wandler, der nach dem Wägeverfahren arbeitet?**

  [[X]] Er benötigt keinen Digital-Analog-Wandler
  [[ ]] Er benötigt weniger Komparatoren
  [[X]] Er benötigt weniger Taktzyklen
  [[ ]] Er hat (im Allgemeinen) geringere Anschaffungskosten

**Welche Schaltung wird zur Umsetzung eines Analog-Digital-Wandler benötigt?**

  [( )] Eine H-Brücke
  [( )] Ein Schwingkreis
  [(X)] Eine Sample-And-Hold Schaltung
  [( )] Ein Gleichrichter
  
**Wie viele Komparatoren benötigt ein 10-Bit Flash-ADC?**

  [( )] 1021
  [( )] 1022
  [(X)] 1023
  [( )] 1024


## Distanzsensoren

**Welche Vorgehen zur lichtbasieren Distanzmessung können unterschieden werden?**

  [[X]] Time-of-Flight
  [[X]] Laufzeitmessung
  [[X]] Triangulation
  [[ ]] Quadratur
  [[X]] Phasenverschiebung
  [[ ]] Parallelverschiebung

**Auf welchem Vorgehen zur Distanzmessung basiert der SharpGP2Y0A41SK0F?**

  [( )] Time-of-Flight
  [( )] Laufzeitmessung
  [(X)] Triangulation
  [( )] Quadratur
  [( )] Phasenverschiebung
  [( )] Parallelverschiebung
  
## Inertiale Messeinheit

**Aus welchen Sensoren besteht die IMU (MPU-9255)?**

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
  [( )] magnetische Feldstärke, Beschleunigung, Orientierung

