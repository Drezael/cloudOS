# cloudOS – Settings und geschützte Sub-Apps (MVP)

> **Stand:** 16.11.2026  
> **Status:** Architekturentscheidung, keine implementierte Anwendung.  
> **Fachentscheidung:** [ADR-142, Option A](ADR_REGISTER.md), [Beschlussnachtrag](ARCHITECTURE_DECISIONS_2026-11-16.md).  
> **Abhängigkeiten:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [APP_API_ACCESS.md](APP_API_ACCESS.md), [WINDOW_FRAMEWORK.md](WINDOW_FRAMEWORK.md).

## 1. MVP-Umfang

`Settings.cloudApp` ist die zentrale Einstellungs- und Navigationsoberfläche, keine universell privilegierte Verwaltungs-App. `UserManagement.cloudApp` und `GroupManagement.cloudApp` sind separate, signierte Pflicht-Core-Apps. `PermissionManagement.cloudApp` ist eine **spätere** Erweiterung und nicht Voraussetzung für die Freigabe des MVP. Zwingend notwendige administrative Rollen-/Gruppenoperationen werden über eigenständige, kontrollierte Go-Core-Verträge, nicht über implizite Settings-Rootrechte umgesetzt.

## 2. Verantwortungs- und Berechtigungsgrenzen

| Komponente | Aufgabe | Vertrauen und API-Berechtigung |
|---|---|---|
| Settings.cloudApp | Navigation, Profileinstieg und geschützte Deep Links | Nur eigene signierte API-Allowlist |
| UserManagement.cloudApp | Benutzer, Kontostatus, sichere Reset-Workflows | Eigene signierte App-/Instanzidentität, passende IAM-/Step-up-Rechte |
| GroupManagement.cloudApp | Gruppen, Mitgliedschaften, Primärgruppenzuordnung | Eigene signierte App-/Instanzidentität, passende IAM-/Step-up-Rechte |
| PermissionManagement.cloudApp | Zusätzliche Rollen-/Policy-Verwaltungsansicht | Nach MVP, nicht über die Settings-Identität simulieren |

Eine eingebettete oder über einen Deep Link geöffnete Sub-App kann niemals die Elternrechte erweitern; ebenso darf die Eltern-App keine Rechte ihres Kindes erben. Berechtigungsprüfungen erfolgen in der Core-API für jede Aktion auf Benutzer-, App-Instanz-, Operationen- und Ressourcenscope-Ebene, auch wenn die UI einen Button versteckt.

## 3. Navigations- und Fenstervertrag

- Deep Links sind **versioniert** und enthalten nur überprüfte Routen-/Objektparameter. Die Ziel-App wird als registrierte, signierte App-Instanz geladen, nicht als frei austauschbarer privilegierter iframe-Inhalt.
- Der trusted Window Host kontrolliert Frame, Fokus, Lifecycle und sichere Dialoge. Browser-Origin-/Message-Bridge-Isolation bleibt erhalten.
- Fehlende App-Zuweisungen, ungültige Deep Links oder ein Widerruf führen zu einer kontrollierten Zugriffsverweigerung, nicht zu privilegierter Umleitung.
- Alle Oberflächen werden als cloudOS-Core-UI mit Vue 3 und TypeScript entwickelt, Vite ist bevorzugt; das öffentliche SDK bleibt frameworkunabhängig (ADR-002).
- Accessibility und Browserverhalten werden nach der verbindlichen Matrix von ADR-060 geprüft.

## 4. Sicherheits- und Abnahmekriterien

1. Ein normaler User kann die für ihn erlaubten persönlichen Settings sehen, aber keine administrative Child-App allein durch direkte URL öffnen.
2. Ein berechtigter Administrator kann UserManagement und GroupManagement über versionierte Navigation starten; jede App besitzt ihre eigene geprüfte Signatur-/API-Identität.
3. Ein Child-App-Token oder eine API-Allowlist ist aus Settings weder lesbar noch anderweitig übertragbar.
4. Logout, Konto- oder Rollenentzug sperren neue Operationen unmittelbar; veraltete UI-/Browsercaches setzen keine Rechte durch.
5. Sensible Änderungen sind auditierbar und verlangen gegebenenfalls vorgeschriebene Step-up-Sicherheit.
6. Fehlt die spätere PermissionManagement-App, bleiben bereits festgelegte Sicherheits- und Recoveryregeln des Core gültig.

## 5. Bewusst offen

Konkrete Deep-Link-Routenschemas, Manifestfelder, Fensterkonfigurationsparameter und vollständige UI-Mockups benötigen die spätere Contract- und Release-Abnahme. Aus dem Verzicht auf die PermissionManagement-MVP-App entsteht kein stillschweigender Freibrief für administrative API-Nutzung.
