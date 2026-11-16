# cloudOS – Vollständige Dokumentationsaktualisierung (16.11.2026)

Dieses Paket enthält **vollständige Markdown-Dokumente** für `Drezael/cloudOS` (Branch `main`), keine Diffs, Patches, Skripte oder Implementierungsdateien.

## Umfang

- **13 vollständige aktualisierte Bestandsdokumente** unter `docs/`.
- **2 vollständige neue Dokumente** unter `docs/`:
  - `ARCHITECTURE_DECISIONS_2026-11-16.md`: Nachweis und vollständiger Wortlaut der 13 ausdrücklich beschlossenen ADRs.
  - `SETTINGS_SYSTEM.md`: spezifizierter MVP-Vertrag für Settings und getrennte Verwaltungs-Sub-Apps.
- Nicht betroffene GitHub-Dokumente sind absichtlich nicht im Paket und bleiben unverändert.

## Berücksichtigte Entscheidungen

| ADR | Gewählte Option | Kernthema |
|---|---|---|
| ADR-002 | C | Vue 3/TypeScript, Vite, frameworkneutrales SDK |
| ADR-012 | B | Messbare Recovery- und Leistungsziele ohne erfundene Zahlen |
| ADR-017 | C | Rust/WASM, GPU- und Thread-Anforderungen |
| ADR-036 | C | SHA-512, RFC 8785/JCS, RSA-PSS/SHA-512, X.509 |
| ADR-059 | A | Alte Desktop-Verzeichnisse erst nach dem MVP migrieren |
| ADR-060 | B | Fähigkeitsbasierte Browser-/Accessibility-/Performance-Matrix |
| ADR-131 | C | Unicode NFC, case-sensitive VFS, Konfliktsicherheit |
| ADR-132 | B | Standardisierte Home-Struktur |
| ADR-136 | B | Koordinierte versionierte Backup-/Restore-Snapshots |
| ADR-138 | C | UID-/GID-Nummernkreise und permanente Nichtwiederverwendung |
| ADR-140 | C | Argon2id, Passwortpolitik und Recovery |
| ADR-142 | A | Schlanke Settings-Sub-App-Architektur |
| ADR-150 | B | Optionale Ablaufzeiten für interne Freigaben |

**ADR-094 bleibt `Proposed`** und wurde weder freigegeben noch neu zur Wahl gestellt. Ebenso bleiben alle weiteren nicht beschlossenen ADRs unverändert offen.

## GitHub-Basis und Prüfergebnis

Die Dokumentation wurde am **16.11.2026** gegen den erreichbaren aktuellen `main`-Dokumentationsstand geprüft. Die zehn Bestandsdokumente des verifizierten Volltextpakets vom 15.11.2026 stimmten vor den hier vorgenommenen Änderungen **bytegenau** mit den GitHub-Blobs überein; weitere Fachdateien wurden per SHA oder vollständigem Textinhalt mit dem aktuellen GitHub-Stand abgeglichen.

Das ADR-Register enthält jetzt **180** globale ADRs mit **165 `Accepted`** und **15 `Proposed`**. Index und vollständige ADR-Einträge wurden gegeneinander geprüft; sämtliche 13 neuen Beschlüsse sind in beiden Ansichten `Accepted`. Dateien enthalten vollständigen Markdown-Text; Code-Fences sind syntaktisch ausgeglichen. Die tatsächlichen Anwendungen und die Vertragstests wurden **nicht** implementiert.

## Übernahme ins Repository

1. Im eigenen lokalen Clone von `Drezael/cloudOS` den aktuellen Branch `main` prüfen und auf unerwartete parallele Änderungen achten.
2. Die **vollständigen** Markdown-Dateien aus diesem Paket unter den identischen `docs/`-Pfaden übernehmen. Die beiden neuen Dokumente als neue Dateien hinzufügen.
3. Die Änderungen fachlich prüfen und mit den eigenen Git-Berechtigungen committen/pushen. **Kein Skript aus diesem Paket ist auszuführen.**

**Wichtig:** Die verbundene GitHub-Integration in dieser Sitzung besitzt Lese-, aber **keine Schreibberechtigung** (`push: false`). Deshalb ist auf GitHub noch **nichts aktualisiert**. Ein späterer konkurrierender Commit auf `main` erfordert vor Übernahme einen erneuten Abgleich.
