# cloudOS – API-Verträge, SDK-, Sicherheits-, Synchronisierungs- und Speicherverträge

> **Verbindlicher Architekturabgleich vom 15.11.2026:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md). Die dort ausdruecklich bestaetigten neueren Infrastrukturentscheidungen gehen widersprechenden aelteren Entwurfs- und Rollout-Formulierungen vor; dies ist kein Implementierungsnachweis.

> **Datei:** `docs/API_CONTRACTS.md`  
> **Dokumentversion:** 1.0 (verbindliches Fach- und Architekturkonzept)  
> **Stand:** 15.11.2026  
> **Projekt:** [Drezael/cloudOS](https://github.com/Drezael/cloudOS)  
> **Basis:** Die 74 abgestimmten Konzeptentscheidungen vom 15.11.2026 und die am selben Tag im Repository `main` eingesehenen Architektur- und Fachkonzepte.  
> **Status:** Konzeptionell verbindlich; kein Nachweis einer Implementierung. Schemafragmente, Routen und Abläufe sind, soweit nicht bereits in anderen Dokumenten definiert, **Vertragsentwürfe**, keine vorhandenen Endpunkte.  
> **Fachlich führend für:** Schnittstellenregeln, Protokollverträge, API-/SDK-Zugriffe, Berechtigungsdurchsetzung, Fehler-, Event- und Jobsemantik, Quality Gates sowie die im API-Verhalten sichtbaren Teile der Sitzungs-, Offline- und Datei-Synchronisierung.

## 0. Dokumentstruktur und Verbindlichkeit

Dieses Dokument konsolidiert **alle 74 getroffenen Entscheidungen**. Die Kapitel 1 bis 20 formulieren daraus normativ nutzbare Verträge. Kapitel 21 enthält den vollständigen, nummerierten Entscheidungsnachweis; Kapitel 22 grenzt andere Fachdokumente und später zu präzisierende Details ab.

Die Begriffe **MUSS**, **DARF NICHT**, **SOLL** und **KANN** bezeichnen verbindliche Anforderungen, Verbote, starke Sollvorgaben und Erweiterungsmöglichkeiten. Nicht festgelegte numerische Grenzwerte, genaue Namen neuer Routen sowie konkrete kryptografische Parameter dürfen nicht als bereits beschlossen oder implementiert ausgegeben werden. Sie werden vor der technischen Umsetzung in versionierten Verträgen bzw. ADRs präzisiert.

Fachlich zugeordnete Referenzen:

- [CONCEPT.md](CONCEPT.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md)
- [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [APP_API_ACCESS.md](APP_API_ACCESS.md), [APP_SIGNING.md](APP_SIGNING.md)
- [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md), [CLIENT_DEVICES.md](CLIENT_DEVICES.md)
- [PUSH_REALTIME.md](PUSH_REALTIME.md), [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md)
- [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [SHARING_SPACES.md](SHARING_SPACES.md)
- [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md)
- [THEMING.md](THEMING.md), [WALLPAPER_ENGINE.md](WALLPAPER_ENGINE.md)

Soweit ein älteres Konzept zum selben Thema abweicht, gilt für den hier geregelten **API-Vertrag** die jüngere, ausdrückliche Entscheidung dieses Dokuments. Insbesondere sind **SSE statt WebSocket als regulärer Browser-Push-Transport** und **nur die jeweils aktive API-Major-Version ohne parallele Alt-Major-API** verbindlich. Das führt nicht zu einer stillschweigenden Änderung anderer fachlich führender Konzepte; notwendige Folgeanpassungen werden gesondert dokumentiert.

## 1. Architekturziele und unverhandelbare Invarianten

1. **API-first, nicht App-first privilegiert:** Jede Fachfunktion gehört einem autoritativen Go-Core-Service. Apps benutzen ausschließlich freigegebene SDK-Funktionen; ein Core-App-Status ist kein automatischer Vollzugriff.
2. **Default Deny:** Identität, App-Instanz, deklarierte und genehmigte API-Operation, Ressourcenscope, aktuelle Benutzerrechte, Consent und Sicherheitsrichtlinien müssen bei jeder sensiblen Operation wirksam sein.
3. **Deployment-unabhängige Semantik:** `combined`, `split` und später `kubernetes-distributed` implementieren dieselben fachlichen Vertrage. Weder Routen, DTOs, Berechtigungen noch Idempotenz dürfen vom Betriebsmodus abhängen.
4. **Ein Service-Owner pro Ressource:** Schreibende Zustandsänderungen werden von einem fachlich verantwortlichen Owner kontrolliert. Dienste dürfen fremde Owner-Daten nicht direkt manipulieren.
5. **Keine permanenten Client-Statusabfragen:** Serverinitiiertes SSE liefert fortlaufende Änderungen; ein punktueller Snapshot bei der Initialisierung oder nach nachgewiesener Replay-Lücke bleibt erlaubt.
6. **Revisions- und Wiederholsicherheit:** API-Schreibzugriffe benötigen die passende Konfliktsemantik; unabhängig vom Transport darf ein Retry keine unkontrollierte zweite Wirkung erzeugen.
7. **Fail closed bei Sicherheit, degrade gracefully bei Funktionen:** Ein fehlender Fachservice darf unabhängige Funktionen nicht mitreißen, fehlende Berechtigungsprüfungen dürfen aber nicht übergangen werden.
8. **Audit ist unabhängig von Diagnoselogs:** Pflichtsicherheitsereignisse bleiben selbst bei abgeschalteter Debug-/Diagnostikausgabe erfasst.
9. **Dauerhafte Serverwahrheit:** Authentifizierungs- und Berechtigungszustand, langlebige Jobs, relevante Synchronisierungsrevisionen und Datei-Manifeste dürfen nicht nur in einer Core-Replik liegen.
10. **Entwurfsstatus transparent:** Dieses Dokument beschreibt Zielverhalten. Ein Beispiel für ein JSON-Objekt oder eine Route ist noch keine Aussage über vorhandene Implementierungen.

## 2. API-Akteure, Vertrauensgrenzen und Bereitstellungsmodi

### 2.1 Akteure

| Akteur | Vertrauensstellung | Zulässiger Zugriff |
|---|---|---|
| Browser-Shell / vertrauter App-Host | Privilegierte UI-Host-Komponente, **keine** fachliche Berechtigungsquelle | Authentifizierte BFF-/Gateway-Operationen; vermittelt SDK und SSE |
| Gewöhnliche `.cloudApp` | Nicht vertrauenswürdiger, signierter App-Code in isolierter Runtime | Nur deklarierte, genehmigte SDK-Operationen über Broker |
| Vorinstallierte Core-`.cloudApp` | Geschützte, signierte Baseline, weiterhin sandbox-isoliert | Nur eigene genehmigte Baseline-Operationen und aktuelle Benutzerrechte |
| Go-Core-Fachdienst | Vertrauenswürdiger Workload mit begrenztem Aufgabenbereich | Typisierte lokale Ports oder mTLS-geschütztes gRPC; Service-Capabilities |
| Asynchroner Worker / Event-Consumer | Nur für seinen Handler autorisiert | Validierte JetStream-Events, Job-Leases und fachliche Owner-APIs |
| Zukünftiger externer API-Client | Anfangs **nicht** freigeschaltet | Erst nach gesondert freigegebenem externem Auth-/API-Profil |

### 2.2 Betriebsprofile

- **Standard:** `combined` mit **einem Go-Core-Prozess in genau einem Core-Docker-Container**. PostgreSQL und NATS JetStream bleiben separate Infrastruktur. Lokale Fachdienstaufrufe laufen über typisierte Go-Ports und transportieren denselben Auth-, Deadline-, Fehler- und Trace-Kontext wie entfernte Aufrufe.
- **Optional:** `split` mit ausgewählten Core-Diensten als eigenständige Prozesse. Die Kommunikation nutzt gRPC mit authentifizierten Service-Identitäten.
- **Später:** `kubernetes-distributed` mit replizierbaren Diensten, gemeinsamem dauerhaftem Zustand, Lastverteilung, HPA/Worker-Skalierung und identischen Fachverträgen. Verteiltes Deployment ist **keine** Voraussetzung für den normalen Desktopbetrieb.
- Lokale Modulgrenzen sind keine realen Fehlerisolationsgrenzen auf Prozessebene: Ein Absturz des einzigen Go-Prozesses kann im `combined`-Modus alle darin enthaltenen Module gleichzeitig betreffen. Die Architektur muss diesen Unterschied im Monitoring korrekt ausweisen.

### 2.3 Transportschema

| Verbindung | Verbindlicher Vertrag | Sicherheitskontext |
|---|---|---|
| App -> SDK -> App-Bridge/Broker -> Gateway | Fachlich benannte SDK-Operationen; darunter HTTPS/JSON oder Host-Aufruf | Benutzer + App + Instanz + genehmigte Capability + Ressource |
| Browser-Shell -> Gateway/BFF | HTTPS, REST/JSON, Streaming für Datei-Bytes | Gebundene, geschützte Sitzung und serverseitige Autorisierung |
| Browser-Shell -> Realtime | Authentifiziertes **SSE** (`text/event-stream`) | Sicherheitsgefilterte Audience und Event-Subscription |
| Go-Dienst -> Go-Dienst im Prozess | Typisierte lokale Go-Schnittstelle | Expliziter Service-/Delegationskontext |
| Go-Dienst -> Go-Dienst verteilt | gRPC + Protocol Buffers über mTLS | Workload-Identität + kurzlebige Service-Capability |
| Core -> asynchrone Verarbeitung | NATS JetStream + CloudEvents-Vertrag | Authentifizierter Producer und berechtigter Consumer |
| Core -> PostgreSQL / Block-Backend | Owner-spezifischer Speicher- und Transaktionsvertrag | Least-Privilege-Datenbank-/Speicherzugriff |

## 3. Versionierung, Kompatibilität und Lifecycle

### 3.1 Genau eine aktive Major-API

- Die HTTP-API verwendet das im Repository etablierte Präfix **`/api/v1`** für die erste Major-Version. Das Suffix bezeichnet die aktuelle Hauptversion, nicht eine dauerhaft weiterbetriebene Alt-API.
- cloudOS betreibt **nur die aktuelle API-Major-Version**. Nach einem Major-Wechsel gibt es keinen parallelen Betrieb alter Major-REST-, gRPC-, SDK- oder Event-Verträge als allgemeine Kompatibilitätsgarantie.
- Innerhalb einer aktiven Major-Version dürfen bestehende Bedeutungen, erforderliche Felder und Sicherheitsgarantien nicht stillschweigend gebrochen werden. Optionale Felder sind möglich, wenn der Vertrag deren Kompatibilitätsverhalten definiert; unbekannte nichtkritische Felder sind von Clients zu tolerieren.
- Breaking Changes erhalten eine explizite Impact-Analyse, neue Verträge und Migrationsanweisungen. Geplante Deprecations enthalten Ersatz, Ankündigung und Abkündigungszeitpunkt. **Kritische Sicherheitsrisiken** dürfen eine sofortige Entfernung rechtfertigen.
- Bei einem notwendigen Core-Upgrade wird jede installierte App-Version gegen die Zielverträge geprüft. Inkompatible Drittanbieter-Apps bleiben installiert, erhalten eine klare Warnung und werden deaktiviert, bis ein kompatibles signiertes Update vorliegt.
- **Geschützte Baseline-Core-Apps müssen kompatibel bleiben.** Fehlt eine kompatible Baseline, muss das Core-Upgrade vor Aktivierung der unvereinbaren Version blockiert werden; ein systemkritischer Ausfall wird nicht bewusst in Kauf genommen.
- Externe Clients sind erst für eine gesonderte Ausbaustufe vorgesehen. Die Existenz eines REST-Pfads allein macht ihn nicht frei extern nutzbar.

### 3.2 Manifest- und SDK-Kompatibilität

Jedes signierte `.cloudApp`-Paket deklariert mindestens die benötigte Core-API-Major-Version, erforderliche SDK-Capabilities und relevante Mindestversionen. Installation, Update, Start und Core-Upgrade validieren diese Angaben gegen den aktuellen API-Katalog. Die signierte Datei **`permissions/api-access.json`** bleibt die maßgebliche Liste beantragter Core-Operationen (siehe [APP_API_ACCESS.md](APP_API_ACCESS.md)); ein kompatibles SDK ersetzt weder diese Deklaration noch die Administratorfreigabe.

Die offiziellen SDKs werden versioniert und vertrauenswürdig verteilt: JavaScript-/TypeScript-Pakete über einen offiziellen npm-Verteilungspfad und Rust über ein offizielles Crate. Weitere Sprachen können später über standardisierte Bindings hinzukommen. Das SDK ist beim Entwickeln vollständig dokumentiert, stellt einer laufenden App jedoch nur ihre **effektiv freigegebenen** Operationen bereit.

### 3.3 Deprecation und Dokumentation

Jede Freigabe mit API-Änderungen MUSS enthalten:

1. Maschinenlesbare, gültige Verträge samt Versionsmetadaten.
2. Automatisch ermittelten Vertragsdiff und Einordnung in kompatibel / breaking / sicherheitskritisch.
3. Generiertes Changelog und bei Migrationen eine Markdown-Anleitung mit JavaScript-, TypeScript- und Rust-Beispielen.
4. App-Kompatibilitätsbericht mit gesonderter Prüfung der geschützten Core-App-Baseline.
5. Dokumentierte, autorisierte Releasefreigabe nach bestandenen Pflichtprüfungen.

Die Entwicklerdokumentation ist **statisch in Markdown** verfügbar. Ein interaktives Entwicklerportal und eine in cloudOS integrierte Entwicklungsumgebung sind anfangs keine Pflicht. Externe Editoren/IDEs und Toolchains sind der vorgesehene Entwicklungsweg; eine `Developer.cloudApp` bleibt eine spätere Option.

## 4. Formale Verträge, Eigentümerschaft und Katalog

### 4.1 Standardformate

| Kanal | Normatives maschinenlesbares Format | Weitere Regeln |
|---|---|---|
| REST/JSON | **OpenAPI 3.1** | Exakte Methoden, Pfade, `operationId`, Parameter, Antworten, Fehler, Security, Quotas |
| gRPC | **Protocol Buffers (`.proto`)** | Versionspakete, Feldreservierung, Deadline-/Statusmapping, Service-Capabilities |
| Asynchrone Events und Jobs | **AsyncAPI** plus versioniertes Payload-/CloudEvents-Schema | Topic/Subject, Producer, Consumer, Ordering, Idempotenz, Retention |
| SSE | Versionierte Ereignisschemas, in den zentralen Eventkatalog integrierbar | Audience, Replay-Cursor, Deltas, Sicherheit und Resync-Regeln |
| SDK | Typisierte Operationen aus verifizierten Verträgen | JS/TS/Rust-Bindings; Runtime-Capability-Filter |
| Technische Doku | Generiertes und redaktionell erweitertes Markdown | Beispiele, Migrationshinweise, Sicherheitsgrenzen |

Jeder fachliche Service **besitzt** seine API- und Event-Verträge. Ein zentraler, automatisch validierter Katalog aggregiert sie und bildet die einzige vertrauenswürdige Quelle für `operation_id`, kanonische Route/Methode, Ressourcenscope, Risikoeinstufung, minimale Version und Capability-Zuordnung. Die Deployment-Topologie beeinflusst die Registry nicht.

### 4.2 Registrierungs- und Änderungsregeln

- Keine App darf durch einen frei formulierten Namen, eine URL-Wildcard oder ein selbst deklariertes SDK-Symbol neue Rechte erzeugen.
- Alle operationIds sind stabil und eindeutig. HTTP-Methode, Route und Ressourcenscope müssen mit dem Katalog übereinstimmen.
- Entfernte Protobuf-Feldnummern werden reserviert und niemals neu belegt. Event-Schemaänderungen werden auf kompatible Erweiterungen und sichere Replays geprüft.
- Verträge enthalten fachliche Eigentümerschaft, Sicherheitsstufe, Fehlercode-Katalog, Idempotenzklassifikation, Aufruf-/Rate-Limits und erwartetes Fallback-Verhalten.
- PR-/CI-Prüfungen identifizieren unregistrierte Operationen, gebrochene Referenzen, unvereinbare Schemas, fehlende Dokumentation und geänderte App-Permissions.
- Die zentrale Registry darf keine fachlich unabgestimmte Berechtigungsvergabe oder Versionsfreigabe vornehmen; diese bleibt Bestandteil des kontrollierten Releaseprozesses.

## 5. REST-/Gateway-Verträge

### 5.1 Transport, Darstellung und Idempotenz

- HTTPS ist für produktive Browser- und externe Verbindungen verpflichtend; Browser-Sitzungen werden über den geschützten BFF-/Gateway-Kontext geführt. Schreibende Cookie-basierte Requests benötigen geeigneten CSRF-Schutz.
- Erfolgreiche JSON-Antworten sind schemavalidiert und typisiert. UTC-Zeitstempel verwenden RFC 3339; fachliche IDs sind stabile opake Kennungen/UUIDs, keine Host-Dateipfade.
- Listen sind cursorpaginiert mit dokumentierter stabiler Sortierung. Grosse Inhalte werden gestreamt; niemals als riesige JSON-Listen oder unkontrollierte In-Memory-Puffer behandelt.
- Ressourcen mit konkurrierenden Änderungen liefern `ETag`/Revisionen. Schreibende Aufrufe verwenden `If-Match` beziehungsweise vertraglich gleichwertige Vergleichsbedingungen; eine veraltete Revision verursacht einen definierten Konflikt, kein stilles Überschreiben.
- `Idempotency-Key` bindet wiederholte logische Schreibvorgänge an denselben Benutzer-/App-/Operations-/Payloadkontext. Wiederholtes Absenden darf keine doppelte fachliche Wirkung erzeugen. Schlüssel und Ergebnisse sind zeitlich begrenzt und sicher gespeichert.
- Timeouts, maximale Bodygrößen, Verbindungslimits, Streaminggrenzen, Dateiquotas und Rate-Limits werden pro Operation erfasst und serverseitig umgesetzt.
- `traceparent` und eine opake `request_id` verknüpfen Verarbeitung und Logs. Geheimnisse und persönliche Inhalte gehören nicht in URL-Parameter, Fehlerdetails oder Traces.

### 5.2 Einheitliche Fehlersemantik

Fehler werden nach **RFC 9457 Problem Details** mit `Content-Type: application/problem+json` ausgegeben. Neben `type`, `title`, `status`, `detail` und optional `instance` verwendet cloudOS strukturierte Erweiterungen, insbesondere `code`, `request_id`, nach Bedarf `retryable`, `retry_after` und feldbezogene Validierungsfehler. Der `detail`-Text ist keine maschinenlesbare Kontrollschnittstelle.

| HTTP-Status | Typischer Fall | Besondere Regel |
|---|---|---|
| `400` / `422` | Syntaktisch ungültige / fachlich unzulässige Eingabe | Keine internen Details oder Geheimnisse ausgeben |
| `401` | Keine gültige Anmeldung | Nicht durch alte Offline-/SSE-Daten umgehen |
| `403` | Fehlende Berechtigung oder Capability | Fachlich nicht sichtbare Ressourcen ggf. bewusst mit `404` maskieren |
| `404` | Nicht vorhanden oder absichtlich verborgen | Ressourcenauskunft darf keine ID-Enumeration ermöglichen |
| `409` | Fachlicher Konflikt, laufende Operation, unvereinbare Zustandsänderung | Definierten Konfliktcode und sicheren Recovery-Pfad liefern |
| `412` | `If-Match`/Revision stimmt nicht | Aktuelle Revision nur nach Berechtigungsprüfung offenlegen |
| `413` | Daten-/Uploadgrenze überschritten | Kein inkonsistenter Teilcommit |
| `429` | Rate-Limit | Begrenzte Retry-Information und angemessene Header |
| `503` | Notwendiger Fachservice nicht verfügbar | Nur sichere Wiederholungsversuche; unabhängige Dienste bleiben nutzbar |

Konzeptionelles Problem-Details-Beispiel (keine bereits implementierte Antwort):

```json
{
  "type": "https://docs.example.invalid/problems/revision-conflict",
  "title": "Revision conflict",
  "status": 412,
  "detail": "Die Ressource wurde zwischenzeitlich aktualisiert.",
  "code": "REVISION_CONFLICT",
  "request_id": "req-example-001",
  "retryable": false
}
```

Die Domain `docs.example.invalid` ist ausdrücklich ein Platzhalter. Der finale Fehlerkatalog definiert pro Operation erlaubte Status-/Code-Kombinationen und prüft deren Stabilität.

### 5.3 API-Schnittstellenkatalog (bestehende Entwurfsfamilien)

Die bereits in anderen Dokumenten beschriebenen Routenfamilien gelten als **Orientierungs- und Vertragsentwurf**, nicht als Nachweis fertiger APIs:

| Familie | Beispiele aus bestehenden Konzepten | Owner |
|---|---|---|
| Capability-/Systeminformationen | `GET /api/v1/system/capabilities` | Gateway/System |
| Aktueller Benutzer / Sitzungen | `GET /api/v1/users/me` | Identity/Sessions |
| Desktopzustand | `GET/PATCH /api/v1/desktops/current` | Desktop |
| Dateien/Verzeichnisse | `GET /api/v1/files/{id}`, `GET /api/v1/files/{id}/children` | Files/VFS |
| Upload und Abschluss | `POST /api/v1/uploads`, `POST /api/v1/uploads/{id}/complete` | Files |
| Download/Streaming | `GET /api/v1/files/{id}/content` | Files |
| App-Verwaltung | `GET /api/v1/apps`, `POST /api/v1/apps/{id}/launches` | Applications |
| Aufträge | `POST /api/v1/jobs`, `GET /api/v1/jobs/{id}` | Jobs |
| Push | `GET /api/v1/events/stream` | Realtime |
| Einstellungen/Themes | `GET/PATCH /api/v1/settings`, `GET /api/v1/themes/active` | Settings/Theme |

Neue REST-Routen für Offline-Caches, Revisionshistorie, Blocktransfers, Synchronisierungs-Checkpoints, Garbage Collection, Status und Schlüsselwiederherstellung erhalten **erst in der jeweiligen fachlichen OpenAPI-Spezifikation** verbindliche Namen. Apps sehen niemals frei verfügbare interne Speicher-/KMS-/Datenbank-APIs.

## 6. Go-Ports, gRPC und serviceübergreifende Autorisierung

### 6.1 Semantisch identische lokale und entfernte Vertrage

Ein Core-Fachmodul deklariert einen fachlichen Port. Im `combined`-Modus wird er als typisierter lokaler Go-Aufruf realisiert; in verteilten Modi bildet ein gRPC-/Protobuf-Adapter denselben Vertrag ab. Folgende Eigenschaften **MÜSSEN** in beiden Fällen gleichwertig sein:

- Operation und Ressourcenzuordnung, annehmende und liefernde DTOs, Normalisierung und Fehlercodes.
- Aufrufidentität, zugrunde liegende Benutzer-/App-Delegation, Entscheidungsrevision der Rechte und Scope-Grenzen.
- Deadline, Cancellation, Request-/Trace-Korrelation und Idempotenz- beziehungsweise Retryregeln.
- Service-Owner und fachliche Datenhoheit; direkte SQL-Zugriffe auf fremde Serviceschemas bleiben unzulässig.
- Aktivierungsreihenfolge und sicherer Umgang mit ausfallenden Abhängigkeiten.

Ein lokaler Methodenaufruf ist **kein** impliziter privilegierter Bypass. Der Zielservice validiert seine relevanten Fach- und Sicherheitsinvarianten selbst.

### 6.2 mTLS, Service-Capabilities und Delegation

Verteilte Core-Dienstverbindungen verwenden wechselseitiges TLS mit authentifizierter Workload-Identität. Eine DNS-Adresse, Pod-IP oder Containerbezeichnung stellt keine Berechtigung dar. Zusätzlich besitzt der Aufruf eine **kurzlebige, minimal berechtigte Service-Capability**, die nur erforderliche Operationen, Zielservice und gegebenenfalls konkrete Ressourcen umfasst.

Ruft ein Dienst stellvertretend für einen Benutzer oder eine App einen zweiten Dienst auf, wird eine **signierte, abschwächbare (attenuating) Delegation** verwendet. Ihr effektiver Scope kann entlang der Aufrufkette nur gleich bleiben oder kleiner werden. Eine solche Delegation enthält beziehungsweise bindet:

- Ursprungsidentität von Benutzer und App sowie die aktive App-Instanz, soweit anwendbar.
- Aufrufenden Workload, Zielservice, Zieloperation und zulässigen Ressourcenscope.
- Enge Gültigkeitszeit, Audience und eine widerrufs-/sicherheitsrevisionstaugliche Bindung.
- Nachvollziehbare Delegationskette, ohne Zugriffstokens oder Schlüsselmaterial zu protokollieren.

Der Zielservice validiert Signatur, Audience, Gültigkeit, Delegationsgrenzen und aktuelle Widerrufs-/Autorisierungsregeln. Ein bereits ausgestelltes Token darf niemals als Garantie weitergültiger Rechte dienen. Im kombinierten Prozess gilt dieselbe fachliche Identitäts- und Delegationssemantik ohne vorgetäuschte mTLS-Prozessgrenze.

### 6.3 Fehlertoleranz innerhalb der Servicegrenzen

Ein nicht verfügbarer Fachservice liefert einen wohldefinierten Fehler; unabhängige Dienste bleiben benutzbar, soweit der gewählte Betriebsmodus dies technisch zulässt. Circuit Breaker, Ressourcenbudgets, Deadlines und begrenzte Retries verhindern Fehlerkaskaden. Der Zielservice darf niemals fehlende Auth- oder Permission-Dienste als Zustimmung interpretieren.

## 7. Ereignis- und Realtime-Verträge

### 7.1 Dauerhafte Core-Events

Core-Dienste publizieren asynchrone, fachlich relevante Ereignisse über **NATS JetStream**. Der verbindliche Liefervertrag ist **at least once**, nicht exactly once. Wiederholte Zustellung ist normal und muss von Consumern korrekt behandelt werden. Datenbankzustand und Event-Publikation werden über eine **transaktionale Outbox** beziehungsweise ein gleichwertig nachweisbar sicheres Verfahren gekoppelt.

- Jeder Eventtyp hat einen fachlichen Owner, Schema- und Versionskennung sowie einen stabilen Ereignis-Identifier.
- Ereignisse verwenden **CloudEvents 1.0** als standardisiertes Envelope-Modell mit `specversion`, `type`, `source`, `id`, `time` und definiertem Datenschema.
- Consumer führen Seiteneffekte idempotent aus; die Duplikaterkennung und deren Retention sind dokumentiert.
- Reihenfolge wird **pro fachlicher Ressource** durch Revisionen/Sequenzen gesichert; eine universelle Ordnung aller Events ist nicht vorausgesetzt.
- Events tragen nur die für Empfänger benötigten Informationen. Sicherheitskritische oder nutzerbezogene Daten werden nicht pauschal in globale Topics gesendet.
- Inkompatible Event-Schemaänderungen unterliegen denselben Release- und Migrationsregeln wie REST-/gRPC-Verträge.

Illustratives CloudEvents-Objekt, **kein finaler Typname**:

```json
{
  "specversion": "1.0",
  "id": "evt-example-052",
  "source": "/cloudos/files",
  "type": "dev.cloudos.files.node.changed.v1",
  "subject": "node-example",
  "time": "2026-11-15T02:00:00Z",
  "datacontenttype": "application/json",
  "data": {
    "node_id": "node-example",
    "revision": 17,
    "change": "metadata_updated"
  }
}
```

### 7.2 SSE ist der verbindliche Browser-Push-Kanal

`GET /api/v1/events/stream` liefert einen authentifizierten SSE-Strom. Die vertraute Browser-Shell kann **einen** angemessen verwalteten Stream für die aktuelle Sitzung betreiben und ausschließlich genehmigte, validierte Ereignisse über die App-Bridge an `.cloudApp`-Instanzen weitergeben. Ein App-Frame darf nicht mit dem allgemeinen Browser-Sessioncookie den ungefilterten Event-Stream abonnieren.

SSE-Vertrag:

1. Initialer, autorisierter **Snapshot** und registrierte Stream-Position müssen ohne verlorene Deltas aufeinander abgestimmt werden.
2. Jede fachliche Zustellung besitzt eine opake, audiencegebundene Event-ID; bei Verbindungsabbruch erfolgt **Replay über `Last-Event-ID`** beziehungsweise einen gebundenen Resume-Cursor.
3. Reicht die Replay-Historie nicht mehr aus, sendet der Server ein explizites `resync_required`; darauf folgt **eine gezielte erneute Snapshot-Abfrage**, kein periodisches Polling.
4. Filterung erfolgt nach gegenwärtig gültiger Benutzer-, Gruppen-, Space-, App-, Instanz- und Event-Subscription-Berechtigung. Ein alter Replay-Cursor eröffnet keine alten Rechte.
5. Der Client validiert Version, Reihenfolge und Revision, erkennt Verbindungsverlust und kennzeichnet veraltete Daten als solche.
6. Automatische Wiederverbindung verwendet begrenzten Backoff mit Jitter; SSE-Keepalive-Kommentare sind erlaubt, Statusabfrage-Schleifen nicht.
7. Geringe oder keine Konnektivität führt zu klarer Offline-/Degraded-Anzeige und zu den definierten Offline-Workflows.
8. Theme-, Desktop-, Fenster-, Jobs-, Permission-, Health-, Alarm- und Dateisystemänderungen verwenden denselben Sicherheitsstandard.

Als **keinen** verpflichtenden zweiten Push-Kanal führt cloudOS WebSockets ein. Spätere echte bidirektionale Kollaborationsfälle können einen eigens genehmigten, dokumentierten Vertrag erhalten, ohne die allgemeinen SSE-/Berechtigungsregeln zu ersetzen.

### 7.3 Event-Klassen und Adressaten

| Klasse | Beispiele / Verantwortlichkeit | Sichtbarkeit |
|---|---|---|
| Dateisystem | `files.node.changed`, Upload-/Versionierungs-/Papierkorb-Deltas | Nur betroffene autorisierte Benutzer/Apps |
| Desktop und Fenster | Fensteröffnung, -zustand, -schließung; Arbeitsumgebungsrevision | Nur aktive autorisierte Sitzungen des Benutzers |
| Berechtigungen | App-Capability entzogen, Space-Mitgliedschaft geändert, Sitzung widerrufen | Betroffene Instanz/Sitzung; Detailtiefe begrenzt |
| Jobs und Sagas | Laufzeit, Fortschritt, Checkpoint, Fehler, Eingriffsbedarf | Besitzer und berechtigte Betreiber |
| Monitoring und Alarmierung | Health, SLO-Verletzung, Wartung, bestätigter Vorfall | Rollen- und risikobasiert |
| Offline-/Sync-Status | Auftragsfortschritt, bestätigte Revision, Konflikt/Resync | Betroffene Benutzer und Geräte |
| Oberflächenressourcen | Theme-, Wallpaper-, Taskleisten- und Einstellungsdeltas | Nur berechtigte Betroffene |

**Hinweis:** In der Tabelle sind Eventnamen ohne vollständigen Namespace als beschreibende Namen verwendet; eine konkrete vollqualifizierte Typkennung ist Teil des jeweiligen Eventschemas.

## 8. App-SDK, Sandbox, Berechtigungen und Netzwerk

### 8.1 Vollständige Isolation auch für Core-Apps

Jede `.cloudApp`, auch eine vorinstallierte Core-App, läuft innerhalb eines geeigneten isolierten App-Hosts. Der privilegierte Fenster-/Desktop-Host, die SDK-Bridge, API-Broker und fachlichen Security-Services befinden sich **ausserhalb** des nicht vertrauenswürdigen App-Kontexts. Die App darf nicht direkt auf Go-Core-Ports, NATS, gRPC, PostgreSQL, Storage-Adapter, Schlüsselmaterial oder die allgemeine Browsersitzung zugreifen.

Browserisolierung wird durch geeignete Origins, Sandbox-Attribute, CSP, strikt definierte `postMessage`-/MessagePort-Grenzen, Hostvalidierung und weitere passende Browsermechanismen unterstützt. **Eine Browser-Sandbox ist keine vollständige Garantie gegen einen kompromittierten Browser oder alle möglichen Browser-Egress-Kanäle.** Das Bedrohungsmodell muss technisch realisierbare Grenzen ehrlich dokumentieren; serverseitige Verträge dürfen sich niemals allein auf clientseitige Sperren verlassen.

### 8.2 SDK-only und Capability-Schnittmenge

App-Code **MUSS** Core-Fachoperationen ausschließlich über das offizielle cloudOS SDK aufrufen. Es gibt keinen legitimen Direktzugriff per eigener REST-URL, WebSocket, EventSource, gRPC oder selbstgebautem Transport, um die Bridge zu umgehen. Das SDK kann JavaScript, TypeScript oder Rust benutzen und muss transportneutrale fachliche Operationen bereitstellen.

Die wirksame Berechtigung ist die Schnittmenge aus:

1. Signierten, kataloggültigen Operationsdeklarationen des installierten `.cloudApp`-Pakets.
2. Aktiver administrativer Genehmigung der konkreten App-Version und API-Operationen.
3. Wirksamem App-Instanz-/SDK-Capability-Kontext mit engen Ressourcenscopes.
4. Aktueller Identität, Gruppen/Rollen, VFS-/Space-Rechten und Sicherheitsrichtlinien.
5. Gegebenenfalls expliziter, aktiver **Benutzerfreigabe für eine konkrete Ressource**.
6. Nicht widerrufener Sitzung, geprüfter Delegation und erlaubtem Zielservice.

Eine gesonderte Laufzeitfreigabe für etwa eine vom Benutzer ausgewählte Datei oder ein USB-Gerät darf **nur die bereits installierten und genehmigten Operationen konkretisieren**, niemals neue API-Operationen hinzufügen.

### 8.3 Zusätzliche API-Rechte ausschließlich durch signiertes Update

Eine App darf **keine neue API-Operation während ihrer Laufzeit nachbeantragen oder freischalten**. Braucht ein Update weitere API-Endpunkte, ist ein neues **gültig signiertes App-Paket** mit entsprechend geänderter `permissions/api-access.json` erforderlich. Vor Aktivierung muss ein berechtigter Administrator den vollständigen Berechtigungsdiff genehmigen. Alte Paketsignaturen, unbestätigte App-Dialoge oder JS-seitig deklarierte Permissions gelten nicht.

Wird einer laufenden App eine **einzelne** Operation entzogen, werden nur die davon abhängigen Funktionen deaktiviert. Andere genehmigte Funktionen und das App-Fenster können weiterlaufen. Der Widerruf wird **sofort serverseitig** wirksam; die betroffene App bekommt zusätzlich eine SSE-Mitteilung und kann den aktuellen Stand ihrer **eigenen** effektiven Rechte über das SDK ermitteln. Eine vollständige App-Start-/Zuweisungsentziehung bleibt ein eigener Lebenszyklusfall und kann die Instanz insgesamt beenden.

### 8.4 Ressourcenzustimmung und ihre Dauer

Sensible Ressourcen (beispielsweise bestimmte Dateien, Ordner, Mediengeräte oder geeignete Browsergerätezugriffe) benötigen – zusätzlich zu genehmigten App-Operationen – einen vertrauenswürdigen Nutzerentscheid, sofern der Risikokatalog dies vorsieht. Consent wird über einen vom Core kontrollierten Dialog eingeholt, an Ressource/App/Benutzer/Sitzung gebunden und gegen Confused-Deputy-Angriffe abgesichert.

Je nach Risiko und Ressourcentyp kann die Zustimmung **einmalig, sitzungsbezogen oder dauerhaft** gelten, soweit die Policy es erlaubt. Sie ist zentral widerrufbar und wird beim relevanten Aufruf gegen die aktuelle Sicherheitsrevision geprüft. Kritische, hochsensible oder lokal gerätegebundene Freigaben erhalten besondere Grenzen und dürfen nicht auf andere Geräte kopiert werden.

### 8.5 Externes Netzwerk ausschließlich über Network Broker

App-gesteuerte ausgehende Netzwerkkommunikation verläuft **über einen Core Network Broker**. Der Broker erzwingt genehmigte Zieladressen/Hosts, Methoden, Protokolle, Content- und Größenlimits, Rate-Limits, Credential-Handling und Audit. Direkte App-`fetch`-Aufrufe zu beliebigen Hosts sind kein legitimierbarer App-API-Weg. Browserseitige CSP/Origin-Isolation beschränkt mögliche Umgehungen nach technischem Stand; unmögliche Sicherheitsgarantien werden nicht behauptet.

Der Broker verhindert insbesondere die unkontrollierte Nutzung von Benutzer-Cookies oder vom App-Code bereitgestellten internen Hosts als beliebiges Proxy-Ziel. Zugang zu externen APIs ergibt sich nicht schon aus der Installation einer App.

## 9. Asynchrone Aufträge, Wiederholungen und Sagas

### 9.1 Synchron oder asynchron

Kurze, begrenzte Operationen können synchron beantwortet werden. Lange, ressourcenintensive, mehrstufige oder wiederaufnehmbare Aktionen werden als **persistente Jobs** mit einer opaken `job_id`, Status, Fortschritt, Eigentümerkontext, zulässigen Aktionen und Ergebnissen modelliert. Ein Job kann in der fachlichen Antwort per `202 Accepted` plus Zustandsreferenz zurückgegeben werden; **Statusänderungen werden per SSE gepusht**, nicht periodisch abgefragt.

Jeder Job ist explizit einer Kategorie zugewiesen:

- **Durable:** Kann unabhängig von Browserfenster und App-Sitzung fortlaufen, solange Sicherheits- und Owner-Policy dies erlauben.
- **Session-bound:** Wird nach dem Ende oder Widerruf der zugrundeliegenden Sitzung kontrolliert abgebrochen beziehungsweise sicher beendet.

Die Core-Richtlinie legt für jede Jobart die Kategorie, Stopp-/Cancel-Semantik, Ressourcenbudgets und Autorisierung fest. Eine App darf nicht beliebige Serverprogramme oder frei definierte privilegierte Worker starten.

`Tasks.cloudApp` ist eine vorinstallierte Core-App für autorisierte Übersicht, Fortschritt, Abbruch und berechtigte administrative Eingriffe. Die Taskleiste kann laufende Aufträge anzeigen. Die App ist ein Viewer/Controller; Jobs bleiben auch ohne geöffnetes Fenster persistent im Core.

### 9.2 Wiederaufnahme, Leases und Retry-Sicherheit

Persistente Jobs besitzen geeignete dauerhaft gespeicherte **validierte Checkpoints**, Besitzer, Versuchszahlen, Lease-/Fencing-Informationen und Idempotenzreferenzen. Nach Worker-Ausfall oder Core-Neustart kann die Verarbeitung am letzten bestätigten Checkpoint wiederaufgenommen werden. Bei mehreren Replikaten darf ein Job nicht wirksam doppelt ausgeführt werden, nur weil zwei Worker ihn gleichzeitig beanspruchen.

- Wiederholungen erfolgen nur für klassifizierte, vorübergehende Fehler und mit begrenzter Anzahl, Frist, Backoff und Jitter.
- Wiederholbare Writes besitzen einen nachweisbar wirksamen Idempotenzschutz; reine Read-Operationen können unter dokumentierten Bedingungen sicher neu angefordert werden.
- Nach Ablauf des Retry-Budgets folgt ein klarer Fehler- oder Eingriffsstatus, keine Endlosschleife.
- Status und Checkpoints dürfen keine Geheimnisse oder unerlaubten Dateiinhalte an Aufgabenbetrachter offenlegen.
- Abbruch und Rücknahme sind fachlich verschieden: Ein abgebrochener Job kann bereits wirksame Teiloperationen besitzen und braucht gegebenenfalls eine Kompensation.

### 9.3 Serviceübergreifende Saga-Verträge

Ein serviceübergreifender Workflow verwendet eine **persistente Saga** statt einer vorgetäuschten verteilten ACID-Transaktion. Jeder Fachservice führt seine eigene atomare lokale PostgreSQL-Transaktion aus, erzeugt den nötigen Outbox-Eintrag und stellt dokumentierte, idempotente Ausführungs-/Kompensationsoperationen bereit.

Ein Saga-Vertrag definiert mindestens `saga_id`, Owner, betroffene Ressourcen, Schritte, aktuelle Revision, Checkpoints, Zeitbudgets, Versuchszahlen, Kompensationsstrategie und Verantwortlichkeit für den Abschluss. Kann die Saga nach begrenzten Wiederholungen nicht sicher abschließen, werden bereits ausführbare Kompensationen angewandt und die Saga in einem geschützten, nachvollziehbaren Fehlerzustand geparkt. **Keine unkontrollierte manuelle Datenbankkorrektur**: Ein berechtigter Administrator kann einen gezielten, auditierten Eingriff über `Tasks.cloudApp` veranlassen.

Konzeptionelle Zustandsfolge:

```text
CREATED -> RUNNING -> COMPLETED
             |\
             | +-> WAITING_RETRY -> RUNNING
             +---> COMPENSATING -> COMPENSATED
                                   |\
                                   +-> INTERVENTION_REQUIRED
```

Für jede fachlich irreversibel wirkende Teiloperation muss ein separater, ausdrücklicher Fehler- und Recovery-Vertrag vorliegen.

## 10. Listen, Streaming, Limits und Fairness

### 10.1 Listen und grosse Datenströme

- Listen verwenden Cursor, stabile Sortierung und berechtigungssichere Seitengrenzen. Eine Verschiebung von Daten darf keine stillen Duplikate, ausgelassenen Einträge oder Sichtbarkeitslecks durch den Cursor erzeugen.
- Grosse Datei-/Exportdaten werden als Stream mit korrekter Abbruchbehandlung, Backpressure, Quota und Integritätsprüfung geliefert. SSE ist für **Zustandsänderungen**, nicht für beliebige Nutzdaten-Downloads vorgesehen.
- Clients dürfen nach einer echten Initialisierung oder Resync-Lücke eine einzelne gezielte Zustandsabfrage ausführen. Sie dürfen nicht in wiederkehrenden Intervallen nach neuen Dateilisten, Jobs oder Benachrichtigungen suchen.

### 10.2 Hierarchische Rate-Limits

cloudOS führt **dynamische, hierarchische** Rate- und Ressourcenlimits auf System-, Service-, Benutzer-, App-, Instanz-, Operation- und bei Bedarf Ressourcenscope-Ebene ein. Eine zulässige Anfrage muss alle einschlägigen Budgetebenen erfüllen.

- Kritische System- und Auth-Funktionen erhalten angemessene Schutz- und Prioritätsklassen.
- Fairness verhindert, dass eine einzelne App oder ein Benutzer alle Ressourcen verbraucht.
- Schreiboperationen und grosse Uploads besitzen von kleinen lesenden Aufrufen getrennte Budgets.
- Limits sind für mehrere Kubernetes-Replikate **konsistent** zu koordinieren, nicht als voneinander unabhängige unlimitierte pro-Pod-Zähler zu behandeln.
- Ablehnungen geben einen vertraglichen Rate-Limit-Fehler, nach Bedarf einen begrenzten `Retry-After`-Hinweis, und erzeugen geeignete Metriken.
- Administrativ geänderte Limits werden versioniert, auditiert und ohne unsichere Sonderrechte wirksam.

## 11. Logging, Audit und Distributed Tracing

### 11.1 Unterschiedliche Datenklassen

| Datenklasse | Pflicht und Zugriff | Steuerung |
|---|---|---|
| Security-Audit | Verpflichtend für Auth-, Permission-, App-Signing-, Sicherheits- und Schlüssel-/Recovery-Vorgänge | Nicht durch normale Log-Level deaktivierbar |
| API-Diagnostik | Strukturierte API-Aufrufe, Fehler, Latenzen, Rate- und Retry-Ereignisse | Pro Kategorie/Dienst/Endpoint mit Level und Sampling |
| Fach-/App-Diagnostik | Nur erlaubte technische Details und App-Scopes | Signierte Operationsrechte; keine frei global lesbaren App-Logs |
| Betriebstelemetrie | Metriken, Traces, Ressourcen und SLO-Werte | Eigene Retention-/Sampling-Policy |

Aufbewahrungsdauer, Volumen-/Speicherbudgets, Bereinigung und gegebenenfalls Archivierung sind **je Datenklasse** konfigurierbar. Security-Audit besitzt strengere Schutz-, Integritäts- und Aufbewahrungsregeln als abschaltbare Diagnoselogs. Eine ablaufende Retention darf gesetzte Mindestanforderungen nicht stillschweigend umgehen.

Der Zugriff auf Logs wird nach Benutzer, Gruppe, Logkategorie, Dienst und Datenbereich granular autorisiert. **Einsehen**, **Konfigurieren** und **Auditieren/Exportieren** sind getrennte Rechte. Einsichtnahmen in sensible Logdaten werden selbst auditiert. `Logs.cloudApp` bleibt die geschützte Core-App für berechtigte Log- und Trace-Ansichten.

### 11.2 OpenTelemetry und W3C Trace Context

REST, Go-Ports, gRPC, NATS-Events, Jobs und Sagas propagieren nachvollziehbare Korrelationsinformationen gemäß **W3C Trace Context**. OpenTelemetry standardisiert Traces/Spans und die weiteren Observability-Signale. Sampling, Trace-Aufbewahrung und Export lassen sich zentral steuern. Traces dürfen insbesondere keine Passwörter, Access-/Refresh-Tokens, privaten Dateiinhalte, unnötige Nutzerkennungen oder Schlüssel enthalten.

- Trace-/Span-IDs erlauben die Rekonstruktion einer verteilten Verarbeitung, ersetzen aber nicht die gesonderte Security-Audit-Kette.
- Asynchrone Folgeereignisse und Wiederholungen müssen fachlich korrekt verknüpft werden; keine ungeprüften, vom Client eingeschleusten Trace-Felder als Autorisierung verwenden.
- Zulässige Administratoren können einen Request vom Gateway über Serviceaufrufe bis zu Event-/Job-Consumer-Schritten verfolgen.
- Ein Telemetrieausfall darf keine geheimen Daten ausgeben und darf nicht dazu führen, dass die API in ungesicherten Debug-Modus fällt.

## 12. Monitoring, Health Checks, automatische Reaktion und Alarmierung

### 12.1 Verbindliches Monitoringmodell

cloudOS erfasst über OpenTelemetry und geeignete Infrastruktur-Adapter **Serviceverfügbarkeit, CPU/RAM, Antwortzeiten, Fehlerquoten, Durchsatz, Queue-Lag, Job-/Saga-Stände, SSE-Verbindungen, API-Gateway-Verhalten sowie PostgreSQL- und NATS-Zustand**. Die Metriken sind service- und rollenbezogen auswertbar; historische Reihen und **Service-Level-Objectives (SLOs)** unterstützen Trendanalysen und Kapazitätsplanung. Die technische Status- und Monitoring-Darstellung darf nur nach berechtigter Anmeldung Details enthalten.

Eine geschützte `Monitoring.cloudApp` ist als Core-App für diese Auswertungen vorgesehen. Die Alarm-, Export- und Regelengine muss unabhängig davon laufen, ob die grafische App geöffnet ist. HPA/Autoscaling ist als Grundlage für spätere Kubernetes-Profile vorzusehen, **nicht** als Pflicht zum sofortigen verteilten Core-Deployment.

### 12.2 Abgestufte Selbstheilung mit Sicherheitsgrenzen

cloudOS darf bei klar definierten, risikoarmen Störungen **automatisch** eingreifen:

| Störung | Erlaubte automatische Massnahmen | Sicherheitsgrenze |
|---|---|---|
| Einzelne nicht reagierende Replik | Readiness entziehen, gesunde Replik verwenden, kontrollierter Neustart/Ersatz | Keine Endlos-Neustartschleife |
| Erhöhte Last | Begrenzte Skalierung/Backpressure, Queue-Priorisierung | Nur innerhalb autorisierter Ressourcenbudgets |
| Vorübergehender Servicefehler | Bounded Retry, Circuit Breaker, zeitbegrenzter Fallback | Keine Sicherheitsprüfung umgehen |
| Unterbrochener langlebiger Job | Checkpoint-validierte Wiederaufnahme | Lease/Idempotenz und Rechte erneut prüfen |
| Datenbankinkonsistenz, Datenkorruption oder fragliche Schlüssel | Stoppen/isolieren und eskalieren | **Keine** eigenmächtige destruktive Datenreparatur |
| Sicherheitskritische Richtlinien-/Konfigurationsänderung | Alarmieren, Vorschlag/diagnostischer Read-only-Modus | Administratorfreigabe bei riskanten Änderungen |

Jede automatische Massnahme besitzt Auslöser, Sicherheitsklassifikation, Retry-/Cooldown-Budget, maximalen Wirkbereich, Beobachtungsfenster, Erfolgsprüfung und Auditspur. Schlägt sie wiederholt fehl, darf sie nicht endlos fortgesetzt werden; cloudOS geht in einen sicheren Eskalationszustand. Die Möglichkeit einer administrativen Freigabe ist nicht mit einem automatischen Rollback-/KMS-/DB-Bypass gleichzusetzen.

### 12.3 Alarmkanäle, Eskalation und Unterdrückung

Alarmregeln sind nach **Ereignistyp, Schweregrad, Empfänger, Gruppe, Systemzustand und Zeitfenster** konfigurierbar. Verbindliche Kanäle sind interne SSE-/Desktop-Benachrichtigungen und E-Mail; optionale Webhooks erlauben spätere Integrationen. Kritische Alarme können eine Bestätigung mit dokumentierter Frist verlangen. Fehlt diese, folgt eine regelbasierte Eskalation an berechtigte Empfänger.

| Schweregrad | Typisches Verhalten |
|---|---|
| Info | In `Monitoring.cloudApp` sichtbar |
| Warning | SSE an zuständige Betreiber, optional E-Mail |
| Error | Gezielter Alarm mit Diagnose-/Incident-Referenz |
| Critical | Unverzügliche Zustellung, Bestätigung und Eskalation |
| Security Critical | Gesonderte Sicherheitsadressaten, verpflichtendes Audit, **nie** durch Wartung unterdrücken |

Duplikate werden korreliert und gedrosselt. Zustellung über E-Mail/Webhook verwendet dauerhaft nachvollziehbare Versandaufträge, begrenzte Retries und kontrollierte Zustellfehler. Ein ausgefallener SSE-Transport darf nicht den einzigen möglichen Meldeweg für kritische Ereignisse bilden. Die eigentliche Empfängerliste und technische Zustellimplementierung werden separat spezifiziert.

### 12.4 Geplante Wartungsfenster

Ein Wartungsfenster kann manuell geplant werden und einzelne Dienste, Servicegruppen oder das Gesamtsystem betreffen. Es besitzt Anfang, Ende, optionales Wiederholungsmuster, begründeten Verantwortlichen und dokumentierten Scope. Erwartete Wartungsstörungen werden **weiterhin erfasst**, aber in der Benachrichtigung gezielt zurückgehalten. Nicht erwartete, ausserhalb des Scopes liegende und sicherheitskritische Ereignisse werden weiter alarmiert. Ein abgelaufenes Fenster endet automatisch; eine unbefristete unbemerkte Unterdrückung ist unzulässig.

### 12.5 Health-Status und K8s-Probes

Startup-, Liveness- und Readiness-Prüfungen sind getrennt von fachlichen **Functional-Health-Checks**. Das Zustandsmodell umfasst mindestens:

| Zustand | Fachliche Interpretation | Betriebsfolge |
|---|---|---|
| `starting` | Initialisierung noch nicht abgeschlossen | Keine regulären neuen Anfragen |
| `healthy` | Vorgesehene Funktionen erfüllbar | Normalbetrieb |
| `degraded` | Teilfunktionen beeinträchtigt, andere verfügbar | Selektive Fehlermeldung; keine unbedingte Neustartschleife |
| `not_ready` | Instanz momentan kein geeignetes Ziel | Von neuer Last fernhalten |
| `unhealthy` | Ernsthafte Störung / Betrieb nicht korrekt | Bounded Remediation gemäß Policy |
| `maintenance` | Geplantes, transparentes Wartungsfenster | Erwartete Alarme korrekt kennzeichnen |

Ein Service kann **fachlich degraded und dennoch liveness-gesund** sein. Readiness-Ausfall ist keine generelle Aussage, dass ein Prozess neu gestartet werden muss. Im `combined`-Modus existieren Health-Angaben **pro logischem Modul**, auch wenn nur der Gesamtprozess neu gestartet werden kann.

### 12.6 `Status.cloudApp` und Sichtbarkeit

`Status.cloudApp` ist eine **vorinstallierte, sandbox-isolierte Core-App** mit rollenbasierten Ansichten:

- Normale Benutzer sehen zugeordnete **Funktionen** wie Dateiablage, Anmeldung oder Synchronisierung als verfügbar, eingeschränkt oder nicht verfügbar sowie relevante Wartungshinweise.
- Berechtigte Administratoren können die zugehörigen technischen Dienstzustande, Abhängigkeiten, Replikate und Alarme untersuchen.
- Kritische Störungen werden zusätzlich in geeigneten Desktop-/Taskleistenanzeigen hervorgehoben.
- Änderungen kommen über das sicherheitsgefilterte SSE-System. Bei Verbindungsverlust wird der zuletzt bekannte Zustand deutlich als **veraltet/unbestätigt** angezeigt.
- Administratives Monitoring und normaler Benutzerstatus sind unterschiedliche Rechte. Eine Health-API ermöglicht keine unauthorisierte Kenntnis interner Infrastruktur.

## 13. Sitzungen, Fenster, Auto-Save und gerateübergreifender Desktop

### 13.1 Verhalten bei Core-/Service-Ausfall

Ein nur teilweise gestörter Core darf bestehende Browserfenster und nicht betroffene Apps **nicht grundlos schließen**. Die Shell hält den lokalen Fenster- und Arbeitszustand sichtbar, zeigt betroffene Teilfunktionen als eingeschränkt und verbindet sich nach Wiederherstellung kontrolliert erneut. Nach einem Core-Prozessneustart soll der Desktop ohne Browserneustart wieder nutzbar werden, soweit Browser und gültige Sitzung bestehen.

Ein Widerruf, eine abgelaufene Sitzung oder eine fehlende Berechtigungsentscheidung hat Vorrang vor Komfort und Verfügbarkeit. Unsicher autorisierbare API-Operationen werden abgelehnt, bis die Autorisierung wieder zuverlässig möglich ist.

### 13.2 Vollständige Sitzungswiederherstellung

Das cloudOS Framework bietet eine standardisierte Wiederherstellung für **Fensterpositionen, Fenstergrößen, Minimierungs-/Fokuszustände, gestartete Apps, App-Navigation, ausgewählte Dateien und unterstützte ungespeicherte Arbeitsstände**. App-spezifische Inhalte werden über klar begrenzte SDK-Serialize-/Restore-Verträge verarbeitet. Nur Apps mit entsprechender Unterstützung können fachlich beliebige Inhalte rekonstruieren.

Nach einem Browser- oder Client-Absturz wird ein gültiger Checkpoint angeboten beziehungsweise nach Policy wiederhergestellt. Bei bereits abgelaufener Sitzung muss zuerst eine neue gültige Anmeldung stattfinden. Die Wiederherstellung validiert aktuelle App-Versionen, Operationsrechte, Ressourcen-Handles und Revisionskonflikte. **Eine automatische, garantiert verlustfreie Rekonstruktion beliebiger, noch nie persistierter Eingaben ist technisch nicht möglich** und darf nicht versprochen werden.

### 13.3 Hybrides intelligentes Auto-Save

Das Framework speichert nach einer kurzen Änderungs-/Eingabepause, bei geeigneten wichtigen Aktionen (Fensterwechsel, Minimierung, expliziter Save-Checkpoint) sowie durch begrenzte zeitliche Sicherheits-Checkpoints. Speicherlast, Änderungsgröße und System-/Netzwerkkapazität bestimmen die gebündelte Frequenz.

- Auto-Save darf bei einer reinen Eingabe keine unkontrollierte Welle simultaner Server-Writes auslösen.
- Gesicherte, unsynchronisierte und fehlgeschlagene Speicherstände sind **sichtbar unterscheidbar**.
- Jede serverseitige Schreiboperation respektiert Revisionen, aktuelle Berechtigungen, Quotas und Idempotenz.
- Browserseitige UI-Events, die selbst Änderungen erzeugen, sind kein Status-Polling.
- Checkpoint-Fehler dürfen nicht als erfolgreiche Sicherung angezeigt werden.

### 13.4 Hybride Checkpoint-Speicherung und Historie

Der Server führt die **dauerhafte, gerateübergreifende** Quelle für wiederherstellbare Arbeitsstände. Ein geschützter lokaler Browser-Cache kann bei fehlender Verbindung temporäre, noch nicht synchronisierte Änderungen aufnehmen. Die Replikation verwendet Auth-, Integritäts-, Revisions- und Konfliktprüfungen. Es wird **kein neues drittes virtuelles Laufwerk** benötigt: benutzerbezogene Daten bleiben fachlich im `users`-Volume bzw. der dortigen Home-/App-Datenverwaltung; die konkrete interne physische Ablage bestimmt der jeweilige Owner.

Die **Wiederherstellungshistorie ist konfigurierbar** nach Aufbewahrungsdauer, maximaler Checkpointzahl, Speichergrenzen, App, Benutzer und Gruppe. Berechtigte Benutzer können ältere Stände einsehen und gezielt wiederherstellen. Geschützte Checkpoints können nach Richtlinie aus der normalen Bereinigung ausgenommen werden. Eine Wiederherstellung verwendet neue Revisionen oder explizite Konfliktauflösung statt still die aktuelle Version zu ersetzen.

30 Tage und 100 Checkpoints pro App-Arbeitsstand sind lediglich **diskutierte Vorschlagswerte**, keine verbindlichen Produktegrenzen. Die Dateiversionshistorie des File Service ist von der Auto-Save-Historie zu unterscheiden.

### 13.5 Eine gemeinsame Desktop-Umgebung auf mehreren Geräten

Ein Benutzer darf gleichzeitig mit unterschiedlichen gültigen **gerätespezifischen Sitzungen** angemeldet sein. Seine **logische Desktop-Arbeitsumgebung** (App-Instanzen in ihrer fachlichen Zuordnung, gemeinsamer Fenster-/Workspacezustand und unterstützte App-Arbeitsstände) wird auf allen berechtigten Geräten in Echtzeit abgeglichen.

- Öffnen und Schließen einer App beziehungsweise eines gemeinsamen Fensters wird als autorisierte Operation mit eindeutiger Event-/Revisionssemantik synchronisiert.
- Die **Darstellungsprojektion** auf Bildschirmgröße, Zoom, Eingabemethode und lokale Fensterkoordinaten darf gerätespezifisch bleiben; eine kleine Tablet-Auflösung darf das Layout eines grossen Monitors nicht unbrauchbar machen.
- App-Zustand und Desktopzustand sind getrennte Eigentümerdomänen; Geräte werden nicht mit denselben Sitzungstokens, lokalen Schlüsseln oder Hardware-Consents ausgestattet.
- Die Server- und Event-Verträge verhindern dauerhafte Divergenz bei gleichzeitiger Bedienung; Clients können temporär ausstehen und gleichen nach Reconnect an einer bekannten Revision ab.

### 13.6 Aktive Sitzungen und Geräteverwaltung

Benutzer sehen ihre aktiven Sitzungen und angemeldeten Geräte mit geeigneten Sicherheitsinformationen wie Name/Typ, Anmeldung, letzter Aktivität und Status. Sie können **einzelne Sitzungen oder alle Sitzungen** widerrufen. Neue Anmeldungen erzeugen nach Policy nachvollziehbare Benachrichtigungen. Administratoren können Sicherheitsrichtlinien für Sitzung und Gerätezugriff konfigurieren.

Ein Sitzungswiderruf wirkt **serverseitig sofort**: neue API-Anfragen, nicht mehr autorisierte Event-Zustellungen, neue Leases und serverseitige Sync-Schreibvorgänge werden abgelehnt. Ein SSE-Widerrufshinweis verbessert die UI, ist aber nicht die Sicherheitsgrenze. Andere nicht widerrufene Sitzungen desselben Benutzers bleiben funktionsfähig. Längere lokale Offline-Freigaben unterliegen einer eigenständigen beschränkten Gültigkeit (Kapitel 14).

### 13.7 Echtzeit-Zusammenführung konkurrierender Änderungen

cloudOS bietet über das SDK **CRDT-/OT-taugliche Synchronisierungsverträge** für dafür geeignete Datenmodelle. Zusammenführbare Operationen werden automatisch, nach dokumentierter Reihenfolge und korrektem Ownership-Kontext integriert. Das gilt für kollaborative Text-/Strukturdaten, wenn die jeweilige App sie entsprechend modelliert.

- Gerätespezifische Fenstermasse/-positionen werden als Projektion und nicht als fachliche Inhaltskonflikte behandelt.
- Gleichzeitiges Ändern desselben fachlichen Zustands braucht versionierte Operations-IDs, deduplizierte Einspielung und gegebenenfalls einen geeigneten CRDT-/OT-Algorithmus.
- **Binärdateien und nicht mergefähige App-Zustände** werden nicht aufgrund des Stichworts CRDT automatisch zusammenführbar; die App muss gesonderte Konfliktauflösung bereitstellen.
- Neue oder widerrufene Benutzerrechte werden nicht durch bereits ausgehandelte Synchronisierungsoperationen übergangen.
- Es gilt ein konvergenter, serverseitig autorisierter Zustand; SSE liefert bestätigte Deltas und Wiederaufnahmeinformationen.

## 14. Offline-First, lokale Sicherheit und Synchronisierungsjournal

### 14.1 Abgestufte Offline-Verfügbarkeit

cloudOS ist **offline-first für geeignete Funktionen**, nicht eine vollständige lokale Kopie aller serverseitigen Dienste. Solange erforderliche Ressourcen bereits sicher lokal vorhanden sind, können Desktop-Shell, Startmenu, Taskleiste, Fensterverwaltung und zugelassene App-Ansichten ohne Verbindung weiterlaufen. Apps deklarieren in ihrem signierten Manifest, welche Offline-Funktionen und Datentypen sie unterstützen.

| Bereich | Ohne Verbindung |
|---|---|
| Desktop und Fensterverwaltung | Aus lokal verfügbaren Assets startbar und weiter bedienbar |
| Offlinefähiger Editor | Bereits erlaubte lokale Dokumentkopie bearbeiten, Änderungen journalisieren |
| Dateimanager | Ausgewählte gecachte Dateien/Ordner anzeigen, Sync-Zustand kenntlich machen |
| `Tasks.cloudApp` | Letzten bekannten Jobstatus als **nicht aktuell** darstellen |
| `Status.cloudApp` | Offline-/Verbindungsverlust anzeigen, keine falschen Live-Werte |
| Theme-/Darstellungsoptionen | Lokale Anzeigeänderungen, sofern Assets/Richtlinie vorhanden |
| Benutzer-/Gruppen-/API-Rechteverwaltung | Keine serverpflichtigen privilegierten Änderungen offline |
| KMS-/Key-Recovery oder sicherheitskritische Adminaktionen | Nicht offline ausführbar |

Startet der Browser nach einem Absturz komplett offline, ist die Wiederherstellung nur möglich, wenn die notwendigen Shell-/App-Assets zuvor kontrolliert und kompatibel lokal gecacht wurden. Abhängigkeiten und Browsergrenzen sind im UI sichtbar zu machen.

### 14.2 Geschütztes lokales Änderungsjournal

Unterstützte Apps legen Offline-Änderungen als **versioniertes, integritätsgeschütztes und idempotent einspielbares Journal** ab. Jeder Eintrag verweist auf App/Ressource/Operation, Basisrevision, lokale Reihenfolge und eine deduplizierbare Operation-ID. Die eigentlichen Daten sind auf den erforderlichen Umfang begrenzt. Ein lokaler Status unterscheidet mindestens `pending`, `syncing`, `confirmed`, `conflict`, `rejected` und `failed`.

Nach Wiederverbindung:

1. Die Sitzung und die autorisierte App-Instanz werden gültig authentifiziert; ist sie widerrufen, darf keine automatische serverseitige Übernahme stattfinden.
2. Aktuelle App-Capabilities, Benutzer-/Gruppen-/Space-Rechte, Ressourcenzustand, Offlinerevisionen und Quotas werden erneut geprüft.
3. Eintrag für Eintrag oder als fachlich sichere Transaktion werden Änderungen idempotent verarbeitet; CRDT-/OT-fähige Änderungen können automatisch zusammengeführt werden.
4. Nicht mergefähige Konflikte erfordern eine App-/Benutzerentscheidung. Ablehnungen werden angezeigt und übergeben keine unerlaubten Daten an andere Ressourcen.
5. Erst eine serverseitig bestätigte Revision gilt als **dauerhaft synchronisiert**. Bis dahin bleiben lokale Änderungen als solche gekennzeichnet.

### 14.3 Clientseitige Verschlüsselung, Gerätebindung und Offline-Freigabe

Schutzbedürftige Offline-Daten werden **clientseitig verschlüsselt** und pro Benutzer, Gerät und App isoliert. Der Offline-Zugriff ist von einer **separaten lokalen Entsperrung** abhängig und auf administrativ erlaubte Datenklassen und Gültigkeitszeiträume begrenzt. Secrets, permanente Sitzungstokens oder frei exportierbares Schlüsselmaterial dürfen nicht als normaler Arbeitsstand gespeichert werden.

Die technische Schlüsselbindung an Browser-/Gerätesicherheitsfunktionen darf nur beansprucht werden, soweit die jeweilige Plattform dies wirklich garantiert. Eine Browser-WebCrypto-Schlüsselreferenz ist **nicht automatisch** identisch mit einem hardwaregebundenen, nicht exportierbaren TPM-Schlüssel. Das konkrete Sicherheitsprofil muss die erreichbaren Browser-/OS-Fähigkeiten prüfen und Fallbacks begrenzen.

Ein **serverseitiger Remote-Widerruf** wird sofort für Serveraktionen wirksam, kann aber ein zum Widerrufszeitpunkt wirklich offline getrenntes Gerät **nicht augenblicklich fernsteuern oder sicher löschen**. Deshalb erfordert die lokale Offline-Freigabe eine unabhängig durchsetzbare kurze/konfigurierbare Geltungsdauer, erneute lokale Entsperrung und nach Wiederverbindung eine neue Serverprüfung. Bereits lokal entschlüsselte oder vor dem Widerruf exportierte Klartextkopien lassen sich nicht rückwirkend vernichten.

### 14.4 Hierarchisches lokales Speichermanagement

Der lokale Speicher-Broker verwaltet **globale, benutzer-, geräte- und appbezogene** Speicherbudgets sowie Datenklassen und Prioritäten. Administrativ festgelegte Regeln begrenzen die Offline-Haltung. Benutzer können ihren Verbrauch, gewählte Offline-Ordner und ausstehende Änderungen sehen.

| Klasse | Schutz / Bereinigung |
|---|---|
| Ungesicherte, nicht serverseitig bestätigte Benutzerarbeiten | Höchste Priorität; nicht still löschen |
| Aktuelle Auto-Save-Checkpoints | Hoch; Retention und Wiederherstellungsrichtlinie |
| Ausdrücklich offline markierte Dateien | Hoch; solange erlaubt und Kapazität vorhanden |
| Ältere synchronisierte Versionen | Nach alter und Quota konfigurierbar |
| Reproduzierbare App-Assets und Ansichten | Bei Speicherbedarf bevorzugt ersetzbar |
| Temporäre Dateien | Regelmäßig bereinigen |

Vor der Bereinigung wird geprüft, ob Daten bereits dauerhaft bestätigt sind. Reicht der lokale Speicher nicht aus, muss Auto-Save **einen kontrollierten Fehler** melden und den Benutzer warnen. Kein Browser-API garantiert, dass der Browser oder das Betriebssystem Caches nie eigenständig entfernt; das UI darf daher keine absolute Offline-Verfügbarkeit behaupten.

### 14.5 Selektive Offline-Dateien und -Ordner

Der Dateimanager bietet die Markierung einzelner Dateien und **rekursiver Ordner** als offline verfügbar an. Die Auswahl ist **gerätespezifisch**, nicht automatisch auf alle Geräte zu kopieren. Für jeden Sync-Auftrag werden Speicherbedarf, Download-Fortschritt, Version, Integrität und Status erfasst. Neue Unterdateien können innerhalb genehmigter Regeln automatisch berücksichtigt werden. Veränderte oder widerrufene Rechte werden bei einer Verbindung neu geprüft; eine veraltete Offline-Kopie ist niemals eine neue serverseitige Leseberechtigung.

### 14.6 Adaptive Synchronisierung und Bandbreite

Synchronisierung wird über globale und appbezogene Budgets sowie die Netzwerkbedingungen gesteuert. Benutzer und Administratoren können Upload-/Downloadgrenzen definieren; volumenbegrenzte Netze können ausdrücklich gekennzeichnet werden. Unsichere, ausstehende Benutzerarbeiten erhalten höhere Priorität als grosse Hintergrunddownloads.

| Priorität | Beispiel | Verhalten |
|---|---|---|
| Kritisch | Noch nicht bestätigte Dokumentänderungen | Frühzeitig sichern, soweit erlaubt |
| Hoch | Aktive Dokumente und benötigte Metadaten | Bevorzugt synchronisieren |
| Normal | Delta-Sync offline markierter Ordner | Gesteuerte Hintergrundarbeit |
| Niedrig | Grosse geplante Offline-Downloads | Drosselbar/pausierbar |
| Sehr niedrig | Reproduzierbare Cacheauffrischung | Zurückstellen bei knapper Bandbreite |

Grosse Transfers sind pausierbar und wiederaufnehmbar. Die Browser-Laufzeit kann Hintergrundarbeit beim Schließen des Browsers unterbrechen; ausstehende durable Aufträge und lokale Journalstände dürfen deshalb nicht auf eine stets aktive Client-Worker-Schleife angewiesen sein. `Tasks.cloudApp` und der Dateimanager unterscheiden **geplant, laufend, pausiert, gescheitert und serverseitig bestätigt**.

## 15. Dateitransfer, Resumable Upload und Delta-Synchronisierung

### 15.1 Robuste parallele Blockübertragung

Grosse Dateien werden für die Übertragung in **einzeln identifizierbare, integritätsprüfbare Blöcke** zerlegt. Der File Service verwaltet eine authentifizierte, zeitlich begrenzte Upload-Sitzung und quittiert nur tatsächlich persistierte, geprüfte Blöcke. Die Blockgröße und erlaubte Parallelität sind administrativ und technisch steuerbar.

Ein Upload-Vertrag benötigt fachlich mindestens:

| Feld / Eigenschaft | Semantik |
|---|---|
| `upload_id` | Opake, an Benutzer, App, Ziel und Gültigkeitsbereich gebundene Session-ID |
| Zielreferenz und `base_revision` | Autorisiertes Dateiziel samt erwarteter Ausgangsrevision |
| `total_size`, Blockschema | Erwartete Gesamtgröße, Reihenfolge/Offsets und Blockgrößen |
| Blockidentität und Hash | Integrität des einzelnen übertragenen Blocks |
| `accepted_blocks` | Vom Server dauerhaft bestätigte Abschnitte; niemals nur clientseitige Zähler |
| Ablaufzeit / Lease | Begrenzte Aufbewahrung unvollständiger Staging-Daten |
| `commit` / Completion | Finale Validierung, Dateimanifest, Quota, Revision und atomare Veröffentlichung |
| Fehler-/Resume-Status | Fehlende oder abgelehnte Blöcke ohne erneuten Komplettupload erkennen |

Blöcke dürfen parallel, begrenzt wiederholt und nach Unterbrechung gezielt nachgereicht werden. Die Gesamtintegrität wird anhand des finalen Manifests und einer kryptografischen Gesamtprüfung festgestellt. **Vor erfolgreicher Validierung darf keine unvollständige Datei als neue gültige Version sichtbar werden.**

### 15.2 Sicherheits- und Fehlerregeln für Transfers

- Vor Start, Blockannahme und Commit werden relevante Operationen, Dateirechte, App-Scopes, Quota und aktuelle Zielrevision geprüft.
- Fortsetzen eines Uploads ist kein Authentifizierungs-Bypass. Ein abgelaufener Lease oder widerrufene Berechtigung verhindert weitere Anhänge-/Commit-Vorgänge.
- Wiederholte Blockübermittlung ist idempotent; ein gleicher Block-Identifier mit abweichendem Inhalt wird abgewiesen.
- Staging-Blöcke werden getrennt von gültigen Dateimanifesten geführt und nach sicherer Frist/Policy bereinigt.
- Unvollständige, fehlerhafte oder quarantinierte Blöcke können nicht durch ungeprüfte Clientangaben als korrekt markiert werden.
- Bandbreiten-/Quota-Reservierung darf nicht auf Null gesetzt werden, nur weil ein Transfer unterbrochen wurde.
- Fortschrittsanzeigen beziehen sich auf **serverseitig bestätigte Daten**, nicht allein auf gesendete Bytes.

### 15.3 Adaptive Delta-Synchronisierung

Bei Änderungen größerer Dateien kann cloudOS unveränderte Datenbereiche identifizieren und vorhandene Blöcke wiederverwenden. Das Verfahren ist **adaptiv**: Je nach Dateiformat, Größe, Verschiebung von Daten, lokaler Rechenlast und erwarteter Einsparung werden feste oder inhaltsabhängige Blockgrenzen und andere geeignete Delta-Verfahren verwendet. Komprimierte oder verschlüsselte Dateien können sehr geringe Delta-Vorteile bieten.

Wenn der erforderliche Vergleich größere Kosten als die Einsparung verursacht oder der Zielinhalt nicht sicher wiederverwendet werden kann, erfolgt ein **Fallback auf normale, fortsetzbare Blockübertragung**. Wiederverwendung erfolgt nur innerhalb autorisierter Sicherheitsdomänen und unter erneuter Prüfung von Zielrevision, Blockintegrität und Datenhoheit. Eine Delta-Optimierung ist **keine automatische Konfliktzusammenführung beliebiger Binärdateien**.

## 16. Permanenter versionierter Blockspeicher

### 16.1 Datenmodell und atomare Versionen

Der File Service verwaltet Dateiinhalte dauerhaft in **unveränderlichen Datenblöcken**. Ein Dateimanifest referenziert Blöcke in ihrer logischen Reihenfolge und besitzt eine eindeutige Dateirevision. Unveränderte Blöcke können zwischen mehreren gültigen Dateiversionen geteilt werden. Metadaten, Manifeste, Blob-Speicher und Dateiversions-Commit müssen ohne sichtbare Teilzustande zusammenwirken.

Konzeptionelle Entitäten:

| Entität | Aufgabe |
|---|---|
| `FileNode` | Logische Datei-ID, sichtbare VFS-Metadaten, Owner, ACL, aktuelle Revision |
| `FileVersion` | Unveränderliche Version mit Zeit, Provenienz und Manifestreferenz |
| `BlockManifest` | Geordnete Blockreferenzen, Offsets, Größen, Integritätsinformationen |
| `EncryptedBlock` | Verschlüsselter, unveränderlicher physischer Datenblock innerhalb einer Domäne |
| `DedupDomain` | Isolierter Bereich für Blockwiederverwendung, Verschlüsselung und Quota |
| `UploadLease` | Noch nicht veröffentlichte Daten, Checkpoints und Zeitbudget |
| `ReferenceRoot` | Aktive Dateiversion, Papierkorb, Snapshot oder sonstige geschützte Wurzel |

Diese Entitätsbezeichnungen sind **konzeptionell**. Tabellen- und API-Namen sowie die physische Blockablage werden später im Speicher-Datenmodell definiert. PostgreSQL kann Metadaten und transaktionale Manifeste halten; der eigentliche Blockspeicher wird über einen eigenen, vom File Service kontrollierten Storage-Adapter angesprochen. App-Code erhält keine Rohreferenzen auf physische Blöcke.

### 16.2 Deduplizierung nur innerhalb isolierter Sicherheitsdomänen

Ein privater Benutzerbereich und jeder gemeinsame Space bilden **separate Deduplizierungsdomänen**. Identische Blöcke können zwischen Dateien oder Versionen **innerhalb derselben Domäne** wiederverwendet werden. Zwischen verschiedenen Benutzern oder voneinander isolierten Spaces erfolgt **keine automatische Deduplizierung**, selbst wenn der Klartext identisch ist.

- Interne Block-Hashes und Speicherreferenzen sind keine durch Apps frei aufrufbaren Erkennungsorakel.
- Eine Domänverschiebung (z. B. private Datei -> gemeinsamer Space) benötigt einen explizit autorisierten, atomaren Import/Transfer und gegebenenfalls neue Verschlüsselung, Metadaten und Referenzen.
- Datenwiederverwendung darf weder Eigentum noch ACL, Space-Mitgliedschaft, File-Handle-Bindung oder lösch-/aufbewahrungsbezogene Pflichten umgehen.
- Quoten müssen **logischen Dateiverbrauch und physisch belegte Blöcke** auseinanderhalten. Die Entscheidung über Quota-Anrechnung wird pro fachlichem Bereich spezifiziert, ohne sicherheitsbereichsübergreifende Effekte.
- Der Blockspeicher macht für normale Nutzer das VFS nicht anders sichtbar: das Home ist weiterhin `/` im `users`-Volume, das geschützte Systemvolume liegt auf `system:/`.

### 16.3 Referenzen und gleichzeitige Publikation

Ein Blöckereferenzzaehler kann die Verwaltung optimieren, ist aber **nicht alleiniger vertrauenswürdiger Löschnachweis**. Gültige Manifeste, aktuelle/alte Versionen, Papierkorb, Snapshots, Uploads und laufende Fachtransaktionen müssen in einer konsistenten Referenzsicht zusammengeführt werden. Ein Publish-/Delete-Race muss durch transaktionale Versionen, Leases/Fencing oder gleichwertige Sicherungen ausgeschlossen sein.

## 17. Verschlüsselung, Schlüsselrotation und Recovery

### 17.1 Envelope Encryption und Schlüsseltrennung

Alle schutzbedürftigen dauerhaft gespeicherten Datenblöcke werden **serverseitig authentifiziert verschlüsselt**. Die Verschlüsselungs- und Deduplizierungsdomänen werden logisch passend isoliert. Datenschlüssel beziehungsweise Domänenschlüssel werden durch versionierte übergeordnete Schlüssel geschützt (**Envelope Encryption**). Schlüssel und Schlüsselmetadaten stehen unter einer dedizierten, zentralen Verwaltungsgrenze mit fein begrenzten Service-Rechten.

| Ebene | Vertrag |
|---|---|
| Datenblock | Integritätsprüfbare Verschlüsselung mit passender Domänen-/Versionsbindung |
| Domän-/Datenschlüssel | Nur für erlaubte private Benutzer- oder Space-Domäne verwendbar |
| Übergeordneter Wrapping-Schlüssel | Kontrollierter Schutz und versionierte Rotation für Daten-/Domänenschlüssel |
| KMS-Port | Austauschbare, auditiert nutzbare Schnittstelle; spätere externe KMS/HSM-Integration |
| Berechtigter Datenzugriff | Der File Service entschlüsselt nur nach autorisierter Operation; App erhält keine Server-Schlüssel |

Die Deduplizierung muss mit nichtdeterministischer authentifizierter Verschlüsselung kompatibel geplant werden. **Identische Klartexte bedeuten nicht automatisch identische Ciphertexts.** Eine deduplizierte physische Blockreferenz kann innerhalb der Sicherheitsdomäne auf einen einmal verschlüsselten, unveränderlichen Block verweisen. Hash-/Vergleichsverfahren und kryptografische Modi werden vor Implementierung gesondert spezifiziert und sicherheitsgeprüft. Eine unkontrollierte konvergente, benutzerübergreifende Verschlüsselung ist ausgeschlossen.

Serverseitige Verschlüsselung ist **keine Ende-zu-Ende-Verschlüsselung**: Ein autorisierter File Service besitzt Zugriff auf die zur Verarbeitung benötigten Klartexte. E2EE wäre ein gesondertes fachliches Modell und wird nicht still als Eigenschaft behauptet.

### 17.2 Risikobasierte mehrstufige Rotation

- **Regelmäßig:** Übergeordnete Wrapping-Schlüssel werden nach definierter Policy erneuert und Daten-/Domänenschlüssel kontrolliert **neu verpackt** (Re-Wrapping).
- **Erhöhtes Risiko:** Betroffene Daten-, Domänen- oder Wrapping-Schlüssel werden je nach Bedrohungsanalyse ausgetauscht; die betroffenen Daten werden, falls erforderlich, mit frischen Datenschlüsseln **neu verschlüsselt**.
- **Rechteentzug:** Space-/VFS-/App-Zugriff wird sofort serverseitig entzogen, unabhängig davon, ob eine physische Neuverschlüsselung bereits läuft.
- **Hintergrundverarbeitung:** Umfangreiche Rotationen laufen als durable, checkpointfähige und auditierte Aufträge mit expliziten Erfolgs-/Fehlerstatus.
- **Kein falsches Sicherheitsversprechen:** Ein blosses Re-Wrapping kann einen bereits kopierten alten Datenschlüssel nicht entwerten. Eine vor dem Widerruf heruntergeladene Klartextkopie kann nicht nachträglich gelöscht werden.

### 17.3 Mehrstufiges Recovery und Notfallfreigabe

Schlüsselmaterial und Verschlüsselungsmetadaten werden **separat, verschlüsselt, versioniert und redundant** gesichert. Backups der Dateien/Manifeste und die dazu benötigten Schlüsselgenerationen müssen zusammen konsistent wiederherstellbar sein. Recovery-Geheimnisse liegen nicht ungeschützt am selben Ort wie die dazugehörigen Schlüsselbackups.

Kritische Wiederherstellungen erfordern eine **kontrollierte Mehrpersonenfreigabe**, dokumentierte Zustimmungs- und Rollenprüfung sowie verpflichtendes Security-Audit. Regelmäßige Restore-Tests erfolgen in einer isolierten Umgebung mit erneuter Verifikation der entschlüsselten Daten und Manifeste. Ein kompromittierter Schlüssel wird nicht unkritisch in eine produktive Umgebung zurückgespielt; notwendige Folgerotation wird eingeplant.

Sind alle benötigten Schlüssel und ihre Recovery-Möglichkeiten unwiederbringlich verloren, sind die verschlüsselten Daten **nicht rekonstruierbar**. Ein Teilrestore darf niemals als vollständig erfolgreich dargestellt werden.

## 18. Integritätsüberwachung und kontrollierte Selbstheilung

### 18.1 Prüfung beim Zugriff und im Hintergrund

Der File Service prüft Blöcke beim Schreiben/Lesen durch geeignete kryptografische Hashes und Authentifizierungsmerkmale. Zusätzlich führt er **priorisierte, lastgesteuerte Hintergrundprüfungen** durch (Scrubbing). Ergebnisse und Fehler sind revisions-/domänenbezogen nachvollziehbar; sensible Inhaltsdaten werden nicht in technischen Logs offengelegt.

Ein fehlerhafter Block wird sicher markiert und isoliert. Alle betroffenen Datei-Manifeste und Versionen werden ermittelt. Anschließend gilt:

1. Eine **unabhängige, kryptografisch validierte Kopie** ist vorhanden: kontrollierte Wiederherstellung aus dieser Kopie, atomare Korrektur des Referenz-/Storage-Zustands und erneute Integritätsprüfung.
2. Es ist **keine** intakte unabhängige Kopie vorhanden: kein Erraten, keine stillschweigende Korrektur, kein Freigeben beschädigter Nutzdaten; klare Fehlermeldung und Alarmierung berechtigter Administratoren.
3. Wiederholte Blockfehler oder ein Hinweis auf systematische Korruption werden als höherrangiger Storage-/Security-Incident eskaliert.

Eine Prüfsumme **erkennt** Datenveränderungen, stellt aber allein **keine** Daten wieder her. Eine vollständige redundante Storage-Architektur wird durch diese Entscheidung nicht automatisch vorgeschrieben. Backup-/Restore-Kopien und eventuell vorhandene Replikate müssen als tatsächlich brauchbare Reparaturquelle geprüft sein.

`Monitoring.cloudApp` präsentiert den Gesundheitszustand, `Status.cloudApp` die passenden rollenbasierten Auswirkungen und `Tasks.cloudApp` die kontrollierten Reparaturaufträge.

## 19. Transaktionssichere Garbage Collection und Papierkorb

### 19.1 Mehrstufiges Mark-and-Sweep-Verfahren

Die **Garbage Collection (GC)** ermittelt von vertrauenswürdigen Referenzwurzeln aus alle noch erreichbaren Datenblöcke. Als Roots gelten insbesondere aktive Dateimanifeste, geschützte ältere Versionen, aufbewahrte Papierkorbobjekte, Snapshots und laufende Transaktionen/gültige Upload-Leases. Ein Block wird nicht allein aufgrund eines kurzfristig auf null gefallenen Referenzzählers gelöscht.

Verbindlicher Ablauf:

1. **Konsistente Sicht / Generation festhalten:** Snapshot der Referenzwurzeln und wirksamen Retention-/Lease-Regeln ermitteln.
2. **Kandidaten markieren:** Nur nicht erreichbare Blöcke als vorläufige Löschkandidaten erfassen.
3. **Quarantäne/Sicherheitsfrist:** Kandidaten werden für eine definierte Frist vom Endlöschschritt getrennt aufbewahrt; aktive neue Uploads oder Wiederherstellungen können sie nicht unbemerkt reaktivieren.
4. **Erneute transaktionssichere Prüfung:** Vor dem physischen Delete Revision, neue Referenzen, Lease-/Fencing-Zustände, Aufbewahrungssperren und gegebenenfalls aktive Snapshots erneut kontrollieren.
5. **Endgültige Freigabe:** Nur bewiesenermassen unreferenzierte Blöcke entfernen, Quoten/Metriken konsistent fortschreiben und den Jobabschluss auditieren.

GC-Jobs sind **idempotent und wiederaufnehmbar**, können bei Last gedrosselt werden und können im verteilten Betrieb nicht unkoordiniert parallel denselben Speicherbereich bereinigen. Ein verifizierter Blockkorruptionsfall ist kein Nachweis, dass der Block verwaist ist. Externe Backups haben einen **eigenen** Speicher- und Retention-Lebenszyklus.

### 19.2 Papierkorb und endgültige Löschung

Eine normale Datei-/Ordner-Löschoperation verschiebt die Ressource in den zuständigen **konfigurierbaren Papierkorb**, sofern keine gesonderte autorisierte Regel etwas anderes bestimmt. Benutzer-/Space-bezogene Retention, Speicherlimits und Wiederherstellungsrechte werden geprüft. Der Papierkorb gehört zum selben Sicherheitsbereich wie die ursprüngliche Ressource.

| Operation | API- und Datenregel |
|---|---|
| In Papierkorb verschieben | Autorisierter Zustandswechsel mit Revision, Audit und SSE-Delta; Blöcke bleiben referenziert |
| Papierkorbinhalt auflisten | Nach Benutzer-/Space-/Rollenrechten gefiltert; keine Anzeige fremder Inhalte |
| Wiederherstellen | Auth-/ACL-/Pfad-/Namenskonflikte prüfen; keine stille Überschreibung neuerer Daten |
| Papierkorb leeren | Expliziter erlaubter Vorgang; Retention-/Schutzsperren beachten, GC anstossen |
| Frist abgelaufen | Automatische, kontrollierte Bereinigung gemäß Policy |
| Endgültig gelöscht | Aktiver Speicher bereinigt, **nicht** pauschal alle Backups und bereits exportierten Offline-Kopien |

Dauerhafte Sicherungen und Snapshot-Referenzen verhindern eine vorschnelle Blocklöschung. Löschaufträge können asynchron sein; der Fortschritt und die tatsächliche physische Bereinigung müssen voneinander unterscheidbar bleiben. Sensible endgültige Löschung ist mit dem Backup-/Restore-Konzept abzustimmen.

## 20. Verbindliche API-Qualitätsprüfung und Releasefreigabe

### 20.1 Quality Gate vor jeder Veröffentlichung

**Jede** API-relevante Freigabe muss die für ihren Umfang einschlägigen **verbindlichen Pflichtprüfungen** erfolgreich bestehen. Das Ergebnis wird versioniert dokumentiert; ein Fehler in einer sicherheitskritischen Pflichtprüfung **blockiert** das Release und darf nicht durch eine einfache manuelle Ausnahmegenehmigung aufgehoben werden.

| Prüfklasse | Verbindlicher Nachweis |
|---|---|
| Schema/Syntax | OpenAPI, Protobuf, AsyncAPI, CloudEvents-/SSE-Payloadschemas und SDK-Bindings sind gültig |
| API-Katalog | Alle OperationIds, Routen, Scopes, Service-Owner und Signaturdeklarationen sind konsistent |
| Vertragstests | Gültige/ungültige Daten, Fehlercodes, Content-Types, Paginierung, Revisions-/Idempotenzverhalten |
| Integration | REST -> Go-Port/gRPC -> DB/Event -> SSE und verteilte Backends funktionieren semantisch gleich |
| Authentifizierung / Autorisierung | Cookie-/CSRF-Schutz, mTLS/Delegation, App-/Ressourcenrechte, Consent, Widerruf, Isolation |
| Sandbox / Egress | Kein direkter unsicherer App-Core-/Network-Zugriff; Broker-/CSP-/Runtimegrenzen |
| Kompatibilität | Nur aktuelle Major, Deprecation-Checks, App-Manifest und Core-App-Baseline |
| Fehler-/Ausfallszenarien | Timeouts, Replay-Lücken, Retry, Idempotenz, Lease-Expiry, Job-/Saga-Recovery, GC-Sicherheit |
| Observability / Audit | Pflicht-Audit vorhanden; keine Geheimnisse in Logs/Traces; klare Monitoring-/Health-Zustände |
| Offline-/Sync-Sicherheit | Wiedereinspielen nach Logout/Widerruf verboten, Konflikte erkannt, lokale und serverseitige Revisionen getrennt |
| Datei- und Schlüssel-Integrität | Blockhash, Commit-Atomarität, Deduplizierungsgrenzen, Rotation, Recovery und GC-Failsafe |
| Dokumentation | Changelog, Migration, JavaScript-/TypeScript-/Rust-SDK-Hinweise und Impact-Bericht aktuell |

### 20.2 Mindest-Negativ- und Störfalltests

Die Testmatrix MUSS mindestens diese Klassen abdecken:

1. App verlangt unregistrierten Endpoint oder benutzt die richtige Route mit falscher HTTP-Methode.
2. App umgeht das SDK und benutzt eine fremde Sitzung beziehungsweise ein fremdes App-Instanztoken.
3. Nicht genehmigter neuer Endpoint wird durch Update oder Runtime-Code nachgereicht.
4. Eine einzelne Operation wird entzogen, während andere App-Funktionen weiterlaufen.
5. Geräte- oder Datei-Consent wird widerrufen; ein altes Handle bleibt nicht verwendbar.
6. Service-Delegation überschreitet ihren ursprünglichen Scope oder ist abgelaufen.
7. `If-Match` ist veraltet; ein paralleler Schreibzugriff geht nicht verloren.
8. Doppelte `Idempotency-Key`-/Job-/Event-Zustellung löst keine doppelte fachliche Wirkung aus.
9. JetStream-/DB-/Core-Restart zwischen Transaktion, Outbox und SSE-Replay erzeugt keinen stillen Eventverlust.
10. Replay nach Sitzungswiderruf, fehlender Subscription oder Space-Ausschluss liefert keine geschützten Daten.
11. Rate-Limits und Worker-Leases bleiben über mehrere Core-Replikate wirksam.
12. Partielle Dienststörung wird als `degraded` angezeigt, ohne einen ansonsten gesunden Pod in Endlosschleife neu zu starten.
13. Während Wartungsfenster werden Sicherheitsalarme weiterhin zugestellt und auditiert.
14. Browsercrash und Reconnect verlieren keine **bereits bestätigten** Checkpoints; nicht bestätigte Zustande bleiben als Risiko sichtbar.
15. Gleichzeitige Desktop-Änderungen auf zwei Geräten konvergieren unter den vorgesehenen CRDT-/OT-/Konfliktregeln.
16. Offline-Client versucht nach einem Widerruf, seine alte Änderungsjournal-Warteschlange hochzuladen.
17. Lokaler Cache überschreitet Quota; nicht persistierte Änderungen werden nicht als erfolgreich gesichert ausgegeben.
18. Upload bricht zwischen Blockannahme und Commit ab; Wiederaufnahme und erneute Rechteprüfung funktionieren.
19. Delta-Sync trifft auf neuen Zieldatei-Stand oder fremde Deduplizierungsdomäne und fällt sicher zurück.
20. Manipulierter Ciphertext, Hashkonflikt oder fehlender Wrapping-Schlüssel werden erkannt und nicht als intakte Datei ausgegeben.
21. GC läuft parallel zu Upload, Wiederherstellung, Versionsänderung und Snapshot-Erzeugung; kein noch benötigter Block wird gelöscht.
22. Papierkorb-/Restore-Vorgang respektiert ACL, Sperren, Namen-/Revisionskonflikte und Retention.
23. Test der Schlüsselwiederherstellung unter Mehrpersonenfreigabe einschließlich Audit und anschließender Datenvalidierung.
24. Breaking API-Änderung deaktiviert inkompatible Drittanbieter-App kontrolliert; inkompatible geschützte Core-App blockiert Release.

### 20.3 Freigabeentscheid und Nachweise

Der Releasebericht dokumentiert Contract-Hash/Version, Testnachweise, Sicherheitsprüfungen, sichtbare Breaking Changes, betroffene Core- und Drittanbieter-Apps, Freigabeverantwortliche, Migrationsschritte und erkannten Rest-Risiken. Der automatisch validierte Bericht wird erst **nach ausdrücklich erteilter kontrollierter Freigabe** Bestandteil eines Releases. Eine fehlgeschlagene Pflichtprüfung oder fehlende geschützte Core-App-Kompatibilität blockiert die Freigabe.

## 21. Entscheidungskatalog: Rückverfolgbarkeit aller 74 Antworten

Die folgende Tabelle bildet **jede der 74 ausdrücklich getroffenen Entscheidungen** ab. Ihr Geltungsbereich ist die verbindliche Fachkonzeption; Detailwerte und physische Implementierung bleiben dort offen, wo sie nicht Bestandteil der Auswahl waren.

| Nr. | Thema | Verbindliche Entscheidung | Ausführung |
|---:|---|---|---|
| 01 | Externe API | Zunächst nur autorisierte Apps und Core-Dienste; externe Clients erst später | 2, 3 |
| 02 | Major-Versionierung | Nur jeweils aktuelle API-Major, keine parallelen Alt-Major-APIs | 3 |
| 03 | Inkompatible Apps | Drittanbieter-Apps deaktivieren und installiert belassen; inkompatible Baseline blockiert Upgrade | 3, 20 |
| 04 | App-API-Zugriff | Ausschließlich über das offizielle cloudOS SDK | 8 |
| 05 | SDK-Sprachen | Erweiterbar; JavaScript, TypeScript und Rust zuerst, weitere Bindings später | 3, 8 |
| 06 | Sandbox | Isolierung aller Apps einschließlich Core-Apps | 2, 8 |
| 07 | App-Netzwerk | Externe App-Verbindungen nur über den Core Network Broker | 8 |
| 08 | Retries | Begrenzte automatische Retries, Writes nur idempotent geschützt | 5, 9 |
| 09 | Langlaufende Aktionen | Kurz synchron, lang als persistente Jobs mit SSE-Fortschritt | 9 |
| 10 | Job-Lifecycle | Durable oder session-bound je Core-Policy | 9 |
| 11 | Aufgabenverwaltung | Vorinstallierte `Tasks.cloudApp`, auch Taskleistenfortschritt | 9 |
| 12 | Gleichzeitige Writes | Optimistische Konflikterkennung per ETag/Revision | 5, 13 |
| 13 | Antwort-/Fehlerformat | Typisierte JSON-Antworten und RFC-9457-Problem-Details mit cloudOS-Codes | 5 |
| 14 | Interne Kommunikation | Go-Ports im Prozess, gRPC/Protobuf verteilt, JetStream asynchron | 2, 6, 7 |
| 15 | Service-Authentifizierung | mTLS-Workload-Identität plus kurzlebige Service-Capabilities | 6 |
| 16 | Service-Delegation | Signiert, kurzlebig, auf Benutzer/App/Operation eingeengt und widerrufsgeprüft | 6 |
| 17 | Nutzer-Consent | Explizite Zustimmung für sensible konkrete Ressourcen zusätzlich zur App-API-Freigabe | 8 |
| 18 | Consent-Laufzeit | Risiko-/ressourcenabhängig einmalig, Sitzung oder dauerhaft, zentral widerrufbar | 8 |
| 19 | Logging | Verpflichtendes Security-Audit, Diagnostik granular konfigurierbar | 11 |
| 20 | Log-Retention | Aufbewahrung, Quoten, Bereinigung und Archivierung je Logkategorie | 11 |
| 21 | Log-Rechte | Granularer Zugriff; Einsehen, Konfiguration und Audit getrennt, sensitive Einsichten auditiert | 11 |
| 22 | Listen/Streams | Cursor-Pagination, Streaming grosser Daten, SSE-Änderungen ohne Polling | 5, 7, 10 |
| 23 | Rate-Limits | Dynamische hierarchische Limits und replikenübergreifende Fairness | 10 |
| 24 | Teilstörungen | Isolierte Auswirkung, Degradierung und Failover gesunder Replikate | 2, 6, 12 |
| 25 | Job-Recovery | Automatische Wiederaufnahme am geprüften Checkpoint mit Leases und Idempotenz | 9 |
| 26 | Event-Zustellung | At least once, idempotente Effekte, pro Ressource geordnet, Outbox und SSE-Replay | 7 |
| 27 | Ereignisformat | CloudEvents 1.0 mit versionierten Schemas und sicherheitsgefilterten Deltas | 7 |
| 28 | API-Katalog | Serviceeigene Verträge in automatisiert geprüftem zentralen Katalog | 4 |
| 29 | Contract-Änderungen | Automatische Kompatibilitätsprüfung und kontrollierte Releasefreigabe | 3, 4, 20 |
| 30 | Entwickerdokumentation | Statisches Markdown; kein initiales interaktives API-Portal | 3, 4 |
| 31 | IDEs | Externe IDEs/Toolchains; mögliche `Developer.cloudApp` später | 3 |
| 32 | App-Kompatibilität | Signierte Deklaration von Core-API-Major, SDK-Capabilities und Mindestversionen | 3 |
| 33 | Deprecation | Geplant und mit Ersatz; kritische Sicherheitsausnahme für sofortige Entfernung | 3 |
| 34 | Migration | Generiertes Changelog/Markdown-Migration für JS, TS und Rust | 3 |
| 35 | SDK-Verteilung | Offizielle versionierte npm-Pakete und Rust-Crate | 3 |
| 36 | SDK-Capability-Filter | Dokumentiertes Voll-SDK, laufende App sieht nur eigene effektive Capabilities | 3, 8 |
| 37 | Neue API-Rechte | Nur über signiertes App-Update und zwingende Administratorfreigabe | 8 |
| 38 | Einzelwiderruf | Nur betroffene Funktion entziehen; laufende App bleibt nach Möglichkeit offen | 8 |
| 39 | Effektive Rechte | SDK liefert eigene effektive Rechte und SSE-Änderungen; Server bleibt maßgeblich | 8 |
| 40 | Dienstübergreifende Transaktionen | Persistente Sagas mit lokalen ACID-Schritten und Kompensationen | 9 |
| 41 | Saga-Fehler | Begrenzte Retries, Kompensation, sicherer Fehlerzustand und auditierter Admin-Eingriff | 9 |
| 42 | Distributed Tracing | OpenTelemetry, W3C Trace Context, Sampling/Retention, Ansicht in `Logs.cloudApp` | 11 |
| 43 | Monitoring | Umfassende OTel-Metriken, Historie, SLOs, Alarme und Autoscaling-Grundlage | 12 |
| 44 | Selbstheilung | Abgestufte automatische sichere Reparaturen, Grenzen und Admin-Freigabe für Risiken | 12 |
| 45 | Alarmierung | Konfigurierbare SSE-/E-Mail-/Webhook-Kanäle, Schweregrade und Eskalation | 12 |
| 46 | Wartung | Geplante scoped Wartungsfenster mit intelligenter Alarmsteuerung | 12 |
| 47 | Health | Startup, Liveness, Readiness und fachlicher Funktionsstatus getrennt | 12 |
| 48 | Systemstatus | Vorinstallierte `Status.cloudApp` mit rollenbasierten Ansichten | 12 |
| 49 | Core-Ausfall und Sitzung | Sitzung/Fenster erhalten, nicht betroffene Apps arbeiten weiter | 13 |
| 50 | Crash-Recovery | Vollständige unterstützte Workspace-/App-Zustandswiederherstellung mit Auto-Save | 13 |
| 51 | Auto-Save-Zeitpunkte | Hybrid: Ereignisse, kurze Ruhephase, Sicherheits-Checkpoints und Laststeuerung | 13 |
| 52 | Auto-Save-Speicherung | Dauerhafte Server-Checkpoints plus geschützter lokaler Cache | 13, 14 |
| 53 | Historie | Konfigurierbare Wiederherstellungshistorie mit Benutzer-/App-/Gruppenregeln | 13 |
| 54 | Mehrgeräte-Sitzung | Vollständig synchronisierte logische Desktop-Umgebung | 13 |
| 55 | Session-/Gerätemanagement | Übersicht, Einzel-/Gesamtwiderruf, neue Anmeldungen, Admin-Policies | 13 |
| 56 | Gleichzeitige Änderungen | Echtzeit-Zusammenführung mit CRDT/OT für geeignete Daten | 13 |
| 57 | Reconnect nach Offlinephase | Offline-first-Journal und automatische berechtigungsgeprüfte Zusammenführung | 14 |
| 58 | Offline-Funktionsumfang | Abgestufte Offline-Fähigkeit für Core und Apps | 14 |
| 59 | Lokale Sicherheit | Verschlüsselung, benutzer-/app-/gerätegebundene Offline-Freigabe | 14 |
| 60 | Offline-Quoten | Hierarchische Speichergrenzen und automatische sichere Cachebereinigung | 14 |
| 61 | Offline-Auswahl | Selektive Datei-/Ordnersynchronisierung pro Gerät | 14 |
| 62 | Transferplanung | Intelligente adaptive Priorisierung und Bandbreitensteuerung | 14, 15 |
| 63 | Grosse Dateiübertragung | Parallele, fortsetzbare Blocktransfers mit Hash und atomarem Commit | 15 |
| 64 | Delta-Sync | Adaptive inhaltsbewusste Blockerkennung mit sicherem Fallback | 15 |
| 65 | Blockspeicher | Permanenter versionierter Speicher mit unveränderlichen Datenblöcken | 16 |
| 66 | Deduplizierung | Nur innerhalb definierter privater Benutzer-/Space-Sicherheitsdomänen | 16 |
| 67 | At-rest-Kryptografie | Mehrstufige Envelope Encryption und zentrale Schlüsselverwaltung | 17 |
| 68 | Rotation | Risikobasiert, Re-Wrapping regulär, Neuverschlüsselung bei Bedarf | 17 |
| 69 | Key Recovery | Redundante getrennte Backups, Mehrpersonen-Notfallfreigabe und Restore-Tests | 17 |
| 70 | Blockintegrität | Kontinuierliche Prüfung und Selbstheilung nur aus vertrauenswürdiger intakter Kopie | 18 |
| 71 | Garbage Collection | Mehrstufig, transaktionssicher, Sicherheitsfrist und erneute Referenzprüfung | 19 |
| 72 | Papierkorb | Konfigurierbar mit kontrollierter endgültiger Löschung | 19 |
| 73 | Formale API-Verträge | OpenAPI, Protobuf, AsyncAPI/Ereignisschemas und generiertes Markdown | 4 |
| 74 | Release-Qualität | Vollständige verbindliche Vertrags-, Sicherheits-, Integrations- und Fehlertests mit Releaseblockade | 20 |

## 22. Abgrenzungen, Folgedokumente und konkrete Nacharbeiten

### 22.1 Inhaltliche Führung und Dokumentabgleich

| Fachbereich | Führendes Detaildokument | Hier verbindlicher Teil |
|---|---|---|
| Go-Core-Module, Deployment, Kubernetes | `SYSTEM_ARCHITECTURE.md`, `CORE_DEPLOYMENT_SCALING.md` | Semantische API-Gleichheit, Health-/Monitoring-Verhalten, Service-Auth |
| `.cloudApp`-Paket und Signing | `APPLICATION_SYSTEM.md`, `APP_SIGNING.md` | SDK-Zugriff, Versionen, Genehmigung und Widerruf |
| API-Allowlist und Installationsdialog | `APP_API_ACCESS.md` | OperationId-/Scope-Modell, Default Deny, keine Runtime-Rechteausweitung |
| Browser-Push | `PUSH_REALTIME.md` | SSE, Audience, Replay, Resync, kein Polling |
| Login, Sitzungen, Geräte | `AUTHENTICATION_AUTHORIZATION.md`, `AUTHENTICATION_METHODS.md`, `CLIENT_DEVICES.md` | Delegation, Consent, Widerruf und Offline-Grenzen |
| Logs und Traces | `LOGGING_OBSERVABILITY.md` | Berechtigungs- und Korrelationsverträge |
| VFS, Speicher, Spaces | `VIRTUAL_FILESYSTEM.md`, `SHARING_SPACES.md` | Upload-/Delta-/Block-/Delete-Semantik und Sicherheitsdomänen |
| Desktop, Window Host, Themes | `DESKTOP_ENVIRONMENT.md`, `WINDOW_FRAMEWORK.md`, `THEMING.md` | Zustand, Realtime-/Offline-Synchronisierung, Wiederherstellung |
| Spätere detaillierte Datenmodell- und Backup-Regeln | `STORAGE_DATA_MODEL.md`, `BACKUP_RESTORE.md` (geplant) | Verträgliche Schnittstelleninvarianten und Key-Recovery-Grenzen |
| SDK, Betrieb, Einstellungen | `APP_FRAMEWORK_SDK.md`, `SETTINGS_SYSTEM.md` (geplant) | SDK-/Consent-/Rate-/Offline-Funktionsverträge |
| Architekturentscheidungen | `ADR_REGISTER.md` (geplant) | Begründung, Alternativen und Abnahme der einzelnen ADRs |

### 22.2 Vor Implementierung zu präzisierende Parameter

Die folgenden Punkte sind **keine neuen Produktfragen und keine Widerrufe der 74 Entscheidungen**. Sie sind innerhalb des gewählten Modells noch konkret zu spezifizieren:

1. Endgültiger kanonischer API-OperationId-/SSE-Eventkatalog und genaue OpenAPI-/AsyncAPI-/Proto-Pfade sowie Namensräume.
2. Versionierung einzelner SDK-Capabilities und die konkreten Kompatibilitätsfenster **innerhalb** der jeweils aktiven Major-Version.
3. TTLs für Delegationen, `Idempotency-Key`, Replay, durable Jobs, lokale Offline-Freigaben, temporäre Uploads und Garbage-Collection-Quarantäne.
4. Retry-Budgets, Deadlines, Backpressure-Werte, Rate-Limits, App-/Benutzer-/Systemquoten und Wartungsfenster-Policies.
5. Abnahmefähige CRDT-/OT-Datenmodelle pro App/Datentyp, Konfliktprotokolle und irreversible Operationen.
6. Bei Browser- und Gerätekompatibilität tatsächlich erreichbare verschlüsselte Offline-Schlüsselhaltung, Entsperrmethode und defensiver Fallback.
7. Blockgrößen, inhaltsabhängige Delta-Strategie, Manifestformat, Quota-Verrechnung und effiziente, sichere Domänen-Deduplizierung.
8. Kryptografische Algorithmen, Nonce-/Key-Derivation-Regeln, Schlüsselrotation, Recovery-Quorum und Key-Backup-Medien.
9. Alarm-Eskalationszeiten, priorisierte SLOs, Health-Thresholds und Policy für automatische Reparaturen.
10. Retention der Arbeitsstände, Papierkorbinhalte, Backups und rechtliche/politische Aufbewahrungssperren.
11. Welche Core-App-Baseline `Monitoring.cloudApp`, `Tasks.cloudApp`, `Status.cloudApp` und `Logs.cloudApp` zu welchem Release zwingend enthält.
12. Exakte Reihenfolge und Voraussetzungen für ein späteres externes API-Client-Profil sowie die dann erforderlichen zusätzlichen Sicherheitsverträge.

### 22.3 Konsistenz- und Nichtziele

- Dieses Dokument **implementiert keine API**, keinen Speicheradapter, kein SDK und keine Runtime; auch die beispielhaften JSON-Verträge erheben keinen Produktionsstatus.
- Das einzige Standard-Core-Deployment bleibt der gemeinsame Go-Prozess im Core-Container. Eine optionale Kubernetes-Skalierung wird nicht versehentlich zur Mindestvoraussetzung.
- Es werden **keine neuen virtuellen Volumes** eingeführt. Sichtbare Benutzerdaten liegen weiterhin auf `users` mit dem Home-Alias `/`; System-/App-Pakete verbleiben auf `system` bzw. `system:/apps`.
- Ein App-Consent für eine Datei ist **keine** Genehmigung neuer Core-API-Operationen. Neue Operationen benötigen weiterhin ein signiertes Update plus Administratorfreigabe.
- SSE-Push, ein automatisch ablaufendes Auto-Save oder die serverseitige Wiederaufnahme eines Jobs sind **keine** periodischen Browser-Polling-Loops.
- Deduplizierung, Blockverschlüsselung und Offline-Recovery dürfen nicht als Beleg für echte Ende-zu-Ende-Verschlüsselung, sofortige Remote-Löschung oder vollständige Datenrekonstruktion nach Schlüsselverlust ausgegeben werden.
- Unabhängige Release-Gates müssen bei Sicherheitsfehlern blockieren, selbst wenn ein menschlicher Freigabeprozess ansonsten Zustimmung erteilen würde.

## 23. Definition of Done für das Vertragskonzept

Das vorliegende **Konzeptdokument** gilt als konsolidiert, wenn alle folgenden Punkte erfüllt sind:

- [x] Alle 74 abgestimmten Entscheidungen sind in einer eindeutigen Nummernmatrix erfasst.
- [x] Sicherheits-, Deployment-, App-SDK- und SSE-Grenzen sind mit den vorhandenen Fachkonzepten abgeglichen.
- [x] REST, gRPC, Events und SSE haben definierte Standardformate und zentrale Vertrags-Eigentümer.
- [x] Job-, Saga-, Fehler-, Retry-, Versions-, Audit- und Releaseverhalten ist fachlich festgelegt.
- [x] Sitzungs-/Mehrgeräte-/Offline-/Dateisynchronisierungsentscheidungen sind konsistent beschrieben.
- [x] Blockspeicher, Deduplizierung, Verschlüsselung, Rotation, Recovery, Scrubbing, GC und Papierkorb sind API-seitig abgegrenzt.
- [x] Nicht beschlossene Parameter und noch nicht implementierte Schnittstellen sind ausdrücklich als solche bezeichnet.

**Nicht damit behauptet:** Implementierungs-, CI-, Sicherheitstest-, Integrations- oder Produktionsreife. Für diese Stufen sind die in Kapitel 20 definierten verbindlichen Prüfungen erst tatsächlich auszuführen und ihre Ergebnisse nachzuweisen.

---

**Ende des API-Vertragskonzepts – 74/74 Entscheidungen erfasst.**

## 23. Verbindliche Infrastruktur- und Event-Vertraege (Entscheidungen 15.11.2026)

Fachlich fuehrend fuer Betriebsprofile ist [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md). Die nachfolgenden Vertraege ergaenzen die bestehenden REST-, gRPC-, CloudEvents-, SSE- und Saga-Vorgaben und sind **konzeptionell beschlossen, nicht implementiert**.

### 23.1 Ereignis- und Consumer-Vertraege

- Ein Core-Owner erzeugt seine Mutation und einen eindeutig identifizierten Outbox-Eintrag in derselben lokalen PostgreSQL-Transaktion. Ein fremder Owner liest oder manipuliert dessen Tabellen nicht direkt.
- Der Publisher uebernimmt Outbox-Eintraege mit atomarem Claim und begrenzter Lease. Die Uebertragung gilt erst nach gueltiger JetStream-Persistenzbestaetigung als abgeschlossen. Unklarer Ack nach Crash erlaubt dieselbe Event-ID erneut zu publizieren.
- Jeder Eventtyp besitzt fachlichen Owner, stabile `event_id`, CloudEvents-Schema und Versionsinformation. Stream- und Subject-Zuordnung werden zentral validiert, fachlich getrennt und mit Zugriffspolitiken versehen.
- Durable Consumer verfolgen unabhaengige fachliche Positionen; Replikate desselben logischen Consumers koordinieren eine gemeinsame Workqueue. Doppelte Lieferung ist normal und darf keine zweite wirksame Mutation ausloesen.
- Consumer-Acks erfolgen erst nach abgeschlossener oder nachweislich dauerhaft uebernommener Arbeit. Wiederholungsbudget, Quarantaene, Sichtbarkeit des Fehlers und Reprocessing-Regeln sind verbindlich.
- Die Outbox-Retention ist nicht an die Ack-Position jedes einzelnen Consumers gekoppelt. Nach bestaetigter Uebertragung bleiben Eintraege fuer eine eventklassenspezifische Sicherheits-/Recovery-Frist erhalten.
- Retention-Luecken werden explizit erkannt. Rekonstruierbare Projektionen koennen aus einem konsistenten Owner-Snapshot samt revisionssicherem Uebergang neu aufgebaut werden; nicht sicher rekonstruierbare Seiteneffekte werden blockiert statt blind wiederholt.

### 23.2 Infrastruktur-Failover und konsistente Autorisierung

PostgreSQL-HA und JetStream-HA werden extern beziehungsweise infrastrukturseitig verwaltet. cloudOS prueft eindeutige schreibende Primaritaet, durable Zustellbestaetigungen, sichere Wiederverbindungen und einen unklaren Commit-Status. Sicherheitskritische Rechtepruefungen verwenden keinen unzureichend aktuellen Replika-Zustand. Ein lokaler HA-Failover ist kein DR-Failback.

### 23.3 Core-Release und Kommunikation

Produktive Core-Updates erfolgen nur manuell autorisiert unter vollstaendigem Wartungsstopp; eine neue Readiness allein aktiviert nicht die öffentliche API. Kein Canary-, Rolling- oder automatischer Release-Rollback als Standard. Bei normalem Replika-Ausfall koennen SSE-Streams mit geschuetztem Cursor und Replay wiederhergestellt werden. Interne Remote-Ports erfordern mTLS und Workload-gebundene Service-Capabilities; Compose combined bleibt der Standard.

### 23.4 Offene Zahlenwerte

Ack-Fristen, Retention-Dauer, Idempotenzfenster, Retry-Budgets, Poollimits, RTO/RPO, p95-/p99-Ziele und die konkreten Protokoll-/Kryptografieprofile werden erst im jeweiligen Implementierungs-/Abnahmedokument nach Messung bzw. Sicherheitsbewertung festgelegt. Diese Detailwerte sind nicht durch pauschale Beispiele als produktive Defaults genehmigt.
