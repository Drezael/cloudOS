# cloudOS – Multi-Method-Authentifizierung und Credential-Verwaltung

> **Datei:** `docs/AUTHENTICATION_METHODS.md`  
> **Version:** 0.1 – Architektur für cloudOS 0.12  
> **Stand:** 15.11.2026  
> **Ausgangsbasis:** `Drezael/cloudOS`, `main`, letzter zum Arbeitsbeginn verifizierter Commit `e8c624242845907912ac0ff49969d7e251c51fe3` sowie die vollständig vorbereitete Deployment-Version v0.11  
> **Status:** Verbindlicher Entwurf; die Verfahren sind **noch nicht implementiert**.  
> **Führende Bezüge:** [AUTHENTICATION_AUTHORIZATION.md](AUTHENTICATION_AUTHORIZATION.md), [IDENTITY_INTEGRATION.md](IDENTITY_INTEGRATION.md), [APP_API_ACCESS.md](APP_API_ACCESS.md), [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md), [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md)

## 1. Verbindliches Zielbild

Ein cloudOS-Benutzer kann sich – entsprechend den freigegebenen Richtlinien – auf folgenden Wegen anmelden:

1. **Benutzername oder verifizierte E-Mail-Adresse + Passwort**.
2. **Passkey** über WebAuthn/FIDO2, auch ohne vorherige Eingabe eines Benutzernamens.
3. **SSH-Schlüssel** über einen nachweisbar signierten Challenge-Response-Dialog, niemals durch Übermittlung eines privaten Schlüssels.
4. **YubiKey bzw. kompatibler Hardware-Sicherheitsschlüssel** über den WebAuthn/FIDO2-Weg, passwortlos oder als zweiter Faktor.

Die Verfahren werden für denselben stabilen `user_id` verwaltet. Die App- und VFS-Rechte stammen **nicht** aus dem verwendeten Loginverfahren, sondern weiterhin aus Identität, Gruppen, POSIX-Rechten, App-Zuweisungen, API-Allowlist und Ressourcen-ACLs. Die Loginmethode begründet nur eine definierte **Authentication Assurance Level**-Eigenschaft, die für Step-up-Operationen relevant sein kann.

**Wichtige Trennung:** Passkey und YubiKey sind keine zwei unabhängigen kryptographischen Protokolle. Ein YubiKey kann ein **hardwaregebundener WebAuthn-Authenticator** sein. Die Oberfläche darf beide Einstiegsoptionen anbieten, aber die Sicherheitsprüfung darf ein und dieselbe WebAuthn-Credential nicht doppelt als zwei Faktoren zählen. YubiKey-OTP/PIV werden nicht stillschweigend als WebAuthn behandelt und gehören nur in gesondert genehmigte spätere Adapter.

## 2. Bestehende Basis bleibt erhalten

- Der lokale Go-Identity-/Directory-Service ist zunächst maßgeblich; optional synchronisierter LDAP übernimmt später Provisionierungsdaten.
- Ein später angeschlossener OIDC-IdP ist ein **alternativer Authentifizierungsadapter**; er ersetzt nicht automatisch das interne UID-/GID- und Autorisierungssystem.
- `Settings.cloudApp` bindet die signierte und vorinstallierte `AccountSecurity.cloudApp` ein. Benutzer sehen ihre eigenen Methoden, registrierten Geräte, Sicherheitsereignisse, Sessions und Recovery-Optionen. Administratoren erhalten eine gesonderte Policy-Verwaltung.
- Die Bootstrap-/Login-Ansicht liegt **vor** dem normalen App-Host und ist die eng begrenzte, bereits vereinbarte Ausnahme vom `.cloudApp`-Fensterzwang. Nach erfolgreichem Login werden alle regulären Ansichten als `.cloudApp` in cloudOS-Fenstern dargestellt.
- Alle Identitäts- und Sessionänderungen werden über [PUSH_REALTIME.md](PUSH_REALTIME.md) bekanntgegeben; es gibt kein regelmäßiges Client-Polling.

