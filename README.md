# Microtris2090

Tetris für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4 Tastern.

- 248 Programmschritte, 8 frei
- Datei: [microtris.mic](microtris.mic)

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

## Geschichte

- **Alexei Paschitnow** (Alexey Pajitnov) schrieb *Tetris* 1984 am Rechenzentrum Dorodnizyn der Akademie der Wissenschaften der Sowjetunion in Moskau. Er hat es nach eigener Aussage aus Spaß gemacht, nicht um Geld zu verdienen.
- Die Idee kam von einem Puzzle mit hölzernen **Pentominos**, Figuren aus fünf Quadraten. Paschitnow nahm Figuren aus vier Quadraten, damit die Auswahl überschaubar bleibt. Der Name verbindet griechisch *tetra* (vier) mit *Tennis*, seinem Lieblingssport.
- Die Rechte lagen bei der staatlichen Stelle *Elorg*, die sowjetische Software im Ausland vermarktete. Lizenzen wurden an westliche Firmen vergeben, teils ohne Wissen von Elorg. Es kam zum Rechtsstreit zwischen Mirrorsoft (Lizenz für Spielhalle und Konsole an Atari und Sega) und Nintendo, den Nintendo gewann.
- **Henk Rogers** (BulletProof Software) reiste dazu nach Moskau und überzeugte die Beamten, die Handheld-Rechte an Nintendo zu geben. Nintendo legte *Tetris* jedem **Game Boy** bei.
- Paschitnow selbst blieb zunächst außen vor und bekam nach Berichten rund 40 Millionen Dollar nicht. 1996, als Elorg aufgelöst wurde, gab Rogers die Rechte an ihn zurück.
- Das Microtronic-Programm vereinfacht stark: Es kennt nur drei Formen (Ecke, Zweier, Quadrat) im 2x2-Kasten statt der sieben Tetrominos. Das Spielprinzip ist dasselbe.

Quelle: LiveScience „The Bizarre History of Tetris“. Weitere Details, etwa die Mitarbeit weiterer Programmierer, habe ich nicht geprüft.

## Technik: das Feld und der Stein

**Schreibweise in diesem README:** Ein Register heißt `R0` bis `RF` (Speicher: `M0` bis `MF`), ein Wert beginnt mit `#`, zum Beispiel `#3` oder `#F`. `RF` ist also ein Register, `#F` die Zahl fünfzehn. In den Programmzeilen stehen wie im Programm Namen statt Nummern (`AKT` = `RF`), Register `r0`-`r7` klein, Werte mit `#`.

Das ganze Spielfeld liegt in `R0`-`R7`: eine Reihe ist ein Register, Bit `#1` ist die linke Spalte. Der fallende Stein ist **in das Feld eingezeichnet**, es gibt keinen getrennten Bildspeicher. Die Matrix bekommt `R0`-`R7` als linke Hälften der Reihen 1-8, in den rechten Hälften stehen nur die Vorschau (Reihe 7-8) und sonst Nullen.

`R8`-`RF` gibt es zweimal. Zwei Kontexte teilen sich diese Register, der jeweils andere liegt in `M8`-`MF` und wird mit `EXRM` getauscht:

| Register | Kontext A (Stein) | Kontext B (Spiel) |
|---|---|---|
| `R8` | FORM: Form im 2x2-Kasten | TASTE: Taster der letzten Abfrage |
| `R9` | SPALTE: Kasten-Spalte `#0`-`#4` | ZAEHL: Fallzähler |
| `RA` | ZEILE: obere Kastenzeile | HILF: Hilfsregister |
| `RB` | UNTEN: Maske untere Zeile | VOR_U: Vorschau unten |
| `RC` | OBEN: Maske obere Zeile | VOR_O: Vorschau oben |
| `RD` | MASKE: Hilfsregister | REIHEN_L: gelöschte Reihen, Einer |
| `RE` | ALT: Hilfsregister | REIHEN_H: gelöschte Reihen, Sechzehner |
| `RF` | AKT: Aktion (in beiden Kontexten) | AKT |

Die Aktion `AKT` liegt in `RF` und wird bei jedem Wechsel mitgenommen. Sie ist ein Wert: `#0` hoch, `#1` links, `#2` rechts drehen, `#3` links drehen, `#4` rechts, `#5` fallen.

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) im Blockmodus, die Matrix schläft zwischen den Bildern:

