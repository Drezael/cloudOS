# cloudOS – Beschlussnachtrag: Abschluss der offenen ADRs (16.11.2026)

> **Status:** Fachlich entschieden durch ausdrückliche Delegation des Projektverantwortlichen im Gespräch vom 16.11.2026. **Keine Implementierungs-, Sicherheits- oder Releasefreigabe.**
> **Basis:** `Drezael/cloudOS`, Branch `main`, `docs/ADR_REGISTER.md`, ursprünglicher Git-Blob `0482ba2798146b7be89609bffb30f3be0353c58c`.
> **Geltungsbereich:** Genau 14 zuvor `Proposed`-ADRs; **ADR-094 bleibt unverändert `Proposed`**.

## Ergebnis und Status

| Kennzahl | Nach delegierter Entscheidung |
|---|---:|
| Erfasste globale ADRs | 180 |
| Fachlich `Accepted` | 179 |
| Weiterhin `Proposed` | 1 (`ADR-094`) |
| Neu entschiedene ADRs | 14 |

Diese Entscheidung ändert weder die MVP-Scope-Grenzen noch die Pflicht zu separaten Test-, Lizenz-, Sicherheits- und Abnahmeartefakten. Mehrere ADRs beschließen ausdrücklich eine **deaktivierte spätere Erweiterung** statt sofortiger Aktivierung.

## Vollständige Entscheidungen

### ADR-013

**Thema:** Externe API-Clients und OAuth-/Grant-Modell

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Externe eigenständige API-Clients bleiben im MVP deaktiviert. Für eine spätere gesondert freigegebene Erweiterung ist ein registriertes, providerunabhängiges OAuth-2.0-Sicherheitsprofil nach aktueller Security BCP mit kurzlebigen, audience-, client-, scope- und gegebenenfalls benutzergebundenen Tokens verbindlich. Interaktive Clients verwenden Authorization Code mit PKCE; Maschinenzugriffe benötigen eine eigenständig genehmigte, eng begrenzte Client-/Workload-Identität. Keine allgemeinen statischen Master-API-Keys.

**Begründung und verworfene Richtung:**

Externe Zugänge dürfen weder die signierte App-API-Allowlist noch BFF-Sessions, Resource-Scope oder Tenant-Isolation umgehen. Ein API-Contract allein autorisiert keine externe Veröffentlichung.

**Verbindlicher Freigabe-/Prüfnachweis:**

Vor externer Aktivierung: Client-Registrierung, Consent/Delegation, Token-Rotation und Widerruf, Replay-/SSRF-/Rate-Limit-Tests, AuthZ-Audit und ausdrückliche Betriebsfreigabe.

**Einführung:** Nach MVP, standardmäßig aus.

**Quellen:** [API_CONTRACTS.md](API_CONTRACTS.md), [APP_API_ACCESS.md](APP_API_ACCESS.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

### ADR-018

**Thema:** Wasmtime/WASI-Komponenten und Host-Funktionen

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Serverseitige WASI-Komponenten bleiben im MVP deaktiviert. Für die spätere Erweiterung werden Wasmtime, das WASI Component Model und versionierte, freigegebene WIT-Contracts eingesetzt; Komponenten laufen ausschließlich in einem vom Go-Core getrennten, unprivilegierten Runner. Standardmäßig kein Host-Filesystem, Netzwerk, Environment/Secrets, Docker-Socket oder Datenbankzugang. Jeder Host-Call erfordert einen expliziten instanz-/jobgebundenen Grant.

**Begründung und verworfene Richtung:**

Ausführung von nicht vertrauenswürdigem WASM ist keine Autorisierung und darf den Go-Core nicht kompromittieren.

**Verbindlicher Freigabe-/Prüfnachweis:**

Vor Runtime-Freigabe: gepinnte, getestete Wasmtime-/WASI-/WIT-Versionen; Fuel/Epoch, RAM, Laufzeit, Ausgabe, Parallelität, Egress, Abbruch und Crash-Isolation; Negativtests. Keine Produktionsbehauptung allein aus der Architektur.

**Einführung:** Nach MVP, standardmäßig aus.

**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md).

### ADR-020

**Thema:** ArtCraft als Integrations-PoC

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

