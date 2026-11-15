# Architekturentscheidungen K1–K9 – konsolidierter Beschlussnachweis

> **Projekt:** cloudOS · **Beschlussdatum:** 15.11.2026 · **Status:** `KONZEPTIONELL_BESCHLOSSEN`  
> **Geltungsbereich:** App Framework SDK und die jeweils fachlich zuständigen cloudOS-Core-Verträge.  
> **Wichtig:** Diese Datei dokumentiert die ausdrücklich gewählten Optionen. Sie ist weder technischer Abnahmenachweis noch stillschweigender Ersatz für `APP_SIGNING.md`, `API_CONTRACTS.md` oder andere führende Fachkonzepte.

## Gültigkeit, Rang und Nachweis

Die Entscheidungen K1–K9 wurden nach Abschluss der 156 SDK-Fragen explizit bestätigt. Jede Entscheidung verweist auf das im Risk Register vorhandene Konfliktkennzeichen. Wo ein bestehender Fachvertrag bereits einen Teil der Regel enthält, wird er **präzisiert**, nicht verdoppelt. Änderungsbedarf in Bestandsdokumenten ist in [12-legacy-alignment.md](12-legacy-alignment.md) dokumentiert und erfordert eine spätere überprüfte Volltextintegration.

| Beschluss | Risiko | Option | Entscheidung | Fachlich führende Altverträge | Status |
|---|---|---|---|---|---|
| K1/9 | R-002 | C | Selbstsignierte Apps nur mit verpflichtendem Conformance- und Security-Gate | `APP_SIGNING.md`, `APP_API_ACCESS.md` | Konzeptionell beschlossen |
| K2/9 | R-003 | C | Browser standardmäßig; isoliertes WASI später; Native optional und separat freizugeben | `SYSTEM_ARCHITECTURE.md`, `APPLICATION_SYSTEM.md` | Konzeptionell beschlossen |
| K3/9 | R-005 | C | Pro App getrennte Origin, kontrollierter Host, Core-verwalteter Service Worker | `APP_API_ACCESS.md`, `APPLICATION_SYSTEM.md` | Konzeptionell beschlossen |
| K4/9 | R-001 | C | Eine aktive API-Major, mehrere kompatible App-Releases zeitweise parallel | `API_CONTRACTS.md`, `APPLICATION_SYSTEM.md` | Konzeptionell beschlossen |
| K5/9 | R-004 | C | Core SSE, externe SSE/WebSockets und Web Push getrennt | `PUSH_REALTIME.md`, `API_CONTRACTS.md` | Konzeptionell beschlossen |
| K6/9 | R-006 | C | Serververschlüsselung regulär, echte E2EE optional ohne Server-Klartextfunktionen | `API_CONTRACTS.md`, `VIRTUAL_FILESYSTEM.md` | Konzeptionell beschlossen |
| K7/9 | R-010 | C | Getrennte Event-Retention, erneute Replay-Autorisierung, Resync bei Lücke | `PUSH_REALTIME.md`, `API_CONTRACTS.md` | Konzeptionell beschlossen |
| K8/9 | R-011 | C | Widerruf stoppt reguläre Saga-Schritte; separate enge Recovery-Autorisierung | `API_CONTRACTS.md` | Konzeptionell beschlossen |
| K9/9 | R-014 | C | Release-spezifischer Kill Switch/Quarantäne; gesperrte Digests niemals Rollback-Ziel | `APP_SIGNING.md`, `APPLICATION_SYSTEM.md` | Konzeptionell beschlossen |

## K1/9 – R-002: Signing, Self-Signing und Conformance

**Beschluss:** Ein korrekt signiertes Paket mit lokal selbstsigniertem, App-ID-gebundenem X.509-Zertifikat kann nach gesonderter ausdrücklicher Administrationsfreigabe zugelassen werden. Diese Freigabe akzeptiert ausschließlich die fehlende unabhängige Publisher-Verifikation – **keine** fehlgeschlagenen Sicherheitsprüfungen.

