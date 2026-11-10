# Microtris2090

Tetris für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4 Tastern.

- 248 Programmschritte, 8 frei
- Datei: [microtris.mic](microtris.mic) (Version 09)

## Spiel

- **Schacht:** Spalte 1-4, Reihe 1-8. Eine Reihe ist genau ein Register, das ganze Feld liegt in R0-R7.
- **Steine:** Formen im 2x2-Kasten: Ecke (1/2), Zweier (1/4), Quadrat (1/4), zufällig gedreht.
- **Vorschau:** Der nächste Stein steht rechts in Spalte 6-7, Reihe 7-8. Er fährt dann in Spalte 2-3 von oben ein.
- **Reihen:** Volle Reihen verschwinden.
- **Spielende:** wenn nach dem Landen etwas in Reihe 8 liegt. Die Anzeige zeigt die gelöschten Reihen dezimal.
- **Tempo:** Es steigt mit den gelöschten Reihen. Eine Reihe tiefer dauert 2,5 s bei 0-3 Reihen und sinkt bis 1,0 s ab 16 Reihen.

## Bedienung

| Taster | Pfeiltaste im Emulator | Wirkung |
|---|---|---|
| Eingang 1 | links | Stein nach links |
| Eingang 2 | hoch | rechts drehen |
| Eingang 3 | runter | links drehen |
| Eingang 4 | rechts | Stein nach rechts |

Nur neu gedrückte Taster zählen: ein Druck ist ein Schritt.

## Start

RESET (alle Register 0, das Programm enthält kein CLEAR), Matrix einschalten (siehe unten), `HALT` `NEXT` `00` `RUN`. Neues Spiel: RESET.

## Technik: das Feld und der Stein

Das ganze Spielfeld liegt in R0-R7: eine Reihe ist ein Register, Bit 1 ist die linke Spalte. Der fallende Stein ist **in das Feld eingezeichnet**, es gibt keinen getrennten Bildspeicher. Die Matrix bekommt R0-R7 als linke Hälften der Reihen 1-8, in den rechten Hälften stehen nur die Vorschau (Reihe 7-8) und sonst Nullen.

R8-RF gibt es zweimal. Zwei Kontexte teilen sich diese Register, der jeweils andere liegt in M8-MF und wird mit `EXRM` getauscht:

| Register | Kontext A (Stein) | Kontext B (Spiel) |
|---|---|---|
| R8 | FORM: Form im 2x2-Kasten | TASTE: Taster der letzten Abfrage |
| R9 | SPALTE: Kasten-Spalte 0-4 | ZAEHL: Fallzähler |
| RA | ZEILE: obere Kastenzeile | HILF: Hilfsregister |
| RB | UNTEN: Maske untere Zeile | VOR_U: Vorschau unten |
| RC | OBEN: Maske obere Zeile | VOR_O: Vorschau oben |
| RD | MASKE: Hilfsregister | REIHEN_L: gelöschte Reihen, Einer |
| RE | ALT: Hilfsregister | REIHEN_H: gelöschte Reihen, Sechzehner |
| RF | AKT: Aktion (in beiden Kontexten) | AKT |

Die Aktion AKT liegt in RF und wird bei jedem Wechsel mitgenommen: 0 hoch, 1 links, 2 rechts drehen, 3 links drehen, 4 rechts, 5 fallen.

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) im Blockmodus, die Matrix schläft zwischen den Bildern:

1. **Wecken:** Das Programm legt F an und hält es, bis der ESP mit REQ antwortet.
2. **Bild:** Dann sendet es direkt ACK 1, genau einen Befehl, die 16 Register und zum Schluss 0.
3. **Schlafen:** Danach schläft der ESP von selbst wieder ein. Er setzt kein REQ und beachtet die Ausgänge nicht, so ist Eingang 1 frei für Taster links.

