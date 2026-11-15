# Abgleichsmatrix zu führenden cloudOS-Bestandsverträgen

> **Stand:** 15.11.2026 · **Status:** Quellenabgleich für Phase 1; **keine Bearbeitung der Bestandsdateien in diesem Paket**.  
> Die elf SDK-Dateien wurden gegen die von GitHub `main` gelesenen Blob-SHAs verifiziert; die nachstehend genannten Bestandsdokumente wurden über den GitHub-Connector hinsichtlich ihrer relevanten Fachregeln geprüft. Eine vollständige zeichengetreue Übernahme der Bestandsdateien in dieses ZIP wäre ohne lokalen Originaldateibestand nicht sicher. Deshalb bleiben sie absichtlich unangetastet und sind Gegenstand der nächsten Volltext-Konsolidierung.

## Ziel und Änderungsregeln

Bei Integrationen dürfen vorhandene Normen nicht verdoppelt oder ohne explizite Entscheidung abgeschwächt werden. Jede später tatsächlich geänderte Bestandsdatei muss aus ihrem dann aktuellen vollständigen Inhalt erstellt werden; GitHub-Blob-SHA und Umfang des Eingriffs sind zu dokumentieren. Keine losen Patches als angeblich vollständige Dateien deklarieren.

| Bestandsdokument | K-Beschlüsse | Bereits nachgewiesene Regeln | Notwendige Prüfung / Ergänzung bei Folgerevision |
|---|---|---|---|
| `docs/APP_SIGNING.md` | K1, K9 | X.509, Self-signed nur Digest-/Admin-gebunden, Integrität hart, Rollback nur auf gültige Digests | Conformance-Gate ohne Sicherheits-Bypass als zusätzliche Voraussetzung; Kill-Switch-Sperrklassen, Policy-Epoch und Quarantäne |
| `docs/APP_API_ACCESS.md` | K1, K3, K5, K9 | Signierte Endpoint-Allowlist, verpflichtender Admin-Review, App-Bridge/Origins, autorisierte SSE-Subscription | Conformance-Status im Installationspfad; SW-/Origin-Topologie, Release-Revoke vor Operation und Network-Broker-Verbindungsidentität |
| `docs/APPLICATION_SYSTEM.md` | K1, K2, K3, K4, K9 | Browser/Web-WASM, optionale WASI/Native, signierte Baseline, mehrdimensionale App-Kompatibilität | Runtime-Freigabematrix präzisieren, pro App eigene Origin, Kill-Switch und ältere aktive Instanzen |
| `docs/SYSTEM_ARCHITECTURE.md` | K2, K3 | Go-Core, Browser-WASM, WASI opt-in, Native nur isoliert, separate Vertrauenszonen | Browser-/Server-Runtime-Topologie und einheitliche Owner-Funktionszuordnung bestätigen |
| `docs/API_CONTRACTS.md` | K4, K5, K6, K7, K8, K9 | Nur eine API-Major, Core-SSE, Saga/Delegation, serverseitige Verschlüsselung, Replay-Autorisierung | K9-Revoke-Mechanismus, Recovery-Delegation, E2EE-Featuregrenzen und Retention-Datenklassen explizit verlinken |
| `docs/PUSH_REALTIME.md` | K5, K7 | SSE als Core-Standard, transaktionale Outbox, `Last-Event-ID`, Policy-Replay, optional Web Push | Externe SSE/WS als andere Infrastruktur abgrenzen; Purge/Replay/Retention je Klasse fixieren |
| `docs/VIRTUAL_FILESYSTEM.md` | K6, K7 | Berechtigte Vorschauen, Indexe und Ressourcen-Scopes; VFS-Ownership | E2EE-Ressourcenprofile, clientseitige Vorschau und Such-Limits, Datenlösch-/Cachefolgen |
| `docs/CORE_DEPLOYMENT_SCALING.md` | K2, K3, K5 | Combined-Core-Standard und logisch entkoppelte Services | Network-/Browser-Origin-/Runner-Grenzen in Split/Kubernetes aufrechterhalten |
| `docs/APP_MANAGEMENT_STORE.md` | K1, K9 | App-Release- und Store-Freigabekontext | Installations- und Sicherheitsquarantäne-UX gegen Bestand prüfen |

## Keine stillschweigenden Festlegungen

- **Nicht** beschlossen: ein konkretes DNS-/Origin-Schema, neue REST-Routen, numerische Retention-Werte, spezifische KMS/E2EE-Algorithmen, Container-Runtime für Native oder endgültige Zustands-Enum-Namen.
- **Nicht** zulässig: zweite parallele Core-API-Major, zusätzlicher regulärer Core-WebSocket-Push, Self-signed-Integritätsausnahme, Direktstart unsignierter App-Payloads, beliebige App-Service-Worker.
- **Nicht** nachgewiesen: vollständige Rekonstruktion der Q001–Q033; Repository-weite Owner-/Datenmodell-/Security-Validierung; vollständige Link-Verifikation aller externen Bestandsdokumente.

## Nächste Abnahme

1. Vollständigen Bestand aller oben aufgeführten Dateien von `main` in ein lokales Arbeitsverzeichnis holen.
2. Jedes Bestandsdokument **vollständig** unter Beibehaltung der bestehenden Regeln aktualisieren.
3. Diff, Terminologie, Referenzen, Tabellen und Mermaid-Diagramme gegen bestehende Verträge prüfen.
4. Risk Register und Readiness-Bericht erst nach vollendeter Quellen-/Technikprüfung auf `VERIFIZIERT` beziehungsweise `READY` setzen.