ArtCraft ist keine Pflichtabhängigkeit des cloudOS-MVP. Als optionale, unverändert bezogene Rust-Browser-WASM-PoC-Referenz wird PhotoCraft v0.23.1 (Upstream-Tag v0.23.1; Stand 16.11.2026) ausgewählt. Zunächst nur Integrationsstufe A: lokal gehosteter, isolierter App-Host-Start mit Browser-/WebGPU-/Thread-/Origin-Konformität. Stufe B (File Bridge) erst nach eigenem separatem Gate; Stufe C ist kein Teil dieses Beschlusses. Für den Quellcode kann nach Prüfung die Apache-2.0-Lizenzoption gewählt werden; Dritt-Assets und Marken bleiben gesondert geschützt.

**Begründung und verworfene Richtung:**

Ein reproduzierbarer Fremd-App-Test ist nützlich, darf aber weder Build- noch Rechts- oder Produktzwänge für den MVP erzeugen.

**Verbindlicher Freigabe-/Prüfnachweis:**

Vor Ausführung/Distribution: Upstream-Tag, heruntergeladenes Artefakt und SHA-256, Source-/NOTICE-/Asset-/Trademark-Lizenzen, CSP/COEP/CORP, App-Signatur, Browser-Ressourcen und Isolation prüfen. Kein automatisches Repaketieren/Branding.

**Einführung:** Optionaler, nicht blockierender PoC nach Browser-Baseline.

**Quellen:** [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md), [PhotoCraft Releases](https://github.com/storytold/photocraft/releases), [PhotoCraft README](https://github.com/storytold/photocraft#license-and-credits).

### ADR-082

**Thema:** Web Push bei geschlossenem Browser

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Browser-Web-Push bleibt außerhalb des MVP und ist strikt opt-in: ausdrückliche Browser- und cloudOS-Zustimmung pro Benutzer/Gerät, widerrufbare Subscription, serverseitig verwaltete Push-Schlüssel und privacy-minimierter Payload. Externer Push überträgt ausschließlich einen neutralen Zustellungshinweis und niemals vertrauliche Dokumente, Nutzinhalte, Berechtigungsdaten oder ausführbare Aktionen. Der Notification Owner bleibt autoritativ; Details erst nach gültigem Login per bestehender autorisierter Core-API. SSE bleibt für offene Tabs verbindlich.

**Begründung und verworfene Richtung:**

Browser-/Push-Dienst-Verfügbarkeit ist nicht garantiert und Web Push darf keine zweite Rechteautorität werden.

**Verbindlicher Freigabe-/Prüfnachweis:**

Datenschutzprüfung der Push-Provider/Egress-Domäne, Consent, Logout/Revoke, veraltete Subscriptions, SW-Origin-Policy, Browser-Kompatibilität, Zustelllücken/Replay.

**Einführung:** Nach MVP, standardmäßig aus.

**Quellen:** [PUSH_REALTIME.md](PUSH_REALTIME.md), [API_CONTRACTS.md](API_CONTRACTS.md).

### ADR-103

**Thema:** Externe Identity-Verknüpfung zu lokalen Credentials

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Externe Konten werden ausschließlich an stabile lokale `user_id` gebunden. LDAP nutzt stabile source_id + source_object_id; OIDC nutzt `(issuer, sub)`. Keine automatische Fusion anhand E-Mail, Loginname, DN oder Anzeigename. Die erstmalige Verknüpfung erfordert einen nachgewiesenen Besitz-/Authentifizierungsbezug zu beiden Identitäten oder einen explizit genehmigten, step-up-geschützten, auditierten Admin-Workflow. Konflikte werden blockiert/quarantänisiert, keine automatische Rechteübernahme oder Home-Neuanlage durch Providerwechsel.

**Begründung und verworfene Richtung:**

Account-Takeover und Verwechslung vorhandener Benutzerrechte, Homes oder Space-Mitgliedschaften verhindern.

**Verbindlicher Freigabe-/Prüfnachweis:**

Tests für Rename, Dubletten, Identifier-Reuse, Provider-Migration, Revocation, OIDC-Sub-Wechsel, verlorenes externes Konto und sicheren Break-Glass-Pfad.

**Einführung:** Für optionale LDAP/OIDC-Adapter; lokal im MVP unabhängig.

**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md).

