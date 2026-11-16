# cloudOS – Einheitliches Window Framework und Taskleisten-Integration

> **Theming und dynamische Wallpapers v0.13:** [THEMING.md](THEMING.md) – sechs OS-inspirierte Themes und einheitliche Theme-Tokens; [WALLPAPER_ENGINE.md](WALLPAPER_ENGINE.md) – Milky Way, Waves, Hexells, Matrix 2D/3D, Coastal Landscape. Beide Dokumente definieren ausschließlich Konzepte, keine implementierten Dienste.


> **Datei:** `docs/WINDOW_FRAMEWORK.md`  
> **Version:** 0.1 – Architektur für cloudOS 0.12  
> **Stand:** 15.11.2026  
> **Status:** Verpflichtender App-UI-Vertrag; Fenster- und Taskleisten-APIs sind noch nicht implementiert.  
> **Core-Deployment:** [CORE_DEPLOYMENT_SCALING.md](CORE_DEPLOYMENT_SCALING.md); zunächst Combined.  
> **Referenzen:** [DESKTOP_ENVIRONMENT.md](DESKTOP_ENVIRONMENT.md), [APPLICATION_SYSTEM.md](APPLICATION_SYSTEM.md), [APP_API_ACCESS.md](APP_API_ACCESS.md), [PUSH_REALTIME.md](PUSH_REALTIME.md), [CLIENT_DEVICES.md](CLIENT_DEVICES.md), [LOGGING_OBSERVABILITY.md](LOGGING_OBSERVABILITY.md)

## 1. Entscheidung

**Jede regulär sichtbare `.cloudApp` wird ausschließlich innerhalb eines vom cloudOS-Framework erzeugten Fensters gerendert.** Eine App kann Inhalt und unterstützte Bedienaktionen liefern, aber **nicht** selbst die übergeordnete Fensterschale, die globalen Taskleisten-Einträge, privilegierte Systemdialoge oder den Window Manager ersetzen. Das Fensterframework stellt einheitliche Steuerung, Theme-Hooks, Fensterzustände, Taskleisten-Integration und Accessibility bereit.

**Ausnahmen:** `Desktop.cloudApp` ist die registrierte Shell-App und rendert die Arbeitsfläche einschließlich verwalteter Fensterschalen. Das minimal nötige Login-/Recovery-Bootstrap liegt vor der Desktop-Shell. Ein reiner Hintergrundjob (`wasi-component`) benötigt keinen GUI-Fenster-Host; sobald er eine GUI öffnet, geschieht dies über eine registrierte `.cloudApp`-Fensterinstanz. Notifications, Kontextmenüs und Dialoge können vom Framework als kontrollierte Overlay-/Dialogflächen gerendert werden, aber nicht als unkontrollierte Fremd-App-DOM-Wurzel.

## 2. Bestandteile und Datenhoheit

| Komponente | Verantwortung |
|---|---|
| `Desktop.cloudApp` | Shell, Workspace, Taskleiste, Startmenü und UI-Komposition |
| Window Host / Window Manager SDK | Echtes Fenster-Chrome, Drag/Resize/Snap, Fokus, Z-Order, Min/Max/Restore, Theme, A11y |
| App Host / App Bridge | Sandboxed Browser-/Rust-WASM-App-Inhalt in zugewiesener Viewport-Fläche |
| Go Desktop/Windows Owner | Persistierbare Fenstermetadaten, App-/Window-IDs, autorisierte Aktionen, Layoutrevisionen |
| Go App Lifecycle Owner | Start/Stop-Status einer installierten und dem Benutzer zugewiesenen App-Instanz |
| Permission Service | App-API-Allowlist, App-Zuweisung, Benutzer- und Aktionsrechte |
| SSE Push | Adressatengerechte Updates von Zustand, Fensterliste, Start- und Stopereignissen |

