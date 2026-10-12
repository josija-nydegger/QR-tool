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

## 🚀 Empfohlene Einstellungen
* **Start:** Paketgrösse *Normal (400 B)*, 1 Code pro Bild, 15 Bilder/Sek. ≈ 6 KB/s – sicher auf fast allen Handys.
* **Schneller:** Im **Kino-Modus** Paketgrösse *Gross (700 B)* oder *Riesig (1000 B)* und **2 oder 4 Codes pro Bild** ausprobieren. Damit sind theoretisch 20–60 KB/s drin; in der Praxis begrenzt die Kamera bzw. die Scan-Geschwindigkeit des Handys.
* **Faustregel:** Wenn der Empfänger kaum noch Pakete zählt, Paketgrösse oder Anzahl Codes eine Stufe zurück. Dank Fountain-Code ist es egal, wenn einzelne Bilder verloren gehen.

---

## 📦 System-Komponenten

Das Tool besteht aus zwei simplen, schlanken und autarken HTML-Dateien:

### 1. `QR-Streamer.html` (Der Sender)
Diese Datei wird lokal auf dem Computer (z. B. Mac) ausgeführt.
* **Modus-Vielfalt:** Unterstützt reinen Text (`Raw Text`), formatiertes `Markdown` (inkl. LaTeX-Formeln) sowie ein dediziertes Formular für `vCard-Visitenkarten`.
* **Verschlüsselung (AES-256-GCM):** Optional via eingebauter Web Crypto API des Browsers. Der Schlüssel wird mit PBKDF2-SHA256 (600'000 Runden) aus dem PIN abgeleitet, GCM erkennt falsche PINs und manipulierte Daten zuverlässig. Eine Stärke-Anzeige warnt vor zu kurzen PINs. Der PIN wird auf Wunsch für die Sitzung im RAM gemerkt.
* **100 % offline:** Die QR-Bibliothek (qrcode-generator) ist direkt in die Datei eingebettet, Kompression und Verschlüsselung sind im Browser eingebaut. Eine Content-Security-Policy verbietet der Seite jede Netzwerkverbindung.
* **Kompression:** Deflate über die eingebaute `CompressionStream`-API (bei normalem Text ~20 % kleiner als das frühere lz-string).
* **Fountain-Code (🌊 keine fehlenden Frames mehr):** Zuerst wird jeder Datenblock einmal direkt gesendet, danach endlos neue, zufällige XOR-Kombinationen aller Blöcke. Der Empfänger braucht nur so viele *beliebige* Pakete, wie es Blöcke gibt (plus meist 1–2). Welche Bilder verpasst werden, spielt keine Rolle – das frühere manuelle Nachsenden einzelner Frames ist damit überflüssig.
* **Mehrere Codes pro Bild:** 1, 2 oder 4 QR-Codes gleichzeitig vervielfachen den Durchsatz.
* **Dichtere Codes (Base45):** Daten werden im kompakten QR-Alphanumerik-Modus kodiert – rund 20 % mehr Nutzdaten pro Code als mit Base64.
* **Kino-Modus:** Blendet die Benutzeroberfläche aus und zeigt die Codes bildschirmfüllend und pixelgenau. Stoppen und Neustarten mit gleichem Text und PIN setzt den Stream fort, statt neu zu beginnen.

### 2. `QR-Reader.html` (Der Empfänger)
Diese Datei wird auf dem Smartphone ausgeführt und über GitHub Pages bereitgestellt (oder kann komplett lokal aus der Dateien-App gestartet werden).
* **Echtzeit-Dekodierung via WASM:** Analysiert den Kamerastream (1080p) mit der `ZBar`-Engine als WebAssembly und liest **alle** QR-Codes in einem Bild.
* **Fountain-Decoder:** Setzt die Daten per Gauss-Elimination zusammen, sobald genug Pakete da sind, und prüft das Ergebnis mit einer SHA-256-Prüfsumme.
* **Intelligente Live-Matrix mit CSS-Sortierung (🔄 Dynamic Order):** Zeigt den Fortschritt als Kacheln an (eine Kachel pro rekonstruierbarem Block). **Der Clou:** Noch ungescannte (graue) Kacheln werden per Grafikkarten-Beschleunigung (CSS Grid Order) *automatisch immer nach oben sortiert*, während erfolgreich gescannte (grüne) Kacheln nach unten wegrutschen. Man sieht den Fortschritt sofort auf einen Blick.
* **Auto-Entschlüsselung & Decompress:** Erkennt verschlüsselte und komprimierte Datenströme vollautomatisch. Bei verschlüsselten Streams öffnet sich ein PIN-Eingabefeld direkt auf der Karte.
* **Gehärtet:** Markdown wird mit `DOMPurify` bereinigt (kein Script-Injection über präparierte QR-Codes), alle Bibliotheken sind auf feste Versionen mit Integritäts-Hash (SRI) gepinnt, die Kamera wird nach dem Empfang ausgeschaltet.
* **Markdown, KaTeX & PDF-Export (📄):** Rendert Markdown-Code inklusive harter Zeilenumbrüche (`breaks: true`) und komplexer Mathematik via `KaTeX`. Im Markdown-Modus erscheint ein **PDF-Export-Button**, der über die native Druck-Engine des Smartphones ein gestochen scharfes, perfekt formatiertes PDF-Dokument generiert – ganz ohne schweren Code-Ballast.
* **Transfer-Statistik (📊):** Nach dem Empfang zeigt die Karte Dauer, Ø- und Max.-Durchsatz, gelesene vs. benötigte Pakete sowie übertragene Datenmenge und Kompressionsrate. Ideal, um die besten Sender-Einstellungen für dein Handy zu finden. Erscheint nicht im PDF.
* **Datei-Export (💾):** Text-Modus → `.txt`. Markdown-Modus → `.md` (Originaltext für Obsidian, Notion, VS Code). Erkennt der Reader eine Tabelle → zusätzlich `.csv` (öffnet sich mit korrekten Umlauten in Excel, Numbers und Google Sheets). Alle Dateien in UTF-8, z. B. direkt in die Dateien-App.
* **System-Integration:** Bietet haptisches Feedback (Vibration bei Erfolg/Fehler) sowie direkte Buttons für die Zwischenablage (`Kopieren 📋`) und das native OS-Teilen-Menü (`Teilen 📤`).

---

## 📋 Übertragungsprotokoll (v3)

Jeder QR-Code enthält `QR3:` gefolgt von Base45-kodierten Bytes: einem 20-Byte-Header und einem Datenblock.

| Bytes | Inhalt |
|---|---|
| 0 | Version (3) |
| 1 | Flags: Bit 0 = Markdown, Bit 1 = verschlüsselt |
| 2–5 | Zufällige Stream-ID (verhindert das Vermischen zweier Streams) |
| 6–9 | Gesamtlänge der Daten |
| 10–11 | Blockgrösse |
| 12–15 | Paketnummer (0 … K−1 = Blöcke direkt, ab K = Fountain-Kombinationen) |
| 16–19 | Prüfsumme (erste 4 Bytes SHA-256 der Daten) |

**Daten:** `Deflate(UTF-8-Text)`, bei gesetztem PIN danach `Salt (16 B) · IV (12 B) · AES-256-GCM-Chiffretext` (Schlüssel via PBKDF2-SHA256, 600'000 Runden).

**Fountain:** Welche Blöcke in Paket *n* XOR-verknüpft sind, wird aus Stream-ID und Paketnummer per Pseudozufall abgeleitet – Sender und Empfänger berechnen dieselbe Auswahl.

vCard-Visitenkarten werden **ohne** dieses Protokoll als Standard-vCard gesendet, damit jede normale Handykamera sie direkt als Kontakt erkennt.

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
4. Die Kacheln färben sich grün. Einfach laufen lassen, bis alle grün sind – verpasste Bilder werden automatisch durch spätere Pakete ersetzt.
5. Nach erfolgreichem Scan fährt die elegante Ergebnis-Karte hoch. Bei vergebenem PIN diesen kurz auf dem Smartphone eintippen.
6. Nutze **Kopieren**, **Teilen** oder die Export-Buttons (**TXT** im Text-Modus, **PDF** und **MD** im Markdown-Modus, **CSV** bei Tabellen), um das Ergebnis weiterzuverarbeiten.

---

## 🔒 Sicherheitsmerkmale & Grenzen
* **Echte Einbahnstrasse (Simplex):** Es existiert kein physikalischer oder funkbasierter Rückkanal vom Smartphone zum Computer. Ein kompromittiertes Smartphone kann über diesen Kanal nichts zurück auf den PC schleusen.
* **Sender ohne Netzwerk:** Der Sender bettet alle Bibliotheken ein und verbietet sich per Content-Security-Policy jede Netzwerkverbindung.
* **Gehärteter Empfänger:** Empfangenes Markdown wird mit DOMPurify bereinigt, Bibliotheken sind per SRI-Hash gegen Manipulation am CDN geschützt, und die CSP (`connect-src 'none'`) verhindert, dass die Seite empfangene Daten per Netzwerk weiterschickt.
* **Verlustfreie Normalisierung:** Der Sender vereinheitlicht nur Zeilenumbrüche und entfernt BOM/Zero-Width-Spaces. Umlaute, €, Emojis, griechische Buchstaben und mathematische Zeichen bleiben erhalten.
* **Flüchtige Verarbeitung:** Es gibt keine Datenbank und keinen Server im Hintergrund. Daten liegen nur im Arbeitsspeicher des Browser-Tabs.

**Wichtig – was die Verschlüsselung NICHT kann:** Jeder, der den Bildschirm sieht oder filmt (Kollegen, Überwachungskamera, Spiegelung im Fenster), besitzt den verschlüsselten Datenstrom und kann den PIN offline beliebig oft durchprobieren. Ein 4-stelliger Zahlen-PIN ist trotz PBKDF2 in Minuten geknackt. Für sensible Daten eine **lange Passphrase** (z. B. 4–5 zufällige Wörter) verwenden und den PIN auf einem anderen Weg mitteilen.

**Hinweis zur Kompatibilität:** Sender und Empfänger müssen beide auf Protokoll v3 aktualisiert sein. Erkennt der Reader einen alten Sender (v1/v2), zeigt er einen Hinweis zum Aktualisieren. Benötigt werden iOS 16.4+ bzw. ein aktuelles Chrome/Firefox/Edge.

**Lizenz-Hinweis:** `QR-Streamer.html` enthält die Bibliothek qrcode-generator von Kazuhiko Arase (MIT) mit einer kleinen, markierten Anpassung.
