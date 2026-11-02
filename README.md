# Leasing-Radar

Prüft jeden Morgen die großen Leasing-Portale in Deutschland und zeigt die günstigsten Privatleasing-Angebote bis 150 € im Monat auf einer Webseite, die du am Rechner und am Handy öffnen kannst.

- **Quellen:** LeasingMarkt.de, mobile.de, Null-Leasing (GoLeasy ist vorbereitet, aber ausgeschaltet)
- **Läuft kostenlos** in GitHub Actions, die Seite liegt auf GitHub Pages
- **Zeigt** Rate (inkl. MwSt.), Laufzeit, Fahrleistung, Anzahlung, Bereitstellung und Gesamtkosten, dazu „Neu“ und „Rate gesenkt“ gegenüber dem Vortag
- **Filter:** Zustand, Laufzeit, Fahrleistung, Marke, Portal, Modellsuche, maximale Rate

## Einrichtung (einmalig, ca. 10 Minuten)

1. Konto bei [github.com](https://github.com) anlegen (kostenlos).
2. Neues Repository **leasing-radar** erstellen. Es muss **öffentlich** sein, weil GitHub Pages im kostenlosen Tarif nur so funktioniert. Es enthält nur öffentliche Angebotsdaten.
3. Den Inhalt dieses Ordners hochladen. Wichtig ist der versteckte Ordner `.github`. Am einfachsten geht es mit Git:
   ```
   cd leasing-radar
   git init -b main
   git add .
   git commit -m "Leasing-Radar"
   git remote add origin https://github.com/<dein-name>/leasing-radar.git
   git push -u origin main
   ```
4. Im Repository: **Settings → Pages → Source: GitHub Actions** wählen.
5. Im Reiter **Actions** den Eintrag „Tägliche Leasing-Prüfung“ öffnen und **Run workflow** drücken. Nach ein bis drei Minuten ist die Seite unter `https://<dein-name>.github.io/leasing-radar/` erreichbar.
6. Ab jetzt läuft die Prüfung automatisch jeden Tag gegen 06:20 Uhr. Die Adresse als Lesezeichen oder Startbildschirm-Symbol auf dem Handy speichern.

Wenn **kein** Portal Angebote liefert, wird der Lauf rot und GitHub schickt dir eine E-Mail. Fällt nur ein Portal aus, zeigt die Seite oben eine Warnung und für drei Tage den letzten bekannten Stand dieses Portals.

## Einstellungen

Alles steht in `config.toml`:

| Einstellung | Bedeutung |
|---|---|
| `max_rate` | Höchste Monatsrate in Euro, Standard 150 |
| `customer` | `privat`, `gewerbe` oder `beide` |
| `condition` | `alle`, `neu` oder `gebraucht` |
| `[[source]]` | Je Portal Start-Adressen, Seitenzahl und Ein-/Ausschalter |

Eine Änderung gilt ab dem nächsten Lauf. Sofort geht es über **Actions → Run workflow**.

## Wenn ein Portal nichts liefert

Die Portale ändern ihre Seiten regelmäßig, und einige blocken automatische Abrufe. Das Projekt umgeht solche Sperren bewusst nicht. Dann gilt:

1. Auf der Seite steht beim Portal die Fehlermeldung, zum Beispiel „HTTP 403“.
2. Meldet der Lauf „Keine Angebote erkannt“, liegt im Lauf unter **Artifacts → debug-html** die abgerufene Seite. Daran lässt sich der Parser nachjustieren. In `config.toml` kann pro Portal ein `card_selector` (CSS-Selektor für eine Angebotskarte) gesetzt werden.
3. Bei GoLeasy und bei weiteren Markenseiten hilft es, im Browser den Filter „bis 150 €“ zu setzen, die Adresse zu kopieren und bei `start_urls` einzutragen.

## Wie die Erkennung funktioniert

Der Parser verlässt sich nicht auf CSS-Klassen. Er sucht auf jeder Seite nach Monatsraten in Euro mit einer Laufzeit in Monaten, bestimmt die umschließende Angebotskarte und liest daraus Titel, Fahrleistung, Anzahlung, Bereitstellung, Kundengruppe, Neu/Gebraucht und Link. Netto-Raten („zzgl. MwSt.“) werden mit 19 % auf Brutto umgerechnet, Angebote „nur Gewerbe“ fallen bei `customer = "privat"` heraus. Gleiche Angebote von mehreren Portalen werden zusammengelegt.

## Wichtige Hinweise

- **Er wurde nur an nachgebauten Seiten getestet.** Beim Bauen kam die Entwicklungsumgebung nicht an die Portale heran. Wie gut die Erkennung bei den echten Seiten greift, zeigt erst der erste Lauf. Die Seite zeigt je Portal an, ob der Abruf geklappt hat und wie viele Angebote gefunden wurden.
- **Die ausgelieferten Beispieldaten** (Banner „Beispieldaten“) stammen aus den Übersichtsseiten vom 23.10.2026 und werden beim ersten Lauf ersetzt.
- **Höflich abrufen:** Das Skript liest `robots.txt`, wartet 2,5 Sekunden zwischen Abrufen und ruft jedes Portal nur einmal am Tag ab. Bitte die Nutzungsbedingungen der Portale für automatisierte Abrufe selbst prüfen. Einige untersagen sie. Das Projekt ist für den persönlichen Gebrauch gedacht.
- **Gesamtkosten** sind Rate × Laufzeit plus Anzahlung und Bereitstellung, soweit das Portal sie nennt. Überführung, Zulassung, Versicherung und Restwert-Risiken sind nicht enthalten. Vor Vertragsabschluss alles beim Anbieter prüfen.

## Lokal ausführen

```
pip install -r requirements.txt
python -m scraper.main                 # ruft die Portale ab und schreibt docs/data/offers.json
python -m http.server -d docs 8000     # Seite unter http://localhost:8000 öffnen
python -m unittest discover -s tests -t .   # Tests
```

Für einen täglichen Lauf auf dem eigenen Rechner reicht ein Eintrag in der Aufgabenplanung (Windows) oder `cron` (Mac/Linux), der `python -m scraper.main` im Projektordner startet.

## Aufbau

```
config.toml              Einstellungen und Quellen
scraper/                 Abruf (fetch, robots), Erkennung (parse), Ablauf (crawl, dataset, main)
docs/index.html          die Webseite
docs/data/offers.json    die Angebote des letzten Laufs
data/history.json        Verlauf für „Neu“ und „Rate gesenkt“
.github/workflows/       täglicher Lauf und Veröffentlichung
tests/                   Tests mit nachgebauten Portalseiten
```
