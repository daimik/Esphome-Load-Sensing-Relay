# 🌀 ESPHome Power-Triggered Fan

> **Turn a fan on when something else starts drawing power.** This ESPHome config turns a Sonoff Dual R3 into a load-following relay: channel 1 meters a device, channel 2 switches a fan when that device wakes up. Built for a 3D printer fume extractor, works for anything that pulls current when it's busy. 🖨️💨

---

## 🤔 What Is This?

A ready-to-use **ESPHome YAML configuration** for the Sonoff Dual R3 (PCB v1.x, CSE7761 metering chip). The printer stays plugged into one relay so its power draw can be measured. When the heaters kick in, the fan relay turns on. When the print finishes and power drops, the fan keeps running for a while to clear the last fumes, then switches off. No printer integration, no OctoPrint, no G-code hooks. Just watts. ⚡

Also fits: soldering station extractors 🔧, CNC or laser dust collection 🪚, bathroom fans on a heater 🔥, compressor cooling fans.

---

## ✨ Features

| Feature | Details |
| --- | --- |
| ⚡ **Power-spike detection** | Fan turns on when the metered load crosses a configurable threshold |
| ⏳ **Run-on timer** | Fan keeps running after the load goes idle (default 5 min) |
| 🔒 **Minimum runtime** | Fan never short-cycles, even on tiny prints (default 10 min) |
| 🎚️ **Live tuning** | All four thresholds/timers adjustable from the web UI or Home Assistant, saved across reboots |
| 🔘 **Manual override** | On-board button or external pushbutton on S2 toggles the fan and pauses automation |
| 🔄 **Smart auto-resume** | Manual mode returns to automatic on the next new print, not mid-print |
| 🚨 **Fan fault detection** | Relay on but no fan current = alert in Home Assistant |
| 📊 **Energy monitoring** | Voltage, current, power and daily kWh for both channels |
| 🧮 **Print counter** | Counts detected prints and reports last print duration |
| 🏷️ **Status sensor** | `Idle` / `Printing` / `Purging` / `Manual mode` at a glance |
| 🌐 **Built-in web server** | Local control on port 80, no HA required |
| 📡 **OTA updates** | Serial flash once, wireless forever after |
| 🖨️ **Printable case** | STL included, socket-style enclosure for the Dual R3 |

---

## 🏗️ Hardware You'll Need

