# R-015 – Konformitätsprüfung auf einer echten cloudOS-Testinstanz

> **Stand:** 16.11.2026 · **Status:** Fachliche Option A dokumentiert, kein produktives Deployment und keine Testläufe.  
> **Grundsatz:** Eine **gemeinsame echte cloudOS-Testinstanz** mit synthetischen Benutzer-/Ressourcendaten; **kein SDK-Simulator** als gleichwertiger Konformitätsnachweis.  
> **Quellen:** [Q105/Q154](02-decision-register.md), [Negative Tests](17-negative-contract-tests.md), [Go-Core-Betrieb](../CORE_DEPLOYMENT_SCALING.md).

## 1. Isolations- und Freigabemodell

- Testinstanz darf weder dieselbe PostgreSQL-/NATS-/Blob-Ablage noch Secrets, Signaturprivatekeys, Administrator-Sessions oder Auth-Domains wie Produktion teilen.
- Synthetische Benutzer/IDs, Test-Shares, Test-Spaces, Test-Apps und Test-Credentials werden getrennt provisioniert; keine echten personenbezogenen Dateien/Providerzugänge.
- Administrativ gefährliche Conformance-Funktionen laufen unter gesondertem, zeitlich/ressourcenbegrenzt berechtigtem Testprincipal und genehmigtem App-/Operationenscope; keine pauschale `Admin`-Session im App-Code.
- Der Test muss gegen die tatsächlich ausgeführten Go-Core-Owner und Browser-Sandbox laufen, nicht gegen nur aus Swagger/Mocks generierte Antworten.
- Eine auf dem Testsystem freigegebene App ist dadurch **nicht** für Produktion freigegeben; Release-/Signatur-/Quarantänepolicy bleiben separater Gate.

## 2. Testlebenszyklus (fachliche Reihenfolge)

1. Referenzbasis aus freigegebenem Testrelease, bereinigtem synthetischem Datenstand und nachgewiesenem Sicherheitsprofil herstellen.
2. Umgebung/Versions-Digests und Vertragsrevisionen protokollieren; Testprincipal mit minimalen Capabilities ausstatten.
3. Positive Kontrollprobe und Negativtests einschließlich Revocation, App-Origin, Release-Kill-Switch, API-Kompatibilität und Datenisolierung ausführen.
4. Bei mutierenden Tests ausschließlich Testressourcen innerhalb explizit zugewiesener Quotas verwenden; riskante Deletes/Policy-Änderungen nur mit separater Freigabe.
5. Testjournal mit Trace-/Audit-Belegen erstellen, bekannte Einschränkungen verzeichnen, Ergebnis `PASS`/`FAIL`/`SKIPPED`/`UNAVAILABLE` begründen.
6. Nach jedem Lauf Testzustand kontrolliert bereinigen oder aus einem **vorherigen verifizierten** Baseline-Restore wiederherstellen; wichtige Logs/Audits vor Bereinigung gemäß Test-Retention sichern.
7. Residualdaten/Grants/Leases prüfen; bei unsicherer Bereinigung Umgebung außer Betrieb nehmen statt Tests zu „überspringen und bestehen“.

## 3. Sicherheits- und Fehlerfälle

| ID | Szenario | Verpflichtender Nachweis |
|---|---|---|
| TEST-ENV-01 | Test-App versucht Produktions-/fremden Test-Tenant-Zugriff | Serverseitige Sperre, kein Datenabfluss |
| TEST-ENV-02 | Testjob verändert eine eigene Datei/Quota | Revisions- und Owner-Deltas bleiben im Testscope |
| TEST-ENV-03 | Testlauf bricht während mutierender Operation ab | Reproduzierbarer Rollback/Restore oder Quarantäne |
| TEST-ENV-04 | Test-Grants werden widerrufen | Unmittelbare Core-Sperre unabhängig von SSE-Nachricht |
| TEST-ENV-05 | Browser-Sandbox ist unvollständig | Conformance nicht erfolgreich; klare Unsupported-/Fehlerdiagnose |
| TEST-ENV-06 | App nutzt WebGPU/WASM-Threads auf nicht kompatiblem Browser | Rust-Profilstart gesperrt, kein stiller Ersatz |
| TEST-ENV-07 | Test verursacht unzulässige Secret-/Admin-Eskalation | Abbruch/Alarmierung, keine Prod-Secrets betroffen |
| TEST-ENV-08 | Testmeldung behauptet erfolgreiche Bereinigung ohne Nachweis | Nicht als freigegebener Testlauf werten |

## 4. Noch ausstehende technische Dokumentation

Versionierte Instanzdefinition, synthetischer Seed, genaue Principal-Scopes, Test-Provider, Retention-Dauer, positive/negative Testvektoren, Browsermatrix, CI-Integration, Reset-Mechanismus und Freigabezeichnung sind **vor der jeweiligen Implementierung** zu spezifizieren. Es werden **keine** ungemessenen SLA- oder Prozesswerte behauptet.

**Readiness bleibt offen:** Ein geplanter Test ist kein ausgeführter, bestandener Test. Der Readiness-Bericht muss dies getrennt ausweisen.