1. **Wecken:** Das Programm legt den Wert `#F` an und hält ihn, bis der ESP mit REQ antwortet.
2. **Bild:** Dann sendet es direkt ACK 1, genau einen Befehl, die 16 Register und zum Schluss 0.
3. **Schlafen:** Danach schläft der ESP von selbst wieder ein. Er setzt kein REQ und beachtet die Ausgänge nicht, so ist Eingang 1 frei für Taster links.

Weil der Wert `#F` stehen bleibt, bis der ESP antwortet, geht kein Wecken verloren, auch wenn er gerade mit der Weboberfläche beschäftigt ist.

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP).

## Wie es gemacht ist

Der 2090 hat keine indirekte Adressierung, keine Bitbefehle und nur ein Rücksprungregister. Dass Tetris trotzdem in 248 Schritte passt, liegt an ein paar Kniffen. Schreibweise wie oben: `RF` ist ein Register, `#F` ein Wert.

### Eine Routine für alle Bewegungen

Jede Aktion läuft nach demselben Muster ab:

1. **Stein löschen:** `Neg` negiert die Zeilenmasken (`INV`, `+1` = 0 − Maske), `Acc` addiert sie zum Feld. Der Stein verschwindet.
2. **Aktion ausführen:** `Mod` ändert nur `FORM`, `SPALTE` oder `ZEILE`.
3. **Masken neu bauen:** `Masken` rechnet `FORM` und `SPALTE` in zwei Zeilenmasken um.
4. **Stein setzen:** `Acc` addiert die Masken und liefert die alten Zeilen zurück. Ist (alte Zeile UND Maske) nicht `#0`, überlappt der Stein etwas. Dann wird er wieder gelöscht, und die **Gegenaktion** `#5` − `AKT` ausgeführt: links wird rechts (`#1` ↔ `#4`), rechts drehen wird links drehen (`#2` ↔ `#3`), fallen wird hoch (`#5` ↔ `#0`).

```
Zug:    CALL Neg        ; 1. Stein löschen: minus Masken addieren
        CALL Acc
ZugM:   CALL Mod        ; 2. Aktion ausführen (ändert FORM, SPALTE oder ZEILE)
        CALL Masken     ; 3. Masken neu bauen
        BRC  Rand       ;    Carry: über den Rand
Setzen: CALL Acc        ; 4. Stein addieren; MASKE, ALT = alte Zeilen
        AND  UNTEN,MASKE   ; Treffer = alte Zeile UND Maske
        AND  OBEN,ALT
        OR   MASKE,ALT
        BRZ  Fertig     ; kein Treffer: fertig
Kolli:  CALL Neg        ; Treffer: Stein wieder löschen ...
        CALL Acc
Rand:   INV  AKT        ; ... Gegenaktion: AKT = #F - AKT
        SUBI #A,AKT     ;                  AKT = AKT - #A  =  #5 - alte AKT
        GOTO ZugM       ; die Gegenaktion ausführen
```

Eine Kollision mit Rand, Boden oder anderen Steinen ist also immer dasselbe: Rückgängig machen. Das Programm unterscheidet nicht, was im Weg war.

### Addieren statt Setzen und Löschen

Weil sich zwei Steine nie überlappen dürfen, können die Bits eines Steins addiert und subtrahiert werden, und es entsteht kein Übertrag in die Nachbarspalte. Setzen und Löschen sind dieselbe Addition, nur mit der Maske oder ihrem Negativ. Dasselbe Prinzip nutzt auch Mole2090 für seine Maulwürfe.

```
Neg:  INV  UNTEN        ; UNTEN = #F - UNTEN
      ADDI #1,UNTEN     ; UNTEN = 0 - UNTEN (Negativ modulo 16)
      INV  OBEN
      ADDI #1,OBEN
      RET
Acc:  ...               ; addiert UNTEN und OBEN zu den beiden Feldregistern
A3:   MOV  r2,MASKE     ; Beispiel ZEILE = 3: alte untere Zeile merken ...
      ADD  UNTEN,r2     ; ... r2 plus untere Maske
      MOV  r3,ALT       ; alte obere Zeile merken
      ADD  OBEN,r3      ; r3 plus obere Maske
```

### Landen: die Gegenaktion zum Fallen

