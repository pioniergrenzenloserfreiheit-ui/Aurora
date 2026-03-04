# GPT-5.2 / ChatGPT-Aurora in der Praxis

## Text- und Sprachfunktionen mit OpenAI-APIs integrieren – strukturiert, validiert, produktionsreif

## Kontext & Ziel

Dieses Dokument richtet sich an Entwickler:innen, die mit OpenAI-APIs belastbare Text- und Sprachfunktionen umsetzen möchten — vom schnellen Prototyp bis zum auditierbaren Produktivsystem. Abgedeckt werden: Textgenerierung (Responses API), Text-to-Speech (TTS), Speech-to-Text (STT), Echtzeit-Streaming, Audioformate, Prompt-Engineering, VS-Code-Workflows sowie Compliance/Governance.

**Leitidee (produktionstauglich):**
Du baust nicht nur eine Demo, sondern eine Pipeline, die im Betrieb stabil bleibt: klare Verträge (Inputs/Outputs), definierte Grenzen (Limits), reproduzierbares Verhalten (Versionierung/Evals) und saubere Pflichtenlage (Compliance/Secrets).

---

## Normativität (auditierbar)

Damit zwingende Anforderungen und Best Practices nicht vermischt werden, gelten folgende Marker:

- **MUSS** = zwingende Anforderung (Compliance, Security, Betriebssicherheit, Datenintegrität)
- **SOLLTE** = starke Empfehlung (Qualitäts- und Risiko-Reduktion)
- **KANN** = optional, Use-Case-abhängig (Komfort, Optimierung, Erweiterungen)

---

## 1) Textgenerierung mit der Responses API (empfohlener Einstieg)

Die Responses API ist der bevorzugte Startpunkt für neue Integrationen, da sie moderne Text- und strukturierte Ausgabe-Workflows konsistent abbildet (freie Texte, Transformationsaufgaben, strukturierte Outputs wie JSON).

### Prinzip

Instruktion/Prompt → Modell → freie oder strukturierte Antwort (z. B. JSON).

### Typische Outputs

- Fachtexte, Zusammenfassungen, Transformationen (z. B. „Markdown → Outline“)
- Marketing-/Kreativtexte
- Code (JS/TS/Python/cURL)
- Strukturierte Daten für nachgelagerte Systeme (Parser, Pipelines, Datenbanken)

### Für Produktion

- Du **SOLLTEST** Modellversionen/Snapshots pinnen, sofern dein Stack das unterstützt (Drift-Reduktion).
- Du **SOLLTEST** Evals/Regressionstests für Prompt- und Modelländerungen etablieren.
- Du **SOLLTEST** Prompts, Releases und Entscheidungen versionieren (Audit-Trail).

---

## 2) Text-to-Speech (TTS) – Speech-Endpunkt

### 2.1 Zweck

Der Speech-/TTS-Endpunkt erzeugt gesprochene Audiodaten aus Text. Typische Use-Cases: Vorlesen von Artikeln/Dokumentation, mehrsprachige Audioausgabe, Streaming-Audio mit geringer wahrgenommener Latenz.

### 2.2 Mindestparameter (Pflicht)

Eine TTS-Anfrage **MUSS** mindestens enthalten:

- `model`
- `input` (zu sprechender Text)
- `voice` (integrierte oder benutzerdefinierte Stimme)

### 2.3 Optionale Parameter (modell-/endpointabhängig)

Folgende Felder **KÖNNEN** existieren, sind aber modell-/endpoint-/zeitabhängig. Du **SOLLTEST** ihre Verfügbarkeit immer gegen die jeweils aktuelle API-Reference prüfen, bevor du sie produktiv freigibst:

- `instructions` (Sprechstil: Tonfall, Emotion, Tempo, Intonation, Akzent, Effekte wie Flüstern)
- `response_format` (z. B. `mp3`, `wav`, `opus`, `flac`, `aac`, `pcm`)
- weitere Parameter (z. B. Geschwindigkeit/Streaming-Optionen) nur nach Verifikation

### 2.4 Harte Limits (Design-Gate)

