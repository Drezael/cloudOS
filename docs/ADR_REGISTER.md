# cloudOS – Zentrales Architecture Decision Record (ADR) Register

> **Verbindlicher Architekturabgleich vom 15.11.2026:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md). Die dort ausdruecklich bestaetigten neueren Infrastrukturentscheidungen gehen widersprechenden aelteren Entwurfs- und Rollout-Formulierungen vor; dies ist kein Implementierungsnachweis.

> **Datei:** `docs/ADR_REGISTER.md`  
> **Dokumentversion:** 1.3 – Abschluss von 14 zusätzlichen ADRs (delegierte Projektwahl) vom 16.11.2026  
> **Stand:** 16.11.2026  
> **Repository:** [Drezael/cloudOS](https://github.com/Drezael/cloudOS), geprüfter `main`-Dokumentenbestand  
> **Status:** Verbindliches Organisations- und Entscheidungsregister; technische ADR-Statusangaben beschreiben **fachliche Architekturfestlegungen**, keine implementierte Software.  
> **Umfang:** 20 geprüfte Ausgangsdokumente, 121 ältere globale ADR-Kandidaten, 9 unbelegte reservierte Nummern, 41 ältere Fach-ADR-Aliase, 51 neu zugeordnete globale ADRs, 8 spaeter bestaetigte Infrastruktur-ADRs, 74 abgeglichene Entscheidungen aus `API_CONTRACTS.md`.  
> **Änderungshoheit:** Neue fachliche Entscheidungen und Änderungen akzeptierter Entscheidungen benötigen ausdrücklich die Zustimmung des Projektverantwortlichen; rein redaktionelle Updates nicht.

**Hinweis zur Nummernvergabe:** Von den 130 älteren numerischen IDs sind 121 belegt und `ADR-042` bis `ADR-050` nicht zugeordnet. Dieses Register erhält ihre Historie und führt 51 zusätzliche IDs (`ADR-131` bis `ADR-181`) aus eindeutig unterscheidbaren Fachthemen und den API-Entscheidungen ein. Die ID-Vergabe ist Dokumentationsorganisation, **keine neue technische Architekturentscheidung**. Alle 41 fachlichen Altkennungen bleiben über Aliaszeilen auffindbar; einige Aliase verweisen auf mehrere bereits existierende ADRs.

> **Beschlussabgleich vom 16.11.2026:** Dreizehn zuvor als `Proposed` geführte ADRs wurden vom Projektverantwortlichen ausdrücklich entschieden (ADR-002, -012, -017, -036, -059, -060, -131, -132, -136, -138, -140, -142, -150). Die vollständigen Entscheidungsnachweise sind in [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md) konsolidiert. Die verbleibenden `Proposed`-Themen werden durch diesen Nachtrag nicht automatisch freigegeben. ADR-094 bleibt ausdrücklich unverändert offen und wird in dieser Runde nicht erneut zur Entscheidung gestellt. `Accepted` bedeutet nicht implementiert.

> **Delegierter Beschlussnachtrag vom 16.11.2026:** Der Projektverantwortliche hat ausdrücklich autorisiert, alle noch offenen ADRs außer ADR-094 anhand der fachlich empfohlenen Option abzuschließen. 14 Architekturentscheidungen (ADR-013, -018, -020, -082, -103, -118, -143, -154 bis -160) sind in [ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md) begründet. **ADR-094 bleibt alleiniger offener globaler ADR; dies ist keine Implementierungs-/Lizenz-/Sicherheitsfreigabe.** Die technischen App-Data-Detailentscheidungen 1–17 sind separat in [app-framework/20-technical-decisions-E03-E04-2026-11-16.md](app-framework/20-technical-decisions-E03-E04-2026-11-16.md) dokumentiert.

## 1. Zweck, Geltungsbereich und normativer Rahmen

Dieses Dokument ist der zentrale Index aller bekannten cloudOS-Architekturentscheidungen. Es hält fest, **was entschieden ist**, warum dies gilt, welche Konsequenzen daraus folgen, welche Quelle maßgeblich ist und welche Details noch offen sind. Es ersetzt nicht die führenden Fachkonzepte und erstellt weder produktiven Quellcode noch API-/Kubernetes-/Datenbankartefakte.

Eine Architekturentscheidung kann `Accepted` sein, ohne bereits implementiert oder getestet zu sein. Umgekehrt kann ein vorhandener Beispielcode eine Architekturfrage **nicht** selbstständig zu `Accepted` machen. Die hier aufgeführten Status sind ein quellengestütztes konsolidiertes Bild, kein rückwirkend erfundenes Protokoll früherer Einzelgenehmigungen.

### 1.1 Verbindliche Regeln aus den vier Entscheidungen zur ADR-Organisation

1. **Stabile globale Kennungen:** Bereits vergebene globale ADR-IDs bleiben bestehen und werden nicht für andere Sachverhalte recycelt. `ADR-042` bis `ADR-050` bleiben unbesetzt; neue IDs beginnen ab `ADR-131`.
2. **Quellengestützte Übernahme:** Ausdrücklich bestätigte und eindeutig verbindlich formulierte Entscheidungen gelten als `Accepted`, sofern keine jüngere ausdrückliche Entscheidung widerspricht. Unverbindliche Alternativen und Parametervorschläge bleiben `Proposed`.
3. **Hybride Dokumentation:** Jeder ADR steht mit Entscheidung, knapper Begründung, Folgen und Quellen hier im Register. Nur bei entsprechendem fachlichen Nutzen wird später ein ergänzendes Detaildokument unter `docs/adr/` angelegt.
4. **Kontrollierte Änderungen:** Neue oder fachlich geänderte Entscheidungen erfordern ausdrückliche Bestätigung. Redaktionelle Korrekturen, Links und Quellennachträge dürfen den Inhalt nicht umdeuten. Fachliche Ersetzungen erfolgen nachvollziehbar mittels Nachfolge-ADR und `Superseded`-Kennzeichnung.

### 1.2 Statusmodell

| Status | Verbindliche Bedeutung | Zulässige Behandlung |
|---|---|---|
| `Accepted` | Fachentscheidung anhand ausdrücklicher Projektwahl oder eindeutig verbindlicher jüngerer Quelle belegt. | Nur redaktionell ohne neue Freigabe; fachliche Änderung über bestätigte Nachfolgeentscheidung. |
| `Proposed` | Thema oder Ansatz dokumentiert, aber konkrete Entscheidung noch nicht vollständig getroffen. | Präzisieren, vergleichen und gesondert zur Freigabe vorlegen. |
| `Rejected` | Ausdrücklich abgelehnte Option/ADR. | Historisch erhalten; keine aktive normative Wirkung. |
| `Superseded` | Früher akzeptierte Entscheidung wurde ausdrücklich ersetzt. | Bisherige ID, Datum und Nachfolger bleiben sichtbar. |
| `Reserved` | Technisch keine ADR-Entscheidung; Kennung ist historisch unbesetzt. | Nicht für andere Themen nutzen, solange der ältere Nummernraum stabilisiert wird. |

**Zusatzattribute, keine eigenen Status:** `Implementierung: offen / teilweise / umgesetzt / verifiziert`, `Nachweis: Gespräch / Fachkonzept / Testbericht` und `Prüfbedarf: ja / nein`. Implementierung und Architekturannahme dürfen nicht in derselben Statusspalte vermischt werden.

### 1.3 Entscheidungsinstanz und Änderungen

Der Projektverantwortliche ist fachliche Freigabeinstanz. Ein Konzept-Autor oder automatischer Dokumentenabgleich kann einen `Proposed`-Eintrag vorbereiten, aber **keine neue technische Festlegung eigenständig akzeptieren**. Fachliche Änderungen sind mit Grund, betroffener ID, Quellverweisen und erforderlichen Migrationen zu dokumentieren. Wesentliche Ablösungen erhalten eine neue ID; die alte wird als `Superseded` mit exakter Nachfolgerreferenz konserviert. Kritische Sicherheitsregeländerungen dürfen nicht durch bloße Redaktion versteckt werden.

### 1.4 Mindestmetadaten eines vollständigen Einzel-ADR

Ein späteres eigenes `docs/adr/ADR-NNN-*.md` enthält mindestens: ID, Titel, Status, Entscheidungsträger/Freigabe, Datum (nur wenn nachweisbar), Kontext, Entscheidung, verworfene Alternativen, Begründung, Folgen/Risiken, Sicherheitsaspekte, Migration/Kompatibilität, Nachweis-/Testplan, Quellendokumente sowie Vorgänger-/Nachfolger-ID. Das Register führt den wesentlichen Kern bereits kompakt. Leere oder unbekannte Datum-/Owner-Felder dürfen nicht erfunden werden.

## 2. Quellenhoheit und Widerspruchsbehandlung

**Fachlich maßgeblich ist die jüngste nachweislich bestätigte Entscheidung zum betroffenen Thema.** Wenn keine ausdrückliche neuere Entscheidung vorliegt, bleibt das jeweilige fachlich führende Dokument entscheidend. Eine ältere allgemeine Formulierung wird nicht als zweiter gültiger Lösungsweg parallelisiert. Ein echter, durch Quellen und Zeitfolge nicht auflösbarer Widerspruch wird mit beteiligten IDs gesondert als `Proposed/Prüfbedarf` geführt und vor Implementierungsbeginn geklärt.

| Reihenfolge | Auslegungsregel |
|---|---|
| 1 | Nachweisbare ausdrückliche bestätigte Architekturentscheidung; insbesondere die 74 Entscheidungen in `API_CONTRACTS.md` für deren Vertragsbereich. |
| 2 | Bereits akzeptierter zentraler ADR mit unverändertem fachlichen Geltungsbereich. |
| 3 | Jüngeres fachlich führendes Konzept für sein Spezialgebiet. |
| 4 | Ältere allgemeine Architektur-/Konzeptfassungen und ausdrücklich unverbindliche Entwürfe. |

Eine jüngere **ungenehmigte** Textänderung allein darf einen `Accepted`-ADR nicht ablösen. Die obige Reihenfolge setzt eine fachlich bestätigte Quelle voraus.

### 2.1 geprüfte Quelldokumente

| Fachdatei | Zuständigkeit |
|---|---|
| [CONCEPT.md](CONCEPT.md) | Gesamtidee und Zielbild |
| [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md) | Core-Architektur, Ownership und ältere globale ADRs |
| [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md) | Combined-/Split-/K8s-Architektur und ADR-083–098 |
| [APP_API_ACCESS.md](APP_API_ACCESS.md) | Signierte App-API-Operationen und ADR-037–041 |
| [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md) | `.cloudApp`-Laufzeiten und ADR-025–036 |
| [APP_SIGNING.md](APP_SIGNING.md) | Zertifikate, Signaturen und Trust |
| [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md) | Zwei-Volume-VFS und VFS-Fach-ADRs |
| [SHARING_SPACES.md](SHARING_SPACES.md) | Interne Freigaben/Spaces und ADR-COL |
| [PUSH_REALTIME.md](PUSH_REALTIME.md) | SSE/Outbox/Replay und ADR-073–082 |
| [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md) | IAM, lokale Accounts und IAM-ADRs |
| [SETTINGS_SYSTEM.md](SETTINGS_SYSTEM.md) | Settings-Host und geschützte MVP-Sub-Apps (ADR-142) |
| [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md) | Passkey/SSH/Step-up und ADR-099–103 |
| [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md) | Diagnostik/Audit/Logs und ADR-104–108 |
| [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md) | Window Host und ADR-109–113 |
| [CLIENT_DEVICES.md](CLIENT_DEVICES.md) | Device Broker und ADR-114–118 |
| [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md) | Desktop/Taskbar und ADR-051–060 |
| [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md) | AppStore/Zuweisungen und ADR-061–072 |
| [THEMING.md](THEMING.md) | Themes und ADR-119–124 |
| [WALLPAPER_ENGINE.md](WALLPAPER_ENGINE.md) | Wallpaper und ADR-125–130 |
| [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md) | LDAP/OIDC-Konnektoren und ADR-ID |
| [API_CONTRACTS.md](API_CONTRACTS.md) | 74 verbindliche API-/Sync-/Storage-Entscheidungen |
| [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md) | Ausdrückliche Freigaben der 13 ADRs vom 16.11.2026 |

### 2.2 Fachgrenzen bleiben erhalten

- **`SYSTEM_ARCHITECTURE.md` / `CORE_DEPLOYMENT_SCALING.md`:** Servicegrenzen, Deployment, Replicas, Infrastruktur.
- **`APP_API_ACCESS.md` / `APP_SIGNING.md` / `APPLICATION_SYSTEM.md`:** App-Operationen, Signaturen, Paket-Lifecycle.
- **`PUSH_REALTIME.md`:** SSE, Replay, Audience-/Subscription-Sicherheit; `API_CONTRACTS.md` ergänzt verbindliche Formate und Failure-Semantik.
- **`VIRTUAL_FILESYSTEM.md` / `SHARING_SPACES.md`:** Sichtbarkeit, Dateirechte, Sharing und Space-Lifecycle; `API_CONTRACTS.md` konkretisiert Block-/Sync-/Key-Verträge.
- **`AUTHENTICATION_AUTHORIZATION.md` / `AUTHENTICATION_METHODS.md` / `IDENTITY_INTEGRATION.md`:** Identitäten, Loginmethoden und optionale Connectoren.
- **`LOGGING_OBSERVABILITY.md`:** Logging, Pflicht-Audit und Logs-App; `API_CONTRACTS.md` ergänzt Distributed Tracing, Rate- und Qualitätsverträge.
- **`DESKTOP_ENVIRONMENT.md` / `WINDOW_FRAMEWORK.md` / `THEMING.md` / `WALLPAPER_ENGINE.md`:** sichtbare UI, Window Host und Darstellung; `API_CONTRACTS.md` aktualisiert verbindlich Mehrgeräte-/Offline-Verhalten.

## 3. Inventar und Nummernpolitik

| Kennzahl | Bestand |
|---|---:|
| Ältere globale ADR-Kandidaten (`001–041`, `051–130`) | 121 |
| Historische Lücke (`042–050`), reserviert | 9 |
| Zusätzlich eindeutig konsolidierte Themen (`131–181`) | 51 |
| Neue am 15.11.2026 entschiedene Infrastruktur-ADRs (`182-189`) | 8 |
| **Erfasste globale Architektur-ADRs** | **180** |
| Davon `Accepted` (fachlich) | 179 |
| Davon `Proposed` | 1 |
| Davon `Superseded` als einzelne ADR-ID | 0 |
| Beibehaltende Legacy-Fachaliase | 41 |
| Aus `API_CONTRACTS.md` verlinkte bestätigte Fragen | 74 |

Die globale ID ist ein stabiler Identifikator, keine Priorität und kein Ausführungsdatum. **Die Lücke 042–050 wird nicht stillschweigend aufgefüllt.** Es werden keine freien IDs für neue Themen vergeben, wenn das Thema bereits ein bestehendes, inhaltlich passendes ADR besitzt.

## 4. Kompakter Statusindex

Dieser Index dient zum Auffinden; der vollständige begründete Datensatz folgt in Kapitel 5.

| ADR | Status | Gegenstand |
|---|---|---|
| `001` | `Accepted` | Core-Service-Ownership und logische Dienstgrenzen |
| `002` | `Accepted` | Vue 3/Vite und Desktop-Host-Rendering |
| `003` | `Accepted` | Lokale Identität, BFF-Sitzungen und optionale externe IdPs |
| `004` | `Accepted` | Dienstidentität und interne Ende-zu-Ende-Autorisierung |
| `005` | `Accepted` | PostgreSQL-Schema-Ownership und Migrationen |
| `006` | `Accepted` | Formale API-Verträge und semantische Versionierung |
| `007` | `Accepted` | JetStream, transaktionale Outbox und Event-Zustellung |
| `008` | `Accepted` | Datei-Blob-Speicher, Crash-Recovery und Backup-Konsistenz |
| `009` | `Accepted` | VFS-IDs, Zugriffsgrenzen und File-Capabilities |
| `010` | `Accepted` | Desktop- und Mehrgeräte-Synchronisierung |
| `011` | `Accepted` | App-Origin-Trennung und Browser-Sandbox |
| `012` | `Accepted` | Backup-RPO/RTO und Browser-/Leistungsbudgets |
| `013` | `Accepted` | Externe API-Clients und OAuth-/Grant-Modell |
| `014` | `Accepted` | App-Runtime-Profile |
| `015` | `Accepted` | App-Paketvertrauen, Manifest und Update-Modell |
| `016` | `Accepted` | Capability Broker, Dateiauswahl und Consent |
| `017` | `Accepted` | Rust-Web-Toolchain und GPU-Kompatibilität |
| `018` | `Accepted` | Wasmtime/WASI-Komponenten und Host-Funktionen |
| `019` | `Accepted` | Isolierte Worker und Native-Runtimes |
| `020` | `Accepted` | ArtCraft als Integrations-PoC |
| `021` | `Accepted` | App-zu-App-Kommunikation und Message-Bridge |
| `022` | `Accepted` | Service Discovery, Skalierung und Runtime-Leases |
| `023` | `Accepted` | Designsystem und Theme-Resolver |
| `024` | `Accepted` | Theme-Pakete und Theme-Sicherheitsmodell |
| `025` | `Accepted` | App-first-Oberflächen und Core-Service-Grenze |
| `026` | `Accepted` | `.cloudApp`-Paketstruktur und signierte Metadaten |
| `027` | `Accepted` | X.509-Publisher-Vertrauen und signierter Paketindex |
| `028` | `Accepted` | Geschützte Core-App-Baseline und Recovery |
| `029` | `Accepted` | Administrative App-Installation über Dateimanager |
| `030` | `Accepted` | Installation, Zuweisung, Grant und Launch trennen |
| `031` | `Accepted` | Sandbox, App-Bridge und authentifizierter App-Host |
| `032` | `Accepted` | Unveränderliche App-Versionen und Rollback |
| `033` | `Accepted` | Core-App-Theming und vertrauenswürdige Systemdialoge |
| `034` | `Accepted` | WASM-/WASI-/Native-Paketvarianten |
| `035` | `Accepted` | Selbstsignierte App-Pakete und Risikoakzeptanz |
| `036` | `Accepted` | Signaturalgorithmen und kanonische Serialisierung |
| `037` | `Accepted` | Signierte App-API-Allowlist und zentraler Operationenkatalog |
| `038` | `Accepted` | Enforcement an allen App- und Servicegrenzen |
| `039` | `Accepted` | API-Risikoklassen und verpflichtender Installationsdialog |
| `040` | `Accepted` | Revisionsgebundene App-Grants und sofortiger Einzelwiderruf |
| `041` | `Accepted` | Ausgehendes App-Netzwerk ausschließlich über Network Broker |
| `051` | `Accepted` | `/.desktop/` als verbindliche Desktop-Dateiquelle |
| `052` | `Accepted` | Taskbar-Konfiguration `/.taskbar.json` |
| `053` | `Accepted` | Desktop-Layout `/.desktop-layout.json` |
| `054` | `Accepted` | `.cloudlink`-Verknüpfungen und sichere Targets |
| `055` | `Accepted` | Directory-Realtime und sichere SSE-Deltas |
| `056` | `Accepted` | Taskbar an vier Bildschirmkanten |
| `057` | `Accepted` | Signierte Dateisystem-Operationen von Core-Apps |
| `058` | `Accepted` | Taskbar-Einstellungen als eigener geschützter App-Kontext |
| `059` | `Accepted` | Migration alter Desktop-Home-Verzeichnisse |
| `060` | `Accepted` | Desktop-Performance, Browser und Barrierefreiheit |
| `061` | `Accepted` | `system:/apps/` als kanonischer App-Paketspeicher |
| `062` | `Accepted` | `Apps.cloudApp` als reguläre Installationsoberfläche |
| `063` | `Accepted` | Globale App-Installation und Admin-Default |
| `064` | `Accepted` | Immutable App-Registry, Rollback und Package-GC |
| `065` | `Accepted` | Moderierter GitHub-App-Katalog |
| `066` | `Accepted` | Mehrere App-Katalogquellen und Publisher-Vertrauen |
| `067` | `Accepted` | Store-Provenienz und Store-verwaltete Installationen |
| `068` | `Accepted` | Sofortige App-Zuweisungsinvalidierung |
| `069` | `Accepted` | App-Zuweisungen getrennt von POSIX-Gruppenrechten |
| `070` | `Accepted` | Deinstallation und Löschung persönlicher App-Daten trennen |
| `071` | `Accepted` | App-Asset-Delivery und Browsercache nach Widerruf |
| `072` | `Accepted` | Lokales App-Management, Store-Sync und Egress |
| `073` | `Accepted` | SSE als verbindlicher Browser-Push-Transport |
| `074` | `Accepted` | Outbox, JetStream, Delivery-Log und Stream-Ordering |
| `075` | `Accepted` | Snapshot-/Watermark-/Barrier-Bootstrap |
| `076` | `Accepted` | Kein Browser-Polling und sicheres Replay |
| `077` | `Accepted` | Direkt anwendbare, berechtigungsgefilterte Events |
| `078` | `Accepted` | Signierte Event-Subscriptions pro App |
| `079` | `Accepted` | Persistente Benutzerbenachrichtigungen |
| `080` | `Accepted` | SSE-Auth, Origin-Bindung und Widerruf |
| `081` | `Accepted` | SSE-Proxy, Multi-Tab und Backpressure |
| `082` | `Accepted` | Web Push bei geschlossenem Browser |
| `083` | `Accepted` | Combined-Core als Standard, Split/Kubernetes opt-in |
| `084` | `Accepted` | Gemeinsamer Servicekatalog und identischer Go-Code |
| `085` | `Accepted` | Lokale Adapter versus gRPC-Ports |
| `086` | `Accepted` | Discovery und Retry-/Deadline-Budgets |
| `087` | `Accepted` | Shared State, Owner-Schemas und Revisionen |
| `088` | `Accepted` | Dauerhafte Worker-Leases und Idempotenz |
| `089` | `Accepted` | SSE-Fanout, Replay und Rolling-Drain |
| `090` | `Accepted` | HPA, Autoscaling und Kapazitätsmetriken |
| `091` | `Accepted` | Workload-Identitäten, mTLS und interne API-Grenzen |
| `092` | `Accepted` | Gemeinsamer Storage und immutable Paketversionen |
| `093` | `Accepted` | Datenbankmigrationen bei Version-Skew |
| `094` | `Proposed` | Kubernetes-Produktionshärtung und Secrets |
| `095` | `Accepted` | Combined-to-Split-Migration ohne doppelte Owner |
| `096` | `Accepted` | Startup, Liveness, Readiness und Funktionsgesundheit |
| `097` | `Accepted` | Service-SLOs, Observability und Lasttests |
| `098` | `Accepted` | Helm/Kustomize/Operator-Werkzeuge |
| `099` | `Accepted` | Credential-Lifecycle und Aliasmodell |
| `100` | `Accepted` | WebAuthn-/Passkey-Relying-Party-Policy |
| `101` | `Accepted` | SSH-Agent-Signaturen und SSH-Key-Anmeldung |
| `102` | `Accepted` | Assurance-Level, Step-up und Admin-Recovery |
| `103` | `Accepted` | Externe Identity-Verknüpfung zu lokalen Credentials |
| `104` | `Accepted` | Strukturierte Logs, OpenTelemetry und Logger-Hierarchie |
| `105` | `Accepted` | Verteilung der Logging-Policy und sicherer Fallback |
| `106` | `Accepted` | Unabschaltbares Security-Audit |
| `107` | `Accepted` | Log-Aufbewahrung, Datenschutz und Export |
| `108` | `Accepted` | `Logs.cloudApp` und Log-Berechtigungen |
| `109` | `Accepted` | Verbindlicher Window Host für alle sichtbaren Apps |
| `110` | `Accepted` | Fensterzustände, Taskbar und Close-Semantik |
| `111` | `Accepted` | Runtime-Bridge und sichere Modal-Dialoge |
| `112` | `Accepted` | Workspace-/Mehrtab-Zustand und Replikation |
| `113` | `Accepted` | Theme- und Accessibility-Vertrag des Window Hosts |
| `114` | `Accepted` | Client Device Broker statt direkter Geräte-APIs |
| `115` | `Accepted` | Device-Capabilities und Installations-Risikoanzeigen |
| `116` | `Accepted` | Browser-Permission, User Activation und Grant-TTL |
| `117` | `Accepted` | USB-Dateiimport und VFS-Isolation |
| `118` | `Accepted` | Browserkompatibilität der Geräteprofile |
| `119` | `Accepted` | Sechs integrierte Theme-Presets |
| `120` | `Accepted` | Benutzer-Overrides für Theme und Wallpaper |
| `121` | `Accepted` | Theme Service und kontrollierte Browser-Tokens |
| `122` | `Accepted` | Theme-Asset-Sandbox und geprüfte Paketfreigabe |
| `123` | `Accepted` | Barrierefreiheit und geschützte Systemdialoge |
| `124` | `Accepted` | Theme-Livewechsel über SSE |
| `125` | `Accepted` | Wallpaper-Rendering ausschließlich im Browser |
| `126` | `Accepted` | Sechs eigenentwickelte animierte Wallpaper |
| `127` | `Accepted` | Wallpaper-Auswahl unabhängig vom Theme |
| `128` | `Accepted` | Worker-/OffscreenCanvas-Strategie mit Fallback |
| `129` | `Accepted` | Wallpaper-Leistungsbudget und Reduced Motion |
| `130` | `Accepted` | Wallpaper-Änderungen per SSE |
| `131` | `Accepted` | VFS-Namen, Unicode-Normalisierung und Case-Policy |
| `132` | `Accepted` | Standardstruktur neu angelegter Home-Verzeichnisse |
| `133` | `Accepted` | Idempotente Home-Bereitstellung und Konto-Lifecycle |
| `134` | `Accepted` | Cross-Volume-Kopie und kontrollierter Import/Export |
| `135` | `Accepted` | VFS-Quotas, Versionen und Papierkorb |
| `136` | `Accepted` | VFS-Backup-Konsistenz und selektives Restore |
| `137` | `Accepted` | Lokaler LDAPv3-Protokollumfang und TLS |
| `138` | `Accepted` | UID-/GID-Nummernkreise und Wiederverwendung |
| `139` | `Accepted` | Kanonisches Benutzerkonto- und Gruppenschema |
| `140` | `Accepted` | Passwort-Hashing, Passwortpolitik und Recovery |
| `141` | `Accepted` | Systemrollen getrennt von POSIX-Gruppen |
| `142` | `Accepted` | Settings-Sub-Apps und eigener Berechtigungskontext |
| `143` | `Accepted` | Erweiterte POSIX-ACLs und Named Grants |
| `144` | `Accepted` | Interne Share-Rollen und Revocation |
| `145` | `Accepted` | Share-Mounts im Benutzer-Home |
| `146` | `Accepted` | Space-Rollen und Meta-Administration |
| `147` | `Accepted` | Space-Root auf dem `users`-Volume |
| `148` | `Accepted` | POSIX-DAC und zusätzliche Share-/Space-Grants |
| `149` | `Accepted` | Gruppenvererbung und unmittelbarer Membership-Entzug |
| `150` | `Accepted` | Share-Expiry, Ablehnung und Weiterfreigabe |
| `151` | `Accepted` | Space-Lifecycle, Archivierung, Papierkorb und Quotas |
| `152` | `Accepted` | Cross-Scope-Move und Space-Konflikte |
| `153` | `Accepted` | Externe Identity-Provenienz und Attributautorität |
| `154` | `Accepted` | LDAP-Sync-Intervalle und Sicherheitsgrenzen bei Massenänderungen |
| `155` | `Accepted` | Missing User, Deaktivierung und Stale-Identity-Richtlinie |
| `156` | `Accepted` | Import externer UID/GID oder interne Nummernvergabe |
| `157` | `Accepted` | Externe Gruppenverschachtelung und lokale Overlays |
| `158` | `Accepted` | LDAP-Bind-Login als optionaler Auth-Adapter |
| `159` | `Accepted` | Integrationsvertrag externer OIDC-Provider |
| `160` | `Accepted` | Secret-Store und Zertifikatsverwaltung externer Connectoren |
| `161` | `Accepted` | Nur eine aktive API-Major und kontrollierte Kompatibilität |
| `162` | `Accepted` | Offizielles SDK, Capability-Filter und Entwicklerdokumentation |
| `163` | `Accepted` | Ressourcenbezogenes Benutzer-Consent |
| `164` | `Accepted` | Hierarchisches, dynamisches API-Rate-Limiting |
| `165` | `Accepted` | Persistente Sagas und kontrollierter Fehlerzustand |
| `166` | `Accepted` | Distributed Tracing über Core, Events und Jobs |
| `167` | `Accepted` | Umfassende Core-Metriken und Monitoring |
| `168` | `Accepted` | Abgestufte automatische Selbstheilung |
| `169` | `Accepted` | Konfigurierbare Alarmierung, Eskalation und Wartungsfenster |
| `170` | `Accepted` | Rollenbasierte `Status.cloudApp` |
| `171` | `Accepted` | Workspace-Recovery, Auto-Save und Checkpoint-Historie |
| `172` | `Accepted` | Zentrale Sitzungs-/Geräteverwaltung und Remote-Revocation |
| `173` | `Accepted` | Offline-First-Apps und sichere Journalsynchronisierung |
| `174` | `Accepted` | Gerätegebundene Offline-Verschlüsselung und Quotas |
| `175` | `Accepted` | Adaptive Transfers, resumable Chunks und Delta-Sync |
| `176` | `Accepted` | Permanenter, versionierter und domänengetrennter Blockspeicher |
| `177` | `Accepted` | Envelope Encryption, risikobasierte Rotation und Schlüssel-Recovery |
| `178` | `Accepted` | Block-Integritätsprüfung und kontrollierte Selbstheilung |
| `179` | `Accepted` | Transaktionssichere Block-GC und Papierkorb |
| `180` | `Accepted` | Verpflichtende API-Qualitätsgates und Releasefreigabe |
| `181` | `Accepted` | Durable Jobs, `Tasks.cloudApp` und auftragssichere Wiederaufnahme |
| `182` | `Accepted` | Bedarfsgesteuerte Core-Auslagerung |
| `183` | `Accepted` | Extern verwaltete PostgreSQL-/JetStream-Hochverfuegbarkeit |
| `184` | `Accepted` | Fachlich getrennte JetStream-Streams und Consumer |
| `185` | `Accepted` | Event-Retention und nachweisbare Gap-Recovery |
| `186` | `Accepted` | Outbox-Publish-Ack, Retention und Reconciliation |
| `187` | `Accepted` | Kontrollierte Offline-Migration von Storage-Backends |
| `188` | `Accepted` | Providerunabhaengige Runtime-Secrets und Rotation |
| `189` | `Accepted` | Providerunabhaengige Edge-, Ingress- und TLS-Architektur |

**Historisch unbesetzt:** `ADR-042`, `ADR-043`, `ADR-044`, `ADR-045`, `ADR-046`, `ADR-047`, `ADR-048`, `ADR-049`, `ADR-050` (`Reserved`, keine zugewiesenen Entscheidungen).

## 5. Vollständiges begründetes ADR-Register

**Darstellungsregel:** Der `Accepted`-Status bezieht sich auf den genannten **fachlichen Kern**; am Ende einer Begründung ausdrücklich offene Parameter und technische Umsetzungsdetails werden dadurch nicht zusätzlich akzeptiert. Quellenlinks zeigen auf die tatsächlichen bestehenden Konzeptdateien im Repository.

### 5.1 Core, Systemarchitektur, App- und API-Sicherheit (ADR-001–041)

#### ADR-001 – Core-Service-Ownership und logische Dienstgrenzen

**Status:** `Accepted`  
**Entscheidung / Stand:** Der Go-Core ist aus fachlich getrennten Owner-Diensten mit klaren Verträgen aufgebaut; `combined` ist ein zulässiger gemeinsamer Ausführungsprozess.  
**Begründung und Folgen:** Unabhängige Datenhoheit und sichere spätere Trennung ohne zweites Fachmodell.  
**Quellen:** [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-002 – Vue 3/Vite und Desktop-Host-Rendering

**Status:** `Accepted`  
**Entscheidung / Stand:** Vue 3 mit TypeScript ist für Desktop, Window Host, App Host und cloudOS-eigene Oberflächen verbindlich; Vite ist das bevorzugte Build-Werkzeug. Das öffentliche App-SDK und der signierte App-Vertrag bleiben frameworkneutral. Drittanbieter können etwa React, Svelte oder Rust/WASM verwenden.  
**Begründung und Folgen:** Die Frameworkwahl betrifft nur eigene Frontend-Implementierungen und darf Drittanbieter nicht an Vue binden. Browser- und Window-Host-Sicherheitsgrenzen bleiben unabhängig davon verpflichtend.  
**Quellen:** [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-003 – Lokale Identität, BFF-Sitzungen und optionale externe IdPs

**Status:** `Accepted`  
**Entscheidung / Stand:** Lokale Go-Core-Authentifizierung ist Standard; BFF-Sitzungen sind geschützt. Externe OIDC-/LDAP-Anbindungen sind optionale Integrationen, keine zwingende MVP-Voraussetzung.  
**Begründung und Folgen:** Ersetzt die frühere Annahme eines verpflichtenden externen OIDC-Providers.  
**Quellen:** [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md), [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-004 – Dienstidentität und interne Ende-zu-Ende-Autorisierung

**Status:** `Accepted`  
**Entscheidung / Stand:** Getrennte Service-Identitäten, im verteilten Betrieb mTLS sowie kurzlebige Service-Capabilities und eingegrenzte Delegationen sind vorgeschrieben.  
**Begründung und Folgen:** Netzwerkstandort und Pod-IP gelten niemals als Identitäts- oder Berechtigungsnachweis.  
**Quellen:** [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [API_CONTRACTS.md](API_CONTRACTS.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-005 – PostgreSQL-Schema-Ownership und Migrationen

**Status:** `Accepted`  
**Entscheidung / Stand:** PostgreSQL speichert autoritative Metadaten mit eindeutiger fachlicher Owner-Zuordnung; Services ändern fremde Daten nicht unmittelbar.  
**Begründung und Folgen:** Migrations-/Versions-Skew-Details müssen vor verteilten Rollouts präzisiert werden.  
**Quellen:** [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-006 – Formale API-Verträge und semantische Versionierung

**Status:** `Accepted`  
**Entscheidung / Stand:** REST wird mit OpenAPI, gRPC mit Protobuf, Events/SSE mit AsyncAPI oder versionierten Schemata spezifiziert; die aktuelle Major-API ist maßgeblich.  
**Begründung und Folgen:** Vertragsprüfung, SDK-Generierung und Sicherheits-Gates sind verpflichtend; ältere Major-APIs laufen nicht parallel.  
**Quellen:** [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-007 – JetStream, transaktionale Outbox und Event-Zustellung

**Status:** `Accepted`  
**Entscheidung / Stand:** NATS JetStream liefert dauerhafte asynchrone Ereignisse; Outbox und idempotente Konsumenten sichern mindestens einmalige Zustellung.  
**Begründung und Folgen:** Ereignisse erfordern per-Ressource-Ordering, Versionierung und korrektes Replay.  
**Quellen:** [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [PUSH_REALTIME.md](PUSH_REALTIME.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-008 – Datei-Blob-Speicher, Crash-Recovery und Backup-Konsistenz

**Status:** `Accepted`  
**Entscheidung / Stand:** Der File Service verantwortet Datei-Manifeste und permanente unveränderliche Datenblöcke; Veröffentlichungen erfolgen atomar und überprüfbar.  
**Begründung und Folgen:** Ein konsistenter Backup-/Restore-Mechanismus muss Metadaten, Bytes und Schlüssel gemeinsam abdecken.  
**Quellen:** [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [API_CONTRACTS.md](API_CONTRACTS.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

#### ADR-009 – VFS-IDs, Zugriffsgrenzen und File-Capabilities

**Status:** `Accepted`  
**Entscheidung / Stand:** Es existieren exakt `system` und `users`; `/` zeigt auf das eigene Home. Dateioperationen erzwingen ACL/POSIX-Regeln und instanzgebundene Handles.  
**Begründung und Folgen:** Fremde Homes und Systemdaten sind nicht per geratenem Pfad oder App-Token zugänglich.  
**Quellen:** [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

#### ADR-010 – Desktop- und Mehrgeräte-Synchronisierung

**Status:** `Accepted`  
**Entscheidung / Stand:** Eine logische Desktop-Umgebung wird geräteübergreifend synchronisiert; für geeignete Datenmodelle sind CRDT/OT-gestützte Zusammenführungen vorgesehen.  
**Begründung und Folgen:** Fensterdarstellung bleibt bildschirmspezifisch; nicht zusammenführbare Änderungen dürfen nicht unbemerkt überschrieben werden.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-011 – App-Origin-Trennung und Browser-Sandbox

**Status:** `Accepted`  
**Entscheidung / Stand:** Jede `.cloudApp` läuft isoliert; Cross-Origin-/CSP-/Bridge-Grenzen verhindern direkten Zugriff auf privilegierte Core-Funktionen.  
**Begründung und Folgen:** Exakte Browser-Sicherheits- und Kompatibilitätsprofile sind vor Laufzeitfreigabe zu testen.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [APP_API_ACCESS.md](APP_API_ACCESS.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-012 – Backup-RPO/RTO und Browser-/Leistungsbudgets

**Status:** `Accepted`  
**Entscheidung / Stand:** Für den MVP werden messbare RPO-/RTO-, Browser- und Leistungsziele sowie eine reproduzierbare Referenzumgebung verbindlich definiert, gemessen, dokumentiert und regelmäßig geprüft. Konkrete Grenzwerte werden erst nach Messung und ausdrücklicher Freigabe als zugesagt behandelt. Eine Hochverfügbarkeitsgarantie ist nicht Bestandteil des MVP.  
**Begründung und Folgen:** Recovery- und Performancebudgets sind Release-Abnahmekriterien, aber es werden keine ungemessenen SLA-Zahlen oder eine Pflicht zu Cluster-/HA-Betrieb vorweggenommen. Browserprofile werden mit ADR-060 abgestimmt.  
**Quellen:** [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [CONCEPT.md](CONCEPT.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-013 – Externe API-Clients und OAuth-/Grant-Modell

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Externe eigenständige API-Clients bleiben im MVP deaktiviert. Für eine spätere gesondert freigegebene Erweiterung ist ein registriertes, providerunabhängiges OAuth-2.0-Sicherheitsprofil nach aktueller Security BCP mit kurzlebigen, audience-, client-, scope- und gegebenenfalls benutzergebundenen Tokens verbindlich. Interaktive Clients verwenden Authorization Code mit PKCE; Maschinenzugriffe benötigen eine eigenständig genehmigte, eng begrenzte Client-/Workload-Identität. Keine allgemeinen statischen Master-API-Keys.  
**Begründung und Folgen:** Externe Zugänge dürfen weder die signierte App-API-Allowlist noch BFF-Sessions, Resource-Scope oder Tenant-Isolation umgehen. Ein API-Contract allein autorisiert keine externe Veröffentlichung.  
**Freigabe-/Nachweisgrenze:** Vor externer Aktivierung: Client-Registrierung, Consent/Delegation, Token-Rotation und Widerruf, Replay-/SSRF-/Rate-Limit-Tests, AuthZ-Audit und ausdrückliche Betriebsfreigabe.  
**Einführungsprofil:** Nach MVP, standardmäßig aus; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [APP_API_ACCESS.md](APP_API_ACCESS.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-013).

#### ADR-014 – App-Runtime-Profile

**Status:** `Accepted`  
**Entscheidung / Stand:** Browser-WASM ist der primäre GUI-Pfad; WASI-/Native-/Remote-Profile sind nur als getrennt abgesicherte Erweiterungen zulässig.  
**Begründung und Folgen:** Nicht vertrauenswürdige Runtimes dürfen nie im privilegierten Go-Core-Prozess laufen.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

#### ADR-015 – App-Paketvertrauen, Manifest und Update-Modell

**Status:** `Accepted`  
**Entscheidung / Stand:** Jede installierbare App ist ein signiertes `.cloudApp`-Paket mit unveränderlichen Versionen und geprüfter Herausgeberidentität.  
**Begründung und Folgen:** Installation, Aktualisierung und Rollback dürfen die signierten Berechtigungen nicht umgehen.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [APP_SIGNING.md](APP_SIGNING.md), [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md).

#### ADR-016 – Capability Broker, Dateiauswahl und Consent

**Status:** `Accepted`  
**Entscheidung / Stand:** Privilegierte Aktionen laufen durch den vertrauenswürdigen Broker; App-Instanz, Benutzerrecht, Ressourcenscope und Nutzereinwilligung wirken kumulativ.  
**Begründung und Folgen:** Dateihandles sind kein Ersatz für eine deklarierte und genehmigte Core-API-Operation.  
**Quellen:** [APP_API_ACCESS.md](APP_API_ACCESS.md), [CLIENT_DEVICES.md](CLIENT_DEVICES.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-017 – Rust-Web-Toolchain und GPU-Kompatibilität

**Status:** `Accepted`  
**Entscheidung / Stand:** Browserfähige Rust-Apps liefern ein signiertes `web-wasm`-Paket mit `wasm32-unknown-unknown`-Artefakt, JS-Glue und Assets. Rust Stable/Cargo/wasm-bindgen/Vite ist die Referenztoolchain; weitere nachweislich konforme Toolchains bleiben zulässig. Für das Rust/WASM-MVP-Profil sind WebGPU und WebAssembly-Multithreading Pflicht, ohne automatischen WebGL-/Single-Thread-Fallback. Strikte Cross-Origin-Isolation mit COEP `require-corp`, abgestimmten COOP/CORP/CORS/CSP/Permissions-Policies und isolierter App-Origin wird vorausgesetzt.  
**Begründung und Folgen:** Fehlende WebGPU-/Thread-Fähigkeiten oder unvollständige Isolation sperren den Start dieses Runtime-Profils mit klarer Inkompatibilitätsmeldung. Reproduzierbare Builds und Browser-Integrationstests sind Teil der Freigabe; die öffentliche API bleibt toolchainneutral.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-018 – Wasmtime/WASI-Komponenten und Host-Funktionen

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Serverseitige WASI-Komponenten bleiben im MVP deaktiviert. Für die spätere Erweiterung werden Wasmtime, das WASI Component Model und versionierte, freigegebene WIT-Contracts eingesetzt; Komponenten laufen ausschließlich in einem vom Go-Core getrennten, unprivilegierten Runner. Standardmäßig kein Host-Filesystem, Netzwerk, Environment/Secrets, Docker-Socket oder Datenbankzugang. Jeder Host-Call erfordert einen expliziten instanz-/jobgebundenen Grant.  
**Begründung und Folgen:** Ausführung von nicht vertrauenswürdigem WASM ist keine Autorisierung und darf den Go-Core nicht kompromittieren.  
**Freigabe-/Nachweisgrenze:** Vor Runtime-Freigabe: gepinnte, getestete Wasmtime-/WASI-/WIT-Versionen; Fuel/Epoch, RAM, Laufzeit, Ausgabe, Parallelität, Egress, Abbruch und Crash-Isolation; Negativtests. Keine Produktionsbehauptung allein aus der Architektur.  
**Einführungsprofil:** Nach MVP, standardmäßig aus; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-018).

#### ADR-019 – Isolierte Worker und Native-Runtimes

**Status:** `Accepted`  
**Entscheidung / Stand:** Server-WASI- und Native-App-Ausführung sind von privilegierten Core-Diensten getrennt und nur über minimal berechtigte Broker zulässig.  
**Begründung und Folgen:** Konkrete Sandbox-Technologie und Betriebsgrenzen bleiben Implementierungs-/Security-Nacharbeiten.  
**Quellen:** [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-020 – ArtCraft als Integrations-PoC

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** ArtCraft ist keine Pflichtabhängigkeit des cloudOS-MVP. Als optionale, unverändert bezogene Rust-Browser-WASM-PoC-Referenz wird PhotoCraft v0.23.1 (Upstream-Tag v0.23.1; Stand 16.11.2026) ausgewählt. Zunächst nur Integrationsstufe A: lokal gehosteter, isolierter App-Host-Start mit Browser-/WebGPU-/Thread-/Origin-Konformität. Stufe B (File Bridge) erst nach eigenem separatem Gate; Stufe C ist kein Teil dieses Beschlusses. Für den Quellcode kann nach Prüfung die Apache-2.0-Lizenzoption gewählt werden; Dritt-Assets und Marken bleiben gesondert geschützt.  
**Begründung und Folgen:** Ein reproduzierbarer Fremd-App-Test ist nützlich, darf aber weder Build- noch Rechts- oder Produktzwänge für den MVP erzeugen.  
**Freigabe-/Nachweisgrenze:** Vor Ausführung/Distribution: Upstream-Tag, heruntergeladenes Artefakt und SHA-256, Source-/NOTICE-/Asset-/Trademark-Lizenzen, CSP/COEP/CORP, App-Signatur, Browser-Ressourcen und Isolation prüfen. Kein automatisches Repaketieren/Branding.  
**Einführungsprofil:** Optionaler, nicht blockierender PoC nach Browser-Baseline; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [PhotoCraft Releases](https://github.com/storytold/photocraft/releases), [PhotoCraft README](https://github.com/storytold/photocraft#license-and-credits); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-020).

#### ADR-021 – App-zu-App-Kommunikation und Message-Bridge

**Status:** `Accepted`  
**Entscheidung / Stand:** App-übergreifende Kommunikation darf nur über geprüfte, instanzgebundene Core-Broker-Verträge erfolgen.  
**Begründung und Folgen:** Ein allgemeiner unkontrollierter Nachrichtenaustausch zwischen Sandboxes ist verboten; konkrete Datenformate bleiben Teil des SDK-Konzepts.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

#### ADR-022 – Service Discovery, Skalierung und Runtime-Leases

**Status:** `Accepted`  
**Entscheidung / Stand:** Getrennte Core-Betriebsarten nutzen Service Discovery und langlebige Leases auf gemeinsamem Zustand; `combined` bleibt Standard.  
**Begründung und Folgen:** Konkrete Skalierungsschwellen und erste Split-Kandidaten werden separat präzisiert.  
**Quellen:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

#### ADR-023 – Designsystem und Theme-Resolver

**Status:** `Accepted`  
**Entscheidung / Stand:** Zentrale semantische Tokens, UI-Komponenten und ein kontrollierter Theme-Resolver gelten für Desktop und Apps.  
**Begründung und Folgen:** Accessibility und Laufzeitwechsel dürfen Sicherheitsdialoge nicht manipulieren.  
**Quellen:** [THEMING.md](THEMING.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

#### ADR-024 – Theme-Pakete und Theme-Sicherheitsmodell

**Status:** `Accepted`  
**Entscheidung / Stand:** Theme-Assets sind kontrolliert, validiert und von Core-App-Rechten getrennt; Apps nutzen die offizielle Theme-Bridge.  
**Begründung und Folgen:** Aktive Styles oder Assets dürfen keine unautorisierten API- oder Browserrechte schaffen.  
**Quellen:** [THEMING.md](THEMING.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [APP_SIGNING.md](APP_SIGNING.md).

#### ADR-025 – App-first-Oberflächen und Core-Service-Grenze

**Status:** `Accepted`  
**Entscheidung / Stand:** Sichtbare Systemfunktionen werden als installierte, signierte Core-Apps angeboten; die autoritativen Dienste verbleiben im Core.  
**Begründung und Folgen:** Eine vertrauenswürdige Bootstrap-Shell darf nicht zu einer versteckten privilegierten App-Laufzeit werden.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

#### ADR-026 – `.cloudApp`-Paketstruktur und signierte Metadaten

**Status:** `Accepted`  
**Entscheidung / Stand:** App-Archive besitzen verpflichtende deklarative Metadaten und eine gesondert geschützte API-Zugriffsdatei.  
**Begründung und Folgen:** Exakte Paketlimits und Schema-Feldversionen müssen vor Produktion eingefroren werden.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [APP_SIGNING.md](APP_SIGNING.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

#### ADR-027 – X.509-Publisher-Vertrauen und signierter Paketindex

**Status:** `Accepted`  
**Entscheidung / Stand:** Die App-Vertrauenskette nutzt X.509-Publisher-Identitäten und einen integritätsgeschützten Dateiindex; Widerruf und Publisher-Bindung sind erforderlich.  
**Begründung und Folgen:** Kryptografisches Feinprofil und Testvektoren müssen vor realen Releases festliegen.  
**Quellen:** [APP_SIGNING.md](APP_SIGNING.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-028 – Geschützte Core-App-Baseline und Recovery

**Status:** `Accepted`  
**Entscheidung / Stand:** Verpflichtende Core-Apps sind signierte Release-Baseline; inkompatible Baseline-Apps blockieren ein Core-Upgrade.  
**Begründung und Folgen:** Defekte Baseline erfordert sicheren Wiederherstellungsweg ohne ungesichertes App-Bypass.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [APP_SIGNING.md](APP_SIGNING.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-029 – Administrative App-Installation über Dateimanager

**Status:** `Accepted`  
**Entscheidung / Stand:** Die Auswahl einer noch nicht installierten `.cloudApp` öffnet eine vertrauenswürdige Installationsvorschau mit ausdrücklicher Admin-Bestätigung.  
**Begründung und Folgen:** Doppelklick darf keine unmittelbare Ausführung oder automatische Rechtevergabe auslösen.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md).

#### ADR-030 – Installation, Zuweisung, Grant und Launch trennen

**Status:** `Accepted`  
**Entscheidung / Stand:** Paketinstallation, Benutzer-/Gruppenzuweisung, API-Genehmigung und App-Start sind getrennte, autorisierte Schritte.  
**Begründung und Folgen:** Zusätzliche systemweit installierte Apps sind standardmäßig nur der Gruppe `Admin` zugeordnet.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md).

#### ADR-031 – Sandbox, App-Bridge und authentifizierter App-Host

**Status:** `Accepted`  
**Entscheidung / Stand:** Jede sichtbare App einschließlich Core-App läuft in eigener Sandbox und kommuniziert ausschließlich über die kontrollierte Host-/SDK-Bridge.  
**Begründung und Folgen:** Browser-Origin allein ist kein ausreichender App-Berechtigungsnachweis.  
**Quellen:** [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md), [APP_API_ACCESS.md](APP_API_ACCESS.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-032 – Unveränderliche App-Versionen und Rollback

**Status:** `Accepted`  
**Entscheidung / Stand:** Installierte App-Versionen sind unveränderlich; Updates und Rollbacks prüfen erneut Signatur, Paketdigest und Rechte-Diff.  
**Begründung und Folgen:** Ein Rollback darf entzogene Berechtigungen nicht wiederherstellen.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-033 – Core-App-Theming und vertrauenswürdige Systemdialoge

**Status:** `Accepted`  
**Entscheidung / Stand:** Theme- und Fensterchrome werden vom privilegierten Host kontrolliert; sicherheitskritische Dialoge sind nicht app-überschreibbar.  
**Begründung und Folgen:** Core-Apps haben kein implizites Sicherheits-Bypass-Recht.  
**Quellen:** [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md), [THEMING.md](THEMING.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-034 – WASM-/WASI-/Native-Paketvarianten

**Status:** `Accepted`  
**Entscheidung / Stand:** Unterschiedliche Runtime-Payloads sind getrennt zu deklarieren; Browser-WASM ist priorisiert, andere Profile erfordern passende isolierte Runner.  
**Begründung und Folgen:** Endgültige ABI-/WIT- und Paketkompatibilitätsmatrix bleibt zu konkretisieren.  
**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

#### ADR-035 – Selbstsignierte App-Pakete und Risikoakzeptanz

**Status:** `Accepted`  
**Entscheidung / Stand:** Kryptografisch gültige selbstsignierte App-Pakete sind nur nach eindeutiger administrativer Risikofreigabe für exakt den freigegebenen Paketdigest installierbar.  
**Begründung und Folgen:** Manipulierte oder unsignierte Pakete bleiben stets gesperrt; Core-Baseline kennt keine Ausnahme.  
**Quellen:** [APP_SIGNING.md](APP_SIGNING.md), [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md).

#### ADR-036 – Signaturalgorithmen und kanonische Serialisierung

**Status:** `Accepted`  
**Entscheidung / Stand:** Das Signaturprofil v1 verwendet SHA-512 für die vollständige Dateiliste, RFC 8785/JCS für kanonisches JSON, RSA-PSS mit SHA-512 und MGF1, einer Salt-Länge von 64 Bytes und RSA-Schlüsseln mit mindestens 3072 Bit sowie X.509 mit überprüfter App-ID-/Publisher-Bindung. Signierte Berechtigungen und Paketdateien werden vollständig und strikt geprüft; mehrdeutige Serialisierung, zusätzliche/unbekannte unsignierte Bytes, ungültige oder widerrufene Zertifikate sind abzulehnen.  
**Begründung und Folgen:** Die Prüfung erfolgt gegen exakt definierte Bytes und negative Interoperabilitätstests. Neue Algorithmen/Signaturprofile dürfen nur ausdrücklich freigegeben und versioniert eingeführt werden; v1 bleibt bis dahin verbindlich.  
**Quellen:** [APP_SIGNING.md](APP_SIGNING.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-037 – Signierte App-API-Allowlist und zentraler Operationenkatalog

**Status:** `Accepted`  
**Entscheidung / Stand:** Jede App deklariert konkret benötigte API-Operationen in `permissions/api-access.json`, verknüpft mit signierter Paketidentität und einem Core-Katalog.  
**Begründung und Folgen:** Nicht deklarierte oder nicht administrativ genehmigte Aufrufe sind per Default-Deny gesperrt.  
**Quellen:** [APP_API_ACCESS.md](APP_API_ACCESS.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-038 – Enforcement an allen App- und Servicegrenzen

**Status:** `Accepted`  
**Entscheidung / Stand:** App-Bridge, Gateway, interne Servicegrenzen, SSE-Subscriptions und WASI-Hosts erzwingen dieselbe effektive Berechtigungsobergrenze.  
**Begründung und Folgen:** Kein Sicherheits-Bypass durch direkte REST-Aufrufe, Sonder-Origins oder Service-Deputy.  
**Quellen:** [APP_API_ACCESS.md](APP_API_ACCESS.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-039 – API-Risikoklassen und verpflichtender Installationsdialog

**Status:** `Accepted`  
**Entscheidung / Stand:** Installationen und Berechtigungserweiterungen zeigen alle beantragten Operationen mit vertrauenswürdig bestimmter Risikodarstellung.  
**Begründung und Folgen:** Entwicklertexte allein bestimmen niemals das tatsächliche Sicherheitsrisiko.  
**Quellen:** [APP_API_ACCESS.md](APP_API_ACCESS.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-040 – Revisionsgebundene App-Grants und sofortiger Einzelwiderruf

**Status:** `Accepted`  
**Entscheidung / Stand:** Entzug einzelner API-Operationen wirkt serverseitig sofort; nur betroffene Funktionen werden deaktiviert. Erweiterungen erfordern signiertes Update plus Adminfreigabe.  
**Begründung und Folgen:** SDK informiert per SSE über effektive Rechte; alte Policy-Caches dürfen kein Recht reaktivieren.  
**Quellen:** [APP_API_ACCESS.md](APP_API_ACCESS.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-041 – Ausgehendes App-Netzwerk ausschließlich über Network Broker

**Status:** `Accepted`  
**Entscheidung / Stand:** Alle zulässigen externen App-Verbindungen werden durch einen zentralen Core Network Broker geprüft, begrenzt und protokolliert.  
**Begründung und Folgen:** Direkte Browser-/Runtime-Netzwerkpfade sind, soweit technisch erzwingbar, zu unterbinden; Grenzen werden dokumentiert.  
**Quellen:** [APP_API_ACCESS.md](APP_API_ACCESS.md), [API_CONTRACTS.md](API_CONTRACTS.md).

**Reservierte Nummern:** `ADR-042` bis einschließlich `ADR-050` sind in den überprüften Quellen nicht belegt und bleiben ohne eigenen Entscheidungseintrag. Die später belegten IDs `ADR-051` ff. werden nicht verschoben.

### 5.2 Desktop, Fensterzustand und Benutzeransichten (ADR-051–060)

#### ADR-051 – `/.desktop/` als verbindliche Desktop-Dateiquelle

**Status:** `Accepted`  
**Entscheidung / Stand:** Sichtbare Desktop-Einträge stammen aus dem Home-Verzeichnis `/.desktop/`.  
**Begründung und Folgen:** Ältere `/Desktop/`-Sichten sind zu migrieren, nicht parallel als Wahrheit zu pflegen.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-052 – Taskbar-Konfiguration `/.taskbar.json`

**Status:** `Accepted`  
**Entscheidung / Stand:** Die persönliche Taskbar-Konfiguration wird im Home über `/.taskbar.json` verwaltet und revisioniert.  
**Begründung und Folgen:** Änderungen gehen über autorisierte APIs und SSE an andere Sitzungen.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [PUSH_REALTIME.md](PUSH_REALTIME.md).

#### ADR-053 – Desktop-Layout `/.desktop-layout.json`

**Status:** `Accepted`  
**Entscheidung / Stand:** Desktop-Positionen referenzieren stabile Datei-/Knoten-IDs in `/.desktop-layout.json`.  
**Begründung und Folgen:** Pfadänderungen dürfen Layoutidentität nicht still zerstören.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-054 – `.cloudlink`-Verknüpfungen und sichere Targets

**Status:** `Accepted`  
**Entscheidung / Stand:** App-/Datei-Verknüpfungen werden als deklarative `.cloudlink`-Dateien aufgelöst.  
**Begründung und Folgen:** Verknüpfungen vergeben keine zusätzlichen Zugriffsrechte und sind keine ausführbaren Pakete.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-055 – Directory-Realtime und sichere SSE-Deltas

**Status:** `Accepted`  
**Entscheidung / Stand:** Desktop-/Dateiänderungen erscheinen als autorisierte SSE-Deltas mit Snapshot-/Replay-Cursor.  
**Begründung und Folgen:** Bei Replay-Lücken erfolgt genau eine kontrollierte Snapshot-Wiederherstellung, kein periodisches Polling.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [PUSH_REALTIME.md](PUSH_REALTIME.md).

#### ADR-056 – Taskbar an vier Bildschirmkanten

**Status:** `Accepted`  
**Entscheidung / Stand:** Die Taskleiste unterstützt alle vier Bildschirmkanten mit entsprechendem Work-Area-/Startmenü-Verhalten.  
**Begründung und Folgen:** Layoutentscheidungen und User-Overrides werden gerätespezifisch sinnvoll gerendert.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [THEMING.md](THEMING.md).

#### ADR-057 – Signierte Dateisystem-Operationen von Core-Apps

**Status:** `Accepted`  
**Entscheidung / Stand:** Auch Desktop und Dateimanager benötigen konkrete signierte API-Operationen für Datei- und Verzeichniszugriffe.  
**Begründung und Folgen:** Core-App-Status erzeugt weder unbeschränkten Dateizugriff noch Installationsrechte.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

#### ADR-058 – Taskbar-Einstellungen als eigener geschützter App-Kontext

**Status:** `Accepted`  
**Entscheidung / Stand:** Taskbar-Einstellungen werden über autorisierte Einstellungsfunktionen aus dem Desktop gestartet.  
**Begründung und Folgen:** Deep Links sind kein Grund, App-Instanz-/SDK-/Berechtigungsprüfungen zu umgehen.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-059 – Migration alter Desktop-Home-Verzeichnisse

**Status:** `Accepted`  
**Entscheidung / Stand:** Im MVP existiert ausschließlich `/.desktop/` als kanonische Desktop-Ablage. Neue Homes folgen ADR-132. Es erfolgt keine automatische Migration alter `/Desktop/`-Strukturen. Bereits vorhandene Altdaten bleiben unverändert und werden weder automatisch gelöscht noch überschrieben; ein kontrollierter Migrationsworkflow ist nach dem MVP gesondert zu spezifizieren.  
**Begründung und Folgen:** Die Migrationsoptionen in älteren Konzepten sind Zukunftsentwürfe und keine MVP-Funktion. Veraltete Beispiele dürfen keinen zweiten Desktop-Root etablieren.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-060 – Desktop-Performance, Browser und Barrierefreiheit

**Status:** `Accepted`  
**Entscheidung / Stand:** Für Chrome, Edge, Firefox und Safari gilt eine verbindliche, fähigkeitsbasierte Browser-, Accessibility- und Performance-Testmatrix. Desktop-Kernfunktionen und das anspruchsvolle Rust/WebGPU/WASM-Profil werden getrennt bewertet. Fehlende Browser-Features werden sichtbar als nicht unterstützt gemeldet; die WebGPU-/WASM-Thread-Anforderungen aus ADR-017 werden nicht stillschweigend abgeschwächt.  
**Begründung und Folgen:** Barrierefreiheit und reproduzierbare Performanceprüfungen sind Release-Gates. Konkrete Browserversionen, Hardwareprofile und Grenzwerte werden nach Prüfung und Messung freigegeben, statt pauschale Parität zu versprechen.  
**Quellen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

### 5.3 App-Installation und Store (ADR-061–072)

#### ADR-061 – `system:/apps/` als kanonischer App-Paketspeicher

**Status:** `Accepted`  
**Entscheidung / Stand:** Installierte, signierte, unveränderliche `.cloudApp`-Archive liegen auf `system:/apps/`.  
**Begründung und Folgen:** Persönliche Dateien und App-Daten bleiben auf dem zweiten Volume `users`.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-062 – `Apps.cloudApp` als reguläre Installationsoberfläche

**Status:** `Accepted`  
**Entscheidung / Stand:** Lokale Installationen erfolgen über `Apps.cloudApp`, aus Settings oder über einen kontrollierten Doppelklick-Intent.  
**Begründung und Folgen:** Keine ungeprüfte Direktinstallation per Explorer-Kontextmenü oder Dateiausführung.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-063 – Globale App-Installation und Admin-Default

**Status:** `Accepted`  
**Entscheidung / Stand:** Apps werden global installiert; neue zusätzliche Apps sind anfänglich nur der Gruppe `Admin` zugewiesen.  
**Begründung und Folgen:** Sichtbarkeit/Startrecht sind nicht mit Dateisicht oder Installation gleichzusetzen.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

#### ADR-064 – Immutable App-Registry, Rollback und Package-GC

**Status:** `Accepted`  
**Entscheidung / Stand:** Paketversionen sind unveränderlich; Aktivierung, Rollback und Bereinigung unterliegen einer nachvollziehbaren Registry-/Referenzpolitik.  
**Begründung und Folgen:** Kein Update-/Rollback-Pfad darf Signaturen, API-Approvals oder gültige Versionen umgehen.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [APP_SIGNING.md](APP_SIGNING.md).

#### ADR-065 – Moderierter GitHub-App-Katalog

**Status:** `Accepted`  
**Entscheidung / Stand:** App-Kataloge können GitHub-Releases einbinden, bleiben aber moderiert und mit Signatur-/Review-Provenienz versehen.  
**Begründung und Folgen:** Ein Review ersetzt weder Paket-Signatur noch administrative API-Berechtigungsfreigabe.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [APP_SIGNING.md](APP_SIGNING.md).

#### ADR-066 – Mehrere App-Katalogquellen und Publisher-Vertrauen

**Status:** `Accepted`  
**Entscheidung / Stand:** Mehrere geprüfte Katalogquellen sind zulässig; ihre Herausgeber- und Repository-Identitäten werden getrennt verifiziert.  
**Begründung und Folgen:** Konkrete Discovery-/Trust-Verfahren sind vor externer Auslieferung festzulegen.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [APP_SIGNING.md](APP_SIGNING.md).

#### ADR-067 – Store-Provenienz und Store-verwaltete Installationen

**Status:** `Accepted`  
**Entscheidung / Stand:** Die installierende Katalogquelle und Installationsprovenienz werden registriert; ein Store verwaltet nur die eigenen Installationen.  
**Begründung und Folgen:** Ein Store erhält keine generelle administrative Hoheit über fremde Apps.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md).

#### ADR-068 – Sofortige App-Zuweisungsinvalidierung

**Status:** `Accepted`  
**Entscheidung / Stand:** Änderungen an App-Zuweisungen werden serverseitig wirksam und über SSE an betroffene Sitzungen verteilt.  
**Begründung und Folgen:** Alte Event-Replays oder Caches dürfen entzogene Launch-Rechte nicht wiederherstellen.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [PUSH_REALTIME.md](PUSH_REALTIME.md).

#### ADR-069 – App-Zuweisungen getrennt von POSIX-Gruppenrechten

**Status:** `Accepted`  
**Entscheidung / Stand:** Explizite App-Zuweisungen bleiben fachlich von Dateirechten und Unix-Gruppenkennungen unterscheidbar.  
**Begründung und Folgen:** Eine gemeinsame Gruppe darf nicht versehentlich VFS- oder Systemrollenrechte erzeugen.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

#### ADR-070 – Deinstallation und Löschung persönlicher App-Daten trennen

**Status:** `Accepted`  
**Entscheidung / Stand:** Deinstallation eines App-Pakets entfernt nicht automatisch alle persönlichen App-Daten.  
**Begründung und Folgen:** Retention und Datenlöschung benötigen eigene autorisierte Aktionen.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-071 – App-Asset-Delivery und Browsercache nach Widerruf

**Status:** `Accepted`  
**Entscheidung / Stand:** Nach App-/Grant-Widerruf sind weitere API-Zugriffe und unzulässige Asset-Abrufe serverseitig gesperrt.  
**Begründung und Folgen:** Bereits lokal heruntergeladene Bytes können nicht magisch aus einem vollständig getrennten Gerät entfernt werden.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

#### ADR-072 – Lokales App-Management, Store-Sync und Egress

**Status:** `Accepted`  
**Entscheidung / Stand:** App-Verwaltung und Katalogsynchronisation verwenden geprüfte Core-Schnittstellen und erlaubte Netzwerkpfade.  
**Begründung und Folgen:** Store-Metadaten werden nicht als unbeschränkte App-Netzwerkberechtigung weitergereicht.  
**Quellen:** [APP_MANAGEMENT_STORE.md](APP_MANAGEMENT_STORE.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

### 5.4 Realtime und Notifications (ADR-073–082)

#### ADR-073 – SSE als verbindlicher Browser-Push-Transport

**Status:** `Accepted`  
**Entscheidung / Stand:** Laufende Core-zu-Client-Zustandsupdates verwenden SSE statt verpflichtender WebSockets.  
**Begründung und Folgen:** Bidirektionale Kommunikation braucht bei Bedarf einen eigenen späteren Vertrag; keine Polling-Fallbackschleife.  
**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-074 – Outbox, JetStream, Delivery-Log und Stream-Ordering

**Status:** `Accepted`  
**Entscheidung / Stand:** Persistente Fachereignisse, Delivery-Cursor und mindestens-einmal-Zustellung mit idempotenter Verarbeitung bilden den Event-Vertrag.  
**Begründung und Folgen:** Systemweite globale Ereignisreihenfolge wird nicht behauptet; Ressourcenkontext und Stream-Epoch sind maßgeblich.  
**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-075 – Snapshot-/Watermark-/Barrier-Bootstrap

**Status:** `Accepted`  
**Entscheidung / Stand:** Initialsnapshot und Stream-Aktivierung vermeiden Zustandslücken beim Start einer Client-Sitzung.  
**Begründung und Folgen:** Bei nachgewiesener Cursor-Lücke erfolgt ein autorisierter einmaliger Resync.  
**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md).

#### ADR-076 – Kein Browser-Polling und sicheres Replay

**Status:** `Accepted`  
**Entscheidung / Stand:** Clients dürfen keine periodischen Statusabrufschleifen verwenden; SSE-Reconnect und Replay übernehmen laufende Änderungszustellung.  
**Begründung und Folgen:** Keepalives sind kein Polling; Cursor werden an Empfänger und Sitzungsrechte gebunden.  
**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-077 – Direkt anwendbare, berechtigungsgefilterte Events

**Status:** `Accepted`  
**Entscheidung / Stand:** Push-Deltas tragen nur für den Empfänger zulässige Zustandsänderungen und verwenden versionierte Schemas.  
**Begründung und Folgen:** Keine unkontrollierten Vollobjekte oder versteckten Ressourcendaten über den Stream.  
**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-078 – Signierte Event-Subscriptions pro App

**Status:** `Accepted`  
**Entscheidung / Stand:** Jede App deklariert benötigte Event-Subscriptions im signierten Paket und benötigt wirksame Freigaben.  
**Begründung und Folgen:** Ein allgemeiner SSE-Stream darf Drittanbieter-Sandboxes nicht direkt offenstehen.  
**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

#### ADR-079 – Persistente Benutzerbenachrichtigungen

**Status:** `Accepted`  
**Entscheidung / Stand:** Benachrichtigungen besitzen einen dauerhaften Zustands-/Read-State; SSE ist der Zustellkanal an aktive Sitzungen.  
**Begründung und Folgen:** Anzeige im Browser ist nicht alleinige Systemwahrheit.  
**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md).

#### ADR-080 – SSE-Auth, Origin-Bindung und Widerruf

**Status:** `Accepted`  
**Entscheidung / Stand:** SSE-Zustellung und Replay sind an sichere BFF-Session, Benutzer, App-Audience und aktuelle Berechtigungen gebunden.  
**Begründung und Folgen:** Entzug eines Grants muss auch alte Subscription-/Replay-Cursor unwirksam machen.  
**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-081 – SSE-Proxy, Multi-Tab und Backpressure

**Status:** `Accepted`  
**Entscheidung / Stand:** Nginx muss SSE ohne aggressives Puffern übertragen; Backpressure, Idle-Reconnect und Sitzungstrennung sind zu erzwingen.  
**Begründung und Folgen:** Mehrere Tabs dürfen keine Konten oder Clientberechtigungen vermischen.  
**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-082 – Web Push bei geschlossenem Browser

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Browser-Web-Push bleibt außerhalb des MVP und ist strikt opt-in: ausdrückliche Browser- und cloudOS-Zustimmung pro Benutzer/Gerät, widerrufbare Subscription, serverseitig verwaltete Push-Schlüssel und privacy-minimierter Payload. Externer Push überträgt ausschließlich einen neutralen Zustellungshinweis und niemals vertrauliche Dokumente, Nutzinhalte, Berechtigungsdaten oder ausführbare Aktionen. Der Notification Owner bleibt autoritativ; Details erst nach gültigem Login per bestehender autorisierter Core-API. SSE bleibt für offene Tabs verbindlich.  
**Begründung und Folgen:** Browser-/Push-Dienst-Verfügbarkeit ist nicht garantiert und Web Push darf keine zweite Rechteautorität werden.  
**Freigabe-/Nachweisgrenze:** Datenschutzprüfung der Push-Provider/Egress-Domäne, Consent, Logout/Revoke, veraltete Subscriptions, SW-Origin-Policy, Browser-Kompatibilität, Zustelllücken/Replay.  
**Einführungsprofil:** Nach MVP, standardmäßig aus; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md), [API_CONTRACTS.md](API_CONTRACTS.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-082).

### 5.5 Core-Deployment und Kubernetes (ADR-083–098)

#### ADR-083 – Combined-Core als Standard, Split/Kubernetes opt-in

**Status:** `Accepted`  
**Entscheidung / Stand:** Sämtliche regulären Core-Dienste laufen standardmäßig in genau einem Go-Prozess/Core-Container.  
**Begründung und Folgen:** Optionaler Split-/Clusterbetrieb muss dieselben Fach- und Sicherheitsverträge einhalten.  
**Quellen:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-084 – Gemeinsamer Servicekatalog und identischer Go-Code

**Status:** `Accepted`  
**Entscheidung / Stand:** Ein Servicekatalog und derselbe Go-Codebestand definieren combined, split und distributed.  
**Begründung und Folgen:** Keine parallel auseinanderlaufenden Fachimplementierungen je Betriebsprofil.  
**Quellen:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-085 – Lokale Adapter versus gRPC-Ports

**Status:** `Accepted`  
**Entscheidung / Stand:** Lokale Go-Ports und entfernte gRPC-/Protobuf-Aufrufe müssen fachlich gleichwertig sein.  
**Begründung und Folgen:** Autorisierung, Deadlines und Fehlercodes bleiben deploymentunabhängig.  
**Quellen:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-086 – Discovery und Retry-/Deadline-Budgets

**Status:** `Accepted`  
**Entscheidung / Stand:** Verteilte Dienste verwenden Cluster-Service-Discovery und begrenzte Retries, Timeouts und Failover.  
**Begründung und Folgen:** Konkrete Budgetwerte werden nach Betriebsprofil und Lasttests festgelegt.  
**Quellen:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-087 – Shared State, Owner-Schemas und Revisionen

**Status:** `Accepted`  
**Entscheidung / Stand:** Alle Replikate benutzen geteilten langlebigen Zustand mit Owner-Zuständigkeiten und revisionsgeprüften Cache-Invalidierungen.  
**Begründung und Folgen:** Kein alleiniger lokaler Pod-Speicher als Quelle für Sessions, Jobs, Datei-Metadaten oder Berechtigungen.  
**Quellen:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-088 – Dauerhafte Worker-Leases und Idempotenz

**Status:** `Accepted`  
**Entscheidung / Stand:** Parallele Worker koordinieren Aufträge über Leases/Fencing und idempotente Effekte.  
**Begründung und Folgen:** Ausgefallene Worker werden kontrolliert ersetzt, ohne doppelte fachliche Wirkung zu erzeugen.  
**Quellen:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-089 – SSE-Fanout, Replay und Rolling-Drain

**Status:** `Accepted`  
**Entscheidung / Stand:** Realtime-Fanout und Cursor-/Replay-Vertraege gelten bei Replikat-Ausfall, Neustart und Wiederverbindung. Geplante Core-Releases nutzen den ausdruecklich bestaetigten vollstaendigen Wartungsstopp, nicht verpflichtende Rolling Releases.  
**Begruendung und Folgen:** Ordinaeres Replica-Drain bleibt ein technische Faehigkeit; es hebt die Core-Update-Freigaberegel nicht auf.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-090 – HPA, Autoscaling und Kapazitätsmetriken

**Status:** `Accepted`  
**Entscheidung / Stand:** Horizontale Skalierung nutzt geeignete CPU-/Custom-Metrics und später ergänzend Queue-Metriken.  
**Begründung und Folgen:** Aktivierung, Schwellen und mögliche KEDA-Nutzung sind kapazitätsabhängig zu bestimmen; im combined-MVP keine automatische Pod-Vervielfachung zwingend.  
**Quellen:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-091 – Workload-Identitäten, mTLS und interne API-Grenzen

**Status:** `Accepted`  
**Entscheidung / Stand:** Verteilte Instanzen erhalten echte Workload-Identitäten und authentifizierte Service-Capabilities.  
**Begründung und Folgen:** Kubernetes-Namen/IPs erzeugen keine fachlichen Berechtigungen.  
**Quellen:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-092 – Gemeinsamer Storage und immutable Paketversionen

**Status:** `Accepted`  
**Entscheidung / Stand:** Geteilte Blob-/Paket-Stores werden durch fachliche Owner kontrolliert; installierte App-Pakete sind unveränderlich.  
**Begründung und Folgen:** Speicheradapter dürfen keine unautorisierte Replika-/App-Direktmanipulation erlauben.  
**Quellen:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-093 – Datenbankmigrationen bei Version-Skew

**Status:** `Accepted`  
**Entscheidung / Stand:** Owner-eigene PostgreSQL-Migrationen mit zentraler dauerhafter Version-/Abhaengigkeitskoordination; Expand -> Migrate -> Contract, checkpointfaehige Backfills, minimale temporaere Rechte.  
**Begruendung und Folgen:** Fehler blockieren Aktivierung; irreversible Schemaaenderungen lassen sich nicht durch Container-Image-Rollback rueckgaengig machen.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-094 – Kubernetes-Produktionshärtung und Secrets

**Status:** `Proposed`  
**Entscheidung / Stand:** Kubernetes-Produktion erfordert non-root, Least Privilege, wirksame NetworkPolicies, workload-gebundene Secrets, geschuetztes TLS/mTLS und getestete Infrastrukturprofile; Secret-Quellen bleiben providerunabhaengig.  
**Begruendung und Folgen:** Konkrete CNI-/Provider-Konfigurationen und Nachweise bleiben technische Abnahmearbeit, nicht offene Grundsatzwahl.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-095 – Combined-to-Split-Migration ohne doppelte Owner

**Status:** `Accepted`  
**Entscheidung / Stand:** Combined-to-Split nur durch vollstaendigen Core-Wartungsstopp mit geprueftem Backup, eindeutiger Owner-Uebergabe, validierten Workload-Identitaeten und anschliessender kontrollierter Freigabe.  
**Begruendung und Folgen:** Kein parallel schreibender alter und neuer Owner; Rueckumstellung nur nach erneuter Konsistenz- und Sicherheitspruefung.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-096 – Startup, Liveness, Readiness und Funktionsgesundheit

**Status:** `Accepted`  
**Entscheidung / Stand:** Startup, Liveness, Readiness und Functional Health getrennt; pro Owner essential/optional Abhaengigkeiten pruefen und selektiv degradieren.  
**Begruendung und Folgen:** Kritische Autorisierungs- und Konsistenzgrenzen bleiben Fail-Closed; ein Readiness-Fehler ist nicht automatisch ein Neustartgrund.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-097 – Service-SLOs, Observability und Lasttests

**Status:** `Accepted`  
**Entscheidung / Stand:** Dienstspezifische SLIs/SLOs und reproduzierbare Baseline-/Stress-/Soak-/Failover-/Skalierungstests. Ressourcenbudgets und HPA-Regeln werden anhand realer Last bestaetigt.  
**Begruendung und Folgen:** Keine verbindlichen unbelegten Produktionsschwellwerte und keine automatischen Core-Releases oder Topologiewechsel aufgrund von SLO-Verletzungen.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-098 – Helm/Kustomize/Operator-Werkzeuge

**Status:** `Accepted`  
**Entscheidung / Stand:** Docker Compose bleibt Standard; Kustomize ist verbindliche Kubernetes-Referenz. Helm nur als semantisch gleichwertiger optionaler Pfad; Operator nur nach eigener Nutzen-/Sicherheitsentscheidung.  
**Begruendung und Folgen:** Deployment-Werkzeuge uebernehmen keine fachliche Datenhoheit oder eigenstaendige Core-Update-Freigabe.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

### 5.6 Authentisierungsmethoden (ADR-099–103)

#### ADR-099 – Credential-Lifecycle und Aliasmodell

**Status:** `Accepted`  
**Entscheidung / Stand:** Lokale Konten besitzen mehrere registrierbare Authentisierungsmethoden mit kontrollierter Zuordnung, Widerruf und Sicherheitshistorie.  
**Begründung und Folgen:** Credential-Änderungen bleiben an Benutzeridentität und Step-up-Policy gebunden.  
**Quellen:** [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

#### ADR-100 – WebAuthn-/Passkey-Relying-Party-Policy

**Status:** `Accepted`  
**Entscheidung / Stand:** Passkeys/WebAuthn sind unterstützte Anmeldemethoden; RP-/Origin-Prüfung und sichere Credential-Lebenszyklen sind Pflicht.  
**Begründung und Folgen:** Konkrete Attestation-/Backup-Flag- und Recovery-Policy muss vor Produktfreigabe präzisiert werden.  
**Quellen:** [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md).

#### ADR-101 – SSH-Agent-Signaturen und SSH-Key-Anmeldung

**Status:** `Accepted`  
**Entscheidung / Stand:** SSH-Schlüsselbasierte Anmeldung kann über einen lokalen Companion/Agent und servergeprüfte Challenge-Signaturen erfolgen.  
**Begründung und Folgen:** Der Browser darf nie private SSH-Schlüssel an den Server übertragen; Browser-/Client-Vertrauensgrenze ist verbindlich.  
**Quellen:** [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md).

#### ADR-102 – Assurance-Level, Step-up und Admin-Recovery

**Status:** `Accepted`  
**Entscheidung / Stand:** Besonders kritische Aktionen benötigen ein angemessenes Authentisierungsniveau und können Step-up bzw. getrennte Recovery erfordern.  
**Begründung und Folgen:** Ein normaler eingeloggter App-Kontext ersetzt keine privilegierte Freigabe.  
**Quellen:** [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-103 – Externe Identity-Verknüpfung zu lokalen Credentials

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Externe Konten werden ausschließlich an stabile lokale `user_id` gebunden. LDAP nutzt stabile source_id + source_object_id; OIDC nutzt `(issuer, sub)`. Keine automatische Fusion anhand E-Mail, Loginname, DN oder Anzeigename. Die erstmalige Verknüpfung erfordert einen nachgewiesenen Besitz-/Authentifizierungsbezug zu beiden Identitäten oder einen explizit genehmigten, step-up-geschützten, auditierten Admin-Workflow. Konflikte werden blockiert/quarantänisiert, keine automatische Rechteübernahme oder Home-Neuanlage durch Providerwechsel.  
**Begründung und Folgen:** Account-Takeover und Verwechslung vorhandener Benutzerrechte, Homes oder Space-Mitgliedschaften verhindern.  
**Freigabe-/Nachweisgrenze:** Tests für Rename, Dubletten, Identifier-Reuse, Provider-Migration, Revocation, OIDC-Sub-Wechsel, verlorenes externes Konto und sicheren Break-Glass-Pfad.  
**Einführungsprofil:** Für optionale LDAP/OIDC-Adapter; lokal im MVP unabhängig; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-103).

### 5.7 Logging, Audit und Tracing-Basis (ADR-104–108)

#### ADR-104 – Strukturierte Logs, OpenTelemetry und Logger-Hierarchie

**Status:** `Accepted`  
**Entscheidung / Stand:** Core-Module erzeugen typisierte strukturierte Logs mit getrennt konfigurierbaren Kategorien und Telemetriesignalen.  
**Begründung und Folgen:** Schutzbedürftige Inhalte sind aus Diagnoselogs und Trace-Attributen auszuschließen.  
**Quellen:** [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-105 – Verteilung der Logging-Policy und sicherer Fallback

**Status:** `Accepted`  
**Entscheidung / Stand:** Logging-Policies sind zentral revisioniert und gelten auch bei mehreren Core-Replikaten konsistent.  
**Begründung und Folgen:** Exakte TTL-/Fallback-/Cache-Strategien bleiben vor Implementierung abnahmebedürftig.  
**Quellen:** [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md).

#### ADR-106 – Unabschaltbares Security-Audit

**Status:** `Accepted`  
**Entscheidung / Stand:** Verpflichtende Sicherheitsereignisse werden unabhängig von Diagnostik-Leveln protokolliert.  
**Begründung und Folgen:** Nutzer-/Adminaktionen am Audit selbst sind sicherheitsrelevant und nicht unterdrückbar.  
**Quellen:** [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-107 – Log-Aufbewahrung, Datenschutz und Export

**Status:** `Accepted`  
**Entscheidung / Stand:** Logkategorien erhalten getrennte Retention-, Quota-, Lösch- und Archivierungsregeln.  
**Begründung und Folgen:** Besonders schützenswerte Audit-Aufzeichnungen sind manipulationsresistent zu halten; Fristen sind als Policy-Werte offen.  
**Quellen:** [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-108 – `Logs.cloudApp` und Log-Berechtigungen

**Status:** `Accepted`  
**Entscheidung / Stand:** Die vorinstallierte `Logs.cloudApp` bietet berechtigungsgefilterte Diagnose-, Audit- und Trace-Ansichten.  
**Begründung und Folgen:** Sehen, Konfigurieren und Audit-Administrieren sind getrennte Privilegien; sensitive Lesezugriffe werden auditiert.  
**Quellen:** [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md), [API_CONTRACTS.md](API_CONTRACTS.md).

### 5.8 Window Host (ADR-109–113)

#### ADR-109 – Verbindlicher Window Host für alle sichtbaren Apps

**Status:** `Accepted`  
**Entscheidung / Stand:** Alle sichtbaren Apps werden im vertrauenswürdigen zentralen Window Host dargestellt.  
**Begründung und Folgen:** App-Inhalte können weder Fensterchrome noch privilegierte Systemdialoge durch unkontrollierten DOM-Zugriff ersetzen.  
**Quellen:** [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-110 – Fensterzustände, Taskbar und Close-Semantik

**Status:** `Accepted`  
**Entscheidung / Stand:** Öffnen, Schließen, Minimieren und Fokus werden durch versionierte Host-/SDK-Verträge verwaltet.  
**Begründung und Folgen:** Persistente Fensterzustände und session-bound App-Jobs besitzen gesonderte Lebenszyklen.  
**Quellen:** [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-111 – Runtime-Bridge und sichere Modal-Dialoge

**Status:** `Accepted`  
**Entscheidung / Stand:** Fenster-/Modal-Anfragen erfolgen ausschließlich durch SDK-Bridge und vertrauenswürdige Host-Dialoge.  
**Begründung und Folgen:** App-generierte UI darf keinen vermeintlichen Core-Sicherheitsdialog imitieren.  
**Quellen:** [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

#### ADR-112 – Workspace-/Mehrtab-Zustand und Replikation

**Status:** `Accepted`  
**Entscheidung / Stand:** Gemeinsame Workspace-Zustände werden revisioniert und über mehrere Tabs/Endgeräte synchronisiert.  
**Begründung und Folgen:** Lokale Bildschirmlayouts können abweichen; serverseitige Konflikt- und Event-Ordnung bleibt maßgeblich.  
**Quellen:** [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-113 – Theme- und Accessibility-Vertrag des Window Hosts

**Status:** `Accepted`  
**Entscheidung / Stand:** Fensterchrome übernehmen zugelassene Theme-Tokens und Barrierefreiheitsanforderungen aus der Host-Umgebung.  
**Begründung und Folgen:** Theme-Auswahl oder Animation darf sicherheitskritische Bedienelemente nicht entstellen.  
**Quellen:** [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md), [THEMING.md](THEMING.md).

### 5.9 Client-Geräte (ADR-114–118)

#### ADR-114 – Client Device Broker statt direkter Geräte-APIs

**Status:** `Accepted`  
**Entscheidung / Stand:** WebUSB, WebHID, WebSerial und lokale Dateiauswahl werden von einem privilegierten Client Device Broker gesteuert.  
**Begründung und Folgen:** Drittanbieter-Apps erhalten nur freigegebene Fähigkeiten und keine unkontrollierte Host-Geräteverbindung.  
**Quellen:** [CLIENT_DEVICES.md](CLIENT_DEVICES.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

#### ADR-115 – Device-Capabilities und Installations-Risikoanzeigen

**Status:** `Accepted`  
**Entscheidung / Stand:** Gerätefähigkeiten sind im signierten App-Kontext separat zu deklarieren und risikogerecht sichtbar zu machen.  
**Begründung und Folgen:** Ein API-Endpunkt-Grant beinhaltet nicht automatisch einen USB-/Serial-Grant.  
**Quellen:** [CLIENT_DEVICES.md](CLIENT_DEVICES.md).

#### ADR-116 – Browser-Permission, User Activation und Grant-TTL

**Status:** `Accepted`  
**Entscheidung / Stand:** Gerätefreigaben erfordern passende Browserberechtigung, echte Nutzeraktion und begrenzte Core-Grants.  
**Begründung und Folgen:** App-/Session-Widerruf entzieht weitere Nutzung, soweit Browser/OS technisch durchsetzbar.  
**Quellen:** [CLIENT_DEVICES.md](CLIENT_DEVICES.md).

#### ADR-117 – USB-Dateiimport und VFS-Isolation

**Status:** `Accepted`  
**Entscheidung / Stand:** Lokale USB-/Dateiimporte verwenden geprüfte Dateiauswahl und autorisierte VFS-Zieloperationen.  
**Begründung und Folgen:** Ein lokaler Gerätepfad wird niemals direkt als vertrauenswürdiger Server-Dateipfad behandelt.  
**Quellen:** [CLIENT_DEVICES.md](CLIENT_DEVICES.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-118 – Browserkompatibilität der Geräteprofile

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Der Device Broker veröffentlicht eine browser- und geräteprofilbezogene, getestete Capability-Matrix. WebUSB, WebHID, Web Serial und File-Picker sind niemals generell vorausgesetzt. Pflichtfähigkeiten fehlen: Profilstart/Operation wird mit `UNSUPPORTED` gesperrt; optionale Fähigkeiten werden nachvollziehbar deaktiviert. Kein Umgehen durch untrusted App-iframe, privilegierte Hintergrundzugriffe oder rohe USB-Mounts. Standard-Dateiauswahl und Keyboard-Wedge nur, wenn die jeweilige fachliche Funktion damit sicher vollständig erfüllt ist.  
**Begründung und Folgen:** Heterogene Browser müssen ehrlich und sicher behandelt werden, ohne implizite Gerätefreigaben.  
**Freigabe-/Nachweisgrenze:** Chrome/Edge/Firefox/Safari mit positiven und negativen Browser-/OS-/Gerätetests; HTTPS, Permissions Policy, User Activation, Revoke, HID-Sperrlisten und Origin-Isolation.  
**Einführungsprofil:** Profilweise vor jeweiliger Device-Releasefreigabe; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [CLIENT_DEVICES.md](CLIENT_DEVICES.md), [APP_API_ACCESS.md](APP_API_ACCESS.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-118).

### 5.10 Themes (ADR-119–124)

#### ADR-119 – Sechs integrierte Theme-Presets

**Status:** `Accepted`  
**Entscheidung / Stand:** cloudOS erhält sechs eigene, OS-inspirierte Design-Presets.  
**Begründung und Folgen:** Anmutung darf fremde geschützte UI-Assets/Marken nicht ungeprüft übernehmen.  
**Quellen:** [THEMING.md](THEMING.md).

#### ADR-120 – Benutzer-Overrides für Theme und Wallpaper

**Status:** `Accepted`  
**Entscheidung / Stand:** Eigene Taskbar-/Theme-/Wallpaper-Einstellungen des Benutzers können zulässige Theme-Defaults überschreiben.  
**Begründung und Folgen:** Wirksame Policy-Grenzen bleiben durchsetzbar; Wallpaper-Auswahl ist unabhängig vom Preset.  
**Quellen:** [THEMING.md](THEMING.md), [WALLPAPER_ENGINE.md](WALLPAPER_ENGINE.md).

#### ADR-121 – Theme Service und kontrollierte Browser-Tokens

**Status:** `Accepted`  
**Entscheidung / Stand:** Core Theme Service besitzt die autoritative Policy; der Browser löst zulässige Tokens lokal auf.  
**Begründung und Folgen:** UI-Reaktionsfähigkeit wird gewahrt, ohne die App-Sandbox zu öffnen.  
**Quellen:** [THEMING.md](THEMING.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

#### ADR-122 – Theme-Asset-Sandbox und geprüfte Paketfreigabe

**Status:** `Accepted`  
**Entscheidung / Stand:** Themes und Assets müssen geprüft, isoliert und mit restriktiven Content-Security-Policies ausgeliefert werden.  
**Begründung und Folgen:** Keine ausführbaren Rechte, fremde Netzwerkfreigaben oder XSS-Wege über Themes.  
**Quellen:** [THEMING.md](THEMING.md), [APP_SIGNING.md](APP_SIGNING.md).

#### ADR-123 – Barrierefreiheit und geschützte Systemdialoge

**Status:** `Accepted`  
**Entscheidung / Stand:** Alle Themes respektieren Kontrast, Reduced Motion und den Sicherheitsschutz vertrauenswürdiger Dialoge.  
**Begründung und Folgen:** Darstellung darf Permission-/Login-Dialoge nicht manipulativ überdecken.  
**Quellen:** [THEMING.md](THEMING.md), [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md).

#### ADR-124 – Theme-Livewechsel über SSE

**Status:** `Accepted`  
**Entscheidung / Stand:** Zulässige Theme-Änderungen werden in aktiven Sitzungen über autorisierte SSE-Deltas zugestellt.  
**Begründung und Folgen:** Keine periodischen Theme-REST-Abfragen.  
**Quellen:** [THEMING.md](THEMING.md), [PUSH_REALTIME.md](PUSH_REALTIME.md).

### 5.11 Wallpaper (ADR-125–130)

#### ADR-125 – Wallpaper-Rendering ausschließlich im Browser

**Status:** `Accepted`  
**Entscheidung / Stand:** Animierte Hintergründe werden lokal gerendert; der Core streamt keine Frames.  
**Begründung und Folgen:** Serverzustand umfasst Auswahl und Policy, nicht das Rendern jedes Einzelbilds.  
**Quellen:** [WALLPAPER_ENGINE.md](WALLPAPER_ENGINE.md).

#### ADR-126 – Sechs eigenentwickelte animierte Wallpaper

**Status:** `Accepted`  
**Entscheidung / Stand:** Sechs geplante Wallpaper-Effekte werden eigenständig entwickelt und als getrennte Presets geführt.  
**Begründung und Folgen:** Drittanbieter-Code/Assets sind keine implizite Lizenzfreigabe.  
**Quellen:** [WALLPAPER_ENGINE.md](WALLPAPER_ENGINE.md).

#### ADR-127 – Wallpaper-Auswahl unabhängig vom Theme

**Status:** `Accepted`  
**Entscheidung / Stand:** Nutzer können das Hintergrundbild unabhängig vom Theme-Preset wählen.  
**Begründung und Folgen:** Effektive Darstellung berücksichtigt Nutzerentscheidung und Admin-Policy.  
**Quellen:** [WALLPAPER_ENGINE.md](WALLPAPER_ENGINE.md), [THEMING.md](THEMING.md).

#### ADR-128 – Worker-/OffscreenCanvas-Strategie mit Fallback

**Status:** `Accepted`  
**Entscheidung / Stand:** Geeignete Wallpaper-Renderer verwenden Worker/OffscreenCanvas, mit statischem Fallback bei fehlenden Fähigkeiten.  
**Begründung und Folgen:** Animationsfähigkeit darf nicht vorausgesetzt werden.  
**Quellen:** [WALLPAPER_ENGINE.md](WALLPAPER_ENGINE.md).

#### ADR-129 – Wallpaper-Leistungsbudget und Reduced Motion

**Status:** `Accepted`  
**Entscheidung / Stand:** Wallpaper-Rendering reagiert auf Unsichtbarkeit, Energiesparen und Accessibility; Pause/Reduzierung sind Pflicht.  
**Begründung und Folgen:** Konkrete Schwellen werden durch Browser- und Performance-Tests nachgewiesen.  
**Quellen:** [WALLPAPER_ENGINE.md](WALLPAPER_ENGINE.md), [THEMING.md](THEMING.md).

#### ADR-130 – Wallpaper-Änderungen per SSE

**Status:** `Accepted`  
**Entscheidung / Stand:** Änderungen von Wallpaper-Präferenzen werden über den bestehenden autorisierten SSE-Kanal verteilt.  
**Begründung und Folgen:** Kein eigener Polling- oder Frame-Transport für Hintergrundbilder.  
**Quellen:** [WALLPAPER_ENGINE.md](WALLPAPER_ENGINE.md), [PUSH_REALTIME.md](PUSH_REALTIME.md).

### 5.12 Konsolidierte eindeutige Fach-ADRs (ADR-131–160)

#### ADR-131 – VFS-Namen, Unicode-Normalisierung und Case-Policy

**Status:** `Accepted`  
**Entscheidung / Stand:** VFS-Namen werden in Unicode NFC kanonisiert und innerhalb eines Verzeichnisses anhand des normalisierten Namens eindeutig geprüft. Vergleiche bleiben case-sensitive (`Bericht.txt` und `bericht.txt` sind verschieden). Server-Validatoren verhindern reservierte Namen, normalisierungsbedingte Kollisionen und unerlaubte Pfadtricks.  
**Begründung und Folgen:** Import, Export und Synchronisation auf abweichend normalisierenden oder case-insensitiven Plattformen müssen Konflikte erkennbar und ohne stillschweigendes Überschreiben behandeln.  
**Quellen:** [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-132 – Standardstruktur neu angelegter Home-Verzeichnisse

**Status:** `Accepted`  
**Entscheidung / Stand:** Ein neues Home enthält die reservierten Desktop-Ressourcen `/.desktop/`, `/.taskbar.json`, `/.desktop-layout.json` und die Standardordner `Documents/`, `Downloads/`, `Pictures/`, `Music/`, `Videos/`, `.config/`, `.local/`. Freigaben und Spaces werden nur als autorisierte virtuelle Einbindungen bereitgestellt.  
**Begründung und Folgen:** Provisionierung ist über stabile Benutzer-IDs idempotent. Vom Benutzer später entfernte reguläre Standardordner werden beim Login nicht ungefragt neu erzeugt; geschützte Desktop-Ressourcen besitzen separate Reparatur- und Recovery-Regeln.  
**Quellen:** [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-133 – Idempotente Home-Bereitstellung und Konto-Lifecycle

**Status:** `Accepted`  
**Entscheidung / Stand:** Home-Provisionierung erfolgt stabil anhand der Benutzer-ID und ist bei wiederholten Starts sicher wiederaufnehmbar.  
**Begründung und Folgen:** Löschung eines Kontos ist vom Retention-/Datenlösch-Lebenszyklus zu trennen.  
**Quellen:** [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

#### ADR-134 – Cross-Volume-Kopie und kontrollierter Import/Export

**Status:** `Accepted`  
**Entscheidung / Stand:** Zwischen `system` und `users` erfolgen keine unkontrollierten atomaren Direkt-Moves; nötig sind autorisierte Kopier-/Import-Workflows.  
**Begründung und Folgen:** Erforderliche neue Berechtigungen, Ziel-Quota und Fehlerkompensation sind serverseitig zu prüfen.  
**Quellen:** [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-135 – VFS-Quotas, Versionen und Papierkorb

**Status:** `Accepted`  
**Entscheidung / Stand:** Logische Quotas berücksichtigen Dateien, Versionen und den konfigurierbaren Papierkorb unabhängig von physischer Deduplizierung.  
**Begründung und Folgen:** Sichere Garbage Collection erfolgt erst nach Ablauf der maßgeblichen Referenz-/Retention-Regeln.  
**Quellen:** [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-136 – VFS-Backup-Konsistenz und selektives Restore

**Status:** `Accepted`  
**Entscheidung / Stand:** Koordinierte versionierte Snapshots verknüpfen PostgreSQL-Metadaten, File-Nodes, ACLs, Space-Referenzen, referenzierte unveränderliche Blobs sowie zugehörige Schlüsselversionen zu einem konsistenten Sicherungsstand. Vollständige Instanz-, Home- und Einzeldatei-Restores sind vorgesehen; Wiederherstellung und Integritätsprüfung erfolgen zunächst isoliert vor kontrollierter Freigabe.  
**Begründung und Folgen:** Backups und Restores dürfen keine aktuellen fremden Daten oder Systemrichtlinien ungewollt überschreiben und sollen den laufenden Betrieb möglichst wenig beeinträchtigen. Konkrete RPO-/RTO-Werte unterliegen ADR-012; kontinuierliche Point-in-Time-Recovery wird nicht allein durch diese ADR zugesagt.  
**Quellen:** [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [API_CONTRACTS.md](API_CONTRACTS.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-137 – Lokaler LDAPv3-Protokollumfang und TLS

**Status:** `Accepted`  
**Entscheidung / Stand:** Ein lokaler LDAPv3-kompatibler Directory-Dienst bietet einen bewusst begrenzten Protokollumfang mit sicheren TLS-Grenzen.  
**Begründung und Folgen:** Das verbindliche Interoperabilitäts-/RootDSE-/Schema-Testprofil ist vor Auslieferung abzunehmen.  
**Quellen:** [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

#### ADR-138 – UID-/GID-Nummernkreise und Wiederverwendung

**Status:** `Accepted`  
**Entscheidung / Stand:** Virtuelle POSIX-UID und -GID werden zentral, eindeutig und transaktional vergeben, unabhängig voneinander in getrennten Nummernräumen. `0` ist reserviert; `1–9999` sind für interne Systemidentitäten vorgesehen; reguläre Benutzer und Gruppen erhalten Werte von `10000` bis `2147483647`. Stabile UUIDs bleiben autoritativ. Einmal vergebene UID-/GID-Werte werden niemals erneut vergeben, auch nicht nach Löschung. Private Primärgruppen werden unterstützt.  
**Begründung und Folgen:** Provisionierung, Parallelität, Backup und Restore müssen Zuordnungen und Tombstones erhalten sowie Nummernkonflikte sicher ablehnen. Externe UID/GID-Importe werden durch ADR-156 separat entschieden und dürfen die lokale Eindeutigkeit nicht aufweichen.  
**Quellen:** [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-139 – Kanonisches Benutzerkonto- und Gruppenschema

**Status:** `Accepted`  
**Entscheidung / Stand:** Lokale Benutzer und Gruppen besitzen stabile Identitäten, Primär- und Zusatzgruppen und kontrollierte Attribute.  
**Begründung und Folgen:** Exakte Normalisierung von Namen/Aliassen sowie Feldlimits erfordern einen Schema-Contract.  
**Quellen:** [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

#### ADR-140 – Passwort-Hashing, Passwortpolitik und Recovery

**Status:** `Accepted`  
**Entscheidung / Stand:** Passwörter werden mit Argon2id gehasht. Einstiegsparameter sind 19 MiB Speicher, zwei Durchläufe und Parallelität 1, mit messungs-/risikobasierter Nachkalibrierung. Mindestlänge 15 Zeichen, kein erzwungener regelmäßiger Passwortwechsel, Prüfung auf kompromittierte Passwörter und risikobasierte Rate-Limits. Self-Service- und administrative Wiederherstellung sind kontrolliert, auditierbar und dürfen kein universelles Masterpasswort benutzen.  
**Begründung und Folgen:** Schlüssel-/Hashmaterial wird nie als Klartext preisgegeben; Recovery darf keinen Authentifizierungs- oder Rechte-Bypass erzeugen. Recovery-Token-Laufzeiten, Bindungen und genaue Re-Hash-Gates werden im Sicherheitsvertrag mit Testnachweisen präzisiert, nicht erfunden.  
**Quellen:** [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-141 – Systemrollen getrennt von POSIX-Gruppen

**Status:** `Accepted`  
**Entscheidung / Stand:** Administrative Systemrollen und Dateisystemgruppen werden getrennt geprüft; Administratorstatus erzeugt keinen automatischen Fremd-Home-Zugriff.  
**Begründung und Folgen:** Gruppen-IDs sind keine versteckten universellen Root-Rechte.  
**Quellen:** [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-142 – Settings-Sub-Apps und eigener Berechtigungskontext

**Status:** `Accepted`  
**Entscheidung / Stand:** Für den MVP dient `Settings.cloudApp` als zentrale Navigation. `UserManagement.cloudApp` und `GroupManagement.cloudApp` werden als getrennte signierte Pflicht-Core-Apps ausgeliefert. `PermissionManagement.cloudApp` wird auf später verschoben; zwingende administrative Berechtigungsoperationen bleiben über gesondert geschützte Core-Verträge kontrollierbar. Deep Links und Fenster-/Startverträge sind versioniert. Jede Sub-App verfügt über eigene Identität und signierte API-Allowlist.  
**Begründung und Folgen:** Die Settings-Eltern-App erbt niemals Child-App-Rechte. Eingebettete Fenster, Deep Links und Navigation dürfen App-Kontext, Session, Signatur- oder API-Genehmigung nicht umgehen.  
**Quellen:** [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [SETTINGS_SYSTEM.md](SETTINGS_SYSTEM.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-143 – Erweiterte POSIX-ACLs und Named Grants

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Named-User/Named-Group-ACLs und Verzeichnis-Default-ACLs werden als optionale spätere VFS-Ausbaustufe nach dem Basismodell `owner:group:mode` eingeführt. Die Auswertung orientiert sich an POSIX-ACL-Einträgen einschließlich effektiver Gruppen-/Named-User-Maske, Vererbungs- und `chmod`-Wechselwirkungen. ACLs erweitern nie Mount-Sicht, Share-/Space-Grant, signierte App-API-Operationen oder aktuelle System-/Deny-Policy. Es gibt keine impliziten Named-ACLs im MVP.  
**Begründung und Folgen:** POSIX-nahe Semantik ist nur sicher, wenn ein `chmod`- oder ACL-Update nicht unbemerkt andere Grants verstärkt.  
**Freigabe-/Nachweisgrenze:** Vor Freigabe: geprüfte Masken-/Vererbungs-Matrix, `chmod`/`setfacl`-Äquivalenzfälle, Widerruf, Multiuser-/Share-/Space-Negativtests, Migration bestehender Mode-Bits.  
**Einführungsprofil:** Nach Basis-VFS-MVP, standardmäßig aus; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-143).

#### ADR-144 – Interne Share-Rollen und Revocation

**Status:** `Accepted`  
**Entscheidung / Stand:** Datei-/Ordnerfreigaben verwenden definierte, ressourcenbezogene Rollen und serverseitig wirksamen Entzug.  
**Begründung und Folgen:** Ein Share erfordert zusätzlich Dateisystem- und App-API-Autorisierung.  
**Quellen:** [SHARING_SPACES.md](SHARING_SPACES.md).

#### ADR-145 – Share-Mounts im Benutzer-Home

**Status:** `Accepted`  
**Entscheidung / Stand:** Empfangene Freigaben werden als autorisierte Mount-/Sichtprojektion des zweiten Volumes in das eigene Home eingebunden.  
**Begründung und Folgen:** Der fremde Originalpfad und die Eigentümerhoheit bleiben geschützt.  
**Quellen:** [SHARING_SPACES.md](SHARING_SPACES.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-146 – Space-Rollen und Meta-Administration

**Status:** `Accepted`  
**Entscheidung / Stand:** Spaces besitzen eigene Mitglieder, Rollen, Quotas und Administratorfunktionen; globale Systemrollen ersetzen nicht die Space-Berechtigungen.  
**Begründung und Folgen:** Rollenentscheidungen folgen expliziten Ressourcen-Scopes.  
**Quellen:** [SHARING_SPACES.md](SHARING_SPACES.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

#### ADR-147 – Space-Root auf dem `users`-Volume

**Status:** `Accepted`  
**Entscheidung / Stand:** Space-Dateien liegen im zweiten vorhandenen virtuellen Volume unter unabhängigem Space-Owner und stabiler ID.  
**Begründung und Folgen:** Es wird weder ein drittes Volume noch ein global frei sichtbarer Space-Dateibaum eingeführt.  
**Quellen:** [SHARING_SPACES.md](SHARING_SPACES.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-148 – POSIX-DAC und zusätzliche Share-/Space-Grants

**Status:** `Accepted`  
**Entscheidung / Stand:** Share-/Space-Rechte wirken zusammen mit der VFS-Autorisierung und sind kein Root-Bypass.  
**Begründung und Folgen:** Der Core muss alle relevanten Rechtebedingungen bei jeder Operation prüfen.  
**Quellen:** [SHARING_SPACES.md](SHARING_SPACES.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

#### ADR-149 – Gruppenvererbung und unmittelbarer Membership-Entzug

**Status:** `Accepted`  
**Entscheidung / Stand:** Gruppen-/Space-Mitgliedschaften sind revisioniert und werden bei Entzug ohne alte Cache- oder Replay-Rechte durchgesetzt.  
**Begründung und Folgen:** Verteilte Berechtigungs-Caches müssen sicher invalidiert werden.  
**Quellen:** [SHARING_SPACES.md](SHARING_SPACES.md), [PUSH_REALTIME.md](PUSH_REALTIME.md).

#### ADR-150 – Share-Expiry, Ablehnung und Weiterfreigabe

**Status:** `Accepted`  
**Entscheidung / Stand:** Interne Shares können eine optionale Ablaufzeit mit Datum und Uhrzeit erhalten. Nach Ablauf verweigert der Go-Core weitere Zugriffe automatisch und serverseitig, auch über alte Mount-Referenzen, Cache oder Event-Replay. Ersteller und berechtigte Akteure können Freigaben widerrufen, Empfänger sie ablehnen. Weiterfreigabe durch Empfänger bleibt im MVP verboten.  
**Begründung und Folgen:** Der Ablauf oder Widerruf eines Share-Grants löscht keine Originaldateien. Der Status muss konsistent über alle Core-Replikate geprüft werden; bereits extern kopierte Daten sind naturgemäß nicht rückholbar.  
**Quellen:** [SHARING_SPACES.md](SHARING_SPACES.md), [ARCHITECTURE_DECISIONS_2026-11-16.md](ARCHITECTURE_DECISIONS_2026-11-16.md)

#### ADR-151 – Space-Lifecycle, Archivierung, Papierkorb und Quotas

**Status:** `Accepted`  
**Entscheidung / Stand:** Spaces besitzen einen eigenen Lebenszyklus mit Archivierung, kontrollierter Löschung, Quotas und Restore-Grenzen.  
**Begründung und Folgen:** Die technische Bereinigung muss auf gültige Dateiversionen und Blockreferenzen Rücksicht nehmen.  
**Quellen:** [SHARING_SPACES.md](SHARING_SPACES.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-152 – Cross-Scope-Move und Space-Konflikte

**Status:** `Accepted`  
**Entscheidung / Stand:** Verschieben zwischen privaten Bereichen und Spaces erfordert einen kontrollierten, autorisierten Workflow mit sicheren Fehlerrücknahmen.  
**Begründung und Folgen:** Berechtigungen, Eigentum, Domain-Key und neue Speicherreferenzen sind neu zu prüfen.  
**Quellen:** [SHARING_SPACES.md](SHARING_SPACES.md), [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-153 – Externe Identity-Provenienz und Attributautorität

**Status:** `Accepted`  
**Entscheidung / Stand:** Extern synchronisierte Konten und Gruppen behalten lokale stabile IDs und nachvollziehbare Attributquellen.  
**Begründung und Folgen:** Externe Verzeichniswerte dürfen lokale privilegierte Rechte nicht ohne ausdrückliche Policy überschreiben.  
**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

#### ADR-154 – LDAP-Sync-Intervalle und Sicherheitsgrenzen bei Massenänderungen

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Optionaler LDAP-Connector synchronisiert nur read-only via paginierte, idempotente Delta-Abgleiche und regelmäßig verifizierte Vollscans. Ausgangskadenz nach aktivierter Quelle: Delta alle 15 Minuten, vollständiger Abgleich alle 24 Stunden; Jitter, Backoff, zentraler Lease/Fencing-Owner und administrative Anpassbarkeit sind Pflicht. Ein unvollständiger Scan erzeugt niemals `missing`- oder Massen-Disable-Entscheidungen. Für Massenänderungen ist ein konfigurierbarer Guard mit konservativen Ausgangsschwellen (mehr als 5 % oder mehr als 20 betroffene Identitäten je verifiziertem Lauf) plus Dry Run, Audit und step-up-geschützter Freigabe vorgesehen; diese Werte sind vor Produktivfreigabe last-/betriebsseitig zu validieren. Explizit nachgewiesene sicherheitsrelevante Einzel-Entzüge werden zeitnah übernommen, ohne den Schutz vor fehlerhaftem Gesamtimport auszuhebeln.  
**Begründung und Folgen:** Filterfehler und leere/inkomplette LDAP-Antworten dürfen nicht zum Massenverlust lokaler Identitäten oder privilegierten Fehlzuordnungen führen.  
**Freigabe-/Nachweisgrenze:** Tests: leerer/abgebrochener Paged Scan, Filterwechsel, Source-ID-Tausch, Delta-/Vollscan-Konflikt, Burst-Änderungen, Abbruch, Wiederaufnahme, Quellenstale und Massenänderungs-Freigabe.  
**Einführungsprofil:** Optionaler LDAP-Connector, standardmäßig aus; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [PUSH_REALTIME.md](PUSH_REALTIME.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-154).

#### ADR-155 – Missing User, Deaktivierung und Stale-Identity-Richtlinie

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Ein Benutzer gilt erst nach einem vollständig erfolgreichen, konsistenten Quellabgleich als `missing`. Er wird dann kontrolliert quarantänisiert: externe Neuanmeldung und wirksame externe Grants werden gesperrt bzw. widerrufen, erforderliche Sitzungen revalidiert; lokale UUID/UID/GID, Home, Space- und Audit-Referenzen bleiben erhalten. Ein 30-tägiges administrativ änderbares Wiederherstellungsfenster trennt Reaktivierung und endgültige fachliche Deaktivierung; es ist keine automatische Datenlöschung. Wiederauftauchen derselben stabilen Quell-ID ermöglicht nach sicheren Prüfungen Reaktivierung, keine stille Identity-Fusion. Ein Ausfall/Stale-Scan ist niemals Nachweis einer Löschung; für sicherheitskritische Berechtigungen greift eine begrenzte Stale-Policy mit Fail-Closed, ohne pauschale Datenvernichtung.  
**Begründung und Folgen:** Deprovisioning muss sicher wirken, ohne Konten durch temporäre Providerstörungen oder falsche Filter unwiederbringlich zu zerstören.  
**Freigabe-/Nachweisgrenze:** Missing-vs-outage, 30-Tage-Recovery, Session-/Grant-Revoke, Offline-Leases, VFS-/Space-Erhalt, Reappear mit gleicher/anderer ID, massenhaft fehlende Benutzer.  
**Einführungsprofil:** Optionaler LDAP-Connector, standardmäßig aus; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-155).

#### ADR-156 – Import externer UID/GID oder interne Nummernvergabe

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Verbindlich ausschließlich lokal kontrollierte UID-/GID-Vergabe nach ADR-138: separate zentrale Nummernkreise, transaktional vergeben, einmalig, keine Wiederverwendung. Externe `uidNumber`/`gidNumber` werden als unverbindliche Quellmetadaten gespeichert und dürfen die lokale Nummer oder Dateieigentümerschaft nicht ersetzen. Es gibt auch unter expliziter Connector-Policy keine direkte externe POSIX-Nummernübernahme; eine spätere echte Migration wäre eine neue separat zu genehmigende Architekturänderung.  
**Begründung und Folgen:** Kein Konflikt mit bereits entschiedenem ADR-138; Verzeichniswechsel können lokale Dateirechte nicht neu zuordnen.  
**Freigabe-/Nachweisgrenze:** UID/GID-Kollisionen, gleichzeitiger Import, Rename, Restore, Tombstone, Quellenwechsel und Test, dass externe Zahländerungen nie VFS-Rechte umhängen.  
**Einführungsprofil:** Bei optionalem LDAP-Import, ADR-138 unverändert; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [ADR-138](ADR_REGISTER.md#adr-138--uid-gid-nummernkreise-und-wiederverwendung); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-156).

#### ADR-157 – Externe Gruppenverschachtelung und lokale Overlays

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Externe Gruppenverschachtelung ist für die erste optionale LDAP-Integration standardmäßig deaktiviert. Spätere explizite Aktivierung nutzt deterministisches, begrenztes Flattening mit maximal 8 Ebenen (Ausgangslimit), Zyklus-/Duplikaterkennung, Member-/Zeitbudgets und vollständiger Provenienz. Zu tiefes, zyklisches oder unvollständiges Nesting darf keine neuen Berechtigungen gewähren. Lokal administrierte Overlays bleiben gesonderte Gruppen oder explizite Ergänzungen mit eigener Herkunft; keine stille Überschreibung extern verwalteter Mitgliedschaften und kein implizites Mapping zur privilegierten `Admin`-Gruppe.  
**Begründung und Folgen:** LDAP-Gruppenrekursion darf weder Endlosschleifen noch unkontrollierte Privileg-Eskalation erzeugen.  
**Freigabe-/Nachweisgrenze:** Zyklen, Alias-/DN-Wechsel, tiefe Gruppen, stale Mitgliedschaften, lokale Overlay-Kollisionen, Admin-Impersonation, sofortige Revoke-Wirkung.  
**Einführungsprofil:** Nesting separat opt-in, erstes LDAP-Profil flach; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [SHARING_SPACES.md](SHARING_SPACES.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-157).

#### ADR-158 – LDAP-Bind-Login als optionaler Auth-Adapter

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** LDAP-Bind-Login bleibt separat vom LDAP-Verzeichnissync und standardmäßig deaktiviert. Eine spätere Aktivierung erfolgt pro Quelle und zulässiger Konto-Policy durch Administratoren. Zugang nur über streng geprüfte LDAPS-/StartTLS-Verbindungen, begrenzte Authentifizierungsversuche und ohne dauerhafte Speicherung/Protokollierung des Benutzerpassworts. Erfolgreicher Bind erstellt nur nach verifizierter lokaler ID-Bindung und lokaler Policy eine cloudOS-BFF-Sitzung. Kein unkontrollierter Fallback auf lokales Passwort bei LDAP-Ausfall; ein gesicherter lokaler Break-Glass-Admin bleibt unabhängig.  
**Begründung und Folgen:** Ein LDAP-Bind darf keine Fremd-Passwortkopie, zusätzliche Directory-Autorität oder stillen Identity-Link erzeugen.  
**Freigabe-/Nachweisgrenze:** TLS-Fehler, LDAP-Injection, unbekannte Bindung, Account-Lockout, Provider-Ausfall, Session-Revoke, Backoff, Audit ohne Credential-Leak.  
**Einführungsprofil:** Nach MVP, standardmäßig aus; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-158).

#### ADR-159 – Integrationsvertrag externer OIDC-Provider

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Optionaler providerunabhängiger OpenID-Connect-Adapter mit Authorization Code Flow, PKCE und vertraulichem serverseitigem BFF. Verbindlich sind exakte Issuer-/Audience-/Redirect-Validierung, `state`/`nonce`, signierte ID-Token-/JWKS-/Expiry-/Algorithmenprüfung, kontrollierte Schlüsselrotation sowie Bindung an `(issuer, sub)` und eine lokal freigeschaltete `user_id`. Keine automatische Provisionierung oder Rechteübernahme allein aufgrund E-Mail-/Gruppenclaims; neue Benutzer nur durch explizite Admin-Policy und sicheren Provisionierungsworkflow. Lokale Sitzung und Grant-/Revocation-Policy bleiben autoritativ; kein stiller Provider-Fallback.  
**Begründung und Folgen:** Login-Delegation darf die lokale AuthZ, POSIX-UID/GID und Space-/App-Policies nicht ersetzen.  
**Freigabe-/Nachweisgrenze:** Login CSRF/PKCE/nonce Replay, falscher Issuer/JWKS, Sub-Takeover, Provider-Ausfall, Admin-Gruppenclaim, Token-Rotation, Logout/Session-Stale und Recovery-Test.  
**Einführungsprofil:** Nach MVP, standardmäßig aus; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-159).

#### ADR-160 – Secret-Store und Zertifikatsverwaltung externer Connectoren

**Status:** `Accepted` (fachlich; delegierte Wahl vom 16.11.2026)  
**Entscheidung / Stand:** Ein zentraler Core-Secret-Broker verwaltet Connector-Credentials, Auth-Client-Secrets, Zertifikats-/CA-Referenzen und Rotation unter getrennten, minimalen Service-Capabilities. Verwaltet werden nur opake Handles und versionierte Metadaten in normalen Fachschemas; sensible Werte liegen KMS-/Envelope-geschützt in einem separaten Secret-Backend, dessen Root-Schlüssel nicht gemeinsam mit verschlüsselten Nutzdaten in derselben Datenbank liegt. Das Adaptermodell ist providerunabhängig und erlaubt später externe Secret-Manager/HSM. Zertifikatsketten und Hostnamen sind zu prüfen, unsichere Trust-Skip-Optionen produktiv verboten; Rotation atomar mit Test-/Rollback-Status und Audit. Kein Credential im App-Paket, Browser, Quellcode, Log oder normalen App-Data-Feld.  
**Begründung und Folgen:** Ein kompromittierter Connector darf nicht alle Core- oder Tenant-Secrets offenlegen; Secret-Lifecycle bleibt unabhängig vom LDAP-/IdP-Provider.  
**Freigabe-/Nachweisgrenze:** Secret-ACL-Isolation, Zertifikatsfehler, Trust-Store-Update, Wechsel während LDAP-Job, Rotation/Outage-Rollback, keine Secret-Logs/Traces/Exports, Key-Recovery mit getrennter Freigabe.  
**Einführungsprofil:** Grundlegende Broker-Schnittstelle im Design; optionaler Connector später; `Accepted` bedeutet nicht implementiert oder freigeschaltet.  
**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md); [Entscheidungsnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md#adr-160).

### 5.13 Konkrete neue Verträge aus 74 API-Entscheidungen (ADR-161–181)

#### ADR-161 – Nur eine aktive API-Major und kontrollierte Kompatibilität

**Status:** `Accepted`  
**Entscheidung / Stand:** Es gibt keine parallele Bereitstellung älterer API-Majors; inkompatible Drittanbieter-Apps bleiben installiert, werden aber deaktiviert. Inkompatible Pflicht-Core-Apps blockieren das Upgrade.  
**Begründung und Folgen:** Manifest-Capabilities, Deprecation, Sicherheitsausnahmen und Migrationsdokumentation sind Release-Gates.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-162 – Offizielles SDK, Capability-Filter und Entwicklerdokumentation

**Status:** `Accepted`  
**Entscheidung / Stand:** Apps verwenden ausschließlich das offizielle SDK; JavaScript, TypeScript und Rust zuerst. SDK-Pakete sind versioniert, zur Laufzeit capability-gefiltert und per Markdown dokumentiert.  
**Begründung und Folgen:** Externe IDEs zuerst; eine `Developer.cloudApp` ist optional später.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

#### ADR-163 – Ressourcenbezogenes Benutzer-Consent

**Status:** `Accepted`  
**Entscheidung / Stand:** Sensible Ressourcen benötigen ausdrückliche Nutzerzustimmung zusätzlich zur genehmigten App-API-Operation; Geltungsdauer hängt von Risiko und Ressource ab.  
**Begründung und Folgen:** Consent ist zentral widerrufbar und darf niemals neue API-Operationsrechte erzeugen.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

#### ADR-164 – Hierarchisches, dynamisches API-Rate-Limiting

**Status:** `Accepted`  
**Entscheidung / Stand:** Limits gelten fair auf System-, Service-, Nutzer-, App- und Operationsebene, auch über mehrere Replikate.  
**Begründung und Folgen:** 429-/Backpressure-/Retry-Semantik ist vertraglich, numerische Budgets bleiben konfigurierbare Policies.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-165 – Persistente Sagas und kontrollierter Fehlerzustand

**Status:** `Accepted`  
**Entscheidung / Stand:** Serviceübergreifende Abläufe verwenden durable Sagas mit lokalen ACID-Schritten, idempotenten Aktionen und Kompensation.  
**Begründung und Folgen:** Fehlerfall erfordert begrenzte Retries, Audit und bei Bedarf autorisierten Eingriff über `Tasks.cloudApp`.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md).

#### ADR-166 – Distributed Tracing über Core, Events und Jobs

**Status:** `Accepted`  
**Entscheidung / Stand:** OpenTelemetry und W3C Trace Context verbinden Serviceaufrufe, Jobs und asynchrone Events.  
**Begründung und Folgen:** Sensible Attribute bleiben ausgeschlossen; Sampling, Retention und rollenbasierte Einsicht erfolgen kontrolliert.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md).

#### ADR-167 – Umfassende Core-Metriken und Monitoring

**Status:** `Accepted`  
**Entscheidung / Stand:** OpenTelemetry-Metriken, Zeitreihen, historische Auswertungen, SLOs und Kapazitätsdaten sind Pflicht.  
**Begründung und Folgen:** Die Monitoringansichten respektieren Rollen, Dienstverfügbarkeit und Datenschutz.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-168 – Abgestufte automatische Selbstheilung

**Status:** `Accepted`  
**Entscheidung / Stand:** Sichere definierte Gegenmaßnahmen wie Neustarts, Failover, Skalierung und Job-Recovery können automatisch laufen; riskante Eingriffe erfordern Adminfreigabe.  
**Begründung und Folgen:** Begrenzte Reparaturversuche verhindern Endlosschleifen und Fehlheilung.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).

#### ADR-169 – Konfigurierbare Alarmierung, Eskalation und Wartungsfenster

**Status:** `Accepted`  
**Entscheidung / Stand:** Alarme über SSE, E-Mail und optionale Webhooks sind nach Schweregrad/Empfänger/Frist steuerbar; Wartungsfenster unterdrücken erwartete Alarme selektiv.  
**Begründung und Folgen:** Kritische Security-Alarme bleiben aktiv; Zustellung und Bestätigung sind persistent und auditiert.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [PUSH_REALTIME.md](PUSH_REALTIME.md).

#### ADR-170 – Rollenbasierte `Status.cloudApp`

**Status:** `Accepted`  
**Entscheidung / Stand:** Eine vorinstallierte Status-App zeigt Nutzern verständliche Funktionsverfügbarkeit und Admins technische Health-Details.  
**Begründung und Folgen:** Nicht mehr aktuelle Daten und SSE-Ausfall sind klar als solche anzuzeigen.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md).

#### ADR-171 – Workspace-Recovery, Auto-Save und Checkpoint-Historie

**Status:** `Accepted`  
**Entscheidung / Stand:** Bei Serviceausfällen bleiben Fenster möglichst erhalten; nach Browserabsturz werden App-/Fensterzustände aus hybriden ereignisgesteuerten Auto-Saves und serverseitigen Checkpoints wiederhergestellt.  
**Begründung und Folgen:** Geschützter lokaler Cache, Retention und Revisionsprüfung begrenzen Datenverlust ohne Erfolgsgarantie.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md).

#### ADR-172 – Zentrale Sitzungs-/Geräteverwaltung und Remote-Revocation

**Status:** `Accepted`  
**Entscheidung / Stand:** Benutzer sehen aktive Geräte und können einzelne oder alle Sitzungen widerrufen; Admin-Policies und neue Login-Benachrichtigungen sind vorgesehen.  
**Begründung und Folgen:** Der serverseitige Widerruf stoppt weitere API- und Desktop-Synchronisierung; Offline-Geräte unterliegen zeitlichen Leases.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

#### ADR-173 – Offline-First-Apps und sichere Journalsynchronisierung

**Status:** `Accepted`  
**Entscheidung / Stand:** Geeignete Core- und Drittanbieter-Apps deklarieren Offline-Fähigkeit und sammeln Änderungen im lokalen Journal; nach Reconnect wird autorisiert und, wenn möglich, via CRDT/OT zusammengeführt.  
**Begründung und Folgen:** Nicht unterstützte Aktionen bleiben onlinepflichtig; kein Recht entsteht durch offline gesammelte Änderungen.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md).

#### ADR-174 – Gerätegebundene Offline-Verschlüsselung und Quotas

**Status:** `Accepted`  
**Entscheidung / Stand:** Offline-Daten sind pro Gerät/Benutzer/App geschützt, lokal entsperrbar und nach Zeit-/Datenklassen-/Quota-Policies begrenzt; Ordner lassen sich pro Gerät auswählen.  
**Begründung und Folgen:** Nicht synchronisierte Änderungen werden bevorzugt geschützt; Browser-Speicher kann dennoch entfallen.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-175 – Adaptive Transfers, resumable Chunks und Delta-Sync

**Status:** `Accepted`  
**Entscheidung / Stand:** Große Dateien verwenden überprüfbare parallele Blöcke und atomaren Commit; Delta-Transfer nutzt adaptive inhaltsbewusste Blockerkennung.  
**Begründung und Folgen:** Ungesicherte Änderungen haben Vorrang, Transfers sind fortsetzbar und auf Integrität/Berechtigung geprüft.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-176 – Permanenter, versionierter und domänengetrennter Blockspeicher

**Status:** `Accepted`  
**Entscheidung / Stand:** Dateien werden als unveränderliche Blöcke mit versionierten Manifesten gespeichert; Deduplizierung erfolgt nur innerhalb privater Benutzer-/Space-Sicherheitsdomänen.  
**Begründung und Folgen:** Benutzerübergreifende Blockexistenz darf nicht ableitbar sein; logische und physische Quotas werden getrennt erfasst.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [SHARING_SPACES.md](SHARING_SPACES.md).

#### ADR-177 – Envelope Encryption, risikobasierte Rotation und Schlüssel-Recovery

**Status:** `Accepted`  
**Entscheidung / Stand:** Blockdaten sind je Sicherheitsdomäne verschlüsselt; versionierte Schlüssel werden geschützt, regelmäßig neu verpackt und bei Risiko neu verschlüsselt.  
**Begründung und Folgen:** Getrennte redundante Key-Backups und Mehrpersonen-Recovery sind Pflicht; Schlüsselverlust kann nicht magisch repariert werden.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [APP_SIGNING.md](APP_SIGNING.md).

#### ADR-178 – Block-Integritätsprüfung und kontrollierte Selbstheilung

**Status:** `Accepted`  
**Entscheidung / Stand:** Blockscrubbing beim Lesen und im Hintergrund erkennt Beschädigung und repariert nur aus separat geprüfter intakter Kopie.  
**Begründung und Folgen:** Fehlende Kopien führen zu sicherem Fehlerzustand und Alarm, nicht zu erfundener Wiederherstellung.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-179 – Transaktionssichere Block-GC und Papierkorb

**Status:** `Accepted`  
**Entscheidung / Stand:** Mark-and-Sweep mit Quarantänefrist, erneuter Referenzprüfung und Schutz aktiver Versionen/Snapshots ergänzt einen konfigurierbaren Papierkorb.  
**Begründung und Folgen:** Endgültige Löschung ist von Backup-Retention und bereits vorhandenen Offline-Kopien abzugrenzen.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

#### ADR-180 – Verpflichtende API-Qualitätsgates und Releasefreigabe

**Status:** `Accepted`  
**Entscheidung / Stand:** API-Schema-, Vertrags-, Sicherheits-, Berechtigungs-, Integrations-, Störfall-, Recovery- und Kompatibilitätstests sind Freigabepflicht.  
**Begründung und Folgen:** Fehlgeschlagene Sicherheits-Pflichtprüfungen blockieren Releases unabhängig von manuellem Wohlwollen.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

#### ADR-181 – Durable Jobs, `Tasks.cloudApp` und auftragssichere Wiederaufnahme

**Status:** `Accepted`  
**Entscheidung / Stand:** Kurze Operationen sind synchron; lange Arbeiten laufen als dauerhafte oder sitzungsgebundene Jobs mit SSE-Fortschritt und Anzeige in `Tasks.cloudApp`.  
**Begründung und Folgen:** Durable Jobs erfordern überprüfte Checkpoints, Leases, Idempotenz, begrenzte Retries und auditierte Abbruchrechte.  
**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md).


### 5.14 Ergaenzende Infrastrukturentscheidungen (ADR-182-189)

#### ADR-182 - Bedarfsgesteuerte Core-Auslagerung

**Status:** `Accepted`  
**Entscheidung / Stand:** Getrennte Core-Prozesse nur bei nachgewiesenem Engpass und validierten Port-/Sicherheitsvertraegen; kein automatischer Split, gemischte Topologien sind opt-in.  
**Begruendung und Folgen:** Ausdruecklich am 15.11.2026 entschieden; numerische technische Detailparameter bleiben abnahme- und lasttestpflichtig.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md).

#### ADR-183 - Extern verwaltete PostgreSQL-/JetStream-Hochverfuegbarkeit

**Status:** `Accepted`  
**Entscheidung / Stand:** Failover und Clusterverwaltung gehoeren der Infrastruktur; cloudOS behandelt Commit-Unklarheiten, Idempotenz, Konsistenz und sichere Wiederverbindung.  
**Begruendung und Folgen:** Ausdruecklich am 15.11.2026 entschieden; numerische technische Detailparameter bleiben abnahme- und lasttestpflichtig.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md).

#### ADR-184 - Fachlich getrennte JetStream-Streams und Consumer

**Status:** `Accepted`  
**Entscheidung / Stand:** Streams nach fachlicher Klasse; durable Consumer mit beschraenkter Berechtigung, Ack-/Retry-/Quarantaene-Regeln, at-least-once und idempotenter Wirkung.  
**Begruendung und Folgen:** Ausdruecklich am 15.11.2026 entschieden; numerische technische Detailparameter bleiben abnahme- und lasttestpflichtig.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md).

#### ADR-185 - Event-Retention und nachweisbare Gap-Recovery

**Status:** `Accepted`  
**Entscheidung / Stand:** Ereignisklassenspezifische Retention mit expliziter Gap-Erkennung; Projektionen aus autoritativen Owner-Zustaenden rekonstruieren, irreversible Effekte nicht blind wiederholen.  
**Begruendung und Folgen:** Ausdruecklich am 15.11.2026 entschieden; numerische technische Detailparameter bleiben abnahme- und lasttestpflichtig.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md).

#### ADR-186 - Outbox-Publish-Ack, Retention und Reconciliation

**Status:** `Accepted`  
**Entscheidung / Stand:** Publisher-Leases, persistierte JetStream-Publish-Bestaetigung, klassenspezifische Outbox-Retention und kontrollierter Abgleich nach Abstuerzen.  
**Begruendung und Folgen:** Ausdruecklich am 15.11.2026 entschieden; numerische technische Detailparameter bleiben abnahme- und lasttestpflichtig.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md).

#### ADR-187 - Kontrollierte Offline-Migration von Storage-Backends

**Status:** `Accepted`  
**Entscheidung / Stand:** Wartungsstopp, konsistente PostgreSQL-/Blob-/Key-Sicht, vollstaendige Integritaetspruefung, gesicherter Cutover; kein Dual-Write.  
**Begruendung und Folgen:** Ausdruecklich am 15.11.2026 entschieden; numerische technische Detailparameter bleiben abnahme- und lasttestpflichtig.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md).

#### ADR-188 - Providerunabhaengige Runtime-Secrets und Rotation

**Status:** `Accepted`  
**Entscheidung / Stand:** Geschuetzte Secret-Quellen fuer Compose/Split/Kubernetes, minimale Rechte, pruefbare Rotation, keine geheimen Werte in Git/Images/Configs.  
**Begruendung und Folgen:** Ausdruecklich am 15.11.2026 entschieden; numerische technische Detailparameter bleiben abnahme- und lasttestpflichtig.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md).

#### ADR-189 - Providerunabhaengige Edge-, Ingress- und TLS-Architektur

**Status:** `Accepted`  
**Entscheidung / Stand:** Nginx fuer Compose, alternative Ingress-Proxys bei gleichem Sicherheitsvertrag; HTTPS, mTLS intern, sichere Forwarded-Header und SSE ohne Buffering.  
**Begruendung und Folgen:** Ausdruecklich am 15.11.2026 entschieden; numerische technische Detailparameter bleiben abnahme- und lasttestpflichtig.  
**Quellen:** [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md).


## 6. Rückverfolgbarkeit älterer Fach-ADR-IDs

Fachkennungen bleiben unverändert als **Aliase**, nicht als zusätzliche unabhängige Architekturentscheidungen. Ein Alias kann mehreren globalen ADRs entsprechen, wenn die historische Fachfrage mehrere bereits beschlossene Aspekte vermischt hat. Dadurch bleiben alte Links und Diskussionen nachvollziehbar, ohne semantische Dubletten zu erzeugen.

| Alte Fachkennung | Globale ADR-Zuordnung | Inhalt |
|---|---|---|
| `VFS-ADR-001` | `ADR-009` | Volume-/Mount-Namensraum |
| `VFS-ADR-002` | `ADR-009`, `ADR-141` | Administrator- und Supportzugriffsrechte |
| `VFS-ADR-003` | `ADR-008` | Physische Datentrennung und Storage-Mounts |
| `VFS-ADR-004` | `ADR-131` | Namen, Normalisierung, Case-Policy |
| `VFS-ADR-005` | `ADR-132` | Standardstruktur des Home |
| `VFS-ADR-006` | `ADR-133` | Home-Provisionierung |
| `VFS-ADR-007` | `ADR-134` | Cross-Volume-Operationen |
| `VFS-ADR-008` | `ADR-135`, `ADR-179` | Quotas und Papierkorb |
| `VFS-ADR-009` | `ADR-136` | Backup und Restore der VFS-Daten |
| `VFS-ADR-010` | `ADR-016` | App-Dateihandles und Broker |
| `IAM-ADR-001` | `ADR-003` | Lokales Directory/optionale externe IdP |
| `IAM-ADR-002` | `ADR-137` | LDAPv3-Subset und TLS |
| `IAM-ADR-003` | `ADR-138` | UID/GID-Nummernkreise |
| `IAM-ADR-004` | `ADR-139` | Kontoschema und Gruppen |
| `IAM-ADR-005` | `ADR-140` | Passwortpolitik und Recovery |
| `IAM-ADR-006` | `ADR-009` | POSIX-DAC-Modusbits |
| `IAM-ADR-007` | `ADR-141` | Systemrollen versus Unix-Gruppen |
| `IAM-ADR-008` | `ADR-142` | Settings-Sub-Apps |
| `IAM-ADR-009` | `ADR-040`, `ADR-080` | Gruppen-/Session-Invalidierung |
| `IAM-ADR-010` | `ADR-133` | Home-Erstellung und Retention |
| `IAM-ADR-011` | `ADR-143` | Erweiterte POSIX-ACL |
| `IAM-ADR-012` | `ADR-003`, `ADR-103` | Externe LDAP-/OIDC-Integration |
| `ADR-COL-01` | `ADR-144` | Share-Rollen und Zugriffsänderungen |
| `ADR-COL-02` | `ADR-145` | Share-Mount-Sichten im Home |
| `ADR-COL-03` | `ADR-146` | Space-Rollen und Meta-Admin |
| `ADR-COL-04` | `ADR-147` | Space-Root auf zweitem Volume |
| `ADR-COL-05` | `ADR-148` | POSIX-DAC und Share-/Space-Grants |
| `ADR-COL-06` | `ADR-149` | Gruppenvererbung und Entzug |
| `ADR-COL-07` | `ADR-150` | Share-Expiry/Decline/Re-Sharing |
| `ADR-COL-08` | `ADR-151` | Space-Lifecycle, Papierkorb, Quota |
| `ADR-COL-09` | `ADR-037`, `ADR-039` | API-Verträge und App-Risiko |
| `ADR-COL-10` | `ADR-152` | Cross-Scope Move und Konflikte |
| `ADR-ID-01` | `ADR-153` | Identity-Mapping und Provenienz |
| `ADR-ID-02` | `ADR-154` | LDAP-Synchronisierungsintervalle |
| `ADR-ID-03` | `ADR-155` | Missing User und Deaktivierung |
| `ADR-ID-04` | `ADR-156` | UID/GID-Importentscheid |
| `ADR-ID-05` | `ADR-157` | Externe Gruppen und Overlays |
| `ADR-ID-06` | `ADR-158` | LDAP-Bind-Anmeldung |
| `ADR-ID-07` | `ADR-159` | Externe OIDC-Anbindung |
| `ADR-ID-08` | `ADR-103` | Sichere Accountverknüpfung |
| `ADR-ID-09` | `ADR-160` | Secret-Store und TLS-Zertifikate |

**Alle 41 Kennungen sind abgebildet.** Unter `ADR-131` bis `ADR-160` wurden nur jene zusätzlichen Sachverhalte aufgenommen, für die eine vorhandene globale Kennung keinen hinreichend eigenständigen fachlichen Kern abdeckt. Ein Verweis auf `Accepted` darf einen in der ursprünglichen Sammelfrage noch offenen Detailaspekt nicht verdecken.

## 7. Zuordnung aller 74 Entscheidungen aus `API_CONTRACTS.md`

Alle nachstehenden 74 Antworten sind **inhaltlich angenommen**. Die globale ADR-Zuordnung gruppiert eng zusammengehörige Entscheidungen; eine einzelne Frage muss daher keinen eigenen ADR erhalten. Ein später geplantes Erweiterungsprofil kann weiterhin `Proposed` bleiben, obwohl die heutige Entscheidung, dieses Profil zunächst **nicht** freizugeben, schon verbindlich ist.

| API-Frage | Beschlossene Regel (Kurzform) | Zuständige globale ADRs |
|---:|---|---|
| 01 | API nur für Apps/Core; externe Clients später | `ADR-013`, `ADR-003` |
| 02 | Nur eine Major-API aktiv | `ADR-006`, `ADR-161` |
| 03 | Drittanbieter-App inkompatibel / Baseline blockiert | `ADR-028`, `ADR-161` |
| 04 | SDK-only für Apps | `ADR-162`, `ADR-037` |
| 05 | Erweiterbare SDK-Sprachen JS/TS/Rust | `ADR-162` |
| 06 | Volle App-Isolation einschließlich Core-Apps | `ADR-031` |
| 07 | Network Broker für App-Egress | `ADR-041` |
| 08 | Sichere begrenzte Retries | `ADR-006`, `ADR-181` |
| 09 | Hybrid synchron/async Jobs mit SSE | `ADR-181` |
| 10 | Durable versus session-bound Jobs | `ADR-181` |
| 11 | `Tasks.cloudApp` | `ADR-181` |
| 12 | Optimistische Konfliktkontrolle | `ADR-006`, `ADR-040` |
| 13 | RFC-9457-Problem-Details | `ADR-006` |
| 14 | Lokale Go-Ports, gRPC und JetStream | `ADR-001`, `ADR-085`, `ADR-007` |
| 15 | mTLS und Service-Capabilities | `ADR-004`, `ADR-091` |
| 16 | Eingegrenzte signierte Delegation | `ADR-004`, `ADR-016` |
| 17 | Ressourcen-Consent | `ADR-016`, `ADR-163` |
| 18 | Consent-Dauer und Widerruf | `ADR-163` |
| 19 | Pflicht-Audit plus Diagnose-Logging | `ADR-104`, `ADR-106` |
| 20 | Aufbewahrung je Logkategorie | `ADR-107` |
| 21 | Rollenbasierter Log-Zugriff | `ADR-108` |
| 22 | Cursor-Pagination und Streams | `ADR-006`, `ADR-073` |
| 23 | Hierarchische Rate-Limits | `ADR-164` |
| 24 | Teilfehler isolieren und Failover | `ADR-083`, `ADR-096` |
| 25 | Job-Recovery und Checkpoints | `ADR-181`, `ADR-088` |
| 26 | At-least-once Events/Outbox/Replay | `ADR-007`, `ADR-074` |
| 27 | CloudEvents-1.0-Envelope | `ADR-074` |
| 28 | Serviceeigener API-Katalog | `ADR-006`, `ADR-037` |
| 29 | Contract-Releaseprüfung | `ADR-006`, `ADR-180` |
| 30 | Markdown-Entwicklerdokumentation | `ADR-162` |
| 31 | Externe IDE zuerst | `ADR-162` |
| 32 | Manifest-Kompatibilität | `ADR-161` |
| 33 | Deprecation mit Security-Ausnahme | `ADR-161` |
| 34 | Generiertes Changelog und Migration | `ADR-161`, `ADR-162` |
| 35 | Versionierte npm- und Rust-SDKs | `ADR-162` |
| 36 | Runtime-gefilterte SDK-Capabilities | `ADR-162` |
| 37 | Neue API-Rechte nur signiertes Update | `ADR-037`, `ADR-040` |
| 38 | Einzelwiderruf nur betroffener Funktionen | `ADR-040` |
| 39 | Effektive App-Rechte und SSE | `ADR-040` |
| 40 | Persistente Sagas | `ADR-165` |
| 41 | Saga-Fehler und Admin-Eingriff | `ADR-165` |
| 42 | OpenTelemetry-Traces/W3C-Kontext | `ADR-166` |
| 43 | Umfassendes Monitoring und SLOs | `ADR-097`, `ADR-167` |
| 44 | Abgestufte Selbstheilung | `ADR-168` |
| 45 | Alarmkanäle und Eskalation | `ADR-169` |
| 46 | Wartungsfenster mit Alarmsteuerung | `ADR-169` |
| 47 | Mehrstufige Health Checks | `ADR-096` |
| 48 | `Status.cloudApp` | `ADR-170` |
| 49 | Sitzung/Fenster bei Ausfall erhalten | `ADR-171` |
| 50 | Wiederherstellung mit Auto-Save | `ADR-171` |
| 51 | Hybrides intelligentes Auto-Save | `ADR-171` |
| 52 | Hybride Checkpoint-Speicherung | `ADR-171` |
| 53 | Konfigurierbare Historie | `ADR-171` |
| 54 | Synchroner logischer Mehrgeräte-Desktop | `ADR-010`, `ADR-112` |
| 55 | Geräte-/Sessionverwaltung und Widerruf | `ADR-172` |
| 56 | CRDT-/OT-Zusammenführung | `ADR-010` |
| 57 | Offline-First mit Merge | `ADR-173` |
| 58 | Abgestufte Offline-Fähigkeit | `ADR-173` |
| 59 | Gerätegebundene Offline-Verschlüsselung | `ADR-174` |
| 60 | Hierarchische Offline-Quotas | `ADR-174` |
| 61 | Selektiver Offline-Ordner-/Dateicache | `ADR-174` |
| 62 | Adaptive Bandbreitensteuerung | `ADR-175` |
| 63 | Parallele, fortsetzbare Blöcke | `ADR-175` |
| 64 | Adaptive Delta-Synchronisierung | `ADR-175` |
| 65 | Permanenter versionierter Blockspeicher | `ADR-176` |
| 66 | Sicherheitsdomänen-Deduplizierung | `ADR-176` |
| 67 | Mehrstufige Envelope Encryption | `ADR-177` |
| 68 | Risikobasierte Schlüsselrotation | `ADR-177` |
| 69 | Abgesichertes Schlüssel-Recovery | `ADR-177` |
| 70 | Kontrollierte Block-Selbstheilung | `ADR-178` |
| 71 | Transaktionssichere Garbage Collection | `ADR-179` |
| 72 | Konfigurierbarer Papierkorb | `ADR-179` |
| 73 | OpenAPI/Protobuf/AsyncAPI | `ADR-006` |
| 74 | Verbindliche API-Qualitätsprüfung | `ADR-180` |

**Vollständige Wortlaute der bestätigten 74 Fachentscheidungen:** [API_CONTRACTS.md](API_CONTRACTS.md), insbesondere der dortige Entscheidungskatalog und die Kapitel 1–20. Die Kurzformulierungen hier sind kein Ersatz für diese Detailverträge.

## 8. Erfasste Widersprüche und deren vorhandene Auflösung

Für die folgenden Konflikte ist **keine weitere Architekturfrage notwendig**, weil eine jüngere, ausdrücklich verbindliche Festlegung bereits vorliegt. Das Register dokumentiert das Ergebnis; veraltete Textstellen der betroffenen Ausgangsdokumente bleiben redaktionelle Folgearbeit.

| Kennung | Ältere abweichende Aussage / Quelle | Verbindliche Auflösung und zuständige ADRs | Folgearbeit |
|---|---|---|---|
| K-01 | WebSocket/WSS in älteren Protokollmatrizen, z. B. `SYSTEM_ARCHITECTURE.md`. | SSE ist der einzige reguläre Core-zu-Browser-Push-Transport; WebSocket allenfalls später für eigene bidirektionale Fälle (`ADR-073`, `ADR-076`). | Ältere WSS-MVP-Texte redaktionell auf SSE ändern. |
| K-02 | Pflicht-OIDC bzw. externer IdP als initialer Standard in frühen Architekturannahmen. | Lokaler Go-Core-Identity-/Auth-Pfad ist Standard; LDAP/OIDC sind optional (`ADR-003`, `ADR-159`). | Pflicht-Provider-Aussagen in Altabschnitten korrigieren. |
| K-03 | Annahmen, ältere API-Major-Versionen würden parallel während Übergangsfristen weiterlaufen. | Nur eine aktive API-Major; Breaking Changes erfordern kontrollierte Migration und App-Kompatibilitätsprüfung (`ADR-006`, `ADR-161`). | Parallel-Major-Formulierungen in älteren API-Kapiteln ersetzen. |
| K-04 | Getrennte Core-Service-Container als verpflichtender MVP-Betriebsmodus. | Ein Go-Prozess in einem Core-Container ist Default; Split und Kubernetes optional (`ADR-083`, `ADR-085`). | Topologiebeispiele als optionale Zielbilder kennzeichnen. |
| K-05 | Desktop-/Fensterkonzept spricht nur begrenzt über Offline-/Mehrgeräte-Verhalten. | Gemeinsamer geräteübergreifender Desktop, CRDT/OT für geeignete Daten und Offline-First sind beschlossen (`ADR-010`, `ADR-171`, `ADR-173`). | Ältere Desktop-/Window-Texte um neuere Invarianten ergänzen. |
| K-06 | Alte App-Grant-APIs könnten als Laufzeit-Erweiterung von beliebigen API-Operationsrechten missverstanden werden. | Weitere Core-API-Operationen benötigen ein signiertes App-Update und Adminfreigabe. Laufzeit-Consent gilt nur für bereits genehmigte Operationen (`ADR-037`, `ADR-040`, `ADR-163`). | API-Beispieltexte sprachlich zwischen Operationserlaubnis und Resource-Consent trennen. |
| K-07 | VFS-Beispiele beschreiben rein zusammengesetzte, vollständig gespeicherte Dateien. | Permanenter versionierter Immutable-Blockspeicher und Sicherheitsdomänen-Deduplizierung sind beschlossen (`ADR-008`, `ADR-176`). | Storage-Backend-Beispiele als historische Entwürfe markieren. |
| K-08 | Allgemeiner „öffentlicher API“-Titel kann externen Drittclient-Zugriff im MVP nahelegen. | REST ist intern über BFF/SDK erreichbar; externe eigene API-Clients sind zurückgestellt (`ADR-013`, `ADR-162`). | Einführungen und Endpoint-Texte um Zielgruppe ergänzen. |
| K-09 | Ältere ADR-Backlogs bezeichnen heute bereits entschiedene Themen weiterhin als „zu beschließen“. | Quellengestützte explizite Entscheidungen, vor allem die 74 API-Fragen, werden fachlich `Accepted`; Detailparameter bleiben offen. | Status in Fach-Backlogs an zentralen ADR-Index anbinden. |
| K-10 | Gleichzeitig „vollständige Echtzeit-Zusammenführung“ und technische Grenzen von Binärdateien. | CRDT/OT gilt für geeignete App-Datenmodelle; nicht sicher mergefähige Binärinhalte bekommen revisionsbasierte Konfliktbehandlung (`ADR-010`, `ADR-173`). | Keine unbedingte „alle Binärdateien automatisch mergen“-Garantie ausgeben. |
| K-11 | Kubernetes-Liveness/Failover erscheint auch für den combined-Einprozessbetrieb. | Mehrstufige Health-Zustände gelten logisch in beiden Profilen; echte Prozess-/Pod-Isolation erst bei getrennten Replikaten (`ADR-096`, `ADR-168`). | Dokumentation zwischen logischer Degradierung und physischem Failover präzisieren. |
| K-12 | At-rest-Verschlüsselung und Remote-Widerruf könnten als E2E oder sichere Sofortlöschung bereits offline kopierter Daten gelesen werden. | Envelope Encryption ist serverseitig; Offline-Widerruf wirkt am Server sofort und lokal erst nach Kontakt bzw. Offline-Lease (`ADR-172`, `ADR-174`, `ADR-177`). | Security-Grenzen in Nutzertexten ausdrücklich benennen. |

Die Auflösungen ersetzen nur **tatsächlich überholte Aussagen**. Sie berechtigen nicht dazu, weitere noch nicht genehmigte Produkt- oder Technologiewahlen stillschweigend zu ändern. In den analysierten Quellen wurde derzeit kein weiterer unauflösbarer Grundsatzkonflikt gefunden, für den in dieser Konsolidierung eine neue Nutzerentscheidung zwingend notwendig wäre.

## 9. Tatsächlich offene Entscheidungen und präzisierende Folgeaufgaben

Die delegierte Architekturentscheidung vom 16.11.2026 hat alle zuvor noch offenen globalen ADRs **außer ADR-094** fachlich abgeschlossen. Die 14 neu angenommenen ADRs sind im [vollständigen Beschlussnachtrag](ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md) dokumentiert. Auch die zuvor bereits vom Projektverantwortlichen bestätigten 13 ADRs sind weiterhin `Accepted`.

### 9.1 Einziger verbliebener globaler ADR (`Proposed`)

| ADR | Offenes Thema | Nächster geeigneter Schritt |
|---|---|---|
| `ADR-094` | Kubernetes-Produktionshärtung und Secrets | Wie ausdrücklich angeordnet, nicht entschieden; vor optionalem Kubernetes-Produktivbetrieb gesonderte Umsetzung/Abnahme, ohne bestehende Sicherheitsinvarianten aufzuweichen. |

**Einführung oder technische Freigabe ist ein anderes Gate als ADR-Annahme:** Noch ausstehende Messwerte, exakte DTOs, Vertragsartefakte, E2EE-Tests, Lizenzprüfungen und konkrete Browser-/WASI-Abnahmen sind keine Gründe, akzeptierte Architekturentscheidungen erneut zur Wahl zu stellen. Sie dürfen aber bis zum Nachweis nicht als implementiert oder produktionsbereit gelten.

### 9.2 Bereits akzeptierte Richtungen mit noch offenen Detailparametern

Diese Punkte sind **keine** offenen Produktalternativen. Sie benötigen technische Ausgestaltung und spätere Abnahme, nicht die Wiederholung bereits beschlossener Architekturfragen:

1. API-Operationen-/Event-Namenskatalog, die exakten OpenAPI-/Protobuf-/AsyncAPI-Schemas und die erlaubten SDK-Capability-Versionen.
2. Timeouts, Idempotency-/Delegation-TTLs, Rate-Limits, SSE-Retention, Worker-Leases, Backpressure und Quoten.
3. Mehrgeräte-/Offline-Datenmodelle: für welchen Dokumenttyp welcher CRDT-/OT-Algorithmus zulässig und getesteter Fallback ist.
4. Browserabdeckung für geschützte lokale Schlüssel, Entsperrverfahren, Offline-Gültigkeit und irreversible lokale Speicherbereinigung.
5. Chunkgrößen, Block-/Manifest-Schema, Domain-Deduplizierung, Content-Defined Chunking, Proof-of-Ownership-Abwehr und Quota-Berechnung.
6. Verschlüsselungsalgorithmen, Key-/Nonce-Ableitung, Rotation, Schutz von Recovery-Geheimnissen und Vier-Augen-Freigaben.
7. Monitoring-Schwellen, Eskalationsfristen, sichere automatische Reparatur-Policies, Health-/HPA-SLOs und Lasttestwerte.
8. Retention und Restore von Papierkorb, App-Checkpoints, Snapshots, Backup-Generationen, rechtlichen Sperren und Logklassen.
9. Exakte Core-App-Baseline-Releases für `Tasks.cloudApp`, `Status.cloudApp`, `Logs.cloudApp` und `Monitoring.cloudApp`.
10. Belastbare Einführungspfade für optionalen Kubernetes-Split, externen IdP und externe API-Clients, ohne die kombinierten MVP-Regeln zu verwässern.

## 10. Sicherheits- und Qualitätsinvarianten über alle ADRs hinweg

1. **Default Deny:** Keine App-, Broker-, Session-, Service-, Share- oder VFS-Grenze darf durch fehlende Prüfung offenstehen.
2. **Signing und wirksame Rechte:** Unsignierte/defekte Apps sind gesperrt. Neue API-Operationen erfordern signiertes Update und Adminfreigabe; Nutzer-Consent betrifft nur konkrete bereits erlaubte Ressourcen.
3. **SSE statt Polling:** Keine periodischen Client-Refresh-Schleifen; notwendige einmalige Snapshots bleiben erlaubt.
4. **Single-Core-Container als Default:** Physische Microservice-Skalierung und Kubernetes sind optionale, kompatible Ausführungsprofile.
5. **Zwei VFS-Volumes:** Genau `system` und `users`; Home-`/` ist ein gemounteter Benutzerausschnitt, kein drittes Laufwerk.
6. **Dauerhafte Fachwahrheit:** Wichtige Zustände liegen in autoritativer Persistenz und sind nicht an ein Browserfenster oder einen einzelnen Pod gebunden.
7. **Kontrollierte Wiederholungen:** Events dürfen mehrfach zugestellt werden; API-Schreibwirkungen, Datei-Commits und Saga-Schritte sind gegen Duplikate abgesichert.
8. **Sicherheitsrevisionen:** Widerruf wirkt serverseitig. Offline-Geräte können nicht über nicht vorhandene Verbindung sofort entfernt werden; Freigaben müssen zeitlich beschränkt sein.
9. **Blockspeicher-Sicherheit:** Keine domänenübergreifende Blockexistenz-Auskunft; keine Neuverschlüsselungs-/Recovery-Erfolgsmeldung ohne verifizierte Kopie und benötigte Schlüssel.
10. **Release-Gates:** Verbindliche Schema-, Berechtigungs-, Integrations-, Fehler- und Security-Tests blockieren fehlerhafte Veröffentlichungen.

## 11. Arbeitsweise zur Pflege des Registers

### 11.1 Neuen ADR-Kandidaten erstellen

1. Prüfen, ob ein bestehender globaler ADR das Thema bereits abdeckt; Fachalias nicht zu eigenständigem Duplikat machen.
2. Falls wirklich neu, nächste freie neue globale ID **oberhalb des bisher höchsten verwendeten Werts** vergeben; historische IDs 042–050 bleiben reserviert.
3. Titel, Kontext, Optionen, Folgen, Sicherheitsaspekte, betroffene Fachkonzepte und den `Proposed`-Status dokumentieren.
4. Nur nach ausdrücklicher fachlicher Bestätigung in `Accepted` überführen; Freigabenachweis verlinken.
5. Falls umfangreich, zusätzlich ein Detail-ADR unter `docs/adr/` anlegen; das Register bleibt der zentrale Index.

### 11.2 Bestehenden ADR ändern

Eine reine Textkorrektur darf keine fachliche Aussage verschieben. Bei geänderter Architektur wird die alte Entscheidung mit ihrem Geltungszeitraum erhalten und eine **ausdrücklich bestätigte Nachfolge-ADR** angelegt. `Superseded` wird erst gesetzt, wenn der Nachfolger tatsächlich angenommen ist; anschließend werden Verweise in betroffenen Fachkonzepten korrigiert. Für eine bloße Berichtigung eines irrtümlich übernommenen Registereintrags wird der Nachweis mit Korrekturhinweis offen dokumentiert, niemals Geschichte gefälscht.

### 11.3 Automatischer Dokumentationsabgleich (Konzept, noch keine Implementierung)

Eine spätere Dokumentationsprüfung soll doppelte IDs, nicht existente Quellreferenzen, noch offene Fachalias-Zuordnungen, widersprüchliche normative Wörter, fehlende Nachfolger von `Superseded` und neue unbelegte Sicherheitsbehauptungen melden. Diese Prüfung **darf keine ADRs automatisch akzeptieren**.

### 11.4 Release- und Architektur-Review

Bei jeder wesentlichen Core-/SDK-/VFS-Änderung werden die betroffenen `Accepted`-ADRs und aktuellen Fachkonzepte auf Vertragskompatibilität geprüft. Bei einer Abweichung gilt: Änderung zurückstellen, Fachentscheidung ausdrücklich neu treffen, Migration und Negativtests dokumentieren, Register und Fachkonzept gemeinsam aktualisieren.

## 12. Abnahme des Register-Konzepts

- [x] Vier Organisationsentscheidungen der ADR-Konzeption umgesetzt (ID-Politik, Quellenstatus, hybrides Format, Freigabeprozess).
- [x] Alle 121 älteren globalen Kandidaten unverändert unter ihrer ID erfasst.
- [x] Die historische Lücke 042–050 ohne Umwidmung erhalten.
- [x] 41 fachliche Altkennungen mit globalen ADRs verknüpft.
- [x] 51 neue globale IDs ab 131 für eindeutig zusätzliche Fach-/API-Themen zugeordnet.
- [x] Alle 74 Entscheidungen aus `API_CONTRACTS.md` zu globalen ADRs verknüpft.
- [x] `Accepted` fachlich von „implementiert“ und „getestet“ getrennt.
- [x] Erkennbar veraltete allgemeine Aussagen ohne neue Nutzerfragen aufgelöst und als Folgearbeiten sichtbar gemacht.
- [x] Nach ausdrücklicher Delegation 14 bisher offene ADRs fachlich entschieden; ausschließlich ADR-094 bleibt `Proposed`. Aktivierungs-/Test-/Lizenz-Gates bleiben offen.
- [ ] Formale Einzel-Freigabe eventuell neu erstellter vollständiger Detail-ADR-Dateien (entsteht erst bei Bedarf).
- [ ] Technische Implementierung, Testausführung und Release-Abnahme (nicht Bestandteil dieses Konzeptdokuments).

**Ende des zentralen ADR-Registers.** Die Quelle ist der geprüfte Konzeptstand vom 15.11.2026; zukünftige Fachentscheidungen sind als neuer, nachvollziehbarer Änderungsstand einzupflegen.
