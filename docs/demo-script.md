# Energy Management Demo – Demo Script

## Goal
Demonstrate an end-to-end Matter-over-Thread energy management system where:
- **Solar production** and **electricity price** are simulated but exposed via **Matter telemetry**
- **Home Assistant** runs an automation that enables EV charging only when conditions are favorable
- The **EVSE + SimEV** simulate a vehicle that charges

This demo is designed to be *interactive* and *explanatory*: while describing the architecture, we use BT0/BT1 on each device to show telemetry changes and the resulting automation behavior.

---

## What the audience sees
- A large display showing **Home Assistant dashboard**:
  - Solar power (kW)
  - E-meter price (ct/kWh)
  - EVSE Enable Charge signal (ON/OFF)
  - EV state of charge (SoC) in kWh 
- Each devkit LCD is on **DemoScreen**:
  - **EVSE LCD**: Connected (yes/no), Charging (yes/no), SoC (%)
  - **E-Meter LCD**: CurrentPrice (ct/kWh)
  - **Solar LCD**: Clear / Cloudy / Night
- Devices are joined via **Thread**, commissioned into HA via **Matter**

---

## Controls (Buttons)
### BT0 (all devices)
Cycles the LCD screens:
1) Commissioning QR code  
2) DemoScreen  (This should be selected for demo runs)
3) Network Status

### BT1 (per-device application function)
- **EVSE**: `ToggleConnect`
  - Connects/Disconnects SimEV from EVSE
  - When connecting: resets SoC to a random lower value than current SoC (but > 15%)
- **Solar Panel**: `ChangeWeather`
  - Cycles: **Clear (~3.5 kW)** → **Cloudy (~1 kW)** → **Night (0 kW)**
- **E-Meter**: `ChangePrice`
  - Cycles CommodityPrice: **133 → 399 → 765 ct/kWh**

---

## Automation behavior (Home Assistant)
Home Assistant sets **Enable Charge** for the EVSE when:

- Solar production **> 3 kW**
  OR
- Solar production is **between 0.5 kW and 3 kW** AND **Price < 500 ct/kWh**

Otherwise, **Enable Charge = OFF**.

> The demo focuses on showing how *grid signals* (price + renewable availability) influence *load control* (EV charging).

---
## Demo run (talk track + choreography)

### 1) System overview 
**Introduction**
- “This demo is a Matter-over-Thread energy system.”
- “We have three Matter nodes: Solar, E-meter (price), and EVSE with a simulated EV.”
- “Home Assistant is the orchestrator: it shows telemetry and runs the automation that decides when EV charging is allowed.”

**Show**
- Dashboard: Solar power, price, Enable Charge, and SoC.
- Device LCDs: Solar weather state, meter price, EVSE connection + charging + SoC.

---

### 2) Show the EV connecting/disconnecting (EVSE BT1)
**Action:**
- Press **EVSE BT1** (`ToggleConnect`) to **connect** the simulated EV.
- Leave on Connected

**Expected:**
- EVSE LCD: `Connected = yes`, `Charging = no` (unless Enable Charge already ON), SoC resets lower (>15%)
- HA dashboard: SoC updates / EVSE state updates

**Say:**
- “Connecting the EV is like plugging in. The charge is dropped on each disconnect to simulate going for a drive.”

**Say:**
- “This shows the EV is an independent actor, but charging permission comes from the energy system policy.”

---

### 3) Demonstrate “bad conditions” block charging (Solar BT1 + E-meter BT1)
Start from:
- Solar = Night (0 kW) or Cloudy (~1 kW)
- Price = 765 (high)

**Action:**
- Ensure Solar is **Night**: press **Solar BT1** until `Night`
- Ensure Price is **765**: press **E-meter BT1** until 765

**Expected:**
- HA: Enable Charge = **OFF**
- EVSE LCD (if connected): Charging = **no**

**Say:**
- “With no solar and high grid price, the system blocks charging.”

---

### 4) Show price-sensitive charging under moderate solar (Cloudy + low price)
**Action:**
- Set Solar to **Cloudy (~1 kW)** with **Solar BT1**
- Set Price to **399** (or **133**) with **E-meter BT1**

**Expected:**
- Solar is between **0.5 kW and 3 kW**
- Price < 500
- HA: Enable Charge flips to **ON**
- EVSE LCD: Charging becomes **yes**
- HA: SoC (kWh) begins increasing rapidly (50× time acceleration)

**Say:**
- “Now we’re in the ‘conditional’ case: moderate solar is available, but we only charge when price is acceptable.”
- “Watch Enable Charge go ON, and then SoC climbs quickly because the EV simulation runs at 50× speed.”

---

### 5) Show “high price overrides moderate solar” (Cloudy + high price)
**Action:**
- Keep Solar at **Cloudy (~1 kW)**
- Toggle Price to **765** using **E-meter BT1**

**Expected:**
- HA: Enable Charge flips **OFF**
- EVSE LCD: Charging becomes **no**
- SoC increase stops

**Say:**
- “Same solar, but price spikes—charging is immediately suspended.”

---

### 6) Show “abundant solar enables charging regardless of price” (Clear + high price)
**Action:**
- Set Solar to **Clear (~3.5 kW)** using **Solar BT1**
- Keep Price at **765** (high)

**Expected:**
- Solar > 3 kW
- HA: Enable Charge flips **ON**
- EVSE: Charging = **yes**
- SoC increases rapidly

**Say:**
- “When renewable production is abundant, we allow charging even if the grid price is high. We’re effectively consuming local generation.”

---

### 7) Wrap-up / key message
**Say:**
- “This is the core energy management story: devices publish telemetry using Matter, and a controller applies policy.”
- “We can easily expand this beyond EV charging—heat pumps, battery storage, appliance scheduling—without changing the networking model.”

**Optional final action:**
- Set Solar to Night and show charging turns off again.

---

## Notes
- Keep EV connected for the main automation showcase; disconnect/reconnect is best used early as a “plug in” metaphor.
- Delayed updates:
  - Point out that we are in a noisy environment which is puttin extra strain on the wireless communication. 
- Use the LCDs as “local truth” and HA dashboard as “system truth.”

---

## Quick reference: expected outcomes table

| Scenario | Solar | Price | Enable Charge | EVSE Charging |
|---|---:|---:|---:|---:|
| Bad conditions | Night (0 kW) | 765 | OFF | No |
| Moderate solar + low price | Cloudy (~1 kW) | 133/399 | ON | Yes |
| Moderate solar + high price | Cloudy (~1 kW) | 765 | OFF | No |
| High solar (overrides price) | Clear (~3.5 kW) | 765 | ON | Yes |