**Wichtig:** Der TTS-Endpunkt hat typischerweise ein Request-Limit pro Eingabetext (z. B. maximale Zeichen-/Tokenlänge). Du **MUSST** daher bei längeren Inhalten segmentieren/chunken und die Assembly (Zusammenführung) explizit designen.

### 2.5 Compliance

- Du **MUSST** Endnutzer:innen transparent informieren, dass das Audio KI-generiert ist.
- Du **SOLLTEST** je Rechtsraum zusätzliche Kennzeichnungs-/Einwilligungspflichten früh klären (nicht erst kurz vor Launch).

---

## 3) Modelle & Stimmen (TTS)

### 3.1 Häufig genutzte TTS-Modelle

Im Projektkontext werden u. a. diese Modelle genannt:

- `gpt-4o-mini-tts` (ausgewogener Allrounder)
- `tts-1` (niedrigere Latenz, tendenziell geringere Qualität)
- `tts-1-hd` (höhere Qualität)

**Hinweis (zeit-/accountabhängig):** Welche Modelle dir tatsächlich verfügbar sind, kann sich nach Account/Region/Release ändern. Du **SOLLTEST** Modell- und Parameterverfügbarkeit vor jeder Produktionseinführung verifizieren.

### 3.2 Integrierte Stimmen (built-in voices)

Beispielhafte Stimmen (modellabhängig):
`alloy, ash, ballad, coral, echo, fable, nova, onyx, sage, shimmer, verse, marin, cedar`

**Wichtig**

- Stimmenverfügbarkeit **KANN** je nach Modell und Realtime-Pfad variieren.
- Stimmen sind derzeit häufig für Englisch optimiert; du **SOLLTEST** bei Nicht-Englisch Prosodie/Aussprache/Namen/Zahlen gezielt testen.

---

## 4) Echtzeit-Streaming & Latenz

### 4.1 Streaming-Prinzip

Audio kann chunkweise ausgegeben werden (Streaming), sodass die Wiedergabe starten kann, bevor die gesamte Ausgabe fertig ist.

### 4.2 Formatwahl nach Ziel

- **WAV**: unkomprimiert, gut für niedrige Latenz ohne Dekodierungsaufwand
- **PCM**: Rohsamples (ohne Header), gut für direkte Pipeline-Verarbeitung; Details (z. B. 24 kHz, 16-Bit, Little-Endian) **MÜSSEN** in der Verarbeitung berücksichtigt werden
- **Opus**: effizient für Streaming/Realtime
- **MP3/AAC**: breite Geräte-/Plattformkompatibilität
- **FLAC**: verlustfrei für Archivierung/Nachbearbeitung

**Faustregel:** Du **SOLLTEST** das Format nach Latenzbudget, Bandbreite, Zielclient und Post-Processing auswählen.

---

## 5) Speech-to-Text (STT)

### 5.1 Zweck

STT wandelt Audio in Text um. Typische Outputs reichen von einfachem Volltext bis zu segmentierten Ergebnissen (z. B. Zeitstempel), je nach Modell/Endpunkt/response_format.

### 5.2 Endpunkte

Im Projektkontext werden Transkription und Übersetzung als zentrale STT-Funktionen beschrieben.

### 5.3 Upload-Limits & Formate

- Max. Uploadgröße pro Datei: **25 MB** (größere Dateien segmentieren)
- Gängige Inputformate: `mp3`, `mp4`, `wav`, `webm` (und je nach Endpunkt weitere)

### 5.4 Output-Formate (robust formuliert)

- Du **SOLLTEST** Outputformate nach Use Case wählen (z. B. `json` für Verarbeitung, `text` für einfache Nutzung, `srt`/`vtt` für Untertitel).
- Du **MUSST** `response_format` als modell-/endpointabhängig behandeln und vor Release gegen die aktuelle API-Reference verifizieren.

### 5.5 Best Practices

- Du **SOLLTEST** lange Audios segmentieren (Qualität/Robustheit/Parallelisierung).
- Du **SOLLTEST** Vorverarbeitung (Rauschreduktion, VAD) nutzen, wenn die Qualität schwankt.
- Du **SOLLTEST** Retries, Timeouts, Rate-Limit-Handling und Monitoring einplanen.

---

## 6) VS Code + Codex-gestützte Entwicklung

### 6.1 VS Code als vollständige API-Werkbank

