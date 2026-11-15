# Risk, Conflict & Open Issues Register – App Framework SDK

> **Stand:** 15.11.2026 · **Status:** Erstpruefung anhand ausgewaehlter realer Repository-Dokumente; **keine vollstaendige Repo-weite Auditierung**.  
> `BLOCKER` = vor betroffener Umsetzung zwingend klaeren; `HIGH` = vor Schnittstellenfreeze klaeren; `MEDIUM` = vor konkretem Schritt; `LOW` = Nachpflege.

## 1. Konkrete Konflikte und Verifikationsaufgaben

| ID | Risiko / Beobachtung | Relevante Quellen | Prioritaet | Aufloesungsbedarf | Status |
|---|---|---|---|---|---|
| R-001 | `API_CONTRACTS.md` verlangt nur aktive API-Major; Q151 erlaubt parallele App-Releases. Ein App-Kompatibilitaetsfenster ist **kein** impliziter paralleler API-Major-Betrieb. | API_CONTRACTS, Q104/Q151 | HIGH | Kompatibilitaet mit nur einer aktiven API-Major konkret nachweisen; Beschluss K4/9 | KONZEPTIONELL_ENTSCHIEDEN |
| R-002 | `APP_SIGNING.md` erlaubt selbstsignierte `.cloudApp` nach Admin-Risikofreigabe; Q154 verlangt verbindliches Conformance-Gate. | APP_SIGNING, APP_API_ACCESS, Q100/Q154 | BLOCKER | Policy-gestuetzte Test-/Admin-Ausnahme definieren; Integritaet nie umgehen; Beschluss K1/9 | KONZEPTIONELL_ENTSCHIEDEN |
| R-003 | Altarchitektur nennt browser-WASM, optionale WASI-/Native-Komponenten; Q113 begrenzt ausfuehrbare App-SDK-Worker auf JS/TS und WASM ohne beliebigen nativen Code. | SYSTEM_ARCHITECTURE, APPLICATION_SYSTEM, Q113/Q137 | BLOCKER | Umfang von WASI, Native-Option und signierten Runtime-Profilen formell festlegen; Beschluss K2/9 | KONZEPTIONELL_ENTSCHIEDEN |
| R-004 | `PUSH_REALTIME.md` definiert SSE als regulaeren Browser-Push; Q035 nennt externe SSE-/WebSocket-Verbindungen. | PUSH_REALTIME, API_CONTRACTS, Q035/Q139 | HIGH | Browser-Push, externe Verbindungen, Web Push eindeutig trennen; Beschluss K5/9 | KONZEPTIONELL_ENTSCHIEDEN |
| R-005 | App-Origin, Service-Worker-Scope, Tab-Coordination und iframe-Sandbox muessen tatsaechlich getrennt sein; geteilte Origin koennte SW-Interception ermoeglichen. | APP_API_ACCESS, Q101/Q136/Q138/Q140 | BLOCKER | Nachweisbare Origin-/Scope-Topologie, Browser-Testmatrix; Beschluss K3/9 | KONZEPTIONELL_ENTSCHIEDEN |
| R-006 | Optionale E2EE verhindert bei clientseitigem Schluesselbesitz gegebenenfalls serverseitige Indexierung, Vorschau, Suche und diagnosebezogene Inhaltspruefung. | Q067/Q068/Q099/Q109 | HIGH | Datenklassifizierung und faehigkeitsbasierte Fallbacks; Beschluss K6/9 | KONZEPTIONELL_ENTSCHIEDEN |
| R-007 | 156 Fragen werden als behandelt gefuehrt; Q001–Q028 und Q030–Q033 sind im vorliegenden Kontext nicht rekonstruierbar. | Gespraechsverlauf, Q001–Q033 | HIGH | Originalentscheidungen verifizieren, Register vervollstaendigen | OFFEN |
| R-008 | Geplante logische Komponenten koennten mit dem vorhandenen Go-Core-Rollenkatalog doppelt erfasst werden. | SYSTEM_ARCHITECTURE, CORE_DEPLOYMENT_SCALING, Q156 | HIGH | Bestehende Service-Owner vollstaendig zuordnen | OFFEN |
| R-009 | Freiwilliger Consent kann nur Core-vermittelte Datenverarbeitung technisch kontrollieren; client-/providerseitige Verarbeitung ist begrenzt kontrollierbar. | Q057/Q102/Q142 | MEDIUM | Controller-/Datenflussgrenzen und Zweckbindung festlegen | OFFEN |
| R-010 | Retention- und Replay-Cursor duerfen Revocation, Loeschung, Audit-Pflicht sowie E2EE nicht umgehen. | API_CONTRACTS, Q058/Q097/Q129/Q148/Q149 | HIGH | Strom-Owner, Dedupe-/Retention-/Purge-Regeln; Beschluss K7/9 | KONZEPTIONELL_ENTSCHIEDEN |
| R-011 | Workflow-Kompensationen koennen nach Widerruf keine alten Rechte nutzen; gleichzeitig kann Recovery erforderlich sein. | Q058/Q127/Q146/Q147 | HIGH | Administrativen Recovery-Prozess und getrennte Autorisierung festlegen; Beschluss K8/9 | KONZEPTIONELL_ENTSCHIEDEN |
| R-012 | `APP_FRAMEWORK_SDK.md` fehlte bei GitHub-Lesezugriff auf `main`; diese Dateien sind ein erstmaliger vorgeschlagener Aufbau und noch keine gepruefte Integration. | Repo `Drezael/cloudOS`, 15.11.2026 | MEDIUM | Elf Dateien auf main abrufbar; Link-QA und Volltextintegration weiter pruefen | GITHUB_DATEIEN_VERIFIZIERT_LINK_QA_OFFEN |
| R-013 | Q036/Q052/Q075/Q090/Q105/Q130 sind ausdrueckliche Negativentscheidungen, die durch spaetere Details unabsichtlich verletzt werden koennten. | Decision Register | HIGH | Negative Contract-Tests und Redaktion | OFFEN |
| R-014 | Alte aktive App-Releases duerfen trotz Altversions-Uebergang keine bereits abgeschalteten API-Major-Versionen oder kompromittierte Module verwenden. | Q050/Q104/Q137/Q151 | HIGH | Kompatibilitaets- und Notfall-Revocation-Plan; Beschluss K9/9 | KONZEPTIONELL_ENTSCHIEDEN |
| R-015 | Conformance-Test gegen echte cloudOS-Instanz kann in Testumgebung privilegierte Ressourcen veraendern. | Q105/Q154 | MEDIUM | Isolierte Test-Tenants, synthetische Daten, Policy-/Rollback-Regeln | OFFEN |