**Ein Window-Host, viele Apps:** Der Wrapper bleibt vertrauenswürdiger cloudOS-Code. Ein App-iframe oder Canvas erhält nur seinen Innenbereich und eine validierte Bridge. App-Code darf keine Host-DOM-Knoten außerhalb seines eigenen Inhalts verändern.

## 3. Fensterstruktur

```text
Desktop.cloudApp (Shell-Root)
└── Workspace / Window Layer
    ├── Framework Window A
    │   ├── Chrome: icon + title + buttons + frame + resize handles
    │   └── App Viewport: Web/WASM iframe oder SDK-Mount
    ├── Framework Window B
    │   ├── Chrome: ...
    │   └── App Viewport: ...
    ├── Trusted Modal / System Dialog Layer
    ├── Framework-owned context menus / overlays
    └── Taskbar/Start Panel
```

Alle Fenster besitzen **stabile `window_id`**, `app_id`, `app_instance_id`, `owner_user_id`, `browser_session_id`, `workspace_id` und einen monotonen UI-Zustands-/Serverrevisionenbereich. App-Anzeigenamen und Icons stammen aus dem geprüften Paketmanifest, nicht aus einem frei eingestellten Titel, der eine andere App oder einen Systemdialog vortäuschen könnte.

## 4. Unterstützte Zustände

| Zustand | UI-Verhalten | Taskbar |
|---|---|---|
| `opening` | Framework reserviert Fenster, wartet auf App-Host | Loading-Zustand |
| `normal` | Frei beweglich/skalierbar, innerhalb Viewport | Running-Markierung |
| `focused` | Oberstes aktives Fenster; Tastaturfokus | Aktives Taskbar-Element |
| `minimized` | Nicht sichtbar auf Workspace, App läuft ggf. weiter | Running, kein aktiver Fokus |
| `maximized` | Füllt freie Arbeitsfläche ohne Taskleiste | Running, aktiv |
| `snapped` | In Snap-Zone ausgerichtet, z. B. Hälfte/Viertel | Running, aktiv |
| `hidden` | Durch Workspacewechsel verborgen, nicht gestoppt | Je nach Workspace-Policy |
| `closing` | App wird um ungespeicherte Daten befragt | Schließen-Indikator |
| `closed` | Fenster entfernt; ggf. App-Lifecycle beendet | Laufmarkierung entfernt, wenn kein anderes Fenster |
| `crashed` | Vertrauenswürdiger Fallback mit Neustart/Fehler | Fehleranzeige statt unsichtbarer Restinstanz |

**Wichtig:** `minimized` ist nicht `closed`. `hidden` ist nicht `stopped`. Ein Fenster kann beendet werden, ohne einen freigegebenen Hintergrundjob zu töten, wenn die App eine legitimierte Background-Policy hat. Ein „App beenden“ aus dem Kontextmenü stoppt dagegen die vollständige ausgewählte App-Instanz nach dem dokumentierten Lifecycle; privilegierte Arbeit darf dabei nicht unkontrolliert fortbestehen.

## 5. Bedienung wie bei Desktop-Betriebssystemen

Pflicht für Maus/Trackpad/Tastatur:

- Fenster per Titelleiste bewegen; Resize über Kanten und Ecken.
- Fenster fokussieren, Z-Reihenfolge verändern, aktive/inaktive Fenstertitelleisten unterschiedlich darstellen.
- Minimieren in die Taskleiste, maximieren in die verfügbare Arbeitsfläche, aus maximiertem Zustand wiederherstellen.
- Doppelklick auf die Titelleiste zum Maximieren/Wiederherstellen.
- Snap an Bildschirmränder (zuerst links/rechts, später Viertel/Layouts), Vorschau und sichere Constraints.
- Tastatursteuerung: `Alt+Tab` bzw. dokumentierte Plattformtaste für Fensterwechsel, `Alt+F4`/äquivalenter Shortcut für Fenster schließen, Escape für unkritische Dialoge, Fokus-Sichtbarkeit und Tastaturbedienbarkeit.
- Fenster können hintereinander liegen und dürfen nicht außerhalb des zugänglichen Viewports dauerhaft unerreichbar werden.
- Modal- und Systemdialoge blockieren nur die für sie geltenden App-/Session-Aktionen; sie werden **nicht** vom App-Code mit höherer Vertrauensstufe gefälscht.
- Responsive Mindestgrößen, Browser-Zoom, mobile Bedienbarkeit und hohe DPI bleiben Teil des Fenstercontracts.

