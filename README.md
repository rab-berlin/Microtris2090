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

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) im Blockmodus, die Matrix schläft zwischen den Bildern:

1. **Wecken:** Das Programm legt F an und hält es, bis der ESP mit REQ antwortet.
2. **Bild:** Dann sendet es direkt ACK 1, genau einen Befehl, die 16 Register und zum Schluss 0.
3. **Schlafen:** Danach schläft der ESP von selbst wieder ein. Er setzt kein REQ und beachtet die Ausgänge nicht, so ist Eingang 1 frei für Taster links.

Weil das F stehen bleibt, bis der ESP antwortet, geht kein Wecken verloren, auch wenn er gerade mit der Weboberfläche beschäftigt ist.

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP). Das Programm braucht die Firmware mit „ein Befehl je Wecken“ (esp2090-firmware ab PR #38).
