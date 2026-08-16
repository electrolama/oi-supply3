<!--
SPDX-FileCopyrightText: 2026 Omer Kilic <omer@electrolama.com>
SPDX-License-Identifier: Apache-2.0
-->

# OI Supply 3

Three-channel programmable power supply reference project from [Electrolama](https://electrolama.com) for the [Open Instrument Project](https://openinstrument.dev).

## Licensing

This repository contains materials under different licensing terms:

- **Documentation**, including this file and other Markdown documentation in the repository, is licensed under the [**Apache License 2.0**](LICENSES/Apache-2.0.txt) (`Apache-2.0`).
- **Hardware source**, when present, including schematics, PCB design files, mechanical CAD, manufacturing files, and other source material required to make or modify the hardware, is licensed under the [**CERN Open Hardware Licence Version 2 — Weakly Reciprocal**](LICENSES/CERN-OHL-W-2.0.txt) (`CERN-OHL-W-2.0`).

See [`LICENSE`](LICENSE) for the repository licensing map.

The [Open Instrument Project](https://openinstrument.dev) name, logos, and other branding are not granted under these licences.

## Current status

**Draft 1 — 2026-08-16**

OI Supply 3 is currently in the architecture and prototyping stage. The current point-in-time design snapshot is:

- [`docs/oi-supply3-spec.md`](docs/oi-supply3-spec.md)

## UI simulator

The current front-panel interaction prototype is available at:

- [`simulator/oi-supply3-sim.html`](simulator/oi-supply3-sim.html)

The simulator is used to prototype and review channel selection, voltage/current editing, output enable behavior, display states, and the overall front-panel interaction model before those behaviors are frozen in firmware and hardware.

## Current headline target

```text
3 channels
0.8–15.0 V
0–5.0 A/channel
20 W/channel
60 W aggregate
```

## Power architecture

- **OI Power** — 12 V housekeeping/service input and Stack pass-through only;
- **USB-C / OI Standalone** — direct communications, firmware/recovery, and adaptive USB-PD input;
- **24 V XT30** — deterministic full-power input;
- **OI Power never feeds the programmable DUT output-energy path.**

A suitable USB-PD source can power the instrument standalone. In an OI Stack, OI Power keeps the instrument alive, visible, discoverable, and configurable even when no output-energy source is present; its pass-through path also carries service power to downstream instruments. OI Power never feeds the programmable DUT output-energy path.

## Open Instrument

OI Supply 3 is a reference hardware project within the [Open Instrument Project](https://openinstrument.dev), an open platform for interoperable, modular test and measurement equipment.

The relevant Open Instrument standards remain separate from this repository. This project currently targets the working **Draft 1 / version 0.1.0 / 2026-08-16** revision of each standard:

- [OI-01 — Open Instrument Link](https://github.com/open-instrument-project/standards/blob/main/OI-01-Link/index.md);
- [OI-02 — Open Instrument Protocol](https://github.com/open-instrument-project/standards/blob/main/OI-02-Protocol/index.md);
- [OI-03 — Open Instrument Power](https://github.com/open-instrument-project/standards/blob/main/OI-03-Power/index.md);
- [OI-04 — Open Instrument Standalone](https://github.com/open-instrument-project/standards/blob/main/OI-04-Standalone/index.md); and
- [OI-05 — Open Instrument Stack](https://github.com/open-instrument-project/standards/blob/main/OI-05-Stack/index.md).

OI Supply 3 targets all five opt-in standards. OI Link requires OI Protocol; because the project also targets OI Stack, OI Link, OI Protocol, and OI Power are a compound requirement:

```text
OI COMPATIBILITY TARGET
[x] OI-01 Link
[x] OI-02 Protocol
[x] OI-03 Power
[x] OI-04 Standalone
[x] OI-05 Stack (height TBD)
```

This is a design target, not a completed compatibility claim. Several Draft 1 electrical and mechanical details remain open in the standards, and OI Supply 3 must validate every applicable requirement before using a final compatibility badge.

## Discussing and contributing

Join the dedicated [OI Supply 3 discussion](https://github.com/open-instrument-project/community/discussions/2) in the Open Instrument community for questions, feedback, design ideas, interoperability discussion, and use cases. Once work is concrete and actionable, use this repository's Issues and pull requests.
