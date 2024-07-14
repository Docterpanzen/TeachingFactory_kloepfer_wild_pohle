# Teaching Factory

## Inhaltsverzeichnis

1. [Einleitung](#einleitung)
2. [Basisaufgabe I Vereinzelung](#basisaufgabe-i-vereinzelung)
3. [Basisaufgabe II Betriebszustandsampel](#basisaufgabe-ii-betriebszustandsampel)
4. [Basisaufgabe III Förderband](#basisaufgabe-iii-förderband)
    - [Motorsteuerung (Conveyer)](#motorsteuerung-conveyer)
    - [Förderband](#förderband)
5. [Basisaufgabe IV Extruder](#basisaufgabe-iv-extruder)
6. [Basisaufgabe V Manipulator-Achsen](#basisaufgabe-v-manipulator-achsen)
7. [Basisaufgabe VI Abfüllmodul](#basisaufgabe-vi-abfüllmodul)
8. [Basisaufgabe VII Manipulatormodul](#basisaufgabe-vii-manipulatormodul)

## Einleitung

Im Rahmen des Kurses Automatisierungstechnik wird ein Projekt durchgeführt, das die Implementierung und Dokumentation verschiedener Aufgabenstellungen in der Automatisierungstechnik umfasst. Ziel des Projekts ist es, praxisrelevante Kenntnisse und Fähigkeiten in der Programmierung, Steuerung und Fehleranalyse von Automatisierungsanlagen zu erwerben. Die Aufgaben reichen von der Implementierung einfacher Steuerungslogiken bis hin zur Auslegung komplexer Regelungssysteme und der Integration von IoT-Komponenten.

Dieser Bericht dokumentiert die schrittweise Umsetzung der Aufgabenstellungen. Es werden die Aufgabenstellung, die Herangehensweise sowie die Umsetzung einschließlich einiger UML-Diagramme beschrieben.

## Basisaufgabe I Vereinzelung

Die Vereinzelung ist der erste Arbeitsschritt im Abfüllprozess der Teaching Factory. Der Vereinzelungsprozess beginnt im Zustand "Off". In diesem Zustand sind beide Schleusen geschlossen. Wenn das bEnable-Signal gesetzt wird, wechselt der Prozess in den Zustand "Check". Hier wird überprüft, ob eine Flasche im Speicher vorhanden ist. Ist dies der Fall, wechselt der Prozess in den Zustand "Init".

Im Zustand "Init" wird die obere Schleuse geöffnet, um eine Flasche in die Kammer zu lassen. Sobald der Sensor in der Kammer eine Flasche erkennt, wird die obere Schleuse wieder geschlossen, und der Prozess wechselt in den Zustand "Init-wait", wo auf das Schließen der Schleuse gewartet wird.

Sobald der Timer abgelaufen ist, wechselt der Prozess in den Zustand "Bereit". In diesem Zustand sind beide Schleusen geschlossen, und der Prozess ist bereit für die nächste Aktion. Wenn das bExecute-Signal gesetzt ist und der Sensor am Auswurf keine Flasche erkennt, wechselt der Prozess in den Zustand "Auswerfen". Hier wird die untere Schleuse geöffnet, um die Flasche auszuwerfen. Sobald der Sensor in der Kammer keine Flasche mehr erkennt, wird die untere Schleuse wieder geschlossen, und der Prozess wechselt in den Zustand "Auswerfen-wait".

Im Zustand "Auswerfen-wait" wird erneut ein Timer gesetzt, um sicherzustellen, dass die Flasche vollständig ausgeworfen ist. Nach Ablauf des Timers wechselt der Prozess in den Zustand "Fertig". Hier sind beide Schleusen geschlossen, und der Prozess wartet auf das Zurücksetzen des bExecute-Signals, um wieder in den Zustand "Check" zu wechseln und den Zyklus zu wiederholen.

## Basisaufgabe II Betriebszustandsampel

Die Betriebszustandsampel zeigt den allgemeinen Anlagenzustand und ermöglicht es den Bedienern, den Betriebszustand der Anlage direkt zu erkennen. Dazu werden definierte Leuchtsignale verwendet, die gemäß der Norm EN ISO 13849-1 definiert sind. Da acht Betriebszustände durch drei Leuchtfarben dargestellt werden müssen, werden verschiedene Leucht- und Blinksignale sowie zweifarbige Signale eingesetzt.

Die Betriebszustandsampel umfasst die folgenden Zustände und Signale:

| Zustand    | Beschreibung |
|------------|--------------|
| **Error**      | Rotes Blinken signalisiert eine Sicherheitsüberschreitung oder einen Produktionsfehler. |
| **Stop**       | Rotes und gelbes Blinken zeigen an, dass die Anlage vom Bediener gestoppt wurde. |
| **Off**        | Rotes Leuchten zeigt einen nicht initialisierten Zustand ohne Sicherheitsüberschreitung an. |
| **Init**       | Grünes Blinken signalisiert die Initialisierung der Anlage. |
| **Production** | Grünes Leuchten zeigt an, dass die Anlage nach Plan produziert. |
| **Ready**      | Gelbes und grünes Blinken signalisiert, dass die Anlage auf ein Produktionsstartsignal durch den Bediener wartet. |
| **Done**       | Gelbes und grünes Leuchten zeigt an, dass der Produktionsprozess abgeschlossen ist und das Lager durch den Bediener geleert werden muss. |
| **Setup**      | Gelbes Blinken signalisiert, dass der Bediener die Anlage manuell einstellt. |

Zur Implementierung der Zustandsampel wird ein benutzerdefinierter Datentyp vom Typ Enumeration verwendet, der die Betriebszustände repräsentiert und als Eingang in den Funktionsbaustein eingebunden wird. Die Ausgänge des Funktionsbausteins werden entsprechend den definierten Leuchtsignalen über eine Case-Anweisung zugewiesen.

## Basisaufgabe III Förderband

Der Förderprozess in der Teaching Factory ist in zwei Hauptkomponenten unterteilt: die Motorsteuerung (Conveyer) und das Förderband. Die Motorsteuerung kümmert sich um den Antrieb und die Bewegungssteuerung, während das Förderband die Sensorik und den Ablauf des Bandes überwacht und steuert. Im Folgenden wird die Funktionsweise dieser Komponenten detailliert beschrieben.

### Motorsteuerung (Conveyer)

Die Motorsteuerung durchläuft verschiedene Zustände, um den Motor des Förderbandes zu steuern:

Im ausgeschalteten Zustand (State Off) sind alle Bewegungen gestoppt, die Achse des Motors ist stromlos, keine Bewegungen werden ausgeführt und der Halt ist aktiviert. Der Übergang zu "State Powering-up" erfolgt, wenn das System aktiviert wird.

Im Zustand "State Powering-up" wird die Achse des Motors aktiviert. Sobald die Achse bereit ist, wechselt der Zustand zu "State Ready". In diesem Zustand ist das System bereit, jedoch findet keine Bewegung statt. Der Übergang zu "State Production" erfolgt, wenn der Startbefehl gegeben wird.

Im Zustand "State Production" wird die Bewegung des Motors gestartet und der Halt deaktiviert. Dieser Zustand bleibt bestehen, bis der Stoppbefehl gegeben wird, woraufhin der Zustand wieder zu "State Ready" wechselt.

### Förderband

Das Förderband überwacht und steuert den gesamten Ablauf des Bandes sowie die Sensorik. Der Prozess beginnt im ausgeschalteten Zustand (State Off), in dem alle Stopper deaktiviert sind und die Motorsteuerung deaktiviert ist. Der Übergang zu "State Power-up" erfolgt, wenn das System aktiviert wird.

Im Zustand "State Power-up" wird die Motorsteuerung aktiviert und in den Zustand "Ready" versetzt. Sobald die Motorsteuerung bereit ist, wechselt der Zustand des Förderbandes zu "State Ready". In diesem Zustand sind alle Stopper weiterhin deaktiviert und das System ist bereit für den nächsten Schritt.

Der nächste Schritt erfolgt im Zustand "State Move-to-next", in dem das Förderband in Bewegung gesetzt wird. Die Position der Flasche wird durch die Sensoren überwacht. Je nach Position der Flasche und den Sensorwerten wird der Übergang zu "State Wait" vollzogen, wo ein Timer aktiviert wird.

Nach Ablauf des Timers wechselt der Zustand zu "State Done". In diesem Zustand wird die Bewegung gestoppt und die aktuelle Position der Flasche aktualisiert. Schließlich wechselt der Zustand zu "State wait-for-next", wo das System auf den nächsten Startbefehl wartet.

## Basisaufgabe IV Extruder

Der Extruder besteht aus wesentlichen Komponenten wie dem Granulatspeicher, der Extruderschnecke, dem Extrudermotor und den Sensoren zur Überwachung des Extrusionsprozesses. Der Prozess beginnt, indem das Granulat aus dem Speicher in die Extruderschnecke geleitet wird. Dabei wird die Schnecke durch den Extrudermotor angetrieben, um das Granulat zu fördern und zu extrudieren.

Der Ablauf des Extrusionsprozesses lässt sich in verschiedene Zustände unterteilen, die im zugehörigen Ablaufdiagramm des Funktionsblocks dargestellt sind.

## Basisaufgabe V Manipulator-Achsen

Die Manipulator-Achsen sind ein zentraler Bestandteil der Automatisierungstechnik. Jede Achse wird durch einen Motor angetrieben und überwacht, um präzise Bewegungen in verschiedene Richtungen zu ermöglichen. Der Funktionsblock zur Steuerung der Manipulator-Achsen umfasst die Initialisierung, bei der die Achsen in eine definierte Ausgangsposition gebracht werden, und die Referenzierungsfahrt, bei der die Achsen zu den Endlagensensoren fahren, um ihre Position zu bestätigen. Die Steuerung erfolgt durch spezifische Befehle, die die Richtung und Geschwindigkeit der Bewegung bestimmen, und der aktuelle Status der Achse wird über den Ausgang überwacht. Diese Struktur gewährleistet eine hohe Präzision und Zuverlässigkeit in der Handhabung und Positionierung.

## Basisaufgabe VI Abfüllmodul

Das Abfüllmodul in der Teaching Factory übernimmt die wesentliche Aufgabe der Befüllung von Flaschen. Diese Implementierung erfordert die Integration verschiedener mechanischer Komponenten und die Steuerung dieser durch entsprechende Funktionsblöcke. Im Folgenden wird der Aufbau und die Funktionsweise des Abfüllmoduls detailliert beschrieben.

Das Abfüllmodul besteht aus mehreren Schlüsselkomponenten: Granulatspeicher, Extruderschnecke, Extrudermotor, Ultraschallsensor und elektromagnetische Hubzylinder mit Federrückstellung. Der Prozess startet, indem die Flasche über ein Förderband unter den Dispenser bewegt wird. Hierbei kommen Näherungssensoren zum Einsatz, um die exakte Position der Flasche zu ermitteln und zu gewährleisten, dass sie sich unter der Befüllstation befindet.

Der Ablauf des Abfüllprozesses lässt sich in mehrere Zustände unterteilen, die im zugehörigen Ablaufdiagramm des Funktionsblocks dargestellt sind:

Im ausgeschalteten Zustand (State Off) sind alle Bewegungen und Aktionen gestoppt. Die Achse des Dispensers ist stromlos, keine Bewegungen werden ausgeführt, der Halt ist aktiviert und die Abfüllung ist nicht abgeschlossen. Der Übergang zu "State Powering-up" erfolgt, wenn das System aktiviert wird.

In "State Powering-up" wird die Achse des Dispensers aktiviert. Sobald die Achse bereit ist, wechselt der Zustand zu "State Ready". In diesem Zustand befindet sich das System in Bereitschaft, ohne Bewegung oder Haltebefehl. Der Übergang zu "State Production" erfolgt, wenn der Startbefehl gegeben wird.

Im Zustand "State Production" beginnt der Dispenser mit der Bewegung und der Halt ist deaktiviert. Dieser Zustand hält an, bis der Abfüllvorgang abgeschlossen ist, woraufhin der Zustand zu "State Done" wechselt.

Nach Abschluss der Befüllung wird die Bewegung gestoppt und der Halt aktiviert. Der Done-Timer wird aktiviert, um einen kurzen Zeitraum zu warten. Nach Ablauf dieses Timers wird der Zustand wieder auf "State Ready" zurückgesetzt, um für den nächsten Befüllvorgang bereit zu sein.

## Basisaufgabe VII Manipulatormodul

Das Manipulatormodul in der Produktionsanlage spielt eine entscheidende Rolle bei der präzisen Handhabung und Positionierung von den Flaschen. Es besteht aus vier Achsen, die durch Endlagensensoren überwacht werden. Diese Achsen sind in einem Robotermodul zusammengefasst, das die Bewegungen des Manipulators steuert. Parallel zum Robotersubmodul werden die Waageneinheit und die Temperierungseinheit von der Graphengruppe des Moduls koordiniert, was eine gleichzeitige und synchronisierte Steuerung verschiedener Funktionseinheiten ermöglicht.

Die Programmstruktur des Manipulatormoduls ist hierarchisch organisiert. Die vier Achsen des Manipulators sind in X-, Y- und Z-Koordinaten sowie Greiferaktionen spezifiziert, die an das Submodul übergeben werden. Dieses Submodul bewegt den Roboter an die gewünschten Positionen und führt die entsprechenden Greiferaktionen durch. Die Koordination dieser Bewegungen erfolgt durch ein Steuerprogramm, das die Zustandsübergänge und die entsprechenden Aktionen definiert.

Der Prozess beginnt mit der Initialisierung, bei der die Achsen des Manipulators in eine definierte Ausgangsposition gebracht werden. Anschließend erfolgt die Referenzierungsfahrt, bei der die Achsen zu den Endlagensensoren fahren, um ihre Position zu bestätigen. Diese Schritte sind entscheidend, um sicherzustellen, dass die Bewegungen des Manipulators präzise und wiederholbar sind.

Ein beispielhafter Ablauf des Robotermoduls zeigt, wie Positionsangaben und Greiferaktionen koordiniert werden. Zunächst werden die X-, Y- und Z-Koordinaten sowie die gewünschten Greiferaktionen an das Submodul übergeben. Das Submodul bewegt den Roboter an die spezifizierte Position und führt die Greiferaktion durch, wie z.B. das Aufnehmen oder Platzieren einer Flasche.