## 3. Go-Core-Dienstgrenzen

| Core-Komponente | Zuständigkeit | Keine Zuständigkeit |
|---|---|---|
| Identity | Kanonische User-ID, verifizierte Login-Aliase, lokale Gruppen, Credential-Referenzen | Kein Browser-Rendering und kein Geräte-Rawzugriff |
| Authentication/Credential Module | Passwortverifier, WebAuthn-Challenges, SSH-Signatur-Challenges, Enrollment, Recovery | Kein Dateizugriff durch Loginentscheidung |
| Session Service | BFF-Session, Assurance-Level, Sessionrotation, Logout, Revoke | Keine direkte Geheimnisablage im Browser |
| Permission Service | Rechteprüfungen, Step-up-Pflicht bei kritischen Operationen | Keine unkontrollierte Rechtevergabe bei starkem Login |
| Audit/Logging | Unveränderliche Sicherheitsereignisse bzw. redigierte technische Logs | Keine Speicherung von Passwort, WebAuthn-Assertion oder SSH-Privatschlüssel |
| Gateway/BFF | TLS, CSRF, Origin, Request-Binding und Secure-HttpOnly-Cookie | Keine Weitergabe langlebiger Credentials an `.cloudApp` |

Alle Dienste benötigen Ports/Contracts, sodass `combined` heute und `split`/Kubernetes später dieselbe Semantik ausführen. Challenges, Benutzerstatus, Schlüsselwiderrufe, Fehlversuchszähler und Session-Revisionswerte dürfen nicht nur im RAM eines Pods gespeichert sein.

## 4. Identitätsreferenzen, Aliase und Email

- Intern ausschließlich stabile `user_id` (UUID) sowie UID/GID; Namen und E-Mails sind veränderliche Aliase.
- Benutzername eindeutig, Unicode-/Casefold- und Sperrwortregeln zentral festlegen; keine Mehrdeutigkeiten durch nicht eindeutig normalisierte Zeichen.
- E-Mail-Login **nur mit zuvor bestätigter und eindeutig zugeordneter E-Mail**; Änderung erfordert neue Bestätigung.
- Gleicher Fehler für unbekannten Account und falschen Nachweis, ohne Benutzerenumeration; Rate Limits auf Account, IP und Geräte-/Risikokontext.
- LDAP- oder OIDC-Attribute dürfen niemals allein durch eine übereinstimmende E-Mail bestehende lokale Konten übernehmen. Föderierte Bindung verlangt gesonderte, autorisierte Verknüpfung.

## 5. Benutzername/E-Mail + Passwort

**Credential-Lebenszyklus:** Registrierung durch lokalen Administrator oder beaufsichtigtes Enrollment, Passwortwechsel, Kontowiederherstellung, Sperrung, Widerruf aller Sessions bei kritischen Ereignissen. Keine Standardpasswörter und keine Auslieferung eines bekannten gemeinsamen Administrator-Passworts.

**Speicherung:** Argon2id mit individuellen zufälligen Salts, serverseitig konfigurierbarer Arbeitsspeicher- und CPU-Kostenkalibrierung; optional zentral geschützter Pepper. Keine reversible Verschlüsselung von Passwörtern, kein Klartext in Logs, Backups, Tests, Datenexporten oder Client-Speichern. Anpassung von Kostenwerten durch Rehash nach erfolgreichem Login.

**Anmeldung:** HTTPS; CSRF-/Origin-Schutz für den BFF; Schutz vor Credential-Stuffing mittels Rate Limits und kontrollierten Verzögerungen; kein Konto-Lockout ohne definierte Recovery, der ein leichtes DoS gegen fremde Benutzer erzeugt. Passwörter nicht in URL, Headern für allgemeine Logs oder SSE-Events. Administrative Konten können MFA-/phishingresistente Methoden verlangen.

## 6. Passkeys über WebAuthn

### 6.1 Registrierung

