# 📡 Optischer QR-Daten-Streamer (Air-Gapped Data Transfer)

Dieses Projekt ist ein hochperformanter Prototyp für ein **optisches Datenübertragungs-Tool**. Es ermöglicht den vollständig isolierten Einweg-Datentransfer (Air-Gap) von einem Computer auf ein Smartphone mittels einer High-Speed-QR-Code-Sequenz. 

Da keine Kabel, kein Internet, kein Bluetooth und kein WLAN für die eigentliche Übertragung im Spiel sind, gibt es auf dem Übertragungsweg keine Netzwerk-Angriffsfläche. Der Sender ist eine einzige, vollständig eigenständige HTML-Datei und funktioniert auf einem Rechner ohne Internet.

---

## 🔗 Direktlink zum Empfänger (Smartphone)

Um den QR-Reader direkt auf dem iPhone oder Android-Gerät zu starten, nutze diesen Link:

👉 **[Hier klicken, um den QR-Reader zu öffnen](https://josija-nydegger.github.io/QR-tool/QR-Reader.html)**

Falls du den Link kopieren und beispielsweise per Messenger an dein Handy schicken möchtest, kannst du diese Adresse nutzen:
https://josija-nydegger.github.io/QR-tool/QR-Reader.html

---

## 🚀 Der "Sweet Spot" (Empfehlung)
* **Geschwindigkeit:** 15 Hz (15 Frames pro Sekunde) – absolut flüssig für moderne Smartphones.
* **Paketgrösse:** Gross (300 Zeichen pro Frame - Idealer Sweet Spot zwischen Datendichte und Scan-Scharfe).
* **Performance-Maximum:** Unter perfekten Bedingungen und mit moderner Hardware sind bis zu **20–25 Hz** stabil möglich! Einzelne QR-Codes (wie Visitenkarten) werden in wenigen Millisekunden instant erfasst.

---

## 📦 System-Komponenten

Das Tool besteht aus zwei simplen, schlanken und autarken HTML-Dateien:

### 1. `QR-Streamer.html` (Der Sender)
Diese Datei wird lokal auf dem Computer (z. B. Mac) ausgeführt.
* **Modus-Vielfalt:** Unterstützt reinen Text (`Raw Text`), formatiertes `Markdown` (inkl. LaTeX-Formeln) sowie ein dediziertes Formular für `vCard-Visitenkarten`.
* **Verschlüsselung (AES-256-GCM):** Optional via eingebauter Web Crypto API des Browsers. Der Schlüssel wird mit PBKDF2-SHA256 (600'000 Runden) aus dem PIN abgeleitet, GCM erkennt falsche PINs und manipulierte Daten zuverlässig. Eine Stärke-Anzeige warnt vor zu kurzen PINs. Der PIN wird auf Wunsch für die Sitzung im RAM gemerkt.
* **100 % offline:** Die Bibliotheken (QRious, lz-string) sind direkt in die Datei eingebettet, eine Content-Security-Policy verbietet der Seite jede Netzwerkverbindung.
* **Turbo-Kompression:** Nutzt die `lz-string`-Bibliothek, um Texte vor dem Senden radikal zu komprimieren (spart bis zu 50 % der Frame-Anzahl).
* **Precision Targeting (🎯 Gezielte Frames):** Wenn am Ende eines riesigen Streams noch einzelne Frames fehlen (z. B. durch Wackeln), können die Nummern (z. B. `2, 15, 36` oder Bereiche wie `10-14`) manuell eingegeben werden. Solange Text und PIN gleich bleiben, werden exakt dieselben Frames erneut gesendet. Der Streamer isoliert diese sofort und sendet nur noch die Lücken in Dauerschleife.
* **Pre-Rendering & Kino-Modus:** Berechnet alle QR-Codes vorab, um Browser-Abstürze zu verhindern. Der *Kino-Modus* blendet die Benutzeroberfläche komplett aus und maximiert den QR-Code bildschirmfüllend für maximale Distanzen. Ein statischer Einzel-Frame blinkt nicht, sondern bleibt stabil stehen.

### 2. `QR-Reader.html` (Der Empfänger)
Diese Datei wird auf dem Smartphone ausgeführt und über GitHub Pages bereitgestellt (oder kann komplett lokal aus der Dateien-App gestartet werden).
* **Echtzeit-Dekodierung via WASM:** Analysiert den Kamerastream mit 30 FPS über die ultraschnelle `ZBar-C++`-Engine, die mittels WebAssembly direkt auf Hardware-Niveau läuft.
* **Intelligente Live-Matrix mit CSS-Sortierung (🔄 Dynamic Order):** Zeigt alle benötigten Frames als Kacheln an. **Der Clou:** Noch ungescannte (graue) Kacheln werden per Grafikkarten-Beschleunigung (CSS Grid Order) *automatisch immer nach oben sortiert*, während erfolgreich gescannte (grüne) Kacheln nach unten wegrutschen. Man sieht fehlende Frames sofort auf einen Blick.
* **Auto-Entschlüsselung & Decompress:** Erkennt verschlüsselte (`EX`/`EM`) oder komprimierte Datenströme vollautomatisch. Bei verschlüsselten Streams öffnet sich ein PIN-Eingabefeld direkt auf der Karte.
* **Gehärtet:** Markdown wird mit `DOMPurify` bereinigt (kein Script-Injection über präparierte QR-Codes), alle Bibliotheken sind auf feste Versionen mit Integritäts-Hash (SRI) gepinnt, die Kamera wird nach dem Empfang ausgeschaltet.
* **Markdown, KaTeX & PDF-Export (📄):** Rendert Markdown-Code inklusive harter Zeilenumbrüche (`breaks: true`) und komplexer Mathematik via `KaTeX`. Im Markdown-Modus erscheint ein **PDF-Export-Button**, der über die native Druck-Engine des Smartphones ein gestochen scharfes, perfekt formatiertes PDF-Dokument generiert – ganz ohne schweren Code-Ballast.
* **System-Integration:** Bietet haptisches Feedback (Vibration bei Erfolg/Fehler) sowie direkte Buttons für die Zwischenablage (`Kopieren 📋`) und das native OS-Teilen-Menü (`Teilen 📤`).

---

## 📋 Übertragungsprotokoll (Header)

Jedes Datenpaket (Protokoll v2) trägt einen Header, den der Empfänger automatisch ausliest:
`[Aktueller Frame]/[Gesamt-Frames]|[Modus-Flag]|[Stream-ID]|[Datenpaket]`

**Beispiel:** `001/005|CM|k3x9|KompakterZahlensalat...`
* `001/005`: Frame 1 von insgesamt 5 Paketen.
* `k3x9`: Zufällige Stream-ID. Verhindert, dass Frames zweier verschiedener Streams vermischt werden.
* **Die Modus-Flags:** * `TX` / `MD`: Rohtext / Unkomprimiertes Markdown
  * `CX` / `CM`: Komprimierter Text / Komprimiertes Markdown
  * `EX` / `EM`: Verschlüsselter Text / Verschlüsseltes Markdown. Datenteil = Base64( Salt 16 B · IV 12 B · AES-256-GCM-Chiffretext )

vCard-Visitenkarten werden **ohne** diesen Header als Standard-vCard gesendet, damit jede normale Handykamera sie direkt als Kontakt erkennt.

---

## 🛠️ Installations- & Setup-Anleitung

### Schritt 1: Den Sender einrichten (PC/Mac)
1. Lade die Datei `QR-Streamer.html` herunter.
2. Öffne die Datei einfach mit einem Doppelklick. Es wird kein Internet und kein Server benötigt (alle Bibliotheken sind eingebettet).

### Schritt 2: Den Empfänger einrichten (Smartphone)

**Option A: Via GitHub Pages (Standard)**
1. Erstelle ein öffentliches GitHub-Repository namens `QR-tool` und lade die `QR-Reader.html` hoch.
2. Gehe im Repo auf **Settings -> Pages**, wähle unter *Build and deployment* den `main`-Branch und klicke auf **Save**. Die App ist nach einer Minute unter deiner GitHub-Pages-URL erreichbar.

**Option B: Zu 100 % Lokal & Offline (Höchste Sicherheitsstufe)**
1. Schicke die Datei `QR-Reader.html` per AirDrop, Kabel oder Speicherkarte auf dein Smartphone.
2. Speichere sie in der Apple **„Dateien“-App** (iOS) oder im lokalen Dateimanager (Android) unter „Auf meinem iPhone/Gerät“.
3. Öffne die Datei direkt von dort. *Hinweis:* Der Reader lädt seine Bibliotheken (ZBar, marked, DOMPurify, KaTeX) beim Start von jsDelivr. Ob der Browser-Cache sie im Flugmodus wieder hergibt, ist nicht garantiert. Für zuverlässigen Offline-Betrieb müssten die Bibliotheken – wie beim Sender – direkt in die Datei eingebettet werden.

---

## 📖 Bedienungsanleitung für den Alltag

1. Öffne den **QR-Streamer** auf dem PC, wähle den Modus (Text, Markdown oder vCard) und füge den Inhalt ein. Optional einen PIN für die Verschlüsselung vergeben.
2. Klicke am PC auf **Stream starten** (oder *Kino-Modus* für ein grosses Bild).
3. Öffne den **QR-Reader** auf dem Smartphone und richte die Kamera auf den Bildschirm.
4. Die Kacheln färben sich grün und sortieren sich live nach unten. Sollten am Ende Frames fehlen, stoppe den Stream kurz am Mac, tippe die oben verbliebenen grauen Nummern in das Feld **"Gezielte Frames"** und starte den Stream kurz erneut.
5. Nach erfolgreichem Scan fährt die elegante Ergebnis-Karte hoch. Bei vergebenem PIN diesen kurz auf dem Smartphone eintippen.
6. Nutze **Kopieren**, **Teilen** oder den **PDF-Export** (nur im Markdown-Modus verfügbar), um das Ergebnis weiterzuverarbeiten.

---

## 🔒 Sicherheitsmerkmale & Grenzen
* **Echte Einbahnstrasse (Simplex):** Es existiert kein physikalischer oder funkbasierter Rückkanal vom Smartphone zum Computer. Ein kompromittiertes Smartphone kann über diesen Kanal nichts zurück auf den PC schleusen.
* **Sender ohne Netzwerk:** Der Sender bettet alle Bibliotheken ein und verbietet sich per Content-Security-Policy jede Netzwerkverbindung.
* **Gehärteter Empfänger:** Empfangenes Markdown wird mit DOMPurify bereinigt, Bibliotheken sind per SRI-Hash gegen Manipulation am CDN geschützt, und die CSP (`connect-src 'none'`) verhindert, dass die Seite empfangene Daten per Netzwerk weiterschickt.
* **Verlustfreie Normalisierung:** Der Sender vereinheitlicht nur Zeilenumbrüche und entfernt BOM/Zero-Width-Spaces. Umlaute, €, Emojis, griechische Buchstaben und mathematische Zeichen bleiben erhalten.
* **Flüchtige Verarbeitung:** Es gibt keine Datenbank und keinen Server im Hintergrund. Daten liegen nur im Arbeitsspeicher des Browser-Tabs.

**Wichtig – was die Verschlüsselung NICHT kann:** Jeder, der den Bildschirm sieht oder filmt (Kollegen, Überwachungskamera, Spiegelung im Fenster), besitzt den verschlüsselten Datenstrom und kann den PIN offline beliebig oft durchprobieren. Ein 4-stelliger Zahlen-PIN ist trotz PBKDF2 in Minuten geknackt. Für sensible Daten eine **lange Passphrase** (z. B. 4–5 zufällige Wörter) verwenden und den PIN auf einem anderen Weg mitteilen.

**Hinweis zur Kompatibilität:** Sender und Empfänger müssen beide auf Protokoll v2 aktualisiert sein. Unverschlüsselte Streams alter Sender werden noch gelesen, alte CryptoJS-verschlüsselte Streams nicht mehr.

**Lizenz-Hinweis:** `QR-Streamer.html` enthält die Bibliothek QRious (GPL-3.0) und lz-string (MIT).
