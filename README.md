![picokit-33-data-logger](https://raw.githubusercontent.com/mytechnotalent/picokit-33-data-logger/main/picokit-33-data-logger.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-33 DATA LOGGER

### Buffered Readings, Batched Flush, and an Authenticated Heartbeat
#### Lesson 33 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

The thirty-third Picokit lesson. The node samples the DHT11 on a two second
cadence and buffers every valid reading into a bounded batch. The batch is
flushed to the gateway on demand from the button or automatically on a timer,
and the authenticated heartbeat reports the current batch depth.

<br>

## What it teaches

- Buffering readings into a bounded in-memory batch.
- Flushing a batch on demand and on a periodic timer.
- Distinguishing on-demand from scheduled work.
- Reporting batch depth in the authenticated heartbeat body.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| DHT11 | GP4 | buffered readings |
| Button | GP15 | on-demand flush |
| Red / Yellow / Green | GP16 / GP18 / GP17 | sample and flush status |
| Onboard LED | GP25 | heartbeat, one blink per transmit |
| RYLR998 | GP8 TX / GP9 RX | LoRa heartbeat |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

The node runs `monitor_step` in a loop. Every 2 seconds it samples the DHT11
and appends a valid reading to a batch of up to sixteen samples. The button
flushes the batch immediately, and the node also flushes every 20 seconds. Each
flush seals the batch summary and transmits it, then clears the batch. Every 5
seconds the node transmits an authenticated heartbeat whose body is
`{"n":33,"s":<seq>,"c":<count>}` sealed with the field key.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_33_data_logger.elf verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset:

```text
BOOT
=== PICOKIT-33 DATA LOGGER // BUFFERED BATCH + AUTHENTICATED HEARTBEAT ===
DHT t=230 h=610
STORE c=1
STORE c=2
STORE c=3
FLUSH c=3
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=33 rssi=...` per authenticated heartbeat. The terminal
dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[picokit-34-remote-actuator](https://github.com/mytechnotalent/picokit-34-remote-actuator)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-33-data-logger/blob/main/LICENSE)