## 6. Taskleiste und Fenster als eine Zustandseinheit

Die Taskleiste zeigt **geöffnete** und **angeheftete** Apps aus der autorisierten Registry. Ein Pin in `/.taskbar.json` ist **keine laufende Instanz**. Für laufende Apps ist der Desktop/Window Owner die Quelle der Fensterliste; die Taskleiste bildet den Zustand daraus ab. Eine App, der eine Benutzer-/Gruppenfreigabe entzogen wurde, verschwindet ohne Reload aus beiden Listen, und alle betroffenen Instanzen werden gemäß Widerrufspolicy beendet.

### 6.1 Linksklick / reguläre Aktion

- Nicht laufende, angeheftete App: Start über App Lifecycle, danach Framework Window öffnen.
- Eine laufende App mit einem Fenster: Wenn minimiert → wiederherstellen und fokussieren; wenn im Hintergrund → fokussieren; wenn bereits im Fokus → je nach dokumentierter UX-Policy minimieren oder unverändert lassen (ADR).
- Eine App mit mehreren Fenstern: gruppiertes Menü/Fenstervorschau anzeigen; Auswahl aktiviert das gewünschte Fenster. Keine willkürliche Beendigung anderer Instanzen.

### 6.2 Rechtsklick auf App in der Taskleiste

Vom **Framework** erzeugtes Kontextmenü (mit Theme-Komponenten):

```text
File Explorer
├── Fenster wiederherstellen       (falls minimiert)
├── Fenster maximieren            (falls erlaubt)
├── Fenster minimieren             (falls laufend)
├── An Taskleiste anheften / lösen
├── Neues Fenster öffnen           (nur falls App unterstützt)
├── Dieses Fenster schließen
└── App beenden                    (alle Fenster dieser Instanz)
```

**Pflicht:** Ein Rechtsklick auf eine **laufende** Taskleisten-App erlaubt **Schließen/Beenden** wie unter Windows/Linux. Der Kontextmenüeintrag „App beenden“ ist der explizite Lifecycle-Befehl. Wenn mehrere Instanzen bestehen, gibt es eine verständliche Auswahl oder „Alle Instanzen beenden“ mit korrekter Scope-Prüfung. Eine nur angeheftete, nicht laufende App erhält **keine** irreführende Schließen-Aktion.

### 6.3 Taskleistenposition

Die bereits vereinbarte `/.taskbar.json` bleibt alleinige Quelle für Position (`top`, `bottom`, `left`, `right`), Pins und persönliche Layoutoptionen. Änderungen aktualisieren laufende Taskbar-Instanzen per autorisiertem SSE; Fenster-Maximierung/Snap berechnen die verfügbare Arbeitsfläche neu, sobald die Taskbar-Seite geändert wird.

## 7. Fenster-Lifecycle und Close-Protokoll

```mermaid
sequenceDiagram
    participant U as Nutzer
    participant TB as Taskbar / Window Framework
    participant D as Desktop/Window Owner
    participant A as App Runtime
    participant L as App Lifecycle Owner
    U->>TB: Rechtsklick → Dieses Fenster schließen
    TB->>D: RequestCloseWindow(window_id, action_id)
    D->>D: Benutzer/Session/App/Window prüfen
    D->>A: beforeClose(reason=user, deadline)
    alt Keine ungespeicherten Änderungen
        A-->>D: allow
        D->>L: Fenster entfernen; letzte Instanz ggf. beenden
        L-->>TB: lifecycle/window.closed (SSE)
    else Ungespeicherte Änderungen
        A-->>D: needsConfirmation
        D-->>TB: Framework-Save/Discard/Cancel-Dialog
        U->>TB: Entscheidung
        TB->>D: confirmClose oder cancel
        D->>L: Nach Freigabe beenden bzw. abbrechen
    else App reagiert nicht
        D-->>TB: Timeout / App antwortet nicht
        U->>TB: Abbrechen oder erzwungen beenden
        TB->>D: ForceStop (Policy + Bestätigung)
    end
```