## 1a. Abschlussstand K1–K9 vom 15.11.2026

Die Risiken **R-001, R-002, R-003, R-004, R-005, R-006, R-010, R-011 und R-014** besitzen jeweils einen ausdrücklich bestätigten Architekturentscheid (alle **Option C**). Ihr Status lautet deshalb **`KONZEPTIONELL_ENTSCHIEDEN`** – nicht `TECHNISCH_VERIFIZIERT` und nicht `CLOSED`.

| Status | Bedeutung |
|---|---|
| `OFFEN` | Prüffrage oder fachlicher Entscheid noch ausstehend |
| `KONZEPTIONELL_ENTSCHIEDEN` | Grundsatz durch K-Beschluss geklärt; Fachvertragsintegration und Nachweise fehlen |
| `DOKUMENTIERT` | Beschluss vollständig in den fachlich führenden Bestandsdokumenten integriert und auf Widersprüche geprüft |
| `TECHNISCH_VERIFIZIERT` | Geforderte Tests und Nachweise bestanden |
| `CLOSED` | Zuständige Abnahme dokumentiert und keine offenen relevanten Folgepunkte |

Die Beschlussinhalte stehen in [11-architecture-decisions-K1-K9.md](11-architecture-decisions-K1-K9.md), die ausdrücklich noch nicht eingearbeiteten Bestandsdateien in [12-legacy-alignment.md](12-legacy-alignment.md).

**Weitere offene Risiken:** R-007 (historische Entscheidungen), R-008 (Core-Owner), R-009 (Consent über Trust Boundaries), R-013 (Negativentscheidungen), R-015 (isolierte Konformitätstests). R-012 ist nur hinsichtlich Datei-Präsenz erledigt; die vollständige Link-QA steht aus. Offene technische Nachweise der konzeptionell entschiedenen Risiken bleiben eigenständige Umsetzungsgates.

## 2. Vorrang und Aenderungsprozess

Bei einem Konflikt wird kein bestehender Fachvertrag stillschweigend ueberschrieben. Fuer jede betroffene Entscheidung: (1) Originaltext finden, (2) Sicherheitsauswirkung und Owner bestimmen, (3) Loesungsvarianten dokumentieren, (4) expliziten ADR/Entscheid einholen, (5) alle betroffenen Dateien aktualisieren, (6) Vertrags-/Integrationstests definieren. Dokumentzeitstempel alleine erzeugen keinen Vorrang.

## 2a. Konsolidierungsstrategie

1. Beschlüsse K1–K9 sind ohne Code getroffen und in den SDK-Dokumenten abgebildet.
2. Führende Bestandsdokumente bleiben fachlich autoritativ; die Volltextintegration ist **nicht** Teil dieses ZIP und darf nicht als abgeschlossen gelten.
3. API-, Browser-, Native-/WASI-, E2EE-, Retention- und Recovery-Verträge müssen in der nächsten Spezifikationsphase technisch nachgewiesen werden.
4. Q001–Q033 und die noch rekonstruierten Entscheidungen bleiben historisch zu prüfen.

## 3. Fehlende Nachweise

Es liegt **kein vollstaendiger Audit aller Repository-Dateien** vor. Die Einsicht umfasst den per GitHub-Connector abgefragten Stand einzelner Dokumente, darunter `CONCEPT.md`, `SYSTEM_ARCHITECTURE.md`, `API_CONTRACTS.md`, `APPLICATION_SYSTEM.md`, `APP_API_ACCESS.md`, `APP_SIGNING.md`, `PUSH_REALTIME.md` und `CORE_DEPLOYMENT_SCALING.md`. Die nicht aufgelisteten weiteren Repository-Dokumente sind vor formeller Freigabe nachzuziehen.

## 4. Freigabeprotokoll (noch auszufuellen)

| Rolle | Verantwortliche Person | Datum | Ergebnis |
|---|---|---|---|
| Architektur | Offen | Offen | NICHT GEPRUEFT |
| Sicherheit | Offen | Offen | NICHT GEPRUEFT |
| API/SDK | Offen | Offen | NICHT GEPRUEFT |
| Betrieb | Offen | Offen | NICHT GEPRUEFT |
