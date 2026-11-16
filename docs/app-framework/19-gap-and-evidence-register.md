# cloudOS – Verbleibende Dokumentations- und Evidenzlücken

> **Stand:** 16.11.2026 · **Status:** Volltextabgleich der bekannten SDK-/K1–K9-Dokumente durchgeführt; **keine** technische Implementierungs- oder Releasefreigabe.  
> **Quellen:** [SDK Readiness](10-implementation-readiness.md), [Risikoregister](09-risk-register.md), [ADR-Register](../ADR_REGISTER.md), [SDK-Gateway](../APP_FRAMEWORK_SDK.md).

## 1. Was rein dokumentarisch behoben wurde

1. Fachlich rekonstruierbare Kurzentscheidungen Q001–Q033 (strikt SDK, nicht API_CONTRACTS) in `02-decision-register.md`, ausführlicher Herkunftsnachweis in `16-historical-evidence.md`.
2. R-008 Option C als logischer App-Data-Owner im vorhandenen Combined-Go-Core beschrieben, ohne eine neue physische Service-Rolle anzulegen; Zuordnung zu bestehenden Core-Funktionen in `14-owner-mapping.md`.
3. R-009 Option C: versionierter Zweck-/Empfänger-/App-/Datenkategorie-Consent und korrekte externe Widerrufsgrenzen in `15-consent-boundaries.md`.
4. R-013: Negative SDK-/Sicherheits-Contract-Testfälle in `17-negative-contract-tests.md`.
5. R-015: gemeinsames echtes, isoliertes cloudOS-Testsystem mit synthetischen Daten und sicherem Cleanup in `18-conformance-test-environment.md`.
6. K1–K9 vollständig in neun maßgeblichen Fachdateien verankert: `APP_SIGNING`, `APP_API_ACCESS`, `APPLICATION_SYSTEM`, `SYSTEM_ARCHITECTURE`, `API_CONTRACTS`, `PUSH_REALTIME`, `VIRTUAL_FILESYSTEM`, `CORE_DEPLOYMENT_SCALING`, `APP_MANAGEMENT_STORE`.
7. Zwei explizite Formulierungskonflikte korrigiert: verbindliche per-App-Origin statt „vorzugsweise“, und ein Standardrelease versus zulässige laufende kompatible Altinstanzen unter einer aktiven API-Major.
8. SDK-Navigation, Risikoregister, historische Abgleichsdokumente und Readiness aktualisiert. Die Zahl `Accepted` im zentralen ADR-Register wird dadurch nicht erhöht; es werden **keine** neuen Grundsatzentscheidungen stillschweigend akzeptiert.

## 2. Noch ausstehende Belege – keine erfundene Vollständigkeit