**Sicherheit:** Ein Client-`beforeClose` ist UX-Hilfe, **keine Autorisierung**. Er darf keine Administrator-Entscheidung verhindern. Für einen erzwungenen Abbruch gibt es separaten Status, Audit und Aufräumen von App-Capability- und Device-Grants. Der Server bestätigt `closed` erst nach Lifecycle-Commit; temporäre UI-Optimierungen korrigieren sich durch Push/Revision. Ein fremder `window_id` darf nicht geschlossen werden.

## 8. API- und SDK-Vertrag

```json
{
  "schema_version": "1.0",
  "window": {
    "window_id": "window-uuid-example",
    "app_id": "org.example.editor",
    "app_instance_id": "instance-uuid-example",
    "title": "Example Editor",
    "state": "normal",
    "bounds": { "x": 120, "y": 65, "width": 980, "height": 640 },
    "min_size": { "width": 340, "height": 220 },
    "capabilities": ["minimize", "maximize", "resize", "close"],
    "revision": 8
  }
}
```

Alle Werte werden validiert. Künftige `cloudos.window`-SDK-Funktionen sind `requestOpen`, `requestClose`, `requestFocus`, `requestMinimize`, `requestMaximize`, `requestRestore`, `requestSetBounds`, `onResize`, `onFocus`, `onClose`, `onTheme`. Eine App kann **Wünsche** anmelden; der Desktop Owner entscheidet, was erlaubt ist.

| Methode | Beispiel-Route | Scopes |
|---|---|---|
| `POST` | `/api/v1/windows` | `windows.open:own_app` |
| `GET` | `/api/v1/windows` | `windows.list:own_session` oder geschützte Desktop-Core-App |
| `POST` | `/api/v1/windows/{id}/focus` | `windows.focus:own_window` |
| `POST` | `/api/v1/windows/{id}/minimize` | `windows.minimize:own_window` |
| `POST` | `/api/v1/windows/{id}/restore` | `windows.restore:own_window` |
| `POST` | `/api/v1/windows/{id}/close` | `windows.close:own_window` |
| `POST` | `/api/v1/apps/instances/{id}/stop` | `apps.stop:own_instance` bzw. Core-Owner-Scope |

Routes und Operation-IDs sind Vorschläge, die später in OpenAPI festgelegt werden. Eine Drittanbieter-App erhält **nicht** automatisch `windows.list:all_users`, `apps.stop:any` oder einen Zugriff auf fremde Windows. Jeder App-Aufruf muss in `permissions/api-access.json` stehen und bei Installation sowie zur Laufzeit genehmigt werden. Die **trusted Desktop-/Taskbar-Core-App** darf im engen Core-Baseline-Contract die Fenster der eigenen Sitzung verwalten.

## 9. Signiertes `.cloudApp`-Manifest

Eine sichtbare App enthält einen geprüften GUI-Vertrag im signierten Paket:

```json
{
  "id": "org.example.editor",
  "runtime": { "type": "web-wasm", "entrypoint": "web/index.html" },
  "window": {
    "mode": "framework",
    "default_width": 980,
    "default_height": 640,
    "min_width": 340,
    "min_height": 220,
    "resizable": true,
    "multi_window": true,
    "unsaved_changes_hook": true
  }
}
```

