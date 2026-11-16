# Phase 2: Bestandsvertrags-Abgleich zu K1–K9

> Projekt: `Drezael/cloudOS` · GitHub-Branch: `main` · geprüft am 15.11.2026  
> Status: **Historischer Phase-2-Review vom 15.11.2026; dokumentarische Volltext-Integration erfolgte im Folgeschritt 16.11.2026 (siehe Nachtrag unten).**  
> Keine Implementierung, kein technischer Freigabebeschluss.

## Ziel und Verbindlichkeit

Die neun Architekturentscheidungen sind in `docs/app-framework/11-architecture-decisions-K1-K9.md` festgelegt. Die ursprünglichen umfangreichen cloudOS-Fachkonzepte dürfen nicht durch stark verkürzte Neufassungen überschrieben werden. Phase 2 muss deren vollständigen aktuellen Inhalt inklusive Abbildungen, Beispiele, Querverweise und bestehender Regelungen erhalten. Dieses Dokument enthält präzise zu integrierende Vertragsklarstellungen, **nicht** den vollständigen Ersatztext der älteren Dateien.

## Quelleninventar, direkt aus GitHub `main` geprüft

| Bestand | GitHub-Blob-SHA | Umfang ca. | Entscheidung |
|---|---|---:|---|
| `docs/APP_SIGNING.md` | `173a4c4bb278927c9f0df62269e2f0b4bd18b9ac` | 551 Zeilen | K1, K9 |
| `docs/APP_API_ACCESS.md` | `f498c244e9f0ba8130e6c6522294331492baa898` | 769 Zeilen | K1, K3, K5, K9 |
| `docs/APPLICATION_SYSTEM.md` | `85ddfda6aae37196ed5b251bde8fcf86b382a7d2` | 1173 Zeilen | K1–K4, K9 |
| `docs/SYSTEM_ARCHITECTURE.md` | `e0397109b11a59667aff03694538b6577fd3f0cb` | 2648 Zeilen | K2, K3 |
| `docs/API_CONTRACTS.md` | `ec0b10f1f68b083f3078d8a4705a4afbe7d363bf` | 957 Zeilen | K4–K9 |
| `docs/PUSH_REALTIME.md` | `1eb04bca0057299befbb9b9a0d0f8f30e9f8677b` | 581 Zeilen | K5, K7 |
| `docs/VIRTUAL_FILESYSTEM.md` | `1fbb089bb213802d589103d4e0d7f27f3da10220` | 1067 Zeilen | K6, K7 |
| `docs/CORE_DEPLOYMENT_SCALING.md` | `8e23a6cdf9c266741a3e32bb2557fdb64e966992` | 665 Zeilen | K2, K3, K5 |
| `docs/APP_MANAGEMENT_STORE.md` | `2544f6f08fc2209f2ef6da19d177ace74b5f1d62` | 833 Zeilen | K1, K4, K9 |

## Fachliche Befunde

### B-01: Scheinbarer Versionswiderspruch mit echter Formulierungslücke (HIGH)

`docs/APP_MANAGEMENT_STORE.md`, Abschnitt 4.1, formuliert: **„Für das MVP gibt es eine aktive Version je App-ID“**. K4/9 erlaubt während eines kontrollierten Übergangszeitraums laufende Instanzen unterschiedlicher kompatibler Releases.

**Konsistente Auflösung ohne neue Grundsatzentscheidung:**

- Die App Registry hat pro App-ID genau **ein aktives Standard-/Zielrelease** für neue Starts.
- Bereits gestartete, konkret digestgebundene Instanzen eines vorherigen Releases dürfen im genehmigten Zeitfenster weiterlaufen, **sofern** sie mit der einen aktiven Core-API-Major und dem aktuellen Berechtigungs-/Sicherheitsstand kompatibel sind.
- Die App-Registrierung wird dadurch nicht mehrfach installiert; Parallelität betrifft **Laufzeitinstanzen**, nicht gleichzeitig konkurrierende Default-Releases.
- Neue Instanzen starten ausschließlich die jeweils freigegebene aktuelle Standardversion.
- Übergangsfrist, maximale Laufzeit und Abschlussbedingungen werden als Limits des Release-Lifecycle dokumentiert, ohne derzeit Zahlen vorzugeben.
- Sicherheitswiderruf K9 beendet den Übergang des betroffenen Digests sofort serverseitig; eine offline geladene App kann nur hinsichtlich neuer Core-Zugriffe gesperrt werden.