**Normative Grenzen:**

1. Paketformat, vollständiger Dateiindex, Signatur, Digest, Manifest, App-ID-Bindung, API-/Device-Deklarationen und obligatorische Sicherheits-/Konformitätsprüfungen sind nicht übersteuerbar.
2. Die Self-signed-Risikoannahme ist von der Genehmigung der Endpoint- und Geräteberechtigungen getrennt; sie wird an App-ID, Version, Digest, Zertifikatsfingerprint und Policy-Revision gebunden.
3. Jede neu signierte Version beziehungsweise jeder andere Digest durchläuft den Prüfpfad neu; keine stillschweigende Selbstsignatur-Aktualisierung.
4. Externe oder selbstsignierte Apps werden niemals durch Manifestangaben zu Core-Apps; die geschützte Core-Release-Baseline gilt uneingeschränkt.
5. Konformitätstests sind gegen isolierte reale cloudOS-Testinstanzen auszuführen; ein rein lokales SDK-Simulatorergebnis ist kein Ersatz für verbindliche Gates.
6. Nicht sicherheitskritische ergänzende Review-Schritte dürfen ausschließlich durch vorab dokumentierte, auditierte Richtlinien differenziert werden. **Harte** Prüffehler bleiben absolut blockierend.

**Nachweis vor Implementierung:** signierte Negativtestpakete (ungültige Hashes, fehlerhafte Trust Chain, manipulierte `api-access.json`, unerlaubte Core-ID), getrennte Adminfreigaben, Wiederholung bei Update, Release-Gate-Status und Audit-Ereignisse.

## K2/9 – R-003: Runtime-Profile

**Beschluss:** `web` (JS/TS) und `web-wasm` (Browser-WASM, z. B. aus Rust) sind reguläre sichtbare App-Profile. `wasi-component` ist ein nachgelagertes, serverseitig stark isoliertes Worker-Profil. `native-isolated` ist nur eine mögliche spätere, standardmäßig deaktivierte und separat sicherheitsabzunehmende Erweiterung. Programmiersprache und Runtime-Profil sind unabhängig.

**Normative Grenzen:**

1. Jede sichtbare App läuft innerhalb des kontrollierten Browser-Window-Hosts; eine native Binary läuft niemals direkt im Browser oder im privilegierten Go-Core.
2. Serverseitige WASI-/Native-Worker sind Infrastrukturprozesse außerhalb des privilegierten Core-Vertrauensraums; keine allgemeine Dateisystem-, Netzwerk-, Docker-Socket-, Datenbank- oder Credential-Berechtigung.
3. Ausführbare Payloads sind signiert, vollständig indiziert und einem Release-Digest zugeordnet; Laufzeitprofile und Host-Bindings sind explizit deklariert.
4. Der Core setzt API-/Ressourcen-/Benutzerrechte in jedem Runtime-Profil autoritativ durch. Fachliche SDK-Operationen bleiben semantisch gleich, auch wenn ihre Adapter verschieden sind.
5. Browser-only-Funktionen wie Clipboard, Fenster und Browsergeräte sind nicht implizit in WASI/Native verfügbar.
6. Q113 wird als Verbot **beliebiger nativer Codeausführung**, nicht als pauschales Verbot eines später geprüften `native-isolated`-Profils präzisiert.

**Nachweis:** Runtime-Fähigkeitsmatrix, signierte Module, überprüfte WASI-Hostimports, Runner-Isolation, Ressourcenbudgets, Prozess-Stop/Revocation, native Profilfreigabe erst nach eigenem Threat Model.

## K3/9 – R-005: Browser-Isolation

**Beschluss:** Unvertrauenswürdige Apps werden jeweils unter eigener separater Origin in vom vertrauenswürdigen Window Host kontrollierten sandboxed iFrames ausgeführt. SDK-Bridge, Browser-Storage, Service Worker und Multi-Tab-Lifecycle erhalten einen verbindlichen Origin-/Instanz-/Policy-Vertrag.