1. Ein authentifizierter Benutzer wählt in `AccountSecurity.cloudApp` **Passkey hinzufügen**; für hohe Risiken ist ein frischer Step-up erforderlich.
2. Der Core erzeugt eine kryptographisch zufällige Challenge mit TTL, eindeutiger Enrollment-Transaktion, Benutzerbindung, erwarteter `rpId`, `origin`, `type` und Policy.
3. Der Browser startet `navigator.credentials.create({publicKey: ...})` in einem sicheren Kontext.
4. Der Core überprüft Challenge, `clientDataJSON`, Origin, RP-ID-Hash, Attestation-Format soweit Policy sie fordert, User-Presence und User-Verification, Authenticator-Daten und den öffentlichen Schlüssel.
5. Nur eine erfolgreich geprüfte Credential wird dem `user_id` hinzugefügt. Der Private Key verlässt den Authenticator nie.

Für echte Passkeys werden *discoverable credentials* erzeugt (`residentKey: required`). Der Core speichert eine stabile Credential-ID und den kryptographischen Public Key; Backup-Flags, Transports und ggf. einen Sign-Counter dienen Risikobewertung, nicht einer pauschalen Geräteidentifikation.

### 6.2 Anmeldung

1. Challenge mit engem Ablaufdatum und einmaliger Verwendung erzeugen.
2. Browser ruft `navigator.credentials.get({publicKey: ...})` auf.
3. Core prüft signierte Assertion, erwartete RP-ID/Origin/Challenge, passende registrierte Credential, UV-/UP-Flags und Policy.
4. Challenge atomar konsumieren; nach erfolgreicher Prüfung Session neu erstellen oder rotieren.
5. Assurance-Level und authentifizierte Methoden mit Session speichern; Apps sehen nur minimale, autorisierte Metadaten.

**Origin/RP-ID:** Der Login liegt auf einer dafür festgelegten cloudOS-Origin. `iframe`-Apps dürfen keine WebAuthn-Anmeldung für den Core an eigener Origin nachahmen. Domainwechsel/Migration benötigen einen eigenen Migrationsplan. Für neue Registrierungen sind Replay, Race und parallele Challenge-Verwendung negativ zu testen.

### 6.3 Hardware- und synchronisierte Passkeys

- Hardwaregebundene Security-Keys können je nach Attestation-Policy erkannt werden; eine bloße Herstellerangabe oder ein ungeprüfter AAGUID ist kein sicherer Herstellerbeweis.
- Synchronisierte Passkeys sind möglich und werden nicht fälschlich als hardwaregebunden ausgegeben.
- Hohe Sicherheitsanforderungen können geprüfte Attestation, `userVerification: required` und Gerätetyp-/Issuer-Policy verlangen, wenn diese reproduzierbar unterstützt werden.
- Verlorene Schlüssel lassen sich nach Step-up/Re-Enrollment widerrufen, ohne den Benutzer oder seine Home-Dateien zu löschen.

## 7. YubiKey als Sicherheits-Login

Im Browser ist **WebAuthn/FIDO2** der priorisierte YubiKey-Weg. Die Oberfläche kann **Mit Sicherheitsschlüssel anmelden** anzeigen; die Registrierung erscheint anschließend als Passkey/Security-Key-Credential mit transparenter Gerätestatusangabe. Benutzeraktion (Touch/PIN) ist je nach Authenticator erforderlich. Ein YubiKey kann unabhängig als zweiter Faktor für Passwortanmeldung genutzt werden; derselbe Nachweis wird nicht doppelt gezählt.

YubiKey-OTP, PIV-/Smartcard-Challenge und OpenSSH-FIDO2-Schlüssel sind gesonderte Protokolle. Sie dürfen nur nach eigener Prüfung/ADR aktiviert werden. Insbesondere darf der Client Device Broker den YubiKey nicht durch beliebige WebUSB-Raw-Kommandos als Ersatz für WebAuthn auslesen.

## 8. SSH-Schlüssel-Login (eigener, optionaler Client-Adapter)

### 8.1 Sicherheitsgrenze im Browser

