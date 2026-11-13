# replACIA USB to W65xx Bus Adapter

*This board is still in development.*

This is a replacement for a number of "serial port" solutions for
W65Cxx-based breadboard computers requiring either the W65C51N ACIA or
custom logic and either a MAX232 and DB9 connector or breakout board with
a serial-to-USB or parallel-to-USB chip.

This solution replaces all that on a single board with a FTDI FT245R (aka.
USB Blaster) chip and some logic ICs, all on a 8x5 DIP16 board, which is
smaller than the 14x6 DIP28 ACIA. The board supports USB2.0 with vertical
USB.C connector.

## Pin List

Pin | Name | Type | Description
--- | ---- | ---- | -----------
1 | &phiv;2 | Input | PHI2: The system clock.
2 | CS# | Input | Chip Select from the address decode logic.
3 | R/W# | Input | RWB signal from the CPU.
4 | GND | Power Input | Ground.
5 | V<sub>CC</sub> | Power Input | +5V DC. (+3.3V is unsupported without adding a 12 MHz crystal or upgrading to FT245RN.)
6 | A0 | Input | Address bit 0 from the CPU.
7 | IRQ# | Open Drain Output | Interrupt to the CPU indicating received data is ready to be read. (If you prefer a push-pull interrupt output, replace the 74LVC1G38 with a 74LVC1G00.)
8 | RES# | Input | RESB reset signal from the CPU. This clears the interrupt enable register.
9&ndash;16 | D0&ndash;D7 | Input/Output | The CPU data bus.

## Register 0 &mdash; CSR

Bit | Name | Type | Description
--- | ---- | ---- | -----------
7 | TXE# | Read-Only | If cleared to 0, the transmit buffer is empty and ready to be written to. (Yes, it actually works!)
6 | RXF# | Read-Only | If cleared to 0, the receive buffer is full and has data ready to be read from.
5 | PWREN# | Read-Only | This is the status of the PWREN# signal from the FT245R. If cleared to 0, the device has been configured by USB. If set to 1, the device is in USB suspend mode. This can be used by software as indicating something similar to DSR#.
4 | VBUS | Read-Only | This is the status of USB power. If set to 1, USB is powered, indicating the remote host should be resent. If cleared to 0, the FT245RN device is in reset. This can be used by software as something similar to DCD#.
3:1 | Reserved | Unused |
0 | IRQ_EN | Read/Write | Set to 1 to enable the receive interrupt. The default reset value is 0 (disabled). There is no transmit interrupt; I assumed USB is fast enough that polling TXE# is sufficient. (The ACIA oddly enables the receive interrupt by default.)

## Register 1 &mdash; DATA

When receive data is present, reading from this register returns a byte of
data received from the remote host. This also clears the IRQ (if it was
enabled). When the transmit buffer is empty, writing to this register
transmits a byte of data to the remote host.
