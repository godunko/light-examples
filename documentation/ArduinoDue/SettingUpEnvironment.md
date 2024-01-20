# Getting Started with Ada and `light` runtime on Arduino Due

## Setting Up Environment

### `Alire`

`Alire` package manager sould be downloaded and installed according to
instructions on https://alire.ada.dev/. No other configuration is necessary.

### BOSSAC

`BOSSAC` as provided by Ubuntu 22.04 is part of the `bossa-cli` package and can
be installed by Ubuntu's `apt` tool. It doesn't require any configuration.

### OpenOCD

`OpenOCD` as provided by Ubuntu 22.04 is part of the `openocd` package and can
be installed by Ubuntu's `apt` tool. It requires some additional configuration
for Arduino Due board itself and for some debug probes.

#### Arduino Due Board

`OpenOCD` doesn't detect Arduino Due board by default configuration. Special
configuration file `arduino_due.cfg` is necessary to be able to detect board:

    #sam3x8e cpuid
    set CPUTAPID 0x2ba01477
    set CHIPNAME at91sam3X8E

    source [find target/at91sam3ax_8x.cfg]

Arduino Due has two headers to connect debug probes. One of them is standard
JTAG header and another one is four pins header. Only JTAG header provides
connection to SWO pin of the CPU, which can be used by the application to
report debug logs to the host computer at high speed.

#### DAPLink Debug Probe

`DAPLink` is a falimy of debug probes with a bit different capabilities. Any of
them can be used to flash and debug Arduino Due board. Some debug probes
doesn't support SWO pin.

Some `DAPLink` debug probes can't be detected by `OpenOCD` in this case you
need to use additional configuration file `daplink.cfg`:

    source [find interface/cmsis-dap.cfg]
    cmsis_dap_backend hid


#### STLink V2

While STLink V2 debug probe is intended to be used with STM32 family of chips,
it works with Arduino Due too. Full version of this debug probe has JTAG
connector, and standard 10-pin 1.27 mm cable can be used to connect to JTAG
header on the board. Mini version of the adapter doesn't support SWO pin.

`OpenOCD` provides two drivers to support STLink V2 probes,
`interface/stlink.cfg` and `interface/stlink-dap.cfg`. Both works with Arduino
Due board.