Eine gewöhnliche Webseite kann einen auf dem Client vorhandenen OpenSSH-`ssh-agent` **nicht einfach auslesen oder frei verwenden**. Darum besteht der SSH-Login aus zwei getrennten Teilen:

- **Core-Unterstützung:** Konto-/Key-Bindung, zeitlich begrenzte Challenge, standardisierte SSH-Signaturverifikation, Session-Binding, Widerruf, Logging. Dieser Teil ist browserunabhängig.
- **Client-Unterstützung:** lokal explizit installierter und vom Benutzer gebundener Companion/Browser-Extension mit nativer Messaging-Schnittstelle oder ein separater CLI-Login-Flow. Dieser darf ausschließlich die spezifische Anmelde-Challenge signieren lassen – keine pauschale Agent-/Filesystem-Bridge.

Ohne einen solchen kontrollierten Adapter wird die SSH-Key-Schaltfläche für den betreffenden Browser deaktiviert/mit einer nachvollziehbaren Unterstützungsmeldung versehen. Ein Passkey auf einem FIDO2-YubiKey ist hingegen über WebAuthn möglich und **kein Ersatzbeweis für SSH-Key-Unterstützung**.

### 8.2 Challenge / Signature-Vertrag

- Server erstellt einen kanonischen, versionierten Challenge-Body mit zufälliger Nonce (mindestens 32 Bytes), User-Bindung, RP/Origin-Bindung, Zweck `cloudos/login`, Ablaufzeit, Challenge-ID und Anti-Replay-State.
- Signaturformat: **OpenSSH `sshsig`** mit einer dedizierten Namespace, vorgeschlagen `cloudos-login@cloudos.local` als Vertragstext (vor Produktivbetrieb durch eine eindeutig kontrollierte Domain ersetzen). Kein wiederverwendbarer File-Signing- oder SSH-Transport-Namespace.
- Client zeigt den Anmeldezweck und die Zielinstanz an; ein bekannter Public Key wird im Benutzerkonto registriert.
- Der Go-Core prüft tatsächliche Signatur, Key-/Account-Zuordnung, Namespace, vollständigen Challenge-Body und atomaren Einmalverbrauch. Er akzeptiert keine simplen SHA-Hashes ohne Signatur, keine `ssh-rsa`/SHA-1-Signaturen und keine vom Client behauptete Key-Identität.
- `ssh-ed25519` und moderne RSA-SHA-2-Varianten sind mögliche Profile. Hardwaregebundene `*-sk`-Signaturen brauchen eigene Verträge und Tests für User-Presence-/Verification-Flags.
- **Keine privaten SSH-Schlüssel auf den Server hochladen**. Kein `ssh-agent`-Socket, Browser-Socket oder lokaler Credential-Cache wird einer `.cloudApp` zugänglich gemacht.
- SSH-Key-Login erzeugt **eine cloudOS-Websession**, keinen Linux-Host-Login, kein Shell-/Root-Recht und keinen Kubernetes-Zugang.

## 9. Login-Policy und Auswahl

Pro Installation, Gruppe, Benutzer und Risikoaktion sind Richtlinien möglich:

```json
{
  "schema_version": "1.0",
  "default_methods": ["password", "passkey", "ssh_signature", "security_key_webauthn"],
  "allow_email_login": true,
  "admin_login_requires_phishing_resistant_method": true,
  "admin_step_up_freshness_seconds": 300,
  "ssh_companion_required": true,
  "session_max_age_seconds": 43200,
  "idle_timeout_seconds": 1800,
  "recovery_policy": "admin_assisted"
}
```

Beispiel ist **Ziel-Schema, keine lauffähige Konfiguration**. Das Feld `security_key_webauthn` bezeichnet eine **Policy-Variante des WebAuthn-Protokolls**, keinen zusätzlichen Faktor. Externe OIDC-IdPs können später Authentication-Context-Informationen liefern; der Core muss diese gegen eine vertrauenswürdige Mapping-Policy prüfen statt fremde `acr`-/`amr`-Strings blind zu übernehmen.

