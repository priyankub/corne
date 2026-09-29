# Corne Chocolate Wireless (with nice!view & RGB LED)

A customized split ergonomic keyboard based on the Corne (crkbd) Chocolate v2.1 (Kailh Choc v1 low-profile switches), redesigned for wireless operation (nice!nano / nRF52840), native 5-pin nice!view display support (Sharp Memory LCD), onboard power switch, JST-PH battery connector, and RGB backlighting.

---

## Hardware Features

* **MCU**: nice!nano v2 (or Pro Micro compatible nRF52840 controller).
* **Switches**: Kailh Choc v1 low-profile switches (hotswap sockets).
* **Display**: Native 5-pin header for **nice!view** or compatible Sharp Memory LCD displays (LS011B7DH03 / AliExpress clone) without requiring external bodge wires.
* **RGB**: 27× SK6812MINI LEDs (6 underglow + 21 per-key).
  - Note: In this board, WS2812 LED Data is routed to Controller **Pin 2 (RX1 / D0 / P0.08)**, freeing Controller **Pin 1 (TX0 / D1 / P0.06)** for the display Chip Select (`CS`).
* **Power**: Onboard JST-PH 2.0mm horizontal battery connector and slide switch (PCM12) routing `BAT+` to the controller `RAW` pin.
* **Reversible PCB**: The same PCB is flipped for Left or Right halves.

---

## Display Pinout & Solder Jumpers

The 5-pin display connector uses the standard nice!view / Sharp Memory LCD pinout (viewed from the front with pins at the bottom):

| Pin | Function | Signal |
|:---:|:---|:---|
| **1** (Leftmost) | Chip Select | `CS` (P0.06 / D1) |
| **2** | Serial Data / MOSI | `SDA` (P0.17 / D2) |
| **3** | Serial Clock / SCK | `SCL` (P0.20 / D3) |
| **4** | Power | `VCC` (3.3V) |
| **5** (Rightmost) | Ground | `GND` |

### Solder Jumper Configuration

Because the PCB is reversible, you select the active half by bridging the 5 solder jumpers on that half's top side:

* **Left Hand (Top / Front side facing up)**: Solder **`JP1` – `JP5`** on `F.Cu`.
  - `JP5`: Connects physical pin 5 (leftmost) to `CS`
  - `JP4`: Connects physical pin 4 to `SDA`
  - `JP3`: Connects physical pin 3 to `SCL`
  - `JP2`: Connects physical pin 2 to `VCC`
  - `JP1`: Connects physical pin 1 (rightmost) to `GND`

* **Right Hand (Bottom / Back side facing up)**: Solder **`JP6` – `JP10`** on `B.Cu`.
  - When flipped, physical pin 1 is on the left and physical pin 5 is on the right.
  - `JP6`: Connects physical pin 1 (new leftmost) to `CS`
  - `JP7`: Connects physical pin 2 to `SDA`
  - `JP8`: Connects physical pin 3 to `SCL`
  - `JP9`: Connects physical pin 4 to `VCC`
  - `JP10`: Connects physical pin 5 (new rightmost) to `GND`

---

## Modifying Existing Rev 1 Printed PCBs (Hand-Wire Fix)

If you have already printed the initial revision of this PCB (where Pin 5 was wired to CS on the right):
To make your display work without reprinting:

On the **Front (Left hand)**, bridge diagonally or run small wire jumpers between the jumper pads:
1. **Screen Pin 1 (CS)**: Bridge `JP5 Pad 2` to `CS` (connect to `JP1 Pad 1`).
2. **Screen Pin 2 (SDA)**: Bridge `JP4 Pad 2` to `SDA` (connect to `JP5 Pad 1`).
3. **Screen Pin 3 (SCL)**: Bridge `JP3 Pad 2` to `SCL` (connect to `JP4 Pad 1`).
4. **Screen Pin 4 (VCC)**: Bridge `JP2 Pad 2` to `VCC` (connect to `JP3 Pad 1`).
5. **Screen Pin 5 (GND)**: Bridge `JP1 Pad 2` to `GND` (connect to `JP2 Pad 1`).

---

## Repository Structure

```text
.
├── hardware/                  # KiCad 9 design files
│   ├── corne-chocolate.kicad_pro
│   ├── corne-chocolate.kicad_sch
│   ├── corne-chocolate.kicad_pcb
│   ├── corne-chocolate.step
│   ├── fp-lib-table
│   ├── sym-lib-table
│   ├── kbd/                   # Footprints, symbols, and 3D packages
│   └── production/            # Gerbers, drill files, BOM, pos files
└── firmware/                  # ZMK Configuration
    ├── config/                # corne.keymap, corne.conf
    ├── boards/
    └── build.yaml             # GitHub Actions build matrix
```

---

## Firmware Configuration (ZMK)

Because WS2812 LED Data was moved to `RX1 / D0 / P0.08` to free up `D1 / P0.06` for display Chip Select (`CS`), `corne.keymap` contains the SPI pin control override:

```dtsi
&pinctrl {
    spi3_default: spi3_default {
        group1 {
            psels = <NRF_PSEL(SPIM_MOSI, 0, 8)>;
        };
    };

    spi3_sleep: spi3_sleep {
        group1 {
            psels = <NRF_PSEL(SPIM_MOSI, 0, 8)>;
            low-power-enable;
        };
    };
};
```
