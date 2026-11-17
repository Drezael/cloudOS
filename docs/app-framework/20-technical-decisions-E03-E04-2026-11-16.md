# cloudOS – App Data / SDK: Technische Detailentscheidungen 1–17

> **Datum:** 16.11.2026 · **Status:** Der Projektverantwortliche hat alle 17 Entscheidungen ausdrücklich mit Option B bestätigt.
> **Geltung:** Fachliche Architekturentscheidungen für E-03/E-04. Dies sind **keine bereits realisierten SQL-Tabellen, REST-Endpunkte oder kryptografischen Bibliotheken**.
> **ADR-094:** Von dieser Entscheidungsserie ausdrücklich ausgenommen.

## Vollständige Beschlussübersicht

| Nr. | Bereich | Gewählte Option |
|---:|---|:---:|
| 1 | PostgreSQL-Partitionierung | **B** |
| 2 | Index-Lifecycle | **B** |
| 3 | Schema-Migrationen | **B** |
| 4 | Speicherquoten und Ressourcenabrechnung | **B** |
| 5 | Deklaratives Abfragemodell | **B** |
| 6 | Atomare Mehrdatensatz-Schreiboperationen | **B** |
| 7 | Referenzintegrität und Löschen | **B** |
| 8 | Transaktionsisolation | **B** |
| 9 | Schutzklassenbasierte Feldverschlüsselung | **B** |
| 10 | Suchbarkeit verschlüsselter Felder | **B** |
| 11 | App-Release-Datenadapter | **B** |
| 12 | Operation-Outcome-Register | **B** |
| 13 | Kanonische Core-operation_id | **B** |
| 14 | Fehlerkatalog | **B** |
| 15 | App-Data-Cursor-Konsistenz | **B** |
| 16 | Kanonische Feldtypen und null | **B** |
| 17 | Datensatz-IDs und Offline-Mapping | **B** |

## Verbindliche Details

### Technische Detailfrage 1: PostgreSQL-Partitionierung

**Entscheidung:** Option B. Gemeinsame, logisch isolierte Tabellen für den MVP; reale Partitionierung erst bei belegtem Bedarf auf Basis der Last- und Volumenmessung vorbereiten. Keine sofortige physische Partitionierungs-Pflicht.

### Technische Detailfrage 2: Index-Lifecycle

**Entscheidung:** Option B. Signierte Schema-/Indexdeklarationen; Core-Validierung; begrenzt asynchroner Indexaufbau, kontrollierte Aktivierung indexabhängiger Funktionen erst nach Bereitschaft; Fortschritt, Fehler und sichere Wiederaufnahme.

### Technische Detailfrage 3: Schema-Migrationen

**Entscheidung:** Option B. Expand-and-Contract mit checkpointfähigem, idempotentem Backfill unter Quotas; kompatible Alt-Funktionen können weiterlaufen, neue Features erst nach Nachweis; kein blinder Daten-Rollback.

### Technische Detailfrage 4: Speicherquoten und Ressourcenabrechnung

**Entscheidung:** Option B. Mehrdimensionale hierarchische Budgets für logisches Volumen, Datensätze, Indexe, Migration und Query/Write; ergänzende physische PostgreSQL-Schutzschwellen; Grenzwerte aus Messungen.

### Technische Detailfrage 5: Deklaratives Abfragemodell

**Entscheidung:** Option B. Typisierte dynamische Filter, begrenzte logische Operatoren, Sortierung, Feldprojektion, Cursor-Pagination; Schema-, Rechte- und Kostengrenzen im Go-Core; keine App-SQL.

### Technische Detailfrage 6: Atomare Mehrdatensatz-Schreiboperationen

**Entscheidung:** Option B. Registrierte, versionierte Domain-Operationen für mehrere Datensätze innerhalb desselben Owners mit Revisionen, Constraints, Idempotenz, Quotas und atomarer Outbox; Cross-Owner-Sagas.

### Technische Detailfrage 7: Referenzintegrität und Löschen

**Entscheidung:** Option B. Versionierte referenzielle Regeln: RESTRICT standardmäßig; SET NULL und CASCADE nur ausdrücklich erlaubt, vorab autorisiert und limitiert. Keine app-/owner-/VFS-übergreifenden Kaskaden.

### Technische Detailfrage 8: Transaktionsisolation

**Entscheidung:** Option B. READ COMMITTED als Normalfall mit Revisions- und Sperrregeln; SERIALIZABLE für besonders kritische Invarianten; kontrollierte Deadlock-/Serialization-Retries ausschließlich sicher idempotent.

### Technische Detailfrage 9: Schutzklassenbasierte Feldverschlüsselung

**Entscheidung:** Option B. Versionierte Schutzklassifikation, authentifizierte Verschlüsselung sensibler Felder, getrennte KMS-Schlüssel; keine unbeabsichtigten Klartextindizes/Logs; server-managed von E2EE unterscheiden.

### Technische Detailfrage 10: Suchbarkeit verschlüsselter Felder

