# R-008 – Zuordnung des SDK-Komponentenkatalogs zum Go-Core

> **Stand:** 16.11.2026 · **Status:** Fachlicher Architekturvertrag, keine implementierte Owner-Registry.  
> **Quellen:** [SDK-Komponenten](03-component-catalog.md), [Core-Servicekatalog](../SYSTEM_ARCHITECTURE.md), [technische Rollen](../CORE_DEPLOYMENT_SCALING.md), [API-Verträge](../API_CONTRACTS.md).  
> **Beschluss:** R-008 Option C; eigenständiger **logischer** App-Data-Daten-Owner im vorhandenen Combined-Go-Core, **kein zusätzlicher technischer HTTP-Dienst/Container**.

## Abgrenzung

`Core-Service` bezeichnet hier eine **fachliche Owner-Grenze**, `core/internal/platform/services.json` einen **technischen Bootstrap-Katalog**. Sie müssen nicht 1:1 übereinstimmen. Browser-Dienste, CI-Konformitätstests und spätere Integrationsbroker sind keine zusätzlichen Bootstrap-Rollen nur aufgrund des SDK-Katalogs. Die bisherigen 22 technischen Rollen bleiben bestehen; diese Dokumentation ordnet Funktionen zu, sie erweitert die Liste nicht.

## Zuordnungstabelle

| SDK-Komponente aus `03-component-catalog.md` | Bestehender zuständiger Core-Kontext / Modul | Fachlicher Zustand und Grenze | Umsetzungsgate |
|---|---|---|---|
| Identity & Session | `identity`, `sessions` | Lokale Benutzer, Credential-Verknüpfung, Sessionstatus und Revoke-Revisionen besitzen getrennte Owner | Auth-Zustands-/Session-Schemas |
| Policy & Capability Gateway | `permissions` und `gateway`; Browser-Bridge bleibt Trusted Host | `permissions` entscheidet autoritativ, Gateway erzwingt am API-Rand, Browser vermittelt lediglich App-Kontext | AuthZ-Prüfpunkte je Operation |
| App Registry & Release | `apps`, `packages`, `appstore` | App-ID/Installation, unveränderlicher Paketdigest, Store-Provenienz sind unterschiedliche Zustandsobjekte | Unique-Constraints, Release-Revocation |
| App Instance & Window | `apps` (Instanz/Lifecycle) und `desktop` (persistenter Fensterzustand); Trusted Browser Window Host | Instanz- und Fensterzustand werden nicht zu einem neuen privilegierten Service verschmolzen | Instanz-/Fenster-Revisionsvertrag |
| Browser Background & Tab Coordination | Browser Host/Service-Worker, `sessions` und `permissions` | Clienttab-Leases sind kein Autoritätsersatz für Core-Session-/Grant-Revisionen | Cross-Origin/Tab-Isolation |
| **App Data Service** | **Eigenständiger logischer App-Data-Owner im Go-Core, keine neue Bootstrap-Rolle** | Typisierte app-/tenantbezogene Domain-Daten, Schemas, Versionen, Revisionen, Transaktionen; nicht `apps`-Registry und nicht `files`-Blob/VFS-Owner | Endgültiger DB-/API-Vertrag **noch offen** |
| VFS / Files | `files`, interne Storage-Adapter | File-Nodes, Blob-Referenzen, Quotas, ACL-Mounts, File-Revisionen | Crash-/Restore-/E2EE-Verträge |
| Query / Index / Resource Watch | `search` (spätere generelle Indexe); fachliche Queries beim jeweiligen Owner; `realtime` für Deltas | Keine eigenmächtigen quer über Owner-Schemas schreibenden Abfrage-/Indexdienste | Indexdatenklassifikation und Zugriffstests |
| Messaging | Eigentümer-eigene DB-Outbox, NATS als Infrastruktur, `push-projector`, `realtime` | Fachzustand besitzt Owner; Brokertransport und SSE-Lieferlog haben getrennte Consumer-/Delivery-Rollen | Transactional Outbox, Replay und Purge |
| Job / Saga / Scheduling | logischer Job-/Workflow-Owner, `runtime-controller` für spätere Ausführung | Job- und Checkpointzustand nicht mit Runner-Arbeitsspeicher gleichsetzen; keine zweite neue Scheduler-Rolle ohne ADR | Job-Datenmodell und Delegation |
| Worker Runtime | `runtime-controller` (später), Browser-Worker im jeweiligen untrusted App-Kontext | Server-WASI nach MVP, Native optional später, niemals in privilegiertem Core-Prozess | Sandbox-/Runtime-Abnahme |
| External Accounts / Secrets | optionaler Integration-/Secret-Broker im Core; externe Identity bei `identity` | Opaque Handles/Secret-Grants, keine Browser-Langzeitsecrets; konkrete Secret-Persistenz noch zu spezifizieren | Credential-/Rotationsvertrag |
| Network Gateway / Webhooks | kontrollierter Integration/Egress Broker und `gateway` für Ingress | Externer SSE/WS/Webhook-Verkehr ist nicht Core-SSE und keine neue ungeschützte Proxy-API | SSRF-/Policy-/Consent-Tests |
| Trusted UI Services | Trusted Desktop/App Host; `notifications`, `settings` | Dialoge/Navigation/i18n/Accessibility sind keine neue Datenbankautorität; Notification Inbox hat einen eigenen Owner | Vertrauenswürdige UI, i18n, A11y |
| Privacy / Consent / Audit | `permissions` für Consent-/Grant-Enforcement, `audit` für Security-Nachweise, fachliche Data-Owner für Inhalte | Versionierter Zweck/Consent ändert weder File-/App-Ownership noch nachträglich Providerzustand | Consent-Datenmodell/Entzugsprüfungen |
| Supervisor / Diagnostics | `logging` getrennt von `audit`, jeweilige Health-Owner | Abschalten von Diagnose-Logs darf Audit nicht unterdrücken | Redaction und Ausfalltests |
| Resource Accounting | vorhandene Besitzer-Kontexte für Quotas; Policy-Budget im Core | Keine zweite unabhängige Quota-Wahrheit; konkrete Kostenklassifizierung pro Owner | Quota-/Fairness-Tests |
| SDK Documentation / Conformance | Entwickler-/CI-Vertrag und `packages`/`permissions` beim Release-Gate | Keine zusätzliche Laufzeitrolle für Dokumentationsgenerator oder Testmatrix | echte cloudOS-Integrationstests |

