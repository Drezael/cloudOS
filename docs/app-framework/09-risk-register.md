# Risk, Conflict & Open Issues Register – App Framework SDK

> **Stand:** 16.11.2026 · **Status:** Fachliche Lücken und Bestandsvertrags-Konflikte konsolidiert; **keine vollständige Repo-weite oder technische Auditierung**.  
> `BLOCKER` = vor betroffener Umsetzung zwingend klaeren; `HIGH` = vor Schnittstellenfreeze klaeren; `MEDIUM` = vor konkretem Schritt; `LOW` = Nachpflege.

## 1. Konkrete Konflikte und Verifikationsaufgaben

| ID | Risiko / Beobachtung | Relevante Quellen | Prioritaet | Aufloesungsbedarf | Status |
|---|---|---|---|---|---|
| R-001 | `API_CONTRACTS.md` verlangt nur aktive API-Major; Q151 erlaubt parallele App-Releases. Ein App-Kompatibilitaetsfenster ist **kein** impliziter paralleler API-Major-Betrieb. | API_CONTRACTS, Q104/Q151 | HIGH | K4 ist in `API_CONTRACTS.md`, `APPLICATION_SYSTEM.md` und `APP_MANAGEMENT_STORE.md` fachlich harmonisiert; technische Versions-/Rollback-Tests folgen | DOKUMENTIERT_TEST_OFFEN |
| R-002 | `APP_SIGNING.md` erlaubt selbstsignierte `.cloudApp` nach Admin-Risikofreigabe; Q154 verlangt verbindliches Conformance-Gate. | APP_SIGNING, APP_API_ACCESS, Q100/Q154 | BLOCKER | K1 in `APP_SIGNING.md`, `APP_API_ACCESS.md` und Store-Vertrag integriert; Conformance-Nachweise folgen | DOKUMENTIERT_TEST_OFFEN |
| R-003 | Altarchitektur nennt browser-WASM, optionale WASI-/Native-Komponenten; Q113 begrenzt ausfuehrbare App-SDK-Worker auf JS/TS und WASM ohne beliebigen nativen Code. | SYSTEM_ARCHITECTURE, APPLICATION_SYSTEM, Q113/Q137 | BLOCKER | K2 in Architektur-, Runtime- und Deployment-Verträgen integriert; Sandbox-Nachweis folgt | DOKUMENTIERT_TEST_OFFEN |
| R-004 | `PUSH_REALTIME.md` definiert SSE als regulaeren Browser-Push; Q035 nennt externe SSE-/WebSocket-Verbindungen. | PUSH_REALTIME, API_CONTRACTS, Q035/Q139 | HIGH | K5 in `PUSH_REALTIME.md`, `APP_API_ACCESS.md` und `API_CONTRACTS.md` abgegrenzt; Tests folgen | DOKUMENTIERT_TEST_OFFEN |
| R-005 | App-Origin, Service-Worker-Scope, Tab-Coordination und iframe-Sandbox muessen tatsaechlich getrennt sein; geteilte Origin koennte SW-Interception ermoeglichen. | APP_API_ACCESS, Q101/Q136/Q138/Q140 | BLOCKER | K3: getrennte Origins im App-API-Vertrag verpflichtend; konkrete Browser-/SW-Topologie noch ungetestet | DOKUMENTIERT_TEST_OFFEN |
| R-006 | Optionale E2EE verhindert bei clientseitigem Schluesselbesitz gegebenenfalls serverseitige Indexierung, Vorschau, Suche und diagnosebezogene Inhaltspruefung. | Q067/Q068/Q099/Q109 | HIGH | K6: serververwaltete Kryptografie und optionale E2EE getrennt; Schlüssel-/Featuretests folgen | DOKUMENTIERT_TEST_OFFEN |
| R-007 | 156 Fragen werden als behandelt gefuehrt; Q001–Q028 und Q030–Q033 sind im vorliegenden Kontext nicht rekonstruierbar. | Gespraechsverlauf, Q001–Q033 | HIGH | Q001–Q033 fachlich als Kurzbeschlüsse aus früherem Gespräch rekonstruierbar; Originalfragen/Optionen teils ohne gesicherten Volltext; siehe `16-historical-evidence.md` | TEILWEISE_NACHGEWIESEN |
| R-008 | Geplante logische Komponenten koennten mit dem vorhandenen Go-Core-Rollenkatalog doppelt erfasst werden. | SYSTEM_ARCHITECTURE, CORE_DEPLOYMENT_SCALING, Q156 | HIGH | Option C beschlossen; logischer App-Data-Owner, keine neue Core-HTTP-Rolle; 22-Rollen-Zuordnung in `14-owner-mapping.md`, maschinenlesbarer Katalog nicht abrufbar | DOKUMENTIERT_QUELLPRUEFUNG_OFFEN |
| R-009 | Freiwilliger Consent kann nur Core-vermittelte Datenverarbeitung technisch kontrollieren; client-/providerseitige Verarbeitung ist begrenzt kontrollierbar. | Q057/Q102/Q142 | MEDIUM | Option C beschlossen; Versionierung, Empfänger-/App-/Datenkategorie-/Purpose-Bindung und begrenzter Entzug in `15-consent-boundaries.md`; Implementierungsnachweise fehlen | DOKUMENTIERT_TEST_OFFEN |
| R-010 | Retention- und Replay-Cursor duerfen Revocation, Loeschung, Audit-Pflicht sowie E2EE nicht umgehen. | API_CONTRACTS, Q058/Q097/Q129/Q148/Q149 | HIGH | K7 in API-/Realtime-/VFS-Bestandsverträgen integriert, numerische Retention und Tests fehlen | DOKUMENTIERT_TEST_OFFEN |
| R-011 | Workflow-Kompensationen koennen nach Widerruf keine alten Rechte nutzen; gleichzeitig kann Recovery erforderlich sein. | Q058/Q127/Q146/Q147 | HIGH | K8 im API-Vertrag integriert; Recovery-/Human-Approval-Test fehlt | DOKUMENTIERT_TEST_OFFEN |
| R-012 | `APP_FRAMEWORK_SDK.md` fehlte bei GitHub-Lesezugriff auf `main`; diese Dateien sind ein erstmaliger vorgeschlagener Aufbau und noch keine gepruefte Integration. | Repo `Drezael/cloudOS`, 15.11.2026 | MEDIUM | SDK-Dateien über GitHub-Blob-SHA bestätigt, relative Links im bearbeiteten Dokumentationsumfang geprüft; keine vollständige Code-/Repo-Auditierung | DOKUMENTIERT_QA_BEGR_ENZT |
| R-013 | Q036/Q052/Q075/Q090/Q105/Q130 sind ausdrueckliche Negativentscheidungen, die durch spaetere Details unabsichtlich verletzt werden koennten. | Decision Register | HIGH | Negativentscheidungen mit Test-ID, Szenario und erwarteter Ablehnung in `17-negative-contract-tests.md` spezifiziert; keine Tests ausgeführt | DOKUMENTIERT_TEST_OFFEN |
| R-014 | Alte aktive App-Releases duerfen trotz Altversions-Uebergang keine bereits abgeschalteten API-Major-Versionen oder kompromittierte Module verwenden. | Q050/Q104/Q137/Q151 | HIGH | K9 in Signing/App/Store/API-Vertrag integriert; Live-Kill-Switch/Quarantäne/Recovery noch technisch zu verifizieren | DOKUMENTIERT_TEST_OFFEN |
| R-015 | Conformance-Test gegen echte cloudOS-Instanz kann in Testumgebung privilegierte Ressourcen veraendern. | Q105/Q154 | MEDIUM | Option A: echte isolierte gemeinsame cloudOS-Testinstanz mit synthetischen Daten in `18-conformance-test-environment.md` dokumentiert; kein Testlauf | DOKUMENTIERT_TEST_OFFEN |

