# Interface Contract Catalog – App Framework SDK

> **Status:** normative Semantik und Vertragsentwuerfe; **keine** Behauptung bereits implementierter API-Routen.  
> **Massgeblich fuer existierende API-Policies:** [API_CONTRACTS.md](../API_CONTRACTS.md), [APP_API_ACCESS.md](../APP_API_ACCESS.md).

## 1. Gemeinsame API-Invarianten

- Apps verwenden freigegebene SDK-Operationen. Die signierte `permissions/api-access.json`, wirksame App-Zuweisung, Endpoint-Grant, Benutzerrecht und Resource Scope werden serverseitig geprueft.
- Vertraglich zulaessige Fehlerschemata sind versioniert; `CANCELLED`, `DEADLINE_EXCEEDED`, `OUTCOME_UNKNOWN` muessen eindeutig abbildbar sein.
- Operationen unterscheiden read-only, lokal transaktionale Mutationen, asynchrone Jobs und externe Seiteneffekte.
- Korrelations-ID und Operations-ID erlauben Traceability, sind aber **keine** Sicherheitsnachweise.
- Zeitstempel sind UTC; Anzeige und Kalenderregeln verwenden IANA-Zeitzonen.
- Eine gleichbleibende API-Semantik gilt unabhaengig von `combined`, `split` oder optionaler Kubernetes-Verteilung.
- Vor jeder finalen Endpoint-Definition bestehende REST/OpenAPI-/gRPC-/SSE-Vertraege abgleichen; keine zusaetzlichen Routen hier erfunden.

## 2. Vertragstabelle

| Vertragsfamilie | Eingang | Ergebnis | Autorisierungs-/Konsistenzgrenze |
|---|---|---|---|
| App Bootstrap / Lifecycle | App-ID, signierte Release-ID, Instanzkontext | Aktivierung/Readiness | Registry, Zuweisung und Capability-Gateway |
| Resource Capabilities | Ressource, Operation, gueltiger Kontext | Erlaubt/Abgelehnt | Serverseitiger Owner + Widerruf |
| App Data Query | Schema, Filter, Umfang, Revision | Autorisierte Daten/Status | Owner-Filter, Limits, versionierte Antwort |
| App Data Mutation | Domain Operation, Payload, erwartete Revision | Commit/Conflict/Unknown | Constraint- und Transaktionsgrenze |
| VFS / File Jobs | Handle, Operation, Revision | Ergebnis oder Job-ID | VFS-Owner, aktuelle Benutzerrechte |
| Resource Watch / Push | Subscriptions und Revisionscursor | SSE-Deltas/Replay-Gap | Signiertes Event-Grant und Empfaengerfilter |
| Messaging | Versioniertes Ereignis und Consumer-Kontext | Persistente Annahme/Fehler | At-least-once, TTL, DLQ |
| Worker / Job | Jobdefinition, Execution Identity, Limits | Jobstatus | Delegierte Rechte, Lifecycle, Quotas |
| Saga / Approval | Versionierte Schritte, Ressourcenrevision | Workflow-/Decision-State | Autorisierte Domain Steps und Step-Up |
| External Account | Provider, Scopes, Benutzerinteraktion | Verbindungsreferenz | OAuth-State/PKCE, Secret Broker, App-Grants |
| External Subscription | Provider-Verbindung, Eventtyp | Subscription-/Renewal-State | Webhook Ingress, Provider-Verifikation |
| Browser Services | Fenster, Dialog, Clipboard/Devices, Push | UI-/Permission-Ergebnis | Origin, native Browser-Permission und Core |
| Diagnostics | Pruefkennung, Scope | PASS/WARN/FAIL/SKIPPED/UNAVAILABLE | Admin-/App-Rechte, Redaction |

## 3. Security Context Contract

Konzeptionelle Attribute: `tenant_context`, `user_session`, `app_installation`, `app_instance`, `release_identity`, `operation_id`, `correlation_id`, `deadline`, `cancellation`, `resource_scope`. **Dies sind semantische Felder, keine festgelegten JSON- oder TypeScript-Namen.** Sicherheitskritische Attribute werden vom Core abgeleitet und niemals blind aus App-Payloads vertraut. Ein Kindaufruf kann hoechstens den benoetigten Teil des berechtigten Scopes erhalten.