Der neue Abschnitt wird in die bestehenden Schemaentwürfe integriert; die im Beispiel gezeigten Felder sind **kein vollständiges finales App-Manifest**. `window.mode` muss für sichtbare Apps `framework` sein; Werte wie `native_popup`, `raw_desktop_root` oder `unmanaged` werden nicht als normale App-UI akzeptiert.

## 10. Browser-Isolation und Rendering

- Web-/Rust-WASM-Apps werden aus geprüften, installierten Paketen geladen und im kontrollierten **App-Viewport** dargestellt. Standard bei nicht privilegierten Apps: eigene Origin und Sandbox-iframe; Browser-/WASM-Assets nur aus freigegebenem, digestgebundenem Paket.
- `window.open`, Pop-up-Rechte, DOM-Escape, Top-Level-Navigation und ungeprüfte `postMessage`-Befehle werden im App-Sandbox- und Host-Contract eingeschränkt.
- WebGPU-/Canvas-Inhalte sind ebenfalls nur **Fensterinhalt**. Sie können das gehostete Fenster nicht eigenmächtig verkleinern, neu anordnen oder visuell eine systemvertrauenswürdige Titelleiste ersetzen.
- Cross-Origin-App-Styles werden nicht auf die Shell übertragen. Das Theme-System gestaltet das Framework-Fenster, nicht automatisch eine fremde App-Canvas.
- Systemdialoge (Device-Grants, Installation, API-Risiken, Signaturwarnungen) müssen vom vertrauenswürdigen Framework gerendert werden. Drittanbieter-Apps dürfen diese nicht durch täuschend echte Eigen-Overlays ersetzen.

## 11. Zustand und Persistenz

- Fensterbewegung, Drag und Resize-Vorschau bleiben **clientlokal** für flüssige Reaktion; synchronisiert wird das Endergebnis oder eine begrenzte Frequenz mit Optimistic Concurrency (`revision`/ETag).
- Der Desktop/Window Core Owner persistiert Sessionfensterliste, zuletzt verwendete Geometrien und erlaubte Restore-Daten gemäß bereits bestehendem Desktop-Konzept. `/.desktop-layout.json` bleibt für Icon-Positionen, **nicht** für einen zweiten, fremden Fensterkatalog.
- `/.desktop/` bleibt Datei-/Shortcut-Datenquelle; Fensterliste nicht dort ablegen. `/.taskbar.json` bleibt für Pins/Position, **nicht** für laufende Prozesse.
- Nach Login müssen Fenster gegebenenfalls mit neuen `app_instance_id`s rekonstruiert werden. Kein Browser-/Linux-RAM-Snapshot wird vorgetäuscht.
- Browser-Tab A und B haben getrennte kurzfristige UI-Zustände, aber dieselben serverseitigen App-Zuweisungen, Berechtigungen und Richtlinien. Kollisionen über Revision und definierte Restore-Policy lösen.

## 12. SSE-Events ohne Polling

| Event | Zweck |
|---|---|
| `windows.opened` | Neues Fenster in autorisierter Sitzung |
| `windows.focus.changed` | Aktiver Window Owner/Tab entsprechend Sessionmodell |
| `windows.state.changed` | Min/Max/Restore/Snap/Bounds geändert |
| `windows.closed` | Fenster endgültig entfernt |
| `apps.instance.stopped` | Taskbar-Running-Status entfernen |
| `apps.assignment.revoked` | Fenster sofort sperren/schließen; Pin ohne App-Recht verstecken |
| `desktop.taskbar.changed` | Position/Pins ohne Reload aktualisieren |
| `devices.grant.revoked` | Betroffene App und Fenster erhalten Statusänderung |

Server Push enthält nur Rechte/Window-Scope des Empfängers und keine beliebigen fremden Nutzerfenster. Nach Reconnect: Cursor-Replay; falls unmöglich, einmaliger autorisierter Zustandssnapshot. **Kein Client-Polling.**

## 13. Sicherheit und Angriffsszenarien

