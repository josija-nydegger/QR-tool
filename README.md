# 📡 Optischer QR-Daten-Streamer (Air-Gapped Data Transfer)

Dieses Projekt ist ein Prototyp für ein **optisches Datenübertragungs-Tool**. Es ermöglicht den vollständig isolierten Einweg-Datentransfer (Air-Gap) von einem Computer auf ein Smartphone mittels einer High-Speed-QR-Code-Sequenz. 

Da keine Kabel, kein Internet, kein Bluetooth und kein WLAN im Spiel sind, ist diese Übertragungsmethode vollkommen immun gegen Angriffe aus dem Netzwerk.

---

## 🔗 Direktlink zum Empfänger (Smartphone)

Um den QR-Reader direkt auf dem iPhone oder Android-Gerät zu starten, nutze diesen Link:

👉 **[Hier klicken, um den QR-Reader zu öffnen](https://josija-nydegger.github.io/QR-tool/QR-Reader.html)**

Falls du den Link kopieren und beispielsweise per Messenger an dein Handy schicken möchtest, kannst du diese Adresse nutzen:
https://josija-nydegger.github.io/QR-tool/QR-Reader.html

---

## 🚀 Der "Sweet Spot" (Empfehlung)
* **Geschwindigkeit:** 10 Hz (10 Frames pro Sekunde)
* **Paketgrösse:** Gross (300 Zeichen pro Frame)
* **Performance-Maximum:** Unter perfekten Bedingungen und mit moderner Hardware sind bis zu **20 Hz** stabil möglich!

---

## 📦 System-Komponenten

Das Tool besteht aus zwei simplen, schlanken HTML-Dateien:

### 1. `QR-Streamer.html` (Der Sender)
Diese Datei wird lokal auf dem Computer (z. B. Mac) ausgeführt.
* **Modus-Auswahl:** Unterstützt wahlweise reinen Text (`Raw Text`) oder formatiertes `Markdown`.
* **Turbo-Kompression:** Nutzt die `lz-string`-Bibliothek, um Texte vor dem Senden im Hintergrund radikal zu komprimieren. Das spart bis zu 50 % der Datenmenge ein und halbiert die nötige Frame-Anzahl.
* **Pre-Rendering:** Um Abstürze im Browser zu verhindern, berechnet der Streamer alle QR-Codes als starre Bilder im Voraus (Filmrolle), bevor der Stream startet. Beim Abspielen werden nur noch die fertigen Bilder mit der gewünschten Hertz-Zahl (FPS) ausgetauscht.

### 2. `QR-Reader.html` (Der Empfänger)
Diese Datei wird auf dem Smartphone ausgeführt und über GitHub Pages unter der oben genannten HTTPS-Adresse bereitgestellt.
* **Echtzeit-Dekodierung:** Greift auf die Handykamera zu und analysiert den Videostream mit 30 FPS.
* **Auto-Decompress:** Erkennt komprimierte Datenströme vollautomatisch und entpackt sie blitzschnell nach dem Empfang.
* **Markdown-Parser:** Integriert `marked.js`, um übertragenen Markdown-Code (Überschriften, Listen, fette Texte, Links) direkt visuell ansprechend auf dem Smartphone-Bildschirm zu rendern.
* **Clipboard-Integration:** Über den Button "Text kopieren" wird der saubere, dekomprimierte Rohtext (inklusive der Markdown-Formatstempel) direkt in die iOS/Android-Zwischenablage kopiert, um ihn in anderen Apps weiterzuverwenden.

---

## 📋 Übertragungsprotokoll (Header)

Jedes Datenpaket wird mit einem intelligenten, dreiteiligen Header versehen, den der Empfänger automatisch ausliest:
`[Aktueller Frame]/[Gesamt-Frames]|[Modus-Flag]|[Datenpaket]`

**Beispiel:** `001/005|CM|KompakterZahlensalat...`
* `001/005`: Frame 1 von insgesamt 5 Paketen.
* `CM`: Modus-Flag für **C**ompressed **M**arkdown (weitere Modi: `CX` für Compressed Text, `TX` für Raw Text, `MD` für Uncompressed Markdown).

---

## 🛠️ Installations- & Setup-Anleitung

### Schritt 1: Den Sender einrichten (PC/Mac)
1. Lade die Datei `QR-Streamer.html` herunter.
2. Öffne die Datei einfach mit einem Doppelklick in einem beliebigen Browser (Safari, Chrome, Firefox). Es wird kein Server benötigt.

### Schritt 2: Den Empfänger einrichten (Smartphone via GitHub Pages)
Da Smartphones den Kamerazugriff im Browser aus Datenschutzgründen nur über gesicherte Leitungen erlauben, ist das Hosting via GitHub Pages ideal:
1. Erstelle ein öffentliches GitHub-Repository namens `QR-tool`.
2. Lade die Datei `QR-Reader.html` in dieses Repository hoch.
3. Gehe in den Repository-Einstellungen auf **Settings -> Pages**.
4. Wähle unter *Build and deployment* den `main`-Branch und den Ordner `/(root)` aus und klicke auf **Save**.
5. Deine App ist nach ca. einer Minute unter dem oben angegebenen Link erreichbar.

---

## 📖 Bedienungsanleitung für einen Testlauf

1. Öffne den **QR-Streamer** auf dem PC, wähle den gewünschten Modus (Raw Text oder Markdown) und füge deinen Text ein.
2. Stelle die **Paketgrösse** auf `Gross (300 Zeichen - Sweet Spot)` und die **Geschwindigkeit** auf `10` (FPS).
3. Die Live-Statistik zeigt dir sofort die originale Byte-Grösse sowie die reale Dateneinsparung durch die Kompression an.
4. Rufe auf dem Smartphone den oben bereitgestellten **Link** auf und erlaube den Kamerazugriff.
5. Klicke am PC auf **Stream starten** (der Streamer berechnet kurz die Bilder vor und startet dann die Sequenz).
6. Richte die Handykamera so auf den Bildschirm, dass der grosse QR-Code gut im quadratischen Sucherfenster zu sehen ist.
7.
