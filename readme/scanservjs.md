# scanservjs Docker-Template – Einrichtungsanleitung

scanservjs ist ein Web-Frontend für SANE-kompatible Scanner. Diese Anleitung
führt dich Schritt für Schritt durch die Einrichtung – von den Basis-Ordnern
bis zum funktionierenden Scan über die Web-Oberfläche.

## Voraussetzung

Passe in allen Befehlen unten den Pfad `/mnt/appdata/scanservjs` an dein
eigenes System an, falls dein Docker-Host ein anderes appdata-Verzeichnis
nutzt.

---

## 1. Basis-Ordner erstellen

Lege auf deinem Docker-Host die benötigten Ordner an:

```bash
mkdir -p /mnt/appdata/scanservjs/config
mkdir -p /mnt/appdata/scanservjs/output
```

## 2. Airscan-Config als Platzhalter-Datei anlegen

Damit der Container beim ersten Start nicht wegen eines fehlenden
Datei-Mounts scheitert, legst du zunächst eine leere Datei an
(auch wichtig, falls dein Scanner automatisch erkannt wird und du diese
Datei gar nicht weiter befüllen musst):

```bash
touch /mnt/appdata/scanservjs/airscan.conf
```

## 3. Rechte setzen

```bash
chown -R 102:65534 /mnt/appdata/scanservjs
```

Diese UID/GID (`102:65534`) entspricht dem internen `scanservjs`-Systemuser
im Container. Ohne diesen Schritt kann es passieren, dass der Container
Dateien nicht lesen/schreiben kann.

## 4. Container starten

Starte den Container jetzt einmal mit den Standardwerten des Templates.

## 5. Web-UI öffnen und prüfen

Rufe die Web-Oberfläche auf:

```
http://<HOST-IP>:<PORT>
```

Taucht dein Scanner bereits in der Geräteliste auf? 🎉
**Dann bist du fertig – die restlichen Schritte sind nicht nötig.**

Viele moderne Netzwerk-Scanner (eSCL/AirPrint-fähig) werden automatisch
per mDNS erkannt.

---

## Falls der Scanner NICHT automatisch gefunden wird

### 6. Scanner manuell suchen

Öffne ein Terminal/eine Konsole im laufenden Container (z. B. über die
"Console"-Funktion deiner Docker-Verwaltungsoberfläche) und führe aus:

```bash
scanimage -L
```

**Ergebnis A – nichts gefunden:**
Prüfe, ob dein Scanner die richtige IP-Adresse hat (meist im Display des
Geräts unter Netzwerkeinstellungen einsehbar) und ob er im selben
Netzwerk/Subnetz wie der Docker-Host hängt.

**Ergebnis B – Scanner gefunden**, z. B.:
```
device `airscan:e0:MeinScanner' is a eSCL MeinScanner ip=192.168.1.35
```
→ Weiter mit Schritt 7.

### 7. airscan.conf mit den echten Werten befüllen

Trage die IP-Adresse und einen frei wählbaren Gerätenamen ein
(keine Leerzeichen im Namen verwenden):

```bash
cat > /mnt/appdata/scanservjs/airscan.conf << 'EOF'
[devices]
DEIN_GERAETENAME = http://DEINE_SCANNER_IP/eSCL, eSCL

[options]
discovery = disable
EOF
```

Beispiel:
```
[devices]
Brother_MFC_J947DW = http://192.168.1.35/eSCL, eSCL

[options]
discovery = disable
```

### 8. Geräte-ID im Template eintragen

Trag den exakten Wert aus der `scanimage -L`-Ausgabe (der Teil zwischen
den Backticks `` ` ``) in die **`DEVICES`**-Variable im Docker-Template ein:

```
DEVICES=airscan:e0:DEIN_GERAETENAME
```

Beispiel:
```
DEVICES=airscan:e0:Brother_MFC_J947DW
```

Das erzwingt eine feste Geräteliste in der Web-UI, unabhängig von der
automatischen Erkennung.

### 9. Container neu erstellen

Erstelle den Container **neu** (nicht nur neu starten), damit die
geänderte `airscan.conf` und die neue `DEVICES`-Variable übernommen
werden.

### 10. Erneut prüfen

```bash
scanimage -L
```

Öffne danach wieder die Web-UI – der Scanner sollte jetzt zuverlässig
in der Geräteliste erscheinen.

---

## Optionale Einstellungen

### OCR-Sprache ändern

Standardmäßig oft Englisch. Für Deutsch die Variable setzen:

```
OCR_LANG=deu
```

Weitere Beispiele: `eng` (Englisch), `fra` (Französisch), `spa` (Spanisch).
Mehrere Sprachen kombinieren: `deu+eng`

---

## Kurz-Checkliste

| Schritt | Nötig für |
|---|---|
| 1–5 | Alle Nutzer (Basissetup) |
| 6–10 | Nur falls der Scanner nicht automatisch gefunden wird |
| OCR-Sprache | Optional, für Texterkennung in gescannten Dokumenten |