Weil das F stehen bleibt, bis der ESP antwortet, geht kein Wecken verloren, auch wenn er gerade mit der Weboberfläche beschäftigt ist.

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP). Das Programm braucht die Firmware mit „ein Befehl je Wecken“ (esp2090-firmware ab PR #38).

## Wie es gemacht ist

Der 2090 hat keine indirekte Adressierung, keine Bitbefehle und nur ein Rücksprungregister. Dass Tetris trotzdem in 248 Schritte passt, liegt an ein paar Kniffen.

### Eine Routine für alle Bewegungen

Jede Aktion läuft nach demselben Muster ab:

1. **Stein löschen:** `Neg` negiert die Zeilenmasken (INV, +1 = 0 − Maske), `Acc` addiert sie zum Feld. Der Stein verschwindet.
2. **Aktion ausführen:** `Mod` ändert nur FORM, SPALTE oder ZEILE.
3. **Masken neu bauen:** `Masken` rechnet FORM und SPALTE in zwei Zeilenmasken um.
4. **Stein setzen:** `Acc` addiert die Masken und liefert die alten Zeilen zurück. Ist (alte Zeile UND Maske) nicht 0, überlappt der Stein etwas. Dann wird er wieder gelöscht, und die **Gegenaktion** 5 − AKT ausgeführt: links wird rechts (1 ↔ 4), rechts drehen wird links drehen (2 ↔ 3), fallen wird hoch (5 ↔ 0).

Eine Kollision mit Rand, Boden oder anderen Steinen ist also immer dasselbe: Rückgängig machen. Das Programm unterscheidet nicht, was im Weg war.

### Addieren statt Setzen und Löschen

Weil sich zwei Steine nie überlappen dürfen, können die Bits eines Steins addiert und subtrahiert werden, und es entsteht kein Übertrag in die Nachbarspalte. Setzen und Löschen sind dieselbe Addition, nur mit der Maske oder ihrem Negativ. Dasselbe Prinzip nutzt auch Mole2090 für seine Maulwürfe.

### Landen: die Gegenaktion zum Fallen

Fällt der Stein nicht mehr, wird das Fallen zurückgenommen, also Aktion 0 (hoch). Das Programm erkennt den Stein als gelandet, wenn AKT nach dem Zug 0 ist. Das ist ein einziges `MOV AKT,AKT` und der Zero-Flag. Der Boden ist keine eigene Prüfung, sondern eine volle Zeile: Ragt die untere Kastenzeile unter Reihe 1, setzt `Acc` dafür die Maske F ein. Sie überlappt mit jedem Stein, also ist der Boden ein Hindernis wie jedes andere.

### Rand durch den Übertrag

`Masken` schiebt die Zeilenmasken von der linken Kastenspalte nach rechts, soweit SPALTE es verlangt. Dabei steht die Vereinigung beider Zeilen in MASKE. Fällt aus ihr ein Bit heraus (Carry), ragt der Stein über den linken oder rechten Rand. Dann sagt `Masken` das mit dem Carry, und `Rand` macht den Zug wie jede Kollision rückgängig.

### Formen und Drehen im 2x2-Kasten

Jede Form liegt in einem 2x2-Kasten, 4 Bit: Bit 1 oben links, 2 oben rechts, 4 unten rechts, 8 unten links. Drehen um 90° ist dann ein Rotieren dieser 4 Bit: `SHL FORM` und `ADC FORM` holt das herausgeschobene Bit auf Platz 1. Links drehen sind drei solche Schritte.

Die Form wird aus einem einzigen Würfelwurf gewählt: Das erste `SHR` auf den Würfel entscheidet mit dem Carry über die Ecke (7, Wahrscheinlichkeit 1/2), das zweite über den Zweier (3, 1/4), sonst bleibt das Quadrat (F, 1/4). Eine zweite Ziffer bestimmt, wie oft die Form gedreht wird (0-3).

### Volle Reihen löschen, ohne Index

Das Programm prüft R0-R7 der Reihe nach auf F (`CMPI`). Findet es eine volle Reihe, springt es in eine Kette `Weg0` ... `Weg6`: Jede Zeile der Kette ist ein `MOV r(n+1),r(n)`. Das Einspringen an der richtigen Stelle schiebt alles darüber um eine Reihe nach unten, und danach läuft die Kette in `Weg7` (oberste Reihe leeren, zählen) aus. Ein Sprung pro Reihe ersetzt eine Schleife mit Index. Danach prüft das Programm von vorn, ob noch eine Reihe voll ist.

Der Zähler für die gelöschten Reihen hat zwei Hex-Ziffern: `ADDI 1` und `ADC` schieben den Übertrag in die obere Ziffer. Am Ende rechnet `HXDZ` ihn nach dezimal.

### Taster ohne Dauerfeuer

Gezählt wird nur ein neu gedrückter Taster: Aus dem alten und dem neuen Zustand rechnet das Programm (nicht alt) UND neu, so ist ein gehaltener Taster nach dem ersten Schritt still. Der niedrigste Taster gewinnt, seine Nummer ergibt sich durch Schieben und Zählen.

Die Taster werden **innerhalb des Bildes** gelesen. Zwischen ACK und dem ersten Wert verlangt das Protokoll genau einen Befehl. Dort steht `DIN`, das den Zustand der Taster liefert: Der Befehl, der ohnehin gebraucht wird, nimmt die Abfrage mit.

### Tempo aus den gelöschten Reihen

Der Stein fällt nicht bei jedem Durchlauf. Ein Fallzähler zählt rückwärts, der Unterlauf (Carry) löst das Fallen aus. Er wird neu geladen mit 5 − Reihen/4: Zwei `SHR` teilen durch 4, `INV` und `SUBI #A` machen daraus 5 − x. Ab 16 Reihen, wenn die obere Ziffer nicht mehr 0 ist, fällt der Stein bei jedem Durchlauf.

### Vorschau und Kontextwechsel

Die Vorschau liegt als zwei Zeilen im Kontext B. Beim Wechsel zum nächsten Stein wird die alte Vorschau der aktuelle Stein, und die neue Vorschau wird gewürfelt. Registerwerte, die in den anderen Kontext müssen (Aktion, Vorschauzeilen), trägt `MAS` hinüber. Beim Spielbeginn ist die Vorschau leer, daran erkennt das Programm den Start (`Neu0`).

Spielende ist, wenn nach dem Landen etwas in Reihe 8 liegt: `MOV r7,r7` und der Zero-Flag, danach `HXDZ` und `DISP 3`.
