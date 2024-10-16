# Kommunikationsprotokoll für Schiffeversenken mit TCP/IP Sockets

Hochschule Hannover, Programmierenprojekt 3. Semester (BIN), Wintersemester 2024/25, Holitschke

Autoren:

- Florian Alexy
- Lucas Bronson
- Luca Conte
- Florian Hantzschel
- Joshua Kuklok
- Peer Ole Wachtel
- Marek Küthe

**Aktuelle Version: 1.1.0**

# Einführung

Im Rahmen des Programmierenprojekts des 3. Semesters im Studiengang Angewandte Informatik an der Hochschule Hannover, Wintersemester 2024/25, ist es die Aufgabe der Studierenden in unabhängigen Gruppen von in der Regel je sechs Studierenden, ein Spiel im Stil des klassischen Schiffeversenken zu planen und zu implementieren. Es soll möglich sein, über ein Netzwerk gegeneinander zu spielen. Hierbei ist es eine Vorgabe, dass diese Netzwerkfunktionen der Spiele der verschiedenen Gruppen untereinander kompatibel sind. Aufgrund dieser Vorgabe gibt es einen Bedarf für diesen Standard, um die unabhängige Entwicklung der Programme innerhalb der Gruppen zu ermöglichen und dennoch die Interoperabilität zu gewährleisten.

Die laufenden, von den Studierenden zu erstellenden Programme werden im Rahmen dieses Dokumentes "Instanzen" genannt.

## Anforderungsstufen

(Grobe Übersetzung von RFC2119)

Um die genauen Anforderungen und deren Bedeutung für die Funktion des Protokolls unmissverständlich zu definieren, werden im Folgenden gewisse Verben und Ausdrücke sowie deren Bedeutung im Rahmen dieses Dokumentes erläutert.

1. **Müssen**

   Dieses Verb sowie das Wort "erforderlich" bedeuten eine Definition, die für die Funktion des Protokolls unbedingt notwendig ist und deren Implementierung für die Unterstützung des Netzwerkaspekts des Projekts unumgänglich ist.
2. **Nicht dürfen**

   Dieser Ausdruck bedeutet ein absolutes Verbot der Spezifikation.
3. **Sollen**

   Dieses Verb sowie das Wort "empfohlen" bedeuten eine Definition, die nicht unmittelbar erforderlich ist, um die Funktion des Protokolls zu gewährleisten. Es kann Gründe geben (so wie Zeitmangel), aus denen diese Definition nicht implementiert wird. Jedoch sollte man die gesamten Auswirkungen des Auslassens dieser Definition verstehen und abwägen, bevor ein anderer Weg eingeschlagen wird.
4. **Nicht sollen**

   Dieser Ausdruck sowie der Ausdruck "nicht empfohlen" bedeuten, dass es Gründe geben kann, aus denen das beschriebene Verhalten akzeptabel oder sogar wünschenswert ist. Jedoch sollte man die gesamten Auswirkungen des Implementierungs dieses Verhaltens verstehen und abwägen, bevor irgendein Verhalten, das mit diesem Ausdruck beschrieben wird, implementiert wird.
5. **Können**

   Dieses Verb sowie das Wort "optional" oder der Ausdruck "nicht müssen" bedeuten eine Definition, die für die Funktion des Protokolls unentscheidend ist. Die Entscheidung, diese Definition zu implementieren, ist den Entscheidungsfällenden frei überlassen.

# Verbindungsaufbau

Um eine Verbindung zwischen zwei Instanzen zu ermöglichen, muss eine Instanz die Rolle eines Servers annehmen und die andere Instanz die Rolle eines Clients. Dies soll jeweils von den Nutzern bestimmt werden. Eine Auswahl der Server-Rolle kann beispielsweise durch eine Option "Online Spiel erstellen" erfolgen und die Auswahl der Client-Rolle durch eine Option "Online Spiel beitreten".

## Server

