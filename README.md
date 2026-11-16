# cloudOS – Beseitigung dokumentarischer Lücken, Stand 16.11.2026

**Quelle:** `Drezael/cloudOS`, Branch `main`, anhand GitHub `fetch_file` und Blob-SHA der in diesem Paket geänderten Quelldateien abgeglichen.
**Lieferumfang:** Vollständige Markdown-Ersatzdateien unter ihren Repositorypfaden. Keine Implementierung, kein Patchskript, keine GitHub-Schreiboperation.
**Ziel:** Bislang offene dokumentarische Konflikte, SDK-Frühentscheidungen und Readiness-Nachweise konsistent dokumentieren; nicht durch nicht vorhandene Quellen oder nicht ausgeführte Tests ein erfundenes `READY` ausstellen.

## Fachlich behobene Dokumentationslücken

- SDK Q001–Q033 aus früheren Gesprächsinhalten als fachliche Kurzentscheidungen rekonstruiert und zwischen belegten Antworten und unvollständigem Originalwortlaut unterschieden.
- R-008 Option C (logischer App-Data-Owner im Combined-Go-Core), R-009 Option C (versionierter Zweck-/Datenklasse-/App-/Empfänger-Consent), R-013 Negativtests und R-015 echte isolierte Testinstanz dokumentiert.
- K1–K9 mit neun fachlich führenden Bestandsdokumenten konsolidiert; zwingende App-Origin-Trennung und App-Release-Parallelitätsgrenzen explizit korrigiert.
- R-007, R-008, R-009, R-013, R-015 und der Readiness-Bericht mit korrekter Differenzierung zwischen fachlicher Entscheidung, Dokumentation, Quellennachweis und technischen Tests aktualisiert.
- Relative Querverweise geprüft; [QA-Bericht](docs/app-framework/20-link-and-markdown-qa.md).

## Vollständige veränderte und neue Repository-Dateien

- `docs/API_CONTRACTS.md`
- `docs/APPLICATION_SYSTEM.md`
- `docs/APP_API_ACCESS.md`
- `docs/APP_FRAMEWORK_SDK.md`
- `docs/APP_MANAGEMENT_STORE.md`
- `docs/APP_SIGNING.md`
- `docs/CORE_DEPLOYMENT_SCALING.md`
- `docs/PUSH_REALTIME.md`
- `docs/SYSTEM_ARCHITECTURE.md`
- `docs/VIRTUAL_FILESYSTEM.md`
- `docs/app-framework/01-architecture-baseline.md`
- `docs/app-framework/02-decision-register.md`
- `docs/app-framework/03-component-catalog.md`
- `docs/app-framework/04-interface-contracts.md`
- `docs/app-framework/05-security-architecture.md`
- `docs/app-framework/06-data-consistency.md`
- `docs/app-framework/08-dependency-matrix.md`
- `docs/app-framework/09-risk-register.md`
- `docs/app-framework/10-implementation-readiness.md`
- `docs/app-framework/11-architecture-decisions-K1-K9.md`
- `docs/app-framework/12-legacy-alignment.md`
- `docs/app-framework/13-phase2-bestandsvertrags-review.md`
- `docs/app-framework/14-owner-mapping.md`
- `docs/app-framework/15-consent-boundaries.md`
- `docs/app-framework/16-historical-evidence.md`
- `docs/app-framework/17-negative-contract-tests.md`
- `docs/app-framework/18-conformance-test-environment.md`
- `docs/app-framework/19-gap-and-evidence-register.md`
- `docs/app-framework/20-link-and-markdown-qa.md`

## Ausgangs-SHA für den manuellen Git-Abgleich

Vor dem Ersetzen prüfen, ob diese Git-Blobs noch dem aktuellen `main` entsprechen. Bei neueren Commits nicht blind überschreiben, sondern das jeweilige vollständige Dokument erneut zusammenführen.

