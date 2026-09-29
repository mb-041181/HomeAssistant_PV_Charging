# Wallbox PV-Ladesteuerung für Home Assistant

🇩🇪 [Deutsch](#wallbox-pv-ladesteuerung-für-home-assistant) (diese Version) | 🇬🇧 [English](#wallbox-pv-charging-control-for-home-assistant)

Ein Home-Assistant-Package, das eine Wallbox über OCPP anhand eines
auswählbaren Lademodus steuert – angelehnt an die klassischen
[evcc](https://evcc.io)-Modi, aber als reine Home-Assistant-Automation ohne
zusätzliche Software.

## Features

- 5 Lademodi: **Aus**, **Sofortladen**, **PV-Überschuss**, **Min+PV**,
  **Akku-Boost** – siehe Tabelle unten
- Schützt den Hausakku durch eine konfigurierbare Batterie-Vorrang-Schwelle
- Optional: dynamische Schwellenwerte auf Basis einer PV-Ertragsprognose
  (getestet mit [Solcast](https://www.home-assistant.io/integrations/solcast_solar/))
- Start-Hysterese gegen Kurzschluss-Ein/Aus bei schwankendem PV-Ertrag
  (evcc-Vorbild: "Enable-Delay")
- Getrennte Start-/Stopp-Schwelle plus sanfte Rampe des Mindeststroms
  verhindern ein Pendeln um die Akku-Vorrang-Schwelle
- Automatischer Moduswechsel beim Ein-/Ausstecken des Fahrzeugs

## Voraussetzungen

- Eine Wallbox, die per **OCPP lokal** angesprochen wird, mit mindestens:
  Start/Stop-Schalter, Ladestrom-Vorgabe (A), Ist-Ladeleistung,
  Connector-Status. Entwickelt/getestet mit einer Wallbox Copper Business
  an einem lokalen OCPP-Server.
- Ein Hausspeichersystem/Wechselrichter mit Sensoren für Netzbezug/
  -einspeisung, Akku-Lade-/Entladeleistung und Akku-SOC – der Hersteller
  spielt keine Rolle, solange diese vier Werte als Home-Assistant-Sensoren
  vorliegen. Entwickelt/getestet mit einem E3DC S10 über die Integration
  `e3dc_rscp_connect`, sollte aber z. B. genauso mit Fronius, SolarEdge,
  sonnen oder Victron funktionieren.
- Optional: ein PV-Ertragsprognose-Sensor (z. B. Solcast) für die
  dynamischen Schwellenwerte. Ohne diesen Sensor funktioniert alles
  weiterhin, nur mit festen statt dynamischen Schwellen.

**Wichtig:** Andere Hersteller/Integrationen können abweichende Vorzeichen-
Konventionen (Netzbezug positiv/negativ) oder andere Connector-Status-Werte
liefern. Vor dem Live-Betrieb unbedingt einmal real verifizieren.

## Installation

1. `wallbox_ladesteuerung.yaml` in dein Home-Assistant-`packages`-Verzeichnis
   kopieren (siehe [Packages-Doku](https://www.home-assistant.io/docs/configuration/packages/);
   dafür muss `homeassistant: packages: !include_dir_named packages` o. ä.
   in deiner `configuration.yaml` aktiviert sein) und YAML-Konfiguration neu
   laden (kein Neustart nötig).
2. Unter **Einstellungen → Geräte & Dienste → Helfer** die neu angelegten
   Einträge **"Wallbox Konfig: ..."** suchen und dort jeweils die Entity-ID
   deiner eigenen Installation eintragen (Start/Stop-Schalter,
   Ladestrom-Vorgabe, Netzbezug, Akku-Leistung, Hausverbrauch, PV-Erzeugung,
   Ist-Ladeleistung, Connector-Status, optional PV-Prognose) – keine
   YAML-Bearbeitung nötig. Der Sensor `binary_sensor.wallbox_konfiguration_
   unvollstaendig` bzw. `sensor.wallbox_konfiguration_fehlende_helfer` zeigt
   an, ob noch Helfer offen sind.
3. **Eine Ausnahme:** Der Akku-SOC-Sensor wird zusätzlich in Automations-
   Triggern verwendet, wo Home Assistant keine Helfer-basierte Entity-ID
   erlaubt (nur echte YAML-Werte). Dafür ganz oben in der Datei im Abschnitt
   **"ABSCHNITT 0: KONFIGURATION"** nach `&battery_soc` suchen und dort
   einmalig die echte Entity-ID eintragen.
4. Optional, aber empfohlen: `dashboard_wallbox.yaml` als eigenes Dashboard
   einbinden (**Einstellungen → Dashboards → Dashboard hinzufügen → im
   YAML-Modus bearbeiten** → Inhalt der Datei einfügen). Zeigt Lademodus,
   Live-Werte und alle Einstellwerte auf einen Blick, inklusive einer Karte
   zum Ausfüllen der Konfig-Helfer direkt im Dashboard.
5. Den Start/Stop-Schalter deiner Wallbox einmal manuell testen, um die
   **Switch-Polarität** zu verifizieren (AN muss "es soll geladen werden"
   bedeuten – siehe Hinweis im Datei-Header).

## Die 5 Lademodi

| Modus | Verhalten |
|---|---|
| **Aus** | Wallbox bleibt dauerhaft pausiert |
| **Sofortladen** | Feste volle Leistung, ignoriert PV und Akku-SOC komplett |
| **PV-Überschuss** | Nur echter PV-Überschuss (Akku-Entladung wird herausgerechnet), Akku-Vorrang-Schwelle + Hysterese |
| **Min+PV** | Garantiert einen Mindeststrom aus dem Netz + PV-Überschuss obendrauf, ignoriert Akku-SOC bewusst |
| **Akku-Boost** | Feste Leistung, erlaubt Akku-Entladung, stoppt an einer konfigurierbaren Akku-Untergrenze |

Ausführliche Erklärung aller Sensoren, Einstellwerte und Automatisierungen:
siehe die Kommentare direkt in `wallbox_ladesteuerung.yaml`.

## Bekannte Einschränkungen

- Kein Hausanschluss-Limit (max. Netzbezug insgesamt) berücksichtigt.
- Kein Zielladen/Ladeplan (dafür würde ein Fahrzeug-SOC-Sensor benötigt).
- Kein Solar-Anteil-Tracking der Ladesession.
- Kein preisbasiertes Laden (Tibber/aWattar) oder Netzdienliches Laden
  (§14a EnWG) – bewusst nicht umgesetzt.

## Lizenz

[MIT](LICENSE) – nutze, verändere und teile es, wie du möchtest.

---

# Wallbox PV Charging Control for Home Assistant

🇩🇪 [Deutsch](#wallbox-pv-ladesteuerung-für-home-assistant) | 🇬🇧 [English](#wallbox-pv-charging-control-for-home-assistant) (this version)

A Home Assistant package that controls a wallbox over OCPP based on a
selectable charging mode – inspired by the classic
[evcc](https://evcc.io) modes, but implemented as a plain Home Assistant
automation without any additional software.

## Features

- 5 charging modes: **Off**, **Boost charging**, **PV surplus**,
  **Min+PV**, **Battery boost** – see table below
- Protects your home battery via a configurable battery-priority threshold
- Optional: dynamic thresholds based on a PV production forecast (tested with
  [Solcast](https://www.home-assistant.io/integrations/solcast_solar/))
- Start hysteresis against rapid on/off cycling on fluctuating PV output
  (evcc-inspired: "Enable Delay")
- Separate start/stop thresholds plus a gentle ramp of the minimum current
  prevent oscillation around the battery-priority threshold
- Automatic mode switch when the vehicle is plugged in/unplugged

## Requirements

- A wallbox controlled via **local OCPP**, providing at least: a start/stop
  switch, a charging current setpoint (A), actual charging power, and a
  connector status. Developed/tested with a Wallbox Copper Business on a
  local OCPP server.
- A home battery/inverter system with sensors for grid import/export,
  battery charge/discharge power, and battery state of charge – the
  manufacturer doesn't matter as long as these four values are available as
  Home Assistant sensors. Developed/tested with an E3DC S10 via the
  `e3dc_rscp_connect` integration, but should work equally well with e.g.
  Fronius, SolarEdge, sonnen, or Victron.
- Optional: a PV production forecast sensor (e.g. Solcast) for the dynamic
  thresholds. Without this sensor everything still works, just with fixed
  instead of dynamic thresholds.

**Important:** Other manufacturers/integrations may use different sign
conventions (grid import positive/negative) or different connector status
values. Make sure to verify this for real before relying on it.

## Installation

1. Copy `wallbox_ladesteuerung.yaml` into your Home Assistant `packages`
   directory (see the [packages docs](https://www.home-assistant.io/docs/configuration/packages/);
   this requires `homeassistant: packages: !include_dir_named packages` or
   similar in your `configuration.yaml`) and reload the YAML configuration
   (no restart needed).
2. Under **Settings → Devices & Services → Helpers**, find the newly created
   entries named **"Wallbox Konfig: ..."** and enter the entity ID of your
   own installation in each one (start/stop switch, charging current
   setpoint, grid power, battery power, home consumption, PV production,
   actual charging power, connector status, optional PV forecast) – no YAML
   editing needed. The entities `binary_sensor.wallbox_konfiguration_
   unvollstaendig` / `sensor.wallbox_konfiguration_fehlende_helfer` show
   whether any helpers are still unconfigured.
3. **One exception:** the battery SOC sensor is also used in automation
   triggers, where Home Assistant does not allow a helper-based (dynamic)
   entity ID – only a literal YAML value. For this one sensor, search for
   `&battery_soc` near the top of the file, in the **"ABSCHNITT 0:
   KONFIGURATION"** section, and enter the real entity ID there once.
4. Optional but recommended: add `dashboard_wallbox.yaml` as its own
   dashboard (**Settings → Dashboards → Add Dashboard → Edit in YAML mode**
   → paste the file's contents). Shows the charging mode, live values, and
   all settings at a glance, including a card to fill in the config helpers
   directly from the dashboard.
5. Test your wallbox's start/stop switch manually once to verify the
   **switch polarity** (ON must mean "charging should happen" – see the note
   in the file header).

## The 5 charging modes

| Mode | Behavior |
|---|---|
| **Off** | Wallbox stays paused permanently |
| **Boost charging** | Fixed full power, ignores PV and battery SOC entirely |
| **PV surplus** | Only real PV surplus (battery discharge is subtracted out), battery-priority threshold + hysteresis |
| **Min+PV** | Guarantees a minimum current from the grid + any PV surplus on top, deliberately ignores battery SOC |
| **Battery boost** | Fixed power, allows battery discharge, stops at a configurable battery lower limit |

For a detailed explanation of all sensors, settings, and automations, see
the comments directly in `wallbox_ladesteuerung.yaml`.

## Known limitations

- No overall grid connection limit (max. total grid import) is considered.
- No target-charging/departure-time schedule (would require a vehicle SOC
  sensor).
- No solar-share tracking for a charging session.
- No price-based charging (Tibber/aWattar) or grid-friendly charging
  (German §14a EnWG signals) – deliberately not implemented.

## License

[MIT](LICENSE) – use, modify, and share it however you like.
