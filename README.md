# I2C Temperature Sensor Board

Small TMP117 temperature-sensor board for the custom Radxa CM5 camera carrier.
The intended use is remote temperature monitoring, including the camera FPGA
area. Two boards can connect using separate cables to the carrier's two
parallel I2C connectors, provided each sensor has a different address.

## KiCad project

Open `Temperature Sensor I2C.kicad_pro` in KiCad 10.

- `Temperature Sensor I2C.kicad_sch`: schematic.
- `Temperature Sensor I2C.kicad_pcb`: routed PCB.
- `includes/TMP117AIDRVR/`: custom symbol, footprint and STEP model.
- `sym-lib-table` and `fp-lib-table`: project-relative library definitions.

Local history, automatic backups, editor preferences and generated `jlcpcb/`
outputs are excluded from Git. Regenerate manufacturing outputs from the
intended design revision before ordering.

## Power and connector

Use **3.3 V power and 3.3 V I2C signals** with the matching carrier.
Do not connect this board to the camera's 12 V supply.

J1 is a four-pin JST-PH, 2.00 mm pitch, vertical header
(`B4B-PH-K` footprint):

| Pin | Signal |
| --- | --- |
| 1 | +3V3 |
| 2 | I2C SDA |
| 3 | I2C SCL |
| 4 | GND |

Verify cable continuity by pin number: **1-to-1, 2-to-2, 3-to-3, 4-to-4**.
Do not rely solely on wire colours or the apparent connector orientation.
Disconnect power before connecting or changing cables.

There are no SDA/SCL pull-up resistors on this sensor board; the matching CM5
carrier supplies them. For another host, provide suitable pull-ups to 3.3 V.
Keep cables short and start testing at 100 kHz.

## I2C address selection

Fit **exactly one** of the four zero-ohm address resistors. Leave all others
unpopulated (DNP). Fitting more than one can short supply or bus signals.

| Fit only | ADD0 connection | 7-bit address |
| --- | --- | --- |
| R3 | GND | `0x48` |
| R4 | +3V3 | `0x49` |
| R1 | SDA | `0x4A` |
| R2 | SCL | `0x4B` |

The saved schematic populates **R3 only**, selecting `0x48`.
For two sensors, assemble one with R3 only and the other with R4 only.
These addresses do not overlap the carrier's INA226 monitors (`0x40`, `0x41`).
Check the assembly BOM/DNP settings for each sensor variant before ordering.

## Assembly and mounting

- U1: TMP117AIDRVR, WSON package.
- C1: 10 uF supply capacitor; C2: 100 nF supply bypass.
- ALERT is unused; the exposed thermal pad is electrically unconnected.
- The thermal pad should remain unsoldered for best accuracy on a rigid PCB,
  as recommended by TI. Preserve the footprint's intentional lack of a
  centre-pad paste opening.
- Ensure components do not obstruct the intended thermal-contact surface.
  Use electrically insulating thermal interface material where needed; do not
  allow board copper or connector pins to short the camera electronics.
- This measures the sensor's temperature. Contact, airflow and mounting affect
  how closely that represents the FPGA/package temperature.

## Initial check

1. Verify the fitted address resistor and cable pinout with power disconnected.
2. Connect to the carrier and confirm 3.3 V between J1 pins 1 and 4.
3. Enable the appropriate I2C controller/pinmux on the CM5. Determine the actual
   Linux bus using `i2cdetect -l`; do not assume a fixed bus number.
4. Confirm communication at the selected address and read temperature using a
   TMP117-compatible driver/application. Allow the first conversion to finish.
5. With two sensors attached, verify both addresses and gently warm one board
   to establish which software channel corresponds to each physical sensor.

Hardware checks do not replace testing an assembled board and its actual cables.

## References

- [TI TMP117 datasheet](https://www.ti.com/lit/ds/symlink/tmp117.pdf): pinout,
  address selection, register interface and thermal-layout guidance.
- [Matching CM5 carrier](https://github.com/ABlairJamieson/cm5_carrier):
  see the `v1_1_power` design branch.