| Pfad | Git-Blob-SHA vor dieser Änderung |
|---|---|
| `docs/API_CONTRACTS.md` | `b078c4abd296d9464042f45a7336c853aebebd7a` |
| `docs/APPLICATION_SYSTEM.md` | `8794c0c3ed64d8cf6b60cf6def802b4157ec998c` |
| `docs/APP_API_ACCESS.md` | `7a78700e515953d221738ed42daf2abf89f3f74b` |
| `docs/APP_FRAMEWORK_SDK.md` | `6a739ca6cf59bfbae2940083bd22731e1d02fad4` |
| `docs/APP_MANAGEMENT_STORE.md` | `2544f6f08fc2209f2ef6da19d177ace74b5f1d62` |
| `docs/APP_SIGNING.md` | `577a908834a603fbd4e221cc514ef4599af33a01` |
| `docs/CORE_DEPLOYMENT_SCALING.md` | `01492e9d80bcc6d467a94fbe3909e52460821a99` |
| `docs/PUSH_REALTIME.md` | `67bbdb283415b11ec13635e61c0371dc8236d8ea` |
| `docs/SYSTEM_ARCHITECTURE.md` | `9e71804b76f574d9b3a3e4256ab236cc68b8a679` |
| `docs/VIRTUAL_FILESYSTEM.md` | `5d6c8b1e95cc85cca136c6b85b1693bad3e934fa` |
| `docs/app-framework/01-architecture-baseline.md` | `efb98d6f372632e13017f7dd445a9773ea18ddef` |
| `docs/app-framework/02-decision-register.md` | `5995eb141efbda88a3ed7ead6e4759b5247bbd50` |
| `docs/app-framework/03-component-catalog.md` | `5dea174a492693aaae1f08573a21d7525d62c29c` |
| `docs/app-framework/04-interface-contracts.md` | `2dc25e8987d35912e6277595faf2c8d6ffd67f8a` |
| `docs/app-framework/05-security-architecture.md` | `db3a315c9d3f6d97f2531cb90013cf7b8efbc16a` |
| `docs/app-framework/06-data-consistency.md` | `78002c096307f59abdf3072a04554fb4728ef96d` |
| `docs/app-framework/08-dependency-matrix.md` | `25fd509705bfa93254cfdae29f6043e817bb7883` |
| `docs/app-framework/09-risk-register.md` | `cfd3373006dd064ee8ba6912ea24458b682d42a9` |
| `docs/app-framework/10-implementation-readiness.md` | `3398d021335060da68b50a541b01c8935ce45a19` |
| `docs/app-framework/11-architecture-decisions-K1-K9.md` | `f96db29ca667c35531636b04515cceb2f90518c0` |
| `docs/app-framework/12-legacy-alignment.md` | `3fd1d3a722738bc6b6da2e65d5264084ba978664` |

## Bewusst nicht als erledigt ausgegeben

1. Vollständige ursprüngliche Frage-/Optionswortlaute der SDK-Fragen sind nicht lückenlos zugänglich. Rekonstruktion ersetzt **nicht** Primärbelege.
2. Das im Dokument genannte `core/internal/platform/services.json` war über die GitHub-Dateiabfrage nicht abrufbar. Die logische Zuordnung beweist **nicht** die genaue technische Implementierung.
3. Finale OpenAPI-/DB-/Consent-/App-Data-Datenstrukturen und spezifische Grenzwerte benötigen nachweisbare Verträge beziehungsweise Messungen und ggf. Freigaben.
4. Kein Browser-/Conformance-/E2EE-/Restore-/Security-Test wurde ausgeführt und kein Implementation-Readiness-Sign-off erteilt.
5. **ADR-094 bleibt `Proposed`.** Andere nicht zum MVP gehörige Entscheidungen bleiben unberührt.

Der aktuelle Status und die jeweils noch benötigten Abschlussbelege stehen in [`docs/app-framework/19-gap-and-evidence-register.md`](docs/app-framework/19-gap-and-evidence-register.md).

**GitHub-Synchronisation:** Die verbundene Integration hat `push: false`. Dateien müssen mit Schreibzugriff auf den Repo-Branch übertragen und danach anhand der neuen Blobs verifiziert werden.
