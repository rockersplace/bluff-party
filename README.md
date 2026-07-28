# Bluff-Partyspiel

Partyspiel für mehrere Handys: Die App liest die Frage vor, alle Mitspieler:innen
erfinden gleichzeitig eine Antwort, danach stimmen alle ab, welche Antwort die
echte ist. Niemand kennt die Lösung vorab – die App selbst übernimmt die
Vorleser:in-Rolle, damit wirklich jede Person in jeder Runde mitschreiben und
mitstimmen kann (auch bei nur 2 Spieler:innen).

Der geteilte Spielzustand (Mitspieler:innen, Runde, Antworten, Stimmen,
Punktestand) läuft über **Firebase Firestore**.

Alles steckt in einer einzigen Datei: [bluff-party.html](bluff-party.html).
Die Firebase-Konfiguration ist bereits eingetragen (Projekt `bluff-party-a9080`).

## 1. Firestore-Regeln setzen

In der Firebase Console unter **Firestore Database → Regeln** den Inhalt von
[firestore.rules](firestore.rules) einfügen und veröffentlichen. Die Regeln
erlauben Lesen/Schreiben nur für angemeldete (auch anonym angemeldete)
Nutzer:innen – offen genug für ein Partyspiel ohne Login, aber nicht komplett
öffentlich lesbar/schreibbar für jeden im Internet.

## 2. Lokal testen

Da die App ES-Module lädt, reicht ein Doppelklick auf `bluff-party.html` nicht (Browser blocken `file://`-Module). Stattdessen einen einfachen lokalen Server starten, z. B.:

```bash
cd schwindelrunde
python3 -m http.server 8080
```

Danach `http://localhost:8080/bluff-party.html` öffnen und über den Button
„Verbindung testen (Diagnose)“ prüfen, ob Firebase korrekt konfiguriert ist.

## 3. Auf GitHub Pages veröffentlichen

```bash
cd schwindelrunde
git init
git add .
git commit -m "Bluff-Partyspiel mit Firebase-Backend"
git branch -M main
git remote add origin https://github.com/<dein-user>/<dein-repo>.git
git push -u origin main
```

Danach in den GitHub-Repo-Einstellungen unter **Settings → Pages** als Quelle
den `main`-Branch (Ordner `/`) auswählen. Die Seite ist dann unter
`https://<dein-user>.github.io/<dein-repo>/bluff-party.html` erreichbar.

GitHub Pages zeigt unter der reinen Repo-URL (ohne Dateinamen) automatisch nur
eine `index.html` an. Wenn du unter `https://<dein-user>.github.io/<dein-repo>/`
(ohne `/bluff-party.html`) direkt starten willst, entweder die Datei zusätzlich
als `index.html` kopieren oder im Repo umbenennen.

Falls das Firebase-Projekt eine **autorisierte Domain** für Authentication
verlangt: In der Firebase Console unter **Authentication → Settings →
Authorized domains** die GitHub-Pages-Domain (`<dein-user>.github.io`)
hinzufügen.

## Spielablauf

1. **Warteraum:** Alle treten mit Name + Raumcode bei. Ab 2 Spieler:innen kann jede Person das Spiel starten.
2. **Schreiben:** Die App zeigt die Frage. Alle (wirklich alle, niemand sitzt aus) erfinden eine möglichst überzeugende falsche Antwort.
3. **Abstimmen:** Sobald alle abgeschickt haben, geht es automatisch zur Abstimmung. Alle sehen die echte Antwort gemischt mit den erfundenen und stimmen für die, die sie für echt halten (nicht für die eigene).
4. **Auflösung:** Sobald alle abgestimmt haben, wird automatisch aufgelöst. Punkte gibt es fürs richtige Erraten und für jede Person, die auf die eigene erfundene Antwort hereingefallen ist.
5. Nach der letzten Runde gibt es den Endstand, danach kann jede Person ein neues Spiel starten.

## Aufbau der Firestore-Daten

- `rooms/{code}` – Dokument mit `game` (Phase, Runde, Fragen-Index, Optionen, Auflösungsdaten) und `scores` (Punkte pro Spieler:in-ID)
- `rooms/{code}/players/{playerId}` – ein Dokument pro Mitspieler:in
- `rooms/{code}/submissions/{runde_playerId}` – erfundene Antworten pro Runde
- `rooms/{code}/votes/{runde_playerId}` – abgegebene Stimmen pro Runde

Alle Änderungen werden per `onSnapshot`-Listener in Echtzeit an alle Geräte im
selben Raum verteilt (kein Polling nötig). Automatische Phasenwechsel laufen
über eine Firestore-Transaktion ab, damit nicht mehrere Geräte gleichzeitig
denselben Wechsel doppelt auslösen.