| Angriff / Fehler | Gegenmaßnahme |
|---|---|
| App versucht andere App zu schließen | Prüfung `window_id`, `app_instance_id`, User, Session und Operation-ID |
| App fälscht Systemdialog | Trusted Host-Chrome, getrennte Origins, kein App-Zugriff auf Shell-DOM |
| App nutzt `window.open` | Sandbox/Policy verweigert unkontrollierten Pop-up/Top-Level-Navigation |
| App spamt Fenster | Pro-App-/Benutzer-Limits, Rate Limiting, Quotas |
| Taskbar-Icon fingiert laufende App | Registry/Window Owner bestimmt Running-State, nicht selbst gemeldetes App-Event |
| Rechteentzug während App läuft | Sofortige Server-Sperre, gezieltes Push, instanzbezogene Bereinigung |
| App reagiert nicht auf Close | Timeout, Nutzerentscheidung, Lifecycle-Finalisierung |
| Browser verliert Verbindung | Lokaler Zustand wird als offline angezeigt, bei Reconnect autorisierter Abgleich |
| Browser tab geschlossen | Leases/Timeout/OnUnload nur Hinweis; Server bereinigt auch ohne zuverlässiges OnUnload |
| Ein Gerät wird beim App-Ende noch verwendet | Device Broker widerruft Session-Grants und schließt Handles |

## 14. Theme und Accessibility

Das Framework bietet **Design-Tokens für jedes sichtbare Chrome-Element**, Focus, Hover, Resize, Drag-Ghost, Taskbar, Menüs und Modals. `window.chrome` kann über signierte, geprüfte Varianten angepasst werden; die Sicherheitssemantik der Close-/Grant-Buttons bleibt unverändert. ARIA-Rollen, Tastaturfokus, Screen-Reader-Ansagen, hohe Kontraste, `prefers-reduced-motion` und sichtbare Fokusindikatoren sind Pflicht.

## 15. Multi-Window, Workspaces und mehrere Browser

- Eine `app_instance_id` kann gemäß Manifest mehrere `window_id`s besitzen. Das Schließen eines Fensters stoppt die App nur, wenn es das letzte relevante Fenster ist und keine ausdrücklich freigegebene Hintergrundaktivität läuft.
- Taskbar gruppiert per `app_id`, optional je Instanz. Auf mehreren Browser-Tabs werden nur Windows der dafür vorgesehenen Session-/Workspace-Sicht angezeigt; keine globale Offenlegung fremder Sitzungen.
- Mehrere Workspaces sind spätere Erweiterung; der Grundvertrag verwendet bereits `workspace_id`, damit künftige virtuelle Desktops keinen Umbau des gesamten Fenstermodells erfordern.
- Kein raw Multi-Monitor-API-Zugriff erforderlich. Browser-Vollbild / Screen-Presentation ist eine separate browserseitige Berechtigung.

## 16. Nicht-Fenster-Runtimes

Ein serverseitiger WASI-Job oder ein geplanter native-isolated Hintergrundprozess besitzt keinen sichtbaren Window Host. Sobald eine begleitende GUI benötigt wird, startet eine installierte, dem Benutzer zugewiesene `.cloudApp` und rendert sich im Framework-Fenster. Hintergrundarbeit ist nicht das Recht, ein unabhängiges Browserfenster außerhalb des Frameworks zu öffnen.

## 17. Skalierung und Deployment

Der Window Manager selbst ist **browserseitiger Teil der signierten Desktop-Core-App**, der Go Desktop/Windows Owner liefert autorisierte, durable Zustände. In `combined` läuft er mit allen Diensten in einem Core-Container, im `split`-/Kubernetes-Modus als eigenständiger `desktop`-Service. Wir benötigen für die Fenstersteuerung **keinen zweiten separaten Go-Service**; das würde das bisherige Ownership-Modell unnötig aufbrechen.

## 18. Umsetzungsschritte