**Redaktionelle Änderung erforderlich in** `docs/APP_MANAGEMENT_STORE.md` Abschnitt 4.1 und abgestimmt in `docs/APPLICATION_SYSTEM.md` Abschnitt 17.3 sowie `docs/API_CONTRACTS.md` Abschnitt 3.1.

### B-02: Verbindlicher Origin-Schutz statt bloßer Präferenz (HIGH)

`docs/APP_API_ACCESS.md` Abschnitt 7.1 sagt „vorzugsweise unter eigener Origin“. K3/9 verlangt getrennte Origins **verbindlich**. Das Wort „vorzugsweise“ muss entsprechend dem neuen Beschluss ersetzt und mit einem nachweisbaren Origin-/Service-Worker-/Sandbox-Vertrag verbunden werden. Noch offen: konkrete Hostnamen/DNS-Topologie, erlaubte Sandbox-Flags, Browser-Testmatrix. Keine riskante blanket-Behauptung, CSP könne alle Browser-Egresspfade verhindern.

### B-03: Signing/Conformance-Gate explizit getrennt (BLOCKER bis technischer Nachweis)

`docs/APP_SIGNING.md` Abschnitte 5.3 und 6 trennen Signaturgültigkeit, Publisher-Vertrauen und Installationsentscheidung bereits korrekt. Ergänzung: technische Conformance gehört als zwingendes **eigenständiges** Install-/Update-/Aktivierungs-Gate davor; Self-signed-Risikofreigabe betrifft ausschließlich Publisher-Vertrauen, nie Integrität oder Conformance. Neue Versionen/Digests erfordern erneute Prüfungen. Ausnahmen für bekannte ungetestete Publisher verleihen keine Core-Rechte.

### B-04: Runtime-Profil-Matrix erforderlich

`docs/SYSTEM_ARCHITECTURE.md` sieht Browser-WASM, optionale WASI und isolierte Native-Jobs bereits vor. K2 präzisiert: Browser JS/TS/WASM als Standard; Server-WASI erst nach MVP, eigene Isolation und Signaturprofile; Native nur künftiges gesondertes Runner-Konzept, nie im privilegierten Go-Core-Prozess. Deployment-Modi `combined`/`split`/Kubernetes dürfen nicht fälschlich Runner-Isolation ersetzen.

### B-05: Realtime-Trennung ist weitgehend vorhanden

`docs/PUSH_REALTIME.md` definiert SSE als regulären Core-Pushkanal, Replay mit aktueller Policy und Web Push als optionale Benachrichtigung. K5 benötigt explizite Abgrenzung externer Provider-SSE-/WebSocket-Verbindungen über Network-/Integration-Broker mit eigenem Egress/Consent/Lifecycle. Kein zweiter generischer Core-WebSocket- oder Polling-Fallback.

### B-06: E2EE nicht serverentschlüsselbar

`docs/API_CONTRACTS.md` Abschnitt 17 stellt selbst fest, dass serverseitige Verschlüsselung **keine** E2EE ist. K6 verlangt zusätzlich ein separat erkennbares E2EE-Datenprofil im VFS mit klar abgeschalteten serverseitigen Klartextfunktionen (Volltextsuche, Preview, Worker-Analyse). Clientseitige Verarbeitung kann optional möglich sein; Schlüsselverteilung/Recovery bleiben offen. Die dokumentierten serverseitigen Deduplizierungsregeln dürfen nicht automatisch für E2EE gelten.

### B-07: Event-Replay und Aufbewahrungsklassen

`docs/PUSH_REALTIME.md` Abschnitt 8 enthält bereits richtige aktuelle AuthZ-Prüfung und Resync bei Replay-Lücken. K7 verlangt die getrennte Retention-Semantik für Fachzustand, Outbox, Inbox/Dedupe, internes Event, Browser-Delivery, Notification Inbox und Security-Audit; keine lange Lebensdauer alter vertraulicher Replay-Payloads. Kein neues garantiertes Exactly-once-Verhalten behaupten.

### B-08: Saga-Kompensation besitzt eigene Autorisierung

`docs/API_CONTRACTS.md` Abschnitt 9 definiert Sagas. K8 verlangt: bei Widerruf normale Schritte stoppen; separate minimal gebundene Recovery-Identität für nötige idempotente Kompensation, bei Sicherheitskritikalität Human Approval, kein Übernehmen widerrufener Benutzerrechte. Keine willkürlichen neuen privilegierten REST-Routen in dieser Phase.

