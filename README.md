# Microtris2090

Tetris für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4 Tastern.

- 256 Programmschritte, alle belegt
- Datei: [microtris.mic](microtris.mic) (Version 06)

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

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) (Blockmodus mit Schlafmodus):

- **Schlafen:** Zwischen den Bildern schläft die Matrix (ACK 15). Der ESP setzt dann kein REQ und ignoriert die Ausgänge, so ist Eingang 1 frei für Taster links.
- **Wecken und Bild:** Für ein Bild weckt das Programm die Matrix mit F und dann 0 auf den Ausgängen. Danach folgen sofort ACK 1, genau ein Befehl und die 16 Register.
- **Wieder schlafen:** Danach setzt das Programm die Ausgänge auf 0 und legt die Matrix wieder schlafen.

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP). Die Matrix muss vor dem Start eingeschaltet sein.

Bis Version 05 lief das Programm mit dem früheren Blockprotokoll (Startnibble 1, Endnibble 0, Umschaltsignal F). In Version 06 liest das Programm die Taster in dem einen Befehl zwischen ACK und erstem Register, weil REQ dann sicher Low ist. Dass Kollision und Rand jetzt denselben Rückweg nehmen, spart den Platz für den Schlafmodus: Es bleibt bei 256 Schritten.