### ADR-118

**Thema:** Browserkompatibilität der Geräteprofile

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Der Device Broker veröffentlicht eine browser- und geräteprofilbezogene, getestete Capability-Matrix. WebUSB, WebHID, Web Serial und File-Picker sind niemals generell vorausgesetzt. Pflichtfähigkeiten fehlen: Profilstart/Operation wird mit `UNSUPPORTED` gesperrt; optionale Fähigkeiten werden nachvollziehbar deaktiviert. Kein Umgehen durch untrusted App-iframe, privilegierte Hintergrundzugriffe oder rohe USB-Mounts. Standard-Dateiauswahl und Keyboard-Wedge nur, wenn die jeweilige fachliche Funktion damit sicher vollständig erfüllt ist.

**Begründung und verworfene Richtung:**

Heterogene Browser müssen ehrlich und sicher behandelt werden, ohne implizite Gerätefreigaben.

**Verbindlicher Freigabe-/Prüfnachweis:**

Chrome/Edge/Firefox/Safari mit positiven und negativen Browser-/OS-/Gerätetests; HTTPS, Permissions Policy, User Activation, Revoke, HID-Sperrlisten und Origin-Isolation.

**Einführung:** Profilweise vor jeweiliger Device-Releasefreigabe.

**Quellen:** [CLIENT_DEVICES.md](CLIENT_DEVICES.md), [APP_API_ACCESS.md](APP_API_ACCESS.md).

### ADR-143

**Thema:** Erweiterte POSIX-ACLs und Named Grants

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Named-User/Named-Group-ACLs und Verzeichnis-Default-ACLs werden als optionale spätere VFS-Ausbaustufe nach dem Basismodell `owner:group:mode` eingeführt. Die Auswertung orientiert sich an POSIX-ACL-Einträgen einschließlich effektiver Gruppen-/Named-User-Maske, Vererbungs- und `chmod`-Wechselwirkungen. ACLs erweitern nie Mount-Sicht, Share-/Space-Grant, signierte App-API-Operationen oder aktuelle System-/Deny-Policy. Es gibt keine impliziten Named-ACLs im MVP.

**Begründung und verworfene Richtung:**

POSIX-nahe Semantik ist nur sicher, wenn ein `chmod`- oder ACL-Update nicht unbemerkt andere Grants verstärkt.

**Verbindlicher Freigabe-/Prüfnachweis:**

Vor Freigabe: geprüfte Masken-/Vererbungs-Matrix, `chmod`/`setfacl`-Äquivalenzfälle, Widerruf, Multiuser-/Share-/Space-Negativtests, Migration bestehender Mode-Bits.

**Einführung:** Nach Basis-VFS-MVP, standardmäßig aus.