### B-09: Notfall-Release-Revocation präzisieren

`docs/APP_SIGNING.md` Abschnitt 9, `docs/APP_MANAGEMENT_STORE.md` Abschnitt 9.2 besitzen bereits Sperr-/Quarantäneregeln. K9 verlangt deren konsistenten Durchgriff auf aktive Fenster/Bridges, serverseitige Runner, Jobtrigger und Rollback-Prüfung: Sperre per Digest, App-ID, Zertifikat, Runtime-Profil oder Capability; keine gewöhnliche Admin-Übersteuerung harter Sicherheitsblocker. Die Signatur einer früheren Version beweist keine heutige Ausführungserlaubnis.

## Verbindliche Bearbeitungsregeln für vollständige Originaldateien

1. Alle Dateien aus derselben aktuellen GitHub-Revision beziehen und SHA prüfen; keine Mischung verschiedener Zwischenstände.
2. Alle Originalabschnitte einschließlich Diagrammen, Links und API-Beispielen vollständig erhalten; Änderungen nur an nachweislich betroffenen Klauseln.
3. Neue Normsätze in fachlich führenden Dateien einfügen, SDK auf diese Norm verlinken; keine doppelt konkurrierenden Owner einführen.
4. Für jede Datei den Grund K1–K9, die jeweilige Einfügestelle und Konfliktauflösung erfassen.
5. Ausgegebene Dateien müssen **vollständige** Markdown-Dokumente sein, keine alleinstehenden Einfügeblöcke.
6. Nach dem Edit vollständige Diff-/Link-/Markdown- und Verifizierungsprüfung, dann ZIP mit originalen Repo-Pfaden.
7. `09-risk-register.md` und `10-implementation-readiness.md` erst nach tatsächlich erfolgter Integration auf `BESTANDSDOKUMENTE_KONSOLIDIERT` setzen; nicht vorab.

## Noch offen

- R-007: 32 historische Originalbeschlüsse nicht nachgewiesen.
- R-008: Go-Core Owner und SDK-Komponenten noch abzugleichen.
- R-009: Consent-Datenflussgrenzen.
- R-013: Negative Contract-Tests.
- R-015: Isolation einer echten cloudOS-Konformitätstestumgebung.
- Technische Validierung K1–K9, insbesondere Origin-Isolation, Runtime-Profil, Cryptography und Kill-Switch.

## Technische Ausgangslage dieser Lieferung

Die GitHub-Verbindung erlaubte vollständige fachliche Quellprüfung und SHA-Ermittlung, jedoch keine direkte Dateiübernahme in den lokalen ZIP-Arbeitsbereich. Daher enthält dieses Zwischenpaket **bewusst keine verkürzten Ersatzfassungen** der neun umfangreichen Bestandsdateien. Die vollständige textbewahrende Integration erfordert den zusammenhängenden Originaldateibestand im lokalen Arbeitsbereich, etwa als Repository-ZIP. Dieses Dokument ist eine verbindliche Vorprüfung, nicht die fälschliche Behauptung einer fertigen Originaldatei-Konsolidierung.

## Nachtrag 16.11.2026 – tatsächliche redaktionelle Übernahme

Die hier ursprünglich geforderten neun Volltext-Konsolidierungen wurden gegen GitHub-Blob-geprüfte Bestandsfassungen ausgeführt. B-01 ist in `APP_MANAGEMENT_STORE.md` mit **einem Defaultrelease** und erlaubten, unverändert digestgebundenen kompatiblen Altinstanzen unter einer aktiven Core-API-Major konkretisiert. B-02 ist in `APP_API_ACCESS.md` von „vorzugsweise“ auf **verpflichtende getrennte Origins** korrigiert. B-03 bis B-09 sind als verbindliche K1–K9-Klauseln in ihren jeweiligen fachlichen Owner-Dokumenten ergänzt.

**Nicht geschlossen** sind technische/laufzeitliche Nachweise, abschließende API-/DB-Schemas, Sicherheits-/Browser-Testresultate und vollständige Originalbelege Q001–Q033. Der ursprüngliche Befund in den Abschnitten oben dokumentiert die Lage **vor** der Änderung; für den aktuellen Status sind [12-legacy-alignment.md](12-legacy-alignment.md), [09-risk-register.md](09-risk-register.md) und [10-implementation-readiness.md](10-implementation-readiness.md) maßgeblich. Kein Release-/Implementierungs-Sign-off.