**Sicherer Start:** Passwort-Login ist der erste ausbaubare End-to-End-Weg. WebAuthn und SSH-Signaturen werden erst als aktiv angeboten, wenn ihre echte Verifikation implementiert und getestet ist. Die vier Oberflächenoptionen sind verbindlich geplant, aber ein UI-Schalter darf noch nicht implementierte Methoden nicht als funktionsfähig darstellen.

## 10. Sessions, Device-Binding und Reauthentication

- Nach erfolgreichem Login serverseitige Session mit zufälliger ID; `Secure`, `HttpOnly`, `SameSite`-Cookie; keine langlebigen Browser-Bearer-Tokens für Apps.
- Sessionrotation bei Login, Privilegienwechsel und riskanter Wiederanmeldung; CSRF-Schutz bei Cookie-authentifizierten Schreiboperationen.
- Speicherung in gemeinsamem persistentem Session-Owner, nicht nur im Arbeitsspeicher einer Core-Replik. Aktueller Accountstatus und Gruppen-/App-Revisionsstände werden validiert.
- Step-up für fremde Home-Freigaben, Signatur-/Trust-Policy, SSH-Key-Hinzufügen, Recovery, Administrationsrechte und gefährliche Device-Operationen.
- Logout-/Account-Sperr-Events über SSE; auf dem Server sofortige Verweigerung unabhängig von der Push-Zustellung.
- Keine Sessionbindung ausschließlich an eine fragile IP-Adresse; Browser-/Gerätemetadaten nur risikobasiert und datenschutzminimiert.

## 11. Credential-Datenmodell (Entwurf)

| Tabelle / Owner | Kerndaten | Invariante |
|---|---|---|
| `identity.users` | `user_id`, `uid`, `primary_gid`, `state` | Stabile UUID, keine Neuerzeugung bei LDAP-Rename |
| `identity.login_aliases` | `user_id`, `kind`, `normalized_value`, `verified` | E-Mail nur verifiziert, eindeutig |
| `auth.password_credentials` | `user_id`, Argon2id-Hashparameter, `changed_at` | Keine Passwort-/Token-Klartexte |
| `auth.webauthn_credentials` | `credential_id`, `user_id`, `public_key`, `rp_id`, `transports`, UV/Backup-Metadaten | Pro Benutzer/Instanz gebunden, widerrufbar |
| `auth.ssh_public_keys` | `key_id`, `user_id`, `public_key`, `fingerprint`, `state` | Ein registrierter Key pro Bindung; kein privater Teil |
| `auth.challenges` | `challenge_id`, `purpose`, `user_id`, `nonce_hash`, `expires_at`, `consumed_at` | Atomar nur einmal nutzbar |
| `sessions.browser_sessions` | `session_id_hash`, `user_id`, `auth_methods`, `assurance`, `revoked_at` | Alle Replikate prüfen denselben Status |
| `auth.enrollment_events` | Akteur, Ziel, Methode, Zeitpunkt, Ergebnis | Sicherheits-Audit, keine geheimen Payloads |

Passwort- und Verifier-Geheimnisse dürfen nicht über den LDAPv3-Adapter ausgeliefert werden. `UserPassword`-LDAP-Änderungen, falls später gewünscht, erhalten eine gesonderte sichere Bind-/Change-Policy; im ersten lokalen LDAP-Dienst bleibt dieser Zugriff gesperrt.

## 12. REST-Verträge (Entwurf)

