# Energy Management Demo (Super Repo)

This repo collects all components of the Energy Management Demo via git submodules:

- Home Assistant setup + automations + dashboards
- EVSE Matter app
- Solar Matter app
- E-meter / Tariff Matter app

```bash
git clone --recurse-submodules git@github.com:silabs-apnagy/energy-management-demo.git
```

If you already cloned without submodules:
```bash
git submodule update --init --recursive
```