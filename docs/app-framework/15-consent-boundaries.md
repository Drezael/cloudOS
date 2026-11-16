# R-009 – Purpose-gebundener Consent und Trust Boundaries

> **Stand:** 16.11.2026 · **Status:** Fachliche R-009-Entscheidung (Option C) präzisiert; technische Consent-API, Persistenz und Tests noch nicht abgenommen.  
> **Quellen:** [SDK-Sicherheit](05-security-architecture.md), [APP_API_ACCESS](../APP_API_ACCESS.md), [API-Verträge](../API_CONTRACTS.md), [Audit](../LOGGING_OBSERVABILITY.md).

## Beschlossene Bindung (Option C)

Ein ausdrücklicher Consent für freiwillige Datenverarbeitung wird an **Benutzer, aufrufende signierte App/Instanz, benannte Datenkategorie, Empfänger/Provider und versionierten Zweck** gebunden. Eine allgemeine Zustimmung „für die App“ ersetzt weder die Installationsgenehmigung, VFS-/Tenant-/Space-Rechte noch die Zustimmung zu einem neuen Verwendungszweck. Bei Änderung von Empfänger, Datenkategorie, Zweckversion oder Schutzbedarf ist der bestehende Consent neu zu bewerten; die fachlich zulässigen Erneuerungstrigger werden im konkreten Provider-/API-Vertrag definiert.

## Vertrauens- und Vollzugsgrenzen

| Bereich | Kontrollierbare Wirkung des Core | Nicht garantierbare Wirkung |
|---|---|---|
| Vor Datenübergabe | Berechtigung, Empfänger, aktiver Consent und App-Manifest prüfbar; fehlende Zustimmung verweigert | Reine Browser-UI darf diese Checks nicht ersetzen |
| Während Core-vermittelter Verarbeitung | Zweck-/Ressourcengrenzen, minimaler Datenumfang, Logging und Security Audit | Freier Drittanbieter-/Gerätecode wird nicht vom Core kontrolliert |
| Widerruf | Weitere Core-vermittelte Übertragungen/Subscriptions sperren, neue Tokens/Handles verweigern, betroffene Leases soweit möglich beenden | Bereits extern empfangene/gespeicherte Daten können nicht rückwirkend entfernt werden |
| Provider-Ereignisse | Verbindung/Subscription bei Neuverwendung erneut auf aktive Grants prüfen | Drittsystem-Logs, Provider-Backups oder selbstständig geführte externe Kopien sind nicht technisch durchsetzbar |
| Lokale Offline-/Browserkopien | Bei nächster kontrollierter Interaktion Cache-/Sessionzugang anhand Policy einschränken | Ein vollständig offline getrennter Client lässt sich nicht augenblicklich fernlöschen |
| Audit | Erteilung, Zweckänderung, Ablehnung, Entzug und Transferberechtigung revisionsgebunden protokollieren | Keine geheimen Nutzdaten oder OAuth-/Provider-Tokens im Audit |

## Datenfluss (fachlich, ohne erfundene Endpunkte)

1. App deklariert signiert zulässige Netzwerk-/API-/Datenoperationen und ihren Zweck.
2. Vertrauenswürdiger Host zeigt vor einer zweckgebundenen Freigabe eine vom Core kontrollierte Entscheidung inklusive Datenkategorie und Empfänger.
3. Permission/Consent Owner persistiert Ergebnis, Geltungsbereich, Zweckversion, Actor und Policy-Revision. Vor jeder neuen relevanten Verarbeitung prüft der serverseitige Gateway/Broker den aktuellen Zustand.
4. Fach-Owner minimiert die Datenmenge und führt lediglich genehmigte Operationen aus; ein Grant an einen Provider erweitert keine Dateisystem-/Systemrechte.
5. Widerruf erhöht die Autorisierungsrevision, beendet neu nutzbare Zugriffe/Subscriptions; bereits gesendete Daten erhalten keinen irreführenden „vollständig gelöscht“-Status.

## Prüfverträge

- CONS-01: Consent ohne konkrete App, Zweckversion, Datenklasse oder Empfänger reicht für eine zweckgebundene Übertragung nicht aus.
- CONS-02: Zweck-/Empfängerwechsel darf nicht unbemerkt auf alten Consent zurückfallen.
- CONS-03: Unterschriebene App-/Netzwerk-Freigabe alleine gewährt keine freiwillige Datenverarbeitung.
- CONS-04: Widerruf blockiert **weitere** Core-vermittelte Operationen, auch bei Replay, Cache oder laufender Subscription.
- CONS-05: UI verspricht **keine** Löschung bereits bei externem Empfänger befindlicher Daten.
- CONS-06: Security Audit bleibt ohne Nutzdaten/Secrets und unabhängig von konfigurierbarem Diagnose-Logging.

**Offen als technische Dokumentation:** Autoritatives Consent-Schema, finale Consent-Dialoge, Zeit-/Retention-Parameter, externe Löschanfrage/Bestätigungsbelege und Provider-Spezifika. Diese Punkte werden vor Implementierung als typisierte Contracts geprüft; sie sind nicht durch diese Grundsatzentscheidung automatisch festgelegt.