| HTTP | Route (Beispiel) | Zweck | Kontrolle |
|---|---|---|---|
| `POST` | `/api/v1/auth/password/login` | Passwort-Login | Öffentlicher Login-Bootstrap, Rate-Limit, CSRF/Origin |
| `POST` | `/api/v1/auth/passkeys/registration/options` | Enrollment-Challenge | Session + Step-up |
| `POST` | `/api/v1/auth/passkeys/registration/verify` | Passkey validieren/speichern | Challenge/Origin/UP/UV prüfen |
| `POST` | `/api/v1/auth/passkeys/authentication/options` | Login-Challenge | Rate-Limit, Risk-Policy |
| `POST` | `/api/v1/auth/passkeys/authentication/verify` | Assertion prüfen | Signatur, One-time, Policy |
| `POST` | `/api/v1/auth/ssh/challenge` | SSH-Login-Challenge | Öffentlicher, begrenzter Auth-Flow |
| `POST` | `/api/v1/auth/ssh/verify` | SSH-Signatur verifizieren | Namespace, Key-Bindung, One-time |
| `GET` | `/api/v1/account/credentials` | Methoden verwalten | Nutzer-Session |
| `DELETE` | `/api/v1/account/credentials/{id}` | Methode widerrufen | Step-up, Schutz gegen Lockout |
| `GET` | `/api/v1/account/sessions` | Eigene Sitzungen | Nutzer-Session |
| `POST` | `/api/v1/account/sessions/revoke` | Sessions sperren | Berechtigt + ggf. Step-up |
| `GET/PATCH` | `/api/v1/admin/auth-policy` | Login-Policy | Administrator + erhöhte Assurance |

Alle Pfade sind vorläufig. Die öffentlichen Login-Endpunkte sind **keine von einer installierten `.cloudApp` frei nutzbaren API-Endpunkte**. Der geprüfte Plattform-Login-Bootstrap darf sie im engen, speziell abgesicherten Context verwenden. `AccountSecurity.cloudApp` muss alle später benötigten geschützten API-Operationen in `permissions/api-access.json` signiert deklarieren und erhält nur freigegebene Scopes.

## 13. Sicherheit und Fehlerfälle

| Fall | Verpflichtendes Verhalten |
|---|---|
| Falsches Passwort/Key | Allgemeine Fehlermeldung; gecappte, auditierte Rate-Limits; keine User-Enumeration |
| Wiederholte Challenge | Immer verweigern; Audit und Risikoanzeige |
| Browser ohne WebAuthn | Methode nicht anbieten; erlaubten, getesteten alternativen Login zeigen |
| SSH-Agent nicht erreichbar | Kein stummes Fallback auf Host-Shell; getrennte Loginmethode anzeigen |
| SSH-Companion manipuliert | Signatur/Nonce/Namespace bindet Antwort an Core-Challenge; Companion selbst darf keine Session als Administrator deklarieren |
| Passkey verloren | Sicherer Widerruf über weiteren Faktor oder beaufsichtigte Recovery |
| Alle Methoden verloren | Kein automatischer, schwacher Passwort-Reset; dokumentierter Admin-Recovery-/Break-glass-Prozess |
| IdP offline | Lokale Methoden nach administrativer Policy, nicht automatisch schwächer als gefordert |
| LDAP-Account deaktiviert | Bestehende Sessions/Passkeys gemäß kanonischem Accountstatus gesperrt; Dateien bleiben gemäß Aufbewahrung erhalten |
| YubiKey im Device Broker | Authentifizierung nur über WebAuthn; `devices.usb.raw` ist kein Weg zur Credential-Extraktion |
| Log-Debug aktiviert | Keine Speicherung von Passwörtern, Authenticator-Antworten, privatem Schlüssel oder vollständigen Sessions |

## 14. Registrierung und Widerruf über Core-Apps

`AccountSecurity.cloudApp` wird als geschützte Core-App in `Settings.cloudApp` angezeigt. Neue Methoden werden nicht frei von Fremd-Apps registriert. Benötigte Funktionen: Passwort ändern, Passkeys hinzufügen/entfernen, SSH-Public-Keys hinzufügen/entfernen, Methodenliste mit Typ/Gerätename/letzter Nutzung, Sessionliste, Sessions widerrufen und Recovery-Status. Admins verwalten globale Regeln und beaufsichtigte Recovery, nicht die privaten Schlüssel anderer Benutzer.

## 15. Externe LDAP- und IdP-Integration

