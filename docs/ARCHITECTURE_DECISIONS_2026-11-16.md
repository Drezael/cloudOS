# cloudOS – Beschlossene Architekturentscheidungen vom 16.11.2026

> **Status:** Fachlich ausdrücklich vom Projektverantwortlichen beschlossen; keine Implementierungs- oder Testerfolgsaussage.  
> **Repository-Basis:** `Drezael/cloudOS`, Branch `main`, geprüft anhand des zentralen Registers und maßgeblicher Fachkonzepte am 16.11.2026.  
> **Normative Integration:** [ADR_REGISTER.md](ADR_REGISTER.md); die nachfolgenden Fachdateien bilden die systembezogene Konkretisierung.  
> **Abgrenzung:** ADR-094 bleibt `Proposed` und wird in diesem Nachtrag weder entschieden noch verändert.  

## 1. Freigabe- und Konsistenzregeln

Die unten dokumentierten Optionen wurden in der fortlaufenden Architekturabstimmung bestätigt. In älteren Fachkonzepten ausdrücklich abweichende vorläufige Vorschläge werden durch die neueren Beschlüsse ersetzt. Andere `Accepted`-ADRs, die acht Infrastrukturentscheidungen vom 15.11.2026 und die bereits abgeschlossenen SDK-Verträge bleiben unberührt. Eine neue technische Verpflichtung, ein Messwert oder eine Abnahmeschwelle ohne gesonderte Freigabe wird hieraus nicht abgeleitet.

| Fachlicher Zustand | Bedeutung |
|---|---|
| Accepted | Vom Projektverantwortlichen fachlich beschlossen |
| Implementierung | Weiterhin gesondert zu planen, zu implementieren und zu testen |
| Release-Gate | Verbindlich, sobald die jeweiligen Zielwerte und Testprofile nachvollziehbar bestimmt sind |

## 2. Entscheidungen

### ADR-002 – Vue 3/Vite und Desktop-Host-Rendering

**Status:** Accepted · **Gewählte Option:** C · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Vue 3 mit TypeScript ist für Desktop, Window Host, App Host und cloudOS-eigene Oberflächen verbindlich; Vite ist das bevorzugte Build-Werkzeug. Das öffentliche App-SDK und der signierte App-Vertrag bleiben frameworkneutral. Drittanbieter können etwa React, Svelte oder Rust/WASM verwenden.

**Begründung und Folgen:** Die Frameworkwahl betrifft nur eigene Frontend-Implementierungen und darf Drittanbieter nicht an Vue binden. Browser- und Window-Host-Sicherheitsgrenzen bleiben unabhängig davon verpflichtend.

### ADR-012 – Backup-RPO/RTO und Browser-/Leistungsbudgets

**Status:** Accepted · **Gewählte Option:** B · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Für den MVP werden messbare RPO-/RTO-, Browser- und Leistungsziele sowie eine reproduzierbare Referenzumgebung verbindlich definiert, gemessen, dokumentiert und regelmäßig geprüft. Konkrete Grenzwerte werden erst nach Messung und ausdrücklicher Freigabe als zugesagt behandelt. Eine Hochverfügbarkeitsgarantie ist nicht Bestandteil des MVP.

**Begründung und Folgen:** Recovery- und Performancebudgets sind Release-Abnahmekriterien, aber es werden keine ungemessenen SLA-Zahlen oder eine Pflicht zu Cluster-/HA-Betrieb vorweggenommen. Browserprofile werden mit ADR-060 abgestimmt.

### ADR-017 – Rust-Web-Toolchain und GPU-Kompatibilität

**Status:** Accepted · **Gewählte Option:** C · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Browserfähige Rust-Apps liefern ein signiertes `web-wasm`-Paket mit `wasm32-unknown-unknown`-Artefakt, JS-Glue und Assets. Rust Stable/Cargo/wasm-bindgen/Vite ist die Referenztoolchain; weitere nachweislich konforme Toolchains bleiben zulässig. Für das Rust/WASM-MVP-Profil sind WebGPU und WebAssembly-Multithreading Pflicht, ohne automatischen WebGL-/Single-Thread-Fallback. Strikte Cross-Origin-Isolation mit COEP `require-corp`, abgestimmten COOP/CORP/CORS/CSP/Permissions-Policies und isolierter App-Origin wird vorausgesetzt.

**Begründung und Folgen:** Fehlende WebGPU-/Thread-Fähigkeiten oder unvollständige Isolation sperren den Start dieses Runtime-Profils mit klarer Inkompatibilitätsmeldung. Reproduzierbare Builds und Browser-Integrationstests sind Teil der Freigabe; die öffentliche API bleibt toolchainneutral.

### ADR-036 – Signaturalgorithmen und kanonische Serialisierung

**Status:** Accepted · **Gewählte Option:** C · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Das Signaturprofil v1 verwendet SHA-512 für die vollständige Dateiliste, RFC 8785/JCS für kanonisches JSON, RSA-PSS mit SHA-512 und MGF1, einer Salt-Länge von 64 Bytes und RSA-Schlüsseln mit mindestens 3072 Bit sowie X.509 mit überprüfter App-ID-/Publisher-Bindung. Signierte Berechtigungen und Paketdateien werden vollständig und strikt geprüft; mehrdeutige Serialisierung, zusätzliche/unbekannte unsignierte Bytes, ungültige oder widerrufene Zertifikate sind abzulehnen.