## 1a. Abschlussstand K1–K9 vom 15.11.2026

Die Risiken **R-001, R-002, R-003, R-004, R-005, R-006, R-010, R-011 und R-014** besitzen jeweils einen ausdrücklich bestätigten Architekturentscheid (alle **Option C**). Ihr Status lautet deshalb **`KONZEPTIONELL_ENTSCHIEDEN`** – nicht `TECHNISCH_VERIFIZIERT` und nicht `CLOSED`.

| Status | Bedeutung |
|---|---|
| `OFFEN` | Prüffrage oder fachlicher Entscheid noch ausstehend |
| `KONZEPTIONELL_ENTSCHIEDEN` | Grundsatz durch K-Beschluss geklärt; Fachvertragsintegration und Nachweise fehlen |
| `DOKUMENTIERT` | Beschluss vollständig in den fachlich führenden Bestandsdokumenten integriert und auf Widersprüche geprüft |
| `TECHNISCH_VERIFIZIERT` | Geforderte Tests und Nachweise bestanden |
| `CLOSED` | Zuständige Abnahme dokumentiert und keine offenen relevanten Folgepunkte |
| `TEILWEISE_NACHGEWIESEN` | Kurzbeschluss nachgetragen; Originalnachweis verbleibt |
| `DOKUMENTIERT_QUELLPRUEFUNG_OFFEN` | Architekturvertrag dokumentiert; eine technische Quellprüfung fehlt |
| `DOKUMENTIERT_TEST_OFFEN` | Fachnorm integriert, Test-/Betriebsnachweis fehlt |
| `DOKUMENTIERT_QA_BEGR_ENZT` | Prüfumfang begrenzt und benannt |

