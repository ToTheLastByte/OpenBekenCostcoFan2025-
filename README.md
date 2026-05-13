# OpenBeken Costco Fan 2025

After a few days going down the Reddit rabbit hole, and with these two repositories to thank and I take a lot of inspiration from them so please check them out:

- https://github.com/surshis/OpenBekenCostcoFan
- https://github.com/phdindota/Omnibreeze-esphome

I was able to successfully flash the 2025 version of the Costco $35 OmniBreeze fan with OpenBeken to create a fully offline Wi-Fi controlled fan.

## Requirements / Compatible Hardware

### Compatible — DC2313R FR-1 V1.0 (2025 Revision)

**PCB marking:**

```text
DC2313R FR-1
2025.07.15 V1.0
```

Unlike the older 2023 revision:

- The Wi-Fi module is soldered directly onto the main PCB
- The fan uses the **Landbook** app instead of **uHome+**
- The module is a soldered **FCM242D** instead of the removable **UAM086 BK7238** board

### Wi-Fi Module

**Module:**

```text
FCM242D
```

**FCC ID:**

```text
XMR2023FCM242D
```

This guide was specifically developed and tested on the 2025 Costco OmniBreeze fan revision listed above.
  
## Hardware Flashing

The Wi-Fi module does **not** need to be removed from the fan. However, you **must disconnect the JST connector on the right side of the board** before flashing.

> ⚠️ **WARNING**
>
> This is **NOT** the same JST connector used on the original 2023 board revision.
>
> This connector carries **main voltage**. Treat it with caution and ensure the fan is unplugged before working on it.
>
> PLEASE - PLEASE - PLEASE


<img width="350" height="254" alt="WhatsApp Image 2026-05-12 at 9 59 45 PM" src="https://github.com/user-attachments/assets/34f15246-7990-4408-a044-5c118d09d312" />
<img width="350" height="254" alt="WhatsApp Image 2026-05-12 at 9 59 46 PM" src="https://github.com/user-attachments/assets/f6690415-61ad-443b-b326-ae1a437283ab" />

## UART Wiring

To flash the device, solder or temporarily connect the following 4 UART connections to the **FCM242D** module.

### Connection

Connect your USB-to-TTL adapter (**set to 3.3V**) as follows:

| USB TTL Adapter | FCM242D Module |
|---|---|
| TX | RX |
| RX | TX |
| GND | GND |
| 3.3V | VCC (3V3) |

> ⚠️ Do **NOT** use 5V.

No reset wire is needed. You can simply leave the 3.3V disconnected initially, then plug it in when starting the flash process.

---

# Flashing Procedure

1. Plug the USB-to-TTL adapter into your PC (**leave 3V3 disconnected**).
2. Open the [BK7231GUIFlashTool](https://github.com/openshwprojects/BK7231GUIFlashTool).
3. Select your COM port.
4. Set the chip type to **BK7238**.
5. Click **Backup & Flash New**.
6. Plug in the 3.3V wire to power the module.
7. The flasher should automatically detect the chip, back up the original firmware, and begin flashing OpenBeken.
8. Wait for the process to complete successfully.

Once finished, the module can be reconnected and configured normally through the OpenBeken web interface.

---

# Tuya DPID Mapping (The Rosetta Stone) (Shameful copy of [surshis](https://github.com/surshis/OpenBekenCostcoFan))

Using information from the original ESPHome project and serial logs, the following Data Point IDs (DPIDs) were decoded for this fan:

| DPID | Function | Type | Values |
|---|---|---|---|
| 1 | Power | Boolean | `1` (On), `0` (Off) |
| 2 | Mode | Enum | `0` (Normal), `1` (Natural), `2` (Sleep), `3` (Auto) |
| 3 | Fan Speed | Integer | `1`, `2`, `3`, `4`, `5` |
| 5 | Oscillation | Boolean | `1` (On), `0` (Off) |
| 13 | Beep Toggle | Boolean | `1` (On), `0` (Off) |
| 15 | Display LEDs | Boolean | `1` (On), `0` (Off) |
| 21 | Temperature | Integer | `°C` (Read-only) |
| 22 | Timer | Enum | `0` (Cancel), `1 - 12` (Hours) |

---

# OpenBeken Configuration (`autoexec.bat`)

To map the Tuya serial data to OpenBeken's internal channels, we use a startup script. Instead of using the short command line box, we will save this permanently to the device's filesystem.

1. From the main OpenBeken UI, click on the **Web App** menu on the left side of your screen.
2. Click on **Filesystem**.
3. Look for a button that says **Create File**.
   - If you already see a file named `autoexec.bat`, click on it to edit it.
4. Name the new file exactly:

```text
autoexec.bat
```

5. Paste the following master script directly into the text editor. Please keep in mind this is not the same as surshis due a different baud command, higher baud, and pins not needed:

```bat
startDriver TuyaMCU
tuyaMcu_setBaudRate 115200

// DPID 1: Power (On/Off)
setChannelLabel 1 "Power" 1
setChannelType 1 toggle
linkTuyaMCUOutputToChannel 1 1 1

// DPID 2: Mode (0=Normal, 1=Natural, 2=Sleep, 3=Auto)
setChannelLabel 2 "Mode"
setChannelType 2 TextField
linkTuyaMCUOutputToChannel 2 4 2

// DPID 3: Fan Speed (1 to 5)
setChannelLabel 3 "Fan Speed"
setChannelType 3 TextField
linkTuyaMCUOutputToChannel 3 2 3

// DPID 5: Oscillation (On/Off)
setChannelLabel 5 "Oscillation" 1
setChannelType 5 toggle
linkTuyaMCUOutputToChannel 5 1 5

// DPID 13: Beep (On/Off)
setChannelLabel 13 "Beep" 1
setChannelType 13 toggle
linkTuyaMCUOutputToChannel 13 1 13

// DPID 15: Display LEDs (On/Off)
setChannelLabel 15 "Display LEDs" 1
setChannelType 15 toggle
linkTuyaMCUOutputToChannel 15 1 15

// DPID 21: Temperature (Read Only)
setChannelLabel 21 "Temperature"
setChannelType 21 temperature
linkTuyaMCUOutputToChannel 21 2 21

// DPID 22: Timer (0 to 12 Hours)
setChannelLabel 22 "Timer"
setChannelType 22 TextField
linkTuyaMCUOutputToChannel 22 4 22
```

---

# Home Assistant Integration

For the Home Assistant MQTT configuration and entity setup, follow **Section 5** of the original guide from:

- https://github.com/surshis/OpenBekenCostcoFan

That section already contains the required MQTT entities, templates, and Home Assistant configuration examples needed to integrate the fan into Home Assistant.