Fällt der Stein nicht mehr, wird das Fallen zurückgenommen, also Aktion `#0` (hoch). Das Programm erkennt den Stein als gelandet, wenn `AKT` nach dem Zug `#0` ist. Das ist ein einziges `MOV AKT,AKT` und der Zero-Flag. Der Boden ist keine eigene Prüfung, sondern eine volle Zeile: Ragt die untere Kastenzeile unter Reihe 1, setzt `Acc` dafür die Maske `#F` ein. Sie überlappt mit jedem Stein, also ist der Boden ein Hindernis wie jedes andere.

```
Fertig: MAS  AKT        ; AKT in den Kontext B hinüberretten
        EXRM            ; Kontext B
        MOV  AKT,AKT    ; Zero, wenn AKT = #0: das Fallen wurde zurückgenommen
        BRZ  Voll       ; gelandet: volle Reihen suchen
```

Der Boden in `Acc`, wenn die untere Kastenzeile unter Reihe 1 liegt:

```
A0:     MOVI #F,MASKE   ; „alte“ untere Zeile = #F: trifft jeden Stein
        MOV  r0,ALT
        ADD  OBEN,r0    ; nur die obere Zeile liegt noch im Feld
        RET
```

### Rand durch den Übertrag

`Masken` schiebt die Zeilenmasken von der linken Kastenspalte nach rechts, soweit `SPALTE` es verlangt. Dabei steht die Vereinigung beider Zeilen in `MASKE`. Fällt aus ihr ein Bit heraus (Carry), ragt der Stein über den linken oder rechten Rand. Dann sagt `Masken` das mit dem Carry, und `Rand` macht den Zug wie jede Kollision rückgängig.

```
MsL:  SUBI #1,ALT       ; ALT zählt SPALTE-1 Schritte herunter
      BRC  MsOk
      SHL  MASKE        ; Vereinigung um eine Spalte nach rechts
      BRC  MsRet        ; Bit fiel heraus: rechter Rand (Carry bleibt stehen)
      SHL  OBEN         ; sonst auch beide Zeilen schieben
      SHL  UNTEN
      GOTO MsL
```

### Formen und Drehen im 2x2-Kasten

Jede Form liegt in einem 2x2-Kasten, 4 Bit: Bit `#1` oben links, `#2` oben rechts, `#4` unten rechts, `#8` unten links. Drehen um 90° ist dann ein Rotieren dieser 4 Bit: `SHL FORM` und `ADC FORM` holt das herausgeschobene Bit auf Platz 1. Links drehen sind drei solche Schritte.

```
MRe:  SHL FORM          ; rechts drehen: Bits nach links, das oberste in den Carry
      ADC FORM          ; FORM = FORM + Carry: Bit kommt unten wieder herein
      RET
```

Die Form wird aus einem einzigen Würfelwurf gewählt: Das erste `SHR` auf den Würfel entscheidet mit dem Carry über die Ecke (`#7`, Wahrscheinlichkeit 1/2), das zweite über den Zweier (`#3`, 1/4), sonst bleibt das Quadrat (`#F`, 1/4). Eine zweite Ziffer bestimmt, wie oft die Form gedreht wird (0-3).

```
      RND               ; Würfel in MASKE (RD), ALT (RE), AKT (RF)
      MOVI #7,FORM      ; Ecke
      SHR  MASKE
      BRC  Dreh         ; Carry: bei Ecke bleiben
      MOVI #3,FORM      ; Zweier
      SHR  MASKE
      BRC  Dreh
      MOVI #F,FORM      ; Quadrat
Dreh: ANDI #3,ALT       ; 0-3 mal drehen
```

### Volle Reihen löschen, ohne Index

Das Programm prüft `R0`-`R7` der Reihe nach auf `#F` (`CMPI`). Findet es eine volle Reihe, springt es in eine Kette `Weg0` ... `Weg6`: Jede Zeile der Kette ist ein `MOV r(n+1),r(n)`. Das Einspringen an der richtigen Stelle schiebt alles darüber um eine Reihe nach unten, und danach läuft die Kette in `Weg7` (oberste Reihe leeren, zählen) aus. Ein Sprung pro Reihe ersetzt eine Schleife mit Index. Danach prüft das Programm von vorn, ob noch eine Reihe voll ist.