Wählt ein Nutzer die Server-Rolle aus, muss die Instanz beginnen, auf dem TCP Port 51525 auf Verbindungen zu warten, sofern vom Nutzer nicht explizit ein anderer Port definiert wurde. Die Änderung dieser Portnummer soll möglich sein, für den Fall, dass der oben genannte Standard-Port durch einen anderen Prozess blockiert ist.

Sobald sich ein Client verbunden hat, müssen keine weiteren Verbindungen mehr angenommen werden, bis die Verbindung zum Client wieder getrennt wurde.

## Client

Wählt ein Nutzer die Client-Rolle aus, so wird die Eingabe einer IP-Adresse benötigt. Für den Fall, dass die Server-Instanz, mit der sich die Client-Instanz verbinden soll, nicht den Standard-Port (TCP 51525) nutzt, soll es möglich sein, gegebenenfalls einen anderen Port auszuwählen. Die Instanz versucht daraufhin eine TCP Verbindung mit der angegebenen IP-Adresse und dem Port herzustellen.

# Protokoll

## Versionierung

Die Protokoll-Version besteht aus drei Integer Komponenten: Major, Minor und Patch, jeweils durch einen Punkt getrennt.

Beispiel:

```
1.0.3
^ ^ ^
| | |
| | +-- Patch
| +---- Minor
+------ Major
```

## Aufbau

Um das Protokoll möglichst platformunabhängig zu halten, wird (mit einigen Ausnahmen) Klartext bzw. ASCII für die Kommunikation verwendet. Ähnlich wie im SMTP Protokoll werden gewisse Schlüsselwörter verwendet, um verschiedene Pakete zu kennzeichnen. Pakete bestehen aus dem Schlüsselwort und gegebenenfalls einem Datensatz. Diese werden durch ein Leerzeichen (`0x20`) getrennt. Benötigt ein Schlüsselwort keinen Datensatz, fällt das Leerzeichen ebenfalls weg. Das Ende von Paketen wird durch einen Carriage Return und einen Zeilenumbruch gekennzeichnet (`\r\n`, `0x0d 0x0a`)


```
<Schlüsselwort> [Datensatz]\r\n
```

Ob ein Datensatz mitgesendet wird oder nicht, ist abhängig vom Schlüsselwort. Erhält eine Instanz ein Paket mit einem ungültigen, fehlenden oder überflüssigen Datensatz entgegen dieser Spezifikation, so soll dieses Paket ignoriert/verworfen werden.

## Vorstellung

Unmittelbar nach Verbindungsherstellung erfolgt die Vorstellung. Sie besteht aus einem Versionsabgleich und einem Austausch von Nutzernamen und Semestern.

### `VERSION` (erforderlich)

Das `VERSION` Paket dient dem Versionsabgleich. Der Datensatz enthält einen implementationsspezifischen String, zum Beispiel eine URL zur Implementierung, gefolgt von allen von der Instanz unterstützten Versionen des Protokolls in einer beliebigen Reihenfolge, jeweils getrennt durch ein Leerzeichen.

Beispiele:

```
VERSION https://example.com/GruppeXX/ProgProjekt 1.0.1
```
```
VERSION GruppeABC 1.0.1 1.2.5 1.0.3
```

Der implementierungsspezifische String darf nur aus ASCII Symbolen bestehen, keine Leerzeichen enthalten und eine Länge von mindestens einem Zeichen einhalten.

Die zu verwendende Version des Protokolls ist immer die höchste, welche von beiden Instanzen unterstützt wird. Die Bestimmung von hoch zu niedrig erfolgt hierbei zuerst nach Major, dann nach Minor und anschließend nach Patch Komponente der Version.

Gibt es keine Übereinstimmung der unterstützten Versionen, muss die Verbindung beendet werden.

### `IAM` (erforderlich)

Nachdem eine Verbindung hergestellt wurde und Versionen abgeglichen wurden, müssen beide Instanzen ein Paket mit dem Schlüsselwort `IAM`, ihrem aktuellen Semester, und einem vom Nutzer definierten Namen, der dann beim Gegner angezeigt werden soll, senden.