| Component | Model |
| --- | --- |
| 🔌 **Dual relay with metering** | [Sonoff Dual R3](https://sonoff.tech/product/diy-smart-switches/dualr3/) (PCB v1.x, CSE7761 chip) |
| 🔧 **USB-serial adapter** | Any 3.3 V UART (FT232RL, CP2102). Needs a solid 3.3 V supply, the ESP32 pulls more than most adapters give at boot |
| 💨 **Fan** | Any mains extraction fan, ≤ 15 A |
| 🔘 **Pushbutton** *(optional)* | Momentary switch between S2 and COM for a manual fan button |
| 🔌 **Panel sockets** *(for the case)* | 2× [ABL 1661-050](https://www.elektramat.nl/abl-machine-contactdoos-inbouw-2-polig-ip54-blauw-1661-050/) Schuko, 50×50 mm flange |

> ⚠️ **PCB v2.x and Dual R3 Lite use a BL0939 chip instead of the CSE7761.** Same pins, different protocol. See [Important Notes](#️-important-notes) for the swap.

---

## 🔌 Wiring

```
Mains L  ──►  L in
Mains N  ──►  N in

O1 (channel 1)  ──►  Printer live      ← metered, relay stays ON
O2 (channel 2)  ──►  Fan live          ← switched by the automation
N               ──►  Printer + fan neutrals

S2 / COM        ──►  optional manual fan button
```

> 💡 The printer is on a relay only so its current can be measured. It stays on by default and remembers its state across reboots.

---

## 🖨️ Printable Case

[`socker.stl`](https://github.com/daimik/Esphome-Load-Sensing-Relay/blob/main/socker.stl) is a case for the Sonoff Dual R3 with cutouts for two panel-mount Schuko sockets, so the printer and the fan just plug in. No hardwiring, no open terminals. 🔌🔌

| Part | Qty | Notes |
| --- | --- | --- |
| 🖨️ `socker.stl` | 1 | Print in PETG or ASA, not PLA. The relay module gets warm |
| 🔌 [ABL 1661-050](https://www.elektramat.nl/abl-machine-contactdoos-inbouw-2-polig-ip54-blauw-1661-050/) | 2 | Schuko panel socket, 16 A, IP54, 50×50 mm flange, 38×38 mm hole spacing |
| 🔩 M3×6 mm screws | 17 | Four per socket flange + case |
| 🔥 M3 heat-set brass inserts | 17 | Melt into the printed bosses with a soldering iron before mounting the sockets |
| 🔌 Mains inlet cable | 1 | 3×1.5 mm² with Schuko plug, or an IEC C14 inlet if you prefer |

Any 50×50 mm flange socket with 38×38 mm hole spacing fits the cutouts, so a grey or black ABL (1661-060, 1661-000) or a Sirox equivalent works too. Blue is what I had.

| Print setting | Value |
| --- | --- |
| 🧵 Material | PETG / ASA |
| 📏 Layer height | 0.2 mm |
| 🧱 Walls | 3 |
| 🔲 Infill | 20 % |
| 🏗️ Supports | None |

> ⚠️ Earth both sockets and, if the case is mounted on anything conductive, run the earth through. The Dual R3 has no earth terminal; loop it straight from the inlet to the two socket earth tabs.

---

## 📋 GPIO Pinout Reference

| GPIO | Function |
| --- | --- |
| `GPIO0` | On-board button (pull-up, inverted) |
| `GPIO13` | Status LED (inverted) |
| `GPIO14` | Relay 2, fan |
| `GPIO27` | Relay 1, printer |
| `GPIO25` | CSE7761 TX |
| `GPIO26` | CSE7761 RX |
| `GPIO32` | S1 input (spare, exposed) |
| `GPIO33` | S2 input, external fan button |

---

## 🚀 Getting Started

### 1️⃣ Prerequisites

- 🏠 [Home Assistant](https://www.home-assistant.io/) up and running
- ⚡ [ESPHome](https://esphome.io/) add-on installed
- 🔌 Sonoff Dual R3, PCB v1.x

### 2️⃣ Configure the YAML

Add these to your ESPHome `secrets.yaml`:

```yaml
wifi_ssid: "your-ssid"
wifi_password: "your-password"
printer_3d_ventilator__encryption_key: "your-api-key"
printer_3d_ventilator__ota_password: "your-ota-password"
printer_3d_ventilator__ap_password: "your-fallback-ap-password"
```

> ⚠️ **Never commit `secrets.yaml`.** Add it to `.gitignore` before your first push.

### 3️⃣ Flash the Device

With **mains disconnected**, connect the 3.3 V serial adapter to the header, hold the on-board button, apply 3.3 V, release the button.

```bash
# First time — via serial
esphome run 3d-printer-ventilator.yaml

# After that — over the air 🎉
esphome run 3d-printer-ventilator.yaml --device 3d-printer-ventilator.local
```

> ⚠️ Disconnect the serial adapter **before** reconnecting mains. The Dual R3 is not isolated; you will put mains potential on your USB port.

### 4️⃣ Add to Home Assistant

The device auto-discovers under **Settings → Devices & Services → ESPHome**. Accept it and you're done. 🎉

---

## ⚙️ How the Automation Works

```
Printer power ≥ ON threshold   →  fan ON, timers cancelled, print counted
Printer power < OFF threshold  →  run-on timer starts
Run-on elapsed                 →  fan OFF (or wait until min runtime is met)
Between thresholds             →  fan held ON (bed PWM off-cycles won't trip it)
```

Two thresholds give hysteresis. Heated beds cycle on and off during a print, so the fan only starts the shutdown timer when power drops *below* the lower value, and cancels it the moment power climbs *above* the upper value.

### 🎚️ Tuning Values (web UI → Configuration)

| Setting | Default | What it does |
| --- | --- | --- |
| ⚡ Fan ON threshold | 80 W | Power that counts as "printing". Set below your heat-up draw, above idle |
| 🔻 Fan OFF threshold | 35 W | Power that counts as "idle". Set above idle standby, below bed-hold draw |
| ⏳ Fan run-on time | 300 s | How long the fan keeps running after the printer goes idle |
| 🔒 Fan minimum runtime | 600 s | Fan never switches off sooner than this after starting |

> 💡 Watch **Printer Power** in Home Assistant through one full print before tuning. Idle standby, heat-up spike and bed-hold are the three numbers you need.

### 🔘 Manual Control

| Action | Result |
| --- | --- |
| On-board button, short press | Toggle fan, switch to **Manual mode** |
| On-board button, long press (0.4–3 s) | Back to **Automatic mode** |
| S2 external button | Toggle fan, switch to **Manual mode** |
| **Automatic Mode** switch in HA | Same thing, remotely |

Manual mode does not stay stuck. If you override during a print, the fan stays where you put it until that print finishes. The next heat-up spike re-enables automation.

---

## 📡 Exposed Entities in Home Assistant

### Sensors

- ⚡ Voltage
- 🖨️ Printer Current / Printer Power / Printer Daily Energy
- 💨 Fan Current / Fan Power / Fan Daily Energy
- 🧮 Prints Detected
- ⏱️ Last Print Duration
- 🏷️ Status (`Idle`, `Printing`, `Purging`, `Manual mode`)

### Controls

- 💨 Fan (relay 2)
- 🖨️ Printer Line (relay 1)
- 🔄 Automatic Mode
- 🎚️ Four tuning numbers (Configuration category)
- 🔁 Restart

### Binary Sensors

- 🚨 Fan Fault (problem class)
- 🔘 S1 Input (spare)

---

## ⚠️ Important Notes

- **The CSE7761 is powered from the mains side.** On the serial adapter alone it will log `Communication failed` and the sensor component marks itself FAILED. That's expected. Connect mains and reboot.
- **Negative power readings** mean L and N are swapped at the input terminals. The config takes the absolute value so the automation still works, but with L/N reversed the relays are switching neutral and the loads stay live when "off". Fix the wiring.
- **PCB v2.x / Dual R3 Lite (BL0939):** change the `uart:` block to `baud_rate: 4800`, `stop_bits: 2`, and swap `platform: cse7761` for `platform: bl0939`. Everything else is unchanged. Sensor names and ids carry over.
- **Chip revision:** the config sets `minimum_chip_revision: "3.1"` and `sram1_as_iram: true` to shrink the binary. If you clone this to an older unit, check the boot log for its revision and remove those two lines if it is below 3.1.
- **Fan fault threshold** is 1 W. Raise it toward half your fan's rated wattage if you get false alarms with a very small fan.

---

## 🤝 Contributing

Adapted this for a different metering relay (Shelly Plus 2PM, Sonoff POW, Athom)? Open a PR, I'd like to collect variants. Bug reports and tuning tips welcome too. 💜

---

## 📄 License

This project is open source. Use it, modify it, make it yours! 🛠️
