# cloudOS – ADR-Entscheidungsabschluss (16.11.2026)

Dieses Archiv enthält **vollständige Markdown-Dateien**, keine Diffs, Patchdateien oder Implementierungsskripte.

**Geprüfter Ausgangspunkt:** GitHub `Drezael/cloudOS`, `main`, `docs/ADR_REGISTER.md`, Blob `0482ba2798146b7be89609bffb30f3be0353c58c`.

## Enthaltene vollständige Zieldateien

1. `docs/ADR_REGISTER.md` – vollständiges zentrales Register, mit 14 fachlich abgeschlossenen ADRs, Zahlenstand **179 Accepted / 1 Proposed**, aktualisiertem Backlog und Quellverweisen.
2. `docs/ARCHITECTURE_DECISIONS_OPEN_ADRS_2026-11-16.md` – vollständige Begründungen, Grenzen und Test-/Freigabe-Gates für die 14 Entscheidungen.
3. `docs/app-framework/20-technical-decisions-E03-E04-2026-11-16.md` – vollständige Zusammenfassung der technischen Detailentscheidungen 1–17 (je Option B).

## Grenzen dieser Lieferung

- **Kein direkter GitHub-Push/Commit** und keine Implementierung. Der Originalstand `main` bleibt unverändert.
- ADR-094 bleibt weiterhin `Proposed`, wie ausdrücklich gewünscht.
- Die fachlichen Architekturentscheidungen sind abgeschlossen; reale technische Prüfungen/Source-Evidence und maschinenlesbare DTO-/SQL-/SDK-Verträge sind dadurch nicht automatisch erstellt oder freigegeben.
- **Vollständige repo-weite Harmonisierung aller historischen Fachkonzepte steht weiterhin aus.** Besonders `IDENTITY_INTEGRATION.md` enthält ältere Texte, die externe UID-/GID-Übernahme unter Ausnahmebedingungen zulassen; nach ADR-156 darf dies nicht mehr autoritativ geschehen. Ebenfalls zu harmonisieren sind die alten ADR-Tabellen und Verweise in Push, Browser, WASI, ACL sowie externe Auth-/Connector-Docs.
- Eine spätere Anwendung auf GitHub muss zunächst prüfen, ob sich die oben genannte Ausgangsrevision seit diesem Paket verändert hat. Bei Abweichung den aktuellen `main`-Stand erneut einlesen und **vollständige Dateien** neu generieren statt ältere Fassungen blind zu überschreiben.

## Import

Die vollständigen Dateien sind unter den angegebenen relativen `docs/`-Pfaden zur späteren kontrollierten GitHub-Dokumentationsaktualisierung vorgesehen.