```
Voll: CMPI #F,r0        ; Reihe 1 voll?
      BRZ  Weg0
      CMPI #F,r1        ; Reihe 2 voll?
      BRZ  Weg1
      ...
Weg0: MOV  r1,r0        ; Reihe 2 nach Reihe 1
Weg1: MOV  r2,r1        ; Reihe 3 nach Reihe 2
Weg2: MOV  r3,r2        ;  ... (Einsprung bei Weg1: Reihe 1 bleibt)
      ...
Weg7: MOVI #0,r7        ; oberste Reihe leeren
      ADDI #1,REIHEN_L  ; Reihen zählen
      ADC  REIHEN_H
      GOTO Voll         ; weitere volle Reihe?
```

Der Zähler für die gelöschten Reihen hat zwei Hex-Ziffern: `ADDI #1` und `ADC` schieben den Übertrag in die obere Ziffer. Am Ende rechnet `HXDZ` ihn nach dezimal.

### Taster ohne Dauerfeuer

Gezählt wird nur ein neu gedrückter Taster: Aus dem alten und dem neuen Zustand rechnet das Programm (nicht alt) UND neu, so ist ein gehaltener Taster nach dem ersten Schritt still. Der niedrigste Taster gewinnt, seine Nummer ergibt sich durch Schieben und Zählen.

```
      MOV  TASTE,HILF   ; HILF = alter Zustand
      INV  HILF         ; HILF = nicht alt
      MOV  AKT,TASTE    ; neuer Zustand wird der alte (AKT kam aus DIN)
      AND  AKT,HILF     ; HILF = (nicht alt) UND neu: nur neu gedrückte
      BRZ  Grav         ; keiner: weiter mit Schwerkraft
      MOVI #0,AKT       ; AKT = Nummer des niedrigsten neuen Tasters
TNr:  ADDI #1,AKT
      SHR  HILF         ; Carry = Taster gedrückt
      BRC  Los
      GOTO TNr
```

Die Taster werden **innerhalb des Bildes** gelesen. Zwischen ACK und dem ersten Wert verlangt das Protokoll genau einen Befehl. Dort steht `DIN`, das den Zustand der Taster liefert: Der Befehl, der ohnehin gebraucht wird, nimmt die Abfrage mit.

```
      DOT  AKT          ; ACK 1 direkt nach dem Wert #F
      DIN  AKT          ; der eine Befehl: AKT = Taster (statt eines Leerbefehls)
      DOT  r0           ; Reihe 1 links, dann die 15 weiteren Register
```

### Tempo aus den gelöschten Reihen

Der Stein fällt nicht bei jedem Durchlauf. Ein Fallzähler zählt rückwärts, der Unterlauf (Carry) löst das Fallen aus. Er wird neu geladen mit 5 − Reihen/4: Zwei `SHR` teilen durch 4, `INV` und `SUBI #A` machen daraus 5 − x. Ab 16 Reihen, wenn die obere Ziffer nicht mehr `#0` ist, fällt der Stein bei jedem Durchlauf.

```
Grav: SUBI #1,ZAEHL     ; Fallzähler herunter
      BRC  Fall         ; Unterlauf: fallen
...
Fall: MOVI #1,ZAEHL     ; ab 16 Reihen: jeden Durchlauf
      CMPI #0,REIHEN_H
      BRC  FallA
      MOV  REIHEN_L,ZAEHL
      SHR  ZAEHL        ; Reihen / 2
      SHR  ZAEHL        ; Reihen / 4
      INV  ZAEHL        ; #F - x
      SUBI #A,ZAEHL     ; #F - x - #A = 5 - x
FallA:MOVI #5,AKT       ; Aktion fallen
```

### Vorschau und Kontextwechsel

Die Vorschau liegt als zwei Zeilen im Kontext B. Beim Wechsel zum nächsten Stein wird die alte Vorschau der aktuelle Stein, und die neue Vorschau wird gewürfelt. Registerwerte, die in den anderen Kontext müssen (Aktion, Vorschauzeilen), trägt `MAS` hinüber. Beim Spielbeginn ist die Vorschau leer, daran erkennt das Programm den Start (`Neu0`).

```
Fertig: MAS  AKT        ; AKT (RF) nach MF: überlebt den Tausch
        EXRM            ; Kontext B ist jetzt in R8-RF
        ...
Neu:    MAS  VOR_O      ; obere Vorschauzeile nach MC = OBEN im Kontext A
        ADD  VOR_U,r7   ; untere Vorschauzeile in Reihe 8
        EXRM            ; Kontext A
```

Spielende ist, wenn nach dem Landen etwas in Reihe 8 liegt: `MOV r7,r7` und der Zero-Flag, danach `HXDZ` und `DISP #3`.