| ID | Kategorie | Konkrete verbleibende Lücke | Erforderlicher Abschlussbeleg | Auswirkung |
|---|---|---|---|---|
| E-01 | Historische Originalquellen | Für einige Q001–Q033 fehlen weiterhin vollständiger Originalfrage-/Optionswortlaut oder Wahlbuchstaben; auch für rekonstruierte spätere Q-Einträge liegen nicht überall Primärbelege vor | Originalgespräch/Entscheidungsprotokoll mit eindeutiger Q-ID und Nutzerwahl gegen Fachkurzbeschlüsse prüfen | Keine stillschweigende Korrektur/Optionserfindung; Originalnachweis im Register offen |
| E-02 | Technischer Source-Owner | `core/internal/platform/services.json` war über die GitHub-Dateiabfrage nicht verfügbar (`404`); die derzeit dokumentierten 22 Rollen sind daher nicht gegen die Maschinenquelle verifiziert | Aktuellen Git-Pfad, Datei, Role-Katalog und technische Namen direkt im Repository vergleichen | Vor technischen Registry-/Service-Änderungen erforderlich |
| E-03 | App-Data-Datenmodell | R-008 legt **Owner**, nicht endgültige Datenstruktur/Partitionierung, SQL-Schema, Crypto-Verfahren oder API-DTOs fest | Aus dem fachlichen Owner abgeleitete versionierte OpenAPI-/Datenmodell-Spezifikation, DB-Constraints und Migrations-/Tenant-Isolationstests | Blockiert Implementierung dieses Daten-Owners |
| E-04 | Weitere SDK-Contracts | Operationen, Eventtypen, Error Codes, Typen, Idempotenz, Revisions-/Policy-Felder und genaue Endpunktnamen noch nicht vollständig maschinenlesbar | Versionierte Contract-Artefakte gegen `API_CONTRACTS.md` und explizite Owner/Sicherheitsfreigabe | Blockiert jeweilige SDK-/API-Implementierung |
| E-05 | Browser-Topologie | K3/ADR-060 sind beschlossen, aber finale DNS-/Origin-/Service-Worker-/iframe-/COEP-/CSP-Profile und echte Browsernachweise fehlen | Browser-Kompatibilitätsmatrix für Chrome/Edge/Firefox/Safari und überprüfte Origin-/SW-Isolation | Blockiert betroffene App-Runtime-Freigabe |
| E-06 | Runtime-Sandbox | Ausführung von Server-WASI/Native erst nach MVP; reale Sandbox/Resource Limits noch nicht spezifiziert | Spätere eigene Profile, Bedrohungsanalyse und Tests; nicht Bestandteil des MVP-Gates | Kein MVP-Blocker, solange Profil deaktiviert |
| E-07 | E2EE/Schlüssel | E2EE-/KMS-Capability-Semantik dokumentiert, konkrete Schlüsselverteilung, Recovery, Rotation und clientseitige Content-Matrix fehlen | Crypto-/Recovery-Fachvertrag und isolierte Testvektoren | Vor optionalem E2EE-Release; kein stiller Klartext-Fallback |
| E-08 | Retention und Daten-Lifecycle | Separate Outbox-/Inbox-/Delivery-/Audit-Owner dokumentiert; numerische Fristen/Quotas, Purge-, Recovery- und Replaygrenzen ungemessen | Datenklassifizierung und signierte Retention-/Purge-Matrix mit Last-/Revoke-Tests | Blockiert Freigabe der jeweiligen Events/Recovery-Komponenten |
| E-09 | RPO/RTO & Performance | ADR-012/060/136 setzen Mess-/Abnahmepflicht, aber keine gemessenen und vom Projektverantwortlichen freigegebenen Zahlen | Reproduzierbare Referenzumgebung, Leistungs-/Backup-/Restore-Messbericht mit bestätigten Grenzwerten | Keine irreführende HA-/SLA-Zusage; Release-Gate offen |
| E-10 | Conformance und Security-Tests | Testspezifikationen R-013/R-015 vorhanden, **keine** ausgeführte echte Instanzprüfung oder Browser-/Secret-/Kill-Switch-Negativtests | Reale isolierte cloudOS-Testumgebung, protokollierte Testergebnisse, Audit-/Rollback-Evidenz | Blockiert technische Abnahme |
| E-11 | Revision/Repo-Weite | Volltext- und relative Links im bearbeiteten Dokumentationsumfang geprüft; übriges Repository, GitHub-Main nach dem nächsten Push und externe Links noch nicht vollständig auditiert | Repo-weite Link-/Anchor-/Code-/Dokumentations-QA und GitHub-Blob-SHA-Abgleich nach Push | Kein pauschaler Voll-Audit behauptbar |
| E-12 | Formelle Genehmigung | Architektur-/Security-/API-/Operations-Sign-off und komponentenweise Definition of Ready nicht erteilt | Reviewprotokoll, Testbelege und ausdrücklich bestätigtes Freigabevotum | Full Implementation bleibt `NOT READY` |

## 3. Unterscheidung der Statusarten

- **`DOKUMENTIERT`**: fachliche Regel ist in vollständiger Fachdokumentation konsistent aufgeführt. Das ist **kein** Implementierungsbeleg.
- **`ORIGINALNACHWEIS_FEHLT`**: frühere Wahl ist fachlich rekonstruierbar, ihr genauer Wortlaut aber nicht zuverlässig belegt; darf nicht erfunden werden.
- **`TECHNISCH_NICHT_VERIFIZIERT`**: Spezifikation steht, aber Quellcode-, Browser-, Crypto- oder Integrationstests wurden nicht ausgeführt.
- **`NICHT_FREIGEGEBEN`**: ein formeller Freigabeakt ist ausstehend, unabhängig von erledigter Redaktion.

## 4. Priorität vor weiterer Entwicklung

1. **Zuerst Quellenbelege:** historisches Q-Protokoll, tatsächlich vorhandener Rollenquellkatalog, aktuelle Repo-Revision prüfen und Änderungen nicht anhand von Dateititeln oder vermuteten Pfaden erfinden.
2. **Dann maschinenlesbare Verträge:** besonders App Data/AuthZ/Consent/Messaging, echte Schemas, Fehlermodell, Datenklassifizierung, Retention und Upgrade/Rollback-Abhängigkeiten.
3. **Danach nachweisbare Tests:** echte cloudOS-Instanz und Browser-Sandbox, E2EE wo aktiviert, Restore-/Performance-/Widerruf-/Quarantäne-Gates.
4. **Erst mit Nachweisen formelle Readiness:** Architektur- und Security-Freigabe dokumentieren; keine automatische Implementation Ready aus `Accepted`-ADRs.

## 5. Verbindliche Abgrenzung

Die bereits beschlossenen MVP-ADRs werden **nicht** erneut zur Abstimmung gestellt. ADR-094 bleibt ausdrücklich `Proposed` und wird hier weder entschieden noch in seinen bestehenden Sicherheitsinvarianten abgeschwächt. Die offenen ADRs zu optionalen externen API-Clients, externen Identitäten, erweitertem WASI/ACL/Push und Integrationen werden nicht als zwingende MVP-Vorbedingungen umdeklariert. Diese Dokumente enthalten **keinen cloudOS-Implementierungscode**.
