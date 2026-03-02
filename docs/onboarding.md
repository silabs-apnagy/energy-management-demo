
---

# Energy Management Demo – Onboarding

This guide walks you through setting up the demo end-to-end:
- Home Assistant OS + OTBR (Thread Border Router)
- Commissioning the Matter devices (EVSE, Solar, Tariff)
- Enabling dashboards and automations

For deeper details on how each app works internally, see the per-component READMEs linked throughout.

---

## Prerequisites

### Hardware
- Raspberry Pi 5 (recommended) with:
  - microSD (HAOS supported boot)
  - Ethernet (recommended for reliability)
- Silicon Labs Thread radio for OT-RCP 
- Development boards for each Matter node:
  - EVSE node board
  - Solar node board
  - Tariff node board
- USB cables / debug adapters as required by your boards
- All of the apps in this demo used BRD4187C boards mounted on BRD4002 WPK developer kits. 

### Software
- Home Assistant OS 2026.2.1 used for development. 
- Silicon Labs tools for building/flashing (Simplicity Studio / SLT, depending on your setup)
- A build environment for Matter/CHIP-based apps (repo-specific, see evse-app setup guide)

---

## Step 1 — Build and flash the demo devices

You will flash three Matter-over-Thread devices:
- EVSE node
- Solar node
- E-meter / price node

Each app has its own README with build + flash instructions:

- EVSE app: `components/evse-app/README.md`
- Solar app: `apps/solar-app/README.md`
- E-Meter app: `apps/e-meter-app/README.md`

---
## Step 2 — Set up Home Assistant OS on Raspberry Pi 5

[Follow this link to setup the homeassistant.](https://github.com/silabs-apnagy/emdemo-homeassistant/blob/main/README.md)


## Step 3 — Commission devices into Home Assistant (Matter)

All of the apps should start in commissioning mode. They all have the same commissioning credentials. 
It is recommended to only have one device powered per commissioning process. 

For each device:
1. Put the device into commissioning mode (factory reset if needed).
2. In Home Assistant
  a. If you have a mobile device on the same network with HomeAssistant Companion app:
    - Settings → Devices & Services → Matter
    - Add device, follow commissioning prompts
  b. Commissioning without mobile app:
    - Settings → Devices & Services → Thread
    - Select the Gear icon "Configure"
    - Select the (i) Icon "Thread Network Information"
      **NOTE** If there is no HomeAssistant OpenThread Border Router as a preferred network here. The OTBR setup in Homeassistant needs to be done.
    - Copy to clipboard: the "Active dataset TLVs:" for example: 0e0800....
    - Settings → Apps → Matter Server → Open Web UI
    - +Commission Node → Commission New Thread Device → Enter the dataset here.
    - Enter the pairing code. (onboardingcodes ble command on the matterCli, same for all devices)
    - The automatic process should take around a minute and a new node should appear in the Matter Server
4. Confirm entities appear:
   - Solar: power output sensor
   - Tariff: price sensor/attributes
   - EVSE: charging state + control/enable entities (depending on exposure)

Troubleshooting tips:
- If entities appear “unavailable” immediately after commissioning, wait briefly for the first reports.
- Confirm Thread network connectivity and OTBR health first.

---

## Step 4 — Apply Home Assistant demo extensions (Commodity Price support)

This demo includes Home Assistant / python-matter-server integration extensions for the Commodity Price cluster (and related device typing), plus reference dashboard + automation YAML.

Follow the exact copy/patch instructions in:
- `components/homeassistant/README.md`

That README should cover:
- Advanced SSH & Web Terminal add-on usage
- Copying the provided `.py` files into the appropriate HAOS filesystem locations
- Restarting / validating the integration behavior
- Importing or referencing `dashboard.yaml` and `automation.yaml`

---

## Step 5 — Import demo dashboard and automation

Use the dashboard and automation YAML as a reference baseline for the last known-good demo configuration.

1. Install/apply the dashboard configuration
2. Add/enable the automation logic that controls EVSE enable based on:
   - solar output thresholds
   - price thresholds

Reference logic example:
- EVSE enable if:
  - Solar > 3000W
  - OR Solar between 800W and 3000W AND Price < 500

(Exact entity IDs will depend on your HA setup after commissioning.)

---

## Step 6 — Demo validation checklist

### Connectivity
- [ ] OTBR running and Thread network formed
- [ ] Matter integration healthy
- [ ] All 3 devices visible in HA

### Telemetry
- [ ] Solar power updates periodically
- [ ] Tariff/price updates periodically
- [ ] EVSE state updates periodically

### Control + automation
- [ ] When conditions are true, EVSE enable signal turns on
- [ ] When conditions are false, EVSE enable signal turns off
- [ ] Dashboard reflects the state transitions clearly

---

## Operational tips

- Prefer Ethernet for the HAOS host during demos.
- Keep device logs available (UART) for rapid debugging.
- If commissioning gets weird, do a clean sequence:
  1) remove device from Matter/HA
  2) factory reset device
  3) recommission

---

## Links to component documentation

- EVSE app README: `components/evse-app/README.md`
- Solar app README: `components/solar-app/README.md`
- E-Meter app README: `components/e-meter-app/README.md`
- Home Assistant integration pack README: `components/homeassistant/README.md`