**Normative Grenzen:**

1. App-Origin, Desktop-/Host-Origin und privilegierte Core-/BFF-Grenze dürfen keine frei wiederverwendbaren Authentifizierungscookies teilen. Unterschiedliche Apps teilen keine sicherheitsrelevante Origin.
2. iFrame-Sandbox-Flags, CSP, Navigation und externe Link-/Popup-Aktionen sind Host-kontrolliert. Origin-Trennung wird **nicht** durch URL-Pfade allein ersetzt.
3. PostMessage/MessagePort-Kommunikation wird gegen tatsächlichen Sender, Kanal, App-Instanz, Nachrichtenschema, Resource Scope und Replay-Regeln geprüft. Bei opaken Origins braucht es eine sichere Host-Kanalbindung statt allein eines Origin-Stringvergleichs.
4. Nur cloudOS kontrolliert Registrierung, Scope, Codeintegrität, Update und Widerruf von Service Workern; App-Code darf keinen privilegierten SW registrieren.
5. Cross-Tab-Koordination darf fremde App-Instanzen nicht auslesen oder ansteuern; gemeinsame Revisionen/Leases benötigen Fencing und aktuelle Rechte.
6. Berechtigungs-/Release-Widerruf verhindert neue geschützte Operationen sofort serverseitig; bereits offline geladener Browsercode kann nicht garantiert fernbeendet werden.
7. Die exakte Hostname-/Origin-Topologie wird vor Implementierung an unterstützten Browsern und Deploymentprofilen nachgewiesen. Sandbox und CSP allein garantieren nicht die Sicherheit eines kompromittierten Browsers.

**Nachweis:** getrennte Origins auch für identische App-Versionen, keine Credential-Übernahme, SW-Scope/Cache-Negativtests, Multi-Tab-Revoke, manipulierte Bridge-Nachrichten, Browser-Kompatibilitätsmatrix.

## K4/9 – R-001: Eine API-Major und kompatible App-Parallelität

**Beschluss:** Pro cloudOS Core existiert genau **eine aktive API-Major-Version**. Innerhalb dieser Major dürfen mehrere kompatible App-Releases derselben App zeitweise parallel laufen. Eine App-Release-Version begründet keinen Anspruch auf eine alte REST-/gRPC-/Event-Major-API.

**Normative Grenzen:**

1. Die signierte App-Kompatibilitätsdeklaration und der aktuelle Core-API-Katalog werden vor Installation, Launch, Update, Rollback und Core-Upgrade geprüft.
2. Bei Core-Major-Wechsel werden inkompatible Drittanbieter-Releases deaktiviert; laufende Instanzen werden vor dem Schnittpunkt kontrolliert beendet. Nicht kompatible Core-Baseline-Apps blockieren das Upgrade.
3. Mehrversionsbetrieb erfordert jeweils gültige Endpoint-/Resource-/Instanzrechte, Daten-/Migrationskompatibilität und eine begrenzte Übergangsphase.
4. Persistente Jobs/Sagas dürfen abgeschaltete API-Verträge nicht verwenden; Abschluss, explizite Migration oder sicherer Fehler-/Recovery-Zustand sind erforderlich.
5. Rollback nur auf geprüfte, kompatible, weiterhin autorisierte Digests; keine Rückgewinnung entfernter API- oder Benutzerrechte.
6. SDK-SemVer, App-Release, API-Major und persistierte Daten-/Prozessversion sind unterschiedliche Versionsachsen.

**Nachweis:** Core-Major-Upgrade-Gate, inkompatible Alt-App, geschützte Core-Baseline, parallele kompatible Releases, Migrations- und Rollback-Test.

## K5/9 – R-004: Realtime-Transportgrenzen

