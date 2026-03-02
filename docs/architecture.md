# Energy Management Demo – Architecture

## Purpose

This repository contains a customer-facing Energy Management demo built on Matter/CHIP concepts. It showcases how multiple simulated energy devices can coordinate using Matter over Thread, while Home Assistant provides visualization and automation.

The demo is intentionally “grid-less”: all grid and device behaviors are simulated, but exposed through real Matter endpoints and clusters so the interactions are realistic.

---

## High-level system view

### Components

- **EVSE Node (Matter over Thread)**
  - Based on the Matter `evse-app` sample with demo extensions.
  - Includes a **Simulated EV** model to drive charging state/behavior.
  - Exposes EVSE state and accepts charge enable/disable commands.

- **Solar Panel Node (Matter over Thread)**
  - Simulates PV generation and exposes power output telemetry.

- **E-Meter / Tariff Node (Matter over Thread)**
  - Simulates electricity price (and optionally other grid attributes).
  - Exposes a *Commodity Price / tariff* data model for automation decisions.

- **Thread Border Router (OTBR)**
  - Runs on **Home Assistant OS** (HAOS), typically on Raspberry Pi 5.
  - Uses a Silicon Labs **OT-RCP** (e.g., BRD4187C) connected over USB/UART.

- **Home Assistant (control plane + UX)**
  - Commissions Matter devices, displays sensor history, and runs automations.
  - Demo includes:
    - Custom extensions for Commodity Price / device typing (see component README).
    - Reference dashboard and automations YAML files (see component README).

---

## Architecture diagram

```mermaid
flowchart LR
  subgraph ThreadNetwork["Thread Network (Matter over Thread)"]
    EVSE["EVSE Node<br>(evse-app + demo extensions)<br>Simulated EV model"]
    SOLAR["Solar Panel Node<br>(simulated generation)"]
    TARIFF["E-Meter / Tariff Node<br>(commodity price simulation)"]
  end

  subgraph HA["Home Assistant OS (Raspberry Pi 5)"]
    OTBR["OTBR (Border Router)<br>+ Matter integration"]
    UI["Home Assistant UI<br>Dashboard + History"]
    AUTO["Home Assistant Automations<br>(price + solar => EVSE enable)"]
  end

  EVSE <-- "Matter (Thread)" --> OTBR
  SOLAR <-- "Matter (Thread)" --> OTBR
  TARIFF <-- "Matter (Thread)" --> OTBR

  OTBR --> UI
  UI --> AUTO
  AUTO -->|"Matter command<br>Enable/Disable charge"| EVSE
  SOLAR -->|"Telemetry"| UI
  TARIFF -->|"Telemetry"| UI
  EVSE -->|"Telemetry"| UI

