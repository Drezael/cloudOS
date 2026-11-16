# Abgleichsmatrix zu führenden cloudOS-Bestandsverträgen

> **Stand:** 16.11.2026 · **Status:** Volltext-Konsolidierung der bekannten neun führenden Bestandsverträge erstellt; **technische Verifikation bleibt offen**.  
> Die SDK-Dokumente und die neun Bestandsdateien wurden auf vollständige, per Blob-SHA geprüfte GitHub-Fassungen zurückgeführt. Die folgenden Zeilen enthalten die ursprüngliche Prüfliste; der dokumentarische Übernahmestand vom 16.11.2026 folgt in Abschnitt 5. Dies ist weder ein technischer Konformitätsnachweis noch ein Push-Nachweis der in diesem Paket neuen Änderungen.

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
| `docs/APP_MANAGEMENT_STORE.md` | K1, K4, K9 | App-Release- und Store-Freigabekontext | Installations- und Sicherheitsquarantäne-UX gegen Bestand prüfen |

## Keine stillschweigenden Festlegungen

- **Nicht** beschlossen: ein konkretes DNS-/Origin-Schema, neue REST-Routen, numerische Retention-Werte, spezifische KMS/E2EE-Algorithmen, Container-Runtime für Native oder endgültige Zustands-Enum-Namen.
- **Nicht** zulässig: zweite parallele Core-API-Major, zusätzlicher regulärer Core-WebSocket-Push, Self-signed-Integritätsausnahme, Direktstart unsignierter App-Payloads, beliebige App-Service-Worker.
- **Quellenkritisch:** Q001–Q033 sind fachlich als Kurzbeschlüsse rekonstruierbar; die vollständigen Originalwortlaute fehlen teils weiter. Eine repository-weite Owner-/Datenmodell-/Security-Validierung und vollständige Link-Verifikation **aller** Repository-Dateien werden nicht behauptet.

## Nächste Abnahme

1. Die vollständigen Bestandsfassungen wurden auf ihre GitHub-Blob-SHAs geprüft; in diesem Dokumentationspaket werden K1–K9 in den neun Fachverträgen integriert.
2. Nach Änderung vollständige Markdown-, Querverweis- und Widerspruchs-QA durchführen, Ergebnis und Grenzen dokumentieren.
3. **Keine** Anhebung auf `TECHNISCH_VERIFIZIERT`, `CLOSED` oder `READY` ohne echte Testresultate, historische Originalquellen und formelle Freigabe.
4. Repo-weite Prüfung und Implementierungsdetails nach [Gap-Register](19-gap-and-evidence-register.md) nachführen.

## 5. Quellenkritischer Stand 16.11.2026

Die folgenden **vollständigen** Bestandsdokumente enthalten jetzt die zuvor nur im SDK-Beschlussprotokoll niedergelegten relevanten K1–K9-Regeln: `APP_SIGNING.md`, `APP_API_ACCESS.md`, `APPLICATION_SYSTEM.md`, `SYSTEM_ARCHITECTURE.md`, `API_CONTRACTS.md`, `PUSH_REALTIME.md`, `VIRTUAL_FILESYSTEM.md`, `CORE_DEPLOYMENT_SCALING.md` und `APP_MANAGEMENT_STORE.md`. Die alte unverbindliche Formulierung zur App-Origin in `APP_API_ACCESS.md` sowie die scheinbar widersprüchliche Versionsregel in `APP_MANAGEMENT_STORE.md` wurden in diesem Paket konkret ersetzt. Keine technische K1–K9-Prüfung wurde ausgeführt.

**Offene Folgebelege:** exakte Browser-Origin-/Service-Worker-Topologie, reale Conformance, Runtime-Sandbox, E2EE-/Recovery-Test, Retention-Grenzwerte und Quellprüfung des technischen Dienstkatalogs. Siehe [Readiness](10-implementation-readiness.md) und [Testmatrix](17-negative-contract-tests.md).