**Quellen:** [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [VIRTUAL_FILESYSTEM.md](VIRTUAL_FILESYSTEM.md).

### ADR-154

**Thema:** LDAP-Sync-Intervalle und Sicherheitsgrenzen bei Massenänderungen

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Optionaler LDAP-Connector synchronisiert nur read-only via paginierte, idempotente Delta-Abgleiche und regelmäßig verifizierte Vollscans. Ausgangskadenz nach aktivierter Quelle: Delta alle 15 Minuten, vollständiger Abgleich alle 24 Stunden; Jitter, Backoff, zentraler Lease/Fencing-Owner und administrative Anpassbarkeit sind Pflicht. Ein unvollständiger Scan erzeugt niemals `missing`- oder Massen-Disable-Entscheidungen. Für Massenänderungen ist ein konfigurierbarer Guard mit konservativen Ausgangsschwellen (mehr als 5 % oder mehr als 20 betroffene Identitäten je verifiziertem Lauf) plus Dry Run, Audit und step-up-geschützter Freigabe vorgesehen; diese Werte sind vor Produktivfreigabe last-/betriebsseitig zu validieren. Explizit nachgewiesene sicherheitsrelevante Einzel-Entzüge werden zeitnah übernommen, ohne den Schutz vor fehlerhaftem Gesamtimport auszuhebeln.

**Begründung und verworfene Richtung:**

Filterfehler und leere/inkomplette LDAP-Antworten dürfen nicht zum Massenverlust lokaler Identitäten oder privilegierten Fehlzuordnungen führen.

**Verbindlicher Freigabe-/Prüfnachweis:**

Tests: leerer/abgebrochener Paged Scan, Filterwechsel, Source-ID-Tausch, Delta-/Vollscan-Konflikt, Burst-Änderungen, Abbruch, Wiederaufnahme, Quellenstale und Massenänderungs-Freigabe.

**Einführung:** Optionaler LDAP-Connector, standardmäßig aus.

**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [PUSH_REALTIME.md](PUSH_REALTIME.md).

### ADR-155

**Thema:** Missing User, Deaktivierung und Stale-Identity-Richtlinie

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Ein Benutzer gilt erst nach einem vollständig erfolgreichen, konsistenten Quellabgleich als `missing`. Er wird dann kontrolliert quarantänisiert: externe Neuanmeldung und wirksame externe Grants werden gesperrt bzw. widerrufen, erforderliche Sitzungen revalidiert; lokale UUID/UID/GID, Home, Space- und Audit-Referenzen bleiben erhalten. Ein 30-tägiges administrativ änderbares Wiederherstellungsfenster trennt Reaktivierung und endgültige fachliche Deaktivierung; es ist keine automatische Datenlöschung. Wiederauftauchen derselben stabilen Quell-ID ermöglicht nach sicheren Prüfungen Reaktivierung, keine stille Identity-Fusion. Ein Ausfall/Stale-Scan ist niemals Nachweis einer Löschung; für sicherheitskritische Berechtigungen greift eine begrenzte Stale-Policy mit Fail-Closed, ohne pauschale Datenvernichtung.

**Begründung und verworfene Richtung:**

Deprovisioning muss sicher wirken, ohne Konten durch temporäre Providerstörungen oder falsche Filter unwiederbringlich zu zerstören.

**Verbindlicher Freigabe-/Prüfnachweis:**

Missing-vs-outage, 30-Tage-Recovery, Session-/Grant-Revoke, Offline-Leases, VFS-/Space-Erhalt, Reappear mit gleicher/anderer ID, massenhaft fehlende Benutzer.

**Einführung:** Optionaler LDAP-Connector, standardmäßig aus.

**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

### ADR-156

**Thema:** Import externer UID/GID oder interne Nummernvergabe

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Verbindlich ausschließlich lokal kontrollierte UID-/GID-Vergabe nach ADR-138: separate zentrale Nummernkreise, transaktional vergeben, einmalig, keine Wiederverwendung. Externe `uidNumber`/`gidNumber` werden als unverbindliche Quellmetadaten gespeichert und dürfen die lokale Nummer oder Dateieigentümerschaft nicht ersetzen. Es gibt auch unter expliziter Connector-Policy keine direkte externe POSIX-Nummernübernahme; eine spätere echte Migration wäre eine neue separat zu genehmigende Architekturänderung.

**Begründung und verworfene Richtung:**

Kein Konflikt mit bereits entschiedenem ADR-138; Verzeichniswechsel können lokale Dateirechte nicht neu zuordnen.

**Verbindlicher Freigabe-/Prüfnachweis:**

UID/GID-Kollisionen, gleichzeitiger Import, Rename, Restore, Tombstone, Quellenwechsel und Test, dass externe Zahländerungen nie VFS-Rechte umhängen.

**Einführung:** Bei optionalem LDAP-Import, ADR-138 unverändert.

**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [ADR-138](ADR_REGISTER.md#adr-138--uid-gid-nummernkreise-und-wiederverwendung).

### ADR-157

**Thema:** Externe Gruppenverschachtelung und lokale Overlays

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Externe Gruppenverschachtelung ist für die erste optionale LDAP-Integration standardmäßig deaktiviert. Spätere explizite Aktivierung nutzt deterministisches, begrenztes Flattening mit maximal 8 Ebenen (Ausgangslimit), Zyklus-/Duplikaterkennung, Member-/Zeitbudgets und vollständiger Provenienz. Zu tiefes, zyklisches oder unvollständiges Nesting darf keine neuen Berechtigungen gewähren. Lokal administrierte Overlays bleiben gesonderte Gruppen oder explizite Ergänzungen mit eigener Herkunft; keine stille Überschreibung extern verwalteter Mitgliedschaften und kein implizites Mapping zur privilegierten `Admin`-Gruppe.

**Begründung und verworfene Richtung:**

LDAP-Gruppenrekursion darf weder Endlosschleifen noch unkontrollierte Privileg-Eskalation erzeugen.

**Verbindlicher Freigabe-/Prüfnachweis:**

Zyklen, Alias-/DN-Wechsel, tiefe Gruppen, stale Mitgliedschaften, lokale Overlay-Kollisionen, Admin-Impersonation, sofortige Revoke-Wirkung.

**Einführung:** Nesting separat opt-in, erstes LDAP-Profil flach.

**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [SHARING_SPACES.md](SHARING_SPACES.md).

### ADR-158

**Thema:** LDAP-Bind-Login als optionaler Auth-Adapter

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

LDAP-Bind-Login bleibt separat vom LDAP-Verzeichnissync und standardmäßig deaktiviert. Eine spätere Aktivierung erfolgt pro Quelle und zulässiger Konto-Policy durch Administratoren. Zugang nur über streng geprüfte LDAPS-/StartTLS-Verbindungen, begrenzte Authentifizierungsversuche und ohne dauerhafte Speicherung/Protokollierung des Benutzerpassworts. Erfolgreicher Bind erstellt nur nach verifizierter lokaler ID-Bindung und lokaler Policy eine cloudOS-BFF-Sitzung. Kein unkontrollierter Fallback auf lokales Passwort bei LDAP-Ausfall; ein gesicherter lokaler Break-Glass-Admin bleibt unabhängig.

**Begründung und verworfene Richtung:**

Ein LDAP-Bind darf keine Fremd-Passwortkopie, zusätzliche Directory-Autorität oder stillen Identity-Link erzeugen.

**Verbindlicher Freigabe-/Prüfnachweis:**

TLS-Fehler, LDAP-Injection, unbekannte Bindung, Account-Lockout, Provider-Ausfall, Session-Revoke, Backoff, Audit ohne Credential-Leak.

**Einführung:** Nach MVP, standardmäßig aus.

**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_METHODS.md](AUTHENTICATION_METHODS.md).

### ADR-159

**Thema:** Integrationsvertrag externer OIDC-Provider

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Optionaler providerunabhängiger OpenID-Connect-Adapter mit Authorization Code Flow, PKCE und vertraulichem serverseitigem BFF. Verbindlich sind exakte Issuer-/Audience-/Redirect-Validierung, `state`/`nonce`, signierte ID-Token-/JWKS-/Expiry-/Algorithmenprüfung, kontrollierte Schlüsselrotation sowie Bindung an `(issuer, sub)` und eine lokal freigeschaltete `user_id`. Keine automatische Provisionierung oder Rechteübernahme allein aufgrund E-Mail-/Gruppenclaims; neue Benutzer nur durch explizite Admin-Policy und sicheren Provisionierungsworkflow. Lokale Sitzung und Grant-/Revocation-Policy bleiben autoritativ; kein stiller Provider-Fallback.

**Begründung und verworfene Richtung:**

Login-Delegation darf die lokale AuthZ, POSIX-UID/GID und Space-/App-Policies nicht ersetzen.

**Verbindlicher Freigabe-/Prüfnachweis:**

Login CSRF/PKCE/nonce Replay, falscher Issuer/JWKS, Sub-Takeover, Provider-Ausfall, Admin-Gruppenclaim, Token-Rotation, Logout/Session-Stale und Recovery-Test.

**Einführung:** Nach MVP, standardmäßig aus.

**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md).

