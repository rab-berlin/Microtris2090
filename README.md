# Microtris2090

Tetris für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4 Tastern.

- 256 Programmschritte, alle belegt
- Datei: [microtris.mic](microtris.mic) (Version 05)

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

RESET (alle Register 0, das Programm enthält kein CLEAR), Matrix einschalten, `HALT` `NEXT` `00` `RUN`. Neues Spiel: RESET.

## Matrixprotokoll

Das Programm verwendet das **frühere Blockprotokoll** der Matrix:
- Bereit-Signal auf Eingang 1
- Startnibble 1, 16 Werte, Endnibble 0
- ein F von 15-25 ms schaltet die Matrix um

Weil Eingang 1 auch Taster ist, schaltet das Programm die Matrix nur für die Bildausgabe ein.

Im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` auf `alt` stellen. Für das aktuelle Protokoll des [ESP2090 Studios](https://github.com/rab-berlin/ESP2090) muss das Programm noch umgestellt werden.
