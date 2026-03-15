# SEGGER J-Link VCOM Setup and Sanity Check

This note summarizes the steps required to enable and verify the **Virtual COM Port (VCOM)** on a **SEGGER J-Link debugger** such as the J-Link Ultra+.

## 1. Verify That the Probe Supports VCOM

Open **J-Link Commander**:

```text
JLinkExe
```

Then check the VCOM command:

```text
J-Link> VCOM ?
```

Expected output:

```text
Syntax: VCOM <State>
```

If you instead see:

```text
The connected probe does not support VCOM functionality
```

the probe firmware or license does not support VCOM.

## 2. Enable VCOM

In J-Link Commander, run either of the following commands:

```text
J-Link> VCOM Enable
```

or:

```text
J-Link> VCOM 1
```

After enabling VCOM, unplug and reconnect the J-Link USB cable.

## 3. Confirm That the COM Port Appears

Open **Device Manager > Ports (COM & LPT)**.

You should see something similar to:

```text
SEGGER J-Link VCOM (COM7)
```

## 4. VTref Is Required

The J-Link will **not drive target I/O pins unless VTref is present**.

Provide a reference voltage with at least these connections:

| Pin | Signal |
| --- | --- |
| 1 | VTref (target voltage, for example 3.3 V) |
| 4 | GND |

You can verify VTref with:

```text
JLinkExe
```

Expected output:

```text
VTref = 3.300V
```

If it shows:

```text
VTref = 0.000V
```

VCOM will not function.

## 5. J-Link 20-Pin Header Pinout in SWD Mode

When the J-Link is used in **SWD mode**, the 20-pin header is typically wired as follows:

| Signal | Odd Pins | Even Pins | Signal |
| --- | --- | --- | --- |
| VTref | 1 | 2 | Vsupply / NC |
| NC (`nTRST` only for JTAG compatibility) | 3 | 4 | GND |
| NC or **VCOM TX** | 5 | 6 | GND |
| SWDIO | 7 | 8 | GND |
| SWCLK | 9 | 10 | GND |
| NC (`RTCK` only for JTAG compatibility) | 11 | 12 | GND |
| SWO | 13 | 14 | Reserved |
| nRESET | 15 | 16 | Reserved |
| NC or **VCOM RX** | 17 | 18 | Reserved |
| 5V target supply | 19 | 20 | Reserved |

Notes:

- Pin `5` becomes **J-Link TX** when VCOM is enabled.
- Pin `17` becomes **J-Link RX** when VCOM is enabled.
- Reserved pins should usually be left open unless required by a specific SEGGER feature.

For **VCOM**, the pins that matter are still:

| Pin | Function |
| --- | --- |
| 1 | VTref |
| 4 | GND |
| 5 | VCOM TX (J-Link to target) |
| 17 | VCOM RX (target to J-Link) |

## 6. Hardware Loopback Test

This verifies the VCOM port without needing MCU firmware.

Connections:

```text
Pin 1  -> 3.3V
Pin 4  -> GND
Pin 5  <-> Pin 17
```

## 7. Test with a Serial Terminal

Open a terminal such as **PuTTY**.

Example settings:

```text
Port: COM7
Baud: 115200
Data: 8
Parity: None
Stop: 1
Flow control: None
```

Type characters in the terminal.

Expected result from the loopback:

```text
hello
hello
```

If nothing appears:

- Check VTref.
- Verify the pin wiring.
- Confirm that VCOM is enabled.

## 8. Connect to an MCU

Wire the J-Link to your target UART:

```text
J-Link TX (pin 5)  -> MCU RX
J-Link RX (pin 17) -> MCU TX
GND                -> GND
VTref              -> Target Vcc
```

The PC serial terminal should then work as the MCU UART console.

## Notes

- VCOM pins are available **only on the 20-pin JTAG connector**.
- The **10-pin Cortex connector does not expose VCOM**.
- VTref must always be present for the J-Link to drive target I/O.

