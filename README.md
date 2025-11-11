# 🏡 Home Assistant — Solis Solar Setup

This repository serves as a **backup and reference** for my Home Assistant–based solar energy setup powered by a **Solis inverter** and **Dyness Powerbox G2** battery.

---

## ⚙️ Hardware Overview

### Solar Hardware

- **Inverter:** Solis S5-EH1P 5 kW
- **Datalogger:** Solis S2-WL-ST
- **Solar Panels:** 16× Longi LR7-54HTB-465M (465 W Monocrystalline)
  - PV1 (Back): 7 panels  
  - PV2 (Front): 9 panels  
- **Battery:** Dyness Powerbox G2 10.24kWh

### Home Assistant Hardware

- Raspberry Pi 4 4GB
- 64GB Sandisk SD Card
- Sonoff ZBDongle-P (for smart plugs and energy monitoring)

---

## 🔌 Extensions & Integrations

- **[Solis Modbus Extension](https://github.com/Pho3niX90/solis_modbus)**  
  Enables **local control** of the inverter (bypassing Solis Cloud), providing:
  - Data updates every **10–30 seconds**
  - Full **local configuration** control
  - Still reports all data except live house consumption (total still reports so no idea why live isn't reporting) to Solis Cloud

- **[Sunsynk Power Flow Card](https://github.com/slipx06/sunsynk-power-flow-card)**  
  Adds a **real-time energy flow dashboard** in Home Assistant.

---

## 🌞 Power Flow Dashboard Card

The **power flow card** displays live system stats — solar generation, battery charge, grid import/export, and household load.

![Power Flow Card](images/power-flow-card.png)

Configuration file: [`power-flow-card.yaml`](/power-flow-card.yaml)

---

## Energy Cost Display

This Lovelace view pairs the tariff-aware template sensors with a simple set of stats cards so I can see, at a glance, how much the house spent or earned during each billing window. The layout stacks **Daily**, **Weekly**, and **Monthly** rows, each row showing:

- `sensor.electricity_cost_<period>_total` for the gross import spend
- `sensor.export_revenue_<period>` for feed-in payments
- `sensor.net_cost_<period>` (gross minus export) highlighted in green/red depending on whether the day was net-positive or net-negative

Each tariff has its own chip that pulls from the `sensor.cost_<period>_<tariff>` series so I can immediately tell which rate is driving the total. Everything shown here comes directly from the template + `utility_meter` block in [`configuration.yaml`](configuration.yaml), so keeping the helpers up to date automatically keeps this dashboard correct.

![Energy usage display](images/energy-usage.png)

---

## 📊 Sensors & Templates

My electricity provider uses **fixed time-of-use (TOU) rates**, so I use a `utility_meter` sensor to monitor and calculate energy costs based on the active rate.

- Consumption Source: `sensor.solis_meter_total_active_energy_from_grid`  
- Export Source: `sensor.solis_meter_total_active_energy_to_grid`  
- Configuration: [`configuration.yaml`](/configuration.yaml)
- Helpers: Rates configurable under `/config/helpers` in Home Assistant

The `utility_meter` block tracks **day/night/peak/EV tariffs** across **daily, weekly, and monthly** cycles. Matching template sensors (e.g., `Cost Daily - Day`, `Cost Weekly - Night`, `Cost Monthly - Peak`) multiply consumption by the helper-defined rate for each tariff, then roll those values up into period totals.

### Key Template Sensors

- **Cost Tracking:** Daily/weekly/monthly cost per tariff (`Cost <Period> - <Tariff>`)
- **Export Revenue:** Daily/weekly/monthly export income from `sensor.grid_export_<period>`
- **Net Cost:** Period totals that subtract export revenue from total consumption cost

---

## ⚡ Automations

To automatically switch the current rate throughout the day, Home Assistant automations update all three selectors — `select.electric_meter_daily`, `select.electric_meter_weekly`, and `select.electric_meter_monthly` — at the same time. This keeps every utility meter aligned with the active TOU window.

Configuration: [`automations.yaml`](/automations.yaml)

### Example Schedule

| Time  | Rate  |
|-------|--------|
| 02:00 | EV     |
| 06:00 | Night  |
| 08:00 | Day    |
| 17:00 | Peak   |
| 19:00 | Day    |
| 23:00 | Night  |

These automations ensure accurate cost tracking across different TOU windows.

---

## 🧮 Rate Configuration

Rates for each tariff can be adjusted directly within Home Assistant via the following helpers:

| Rate Type | Helper ID | Unit | Editable |
|------------|------------|--------|-----------|
| Peak | `input_number.rate_peak` | €/kWh | ✅ |
| Day | `input_number.rate_day` | €/kWh | ✅ |
| Night | `input_number.rate_night` | €/kWh | ✅ |
| EV | `input_number.rate_ev` | €/kWh | ✅ |
| Export | `input_number.rate_export` | €/kWh | ✅ |

---

## 📁 File Summary

| File | Purpose |
|------|----------|
| [`configuration.yaml`](configuration.yaml) | Template sensors, utility meters, and helper inputs for tariffs/costs |
| [`automations.yaml`](automations.yaml) | Keeps every tariff selector in sync with the active TOU window |
| [`power-flow-card.yaml`](power-flow-card.yaml) | Layout + entities for the Sunsynk-inspired power flow view |
| [`energy-cost-column.yaml`](energy-cost-column.yaml) | Lovelace column showing daily/weekly/monthly spend, export, and net cost widgets |

---

## 💡 Notes

- The **Solis Modbus** extension allows for granular, fast updates without reliance on cloud services.  
- The **Sunsynk Power Flow Card** provides a highly visual summary of real-time system performance.  
- Designed for **daily monitoring**, **rate-based cost tracking**, and **home energy optimization**.