## 4. Idempotenz und Commit-Zustand

Eine erneute Mutation mit demselben autorisierten Idempotenzschluessel darf nicht unbeabsichtigt eine zweite fachliche Wirkung ausloesen. Bei verlorener Antwort darf der Client einen unverifizierten Erfolg nicht behaupten. `OUTCOME_UNKNOWN` verlangt Statusabfrage oder sichere Wiederholung. Cancellation ist keine Entschaedigung oder Transaktionsruecknahme.

## 5. Versionierung

API-Major und Minor, App-Release, Dokumentation, Nachrichtenschema und Datenmodell sind **verschiedene Versionsachsen**. Bestehender `API_CONTRACTS.md`-Grundsatz: **nur die jeweils aktive API-Major**, nicht pauschal mehrere API-Majors parallel. Q151 erlaubt zeitlich begrenzte **App-Release-Kompatibilitaet**, nicht automatisch parallele API-Major-Versionen. Der konkrete Adapter-/Uebergangsvertrag ist als Risiko R-001 vermerkt.

## 6. Browsertransport und Nachrichten

Der regulaere clientseitige Server-Push-Kanal ist gemaess [PUSH_REALTIME.md](../PUSH_REALTIME.md) **SSE**. Web Push (Q139) ist ein optionaler browsernativer Zustellkanal. Q035 adressiert das Lifecycle-Management externer SSE-/WebSocket-Verbindungen, **nicht** die Aufhebung der Core-SSE-Festlegung. Externe Webhooks passieren den zentralen Ingress (Q103/Q144).

## 7. Spezifikationsluecken vor Implementierung

Noch zu erstellen bzw. gegen Originalvertraege zu pruefen: ID-Namensraum, Fehlerkatalog, Feldtypen, Pagination, Revisions-/ETag-Rechte, schema versions, Protokoll-Traces, OpenAPI-Definitionen, Interaktionssequenzen, konkretes Auth-/Grant-Format, Testvektoren und Limits. Eine solche Luecke ist keine implizite Freigabe zur Implementierung nach eigenem Ermessen.

## 8. Zusätzliche verbindliche Contract-Invarianten K1–K9

| Vertrag | Normative Pflicht | Nicht erlaubt | Beschluss |
|---|---|---|---|
| Install/Activation | Signatur, Integrität, Conformance, Endpoint-/Device-Policy, Self-signed-Risikoannahme unabhängig | Admin-Override harter Prüffehler | K1 |
| Runtime Selection | Explizites signiertes Runtime-Profil und Capability-Gate; isolierte WASI-/Native-Bindings | Beliebiger nativer Code / Hostzugriff | K2 |
| Browser Bridge | App-eigene Origin, geprüfter Instanzkanal, sichere Sandbox/SW-Policy | Gemeinsame Privileg-Cookies / App-gesteuerter SW | K3 |
| Compatibility | Eine aktive Core-API-Major; kompatible Parallel-App-Releases | Parallele Alt-Major-APIs | K4 |
| Realtime/Egress | Core SSE über Host; externe SSE/WS über Broker; Web Push über Notification | Eigenes App-Core-EventSource/WS-Bypass | K5 |
| Data Profile | `server_managed` versus echtes `e2ee_client_keys`, mit Capability-Fallback | Server-Klartextvorschau bei E2EE | K6 |
| Replay | Audience/Policy-gebundener Cursor, neue AuthZ-Prüfung, expliziter Resync | Alt-Payload nach Grant-Entzug | K7 |
| Saga Recovery | Neue minimale Recovery-Autorisierung und idempotente Kompensationen | Weiterbenutzung alter widerrufener Rechte | K8 |
| Release Kill Switch | Digest-/App-/Trust-/Runtime-Sperre vor Start, Bridge und Rollback | Reaktivierung gesperrter Digests | K9 |

Die Vertragstabelle beschreibt **Semantik**, nicht existierende REST-Endpunkte. Konkrete DTOs, Feature Negotiation, Policy-Revision, Error-Typen, AuthZ-Checkpoints und Zustands-Enums müssen aus diesen Invarianten abgeleitet und mit `API_CONTRACTS.md` abgestimmt werden. Vollständiger Beschlussnachweis: [K1–K9](11-architecture-decisions-K1-K9.md).
