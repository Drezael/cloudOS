# R-007 – Quellenlage der historischen SDK-Fragen Q001–Q033

> **Stand:** 16.11.2026 · **Status:** Fachliche Kurzantworten rekonstruiert; **kein** Nachweis sämtlicher Originalfrage- und Optionswortlaute.  
> **Abgrenzung:** Ausschließlich SDK-Fragen in `docs/APP_FRAMEWORK_SDK.md`, **nicht** die eigenständigen 74 Fragen aus `docs/API_CONTRACTS.md`.  
> **Verbindliches Register:** [02-decision-register.md](02-decision-register.md).

## Prüfergebnis

Im früheren SDK-Register standen 32 historische Fragen (Q001–Q028 und Q030–Q033) als `OFFEN_QUELLNACHWEIS`, obwohl sich in früheren Gesprächszusammenfassungen für **alle Q001–Q033** zumindest ein fachlicher Kurzinhalt wiederfinden lässt. Die Quelle ist **kein vollständig zitierbarer Originalmitschnitt**. Deshalb werden nur nachvollziehbare **Kurzbeschlüsse**, keine neu erfundenen Antwortbuchstaben oder exakten Originalfragen eingetragen. Wo frühere Nachrichten eine direkte Nutzerwahl enthalten, kennzeichnet das Register sie `BESTAETIGT_IM_VERLAUF`; andere bleiben `REKONSTRUIERT`.

**Wichtige Unterscheidungen:** Q010 erlaubt vertragsbasierte Mock-Szenarien für Entwicklertests, aber Q105 verbietet eine eigenständige, als cloudOS-Instanz ausgegebene SDK-Simulation. Q016 betrifft die **eigenen** cloudOS-Oberflächen, während Q028 das öffentliche SDK frameworkneutral hält. Q018/Q019 konkretisieren Pflichtfähigkeit und Isolation des anspruchsvollen Rust-WASM-Profils; dies erzeugt keinen stillen WebGL-/Thread-Fallback. Q030/Q031 ersetzen nie direkt den Network-/Capability-Broker.

## Quellenkritische Prüftabelle

