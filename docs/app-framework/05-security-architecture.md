# Security Architecture – App Framework SDK

> **Status:** Sicherheitsarchitektur als Review-Grundlage, **kein abgeschlossener Threat Model oder Penetrationstest**.

## 1. Bedrohungsmodell

| Angreifer / Fehler | Moeglicher Angriff | Verbindliche Gegenmassnahme |
|---|---|---|
| Manipuliertes App-Paket | Einschleusen fremden Codes | X.509/Index-/Signaturpruefung, Release-Hash, Conformance Gate |
| Bestaetigter, aber boesartiger App-Code | Zugriff auf fremde Ressourcen | Separate Origins, Sandbox, Default Deny, Capability-Enforcement |
| Fremdes HTML/Webseite | XSS, Token-Diebstahl, Navigation | Sanitization, CSP, Trusted Types, isolierte Web-Views |
| Manipulierter Client-Kontext | Andere Benutzer/App impersonieren | Serverseitig abgeleitete Identitaet; kein Vertrauen in Client-Sicherheitsfelder |
| Externer Provider / Redirect | CSRF, Provider Mix-up, Token-Abfluss | PKCE, State, validierte Redirects, Broker, Secret Store |
| Externe URL | SSRF oder unkontrollierte Egress-Routen | Network Gateway, Allowlist/Default Deny |
| Webhook-Manipulation | Replay, fremde Events | Provider-Verifikation, Subscription-/Tenant-Bindung |
| Worker-Fehler | Privilegienausweitung, DoS | Sandbox, kurzlebige Execution Identity, Quotas |
| Cache/Service Worker | Veraltete Rechte, unkontrollierter Hintergrund | Revocation, Origin/Scope-Isolation, Managed Storage |
| Fehlgeschlagene Wiederholung | Doppelte Seiteneffekte | Idempotenz, Outbox/Inbox, Saga-Kompensation |
| Supportpaket | Offenlegung von Secrets/PII | Redaction, Berechtigung, Export-Review, Retention |

## 2. Vertrauensgrenzen

1. **Browser <-> Go-Core**: Session/Auth, Origin/CSP, no client-authoritative grants.
2. **Isolierte App <-> Trusted Host**: nur typisierte SDK-Bridge; fremde App darf keinen Core-Dialog nachahmen.
3. **Worker <-> Core Owner**: signiertes Modul, zweckgebundene Identity, alle Operationen erneut autorisiert.
4. **Core <-> Drittanbieter**: Network Gateway und Broker; keine direkte Weitergabe interner Tokens.
5. **Tenant A <-> Tenant B**: eindeutige Zuordnung von Daten, Nachrichten, Cache, Workflows und Diagnosen.
6. **Releasequelle <-> Runtime**: Signaturpruefung und aktive Freigabe; vertrauenswuerdiger Publisher ist keine Code-Sicherheitsgarantie.

## 3. Authentifizierung, Berechtigung und Zustimmung

Authentifizierung (inkl. Passkey/Step-Up), autorisierte Resource Operation, App-Endpunktgenehmigung und zweckgebundene freiwillige Zustimmung sind **separate** Pruefungen. Ein Consent erteilt keine technische Berechtigung; Step-Up ersetzt kein Resource Grant. Bei Widerruf werden weitere geschuetzte Operationen blockiert, auch wenn ein Event, Background Job oder Cache noch vorhanden ist.

## 4. Paket-, Runtime- und Codeintegritaet

- `.cloudApp`-Pakete und die signierte `permissions/api-access.json` werden gemass bestehenden Signing-/Endpoint-Dokumenten geprueft.
- Nachladen von JS/WASM nur aus identifizierten, autorisierten Release-Artefakten; kein beliebiger CDN-/`eval`-/Blob-Bypass.
- Fremdinhalte in getrennten Modi: Trusted App UI, Sanitized Content, Isolated External Web View.
- Service Worker erhalten keinen Zugang zu fremden Origins, Secrets oder Core-Rechten anderer Apps.
- Runtime-Instanzen unterschiedlicher Releases bleiben nach den Version-Compatibility-Regeln getrennt.

## 5. Kryptographische und Datenregeln

- Bestehendes Signing-Trust-Modell und KMS nicht stillschweigend ersetzen.
- Serverseitige Verschluesselung und optional echte E2EE mit clientgehaltenen Schluesseln strikt unterscheiden; E2EE-Daten duerfen nicht als serverseitig frei auswertbar angenommen werden.
- Keine dauerhaften Geheimnisse in Browser-Storage, Push-Payloads, Diagnosepaketen oder Audit-Daten.
- Kurzlebige Tokens sind zeit-, resource- und zweckgebunden. Replays/Leases/Fencing sind keine unabhängigen Credentials.

## 6. Security Fail-Closed Matrix

| Ausfall | Erlaubtes Verhalten | Verbot |
|---|---|---|
| Autorisierung nicht pruefbar | Geschuetzte Operation ablehnen | Stillschweigend erlauben |
| Secret Broker nicht verfuegbar | Externe Abhaengigkeit degradieren | Token unverschluesselt ausgeben |
| App Data Store unklarer Commit | `OUTCOME_UNKNOWN`, Statusklaerung | Blind neu schreiben und Erfolg behaupten |
| Preview faellt aus | Optional Funktion deaktivieren | Gesamtsicherheit abschalten |
| Event Replay-Luecke | Snapshot/Quelle neu synchronisieren | Alte Berechtigung durch Event behaupten |
| Audit nicht verfuegbar | Gemass Operationsklasse fail closed oder abgesicherte Queue | Pflicht-Audit heimlich verlieren |

## 7. Sicherheitsentscheidungen, die abzugleichen sind

- **R-002:** Signing erlaubt nach dokumentiertem Admin-Risikodialog selbstsignierte Pakete; Q154 verlangt Release Gate. Die Ausnahme darf weder stillschweigend wegfallen noch unkontrolliert Conformance umgehen.
- **R-003:** Vorhandene Architektur nennt WASI/optional Native, Q113 begrenzt App-Worker auf JS/TS/WASM. Scope und Sicherheitsmodell explizit klaeren.
- **R-004:** CSP/Origin/SW-Scope muessen technisch auch in Multi-Tab-Szenarien isolieren.
- **R-005:** Optional E2EE vs. serverseitige Indexierung, Vorschau, Backups, Diagnostik und Suche.

## 8. Mindest-Sicherheitsabnahme

Vor Implementierung je Komponente: Owner, Trust Boundary, Permissions, Failure Mode, Abuse Cases, Datenfluss und negative Integrationstests. Ein bestandener automatisierter Konformitaetstest ist kein Nachweis vollstaendiger Sicherheit.