**Beschluss:** Allgemeine cloudOS-Desktop-/App-Statusänderungen laufen ausschließlich über den autorisierten **Core-SSE-Stream**. Externe SSE-/WebSocket-Verbindungen sind eigenständige kontrollierte Integrationen. Web Push ist ein optionales, browserfreigegebenes Benachrichtigungsfeature.

**Normative Grenzen:**

1. Der vertrauenswürdige App Host hält den privilegierten Core-SSE-Stream und projiziert ausschließlich aktuell autorisierte Events an Apps; keine allgemeinen eigenen Core-SSE-Verbindungen aus fremden App-Frames.
2. Kein zweiter regulärer WebSocket-Push für den Core und kein periodisches Status-Polling als Ersatz für SSE; ein späteres bidirektionales Core-Protokoll erfordert einen eigenständigen Vertragsentscheid.
3. Externe SSE-/WebSocket-Ziele werden über den Network-/Integration-Broker nach Ziel, Methode/Protokoll, Consent, Secret-Grants, Laufzeit und Ressourcenlimits kontrolliert.
4. Browser-/Web Push gehört zum Notification Service; Service-Worker-Lifecycle und Benachrichtigungserlaubnis bleiben zentral kontrolliert. Zustellung ist nicht garantiert und ersetzt keinen Eventstream.
5. Alle Verbindungsarten benötigen Reconnect-Budget, Abbruch/Widerruf, App-/Benutzerbindung, Logging ohne Secrets und definierte Degradation.

**Nachweis:** Core-SSE/Event-Bridge, Provider-Egress ohne SSRF, Consent/Revoke, getrennte Push-Berechtigungen, Offline-/Reconnection-Semantik.

## K6/9 – R-006: E2EE und Klartextverarbeitung

**Beschluss:** Serverseitige authentifizierte Verschlüsselung mit KMS ist das reguläre Datenprofil. Echte clientseitig schlüsselkontrollierte Ende-zu-Ende-Verschlüsselung bleibt optional und wird als eigenes Ressourcen-/Fähigkeitsprofil ausgewiesen.

**Normative Grenzen:**

1. Beim serververwalteten Profil dürfen autorisierte Core-Funktionen Inhalte entschlüsseln, indexieren, durchsuchen und Vorschauen erzeugen; es darf nicht als E2EE dargestellt werden.
2. Bei echter E2EE besitzt der Core keine allgemeinen Inhaltsentschlüsselungsschlüssel. Serverseitige Klartextsuche, Klartext-Vorschau und Inhaltsanalyse stehen dafür nicht zur Verfügung.
3. Optionale clientseitige Suche/Vorschau darf nach lokaler Entschlüsselung erfolgen; Indizes/Caches müssen selbst geschützt sein.
4. App-Zugriff auf E2EE-Klartext benötigt separate, sichtbare Ressourcenzustimmung; unkontrollierte Freigabe von Schlüsseln an Apps oder Server-Worker ist verboten.
5. Verschlüsselte Übertragung, Metadatenprojektion und unverfälschte Speicherung bleiben möglich, soweit die notwendigen Metadaten verfügbar und legitimiert sind; Metadaten sind nicht automatisch E2EE-geschützt.
6. Schutzprofilwechsel erfordern bewusste Autorisierung, technische Datenumwandlung und Audit; kein stiller Downgrade auf serverseitig entschlüsselbares Profil.
7. Schlüsselverteilung, Mehrgerätefähigkeit, Teilen, Rotation, Verlust und Recovery brauchen vor Realisierung einen gesonderten ausführbaren Sicherheitsvertrag; kein versteckter Admin-Generalschlüssel.

**Nachweis:** Schlüsselbesitz-/Server-Unzugänglichkeit, kein Klartext in Serverindizes/Thumbnails/Diagnostik, clientseitige Preview, getrennter Capability-Katalog, E2EE-Recovery-Design.

## K7/9 – R-010: Event-Retention, Replay und Widerruf