Eine vollständige Entwicklung in VS Code ist praxisnah möglich: lokale Runtime (Node/Python), REST-Tests, Debugging (`launch.json`), Secret-Handling (`.env`, Secret Stores, CI-Secrets), automatisierte Tests (Jest/Pytest).

### 6.2 Rolle einer Codex-/KI-Erweiterung

Eine Codex-/KI-Erweiterung kann Scaffolding, Refactoring, Snippets und Tests beschleunigen, ersetzt aber nicht die eigentliche Cloud-API-Ausführung.

**Security-Hinweis:** API-Keys **MÜSSEN** aus Repos herausgehalten und über Secret-Mechanismen verwaltet werden.

---

## 7) Prompt Engineering & Stabilität

**Kernprinzip:** klare, testbare Anweisungen statt impliziter Annahmen.

**Empfehlungen**

- Du **SOLLTEST** Rollen/Instruktionen präzise strukturieren.
- Du **SOLLTEST** Prompts versionieren und Evaluationsfälle („Golden Sets“) pflegen.
- Du **SOLLTEST** Fakten, Annahmen und Policies sauber trennen, um auditfähig zu bleiben.

---

## 8) Benutzerdefinierte Stimmen (Custom Voices)

Custom Voices sind typischerweise nur für berechtigte Accounts verfügbar.

**Typischer Ablauf**

1. Einwilligungsaufnahme (vorgegebener Text)
2. Beispielaufnahme
3. Freigabe/Voice-ID
4. Nutzung der Voice-ID im `voice`-Parameter

**Typische Restriktionen (können sich ändern)**

- Begrenzte Anzahl pro Organisation (Beispiel: 20)
- Längenbegrenzung der Aufnahmen (Beispiel: 30 Sekunden)
- Erlaubte Uploadformate (z. B. `mpeg`, `wav`, `ogg`, `aac`, `flac`, `webm`, `mp4`)

**Consent-Gate:** Der Consent-Text **MUSS** exakt dem offiziellen Wortlaut entsprechen; Abweichungen führen häufig zum Fehlschlag.

---

## 9) Sprachunterstützung

Viele Sprachen werden unterstützt; die wahrgenommene Qualität hängt aber stark von Stimme, Sprache, Tempo und Texttyp ab. Für nicht-englische Inhalte **SOLLTEST** du einen QA-Pass einplanen (native Speaker, Domainbegriffe, Namen, Zahlen).

---

## 10) Produktions-Empfehlungen (Stabilität, Sicherheit, Qualität)

### Stabilität

- Modell-/Prompt-Versionierung **SOLLTE** Standard sein.
- Eval-Gates vor Releases **SOLLEN** Regressionen abfangen.
- Monitoring für Latenz, Fehler, Kosten, Qualität **SOLLTE** früh stehen.

### Sicherheit & Compliance

- API-Keys **MÜSSEN** aus Repos raus (Secrets, Rotation, Least Privilege).
- KI-Kennzeichnung bei TTS **MUSS** umgesetzt werden.

### Qualität

- Für lange TTS-Inputs **MUSST** du Chunking einplanen (TTS-Inputlimits).
- Du **SOLLTEST** reproduzierbare Testdaten (Golden Sets) pflegen.

---

## 11) Entscheidungsregel bei Doku-Konflikten (Guide vs API-Reference)

Wenn Guide und API-Reference auseinanderlaufen:

1. Für Schema/Parameter/Limits gilt: **API-Reference gewinnt** (Engineering-Vertrag).
2. Für Best Practices und Architektur: **Guides** (konzeptionell).
3. Bei Widerspruch: **fail-closed + Feature-Flag** (kein ungeprüfter Parameter im Produktivpfad).
4. Aktivierung **MUSS** durch Staging-Verifikation abgesichert werden (kleiner Canary-Test, dokumentiertes Ergebnis, Versionierung).
5. Entscheidung **SOLLTE** als Decision Log festgehalten werden (Quelle A vs B, Regel angewandt, Verifikationsergebnis).

---

## 12) Praxis: „Text rein → Audiodatei raus“ als robuste Pipeline

### 12.1 Input-Schicht (Markdown/Text)