### ADR-160

**Thema:** Secret-Store und Zertifikatsverwaltung externer Connectoren

**Entscheidung (empfohlene Option, vom Projektverantwortlichen delegiert bestätigt):**

Ein zentraler Core-Secret-Broker verwaltet Connector-Credentials, Auth-Client-Secrets, Zertifikats-/CA-Referenzen und Rotation unter getrennten, minimalen Service-Capabilities. Verwaltet werden nur opake Handles und versionierte Metadaten in normalen Fachschemas; sensible Werte liegen KMS-/Envelope-geschützt in einem separaten Secret-Backend, dessen Root-Schlüssel nicht gemeinsam mit verschlüsselten Nutzdaten in derselben Datenbank liegt. Das Adaptermodell ist providerunabhängig und erlaubt später externe Secret-Manager/HSM. Zertifikatsketten und Hostnamen sind zu prüfen, unsichere Trust-Skip-Optionen produktiv verboten; Rotation atomar mit Test-/Rollback-Status und Audit. Kein Credential im App-Paket, Browser, Quellcode, Log oder normalen App-Data-Feld.

**Begründung und verworfene Richtung:**

Ein kompromittierter Connector darf nicht alle Core- oder Tenant-Secrets offenlegen; Secret-Lifecycle bleibt unabhängig vom LDAP-/IdP-Provider.