**Entscheidung:** Option B. Nur explizit freigegebene Gleichheits-Suche über schlüsselgetrennte, domänengebundene Blind-Indizes. Keine allgemeine Präfix-/Range-/Volltextsuche; Häufigkeits-/Gleichheitslecks berücksichtigen.

### Technische Detailfrage 11: App-Release-Datenadapter

**Entscheidung:** Option B. Ein kanonisches Datenmodell; zeitlich begrenzte, geprüfte Adapter für kompatible Alt-Releases; schreibende alte Versionen dürfen unbekannte neue Felder nicht verlieren; unsichere Übersetzungen ablehnen.

### Technische Detailfrage 12: Operation-Outcome-Register

**Entscheidung:** Option B. Owner-transaktionales, kontextgebundenes Ergebnisregister für mutierende Operationen; Statusklärung bei unklarem Commit, deduplizierte Idempotenz und getrennte Retention.

### Technische Detailfrage 13: Kanonische Core-operation_id

**Entscheidung:** Option B. Hierarchisches Muster owner.resource.action mit stabilen, registrierten IDs; Rechte weiterhin nur durch signierte Allowlist und aktuelle Grants.

### Technische Detailfrage 14: Fehlerkatalog

**Entscheidung:** Option B. RFC-9457-kompatible globale Fehlercodes plus registrierte fachliche Untercodes; Typen für JS/TS/Rust, sichere Status-/Retry-Semantik und keine Datenlecks.

### Technische Detailfrage 15: App-Data-Cursor-Konsistenz

**Entscheidung:** Option B. Hybride revisionsgebundene Keyset-Cursor und begrenzte Ergebnis-Snapshots; aktuelle Rechte bei jeder Seite; ungültige Cursor führen zu sicherem Neustart, keine langlaufende DB-Transaktion.

### Technische Detailfrage 16: Kanonische Feldtypen und null

**Entscheidung:** Option B. Sprachübergreifendes verlustfreies Typmodell: Integer/Decimal/Zeit/IDs/Strukturen; missing != null != value; partielle Updates lassen fehlende Felder unverändert.

### Technische Detailfrage 17: Datensatz-IDs und Offline-Mapping

**Entscheidung:** Option B. Endgültige opake IDs ausschließlich vom Owner, dauerhaft stabil und nicht wiederverwendbar; lokale Offline-IDs werden bei synchronisiertem atomarem Import auf echte IDs und Referenzen abgebildet.

## Sicherheits-, Versions- und Owner-Invarianten

- App Data ist ein separater **logischer** Daten-Owner im bestehenden Combined-Go-Core (R-008 C), nicht ein neuer Pflichtcontainer.
- Kein App-Direktzugriff auf PostgreSQL, fremde Core-Schemas, Dateiblobs, Schlüsselmaterial oder NATS.
- Schreibungen und Outbox innerhalb des Owners werden atomar; Cross-Owner-Abläufe verwenden Sagas und aktuell autorisierte Schritte.
- Nutzer-/Tenant-/App-/Instanzkontext wird vom Go-Core geprüft und nicht blind aus App-IDs oder Payloads vertraut.
- Eine aktive Core-API-Major bleibt verbindlich; zeitweise parallele kompatible App-Releases sind davon getrennt.
- Keine E2EE-Daten in serverseitigen Klartextindizes und keine implizite Klartextverarbeitung echter Client-Key-E2EE-Inhalte.
- Kritische Parameter und Produktivgrenzen (Kryptoalgorithmen/Nonce/Key-Lifecycle, Limits, TTLs, gemessene p95/p99 und RPO/RTO) bleiben vor Produktiveinsatz test- und freigabepflichtig; keine unbelegten Produktivdefaults.

## Spezifikations- und Evidenzgates (nicht automatisch erledigt)

1. **E-03:** endgültiges versioniertes PostgreSQL-Datenmodell (Tabellen, Constraints, RLS-/Isolation, partieller Indexaufbau, Migration, Verschlüsselung und Key-Metadaten) sowie Prüfvektoren und Tenant-Negativtests.
2. **E-04:** finaler maschinenlesbarer Operation-/Error-/DTO-/Eventkatalog über OpenAPI 3.1, Protobuf, AsyncAPI und signierte App-Capabilities.
3. **E-08:** Fach-/Audit-/Outbox-/Outcome-/Snapshot-Retention und sichere Löschung per Datenklasse.
4. **E-09/E-10:** reproduzierbare Last-, Failover-, Backup-/Restore-, Browser- und Sicherheitsabnahmen.
5. **E-11/E-12:** repo-weiter Dokumentations-/Vertragsabgleich und ausdrückliche technische Releasefreigabe.

## Bestehende Referenzen

- [App Framework SDK](../APP_FRAMEWORK_SDK.md)
- [Datenkonsistenz](06-data-consistency.md)
- [SDK-Interface-Verträge](04-interface-contracts.md)
- [Gap-/Evidence-Register](19-gap-and-evidence-register.md)
- [API-Verträge](../API_CONTRACTS.md)
- [ADR-Register](../ADR_REGISTER.md)
