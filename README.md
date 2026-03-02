# Energy Management Demo (Super Repo)

This repo collects all components of the Energy Management Demo via git submodules:

- Home Assistant setup + automations + dashboards
- EVSE Matter app
- Solar Matter app
- E-meter / Tariff Matter app

- ## Documentation

High-level documentation for the full demo:

- 🏗 **Architecture Overview**  
  `docs/architecture.md`  

- 🚀 **Onboarding Guide**  
  `docs/onboarding.md`  
  → Step-by-step setup instructions for:
    - Home Assistant OS + OTBR
    - Flashing and commissioning Matter devices
    - Applying dashboard and automation configuration

For detailed implementation information, refer to the README inside each component submodule.

---

## Clone
```bash
git clone --recurse-submodules git@github.com:silabs-apnagy/energy-management-demo.git
```

If you already cloned without submodules:
```bash
git submodule update --init --recursive
```

## Updating Submodules

```bash
cd components/<component>
git checkout main
git pull
cd ../..
git add components/<component>
git commit -m "Bump <component>"
git push
```
