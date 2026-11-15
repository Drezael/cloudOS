# cloudOS – App Framework SDK

> **Pfad:** `docs/APP_FRAMEWORK_SDK.md`  
> **Projekt:** [Drezael/cloudOS](https://github.com/Drezael/cloudOS) · Branch `main`  
> **Stand:** 15.11.2026  
> **Status:** Konzept und dokumentierte Konfliktbeschlüsse K1–K9; **nicht** als abgeschlossener Architekturreview, API-Implementierung oder verifizierte Implementation Readiness zu verstehen.  
> **Entscheidungsbasis:** 156 in der Fragerunde behandelte Architekturfragen. Die historischen Originalantworten der frühen Fragen sind noch nicht vollständig nachgewiesen.

## 1. Zweck

Dieses Dokument ist der **zentrale Einstiegspunkt** in das App Framework SDK von cloudOS. Es ersetzt weder `docs/APPLICATION_SYSTEM.md` noch `docs/APP_API_ACCESS.md`, `docs/APP_SIGNING.md` oder `docs/API_CONTRACTS.md`. Die zugehörigen Dateien unter `docs/app-framework/` strukturieren die neuen SDK-spezifischen Verträge, Entscheidungsnachweise und Prüfkriterien. Bis zum Abschluss des Repository-weiten Reviews bleiben nicht aufgelöste Konflikte ausdrücklich offen.

cloudOS ist ein browserbasierter Desktop mit einem **Go-Core**, API-first-Systemfunktionen und installierbaren, signierten `.cloudApp`-Paketen. Die produktive Standardbereitstellung ist der kombinierte Core; die fachlichen Servicegrenzen müssen auch bei optionaler Trennung gleich bleiben. Installierte Apps verwenden freigegebene SDK- und Core-APIs, nicht implizit privilegierte interne Interfaces. Der signierte Endpoint-Katalog `permissions/api-access.json` und die serverseitige Genehmigung bleiben verbindlich.

## 2. Dokumentationsnavigation

| Datei | Zweck | Reife |
|---|---|---|
| [01-architecture-baseline.md](app-framework/01-architecture-baseline.md) | SDK-Gesamtarchitektur und Invarianten | Konsolidierungsentwurf |
| [02-decision-register.md](app-framework/02-decision-register.md) | Entscheidungen Q001–Q156, Quellenlage und Verifikation | Teilweise historisch zu verifizieren |
| [03-component-catalog.md](app-framework/03-component-catalog.md) | Dienste, Verantwortlichkeiten, Vertrauensgrenzen | Konzeptionell |
| [04-interface-contracts.md](app-framework/04-interface-contracts.md) | SDK- und Dienstgrenzen, Semantik | Vertragsentwurf, keine realen Routen |
| [05-security-architecture.md](app-framework/05-security-architecture.md) | Identity, Capabilities, Sandbox, Code, Secrets | Review ausstehend |
| [06-data-consistency.md](app-framework/06-data-consistency.md) | Daten, Transaktionen, Outbox/Inbox, Sagas | Vertragsentwurf |
| [07-runtime-lifecycle.md](app-framework/07-runtime-lifecycle.md) | App-, Browser-, Worker- und Release-Lifecycle | Vertragsentwurf |
| [08-dependency-matrix.md](app-framework/08-dependency-matrix.md) | Abhängigkeiten und Implementierungsfolge | Vorläufig |
| [09-risk-register.md](app-framework/09-risk-register.md) | Konkrete Konflikte und Prüflücken | Offen |
| [10-implementation-readiness.md](app-framework/10-implementation-readiness.md) | Definition of Ready und Prüf-/Abnahmeplan | Nicht freigegeben |
| [11-architecture-decisions-K1-K9.md](app-framework/11-architecture-decisions-K1-K9.md) | Wortlaut der neun ausdrücklich beschlossenen Konfliktlösungen | Konzeptionell beschlossen |
| [12-legacy-alignment.md](app-framework/12-legacy-alignment.md) | Abgleichsmatrix zur späteren Volltextintegration führender Bestandsdokumente | Volltextabgleich ausstehend |

## 3. Verbindliche Eckpunkte der Fragerunde

1. Alle App-Operationen erfolgen über freigegebene Core-/SDK-Verträge mit serverseitigem Default Deny. Eine App-Signatur allein gewährt keine Rechte.
2. App-Code bleibt vom privilegierten Go-Core isoliert. Ausführbarer Code muss einem verifizierten Paket und Release zugeordnet werden; fremde HTML-Inhalte erhalten keine App-Privilegien.
3. Externe Zugangsdaten werden zentral und app-spezifisch freigegeben; ein Secret Broker vermeidet Klartextfreigabe an Apps soweit möglich.
4. Der App Data Service kontrolliert Datenhoheit, Schreibzugriffe, Revisionen, Constraints und Migrationen. Dienstübergreifende Vorgänge nutzen Sagas statt einer fiktiven globalen ACID-Transaktion.
5. Messaging verwendet mindestens At-least-once-Semantik; Outbox/Inbox, Idempotenz, begrenztes Event-Replay und autoritative Resynchronisation sind vorgesehen.
6. UI-Aufrufe haben definierte Deadlines/Cancellation; persistente Jobs und Sagas besitzen eigene Recovery-Verträge. Cancellation ist kein Rollback.
7. Bei Sicherheitsfehlern wird fail closed gehandelt; optionale Fachfunktionen dürfen kontrolliert degradieren.
8. App-Releases erfordern Signatur, Integrität, Berechtigungsreview, Konformitätsprüfung und Freigabe. SDK-Dokumentation ist versioniert und prüfbar.
9. Browsernatives Clipboard ohne SDK-Synchronisation; keine App-Suchanbieter; keine App-zu-App-Plugin-Extension-Points; keine externen Gastlinks – diese ausdrücklich ablehnenden Entscheidungen werden gewahrt.
10. Geographische Data-Residency-Policy ist **nicht** Bestandteil des SDK (Q130); Betreiberverantwortung bleibt davon unberührt.

## 4. Bereits bestehende fachlich führende Referenzen

- [CONCEPT.md](CONCEPT.md) und [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md): Produkt-/Core-Architektur.
- [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md): `.cloudApp`-Format, Installation, App-Lifecycle.
- [APP_SIGNING.md](APP_SIGNING.md): X.509-Signing, Vertrauensklassen und zulässige Admin-Ausnahmen.
- [APP_API_ACCESS.md](APP_API_ACCESS.md): signierte `permissions/api-access.json`, Endpoint-Allowlist und API-Enforcement.
- [API_CONTRACTS.md](API_CONTRACTS.md): API-Versionierung, Fehler-, Event-, Ressourcen- und Sicherheitssemantik.
- [PUSH_REALTIME.md](PUSH_REALTIME.md): regulärer Browser-Push über SSE, Replay und Push-/Notifications-Konzept.
- [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md): Combined-Core als Standard, Split/Kubernetes opt-in.
- [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md): Identität und Anmeldung.
- [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md), [SHARING_SPACES.md](SHARING_SPACES.md): Dateisystem und Berechtigungen.
- [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md), [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md): UI-Host und Betriebsdiagnose.

## 5. Status und Vorrang bei Widersprüchen

Die Dateien beschreiben **Zielverträge**, keine bereits verfügbaren Endpunkte. Bei Widersprüchen gilt **kein automatischer pauschaler Vorrang nach Dateidatum**: Die fachliche Owner-Grenze, die konkrete dokumentierte Entscheidung und die Sicherheitswirkung werden in [09-risk-register.md](app-framework/09-risk-register.md) erfasst und über einen expliziten ADR/Freigabeschritt geklärt. Insbesondere darf ein jüngerer SDK-Entwurf nicht stillschweigend die verbindliche SSE-Transportentscheidung oder den Signing-Vertrauenspfad ersetzen.

## 6. Beschlossene Konfliktlösungen K1–K9

Die ursprüngliche Fragerunde mit 156 Architekturfragen ist abgeschlossen. Danach wurden am 15.11.2026 **neun weitere konkrete Konfliktentscheidungen** explizit mit **Option C** getroffen. Die vollständigen normativen Beschlüsse und sicherheitsrelevanten Abgrenzungen stehen ausschließlich im zentralen [Beschlussnachweis K1–K9](app-framework/11-architecture-decisions-K1-K9.md). Alle SDK-Teildokumente verweisen auf diese Entscheidungen; führende Altverträge behalten ihre fachliche Eigentümerschaft.

| Konflikt | Geltender Architekturgrundsatz |
|---|---|
| K1/R-002 | Self-signed nach separater Admin-Herkunftsrisikoannahme, aber niemals ohne verpflichtende technische Prüfung |
| K2/R-003 | JS/TS und Browser-WASM regulär; isoliertes WASI später; Native nur optional nach eigener Abnahme |
| K3/R-005 | Separate App-Origins, Host-gesteuerte iFrames/Bridge und zentral verwaltete Service Worker |
| K4/R-001 | Genau eine aktive Core-API-Major, mehrere kompatible App-Releases innerhalb eines Übergangsfensters |
| K5/R-004 | Core SSE, externe SSE/WS und Web Push sind klar getrennte, kontrollierte Verträge |
| K6/R-006 | Serverseitige Verschlüsselung regulär; echte E2EE optional ohne serverseitige Klartextverarbeitung |
| K7/R-010 | Separate Retention-Klassen, erneute Rechteprüfung beim Replay, kontrollierter Resync |
| K8/R-011 | Nach Widerruf keine alten Saga-Rechte; Kompensation mit eigener, minimaler Recovery-Autorisierung |
| K9/R-014 | Release-/Digest-gezielter Sicherheits-Kill-Switch, Quarantäne und nur sicherer Rollback |

**Abnahmegrenze:** Konzeptionell entschieden bedeutet **nicht** bereits implementiert, technisch verifiziert, formal freigegeben oder in sämtliche bestehenden cloudOS-Bestandsdokumente integriert.

## 7. Nächster Arbeitsschritt

1. Die [Abgleichsmatrix](app-framework/12-legacy-alignment.md) gegen die vollständigen aktuellen Bestandsdokumente bearbeiten, ohne bestehende Normen stillschweigend zu überschreiben.
2. Q001–Q033 historisch verifizieren und R-008/R-009/R-013/R-015 weiter prüfen.
3. Fachliche Schnittstellen, technische Tests und Definition of Ready pro Komponente konkretisieren.
4. **Erst nach Abnahme** die Implementierungsreihenfolge freigeben. Dieses Paket enthält **keinen Implementierungscode**.