**Beschluss:** Primärzustand, Outbox, Inbox/Dedupe, internes Messaging, Browser-SSE-Replay, Notification Inbox und Security-Audit sind getrennte Datenklassen mit eigener Zuständigkeit und Retention. Replay ist eine erneute autorisierte Zustellung, kein Wiederaufleben historischer Rechte.

**Normative Grenzen:**

1. Event-Cursor sind und bleiben opak und an Audience, Sitzung/Policy-Epoch und Retention gebunden. Vor **jeder** Replay-Zustellung werden aktuelle Ressource-, App-, Benutzer- und Subscription-Rechte erneut geprüft.
2. Bei Widerruf dürfen keine vertraulichen Alt-Payloads zugestellt werden; optional nur minimales zulässiges Tombstone-/Revoke-Signal.
3. Bei abgelaufenen oder ungültigen Cursorn muss eine explizite `resync_required`-Semantik mit punktuellem autorisiertem Snapshot greifen; kein periodisches Client-Polling.
4. Eine Fachlöschung darf keine unbegrenzten Schattenkopien in Replay, DLQ oder Projektionscaches hinterlassen. Purge/Retention wird pro Datenklasse und rechtlicher Auditpflicht definiert.
5. Interne At-least-once-Delivery verlangt idempotente Consumer und dokumentierte Dedupe-Fenster. DLQ-Redrive ist eine neue autorisierte Recovery-Aktion; bereits erfolgte externe Seiteneffekte dürfen nicht blind wiederholt werden.
6. Security-Audit ist getrennt geschützt und nicht über App-Replay zugänglich. Audit-Retention ist kein Freibrief zur ungegrenzten Inhaltsspeicherung.
7. E2EE-Events dürfen keinen vom Core nicht legitim verfügbaren Klartext enthalten.

**Nachweis:** alte Cursor nach Entzug, Event-Schema-/Audience-Wechsel, Retention-Gap, sichere Snapshot-Race-Freiheit, DLQ/Dedupe, Lösch- und Audit-Policy.

## K8/9 – R-011: Sagas nach Rechteentzug

**Beschluss:** Laufende Sagas dürfen nach Berechtigungswiderruf keine weiteren Fachschritte mit alten Rechten ausführen. Notwendige Kompensation erhält eine **separat autorisierte, eng zweckgebundene Recovery-Identität**; das ist keine Fortsetzung der ursprünglichen Benutzerdelegation.

**Normative Grenzen:**

1. Jeder geschützte Saga-Schritt wird gegen aktuelle Rechte, Version und Resource Scope geprüft; widerrufene Delegationen sind keine Fortsetzungsberechtigung.
2. Bereits persistierte Fach-Commits sind real. Eine Cancellation ist kein Rollback; Checkpoints und unbekannte Commit-Ergebnisse müssen vor Recovery geklärt werden.
3. Der Saga-/Fach-Owner darf nur vorab klassifizierte, minimale, idempotente Kompensationen mit eigener Recovery-Autorisierung auslösen; keine generelle Admin-Impersonation.
4. Für risikoreiche oder irreversible Recovery ist eine gesonderte administrative Genehmigung nötig, bei entsprechendem Risikoprofil auch Vier-Augen-Freigabe gebunden an Saga, Ressource und Revision.
5. Fehlende Rechte, unklare externe Seiteneffekte, verlorene E2EE-Schlüssel oder erschöpfte Retry-Budgets führen zu sichtbar geschützten Warte-/Fehlerzuständen und manueller Eskalation.
6. Recovery-Aktionen und Genehmigungen werden unabhängig und manipulationssicher auditiert; sie dürfen ehemalige Benutzerrechte nicht wieder aktivieren.

**Konzeptionelle Zustandsklassen:** `AUTHORIZATION_REVOKED`, `COMPENSATION_PENDING`, `COMPENSATING`, `RECOVERY_APPROVAL_REQUIRED`, `RECOVERY_FAILED`, `COMPENSATED`, `MANUAL_REVIEW_REQUIRED`. Endgültige API-/Enum-Namen bedürfen des Vertrag-Freeze.