Die Beschlussinhalte stehen in [11-architecture-decisions-K1-K9.md](11-architecture-decisions-K1-K9.md), die ausdrücklich noch nicht eingearbeiteten Bestandsdateien in [12-legacy-alignment.md](12-legacy-alignment.md).

**Stand nach dem Dokumentationsabgleich 16.11.2026:** R-007 ist fachlich rekonstruiert, aber ohne durchgehende Originalnachweise (keine erfundenen Optionsbuchstaben); R-008 und R-009 sind inhaltlich entschieden und dokumentiert, technische Katalog-/Schema-/Laufzeitnachweise bleiben offen. R-013 und R-015 besitzen vollständige fachliche Prüfpläne, **keine** ausgeführten Tests. R-012 besitzt einen dokumentierten Link-QA-Prüfumfang. Sämtliche K1–K9-Konflikte sind in ihren führenden Bestandsdokumenten konsolidiert; technische Gates und formelles Sign-off bleiben ausstehend. Siehe [Lücken- und Evidenzregister](19-gap-and-evidence-register.md).

## 2. Vorrang und Aenderungsprozess

Bei einem Konflikt wird kein bestehender Fachvertrag stillschweigend ueberschrieben. Fuer jede betroffene Entscheidung: (1) Originaltext finden, (2) Sicherheitsauswirkung und Owner bestimmen, (3) Loesungsvarianten dokumentieren, (4) expliziten ADR/Entscheid einholen, (5) alle betroffenen Dateien aktualisieren, (6) Vertrags-/Integrationstests definieren. Dokumentzeitstempel alleine erzeugen keinen Vorrang.

## 2a. Konsolidierungsstrategie

1. Beschlüsse K1–K9 sind ohne Code getroffen und in den SDK-Dokumenten abgebildet.
2. Führende Bestandsdokumente bleiben fachlich autoritativ; die neun K1–K9-Beschlüsse wurden am 16.11.2026 in die vollständigen, mit GitHub-Blob-SHA abgeglichenen Bestandsdateien integriert. Technische Prüfungen werden nicht als abgeschlossen behauptet.
3. API-, Browser-, Native-/WASI-, E2EE-, Retention- und Recovery-Verträge müssen in der nächsten Spezifikationsphase technisch nachgewiesen werden.
4. Q001–Q033 haben wieder erkennbare Kurzbeschlüsse; der vollständige historische Originalwortlaut bleibt zu prüfen. Keine neu erfundenen Optionsbuchstaben.

## 3. Fehlende Nachweise

Es liegt **kein vollstaendiger Audit aller Repository-Dateien oder der laufenden Software** vor. Der Fokus liegt auf den bekannten SDK-/Architektur- und Bestandsvertragsdateien. Die Einsicht umfasst den per GitHub-Connector abgefragten Stand einzelner Dokumente, darunter `CONCEPT.md`, `SYSTEM_ARCHITECTURE.md`, `API_CONTRACTS.md`, `APPLICATION_SYSTEM.md`, `APP_API_ACCESS.md`, `APP_SIGNING.md`, `PUSH_REALTIME.md` und `CORE_DEPLOYMENT_SCALING.md`. Die nicht aufgelisteten weiteren Repository-Dokumente sind vor formeller Freigabe nachzuziehen.

## 4. Freigabeprotokoll (noch auszufuellen)

| Rolle | Verantwortliche Person | Datum | Ergebnis |
|---|---|---|---|
| Architektur | Offen | Offen | NICHT GEPRUEFT |
| Sicherheit | Offen | Offen | NICHT GEPRUEFT |
| API/SDK | Offen | Offen | NICHT GEPRUEFT |
| Betrieb | Offen | Offen | NICHT GEPRUEFT |