| Phase | Beschreibung | Gate |
|---|---|---|
| WIN-01 | Core Window DTOs und App-Manifest-Schema | Validierte Typen/Bounds und Operation-IDs |
| WIN-02 | Trusted Window Host in Desktop.cloudApp | Jedes normale App-Viewport ist in Framework-Fenster |
| WIN-03 | Fokus, Min/Max/Resize, Z-Order | Keyboard/Maus-Tests |
| WIN-04 | Taskbar-State-Verknüpfung und Kontextmenü | Rechtsklick → App beenden funktioniert |
| WIN-05 | Graceful Close, Unsaved-Dialog, ForceStop | Kein instabiler Restprozess / Leak |
| WIN-06 | SSE-Events, Revisionen, Multi-Tab | Ohne Reload konsistent |
| WIN-07 | Snap, Multi-Window, Accessibility, Themes | Visual/A11y-Regression |
| WIN-08 | Skalierbarer Go Desktop Owner | Replikat-/Failover-Tests |

## 19. Abnahmematrix

| ID | Test | Soll |
|---|---|---|
| WIN-001 | App ohne Framework-Window | Start-/Renderanforderung verweigert |
| WIN-002 | `Desktop.cloudApp` nach Login | Shell vorhanden, Arbeitsfläche und Taskbar sichtbar |
| WIN-003 | App startet | Trusted Chrome + korrektes App-Viewport |
| WIN-004 | App minimieren | Fenster weg, Taskbar-Running sichtbar |
| WIN-005 | Taskbar-Klick auf minimierte App | Wiederherstellen + Fokus |
| WIN-006 | Taskbar-Klick auf laufende App | Korrektes Fenster aktivieren |
| WIN-007 | Taskbar-Rechtsklick | Menü mit Schließen/Beenden |
| WIN-008 | „Dieses Fenster schließen“ | Nur gewähltes Fenster beendet |
| WIN-009 | „App beenden“ | Zugehörige Instanz und Fenster beendet |
| WIN-010 | Ungespeicherte Arbeit | Framework-Save/Discard/Cancel-Dialog |
| WIN-011 | App hängt beim Schließen | Timeout und bewusstes ForceStop |
| WIN-012 | Mehrere Fenster einer App | Gruppiert, getrennte Fensteraktionen |
| WIN-013 | Minimiert vs gestoppt | Nicht verwechselt |
| WIN-014 | Maximieren | Taskbar-Arbeitsfläche berücksichtigt |
| WIN-015 | Taskbar von unten nach links | MaxBounds/Snap neu berechnet |
| WIN-016 | Snap links/rechts | Korrekte Geometrie ohne Fensterverlust |
| WIN-017 | Doppelklick Titelleiste | Max/Restore |
| WIN-018 | Drag/Resize | Flüssig ohne Serverroundtrip je Pixel |
| WIN-019 | Cross-User window_id | Serverseitig verweigert |
| WIN-020 | App spoofed `window.open` | Unterbunden |
| WIN-021 | Falsche App-ID in Window-Request | Abgelehnt |
| WIN-022 | App-Grant widerrufen | Fenster sofort geschützt, Taskbar aktualisiert |
| WIN-023 | Zwei Tabs / SSE-Replay | Konsistent ohne Polling |
| WIN-024 | Crash im iframe | Framework-Error und Restart-Aktion |
| WIN-025 | Rust-WASM/Canvas-App | Im zugewiesenen Fenster-Viewport |
| WIN-026 | Trusted Device-Grant-Dialog | Kein App-DOM-Spoofing |
| WIN-027 | Theme-Wechsel | Fenster/Taskbar ohne Reload angepasst |
| WIN-028 | Tastatur/Screenreader | Focus/ARIA/Shortcuts ausreichend |
| WIN-029 | Browser offline | Klarer Offline-Zustand, keine Phantom-Lifecycle-Antwort |
| WIN-030 | Split-Deployment mit zwei Desktop-Replikaten | Keine Doppelinstanzen / fremden Fenster |

