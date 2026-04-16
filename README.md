# InkTime Smartwatch

**Student:** Cesar Petrescu  
**Group:** 334CD  
**Course:** Testarea Sistemelor de Calcul - Proiect 2026

## Project Overview

This repository contains the complete hardware design for the InkTime smartwatch project.

The watch is built around the Nordic nRF52840 and combines BLE connectivity, an e-paper display, IMU-based motion sensing, USB-C charging, a fuel gauge, a haptic motor driver, and a buck-boost power stage that generates a stable 3.3 V rail from the battery / charger system rail.


## Block Diagram

![InkTime system block diagram](Images/inktime-block-diagram.png)

## Design BOM Summary

The exact assembly-ready BOM, including fitted passive parts, DNP parts, package variants, and JLC / LCSC order codes used for manufacturing, belongs in `Manufacturing/InkTime-BOM.csv`.

| Ref | Part | Function | Procurement | Datasheet / Reference |
|---|---|---|---|---|
| U1 | nRF52840-QF | Main MCU, BLE radio, USB device | [Nordic product page](https://www.nordicsemi.com/Products/nRF52840) | [Product specification](https://docs-be.nordicsemi.com/bundle/nRF52840_PS_v1.8/raw/resource/enus/nRF52840_PS_v1.8.pdf) |
| IC1 | BQ25180YBGR | Li-ion charger with power path and ship mode | [TI product page](https://www.ti.com/product/BQ25180) | [Datasheet](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| IC9 | RT6160AWSC | Buck-boost regulator for 3.3 V rail | [LCSC page](https://www.lcsc.com/product-detail/C7065276.html) | [Datasheet](https://www.richtek.com/assets/product_file/RT6160A/DS6160A-05.pdf) |
| U2 | MAX17048G+T10 | Fuel gauge | [Analog Devices product page](https://www.analog.com/en/products/max17048.html) | [Datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/max17048-max17049.pdf) |
| IC3 | BMA421 | Low-power accelerometer with step counter and interrupts | See `Manufacturing/InkTime-BOM.csv` for final sourcing | [Datasheet reference](https://www.alldatasheet.com/datasheet-pdf/pdf/1137426/BOSCH/BMA421.html) |
| IC2 | DRV2605YZFR | Haptic driver for the vibration motor | [TI product page](https://www.ti.com/product/DRV2605) | [Datasheet](https://www.ti.com/lit/ds/symlink/drv2605.pdf) |
| ANT1 | 2450AT18B100E | 2.4 GHz chip antenna | See `Manufacturing/InkTime-BOM.csv` for final sourcing | [Johanson datasheet](https://www.johansontechnology.com/datasheets/2450AT18B100/2450AT18B100.pdf) |
| J4 | KH-TYPE-C-16P | USB-C connector | [JLCPCB part page](https://jlcpcb.com/partdetail/KH-TYPE-C-16P/C709357) | [JLCPCB page includes datasheet download](https://jlcpcb.com/partdetail/KH-TYPE-C-16P/C709357) |
| D3 | USBLC6-2SC6Y | USB ESD protection | See `Manufacturing/InkTime-BOM.csv` for final sourcing | [Datasheet](https://www.st.com/resource/en/datasheet/usblc6-2.pdf) |
| J2 | 503480-2400 | 24-pin 0.5 mm FPC connector for the display | [DigiKey part page](https://www.digikey.com/en/products/detail/molex/5034802400/7063077) | [DigiKey page includes datasheet link](https://www.digikey.com/en/products/detail/molex/5034802400/7063077) |
| J1 | TC2030-IDC | Tag-Connect SWD header | [DigiKey part page](https://www.digikey.ro/en/products/detail/tag-connect-llc/TC2030-IDC/4571121) | [Mechanical drawing / datasheet](https://www.digikey.com/en/htmldatasheets/production/2094274/0/0/1/tc2030-idc.html) |
| SW_UP, SW_ENT, SW_DN | EVPAKE31A | Side push user buttons | [Panasonic product page](https://industry.panasonic.com/global/en/products/control/switch/light-touch/number/evpake31a) | [Panasonic product page](https://industry.panasonic.com/global/en/products/control/switch/light-touch/number/evpake31a) |
| Battery | LP502030, 3.7 V, 250 mAh | Main energy source | [TME battery page](https://www.tme.eu/ro/details/accu-lp502030_cl/acumulatori/cellevia-batteries/l502030/) | [AKYGA LP502030 datasheet](https://www.tme.eu/Document/b9e12bf26ad0ba929a22ab5d58f022cd/AKY0106.pdf) |
| Display | Waveshare 1.54 inch e-paper, 200x200 | Main user display | [TME product page](https://www.tme.eu/en/details/wsh-12561/e-paper/waveshare/12561/) | [Specification PDF](https://www.tme.eu/Document/0ca57a8ffbcd57b5bca53252eb9d6ec3/WSH-12561.pdf) |
| Motor | FIT0774 | Vibration motor for haptics | [TME product page](https://www.tme.eu/ro/details/df-fit0774/motoare-dc/dfrobot/fit0774/) | [DFRobot product page](https://www.dfrobot.com/product-2297.html) |

### Passives and support parts

- All resistors are 0201, as requested by the course.
- All capacitors up to and including 100 nF are 0201.
- Capacitors above 100 nF are 0402.
- The 22 uF bulk capacitors on the 3.3 V rail are 0603.

## Hardware Architecture and Functionality

### 1. MCU and RF subsystem

The central controller is the **nRF52840**. It provides:

- ARM Cortex-M4F running at 64 MHz
- BLE radio in the 2.4 GHz ISM band
- full-speed USB device support
- SPI, I2C, GPIO, ADC, and SWD interfaces
- enough Flash and RAM for the smartwatch firmware stack

The RF feed uses the nRF52840 ANT pin, a discrete matching network, and the Johanson **2450AT18B100E** chip antenna.

The antenna section follows the project constraints:

- antenna placed toward the PCB edge
- no routing below the antenna
- no GND pour below the antenna

Clocking is split between:

- **X1 32 MHz** for the main system
- **X2 32.768 kHz** for RTC and low-power timing

### 2. Power path

The power tree is:

`USB-C VBUS -> BQ25180 charger / power-path -> VREG -> RT6160 buck-boost -> 3V3 rail`

Battery monitoring is handled in parallel by the MAX17048 fuel gauge.

#### Charger

**IC1 BQ25180YBGR** receives VBUS from USB-C and manages battery charging, ship mode, and the regulated system path. Its interrupt line is exposed to the MCU as `PMIC_INT`.

#### Buck-boost regulator

**IC9 RT6160AWSC** converts the system rail to a stable 3.3 V output used by the MCU and peripherals. The design uses L7 and the required input and output capacitors from the reference schematic.

#### Fuel gauge

**U2 MAX17048G+T10** monitors the battery and exposes an alert output to the MCU. This gives the firmware a direct low-battery interrupt path.

### 3. Display subsystem

The display is a **1.54 inch 200x200 Waveshare e-paper panel** connected through **J2**, a 24-pin 0.5 mm FPC connector.

Interface signals:

- SPI clock: `SCK`
- SPI data: `MOSI`
- Chip select: `EPD_CS`
- Control GPIOs: `EPD_DC`, `EPD_RST`, `EPD_BUSY`

### 4. Motion sensing

**IC3 BMA421** is the accelerometer used for motion sensing, step counting, and interrupt-driven wake-up.

It is connected on the shared I2C bus and exposes two interrupt pins:

- `IMU_INT1`
- `IMU_INT2`

This lets the firmware wake the system on motion-related events without polling continuously.

### 5. Haptic feedback

**IC2 DRV2605YZFR** drives the **FIT0774** vibration motor.

It is controlled over I2C and uses an enable signal from the MCU. The output nodes are also broken out to test pads, which simplifies debugging and rework.

### 6. USB-C and ESD protection

**J4** is the USB-C connector used for power input and USB data.

- `VBUS` feeds the charger input
- `D+` and `D-` go to the nRF52840 USB interface
- `R1_USB` and `R2_USB` are 5.1 k pull-down resistors on the CC pins
- `D3 USBLC6-2SC6Y` protects the USB lines against ESD events

### 7. Debug and bring-up

**J1 TC2030-IDC** provides the SWD programming and debug interface.

The following debug / bring-up signals are also available on dedicated test pads:

- `TP_3V3`
- `TP_3V3_2`
- `TP_VREG`
- `TP_VBAT`
- `TP_BAT_GND`
- `TP_GND`
- `TP_SDA`
- `TP_SCL`
- `TP_SWDIO`
- `TP_SWDCLK`
- `TP_SWO`
- `TP_RESET`
- `TP_OP`
- `TP_ON`

### 8. User interface

The watch uses three side buttons:

- `SW_UP`
- `SW_ENT`
- `SW_DN`

Each button is connected to a GPIO input and paired with a pull resistor and a 1 uF capacitor for debounce.

## nRF52840 Pin Usage

| nRF pin | Signal | Connected block | Reason |
|---|---|---|---|
| P0.00 / XL1 | XL1 | X2 32.768 kHz crystal | RTC clock input |
| P0.01 / XL2 | XL2 | X2 32.768 kHz crystal | RTC clock output |
| XC1 / XC2 | XC1 / XC2 | X1 32 MHz crystal | Main HF clock for MCU and radio |
| P0.02 | SCK | E-paper connector J2 | SPI clock for display |
| P0.03 | MOSI | E-paper connector J2 | SPI data for display |
| P0.05 | EPD_CS | E-paper connector J2 | Display chip select |
| P0.06 | SDA | IC1, IC2, IC3, U2, IC9 through links | Shared I2C data |
| P0.07 | SCL | IC1, IC2, IC3, U2, IC9 through links | Shared I2C clock |
| P0.08 | IMU_INT1 | BMA421 | Motion / wake interrupt |
| P0.10 | ALERT | MAX17048 | Battery alert interrupt |
| P0.11 | PMIC_INT | BQ25180 | Charger / power-path interrupt |
| P0.12 | HAPTIC_EN | DRV2605 | Enables haptic driver |
| P0.13 | SW_UP | User button | Navigation input |
| P0.14 | SW_ENT | User button | Confirm / enter input |
| P0.15 | EPD_DC | E-paper connector J2 | Display data / command select |
| P0.16 | EPD_RST | E-paper connector J2 | Display reset |
| P0.17 | EPD_BUSY | E-paper connector J2 | Display busy indication |
| P0.18 / RESET | RESET | SWD header J1 and TP_RESET | Hardware reset |
| P1.00 | SWO | SWD header J1 and TP_SWO | Debug trace output |
| P1.01 | EPD_GATE | Q1 gate control | Power gates the display rail |
| P1.02 | SW_DN | User button | Navigation input |
| P1.08 | IMU_INT2 | BMA421 | Secondary motion / gesture interrupt |
| D+ / D- | USB D+ / D- | D3 and J4 | USB data |
| VBUS | VBUS detect | J4 / charger path | USB power sense |
| ANT | RF output | Matching network and ANT1 | BLE antenna feed |
| SWDIO | SWDIO | J1 and TP_SWDIO | SWD data |
| SWDCLK | SWDCLK | J1 and TP_SWDCLK | SWD clock |

## Power Budget and Battery-Life Estimation

The battery specified is **3.7 V / 250 mAh LP502030**.

`Battery life [hours] = 250 mAh / average current [mA]`

Example operating points:

| Average current | Estimated life | Approximate duration |
|---:|---:|---:|
| 0.3 mA | 833 h | ~34.7 days |
| 0.5 mA | 500 h | ~20.8 days |
| 1.0 mA | 250 h | ~10.4 days |
| 2.0 mA | 125 h | ~5.2 days |

Actual average current depends mainly on:

- BLE advertising / connection interval
- IMU output data rate and interrupt usage
- e-paper refresh frequency
- haptic duty cycle
- quiescent current of the charger and buck-boost path

This design is structured for low average current because:

- the e-paper display only draws significant current during refresh
- the display rail can be fully gated through Q1
- the RTC uses a dedicated 32.768 kHz crystal
- wake-up can be interrupt-driven through the IMU and fuel-gauge alert
- the charger supports low-power and ship-mode operation

### Target PCB rules

- 4-layer PCB (Top, Inner 1, Inner 2, Bottom)
- all components placed on the **Top** layer
- routing allowed on **Top**, **Inner 2**, and **Bottom**
- **Inner 1** is a dedicated GND plane
- GND pour also present on **Top** and **Bottom**
- PCB thickness set to **1.0 mm**
- power traces targeted at **0.3 mm**
- signal traces targeted at **0.15 mm minimum**
- no right-angle tracks
- no copper and no routing below the antenna
- via stitching to the inner GND plane, especially around the RF area

### Placement strategy

The placement follows the course board-shape reference and groups components around the main ICs:

- U1 and the RF matching network near the antenna edge
- IC1, U2, and J4 grouped in the USB / charging area
- IC9, L7, and bulk caps grouped as the 3.3 V power stage
- J2 and the E-paper boost parts grouped together
- IC3 placed close to the MCU while staying away from the antenna keepout area
- IC2 placed close to the haptic output pads and motor connection path

## Design Log

- Started from the course-provided InkTime schematic and mechanical outline.
- Initially targeted a 2-layer stack, but pivoted to a 4-layer stack because routing was significantly easier and required fewer vias thanks to the additional routing layers and the dedicated inner GND plane.
- Grouped the major components by subsystem to make routing, review, and mechanical integration easier.

## References
- Reference schematic: <https://ocw.cs.pub.ro/courses/_media/tsc/inktime_schematic.pdf>
- nRF52840 product page: <https://www.nordicsemi.com/Products/nRF52840>
- BQ25180 product page: <https://www.ti.com/product/BQ25180>
- DRV2605 product page: <https://www.ti.com/product/DRV2605>
- MAX17048 product page: <https://www.analog.com/en/products/max17048.html>
- EVPAKE31A product page: <https://industry.panasonic.com/global/en/products/control/switch/light-touch/number/evpake31a>

## License

This repository uses the license defined in the root `LICENSE` file.
