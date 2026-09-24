# C99 AI-Hypervisor – Anleitung

Das Programm stellt eine Frage an mehrere KI-Dienste und verbindet sie zu
einem **Ablauf**: Agenten in Stufen, jeder liest, was du ihm zuweist, einer
kann das Urteil sprechen. Dienste werden über die **Weboberfläche mit deiner
Browsersitzung** angesprochen (claude.ai, chat.deepseek.com,
gemini.google.com, chatgpt.com, grok.com) oder über eine **API mit Schlüssel**
(Anthropic, OpenAI, DeepSeek, OpenRouter, Ollama).

![Ansicht Ablauf](bilder/ablauf.png)

[Start](#1-start) · [Begriffe](#2-begriffe) · [Zugang](#3-zugang) ·
[Agenten](#4-agenten) · [Ablauf](#5-ablauf) · [Gespräch](#6-gespräch) ·
[Anbieter](#7-anbieter) · [Speicher](#8-speicher) ·
[Kommandozeile](#9-kommandozeile) · [Fehler](#10-fehler) ·
[Grenzen](#11-grenzen) · [Lizenz](#12-lizenz)

---

## 1. Start

| Ordner | Programme |
|---|---|
| `windows/` | `gpb-gui.exe` (Fenster), `gpb-browser.exe` (Kommandozeile). Keine Installation, keine DLLs; TLS über Windows (Schannel). |
| `linux-x86_64/` | `gpb-gui`, `gpb-browser`. Braucht X11 mit Xft und OpenSSL 3. |

`gpb-gui` starten. Beim ersten Start zeigt die Ansicht *Gespräch* die drei
Schritte mit je einem Knopf: Zugang einrichten → Agent anlegen → Gespräch
zusammenstellen.

**Datenordner.** Alles, was du einrichtest, liegt im Anwendungsordner des
Systems:

| System | Ordner |
|---|---|
| Windows | `%APPDATA%\gpb-browser` = `C:\Users\<du>\AppData\Roaming\gpb-browser` |
| Linux/BSD | `~/.local/share/gpb-browser` (oder `$XDG_DATA_HOME/gpb-browser`) |
| macOS | `~/Library/Application Support/gpb-browser` |

| Unterordner | Inhalt |
|---|---|
| `keks` | Zugänge zu den Weboberflächen (deine Sitzungs-Cookies) |
| `agenten` | Agenten, eine JSON-Datei je Agent |
| `chats` | Abläufe |
| `speicher` | Gedächtnis-Sammlungen |
| `faden` | gemerkte Gespräche beim Anbieter |
| `verlauf` | Gesprächsverläufe |

Die Fußzeile des Fensters zeigt den Pfad. Ein anderer Ort:
`--daten=<verzeichnis>` oder die Umgebungsvariable `GPB_DATEN`.

`keks` ist deine Anmeldung bei den Diensten. Wer den Ordner hat, ist dort
du. Nicht weitergeben, nicht in eine Cloud-Synchronisation legen.

## 2. Begriffe

| Begriff | Bedeutung |
|---|---|
| **Endpunkt** | Ein Dienst, so wie das Programm ihn erreicht. `claude-web` = claude.ai über die Browsersitzung, `claude` = Anthropic-API mit Schlüssel. Endpunkte sind fest eingebaut. |
| **Agent** | Endpunkt + Rolle (Master-Prompt), wahlweise Modell, Speicher, „Gespräch beim Anbieter fortsetzen“. |
| **Ablauf** | Agenten in Stufen mit Verdrahtung: wer läuft wann, wer liest was, wer das Urteil hat, wie viele Runden. |
| **Gespräch** | Ein Ablauf in Betrieb: Frage, Antworten, Bericht. |
| **Speicher** | Textstücke, die ein Agent vor jeder Frage passend dazubekommt. |
| **Anbieter** | Ansicht mit den Gesprächen, die beim Dienst selbst liegen. |

## 3. Zugang

### Was das Programm aus dem Browser braucht

Bei der Anmeldung im Browser legt der Dienst **Cookies** ab. Das Programm
schickt an dieselben Adressen dieselben Cookies und dieselbe Browserkennung
(**User-Agent**) wie der Browser. Nötig sind:

1. **Die Cookies des Dienstes:**

   | Endpunkt | Adresse | Pflicht | dazu |
   |---|---|---|---|
   | `claude-web` | claude.ai | `sessionKey` | `cf_clearance` |
   | `deepseek-web` | chat.deepseek.com | `userToken` oder `ds_session_id` | Kopfzeile `authorization: Bearer …` – steht in keinem Cookie-Export, siehe Weg B |
   | `gemini-web` | gemini.google.com | `__Secure-1PSID` **und** `__Secure-1PSIDTS` | `__Secure-1PSIDTS` erneuert Google laufend; ein alter Export gilt nach Stunden nicht mehr |
   | `chatgpt-web` | chatgpt.com | `__Secure-next-auth.session-token` | `cf_clearance` |
   | `grok-web` | grok.com | `sso` (und `sso-rw`) | `cf_clearance`; Cookies von **grok.com**, nicht von x.com |

2. **Den User-Agent deines Browsers.** Cloudflare bindet `cf_clearance` an
   IP-Adresse, User-Agent und TLS-Fingerabdruck. Mit einem anderen
   User-Agent kommt HTTP 403.

Beides steht in der **Zugangsdatei** je Dienst: `keks\<endpunkt>.txt` im
Datenordner, unter Windows also
`%APPDATA%\gpb-browser\keks\claude-web.txt`. Das Programm zeigt Cookie-Werte
nirgends an und schreibt sie in keine Meldung und kein Protokoll. Nur die
Cookies der Domain des jeweiligen Dienstes werden gelesen; andere Zeilen in
der Datei werden übergangen.

### Weg A: alle Dienste aus einer Cookie-Datei

1. Im Browser bei den Diensten anmelden.
2. Cookies im **Netscape-Format** exportieren, z. B. mit der Erweiterung
   „Get cookies.txt LOCALLY“ (Chrome, Firefox). **Alle** Domains
   exportieren; jeder Dienst bekommt aus der Datei seine eigenen Zeilen.
3. User-Agent holen: **F12** → **Konsole** → `navigator.userAgent` eingeben →
   Text ohne Anführungszeichen kopieren.
4. Fenster, Ansicht **Endpunkte**, Karte **Alle auf einmal einrichten**:
   User-Agent in das Feld eintragen, dann Dateipfad eintragen und
   **Aus Datei** – oder Dateiinhalt in die Zwischenablage kopieren und
   **Aus Zwischenablage**.
5. Die Liste darunter zeigt je Dienst, was er bekommen hat und was fehlt.

Kommandozeile:

```
gpb-browser web einrichten --aus=cookies.txt --user-agent="Mozilla/5.0 (…) Chrome/130.0.0.0 …"
gpb-browser web pruefen
```

Den User-Agent nachträglich für alle Dienste setzen, aus einer beliebigen
„Als cURL kopieren“-Zeile desselben Browsers:
`gpb-browser web kopf --aus=anfrage.txt`. Cookies bleiben unverändert.

**DeepSeek:** Der Zugangstoken liegt im localStorage des Browsers, nicht in
einem Cookie, und geht als Kopfzeile `authorization: Bearer …` mit. Eine
Cookie-Datei enthält ihn nicht. Für DeepSeek zusätzlich Weg B mit einer
Anfrage unter `/api/`.

### Weg B: ein Dienst aus einer kopierten Anfrage

Bringt Cookies, User-Agent und Begleitzeilen genau so mit, wie der Browser
sie schickt.

1. Im Browser beim Dienst anmelden.
2. **F12** → **Netzwerk**. Ist die Liste leer: Seite neu laden.
3. Eine Anfrage an den Dienst selbst wählen (Domain claude.ai, chatgpt.com …).
   Bei DeepSeek eine mit `/api/` im Pfad, z. B. `/api/v0/users/current`.
4. Rechte Maustaste → **Kopieren** → **Als cURL kopieren** (Chrome/Edge unter
   Windows: „cmd“ oder „bash“, beide gehen).
5. Fenster, Ansicht **Endpunkte**, Zeile des Dienstes:
   **Aus Zwischenablage einrichten**.

Kommandozeile, mit dem kopierten Text in einer Datei:

```
gpb-browser web einrichten deepseek-web --aus=anfrage.txt
```

Ohne `--aus` liest der Befehl den Text vom Terminal (einfügen, dann leere
Zeile). Rohe Kopfzeilen oder die Ausgabe von `document.cookie` gehen auch,
enthalten aber keinen User-Agent.

### Prüfen

Im Fenster hat jede Endpunkt-Zeile einen Punkt: grün = eingerichtet (mit
Cookie-Zahl und Ablauf), rot = etwas fehlt oder ist abgelaufen, grau = nicht
eingerichtet. Die Kommandozeile prüft ohne eine Anfrage zu senden:

```
$ gpb-browser web pruefen claude-web

claude-web  (Claude (claude.ai, Browsersitzung))
  Adresse        https://claude.ai
  Zugang         C:\Users\du\AppData\Roaming\gpb-browser\keks\claude-web.txt
  Cookies        7 gelesen: sessionKey cf_clearance __cf_bm …
  Anmeldung      ✓ sessionKey
  cf_clearance   ✓ da (gilt nur fuer dieselbe IP und denselben User-Agent)
  User-Agent     ✓ derselbe wie im Browser
  Laeuft ab      in 13 Tage (fruehestes Cookie)
  → vollstaendig. Ob es DURCHKOMMT, entscheidet der TLS-Fingerabdruck.
```

`gpb-browser web pruefen` ohne Namen prüft alle fünf. Was fehlt, steht als
Satz da, z. B. `das Anmelde-Cookie "__Secure-1PSIDTS" fehlt`.

### Die Zugangsdatei

Textdatei. Gelesen werden Netscape-Zeilen
(`.claude.ai TRUE / TRUE 1790000000 sessionKey sk-…`) oder eine Zeile
`Cookie: sessionKey=…; cf_clearance=…`, dazu `User-Agent: …` und weitere
Kopfzeilen (`sec-ch-ua: …`, `authorization: Bearer …`). Die
Umgebungsvariable `GPB_<DIENST>_COOKIES` (z. B. `GPB_CLAUDE_COOKIES`) zeigt
auf eine andere Datei und hat Vorrang.

**Sitzungen laufen ab.** Anmelde-Cookies gelten Tage bis Wochen,
`cf_clearance` kürzer, `__Secure-1PSIDTS` Stunden. Bei „Sitzung gilt nicht
(mehr)“ oder „Anmeldeseite“: im Browser die Seite neu laden und den Zugang
neu einrichten. **Entfernen** in der Endpunkt-Zeile löscht einen Zugang.

### API-Schlüssel

Als Umgebungsvariable; eine Datei dafür gibt es nicht.

| Endpunkt | Variable | eigene Adresse (optional) |
|---|---|---|
| `claude` | `ANTHROPIC_API_KEY` | `GPB_CLAUDE` |
| `gpt` | `OPENAI_API_KEY` | `GPB_GPT` |
| `deepseek` | `DEEPSEEK_API_KEY` | `GPB_DEEPSEEK` |
| `openrouter` | `OPENROUTER_API_KEY` | `GPB_OPENROUTER` |
| `ollama` | – | `GPB_OLLAMA` (Standard 127.0.0.1:11434) |
| `lokal` | `GPB_LOKAL_KEY` | `GPB_LOKAL` (jeder OpenAI-kompatible Server) |

Windows: Systemsteuerung → System → Umgebungsvariablen, oder
`set ANTHROPIC_API_KEY=sk-…` in der Eingabeaufforderung vor dem Start. Die
Ansicht *Endpunkte* zeigt, ob die Variable gesetzt ist.

**Eigene Zertifizierungsstelle** (Firmen-Proxy, eigener Server): unter
Linux/macOS `GPB_CA_DATEI=<datei.pem>`; sie gilt zusätzlich zu den
Systemzertifikaten, die Prüfung bleibt an. Unter Windows die
Zertifizierungsstelle in die Zertifikatsverwaltung des Systems aufnehmen.

## 4. Agenten

Ansicht **Agenten** → **+ Neuer Agent**.

- **Name** – Buchstaben, Ziffern, `-`, `_`; wird zum Dateinamen.
- **Endpunkt** – der Dienst; die Liste zeigt, ob sein Zugang steht.
- **Modell** – bei APIs eine Liste; bei Weboberflächen meist „wählt der
  Dienst selbst“. `deepseek-web` mit `deepseek-reasoner` = DeepThink.
- **Master-Prompt** – die Rolle. Steht vor jeder Frage dieses Agenten in
  jedem Ablauf. Was nur für einen Ablauf gilt, gehört in den **Auftrag**
  (Abschnitt 5).
- **Speicher / Recht / geteilt** – Abschnitt 8.
- **Gespräch beim Anbieter fortsetzen** – nur Weboberflächen. Ohne Haken
  legt jeder Lauf beim Dienst ein neues Gespräch an. Mit Haken merkt sich
  der Agent das Gespräch (`faden`) und schreibt darin weiter.

## 5. Ablauf

Ansicht **Ablauf**. Oben einen gespeicherten Ablauf wählen oder **+ Neu**.
Der Graph ist der Ablauf.

```
 Frage ──► Stufe 1 ──► Stufe 2 ──► … ──► Ergebnis
           (alle darin gleichzeitig)          │
   ▲──────────── weitere Runde ───────────────┘
```

**Pfeile sind Datenwege.** Was ein Agent liest, bekommt er vollständig, mit
Namensschild („recherche sagt: …“). Am Knoten steht es: „liest: Frage +
recherche + gegenprobe“.

**Stufen.** Agenten in derselben Spalte laufen gleichzeitig; die Spalten
laufen nacheinander.

**Agenten dazunehmen.** **+ gleichzeitig** unter einer Stufe stellt einen
Agenten dazu (liest dasselbe wie die Nachbarn). **+ Agent** unter „neue
Stufe“ hängt eine Stufe an (liest Frage und alles davor).

**Ziehen.** Knoten in eine andere Spalte = läuft dort gleichzeitig.
Zwischen zwei Spalten oder auf „neue Stufe“ = neue Stufe; der Agent liest
Frage und alle Antworten davor. Eingänge, die danach unmöglich sind
(jemand liest einen, der später antwortet), fallen weg; die Meldung nennt
sie.

**Anklicken** wählt einen Agenten:

- An der Frage und an Agenten früherer Stufen erscheinen Anschlusspunkte.
  Klick schaltet um, ob der gewählte Agent das liest. Leuchtend = liest.
- Unten: **früher / später** (eine Stufe verschieben), **Urteil**,
  **Entfernen**, **Auftrag**.

**Auftrag.** Was dieses Glied mit dem Gelesenen tut, z. B. „Prüfe beide
Antworten auf Widersprüche und nenne die tragfähigere: %s“. `%s` = Stelle
des Gelesenen; ohne `%s` kommt es dahinter. Leer = nur das Gelesene.

**Hausordnung.** Ohne gewählten Agenten steht unten ein Text für alle
Agenten dieses Ablaufs, nach ihrer Rolle. Z. B. „Antworte auf Deutsch,
höchstens 200 Wörter.“

**Urteil.** Genau ein Agent kann es tragen. Seine Antwort ist das Ergebnis;
die Abbruchbedingung liest sie. Ohne Urteil: alle Antworten der letzten
Stufe.

**Vorlage** setzt die Verdrahtung auf einmal:

| Vorlage | Ergebnis |
|---|---|
| alle gleichzeitig | alle in Stufe 1, jeder liest die Frage |
| nacheinander, jeder liest alles davor | eigene Stufe je Agent; jeder liest Frage und alle Antworten davor |
| Kette, jeder nur den Vorgänger | jeder liest nur die Antwort des Vorgängers, der erste die Frage. Bei genau einer Quelle geht die Übergabe byte-genau über einen `.gpb`-Container mit Rundlauf-Beleg |

Danach lässt sich jeder Eingang einzeln ändern.

**Runden und Ende.** Bei mehr als einer Runde wird dieselbe Frage erneut
gestellt; jeder Agent behält seinen Verlauf und sieht die neuen Antworten
der anderen. Ende:

- nach allen Runden, oder
- **auf ein Wort**: wenn das Urteil (ohne Urteil: eine Antwort der letzten
  Stufe) das Wort enthält, z. B. `FERTIG`; Groß-/Kleinschreibung egal, oder
- **wenn Ruhe einkehrt**: wenn das Urteil gegenüber der Vorrunde gleich
  bleibt.

Die Rundenzahl ist immer die Obergrenze.

**Speichern und fragen** prüft, speichert und öffnet das Gespräch. Ein
Ablauf, der nicht laufen kann, wird mit Grund abgelehnt.

*Beispiel (Bild oben):* `recherche` (claude.ai) und `gegenprobe` (DeepSeek)
antworten gleichzeitig; `richter` (Gemini) liest Frage und beide Antworten,
Auftrag „Welche der beiden Antworten trägt? Begründe kurz. %s“, hat das
Urteil. Ergebnis = Antwort des Richters.

## 6. Gespräch

![Gespräch mit Ablauf live](bilder/gespraech.png)

Links die gespeicherten Abläufe, rechts das Gespräch. Frage unten
eingeben; mehrzeilig, ohne Längengrenze. **Enter** sendet, **Shift+Enter**
neue Zeile.

**Ablauf live.** Über dem Verlauf steht der Graph in Betrieb: aktive Stufe
hervorgehoben, je Agent *wartet / schreibt … 240 Token / fertig / Fehler*,
bei mehreren Runden „Runde 2 von höchstens 5“. **einklappen** lässt eine
Zeile stehen.

**Was jeder Agent bekam.** Unter jeder Antwort: „bekam: die Frage, die
Antwort von recherche und die Antwort von gegenprobe, mit Auftrag · 303
Zeichen“. **(zeigen)** öffnet den Text, der an den Dienst ging.

**Farben.** Blau = du, weiß = Antworten, grau = Hinweise und Bericht
(Stufen, Urteil, Grund des Endes, Token, Dauer), rot = Fehler.

**Kopieren** neben jeder Antwort: ihr Text in die Zwischenablage.
**Verlauf kopieren** (über dem Eingabefeld): das ganze Gespräch mit
Sprechernamen.

**Abbrechen.** Während einer Antwort wird **Senden** zu **Abbrechen**. Die
Teilantwort bleibt sichtbar, geht aber nicht in den Verlauf.

**Ablauf** (links) öffnet die Ansicht Ablauf; nach dem Speichern gilt der
neue Aufbau ab der nächsten Frage. **Löschen** entfernt den gespeicherten
Ablauf nach einem zweiten Klick.

**Werkzeuge …** (links):

- **Antwort als .gpb** – letzte Antwort als `.gpb`-Container schreiben
  (mit Rundlauf-Beleg) oder einen Container in den Verlauf laden.
- **Verlauf laden / sichern** – Verlauf jedes Agenten als `.jsonl`.
- **Kontextfenster** – ältere Züge wegfallen lassen, in Bytes oder Token
  (Token erst, wenn der Dienst eine Tokenzahl gemeldet hat). Standard aus.
  Jeder Wegfall wird im Verlauf gemeldet.
- **Vergleichen** – bei mehreren Antworten ein Bericht über Invarianten der
  Texte (Substanz, Token, Struktur). Keine Bewertung des Inhalts.
- **Messlauf** – drei feste Fragen an die Agenten des offenen Chats, danach eine
  Tabelle: Antworten, Bruchstücke, Zeit bis zum ersten und letzten Stück,
  Größe als `.gpb`. Läuft im Hintergrund, das Fenster bleibt bedienbar.
- **Übergabe-Hygiene** – bei Ketten: prüft den weitergereichten Text auf
  Steuerzeichen, Bidi-Overrides und Rollenmarken wie `<|im_start|>`;
  *streng* entfernt sie. Kein Schutz gegen Anweisungen im Text.

## 7. Anbieter

Ansicht **Anbieter**: Dienst wählen, **Neu laden** holt die Gesprächsliste
des Dienstes. Gespräch anklicken zeigt es; unten weiterschreiben (Enter
sendet, Shift+Enter neue Zeile), **+ Neues** beginnt eins. Ab sieben
Gesprächen erscheint über der Liste ein Suchfeld, es filtert nach Namen.
Das Gespräch, in das ein offener Ablauf schreibt, trägt die Marke
„Gespräch“.

| Dienst | Was geht |
|---|---|
| claude.ai, chat.deepseek.com, gemini.google.com | Liste, lesen, weiterschreiben, neu beginnen |
| chatgpt.com, grok.com | Liste, lesen. Senden nicht: beide verlangen einen Wert, den nur ihr JavaScript im Browser berechnet (ChatGPT Proof-of-Work in WASM, Grok eine Sitzungskennung). Das Programm meldet das. |

Gespräche, die du hier öffnest, löscht das Programm nie. Aufgeräumt wird
nur, was es selbst angelegt hat.

## 8. Speicher

Ein Speicher ist eine Sammlung von Textstücken. Ein Agent, der ihn nennt,
bekommt vor jeder Frage die Stücke vorangestellt, die die meisten Wörter mit
der Frage teilen. Recht **schreiben**: der Agent legt seine Antworten darin
ab. **Geteilt**: alle Agenten mit diesem Namen sehen dieselbe Datei; sonst
hat jeder seine eigene.

Ansicht **Speicher**: **Neu** legt eine Sammlung an; Stück in den Kasten
schreiben und **Übernehmen**; **Suchen** (die Trefferliste zeigt, warum ein
Stück gefunden wurde); Stück anklicken zum Ändern oder Entfernen;
**Säubern** (Leeres und wortgleich Doppeltes weg); **Backen** (alles neu
rechnen). Ganze Dateien:
`gpb-browser speicher legen wissen @protokoll.md --quelle=protokoll`.

Verglichen werden Wörter (unabhängig von Groß-/Kleinschreibung und
Akzenten) und ein berechneter Merkmalsvektor; kein gelerntes Modell, kein
Download, auf jedem Rechner dasselbe Ergebnis. „Auto“ findet „Fahrzeug“
nicht. Große Stücke werden beim Ablegen geteilt.

## 9. Kommandozeile

`gpb-browser` kann alles, was das Fenster kann. stdout trägt nur
Antworttext, alles andere geht nach stderr.

```
gpb-browser --modell=claude-web "Frage"                 eine Frage
gpb-browser --dialog --modell=claude-web                Gespräch im Terminal (:hilfe)
gpb-browser --chat=rat "Frage"                          gespeicherten Ablauf fragen
gpb-browser --modell=claude --modell=gpt --konsens "F"  zwei Modelle, mit Vergleich
gpb-browser "--kette=claude>deepseek" --beleg "Frage"   Kette mit Rundlauf-Beleg
gpb-browser --modell=claude --fallback=gpt "Frage"      Ausweichen bei 429/5xx
gpb-browser --modell=ollama "Frage" | gpb-browser --modell=gpt

gpb-browser web liste | einrichten | pruefen | kopf | gespraeche | holen | entfernen
gpb-browser agent liste | zeigen | neu | aendern | loeschen
gpb-browser chat liste | zeigen | neu | aendern | loeschen
gpb-browser speicher liste | zeigen | legen | suchen | weg | aendern | backen | saeubern
gpb-browser --hilfe          gpb-browser agent hilfe   (ebenso chat, speicher, web)
```

Im Dialog: `:ende`, `:neu`, `:status`, `:verlauf`, `:kuerzen <n>`, `:paste`,
`:gpb <datei>`, `:hilfe`. Ctrl-C bricht eine laufende Antwort ab.

Ablauf auf der Kommandozeile (entspricht dem Beispiel in Abschnitt 5):

```
gpb-browser agent neu recherche --endpunkt=claude-web --prompt=@rolle.txt
gpb-browser agent neu gegenprobe --endpunkt=deepseek-web
gpb-browser agent neu richter --endpunkt=gemini-web --speicher=wissen --recht=lesen
gpb-browser chat neu rat --agent=recherche --agent=gegenprobe --agent=richter \
    --stufe=richter:1 --hoert=richter:recherche,gegenprobe --schiedsrichter=richter
gpb-browser chat aendern rat --auftrag="richter:Welche Antwort trägt? %s"
gpb-browser chat aendern rat --runden=5 --ende=wort --ende-wort=FERTIG
gpb-browser --chat=rat "Frage"
```

`--ablauf=parallel|runde|reihe` setzt eine Vorlage (alle gleichzeitig /
nacheinander mit allem davor / Kette). `agent aendern <name> --faden=ja`
entspricht „Gespräch beim Anbieter fortsetzen“.

`--daten=<verzeichnis>` vor jedem Befehl wählt einen anderen Datenordner.
`web gespraeche <dienst> [<kennung>]` listet die Gespräche beim Dienst bzw.
zeigt eins. `web holen <dienst> <pfad>` ruft eine Adresse des Dienstes mit
deiner Sitzung ab (`web holen claude-web /api/organizations`).

## 10. Fehler

| Meldung | Ursache | Abhilfe |
|---|---|---|
| „nicht eingerichtet“ | kein Zugang für diesen Dienst | Abschnitt 3 |
| „das Anmelde-Cookie … fehlt“ | Export ohne diese Domain oder im Browser nicht angemeldet | anmelden, neu exportieren |
| „keine User-Agent-Zeile“ | Cookie-Datei ohne Browserkennung | User-Agent eintragen (Weg A, Schritt 3) oder `web kopf` |
| „HTTP 401 — die Sitzung gilt nicht (mehr)“ / „Anmeldeseite“ | Cookies abgelaufen oder im Browser abgemeldet | im Browser neu laden, Zugang neu einrichten |
| „HTTP 403 … Bot-Prüfung“ | `cf_clearance` fehlt, ist abgelaufen oder passt nicht zu User-Agent/IP (anderes Netz, VPN) | Weg B aus demselben Netz; bleibt es, ist es die TLS-Bindung (Abschnitt 11) |
| „HTTP 302 → …“ | der Dienst leitet um; die Adresse in der Kopfdatei gilt nicht mehr | Meldung nennt das Ziel |
| DeepSeek: „Missing Token“ / „authorization fehlt“ | Token liegt nicht in einem Cookie | Weg B mit `/api/`-Anfrage |
| Gemini: Anmeldeseite trotz frischem Export | `__Secure-1PSIDTS` veraltet oder fehlt | Gemini im Browser öffnen, sofort neu exportieren |
| ChatGPT/Grok: Lesen geht, Senden nicht | Browser-Rechenwert | nicht behebbar, Abschnitt 11 |
| Agent im Graphen rot, Verlauf „Fehler · name“ | der Dienst hat die Antwort abgelehnt; die Stufe läuft mit den anderen zu Ende | rote Meldung nennt den Grund |
| Senden bleibt „Abbrechen“ | ein Dienst antwortet nicht | Abbrechen; Leerlauf-Frist 60 s |
| Fenster startet nicht (Linux) | X11/Xft oder OpenSSL fehlt | `libxft2`, `libssl3` installieren |

Fehlersuche: `GPB_ROH=<datei>` schreibt die Antwort weg, wenn ein
Vorlaufschritt scheitert; `GPB_ROH_ANFRAGE=<datei>` hängt jede gesendete
Anfrage an eine Datei an. **Diese Datei enthält deine Cookies im Klartext.**

## 11. Grenzen

- Senden bei chatgpt.com und grok.com (Abschnitt 7).
- TLS-Fingerabdruck eines Browsers. Cookies und Kopfzeilen werden
  gespiegelt; die TLS-Erweiterungen setzt die TLS-Bibliothek des Systems.
  Bindet Cloudflare `cf_clearance` daran, wird die Sitzung abgelehnt.
- Bilder, Dateianhänge, Werkzeugaufrufe. Nur Text.
- Bedeutungssuche im Speicher (Abschnitt 8).
- Kosten. Token werden gezählt und im Bericht genannt, nicht umgerechnet.

## 12. Lizenz

CC BY-NC-SA 4.0 mit Sonderklauseln, siehe [LIZENZ.md](LIZENZ.md): frei für
Forschung, Lehre und private Nutzung; Bearbeitungen bleiben offen;
kommerzielle Nutzung nur nach Absprache. Kontakt: silvano19911@gmail.com.