Syntax:

```
IAM <Semester> <Nutzername>
```

Beispiel:

```
IAM 2 Max Mustermann
```

Nachdem ein `IAM` Paket erhalten wurde, sollen alle weiteren `IAM` Pakete ignoriert werden.

Der im `IAM` Paket enthaltene Nutzername darf eine Länge von 32 Zeichen nicht überschreiten. Sollte doch ein `IAM` Paket mit einem Nutzernamen von mehr als 32 Zeichen erhalten werden, sollte der Name hinter dem 32. Zeichen abgeschnitten werden.

Das Semester, in dem gespielt wird, ist das kleinste Semester der beiden Spieler.

Das Semester, in dem gespielt wird, bestimmt eindeutig die Größe des Spielfelds.

| Semester | Größe   |
|----------|---------|
| 1        | 14 x 14 |
| 2        | 15 x 15 |
| 3        | 16 x 16 |
| 4        | 17 x 17 |
| 5        | 18 x 18 |
| 6        | 19 x 19 |

#### Server Discovery (optional)

Zusätzlich zu der Funktion im eigentlichen Protokoll kann das `IAM` Paket, wenn eine Instanz die Server-Rolle annimmt, auch als UDP Broadcast Nachricht ins Netzwerk geschickt werden, um Clients, die nach einem Server suchen, die Eingabe der IP-Adresse zu ersparen. Hierbei muss der gleiche Port als Ziel verwendet werden wie für das TCP Protokoll.

Zur Schonung der Bandbreite soll dieses Paket nicht häufiger als einmal alle 5 Sekunden gesendet werden.

Nimmt eine Instanz die Server-Rolle an, so kann sie gleichzeitig auch einen UDP Client starten, der das Paket broadcastet.

Nimmt eine Instanz die Client-Rolle an, so kann sie gleichzeitig auch einen UDP Server starten, der auf eingehende Pakete wartet, um die Server-Instanze(n) automatisch zu finden.

Wichtig: Für diese Funktion ist es notwendig, die Broadcast Adresse des Netzwerkes zu errechnen.

### `IAMU` (optional)

Das `IAMU` Paket ähnelt dem `IAM` Paket in der Funktion, mit dem entscheidenden Unterschied, dass das `IAMU` Paket das Semester nicht beinhaltet, und einen UTF-8 kodierten Nutzernamen überträgt, statt einen in ASCII kodierten Namen. Statt einer Maximallänge von 32 Zeichen darf der Nutzername im `IAMU` Paket eine Maximallänge von 32 Bytes nicht überschreiten.

Beispiel:

```
IAMU Günther
```

Hexadezimale Darstellung:

```
49 41 4d 55 20 47 c3 bc 6e 74 68 65 72 0d 0a
|  |  |  |  |  |  |---| |  |  |  |  |  |  |
I  A  M  U     G    Ü   N  T  H  E  R  \r \n
```

Wenn sowohl ein `IAM` als auch ein `IAMU` Paket erhalten werden, sollte das `IAMU` Paket, sofern es unterstützt wird, stets Vorrang in der Bestimmung des Nutzernamens haben.

## Spielstart

### `COIN` (erforderlich)

Nachdem die Spieler ihre Schiffe platziert haben, muss das `COIN` Paket gesendet werden. Beide Instanzen müssen genau einen zufälligen Bit (1 oder 0) generieren und müssen diesen (ASCII formatiert) im `COIN` Paket an den Spielpartner senden.

Beispiel:

```
COIN 0 
```

Um zu bestimmen, welcher der Spieler beginnt, werden die Ergebnisse beider Münzwürfe mit einer XOR Operation kombiniert. Ist das Resultat des XOR eine 1, so beginnt die Server-Instanz. Ist das Resultat eine 0, so beginnt die Client-Instanz.

Beispiel:

> Instanz A wirft 0  
> Instanz B wirft 1  
> -> Server beginnt, da 0 XOR 1 = 1

## Spielablauf

### `SHOOT` (erforderlich)

Das SHOOT Paket teilt dem Spielpartner mit, welches Feld der Spieler "beschossen" hat. Das Paket muss im Datensatz die Koordinaten des ausgewählten Feldes enthalten. Die erste Komponente der Koordinaten ist ein Buchstabe, welcher die Position auf der X-Achse beschreibt. Hierbei korrespondiert die Position des Buchstabens im Alphabet mit der X-Koordinate, an der sich das beschossene Feld befindet. Die zweite Komponente beschreibt die Position auf der Y-Achse durch eine Zahl. Die beiden Komponenten werden einfach konkatiniert.

Beispiel: C5 beschreibt das Feld in der 3. Spalte und in der 5. Zeile.

Merke: Die Koordinaten beginnen bei A bzw. 1. Array-Indizes fangen in der Regel jedoch bei 0 an.

Beispiel:

```
SHOOT F12
```

Als Antwort auf ein `SHOOT` Paket wird ein `HIT` Paket erwartet.

Wenn ein `SHOOT` Paket erhalten wird, während es nicht der Zug des Gegners ist, oder die Koordinaten im Datensatz des Paketes ungültig sind (z. B. außerhalb des Spielfeldes), soll dieses verworfen werden.

### `HIT` (erforderlich)

Das `HIT` Paket ist die Antwort auf jedes `SHOOT` Paket, unabhängig davon, ob es sich tatsächlich um einen Treffer handelt oder nicht.

Der Datensatz des `HIT` Paketes enthält zunächst die Koordinate, die beschossen wurde (wie bei `SHOOT`) und anschließend, mit einem Leerzeichen getrennt, eine Zahl, die angibt, ob es sich bei dem Schuss um einen Treffer handelte oder nicht.

- **0** - kein Treffer
- **1** - Treffer
- **2** - Versenkender Treffer
- **3** - Versenkender Treffer, Spiel gewonnen

Beispiel:

```
HIT F12 0
```

> Schuss F12 war kein Treffer

Erzielt ein Spieler einen Treffer (`HIT <Feld> 1`, bzw. `HIT <Feld> 2` für versenkenden Treffer), so darf kein Zugwechsel stattfinden. Der gleiche Spieler ist also erneut am Zug.

Hat ein Spieler alle Schiffe versenkt, muss als Antwort stattdessen `HIT <Feld> 3` gesendet werden, was somit das Ende des Spiels kennzeichnet.

### `CHAT` (optional)

Um den Spielern Kommunikation zu ermöglichen, können Instanzen Textnachrichten mittels des `CHAT` Paketes versenden.

`CHAT` Pakete enthalten als Datensatz die gesendete Nachricht als UTF-8 String. Zeilenumbrüche und Carriage Returns (`\n` und `\r`) dürfen nicht im Datensatz enthalten sein. Nachrichten dürfen außerdem nicht länger als 256 Zeichen sein. Nachrichten, die länger sind als 256 Zeichen, können in mehrere Nachrichten aufgeteilt werden.

Beispiel:

```
CHAT Gut gespielt!
```

## Spielende

Wird ein `HIT <Feld> 3` Paket gesendet oder empfangen, ist das Spiel zu Ende. Der Spieler, der alle Schiffe des Spielpartnes versenkt hat, steigt ein Semester auf.

Alternativ kann ein Spiel vorzeitig beendet werden, wenn ein Spieler aufgibt.

### `WITHDRAW` (erforderlich)

Das `WITHDRAW` Paket enthält keinen Datensatz. Es signalisiert, dass der Spieler, welcher dieses Paket sendet, die Spielpartie vorzeitig durch Aufgeben beendet. Dieses Paket zu empfangen, bedeutet einen sofortigen Sieg der Spielpartie und den Aufstieg in das nächste Semester.
