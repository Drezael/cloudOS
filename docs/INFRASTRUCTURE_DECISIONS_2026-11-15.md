# cloudOS - Verbindliche Infrastrukturentscheidungen vom 15.11.2026

> **Status:** Fachlich beschlossen durch den Projektverantwortlichen am 15.11.2026.  
> **Verbindlichkeit:** Architekturentscheidungen, **kein** Nachweis der Implementierung oder Produktionsreife.  
> **Ausgangsstand:** `Drezael/cloudOS`, Branch `main`, dokumentierte Git-Blob-SHAs im zugehoerigen Abgleichskript.  
> **Fachliche Referenzen:** [ADR_REGISTER.md](ADR_REGISTER.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md), [API_CONTRACTS.md](API_CONTRACTS.md), [PUSH_REALTIME.md](PUSH_REALTIME.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

## 1. Geltungsbereich und Entscheidungsprioritaet

Die hier aufgefuehrten Entscheidungen sind die spaeteren, ausdruecklich bestaetigten
Architekturentscheidungen fuer Deployment, Infrastruktur, Datenkonsistenz und
Betrieb. Widersprechende aeltere Formulierungen (beispielsweise Rolling Core
Updates als verpflichtende Standardstrategie) gelten insoweit als ueberholt.

Bereits beschlossene unabhaengige Sicherheits- und Fachvertraege bleiben gueltig.
Insbesondere werden keine neuen Privilegien fuer fremde Apps oder globale
Schreibrechte fuer Core-Dienste eingefuehrt. Die Dokumentation wird konsolidiert;
Go-Code, API-Endpunkte und Deployment-Manifeste werden dadurch nicht implementiert.

## 2. Statusuebersicht

| ADR / Entscheidung | Bestaetigte Variante und Status |
|---|---|
| ADR-093 | Owner-eigene DB-Migration mit zentraler, persistenter Koordination und Expand/Migrate/Contract - Accepted |
| ADR-094 | Grundschutz und Teilentscheidungen zu Secrets/Edge festgelegt; konkrete K8s-Produktionsprofile bleiben Proposed |
| ADR-095 | Kontrollierter Combined-to-Split-Cutover mit eindeutiger Owner-Uebergabe - Accepted |
| ADR-096 | Abhaengigkeitsbewertete Health-/Readiness-Semantik und selektive Ausfaelle - Accepted |
| ADR-097 | Owner-spezifische SLIs/SLOs, reproduzierbare Lasttests, kontrollierte Skalierung - Accepted |
| ADR-098 | Kustomize als K8s-Referenz; Helm optional; Operator nur nach eigener Freigabe - Accepted |
| ADR-182 | Bedarfsgesteuerte Auslagerung einzelner Core-Dienste - Accepted |
| ADR-183 | Extern verwaltete PostgreSQL-/JetStream-HA mit Failover-Vertrag - Accepted |
| ADR-184 | Fachlich getrennte JetStream-Streams und zentrale Consumer-Vertraege - Accepted |
| ADR-185 | Ereignisklassenspezifische Retention mit Gap-Recovery - Accepted |
| ADR-186 | Bestaetigte Outbox-Publikation mit Retention und Reconciliation - Accepted |
| ADR-187 | Kontrollierte Offline-Migration des Storage-Backends - Accepted |
| ADR-188 | Providerunabhaengige Versorgung mit Betriebsgeheimnissen - Accepted |
| ADR-189 | Providerunabhaengige Edge-/Ingress-/TLS-Architektur - Accepted |

## 3. Nicht verhandelbare Betriebsgrenzen

1. **Combined bleibt Standard:** Ein Go-Core-Prozess in einem Core-Container; PostgreSQL,
   NATS und Edge sind getrennte Infrastrukturkomponenten. Split und Kubernetes bleiben opt-in.
2. **Manuelle Core-Releases:** Keine automatische Release-Suche, kein automatischer
   Core-Download, keine selbsttaetigen Software-Upgrades. Nur administrativ
   bereitgestellte, signatur-/integritaetsgepruefte, kompatible und freigegebene Releases.
3. **Vollstaendiger Core-Wartungsstopp:** Bei geplanten Core-Upgrades und produktiven
   Topologiemigrationen wird der Core vollstaendig angehalten. Laufende Anfragen,
   Streams und Jobs werden abgebrochen, nicht durch ein verpflichtendes Drain
   automatisch weitergefuehrt. Dauerhaft committete Daten bleiben erhalten;
   unklare externe Seiteneffekte werden geprueft, nie blind wiederholt.
4. **Kein Canary/Rolling-Core-Release:** Kein verpflichtender Progressive Rollout,
   kein automatisches Zwischenstoppen aufgrund von Metriken, kein automatisches
   Release-Rollback. Nachkontrolle und ausdrueckliche Freigabe sind Pflicht.
5. **Core-unabhaengiger Edge-Wartungsmodus:** Der Reverse-Proxy kann bei komplett
   gestopptem Core eine Wartungsseite und fuer APIs passende 503-Antworten liefern.
   Eine erfolgreiche Prozess-Readiness hebt die Wartung nicht selbsttaetig auf.
6. **Kein automatisches globales DR-Fencing oder DR-Failback:** Lokaler Failover
   innerhalb derselben zulaessigen HA-Domaene ist etwas anderes als die
   Aktivierung einer getrennten Disaster-Recovery-Installation.
7. **Strikte Owner-Grenzen:** Pro Ressource genau ein fachlich autoritativer
   Schreib-Owner, owner-spezifische PostgreSQL-Zustaendigkeit, keine
   cross-owner Direkt-SQL-Abfragen oder globale ACID-Transaktion.
8. **Sicherheitsgrenzen:** API-Operationen, Widerrufe, Signaturen, Berechtigungen,
   Audit-Pflichten und App-Origin-Isolation werden nicht durch ein Deployment-Tool
   oder eine Health-/Degraded-Regel geschwaecht.

## 4. Core-Migration, Betrieb und Skalierung

### 4.1 ADR-093: Owner-DB-Migrationen

- Jeder Owner verantwortet seine eigenen Schema-/Datenmigrationen.
- Die zentral koordinierte Ausfuehrung dokumentiert Versionen, Artefakt-Hashes,
  Abhaengigkeiten, Fortschritte und fehlgeschlagene Schritte dauerhaft.
- Ablauf fuer kompatible Aenderungen: Expand -> Migrate -> Contract.
- Lange Backfills verwenden pruefbare Checkpoints; DB-Rechte sind temporaer
  und auf den jeweiligen Owner beschraenkt.
- Unvollstaendige oder inkompatible Migrationen blockieren die Freigabe;
  ein altes Container-Image kann eine irreversible Schemaaenderung nicht zuruecknehmen.

### 4.2 ADR-095: Combined-to-Split

- Vorbereitungspruefung der Zieltopologie, Versions-/Schema-/Capability-Vertraege
  und eines verifizierten Recovery-Punkts.
- Core-unabhaengigen Wartungsmodus aktivieren und alle Core-Prozesse stoppen.
- Alte Schreibzustaendigkeiten nachweislich entziehen, Owner-Registry kontrolliert
  auf die neue Topologie umstellen und genau einen produktiven Writer sicherstellen.
- Neue Workload-Identitaeten, mTLS, Routing, Security-Capabilities sowie Daten-
  und Health-Vertraege pruefen; erst nach Freigabe wieder oeffnen.
- Keine Migration bei live parallel schreibendem Combined- und Split-Owner.
  Eine Rueckumstellung ist eine eigene gepruefte Topologieoperation, kein DR-Failback.

### 4.3 ADR-096 und ADR-097: Health, SLO, Lasttests

- Startup, Liveness, Readiness und Functional Health sind getrennte Vertraege.
- Jeder logische Owner beurteilt zwingende und optionale Abhaengigkeiten einzeln;
  Teilausfaelle sollen unabhaengige Funktionen nicht grundlos deaktivieren.
- Sicherheitsrelevante Autorisierungs- und Datenkonsistenzregeln bleiben Fail-Closed.
- Fehlende Messdaten werden als unbekannt, nicht als erfolgreich, dargestellt.
- SLIs/SLOs sind dienstspezifisch; fuer jedes Betriebsprofil werden die messbaren
  Ziele mit dokumentierter Referenzhardware und Lastprofil validiert.
- K8s-HPA ist im Combined-Standard nicht notwendig. Realtime, Job-Worker und
  IO-lastige Dienste benoetigen passende Fachmetriken statt nur CPU-Schwellen.

### 4.4 ADR-098 und ADR-182: Deployment-/Split-Strategie

- Docker Compose ist der Standard; Kustomize die K8s-Referenz.
- Helm ist optional und muss nachweislich dieselbe fachliche Konfiguration
  abbilden. Ein Operator benoetigt eine gesonderte Sicherheits- und Nutzenfreigabe.
- Core-Dienste werden nur bei messbarem Nutzen und hinreichender technischer Reife
  getrennt. Realtime, Gateway und File sind Beispiele, aber keine feste Reihenfolge.
- Teil-Split-Topologien sind erlaubt, wenn die eindeutige Owner-Registry und
  die kontrollierte Topologie-Uebergabe implementiert und geprueft sind.

## 5. Datenhaltung, JetStream und Wiederherstellung

### 5.1 ADR-183: Infrastruktur-HA

- PostgreSQL und JetStream werden ausserhalb der fachlichen Core-Owner verwaltet.
- Fuer zugesagte HA-Profile sind gepruefte, ausreichend redundante Infrastrukturen
  erforderlich; einzelne Instanzen bleiben fuer einfachere Profile zulaessig.
- DB-Primary-Failover darf niemals Split-Brain-Schreibberechtigungen ermoeglichen.
- Replikationsverzoegerte Daten duerfen sicherheitskritische Entscheidungen nicht
  mit ueberholten Rechten treffen lassen.
- Unklarer DB-Commit wird anhand der Operation-/Idempotenz-ID reconciliert,
  nicht blind wiederholt.
- Ein Infrastruktur-Failover innerhalb der HA-Domaene ist kein DR-Failback.
- Replikation ersetzt kein Backup/PITR und keine koordinierten Recovery-Punkte.

### 5.2 ADR-184: Fachliche JetStream-Topologie

- Fachlich und sicherheitlich passende Stream-Klassen statt eines einzigen
  unbeschraenkten globalen Streams oder eines Streams pro Consumer.
- Jedes Ereignis besitzt Owner, Version, CloudEvents-Envelope und stabile Event-ID.
- Consumer sind durable, scoped und haben ein eigenes Ack-/Retry-/Quarantaene-Profil.
- Verarbeitung ist at-least-once und idempotent. Revisionssicherheit gilt
  dort, wo fachlich notwendig, insbesondere pro Ressource; kein globales Ordering.
- Security Audit bleibt seine eigene persistente Nachweisquelle.

### 5.3 ADR-185: Retention und Replay-Gaps

- Separate Zeit- und Speichergrenzen je Ereignisklasse und Consumer-Anforderung.
- Ein Consumer muss abgelaufene Replay-Positionen als Luecke erkennen koennen.
- Rekonstruierbare Projektionen werden aus autoritativen Owner-Snapshots
  mit nachweisbar konsistentem Revisions-/Cursor-Uebergang wiederaufgebaut.
- Irreversible oder unbekannte externe Nebenwirkungen werden nicht blind nachgeholt.
- Consumer-Lag und bevorstehender Retention-Verlust loesen Monitoring aus;
  Speicherlimits erzwingen kontrollierte Backpressure statt stillem Datenverlust.
- Alte Events duerfen widerrufene Rechte oder geloeschte Daten nicht reaktivieren.

### 5.4 ADR-186: Outbox-Lebenszyklus

- Owner-Mutation und Outbox-Eintrag werden atomar in derselben DB-Transaktion gesichert.
- Publisher koordiniert per Claim/Lease/Fencing, verwendet stabile Event-IDs und
  verarbeitet unklare Publish-Acks durch sichere Wiederholung.
- Ein Outbox-Eintrag ist nach gueltigem JetStream-Persistenz-Ack *published*;
  fachliche Consumer-Acks sind eine separate Verarbeitungsfrage.
- Bestaetigte Events bleiben fuer klassenspezifische Recovery-Anforderungen
  erhalten und werden erst danach kontrolliert bereinigt.
- Reconciliation entdeckt haengende Claims, verwaiste Publisher, Ack-Unklarheiten
  und dauerhafte Publikationsrueckstaende.
- Zwischen PostgreSQL und JetStream wird keine globale ACID-Transaktion behauptet.

### 5.5 ADR-187: Storage-Migration

- Das VFS behaelt exakt zwei logische Sicherheitsdomaenen: `system:/` und `users:/`.
- Wechsel von lokalem zu gemeinsamem Backend nur nach Preflight und vollstaendigem
  Core-Wartungsstopp; kein Produktiv-Dual-Write.
- PostgreSQL-Metadaten, Blob-/Blockmanifeste, Schluesselgenerationen und Daten
  werden auf einen konsistenten Recovery-Punkt bezogen.
- Opake Storage-Keys, authentische Bloecke, Checksums, Revisionen und
  Namespace-Trennung werden vor Cutover vollstaendig verifiziert.
- Quell-Backend bis zur gesicherten Uebernahme und erforderlichen Schutzfrist
  behalten; keine unkontrollierte Loeschung oder Reaktivierung geloeschter Daten.

## 6. Betriebsgeheimnisse, Edge und Ingress

### 6.1 ADR-094 und ADR-188: Secret Provider

- Keine geheimen Werte in Git, Images, gewoehnlichen ConfigMaps oder Klartext-Helm-Values.
- Docker Compose kann geschuetzte read-only Secret-Dateien verwenden;
  Kubernetes und Split-Dienste koennen einen externen Secret-Provider anbinden.
- Betriebsgeheimnisse sind dienstspezifisch und least-privileged. Combined-Module
  sind innerhalb des gemeinsamen Go-Prozesses nicht vollstaendig OS-isoliert.
- Rotation: neue Version autorisiert bereitstellen, validieren, aktivieren,
  Verbindungen erneuern/restarten, verifizieren und alte Version widerrufen.
- Provider-Ausfall erlaubt keinen ungeprueften Secret-/Revoke-Fallback.
- Vorbereitete Kubernetes-Templates sind nicht allein dadurch produktionsreif;
  NetworkPolicies, Secret-Versorgung, TLS, Audit und Tests sind erforderlich.

### 6.2 ADR-189: Edge, Proxy und TLS

- Nginx bleibt Compose-Standard; geeignete andere Reverse-Proxys bzw.
  Kubernetes-Ingress-Controller sind bei gleichem Sicherheitsvertrag erlaubt.
- Produktive Browser/API-Zugriffe sind HTTPS-pflichtig. Interne getrennte
  Core-Dienste verwenden mTLS, echte Workload-Identitaet und Service-Capabilities.
- Forwarded-/Host-/Client-IP-Header nur von explizit autorisierten Proxys
  anerkennen; untrusted Client-Header entfernen oder ueberschreiben.
- Fremde App-Origins erhalten keine wiederverwendbaren BFF-Session-Cookies.
  CSP, CORS, CSRF und App-Bridge-Policies gelten in allen Proxyvarianten.
- SSE wird ohne verzerrendes Response-Buffering uebertragen; kein Polling-Fallback.
- Edge kann waehrend vollstaendigem Core-Stopp eine Wartungsseite anbieten;
  nach dem Upgrade nur kontrollierte administrative Freigabe.
- Automatische TLS-Zertifikatserneuerung ist optional und *kein*
  automatisches Core-Softwareupdate.

## 7. Offene Detailparameter - keine neue Grundsatzfrage

| Teilbereich | Vor Implementierung bzw. produktiver Freigabe festzulegen |
|---|---|
| DB | Migrations-Versionen, Schema-Dependencies, Lock-/Rollback-Grenzen, Poolgroessen |
| Health | Essential/Optional-Abhaengigkeiten, Probe-Timeouts, Degraded-Schwellwerte |
| SLO/Capacity | p95/p99, Fehlerbudgets, Referenzhardware, Nutzerzahl, Testdaten, Lastprofil |
| HA | DB-Replikationsart, Failover-Fristen, RPO/RTO, JetStream-Replikation/Quorum |
| JetStream | Event-Subjects, Streams, Consumer-Namen, Ack-Wait, Max-Attempts, Backoff |
| Retention | Zeit-/Speicherbudgets je Klasse, Replay-/Duplikatfenster, Gap-Warnschwelle |
| Outbox | Claim-/Lease-Dauer, Ack-Status, Retention und Reconciliation-Pruefzyklen |
| Storage | Adapterversion, Staging, S3-/Shared-FS-Vertrag, Checksums, Cutover-Tests |
| Secrets | Anbieterprofile, Bereitstellung, Rotations-/Widerrufsgrenzen, HSM/KMS |
| Edge | Trusted-Proxy-Liste, Host-/Cookie-/CSP-Policy, TLS-Profil, SSE-Timeouts |
| Kubernetes | Requests/Limits, Replikazahl, NetworkPolicies, Ingress, HPA-Konfiguration |
| Abnahme | End-to-end-Funktions-, Restore-, Security-, Failover- und Lasttests |

Die numerischen Werte werden durch fachliche Messungen und konkrete
Betriebsprofile festgelegt, nicht ohne Nachweis als universelle Defaults erfunden.

## 8. Dokumentationsstatus

Diese Fortschreibung ist eine **fachlich beschlossene Dokumentationsversion**.
Aus dem Vorhandensein von Deploymentscaffolds, Go-Bootstrap-Probes, HTTP-Routen-
Entwuerfen und Markdown-Vertraegen folgt keine produktive Featureimplementierung.
Die bestehenden Tests und Sicherheitspruefungen muessen vor Releasefreigabe
wirklich implementiert, ausgefuehrt und nachgewiesen werden.
