# C99 AI-Hypervisor

Ein Programm, das mit KI-Diensten spricht – über deine angemeldete
Browsersitzung bei claude.ai, chat.deepseek.com, gemini.google.com,
chatgpt.com und grok.com, oder über eine API mit Schlüssel – und mehrere
davon zu einem **Ablauf** verbindet: wer zuerst antwortet, wer gleichzeitig,
wer wessen Antwort liest und wessen Urteil am Ende zählt.

![Ein Ablauf: zwei Agenten gleichzeitig, ein dritter liest beide und urteilt](bilder/ablauf.png)

## Inhalt

| Ordner | Programme |
|---|---|
| `windows/` | `gpb-gui.exe` (Fenster), `gpb-browser.exe` (Kommandozeile) – keine Installation nötig |
| `linux-x86_64/` | `gpb-gui`, `gpb-browser` – braucht X11/Xft und OpenSSL (auf üblichen Desktops vorhanden) |

Starten: `gpb-gui` doppelklicken bzw. im Terminal aufrufen. Alles, was du
einrichtest (Zugänge, Agenten, Abläufe, Gedächtnis), liegt im Ordner `daten`
**neben dem Programm**. Wer das Programm mitsamt diesem Ordner kopiert,
nimmt alles mit. Der Ordner enthält unter `daten/keks/` deine
Sitzungs-Cookies – gib ihn nicht weiter.

## In drei Schritten zur ersten Frage

Beim ersten Start zeigt das Fenster diese Schritte selbst an.

**1. Zugang einrichten** (Ansicht *Endpunkte*)

- *Alle auf einmal:* Im Browser bei den Diensten anmelden, mit einer
  Cookie-Erweiterung (z. B. „Get cookies.txt") alle Cookies im
  Netscape-Format exportieren, den Pfad bei **Aus Datei** eintragen. Dazu
  den **User-Agent** deines Browsers eintragen (in der Browser-Konsole
  `navigator.userAgent` eingeben).
- *Ein Dienst:* Im Browser **F12 → Netzwerk**, Seite neu laden, rechte
  Maustaste auf eine Anfrage an den Dienst → **Als cURL kopieren**, dann in
  der Zeile des Dienstes **Aus Zwischenablage**. Bei DeepSeek eine Anfrage
  unter `/api/` nehmen – nur die trägt den nötigen Zugangstoken.
- Die Liste zeigt danach je Dienst, ob etwas fehlt.

Für eine API (Claude, OpenAI, DeepSeek, OpenRouter) wird statt dessen der
Schlüssel als Umgebungsvariable gesetzt: `ANTHROPIC_API_KEY`,
`OPENAI_API_KEY`, `DEEPSEEK_API_KEY`, `OPENROUTER_API_KEY`. Ein lokales
Ollama braucht nichts.

**2. Agent anlegen** (Ansicht *Agenten*)

Ein Agent ist ein Dienst plus eine Rolle („Du prüfst kritisch …") und
optional ein Gedächtnis. Mit *Gespräch fortsetzen* schreibt er beim Anbieter
immer im selben Gespräch weiter, statt jedes Mal ein neues anzulegen.

**3. Ablauf bauen und fragen** (Ansicht *Ablauf*)

- **+ Agent** unter „neue Stufe" nimmt einen Agenten dazu.
- **Ziehen:** in eine Spalte = läuft gleichzeitig mit den anderen dort;
  zwischen zwei Spalten = eine neue Stufe dazwischen. Wer später steht,
  liest die Frage und alle Antworten davor.
- **Anklicken:** An der Frage und an früheren Agenten erscheinen Punkte –
  ein Klick schaltet, ob dieser Agent sie liest. Darunter: **Urteil** (seine
  Antwort ist das Ergebnis), **Auftrag** (was er mit dem Gelesenen tun
  soll, `%s` = das Gelesene), früher/später, Entfernen.
- Ohne gewählten Agenten steht unten die **Hausordnung**: ein Text, der für
  alle Agenten dieses Ablaufs gilt.
- **Vorlage** setzt alles auf einmal: *alle gleichzeitig*,
  *nacheinander, jeder liest alles davor* oder *Kette, jeder nur den
  Vorgänger*.
- **Runden:** Bei mehr als einer Runde wird dieselbe Frage erneut gestellt;
  Schluss ist nach allen Runden, sobald das Urteil ein bestimmtes Wort
  enthält, oder wenn sich nichts mehr ändert.
- **Speichern und fragen** öffnet das Gespräch.

![Gespräch: oben der Ablauf live, unter jeder Antwort, was der Agent bekam](bilder/gespraech.png)

Im **Gespräch** steht der Ablauf live über dem Verlauf: welche Stufe gerade
schreibt, wer fertig ist, welcher Weg Daten trägt. Unter jeder Antwort steht,
was der Agent bekommen hat – ein Klick zeigt den Text, der wirklich an den
Dienst ging.

## Anbieter: die Gespräche beim Dienst

Die Ansicht *Anbieter* zeigt die Gesprächsliste eines Dienstes so, wie sie
auf seiner Webseite steht. Ein Gespräch öffnen, darin weiterschreiben oder
ein neues beginnen – wie im Browser.

| Dienst | Was geht |
|---|---|
| claude.ai, chat.deepseek.com, gemini.google.com | lesen und schreiben |
| chatgpt.com, grok.com | nur lesen – zum Senden verlangen die Seiten einen Wert, den nur ihr eigenes JavaScript im Browser berechnet |

Hört ein Dienst auf zu antworten, ist meist die Browsersitzung abgelaufen:
im Browser neu anmelden und den Zugang neu einrichten. Das Fenster sagt, wenn
es daran liegt.

## Speicher (Gedächtnis)

Ein Speicher ist eine Sammlung von Textstücken. Ein Agent, der ihn benutzt,
bekommt vor jeder Frage die passendsten Stücke mit; mit Schreibrecht legt er
seine Antworten selbst ab. Die Suche vergleicht Wörter und Zeichenmuster,
nicht die Bedeutung – „Auto" findet „Fahrzeug" nicht.

## Kommandozeile

Alles aus dem Fenster geht auch ohne Fenster:

```
gpb-browser --modell=claude-web "Frage"          eine Frage an einen Dienst
gpb-browser --dialog --modell=claude-web         Gespräch im Terminal
gpb-browser --chat=rat "Frage"                   einen gespeicherten Ablauf fragen
gpb-browser web einrichten --aus=cookies.txt     Zugänge einrichten
gpb-browser web pruefen                          was in den Zugängen fehlt
gpb-browser agent hilfe | chat hilfe | speicher hilfe
gpb-browser --hilfe                              alle Optionen
```

## Lizenz

CC BY-NC-SA 4.0 mit Sonderklauseln – siehe [LIZENZ.md](LIZENZ.md). Frei für
Forschung, Lehre und private Nutzung; kommerzielle Nutzung nur nach
Absprache.