**Verbindlicher Freigabe-/Prüfnachweis:**

Secret-ACL-Isolation, Zertifikatsfehler, Trust-Store-Update, Wechsel während LDAP-Job, Rotation/Outage-Rollback, keine Secret-Logs/Traces/Exports, Key-Recovery mit getrennter Freigabe.

**Einführung:** Grundlegende Broker-Schnittstelle im Design; optionaler Connector später.

**Quellen:** [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [INFRASTRUCTURE_DECISIONS_2026-11-15.md](INFRASTRUCTURE_DECISIONS_2026-11-15.md).

## Fachliche Abhängigkeiten und Priorität

1. **MVP unverändert:** lokaler Go-Core/Directory/Auth und SSE, keine externen API-Clients, kein verpflichtendes WASI/LDAP/OIDC/Web Push/Named ACL/ArtCraft.
2. **Identitäten und Rechte:** ADR-103/154–160 sind als kombinierbares, optionales Identity-Integrationsprofil dokumentiert; lokale `user_id`, `group_id`, UID/GID und Autorisierung bleiben autoritativ.
3. **ADR-156 versus ADR-138:** Nur interne Nummernvergabe ist erlaubt; frühere Entwurfstexte, die externen UID/GID-Import per Admin-Policy zuließen, werden bei der späteren fachübergreifenden Aktualisierung ersetzt.
4. **Runtime und Browser:** WASI, Geräteprofile und ArtCraft dürfen erst nach deren individuellen, reproduzierbaren Security-/Browser-/Lizenz-Gates aktiviert werden.
5. **Push:** SSE wird nicht durch Web Push ersetzt. Web Push ist datensparsamer optionaler Benachrichtigungstransport.
6. **Kubernetes:** ADR-094 wurde nicht entschieden. Bereits geltende Härtungsinvarianten bleiben unverändert.

## Noch zu erzeugende technische Nachweise

- Formale Versionierung der OpenAPI-/Protobuf-/AsyncAPI-/WIT-/Device-/Identity-Verträge und reale Konformitätstests.
- Verifizierte, gepinnte Runner-/Browser-/ArtCraft-Versionen und vollständige Asset-/Lizenzprüfung vor Ausführung oder Distribution.
- Für optionale LDAP-Synchronisation: gemessene und ausdrücklich freigegebene Betriebsgrenzen, Schutz bei Massenänderungen, Stale-Identity-/Recovery-Tests.
- Für optionale externe Anmeldung: Provider-Trust, Logout/Revocation, MFA-Assurance, TLS/Secrets/Key-Rotation und separater Break-Glass-Zugang.
- Projektweite Aktualisierung älterer Fachseiten gegen diesen Beschlussnachtrag, einschließlich bislang als offen markierter ADR-Aliase.
- Weiterhin keine automatische Freigabe von ADR-094 oder der technischen Implementierung.

## Quellen

- [Zentrales ADR-Register](ADR_REGISTER.md)
- [Externe Identitäten](IDENTITY_INTEGRATION.md)
- [Authentifizierung und Autorisierung](AUTHENTICATION_AUTHORIZATION.md)
- [Sicherheit und API-Verträge](API_CONTRACTS.md)
- [App-System und WASI](APPLICATION_SYSTEM.md)
- [Push-/Realtime-Architektur](PUSH_REALTIME.md)
- [Client-Geräte](CLIENT_DEVICES.md)
- [Systemarchitektur](SYSTEM_ARCHITECTURE.md)
