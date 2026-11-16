# R-013 – Negative Contract Tests und unverhandelbare Verbote

> **Stand:** 16.11.2026 · **Status:** Verbindliche fachliche Testfälle spezifiziert; **nicht ausgeführt**, keine fiktiven PASS-Ergebnisse.  
> **Quellen:** [SDK Decision Register](02-decision-register.md), [Sicherheitsvertrag](05-security-architecture.md), [K1–K9](11-architecture-decisions-K1-K9.md), [API-Verträge](../API_CONTRACTS.md).

## Negative SDK-Entscheidungen

| ID | Entscheidung | Negatives Szenario | Akzeptanzbedingung |
|---|---|---|---|
| NEG-036 | Q036: keine eigenen globalen App-Suchanbieter | Dritt-App versucht SDK-Registrierung eines Suchproviders oder unautorisierten Index-Hooks | Keine ausführbare Registrierung; keine Erweiterung des Core-Suchindexes oder Privilegien |
| NEG-052 | Q052: keine App-zu-App-Plugins / Extension Points | App lädt als privilegiertes Plugin eine andere App oder injiziert deren Code | Verweigert; zulässige Core-IPC-Aktionen bleiben getrennt und autorisiert |
| NEG-075 | Q075: keine externen Gastlinks im MVP | App versucht anonymen/Public-Link-/Gastfreigabe-Grant zu erzeugen | Keine API-Fähigkeit, keine Grant-Erteilung; interne Shares/Spaces unverändert |
| NEG-090 | Q037/Q090: Browser-Clipboard ohne History/Synchronisation | SDK-App fordert globale Clipboard-History, Keylogging oder geräteübergreifende Ablage an | Kein entsprechender SDK-Broker oder persistenter History-Service; native Browserberechtigung bleibt erforderlich |
| NEG-105 | Q105: kein vollständiger SDK-Simulator als Releasebeleg | Paket liefert nur Mock-/Simulator-Testbericht statt echter cloudOS-Instanztests | Conformance-/Release-Gate bleibt unbestanden; reine Contract-Mocks sind nur Unit-Testhilfen |
| NEG-130 | Q130: keine geografische SDK-Datenresidenz-Policy | App fordert vermeintliche SDK-Region-/Residency-Garantie und interpretiert sie als Autorisierungsgrant | Keine derartige SDK-Zusage; ggf. gesonderte Betreiber-Policy unabhängig davon |

## Ergänzende Sicherheits-Negativtests K1–K9

| ID | Angriffs-/Fehlerfall | Erwartetes Ergebnis |
|---|---|---|
| NEG-K1-01 | Selbstsignierter Publisher bestätigt, Paketindex/Conformance fehlerhaft | Installation und Launch blockiert; Herkunftsfreigabe ist kein Bypass |
| NEG-K1-02 | App-Update erweitert signierte API-/Device-Capabilities ohne neue Genehmigung | Nicht aktivierbar; neue Risikobestätigung und korrekter Paketdigest erforderlich |
| NEG-K2-01 | Server-WASI/Native direkt im privilegierten Core-Prozess beantragt | Verweigert; kein zusätzliches Laufzeitprofil im MVP |
| NEG-K3-01 | App-iframe versucht BFF-Session-Cookie für unautorisierte Core-API | Serverseitig verweigert trotz gleicher Nutzeranmeldung |
| NEG-K3-02 | App kontrolliert Service-Worker-Scope einer fremden App oder des Core | Origin-/Scope-Isolation unterbindet Interception und fremde Speicherzugriffe |
| NEG-K4-01 | Alte App-Instanz verwendet nicht mehr aktive Core-API-Major | Abgewiesen; einzig aktive Major bleibt maßgeblich |
| NEG-K4-02 | Rollout will zwei Default-Releases pro App-ID gleichzeitig aktivieren | Abgewiesen; nur ein Standardrelease für neue Starts |
| NEG-K5-01 | Dritt-App eröffnet direkten allgemeinen Core-WebSocket als SSE-/Broker-Bypass | Verweigert; kein alternativer regulärer Core-Pushkanal |
| NEG-K5-02 | Provider-WebSocket ohne signierte Ziel-/Scope-Genehmigung | Network Broker verweigert Verbindung |
| NEG-K6-01 | Serverprozess versucht E2EE-Klartextvorschau/Index | Unverfügbar; klar als Capability-Limit gekennzeichnet |
| NEG-K7-01 | Replay-Cursor nach Widerruf zeigt alte private Payload | Keine geschützten Inhalte; erneute aktuelle Autorisierung |
| NEG-K7-02 | Dedupe-/Event-Purge erfolgt ohne Blick auf Owner-Retention und Audit | Kein unautorisierter Purge; Audit-Aufbewahrung bleibt getrennt |
| NEG-K8-01 | Saga setzt nach Widerruf reguläre alte User-Capability fort | Gestoppt; keine versteckte Wiederaufnahme |
| NEG-K8-02 | Kompensation nutzt alte, widerrufene User-Rechte | Verweigert, bis minimale neue Recovery-Autorisierung vorliegt |
| NEG-K9-01 | Quarantänierter Digest startet über Rollback oder bestehenden App-Cache neu | Start/Operationen blockiert; keine umgehende Altversion |
| NEG-R008 | App-Data-Service wird als zweite freie PostgreSQL-Anbindung einer `.cloudApp` implementiert | Verweigert; Core-Owner-/App-Scope-/Tenant-Checks erforderlich |
| NEG-R009 | Consent widerrufen, Providerverbindung sendet dennoch neue Daten | Weitere Core-vermittelte Verarbeitung verweigert; keine behauptete Löschung externer Kopien |
| NEG-R015 | Conformance läuft mit realen Benutzerdaten/Produktions-Admin-Token | Nicht zulässig; Testfreigabe verweigert |

## Ausführungsvoraussetzungen und Evidenz

Für jeden Test sind später mindestens zu dokumentieren: Test-ID, verifizierter Anforderungssatz/Release-Digest, spezifizierte Testumgebung, synthetische Daten, positive Kontrollprobe, negativer Aufruf, erwarteter HTTP-/SDK-Fehler **ohne erfundene Fehlercodes**, relevante Audit-/Policy-Revisionen, reproduzierbarer Log-/Trace-Nachweis und Ergebnis mit Zeit/Reviewer.

**Wichtig:** Diese Tabelle definiert fachliche Akzeptanz und keine bereits existierenden API-Routen, technischen Testskripte oder PASS-Ergebnisse. Technische Implementierung und Laufzeitnachweise folgen erst nach gesonderter Implementierungsfreigabe.
