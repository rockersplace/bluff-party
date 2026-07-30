# Bluff-Partyspiel

Partyspiel für mehrere Handys: Pro Runde liest eine reihum wechselnde
„Vorleser:in" die Frage vor. Alle Mitspieler:innen (die Vorleser:in
eingeschlossen) erfinden eine möglichst überzeugende falsche Antwort, danach
stimmt der Rest ab, welche Antwort die echte ist. Beim Abstimmen sind die
Antworten anonymisiert (nur Buchstaben A, B, C, …) sichtbar, damit niemand am
Schreibstil erkannt wird – erst in der Auflösung werden Text und Autor:in
gezeigt.

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
cd Bluff-party
python3 -m http.server 8080
```

Danach `http://localhost:8080/bluff-party.html` öffnen und über den Button
„Verbindung testen (Diagnose)“ prüfen, ob Firebase korrekt konfiguriert ist.

## 3. Auf GitHub Pages veröffentlichen

```bash
cd Bluff-party
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

## Rollen

- **Spielleitung** (👑, wer den Raumcode erzeugt hat): fix für die ganze
  Partie, startet das Spiel (Rundenzahl, Kategorie), setzt es bei Bedarf
  zurück und startet am Ende eine neue Partie.
- **Vorleser:in**: wechselt jede Runde reihum durch alle Mitspieler:innen.
  Steuert den Ablauf der eigenen Runde (Kategoriewahl, Weiterschalten). Bei
  normalen Fragen sieht nur sie/er die echte Antwort, schreibt aber trotzdem
  selbst eine erfundene Antwort mit und stimmt nicht mit ab (kennt die
  Lösung ja schon).

## Spielablauf

1. **Warteraum:** Alle treten mit Name + Raumcode bei. Die Spielleitung wählt
   Rundenzahl (4/6/8) und Kategorie; ab 2 Spieler:innen kann sie starten.
2. **Schreiben:** Alle (auch die/der Vorleser:in) erfinden eine möglichst
   überzeugende falsche Antwort. Bei „Wahr oder falsch?“-Fragen entfällt
   dieser Schritt, hier stimmen direkt alle ab.
3. **Abstimmen:** Sobald alle abgeschickt haben, geht es zur Abstimmung. Die
   Optionen sind anonymisiert nur mit Buchstaben (A, B, C, …) sichtbar. Alle
   außer der/dem Vorleser:in stimmen für die Antwort, die sie für echt
   halten (nicht für die eigene).
4. **Auflösung:** Die/der Vorleser:in löst auf, sobald alle abgestimmt haben
   (oder früher, mit Sicherheitsabfrage). Jetzt werden Texte und Autor:in je
   Buchstabe gezeigt. Anschließend wählt die/der Vorleser:in die Kategorie
   für die nächste Runde, und die Rolle wandert zur nächsten Person weiter.
5. Nach der letzten Runde gibt es eine animierte Siegerehrung (Podium,
   Plätze 3→2→1, von der Spielleitung Schritt für Schritt enthüllt), danach
   kann die Spielleitung ein neues Spiel starten.

Die „Weiter"-Buttons sind nie hart gesperrt (auch wenn nicht alle fertig
sind) – es kommt dann nur eine Sicherheitsabfrage, damit niemand für immer
feststeckt, falls z. B. eine Antwort/Stimme nie ankommt.

## Punktevergabe

- **2 Punkte** für richtiges Erraten der echten Antwort.
- **3 Punkte** für jede Person, die auf deine erfundene Antwort hereinfällt (pro getäuschter Stimme).

Werte stehen ganz oben im `<script>`-Block als `POINTS_CORRECT_GUESS` und
`POINTS_PER_FOOLED` und lassen sich dort bei Bedarf leicht anpassen.

## Fragenkatalog

Aktuell rund 200 Fragen in vier Kategorien (Kuriosität, Wahr oder falsch?,
Schräges Ereignis, Unbekanntes Wort), plus eine „Zufall“-Option, die aus
allen mischt. Die Kategorie lässt sich vor jeder Runde neu wählen. Innerhalb
eines Spiels wiederholt sich keine Frage (`usedQuestions`-Tracking). Bei sehr
vielen Runden über mehrere Spielabende hinweg können sich Fragen irgendwann
trotzdem wiederholen, da der Katalog statisch (nicht KI-generiert) ist.

## Bekannte Stolperfallen beim Testen auf dem Handy

- Mobile Browser (v. a. iOS Safari) frieren die Firestore-Verbindung ein,
  sobald der Tab in den Hintergrund geht (Bildschirm sperren, App wechseln).
  Die App abonniert die Live-Daten deshalb automatisch neu, sobald der Tab
  wieder sichtbar wird oder das Netz zurückkommt – kein manuelles Neuladen
  nötig. Falls ein Tab doch komplett neu lädt: Name, Raumcode und Spieler-ID
  stehen in `localStorage`, die App steigt automatisch wieder im selben Raum
  ein.
- Private/Inkognito-Tabs haben ihren eigenen `localStorage` (eigene
  Spieler-ID). Praktisch zum Testen mehrerer „Personen" im selben Browser,
  aber: kehrt dieselbe reale Person mitten in einer Partie über einen neuen
  Inkognito-Tab zurück, gilt sie technisch als neue Person ohne alten
  Punktestand.
- Nach dem Abschicken einer Antwort bleibt das Textfeld bewusst editierbar
  (Button wechselt zu „Antwort aktualisieren“, plus eine grüne
  Bestätigung „✓ Antwort gesendet“) – so ist klar sichtbar, dass die Antwort
  angekommen ist, auch wenn andere noch schreiben.

## Aufbau der Firestore-Daten

- `rooms/{code}` – Dokument mit `hostId` (Spielleitung, fix für die Partie),
  `game` (Phase, Runde, `readerId` der aktuellen Vorleser:in, Fragen-Index,
  Optionen, Auflösungsdaten) und `scores` (Punkte pro Spieler:in-ID)
- `rooms/{code}/players/{playerId}` – ein Dokument pro Mitspieler:in
- `rooms/{code}/submissions/{runde_playerId}` – erfundene Antworten pro Runde
- `rooms/{code}/votes/{runde_playerId}` – abgegebene Stimmen pro Runde

Alle Änderungen werden per `onSnapshot`-Listener in Echtzeit an alle Geräte im
selben Raum verteilt (kein Polling nötig). Automatische Phasenwechsel laufen
über eine Firestore-Transaktion ab, damit nicht mehrere Geräte gleichzeitig
denselben Wechsel doppelt auslösen.