**Nachweis:** Session-/Grant-Revoke zwischen zwei Saga-Schritten, getrennte Recovery-Capability, Vier-Augen-Fall, externer Commit unbekannt, kompensationssicherer Retry, E2EE-Fall.

## K9/9 – R-014: Release-Sperre, Quarantäne und Rollback

**Beschluss:** Ein zentraler, autoritativer Release-Kill-Switch sperrt gezielt betroffene App-ID-/Version-/Digest-/Zertifikats- oder Runtime-Bereiche. Sicherheitssperren haben Vorrang vor Mehrversions-Übergangsfristen, gewöhnlicher Adminfreigabe und Rollback.

**Normative Grenzen:**

1. Jedes ausführbare Release ist unveränderlich an App-ID, Version, Paket-Digest, Signatur, Trust-Klasse, Runtime-Profil und Berechtigungsdeklarationen gebunden. Gleiche Version mit anderem Digest ist ein Konflikt.
2. Sperrungen können Digest, Version, App-ID, Publisher/Zertifikat, Runtime oder einzelne gefährdete SDK-Operationen betreffen; möglichst eng, soweit sicher, ansonsten bewusst breiter.
3. Neue Starts und **neue geschützte Core-Operationen** gesperrter Releases werden unverzüglich serverseitig verweigert; betroffene Browser-Instanzen und WASI-/Native-Worker werden kontrolliert gestoppt. Offline-Browsercode lässt sich nicht sofort zuverlässig fernbeenden.
4. Quarantäne konserviert signierte Artefakte nur für definierte Forensik-/Retentionzwecke ohne Ausführungsrechte. Betroffene Service-Worker-/Asset-Caches dürfen keine privilegierte Reaktivierung ermöglichen.
5. Nicht betroffene Releases bleiben nur dann aktiv, wenn gemeinsame Abhängigkeiten, Zertifikate und Runtimes tatsächlich nicht kompromittiert sind.
6. Rollback ist ausschließlich auf nicht gesperrte, vollständig signatur-/conformance-geprüfte, aktuelle API-Major-kompatible und migrationskompatible Digests erlaubt. Entfernte Rechte dürfen nicht wiederhergestellt werden.
7. Keine gewöhnliche Administratorwarnung überschreibt Signaturfehler oder harte Sicherheitssperren. Aufhebung einer Sperre verlangt eigenständige begründete, auditiert freigegebene Sicherheitsneubewertung.
8. Benutzer-/App-Daten werden durch die Sperrung nicht automatisch gelöscht. In-flight-Commits und Sagas werden anhand der separaten Lifecycle-/Recovery-Verträge behandelt.

**Nachweis:** Kill-Switch bei laufendem Parallelrelease, blockierte Bridge/Worker, Quarantäne, gefährdetes Zertifikat, Cache-Reaktivierung, Notfall-Rollback-Negativtest, Audit-Log.

## Verbindliche Folgearbeiten / keine fingierte Abnahme

1. Alle Beschlüsse mit führenden Bestandsdokumenten in [12-legacy-alignment.md](12-legacy-alignment.md) abgleichen; deren Inhalt darf nicht durch diese neue Datei stillschweigend ersetzt werden.
2. Betroffene SDK-Teilkonzepte verknüpfen, Decision Register und Risk Register aktualisieren.
3. Reale technische Nachweise, Browser-/Runtime-/Crypto-/Recovery-Testpläne und maschinenlesbare Schnittstellen erst anschließend spezifizieren.
4. Bestehende offene Risiken R-007, R-008, R-009, R-013 und R-015 bleiben offen. Alle anderen Risiken werden durch den Beschluss höchstens **konzeptionell**, nicht technisch erledigt.
5. Keine Implementierungsfreigabe aus der bloßen Existenz dieser Beschlüsse ableiten.