Du **SOLLTEST** eine Narrations-Policy definieren:

- Überschriften: Kapitelansage (KANN) + kurze Pause (KANN)
- Links/URLs: weglassen (KANN) oder „Link in den Shownotes“ (KANN)
- Codeblöcke: weglassen (SOLLTE) oder zusammenfassen (SOLLTE)
- Tabellen: zusammenfassen (SOLLTE)

### 12.2 Segmentierung (Chunking)

Du **MUSST** bei langen TTS-Inputs segmentieren (weil TTS-Inputlimits existieren). Bewährte Heuristik (SOLLTE):

1. nach Absätzen splitten,
2. dann nach Sätzen,
3. Fallback: harte Grenze.

### 12.3 TTS-Erzeugung pro Chunk

Pro Chunk gilt:

- `model`, `voice` und `input` **MÜSSEN** gesetzt sein.
- `response_format` **SOLLTE** bewusst gewählt werden (Kompatibilität vs. Latenz vs. Bearbeitung).
- `instructions` **KANN** genutzt werden (modellabhängig).

### 12.4 Assembly: eine Audiodatei

Du **SOLLTEST** die Merge-Strategie früh entscheiden:

**Strategie A (SOLLTE für Robustheit): WAV/PCM-First**

- pro Chunk `wav` oder `pcm` erzeugen
- sauber zusammenführen
- optional transcodieren zu MP3/AAC für Distribution

**Strategie B (KANN): direkt MP3/AAC**

- sofort kompatibel
- Assembly/Concat ist tool-sensitiver; erfordert disziplinierte Toolchain

### 12.5 Metadaten & Audit

Du **SOLLTEST** pro Job loggen:

- `trace_id`, Zeitstempel, Modell, Voice, Format
- Chunk-Count, Dauer, Fehler, Retries
- Prompt-/Release-Version (Audit-Trail)

---

## Kurzfazit

Mit Responses API (Text), Speech/TTS und STT lassen sich skalierbare Sprach- und Textsysteme bauen. Produktionsreife entsteht vor allem durch:

- saubere Modell-/Formatwahl
- robuste Chunking-, Streaming- und Fehlertoleranzstrategien
- Prompt-Versionierung + Evals
- klare Compliance-/Governance-Prozesse
- durchgängiges Monitoring

So entsteht ein System, das nicht nur funktioniert, sondern verlässlich, reproduzierbar und verantwortbar betrieben werden kann.

---

## Kanonische Referenzen (ohne Tracking, produktneutral)

```text
OpenAI Docs (Guides):
- https://developers.openai.com/api/docs/guides/text-to-speech
- https://developers.openai.com/api/docs/guides/speech-to-text
- https://developers.openai.com/api/docs/guides/realtime

OpenAI API Reference (Audio):
- https://developers.openai.com/api/reference/audio
- https://developers.openai.com/api/reference/resources/audio/subresources/speech/methods/create
- https://developers.openai.com/api/reference/resources/audio/subresources/transcriptions/methods/create
- https://developers.openai.com/api/reference/resources/audio/subresources/translations/methods/create

Fallback-Regel (Link-Resilienz):
- Wenn einzelne Subresource-Links umziehen oder 404 liefern: starte bei https://developers.openai.com/api/reference/audio und navigiere von dort zu „speech“, „transcriptions“ oder „translations“.
- Für Guides analog: starte bei https://developers.openai.com/api/docs/guides/ und wähle den passenden Guide (Text-to-Speech, Speech-to-Text, Realtime).
```

## Repo-/Ops-Hinweise

- Die zentrale Dokumentation **SOLLTE** als exakt `README.md` (Uppercase) im Repo-Root liegen, damit Rendering und CI auf Linux konsistent bleiben.
- Für CI **SOLLTEST** du mindestens folgende Dokument-Checks vorsehen:
  - Existenz/Nicht-Leerheit der `README.md`
  - optionaler Markdown-Lint/Link-Checker/Spellcheck
  - optionaler Tracking-Parameter-Check (z. B. keine `utm_`-Parameter in Referenz-URLs)

## Checks

⚠️ Keine Shell- oder Test-Kommandos ausgeführt (statische Dokument-QA, reine Text-/Strukturarbeit).
