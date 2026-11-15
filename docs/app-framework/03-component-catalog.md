# Component Catalog – cloudOS App Framework SDK

> **Status:** Fachlicher Katalog-Entwurf; logische Komponenten sind **keine** Aussage ueber getrennte Container. Standarddeployment bleibt Combined Go-Core.

## 1. Eigentumsprinzip

Jede Komponente besitzt genau einen fachlichen Owner fuer ihren autoritativen Zustand. Andere Komponenten kommunizieren ueber versionierte, autorisierte Vertraege. Berechtigung und Audit werden nicht auf App-Code delegiert. Die Bezeichnungen hier sind **logische Funktionen**, keine verbindlichen Paket-, Service- oder API-Namen.

## 2. Logischer Komponentenkatalog

| Logische Komponente | Verantwortlichkeit / State-Owner | Eingangsgrenze | Grundlage |
|---|---|---|---|
| Identity & Session | Benutzer/Sitzung, MFA/Passkeys, Step-Up, Widerruf | Trusted Core Auth | Q041/Q058/Q127/Q140/Q141 |
| Policy & Capability Gateway | Signierter Endpoint-Katalog, Policy-/Resource-Scope-Enforcement | Jede App-API-Operation | APP_API_ACCESS, Q041 |
| App Registry & Release | Paketquellen, Signaturen, Freigabe, Versionen | Installer, Administration | Q100/Q117–Q121/Q137/Q151/Q154 |
| App Instance & Window | Fenster, App-Instanzen, Zustands-Lifecycle | Browser-App-Host | Q080–Q083/Q134–Q135 |
| Browser Background & Tab Coordination | Service-Worker-Vertraege, sichere Tab-Leases | Browser-Kontext | Q138–Q140 |
| App Data Service | App-Daten, Owner, Revisionen, Constraints | Autorisierte SDK-/Domain-Operation | Q094–Q099/Q145 |
| VFS / Files | Dateizugriff, Datei-Jobs, Picker, Trash, Revisionen | Core File APIs | Q068–Q075/Q086–Q092 |
| Query / Index / Resource Watch | Autorisierte Abfragen, indexierte Metadaten, Deltas | SDK Query/Watch | Q064/Q066–Q067/Q109–Q110 |
| Messaging | Versionierte Nachrichten, At-least-once, DLQ, Replay | Autorisierte Event-Subscriptions | Q042–Q044/Q148–Q149 |
| Job / Saga / Scheduling | Persistente Jobs, Sagas, Entscheidungen, Deadlines | Worker-/Workflow-API | Q045–Q047/Q111/Q146–Q147 |
| Worker Runtime | Isolierte signierte JS/TS-/WASM-Ausfuehrung | Verwaltete Worker | Q112–Q115/Q127/Q137 |
| External Accounts / Secrets | Credentials, OAuth/OIDC, Secret-Broker, App-Grants | Kontrollierte Provider-Flows | Q034/Q128/Q143 |
| Network Gateway / Webhooks | Egress, SSRF-Schutz, Ingress, Provider-Events | Externe Netzgrenze | Q102–Q103/Q144 |
| Trusted UI Services | Dialoge, Navigation, Notification, i18n, Accessibility | App-Host/SDK | Q039–Q040/Q077–Q079/Q139 |
| Privacy / Consent / Audit | Zweckentscheidungen, Audit-Nachweise, Datenaufbewahrung | Alle geschuetzten Dienste | Q056–Q058/Q097/Q129/Q142 |
| Supervisor / Diagnostics | Health, Degradation, Logs, Supportberichte | Admin-/Betriebsgrenze | Q048/Q124/Q152–Q153 |
| Resource Accounting | Hierarchische Quotas, Rate Limits, Priorisierung | Shared Core Policy | Q047/Q115 |
| SDK Documentation / Conformance | Typ-/Schema-Referenzen, Pruefkatalog, Release-Gates | Build/Release | Q104–Q106/Q154–Q156 |

## 3. Querbezuege und Nicht-Zustaendigkeiten

- `Network Gateway` speichert nicht eigenmaechtig langfristige App-Secrets; er nutzt die berechtigte Vermittlung des Secret Brokers.
- `Messaging` garantiert nicht automatisch fachliche Verarbeitung genau einmal.
- `Diagnostics` darf keine Admin-Shell in unvertrauenswuerdigen Apps exponieren.
- `Window Manager` ist keine Autorisierungsinstanz fuer Dateien oder Daten.
- `Conformance` ersetzt weder Sandbox-Enforcement noch Runtime-Sicherheitspruefungen.
- Keine logische Komponente fuehrt ohne gesonderten ADR eine zweite Auth-, Scheduling-, Notification- oder Worker-Infrastruktur ein.

## 4. Deployment-Bezug

Der Combined-Core bleibt Standard. Split-/Kubernetes-Ausfuehrung sind opt-in. Servicegrenzen, Transaktions-Owner, Identitaet, Audit und Retry-Semantik muessen bei beiden Betriebsarten identisch sein. Die tatsaechliche Zuordnung zu den im Altbestand genannten Go-Core-Rollen ist im Repository-Vollabgleich festzuhalten; dieser Katalog ist kein Ersatz fuer den vorhandenen Deployment-Servicekatalog.

## 5. Offene Abnahmefragen

1. Welche Komponenten entsprechen bereits vorhandenen benannten Go-Core-Diensten?
2. Welche neue logische Funktion ist lediglich ein Zusatzvertrag eines bestehenden Services?
3. Wer haelt persistente Owner-Tabellen, Event-Outbox und Consumer-Positionen?
4. Welche Integrationspunkte sind innerhalb eines Combined-Core atomar und welche nicht?

Siehe [09-risk-register.md](09-risk-register.md).