LDAP-Synchronisation übernimmt Identitäten und Gruppen, nicht WebAuthn-/SSH-private Credentials. Benutzer bleiben an `user_id` gebunden. Bei externem OIDC-Login werden interne Sessions nach validierter IdP-Antwort ausgestellt, und nur ausdrücklich gemappte Methoden-/Assurance-Claims übernommen. Lokaler Passwortlogin und LDAP-Bind-Login sind getrennte Betriebsarten; ein externer LDAP-Server muss nicht öffentlich erreichbar sein, damit cloudOS-Anmeldungen funktionieren.

## 16. Push, Audit und Monitoring

`auth.credential.added`, `auth.credential.revoked`, `auth.session.revoked`, `auth.policy.changed`, `identity.user.disabled` werden über die serverseitige, adressatengeprüfte SSE-Architektur zugestellt. Ereignisse enthalten IDs/Revisionsstände und keine Login-Challenges oder Authenticator-Rohdaten. Sichere Audit-Ereignisse dokumentieren Loginversuche, Methodentyp, Ergebnis, Konto, Step-up, Konto-Sperrung und Administratoraktionen. Technische Debuglogs bleiben durch [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md) begrenzt.

## 17. Kubernetes und kombinierter Core

Standard ist `CLOUDOS_MODE=combined` in einem Core-Container. Auth-/Credential-Module werden als Go-Fachdienste implementiert, die später unverändert als mehrere HTTP/gRPC-Replikate laufen. Challenges und Replay-Schutz benötigen transaktionale Speicherung mit gemeinsamer DB-Zeit; TLS-/WebAuthn-RP-Einstellungen sind bei allen Replikaten identisch. Geheimnisschlüssel sind über Secret-Management gebunden und werden niemals als unverschlüsselte lokale Kubernetes-Manifeste ausgeliefert.

## 18. Implementierungsreihenfolge

| Schritt | Arbeit | Fertig, wenn ... |
|---|---|---|
| AUTH-01 | Benutzername/E-Mail-Aliase, Accountstatus und Credential-Ports | Kanonische User-ID und Sessionbindung funktionieren |
| AUTH-02 | Argon2id-Passwortverifier, Rate Limit, CSRF | Passwort-Login, Sperre/Recovery, Tests bestehen |
| AUTH-03 | Geschützte Sessions und Step-up | Rotationen/Revoke/Assurance nachgewiesen |
| AUTH-04 | WebAuthn RP/Origin, Enrollment, Verification | Passkey-Login inkl. Replay-/Origin-Negativtests |
| AUTH-05 | YubiKey-Hardwaretests | FIDO2 Login auf unterstützten Browsern reproduzierbar |
| AUTH-06 | SSH-Signatur-Protokoll, Key-Registry, Verifier | `sshsig` in separater Domain und One-time-Challenge valide |
| AUTH-07 | Companion-/Extension-Integration | Lokaler SSH-Agent kontrolliert durch Benutzer verwendbar |
| AUTH-08 | AccountSecurity Core-App + Admin-Policy | Alle Methoden registrier-/widerrufbar, signierte API-Allowlist |
| AUTH-09 | LDAP-/IdP-Mapping, Recovery und K8s-Replikation | Alias-/Accountstatus sicher, Last-/Failover-Test bestanden |

## 19. Abnahmetests (verbindlich)