**Begründung und Folgen:** Die Prüfung erfolgt gegen exakt definierte Bytes und negative Interoperabilitätstests. Neue Algorithmen/Signaturprofile dürfen nur ausdrücklich freigegeben und versioniert eingeführt werden; v1 bleibt bis dahin verbindlich.

### ADR-059 – Migration alter Desktop-Home-Verzeichnisse

**Status:** Accepted · **Gewählte Option:** A · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Im MVP existiert ausschließlich `/.desktop/` als kanonische Desktop-Ablage. Neue Homes folgen ADR-132. Es erfolgt keine automatische Migration alter `/Desktop/`-Strukturen. Bereits vorhandene Altdaten bleiben unverändert und werden weder automatisch gelöscht noch überschrieben; ein kontrollierter Migrationsworkflow ist nach dem MVP gesondert zu spezifizieren.

**Begründung und Folgen:** Die Migrationsoptionen in älteren Konzepten sind Zukunftsentwürfe und keine MVP-Funktion. Veraltete Beispiele dürfen keinen zweiten Desktop-Root etablieren.

### ADR-060 – Desktop-Performance, Browser und Barrierefreiheit

**Status:** Accepted · **Gewählte Option:** B · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Für Chrome, Edge, Firefox und Safari gilt eine verbindliche, fähigkeitsbasierte Browser-, Accessibility- und Performance-Testmatrix. Desktop-Kernfunktionen und das anspruchsvolle Rust/WebGPU/WASM-Profil werden getrennt bewertet. Fehlende Browser-Features werden sichtbar als nicht unterstützt gemeldet; die WebGPU-/WASM-Thread-Anforderungen aus ADR-017 werden nicht stillschweigend abgeschwächt.

**Begründung und Folgen:** Barrierefreiheit und reproduzierbare Performanceprüfungen sind Release-Gates. Konkrete Browserversionen, Hardwareprofile und Grenzwerte werden nach Prüfung und Messung freigegeben, statt pauschale Parität zu versprechen.

### ADR-131 – VFS-Namen, Unicode-Normalisierung und Case-Policy

**Status:** Accepted · **Gewählte Option:** C · **Beschlussdatum:** 16.11.2026

**Entscheidung:** VFS-Namen werden in Unicode NFC kanonisiert und innerhalb eines Verzeichnisses anhand des normalisierten Namens eindeutig geprüft. Vergleiche bleiben case-sensitive (`Bericht.txt` und `bericht.txt` sind verschieden). Server-Validatoren verhindern reservierte Namen, normalisierungsbedingte Kollisionen und unerlaubte Pfadtricks.

**Begründung und Folgen:** Import, Export und Synchronisation auf abweichend normalisierenden oder case-insensitiven Plattformen müssen Konflikte erkennbar und ohne stillschweigendes Überschreiben behandeln.

### ADR-132 – Standardstruktur neu angelegter Home-Verzeichnisse

**Status:** Accepted · **Gewählte Option:** B · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Ein neues Home enthält die reservierten Desktop-Ressourcen `/.desktop/`, `/.taskbar.json`, `/.desktop-layout.json` und die Standardordner `Documents/`, `Downloads/`, `Pictures/`, `Music/`, `Videos/`, `.config/`, `.local/`. Freigaben und Spaces werden nur als autorisierte virtuelle Einbindungen bereitgestellt.

**Begründung und Folgen:** Provisionierung ist über stabile Benutzer-IDs idempotent. Vom Benutzer später entfernte reguläre Standardordner werden beim Login nicht ungefragt neu erzeugt; geschützte Desktop-Ressourcen besitzen separate Reparatur- und Recovery-Regeln.

### ADR-136 – VFS-Backup-Konsistenz und selektives Restore

**Status:** Accepted · **Gewählte Option:** B · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Koordinierte versionierte Snapshots verknüpfen PostgreSQL-Metadaten, File-Nodes, ACLs, Space-Referenzen, referenzierte unveränderliche Blobs sowie zugehörige Schlüsselversionen zu einem konsistenten Sicherungsstand. Vollständige Instanz-, Home- und Einzeldatei-Restores sind vorgesehen; Wiederherstellung und Integritätsprüfung erfolgen zunächst isoliert vor kontrollierter Freigabe.

**Begründung und Folgen:** Backups und Restores dürfen keine aktuellen fremden Daten oder Systemrichtlinien ungewollt überschreiben und sollen den laufenden Betrieb möglichst wenig beeinträchtigen. Konkrete RPO-/RTO-Werte unterliegen ADR-012; kontinuierliche Point-in-Time-Recovery wird nicht allein durch diese ADR zugesagt.

### ADR-138 – UID-/GID-Nummernkreise und Wiederverwendung