## Normative Verantwortungsregeln

1. **Keine Doppel-Owner:** `apps` verwaltet App-Identitäten, Registry und Installation, während der App-Data-Owner strukturierte App-Nutzdaten verwaltet und `files` allein VFS/Blobs. Ein App-Data-Aufruf darf nicht beliebig Tabellen anderer Owner lesen oder schreiben.
2. **Kein neuer Container:** R-008 Option C ist ein logischer Ownership-Entscheid; die aktuelle Liste von 22 Bootstrap-Rollen wird nicht allein deswegen geändert. `combined` bleibt MVP-Standard.
3. **Fachliche Transaktion:** App-Data-Mutation und zugehörige Outbox-Einträge gehören in **dieselbe eigene Owner-Transaktion**. Von außen nicht-atomare Workflows verwenden Idempotenz/Saga statt verteilter globaler ACID-Behauptungen.
4. **AuthZ:** Jede App-Data-Operation kontrolliert User, Tenant/Scope, App-Installation/-Instanz, signierte Deklaration und aktuelle Ressourcengrants. Der Client darf weder Tenant-ID noch App-Kontext selbst autoritativ vorgeben.
5. **Secrets:** Secret-Handles dürfen nicht als freie SQL-/VFS-Zugangsdaten oder dauerhafte App-Tokens nutzbar werden.

## Nicht automatisch geklärt / Verifikationsbelege

- Die Dokumentation verweist auf `core/internal/platform/services.json`; die Datei konnte bei der aktuellen GitHub-API-Abfrage nicht gelesen werden (`404`). Das ist eine **Quellenverifikationslücke**, kein Nachweis, dass der technische Katalog fehlt. Repository-Pfad und tatsächliche Rolle müssen mit git vor der Implementierungsfreigabe geprüft werden.
- Für App Data sind **Tabellennamen, Migrationen, Verschlüsselungsprofile, Quer-/Partitionierung und finale API DTOs** noch als technische Detailverträge auszuarbeiten. Eine frühere vorgeschlagene Auswahl zum Speicher-/Datenmodell ist **nicht** als vom Projektverantwortlichen entschieden nachgewiesen und bleibt deshalb ausdrücklich offen.
- Die endgültige Job-/Saga-, Query-/Index- und Secret-Persistenz wird pro Owner spezifiziert, bevor eine entsprechende Komponente implementiert wird. Dieses Mapping gibt dafür keine stillen neuen ADRs frei.