| SDK-Q | Thema | Sicher verfügbare fachliche Kurzfassung | Nachweis | Provenienzqualität |
|---|---|---|---|---|
| Q001 | Modularer SDK-Kern | Zentrales SDK mit modularen, bedarfsorientierten Komponenten | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q002 | Initialisierung | Zweistufige Initialisierung mit App-Kontext und lazy geladenen SDK-Modulen | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q003 | Fehlersemantik | Sprachtypische Fehlerbehandlung mit gemeinsamem, typisiertem Fehlermodell | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q004 | App Lifecycle | Hybrides Lifecycle-System mit Hooks und Events | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q005 | SDK-Zustand | Hybride, versionierte Zustandsverwaltung | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q006 | Laufzeitereignisse | Typisierte Lifecycle-Hooks für Suspend, Resume, Offline, Reconnect, Shutdown und Restore; dauerhafte Checkpoints bleiben notwendig | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q007 | Laufzeitfähigkeiten | Signierte Pflicht-/Optionalkompatibilitäten im Manifest; harte Prüfung vor Start, keine Rechteableitung aus Capabilities | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q008 | Ereignismodell | Hybrides SDK-Ereignissystem | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q009 | Entwicklungsworkflow | Hybrider Entwicklungs- und Testworkflow | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q010 | Mocks und Testbarkeit | Vertragsbasierte Mocks mit programmierbaren Szenarien; keine Vollsimulation einer cloudOS-Instanz (Q105) | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q011 | SDK-Verteilung | SDK vollständig durch den vertrauenswürdigen App Host, modular und kompatibilitätsgeprüft bereitstellen | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q012 | Rust-Bindings | Rust/WASM-SDK-Bindings über wasm-bindgen und die Host-Bridge, keine privilegierte eigene Bridge | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q013 | SDK-Versionen | Ausschließlich neueste SDK-Version; kompatible Release-Übergänge gesondert prüfen | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q014 | SDK-Aktualisierung | Atomarer SDK-Wechsel mit Zustandswiederherstellung | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q015 | SDK-Diagnostik | Hybride SDK-Diagnose ohne Rechte-/Datenschutzumgehung | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q016 | Frontend Referenz | Vue 3 und Vite für cloudOS-eigene Oberflächen; öffentliches SDK bleibt frameworkneutral | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q017 | Rust/WASM Toolchains | Browserfähige Rust-/WASM-Pakete mit mehreren nachweislich kompatiblen Toolchains | `REKONSTRUIERT` | Früherer Gesprächskontext / Zusammenfassung; exakte Optionswahl nicht belegt |
| Q018 | GPU und Threads | WebGPU und WebAssembly-Multithreading als verbindliche Mindestanforderung für das anspruchsvolle Rust-Profil, ohne WebGL-/Single-Thread-Fallback | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q019 | Cross-Origin Isolation | Striktes COEP require-corp und passende COOP/CORP/CORS/CSP-Isolation; bei fehlenden Voraussetzungen kein Rust-Profilstart | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q020 | SDK-Origin | SDK durch Host unter reserviertem, systemverwaltetem Bereich jeder App-Origin bereitstellen; App darf Module nicht überschreiben | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q021 | Offline-App-Start | Kein neuer Offline-App-Start; autorisierte laufende Offline-Funktionen können bei Unterbrechung fortgesetzt werden | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q022 | Schema-Contracts | Contract-first mit signierten, versionierten JSON-Schemas und erzeugten Rust-/TypeScript-Typen; serverseitige Validierung | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q023 | App Actions und IPC | App-Aktionen und laufzeitinternes IPC getrennt: keine automatischen Empfängerstarts durch IPC; IPC nur autorisiert an laufende Instanzen | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q024 | IPC-Zustellung | Hybride Zustellgarantien je Nachrichtenklasse, keine undifferenzierte Exactly-once-Zusage | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q025 | IPC-Routing | Kontextabhängiges IPC-Routing mit sicheren, instanzgebundenen Handles | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q026 | Settings | Typisierte, versionierte Einstellungen mit separaten Benutzer-/Gerätebereichen und passendem Synchronisationsscope | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q027 | Settings Migration | Signierte Einstellungsschemas mit kontrollierten Migrationsverträgen | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q028 | UI-Kit | Frameworkunabhängiges UI-Kit mit optionalen Adaptern; keine Bindung öffentlicher SDK-Apps an Vue | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q029 | Internationalisierung | Vollständiges SDK-i18n-Modul mit signierten Sprachressourcen und sicherem dynamischem Sprachwechsel | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q030 | Web Worker | Schlanker Worker-SDK-Adapter über App-Hauptkontext, keine privilegierte direkte Worker-Bridge | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q031 | Netzwerkclient | Modularer SDK-Netzwerkclient; externe SSE-/WebSocket-Kanäle gesondert über den kontrollierten Network Broker freigeben | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q032 | Netzwerk-Profile | Signierte restriktive Netzwerkziel-Profile mit Protokoll-/Methoden-/Ressourcenbegrenzungen | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |
| Q033 | Credentials | Hybride Benutzer-/Service-Credentials durch geschützte opake Handles statt freier Weitergabe langlebiger Secrets | `BESTAETIGT_IM_VERLAUF` | Frühere ausdrücklich bezeichnete Nutzerantwort; in der verfügbaren Zusammenfassung nachvollziehbar |

## Vollständigkeitsgrenzen

- **Nicht verfügbar:** Ein zusammenhängender, verifizierter Originalmitschnitt sämtlicher Frageformulierungen, Antwortoptionen und gewählter Optionsbuchstaben für Q001–Q033. Ein fachlich sicherer Kurzinhalt ist **nicht identisch** mit dem Nachweis des exakten Wahlbuchstabens.
- **Nicht neu entschieden:** Fehlende Buchstaben, konkrete Endpunkte, technische Grenzwerte, zusätzliche Adapter und aktuelle Toolchainversionen.
- **Nachweisschritte bei Quellenverfügbarkeit:** Originalverlauf pro Q-ID abgleichen, exakten Wortlaut/Entscheidungszeitpunkt beifügen, widersprechende Aussagen gegen jüngere ausdrücklich bestätigte Beschlüsse prüfen, dann erst Quellennachweisstatus erhöhen.
- **Konsequenz:** R-007 hat keine inhaltlich leere frühe SDK-Zeile mehr. Die fehlenden **Originalbelege** bleiben eine explizite externe Evidenzlücke, keine heimlich erledigte Freigabebedingung.

## Aktuelle Gesamtzählung Q001–Q156

| Status | Anzahl | Bedeutung |
|---|---:|---|
| `BESTAETIGT_IM_VERLAUF` | 50 | Kontextbelege zu expliziten Entscheidungen verfügbar |
| `REKONSTRUIERT` | 106 | Fachlicher Kurzinhalt verfügbar; Originalwortlaut nicht vollständig |
| `OFFEN_QUELLNACHWEIS` | 0 | Keine rekonstruierbare Antwort – aktuell keine in der Tabelle |
| **Summe** | **156** | Neun zusätzliche K1–K9-Beschlüsse bleiben getrennt |
