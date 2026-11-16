# cloudOS - Vollstaendige Markdown-Dokumente (15.11.2026)

Diese Lieferung enthaelt **vollstaendige, direkt ersetzbare Markdown-Dateien** im Ordner `docs/`, keine Python-Skripte, keine Patches und keine Diffs.

Die neun ueberarbeiteten Bestandsdokumente beruhen auf den in GitHub `main` geprueften Konzepten. Acht Basisdateien stimmen bytegenau mit den geprueften Git-Blob-SHAs ueberein. Bei `AUTHENTICATION_AUTHORIZATION.md` lag in der Bibliothek eine aeltere Fassung; deren Inhalt wurde unter Beruecksichtigung der auf GitHub eingesehenen neueren Abschnitte 27-29 und der Credential-Assurance-Ergaenzung vervollstaendigt. Diese Datei ist deshalb kein bytegenaues Abbild des GitHub-Originals, sondern eine vollstaendig bereitgestellte konsolidierte Zielfassung.

Die 12 beschlossenen Infrastrukturentscheidungen, ADR-Statuskorrekturen, Upgrade-/Wartungspolitik, NATS-/Outbox-/Retention-Regeln, Offline-Storage-Migration sowie Secrets und Edge wurden in die zustaendigen Dokumente aufgenommen. Das neue `INFRASTRUCTURE_DECISIONS_2026-11-15.md` bildet die gemeinsame verbindliche Referenz.

**Uebernahme:** Die fertigen Dateien aus `docs/` einzeln im Repository unter denselben Pfaden ersetzen; anschliessend alle Aenderungen fachlich pruefen und ueber den eigenen Git-Workflow committen. Es sind keine Skripte auszufuehren.

**GitHub:** Die verbundene App hat `push=false`; daher konnte kein Commit auf `main` erstellt werden. **Kein cloudOS-Implementierungscode** wurde erzeugt oder geaendert.