| ID | Fall | Erwartung |
|---|---|---|
| AU-001 | Benutzername und Passwort | Login erfolgreich nach Argon2id-Verifikation |
| AU-002 | Verifizierte E-Mail und Passwort | Selbe `user_id`, korrekt normiert |
| AU-003 | Nicht verifizierte E-Mail | Keine Anmeldung über Alias |
| AU-004 | User-Enumeration | Generische Antworten und gleichartige Fehlerklasse |
| AU-005 | Fehlversuchslimit | Kontrollierte Rate-Limits, kein trivialer Lockout-DoS |
| AU-006 | Passwort wird geloggt | Rohwert im Log nicht auffindbar |
| AU-007 | WebAuthn Enrollment | Einmalige Challenge, RP-ID/Origin/UV geprüft |
| AU-008 | Passkey-Login | Korrekt signierte Assertion wird anerkannt |
| AU-009 | Replay der Assertion | Verweigert |
| AU-010 | Falsche RP-ID oder Origin | Verweigert |
| AU-011 | WebAuthn von App-iframe | Kein unkontrollierter Core-Login |
| AU-012 | Hardware-YubiKey mit FIDO2 | Authenticator funktioniert, kein Raw-USB nötig |
| AU-013 | Synchronisierter Passkey | Korrekt erkannt ohne gefälschte Hardware-Attestation |
| AU-014 | SSH-`sshsig`-Signatur | Challenge und Namespace geprüft |
| AU-015 | SSH-Replay bzw. falscher Namespace | Verweigert |
| AU-016 | Nicht registrierter SSH-Public-Key | Verweigert |
| AU-017 | SSH ohne Companion | Kein angeblich funktionierender Browser-only-Login |
| AU-018 | SSH-Private-Key-Upload | Keine Upload- oder Speicherroute vorhanden |
| AU-019 | YubiKey-Faktor doppelt gezählt | Dieselbe Assertion zählt nicht doppelt |
| AU-020 | Sessionrotation bei Login | Alte Session-ID ungültig |
| AU-021 | Step-up-Ablaufzeit | Kritische Aktion erfordert frischen Nachweis |
| AU-022 | Benutzer deaktiviert | Alle Core-Replikate verweigern weitere Rechte |
| AU-023 | Auth-Policy geändert | Revision wird per SSE adressatengerecht verteilt |
| AU-024 | LDAP-Rename | Credential-Bindungen bleiben an User-ID |
| AU-025 | IdP-Claims manipuliert | Keine automatische höhere Assurance |
| AU-026 | Zwei Replikate verifizieren eine Challenge | Genau eine kann sie erfolgreich konsumieren |
| AU-027 | Debug-Logging | Keine Challenge-/Secret-Payloads |
| AU-028 | Lost-Key-Recovery | Mehrstufig, auditiert, kein stiller Bypass |
| AU-029 | Ungültiges AccountSecurity-App-Recht | Geschützter Endpoint gesperrt |
| AU-030 | Browser-Login ohne Desktop-App | Minimaler Bootstrap funktioniert unabhängig vom App-Lifecycle |

## 20. Vorgeschlagene Architekturentscheidungen

| ADR | Thema |
|---|---|
| ADR-099 | Kanonische Credential-Lifecycle- und Aliasspeicherung |
| ADR-100 | Passkey-/WebAuthn-RP-Policy, Backup-Flags, Attestation |
| ADR-101 | SSH-Signatur-Companion, `sshsig`-Namespace, Agent-Boundaries |
| ADR-102 | Sicherheitsstufen, Step-up und administrativer Recovery |
| ADR-103 | Externe OIDC-/LDAP-Bindung an lokale Credentials |

## 21. Technische Quellen

- [MDN: Passkeys](https://developer.mozilla.org/en-US/docs/Web/Security/Authentication/Passkeys)
- [MDN: Web Authentication API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API)
- [W3C WebAuthn](https://www.w3.org/TR/webauthn/)
- [OpenSSH `sshsig`-Format](https://github.com/openssh/openssh-portable/blob/master/PROTOCOL.sshsig)
- [OpenBSD `ssh-keygen`-Dokumentation](https://man.openbsd.org/ssh-keygen.1)
- [Yubico: SSH und FIDO2](https://developers.yubico.com/SSH/)
- [Yubico: WebAuthn](https://developers.yubico.com/WebAuthn/WebAuthn_Developer_Guide/)

## 22. Statusabgrenzung

Dieses Dokument ist die **Architektur**. Die Go-Deployment-Bootstraps aus 0.11 besitzen noch keine produktive Passwort-, WebAuthn-, SSH- oder YubiKey-Verifikation. Es dürfen bis zur tatsächlichen Implementierung keine Login-Routen aus Mockantworten einen erfolgreichen Login simulieren.
