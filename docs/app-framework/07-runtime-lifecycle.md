# Runtime & Lifecycle Contracts – App Framework SDK

> **Status:** Ziel-Laufzeitvertraege; keine Implementierung oder Browsergarantie.

## 1. App-Lifecycle

```mermaid
flowchart LR
  I[Installiert / geprueft] --> A[Zugewiesen]
  A --> R[Initialisierung]
  R --> V[READY]
  V --> D[DEGRADED]
  D --> V
  V --> S[SUSPENDED]
  S --> V
  V --> C[Schliessen]
  C --> X[Beendet]
```

Installiert bedeutet nicht aktiv, zugewiesen bedeutet nicht privilegiert und gestartet bedeutet nicht READY. Core-App-Baseline-Schutz, Zuweisung und Signing sind vor Aktivierung zu pruefen. Nach Widerruf keine weiteren geschuetzten Operationen.

## 2. Fenster, Unsaved Work und Zustand

Fenster erhalten eigene Instanz-/Fensteridentitaet. Beim Schliessen beruecksichtigt der Host `CLEAN`, `DIRTY`, `SAVING`, `SAVE_FAILED`, `CONFLICT`. Ein privilegierter Core-Dialog kann vor drohendem Datenverlust warnen; Sicherheitswiderruf darf dadurch nicht blockiert werden. Persistente Snapshots sind versioniert, Berechtigungen nicht dauerhaft lokal gecacht.

## 3. Browser-Background und Tabs

Service Worker sind Core-verwaltet; Origins und Service-Worker-Scopes bleiben technisch getrennt. Browser-Lifecycle ist nicht verlässlich planbar. App-UIs dürfen kurze Aufrufe abbrechen; lang laufende Arbeit wird als persistenter Serverjob definiert. Tab-Leader-Leases sind auf eine Aufgabe begrenzt und nutzen Fencing; sie ersetzen keine serverseitige Autorisierung.

## 4. Worker- und Job-Lifecycle

Worker erhalten signierten/verifizierten Code und kurzlebige gebundene Execution Identity. Start -> autorisierter Schritt -> Commit/Fehler -> Retry/Compensation/Completion. Jeder Schritt beachtet Quotas, Deadlines und gueltige Rechte. Permanente Hintergrundverbindungen aus externen SSE/WebSockets sind lifecycle-gesteuert und nicht identisch zum regulaeren cloudOS-Browser-Push ueber SSE.

## 5. SDK-Aufrufkontext

Normativer Zusammenhang: `operation_id`, `correlation_id`, `deadline`, Cancellation, Owner-/Tenant-/User-/App-/Release-Kontext, Resource Scope. Kein freies Client-Claim wird zum Autorisierungsnachweis. Fehler: `CANCELLED`, `DEADLINE_EXCEEDED`, `OUTCOME_UNKNOWN`, Konflikt, Forbidden, Unavailable, Validation. Bei unbekanntem Commit: Statusaufklaerung/Idempotenz statt blindem Retry.

## 6. Browser-Benachrichtigung

Notification Service ist Owner; Web Push wird nur bei expliziter Benutzer- und Browserfreigabe verwendet. Push darf vertrauliche Inhalte nicht unnoetig offenlegen. Klicks fuehren ueber autorisierte Deep Links, nicht direkt auf priviligierte Daten. Keine garantierte Browserzustellung.

## 7. Release-Lifecycle

Paket verifiziert -> Konformitaetspruefung -> Policy-/Risikoentscheidung -> aktiviert -> versionsgebundene Instanzen -> Update-/Uebergangsphase -> alte Instanz beendet. Sicherheitssperren haben Vorrang vor komfortabler Mehrversionsphase. Bestehende selbstsignierte Admin-Freigaben sind nur mit explizit abgestimmtem Conformance-Vertrag zu interpretieren.

## 8. Runtime-Degradation

| Zustand | Trigger | Erlaubtes Verhalten |
|---|---|---|
| READY | Alle zwingenden Dependencies gueltig | Normalbetrieb |
| DEGRADED | Optionale Dependency fehlt | Eingeschraenkte sichere Funktion |
| READ_ONLY | Geschuetzte Schreibfunktion nicht verfuegbar | Nur nachweislich autorisierte Leseoperationen |
| UNAVAILABLE | Sicherheitskritischer Owner fehlt | Betroffene Operationen fail closed |
| RECOVERING | Dienst wieder erreichbar, Konsistenz ungeprueft | Kontrollierte Reinitialisierung/Resync |

## 9. Diagnostik und Support

Versionierte Diagnosetests `PASS`, `WARN`, `FAIL`, `SKIPPED`, `UNAVAILABLE` unter eingeschraenkter Ausfuehrung. Supportexporte enthalten nur autorisierte und redigierte technische Daten, niemals Secrets oder ungefilterte Nutzerdokumente. Eine Diagnose ist keine Reparaturberechtigung.

## 10. Ausfalltestmatrix

Mindestens: Tab-Absturz, Service-Worker-Ende, Worker-Crash, Berechtigungswiderruf waehrend Auftrag, Netzwerkunterbruch nach Datenbank-Commit, abgeschalteter Provider, inkompatibles Alt-Release, nicht verfuegbarer Auditpfad und teilweise restaurierter Datenbestand. Weitere Details erst nach Repo-Abgleich und konkreten API-Vertraegen.