## 20. Vorgeschlagene ADRs

| ADR | Inhalt |
|---|---|
| ADR-109 | Verbindlicher Window Host für alle sichtbaren Apps |
| ADR-110 | Fensterzustände, Taskbar-Gruppierung und Close-Semantik |
| ADR-111 | Runtime-Bridge, Sandbox und Modal-Sicherheitsgrenze |
| ADR-112 | Workspace-/Multi-Tab-Zustand und Owner-Replikation |
| ADR-113 | Theme-/Accessibility-Contract für Window Chrome |

## 21. Statusabgrenzung

Das Dokument definiert **beabsichtigtes Laufzeitverhalten** und API-Verträge. Der bestehende Go-Deployment-Bootstrap implementiert noch keine Desktop-Oberfläche, keinen Window Host, keinen App Lifecycle und keine Browser-Taskleisten-UI.

## Ergänzung v0.13 – OS-inspirierte Fensterdekoration

Das **Window Framework ist der einzige vertrauenswürdige Host für sichtbare App-Fenster**, unabhängig vom gewählten Theme. Die sechs visuellen Presets aus [`THEMING.md`](THEMING.md) dürfen Titelleisten, Schatten, Rundungen, Fensterschaltflächenposition und geprüfte Snap-/Dock-Vorschauen verändern; sie dürfen keine eigenen alternativen Fensterprozesse oder direkten Host-APIs einführen.

- **Northstar:** abgerundete Fenster und moderne Snap-Vorschauen.
- **Cedar:** kompakte klassische Fenster mit klaren Rändern.
- **Coast:** linke Window-Buttons, ruhige Toolbar-/Dock-Optik.
- **Prism:** variantenreiche, aber geprüfte Dekorations- und Panel-Slots.
- **Slate:** reduzierte Dekoration, zugängliche Controls.
- **RetroGlass:** glasartige Ränder mit sofortigem Opaque-Fallback für Reduced Transparency.

Jedes Theme muss Minimieren, Maximieren, Wiederherstellen, Fokus, Snap, Drag/Resize, Taskleisten-Vorschau und „App beenden“ per Taskbar-Rechtsklick unverändert unterstützen. Die Position einer Taskbar (`/.taskbar.json`) ist **Benutzerentscheidung, nicht Theme-Zwang**. Das Wallpaper wird unter allen Fenstern dargestellt und fängt keine Pointerereignisse ab; siehe [`WALLPAPER_ENGINE.md`](WALLPAPER_ENGINE.md).

**Sicherheit:** Der Window Host zeichnet Berechtigungsdialoge und andere kritische UI in eindeutig vertrauenswürdiger Gestaltung. Ein Theme darf Warnstatus, Fokusindikator oder Bestätigungstext nicht manipulieren. Geöffnete Apps verlieren beim Themewechsel keine Instanz-/Fensteridentitäten.

## Architekturabgleich 16.11.2026 – Framework und Browser-Matrix

[ADR-002](ADR_REGISTER.md): Die trusted Desktop-/Window-/App-Host-Oberflächen verwenden Vue 3/TypeScript und bevorzugt Vite; das öffentliche Window-/App-SDK ist frameworkneutral. Eine Third-Party-App darf weder zu Vue gezwungen werden noch die Host-Fensterkontrolle ersetzen.

[ADR-060](ADR_REGISTER.md): Chrome, Edge, Firefox und Safari werden in getrennten, fähigkeitsbezogenen Desktop- und Rust/WebGPU/WASM-Profilen auf Tastaturbedienung, Fokus, Screenreader-/Kontrastzugänglichkeit und Performance geprüft. Für das Rust/WASM-MVP-Profil gelten die Startvoraussetzungen aus ADR-017 ohne stille Downgrades. Konkrete Browserversionen und Budgets sind erst nach Testnachweis verbindlich.
