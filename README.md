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
* **Funktion:** Sie nimmt einen beliebigen Text auf, filtert automatisch störende Formatierungen oder kritische UTF-8-Sonderzeichen heraus und zerlegt den Text in mathematisch exakte Häppchen.
* **Pre-Rendering:** Um Abstürze im Browser zu verhindern, berechnet der Streamer alle QR-Codes als starre Bilder im Voraus (Filmrolle), bevor der Stream startet. Beim Abspielen werden nur noch die fertigen Bilder mit der gewünschten Hertz-Zahl (FPS) ausgetauscht.

### 2. `QR-Reader.html` (Der Empfänger)
Diese Datei wird auf dem Smartphone ausgeführt und über GitHub Pages unter der oben genannten HTTPS-Adresse bereitgestellt.
* **Funktion:** Sie greift auf die Handykamera zu und analysiert den Videostream in Echtzeit mit 30 FPS.
* **Protokoll-Header:** Jedes empfangene Paket enthält einen unsichtbaren Header (z. B. `001/010|...`). Der Reader erkennt dadurch die Reihenfolge, baut den Text wie ein Puzzle zusammen und signalisiert den Erfolg per Vibration.
* **Clipboard-Integration:** Über einen dedizierten Button kann der empfangene Text direkt in die iOS/Android-Zwischenablage kopiert werden.

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

1. Öffne den **QR-Streamer** auf dem PC und füge einen beliebig langen Text in das Textfeld ein.
2. Stelle die **Paketgrösse** auf `Gross (300 Zeichen)` und die **Geschwindigkeit** auf `10` (FPS).
3. Rufe auf dem Smartphone den oben bereitgestellten **Link** auf und erlaube den Kamerazugriff.
4. Klicke am PC auf **Stream starten** (der Streamer berechnet kurz die Frames und startet dann die Sequenz).
5. Richte die Handykamera so auf den Bildschirm, dass der QR-Code gut im quadratischen Sucherfenster zu sehen ist.
6. Der grüne Fortschrittsbalken füllt sich. Sobald alle Frames einmal erfasst wurden, ploppt der Text auf.
7. Klicke auf **Text kopieren**, um den Inhalt direkt in der Zwischenablage deines Handys zu sichern.

---

## 🔒 Sicherheitsmerkmale
* **Echte Einbahnstrasse:** Es existiert kein Rückkanal vom Smartphone zum Computer. Ein infiziertes Smartphone kann den sendenden Computer niemals infizieren.
* **Automatisches Sanitizing:** Der Sender bügelt komplexe Zeichen glatt, um sicherzustellen, dass die QR-Bibliothek niemals aufgrund von Zeichensatz-Fehlern einfriert.