**Status:** Accepted · **Gewählte Option:** C · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Virtuelle POSIX-UID und -GID werden zentral, eindeutig und transaktional vergeben, unabhängig voneinander in getrennten Nummernräumen. `0` ist reserviert; `1–9999` sind für interne Systemidentitäten vorgesehen; reguläre Benutzer und Gruppen erhalten Werte von `10000` bis `2147483647`. Stabile UUIDs bleiben autoritativ. Einmal vergebene UID-/GID-Werte werden niemals erneut vergeben, auch nicht nach Löschung. Private Primärgruppen werden unterstützt.

**Begründung und Folgen:** Provisionierung, Parallelität, Backup und Restore müssen Zuordnungen und Tombstones erhalten sowie Nummernkonflikte sicher ablehnen. Externe UID/GID-Importe werden durch ADR-156 separat entschieden und dürfen die lokale Eindeutigkeit nicht aufweichen.

### ADR-140 – Passwort-Hashing, Passwortpolitik und Recovery

**Status:** Accepted · **Gewählte Option:** C · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Passwörter werden mit Argon2id gehasht. Einstiegsparameter sind 19 MiB Speicher, zwei Durchläufe und Parallelität 1, mit messungs-/risikobasierter Nachkalibrierung. Mindestlänge 15 Zeichen, kein erzwungener regelmäßiger Passwortwechsel, Prüfung auf kompromittierte Passwörter und risikobasierte Rate-Limits. Self-Service- und administrative Wiederherstellung sind kontrolliert, auditierbar und dürfen kein universelles Masterpasswort benutzen.

**Begründung und Folgen:** Schlüssel-/Hashmaterial wird nie als Klartext preisgegeben; Recovery darf keinen Authentifizierungs- oder Rechte-Bypass erzeugen. Recovery-Token-Laufzeiten, Bindungen und genaue Re-Hash-Gates werden im Sicherheitsvertrag mit Testnachweisen präzisiert, nicht erfunden.

### ADR-142 – Settings-Sub-Apps und eigener Berechtigungskontext

**Status:** Accepted · **Gewählte Option:** A · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Für den MVP dient `Settings.cloudApp` als zentrale Navigation. `UserManagement.cloudApp` und `GroupManagement.cloudApp` werden als getrennte signierte Pflicht-Core-Apps ausgeliefert. `PermissionManagement.cloudApp` wird auf später verschoben; zwingende administrative Berechtigungsoperationen bleiben über gesondert geschützte Core-Verträge kontrollierbar. Deep Links und Fenster-/Startverträge sind versioniert. Jede Sub-App verfügt über eigene Identität und signierte API-Allowlist.

**Begründung und Folgen:** Die Settings-Eltern-App erbt niemals Child-App-Rechte. Eingebettete Fenster, Deep Links und Navigation dürfen App-Kontext, Session, Signatur- oder API-Genehmigung nicht umgehen.

### ADR-150 – Share-Expiry, Ablehnung und Weiterfreigabe

**Status:** Accepted · **Gewählte Option:** B · **Beschlussdatum:** 16.11.2026

**Entscheidung:** Interne Shares können eine optionale Ablaufzeit mit Datum und Uhrzeit erhalten. Nach Ablauf verweigert der Go-Core weitere Zugriffe automatisch und serverseitig, auch über alte Mount-Referenzen, Cache oder Event-Replay. Ersteller und berechtigte Akteure können Freigaben widerrufen, Empfänger sie ablehnen. Weiterfreigabe durch Empfänger bleibt im MVP verboten.

**Begründung und Folgen:** Der Ablauf oder Widerruf eines Share-Grants löscht keine Originaldateien. Der Status muss konsistent über alle Core-Replikate geprüft werden; bereits extern kopierte Daten sind naturgemäß nicht rückholbar.

## 3. Bewusst offen und nicht erneut entschieden

- **ADR-094:** Kubernetes-Produktionshärtung und Secrets bleiben `Proposed`, wie ausdrücklich verlangt. Bestehende technische Sicherheitsinvarianten gelten unverändert.
- **ADR-018, ADR-020, ADR-082, ADR-103, ADR-143, ADR-154 bis ADR-160:** Nicht durch diese Runde akzeptiert; ihre Grundsatz- bzw. Ausbaueinordnung aus dem Register bleibt maßgeblich.
- **ADR-118:** Gerätetests und Kompatibilitätsnachweise sind weiterhin zu erstellen; die Browser-Testmatrix aus ADR-060 ersetzt diese Nachweise nicht.
- **ADR-012 und ADR-060:** Keine ungemessenen RPO/RTO/SLO- oder konkreten Browserversionszusagen.
- **ADR-140:** Keine erfundene Lebensdauer von Recovery-Token.

## 4. Dokumentations- und Implementierungsabgrenzung

Diese Dokumentation enthält ausschließlich fachliche Architekturfestlegungen und keine Quellcode-Implementierung. Der Satz aktualisierter Markdown-Dateien enthält vollständige Ersatzdokumente, keine Patches. Die Veröffentlichung in GitHub erfordert einen eigenständigen Commit durch einen Account mit Schreibberechtigung.
