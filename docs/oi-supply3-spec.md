<!--
SPDX-FileCopyrightText: 2026 Omer Kilic <omer@electrolama.com>
SPDX-License-Identifier: Apache-2.0
-->

# OI Supply 3 — Draft 1 Architecture & Product Specification

**Status:** Draft 1  
**Date:** 2026-08-16  
**Project:** [Open Instrument Project](https://openinstrument.dev)  
**Developer:** [Electrolama](https://electrolama.com)  
**Document purpose:** Point-in-time capture of the current OI Supply 3 product definition, hardware architecture, user interaction model, electrical limits, power architecture, safety philosophy, and open engineering decisions.

> This is a design snapshot, not a frozen production specification. Values and component choices marked as *candidate*, *provisional*, *target*, or *TBD* remain subject to schematic work, prototyping, thermal characterization, EMC review, safety review, and usability testing.

> **Discussion encouraged:** Join the dedicated [OI Supply 3 discussion](https://github.com/open-instrument-project/community/discussions/2) in the Open Instrument community for questions, feedback, design ideas, interoperability discussion, and use cases. Move concrete implementation work to this repository's Issues or pull requests.

## Licensing

This document and the other Markdown documentation in this repository are licensed under the [**Apache License 2.0**](../LICENSES/Apache-2.0.txt) (`Apache-2.0`).

Hardware source produced for OI Supply 3 — including schematics, PCB design files, mechanical CAD, manufacturing files, and other source material required to make or modify the hardware — is licensed under the [**CERN Open Hardware Licence Version 2 — Weakly Reciprocal**](../LICENSES/CERN-OHL-W-2.0.txt) (`CERN-OHL-W-2.0`).

The [Open Instrument Project](https://openinstrument.dev) name, logos, and other branding are not granted under these licences.

---

## Standards alignment and compatibility target

This specification targets the current working **Draft 1 / version 0.1.0 / 2026-08-16** revision of:

- [OI-01 — Open Instrument Link](https://github.com/open-instrument-project/standards/blob/main/OI-01-Link/index.md);
- [OI-02 — Open Instrument Protocol](https://github.com/open-instrument-project/standards/blob/main/OI-02-Protocol/index.md);
- [OI-03 — Open Instrument Power](https://github.com/open-instrument-project/standards/blob/main/OI-03-Power/index.md);
- [OI-04 — Open Instrument Standalone](https://github.com/open-instrument-project/standards/blob/main/OI-04-Standalone/index.md); and
- [OI-05 — Open Instrument Stack](https://github.com/open-instrument-project/standards/blob/main/OI-05-Stack/index.md).

All OI standards are opt-in. For this project, the target dependency graph is:

```text
OI-01 Link ─────── requires ───────> OI-02 Protocol

OI-05 Stack ────── requires ───────> OI-01 Link
             ├──── requires ───────> OI-02 Protocol
             └──── requires ───────> OI-03 Power

OI-04 Standalone is an additional independent target.
```

The resulting project target is:

```text
OI COMPATIBILITY TARGET
[x] OI-01 Link
[x] OI-02 Protocol
[x] OI-03 Power
[x] OI-04 Standalone
[x] OI-05 Stack (height TBD)
```

| Standard | Supply 3 implementation intent | Draft blockers before a final claim |
|---|---|---|
| OI-01 Link | Two equivalent 8P8C ports, passive CAN-FD/trigger continuation, receiver and optional trigger source | CAN rates, trigger PHY/termination, chain limits, and final misconnection tests remain open |
| OI-02 Protocol | One capability/state model over OI Link and OI Standalone | Wire layer, addressing, serialization, units, schemas, USB mapping, and security remain open |
| OI-03 Power | 12 V service input, declared draw, protected downstream-current pass-through, no DUT energy | Voltage tolerance, final chain current/power, connector, cabling, inrush, and protection coordination remain open |
| OI-04 Standalone | USB-C device control, normal update, dedicated 5 V physical recovery | USB class/framing, update format/protocol, VID/PID, signing, and recovery tooling remain open |
| OI-05 Stack | Common envelope, integer-U height, standard rear service zone, airflow and ordinary standalone access | Footprint, height, mounting, connector coordinates, keep-outs, airflow, and structure remain open |

This target is not yet a completed compatibility claim. The specification deliberately labels unfrozen standards details as TBD and shall not invent project-private replacements while claiming interoperability. A final badge requires verification against the applicable standard revisions and, for OI-05, a frozen stack height and mechanical definition.

In this product specification, **shall/must** identify product requirements, **should** identifies a strong design recommendation, and **may** identifies an optional implementation choice. Standards requirements remain authoritative where this document is incomplete.

---

## 1. Product definition

OI Supply 3 is an [Electrolama](https://electrolama.com) compact three-channel programmable bench power supply intended primarily for embedded electronics, board bring-up, automated testing, and use as part of the [Open Instrument Project](https://openinstrument.dev) ecosystem.

The instrument is intentionally not positioned as a conventional high-voltage/high-power bench PSU. Its design priorities are:

- three genuinely useful programmable rails for general embedded systems development;
- compact physical size;
- high current at low voltages;
- independent per-channel current limiting;
- continuously visible voltage/current information for every channel;
- simple direct front-panel interaction;
- USB-C standalone operation;
- deterministic full-power operation from a dedicated DC input;
- first-class integration with the [Open Instrument Project](https://openinstrument.dev);
- robust protection;
- sourceable, documented, repairable hardware.

The current headline target is:

```text
3 independently controlled programmable outputs, with a common return
0.8–15.0 V per channel
0–5.0 A per channel
20 W maximum per channel
60 W maximum aggregate output
```

The outputs share a common return in V1.

---

## 2. Why three channels

Three channels are a good match for typical embedded-system rail requirements.

Typical use cases include:

```text
1.2 V / 1.8 V / 3.3 V
1.8 V / 3.3 V / 5 V
3.3 V / 5 V / 12 V
analogue / digital / auxiliary rail
```

---

## 3. Core electrical specification

### 3.1 Channel voltage

Target programmable range:

```text
0.8 V to 15.0 V
```

`OFF` is a separate state.

The lower limit is driven by the candidate buck-boost regulator architecture.

### 3.2 Channel current

Target programmable current limit:

```text
0 A to 5.0 A maximum
```

The user may request a current limit up to 5 A, but the effective current limit is also constrained by the per-channel power envelope.

### 3.3 Per-channel power

Each channel is limited to:

```text
20 W maximum continuous output
```

The effective current capability is therefore:

```text
I_limit_effective = min(
    I_user,
    5 A,
    20 W / V_set
)
```

Examples:

| Output voltage | Maximum current | Maximum power |
|---:|---:|---:|
| 0.8 V | 5.0 A | 4.0 W |
| 1.2 V | 5.0 A | 6.0 W |
| 1.8 V | 5.0 A | 9.0 W |
| 2.5 V | 5.0 A | 12.5 W |
| 3.3 V | 5.0 A | 16.5 W |
| 4.0 V | 5.0 A | 20.0 W |
| 5.0 V | 4.0 A | 20.0 W |
| 10.0 V | 2.0 A | 20.0 W |
| 15.0 V | ~1.33 A | ~20.0 W |

The purpose of the 5 A rating is therefore to make low-voltage rails genuinely useful; the instrument is not intended to source 5 A at 15 V.

### 3.4 Aggregate output power

Maximum aggregate rated output:

```text
60 W
```

Full aggregate output means:

```text
CH1 20 W
CH2 20 W
CH3 20 W
---------
60 W simultaneously
```

The intent is for this to be a genuine continuous rating under the eventually specified ambient and cooling conditions, not an optimistic short-duration peak.

---

## 4. Power-domain philosophy

OI Supply 3 deliberately separates **instrument/service power** from **DUT output energy**.

There are three physical power inputs, but only two are permitted to feed the programmable output converters.

### 4.1 OI Power

OI Power is used for:

- MCU;
- displays;
- button LEDs;
- OI Link electronics;
- measurement electronics;
- calibration/configuration storage;
- housekeeping rails;
- optional fan/control electronics.

OI Power shall **not** feed the programmable output energy bus in OI Supply 3.

Current design assumption:

```text
OI Power: 12 V nominal service power
```

The current OI-03 Draft 1 platform target is approximately **5 A / 60 W maximum for the complete chain**, but the exact voltage tolerance, connector definition, chain-current limit, hot-plug behavior, and protection coordination remain unfrozen. Supply 3 shall follow the standard as those details are resolved rather than treating its project choices as normative.

Because Supply 3 targets OI Stack, it shall provide:

- an OI Power input and pass-through/output in the OI-05 rear service zone once the locations are frozen;
- a pass-through path designed for the total permitted downstream chain current, not merely Supply 3's own consumption;
- protection covering over-current, reverse polarity, over-voltage, transients, inrush, local shorts, pass-through faults, thermal overload, and backfeed from USB-C or the dedicated input;
- no back-drive from any internal or instrument-specific source into OI Power; and
- pass-through behavior that does not unnecessarily remove service power from downstream instruments when Supply 3 is locally off or faulted.

The current OI-03 connector direction is an approximately 5.5 mm outer / 2.5 mm centre-pin, centre-positive coaxial barrel family, but Supply 3 shall not freeze a connector footprint until OI-03 does.

Supply 3's exact maximum OI Power draw is **TBD** and shall be established by measurement with the displays, communications, measurement electronics, and worst-case service loads active. The declared maximum draw, current power state, and available service/functional-power distinction shall be exposed through OI Protocol.

This allows a stacked OI Supply 3 to remain:

- powered;
- visible;
- discoverable;
- configurable;
- available for control and diagnostics over OI Link/OI Protocol;

even when no USB-C cable or dedicated 24 V supply is connected.

In this state all programmable outputs remain unavailable/off.

Firmware update over OI Link is not a Draft 1 requirement. The guaranteed update and recovery path remains OI Standalone USB-C.

### 4.2 USB-C / OI Standalone

USB-C serves simultaneously as:

- OI Standalone direct host communications;
- firmware update/recovery;
- USB Power Delivery sink input;
- standalone housekeeping power;
- standalone programmable-output energy source when the negotiated contract permits.

### 4.3 Dedicated 24 V input

A dedicated 24 V input provides deterministic full-power operation.

Current connector choice:

```text
XT30
```

Rationale:

- clearly distinct from OI Power;
- keyed and polarized;
- suitable current margin;
- robust;
- widely available;
- avoids confusing 12 V and 24 V barrel connectors;
- explicitly communicates that this is an instrument-specific higher-energy input.

Current target:

```text
24 V nominal
4 A minimum for full capability
5 A recommended design point
```

Input tolerance and protection limits remain TBD.

---

## 5. Output-energy source architecture

Only these two sources may feed the programmable converter bus:

```text
USB-C PD VBUS
24 V dedicated XT30 input
```

OI Power is excluded from this path.

Conceptual architecture:

```text
 USB-C
   │
   ├── D+ / D− ─────────────────────► MCU USB
   ├── CC1 / CC2 ───────────────────► MCU UCPD
   │
   └── VBUS ─► PD protection/eFuse/ideal diode ──┐
                                                  │
 24 V XT30 ─► protection/eFuse/ideal diode ───────┼──► OUTPUT VIN BUS
                                                  │
 OI POWER ────────────────────────────────────────X
                                                  │
                                   ┌──────────────┼──────────────┐
                                   ▼              ▼              ▼
                                  CH1            CH2            CH3
```

### 5.1 Source selection

The two high-energy inputs shall use protected, reverse-blocking power paths.

Requirements:

- no backfeed into USB VBUS;
- no backfeed into the dedicated 24 V input;
- no backfeed into OI Power;
- source voltage/current sensing available to firmware;
- input fault containment;
- over-current protection;
- under-voltage handling;
- transient protection.

Because the dedicated input is nominally 24 V and USB-PD SPR is at most 20 V, a properly designed ideal-diode/power-mux arrangement may naturally prefer the dedicated source when both are present.

The final power-path IC/topology is TBD.

### 5.2 Source-loss behavior

OI Supply 3 does **not** need seamless hot failover of the programmable outputs.

If the active output-energy source disappears while outputs are enabled:

1. disable all affected programmable outputs safely;
2. retain instrument operation from OI Power or another valid housekeeping source where possible;
3. reassess available input capability;
4. report the new state;
5. require explicit re-enable where appropriate.

The instrument shall not silently transfer a running DUT from a high-power source to a weaker source.

---

## 6. USB-C Power Delivery

### 6.1 V1 PD scope

V1 should intentionally keep USB-PD policy simple.

Target:

```text
USB PD SPR
fixed PDOs only
no PPS requirement
no EPR requirement
```

The preferred full-power contract is:

```text
20 V / 4 A
80 W input contract
```

The hardware USB power path should therefore be designed for at least:

```text
20 V
4 A continuous
```

with suitable margin.

### 6.2 Cable requirement

USB operation above 3 A requires a:

> **5 A-rated electronically marked USB-C cable**

Short product wording may use:

> **5 A e-marked USB-C cable**

Cable identity/current capability may also be exposed diagnostically by firmware if practical.

### 6.3 Full-power statement

Recommended public wording:

> **OI Supply 3 adapts to the available USB Power Delivery contract. A USB-PD source offering at least 20 V / 4 A, used with a 5 A-rated electronically marked USB-C cable, enables the full 60 W output capability.**

### 6.4 Why an 80 W input contract for a 60 W supply

OI Supply 3 is fundamentally specified as:

```text
3 × 20 W channels = 60 W output
```

The 20 V / 4 A USB-PD target is selected so that the instrument can **guarantee** 60 W at the output rather than merely approach 60 W under ideal conditions.

Input power must also cover:

- buck-boost conversion losses;
- housekeeping electronics when USB is powering the instrument;
- measurement/sensing losses;
- shunt and switching losses;
- input power-path losses;
- thermal variation;
- component tolerances;
- transient headroom;
- design margin.

The unused portion of an 80 W input contract is intentional engineering headroom. It is not an additional 20 W of output entitlement.

The product should not chase an 80 W output rating merely because a USB source can provide 80 W; doing so would require larger thermal, magnetic, PCB, switching, and enclosure design for limited benefit.

### 6.5 Derated USB operation

Lower-power USB sources remain useful.

Example conceptual states:

```text
5 V USB source
→ instrument operational
→ low aggregate output budget

20 V / 3 A PD
→ full voltage range
→ derated aggregate power

20 V / 4 A PD
→ full voltage range
→ full 60 W aggregate capability

20 V / 5 A source
→ OI Supply 3 still requests only what its hardware is designed to use
```

Exact aggregate budgets for each contract shall be derived from measured efficiency, thermal data, and housekeeping consumption.

---

## 7. Housekeeping power architecture

The instrument electronics must remain stable while output-energy sources appear/disappear.

Conceptually:

```text
                         HOUSEKEEPING

 OI Power IN ───────────┬────────────────────────────► OI Power OUT
                        │
                        └──► protected local branch ──┐
                                                     │
 USB VBUS 5–20 V ─────────► protected power path ────┼──► housekeeping mux
                                                     │             │
 24 V AUX ────────────────► protected power path ────┘             ▼
                                                                  5 V
                                                                  3.3 V
                                                                    │
                                                  ┌─────────────────┼────────────────┐
                                                  ▼                 ▼                ▼
                                                 MCU             displays         OI Link
```

Preferred source behavior when stacked:

```text
OI Power is preferred for housekeeping.
```

This keeps OI service power conceptually separate from the DUT-energy path and allows USB/AUX source changes without rebooting the instrument.

Exact regulator and power-mux implementation remains TBD.

---

## 8. MCU architecture

### 8.1 Default reference MCU candidate

Current preferred MCU ([manufacturer product page](https://www.st.com/en/microcontrollers-microprocessors/stm32g0b1re.html)):

```text
STM32G0B1RE
```

Reasons:

- USB 2.0 Full Speed;
- integrated USB-C / USB-PD UCPD peripheral;
- two FDCAN peripherals;
- sufficient flash/RAM for control + UI;
- multiple I²C peripherals;
- multiple SPI peripherals;
- timers/GPIO suitable for OI trigger and UI;
- STM32 ecosystem support;
- ability to support OI Standalone recovery/update without a separate application processor.

The OI standards shall not mandate this MCU; it is a reference-project choice.

### 8.2 Proposed peripheral use

Conceptually:

```text
USB FS
  → OI Standalone data

UCPD
  → USB-C PD negotiation

FDCAN
  → OI Link CAN-FD

GPIO / timer
  → OI Trigger

I²C1
  → regulator channels 1/2 at the two selectable TPS55289 addresses

I²C2
  → regulator channel 3 + non-conflicting devices as required

I²C3
  → measurement / calibration / thermal / housekeeping

SPI
  → three TFT displays

GPIO / timers
  → buttons, encoder, LEDs, fan, fault inputs
```

Final bus allocation remains subject to schematic-level pin planning.

The TPS55289 exposes only two selectable I²C target addresses (`0x74` and `0x75`). The 2+1 regulator split above is therefore intentional; three regulator channels cannot share one bus without an I²C switch or equivalent isolation. Schematic review shall also verify the selected STM32 package's pin multiplexing across USB, UCPD, FDCAN, three I²C controllers, SPI, trigger timing, and front-panel I/O.

---

## 9. USB-C / OI Standalone

OI Supply 3 targets OI-04 Standalone compatibility.

The USB-C receptacle shall operate as a USB device/peripheral; USB 2.0 Full Speed is sufficient for the baseline. It must support:

- USB device communications;
- machine-readable device identity;
- capability and current-availability discovery;
- requested and actual state access;
- status, diagnostics, faults, and measurements;
- configuration and calibration-information access where appropriate;
- normal software-initiated firmware update;
- firmware recovery;
- USB-PD sink negotiation;
- direct standalone control.

Because Supply 3 also targets OI-02, the USB transport shall expose the same underlying OI Protocol identity, resources, capabilities, state, measurements, faults, and trigger configuration as OI Link. An optional human-friendly USB CDC or SCPI-like interface may be added, but it shall map onto the same underlying state rather than create a conflicting control model.

The canonical USB class, framing, serialization, image format, update protocol, signing model, and VID/PID strategy remain open in OI-04 Draft 1. Supply 3 shall not present a project-specific choice as the OI baseline.

### 9.1 Recovery

OI Supply 3 shall include the standard OI recovery behavior:

1. power the instrument off and remove OI Power and the dedicated 24 V input;
2. disconnect USB-C if already attached;
3. hold the dedicated `RECOVERY` control;
4. connect OI Standalone USB-C to a powered host;
5. enter recovery/update mode directly, without relying on application firmware; and
6. release `RECOVERY` and flash/reinstall firmware.

Ordinary 5 V USB VBUS shall power all circuitry required for recovery. Recovery shall not depend on:

- a negotiated USB-PD contract;
- OI Power;
- the dedicated 24 V input;
- OI Link or OI Base;
- opening the enclosure;
- an SWD/JTAG/debug probe; or
- working application firmware.

`RECOVERY` shall be a dedicated, clearly labelled physical control associated with the USB-C connector. It should be recessed or otherwise protected against accidental operation and shall not be reused as a normal front-panel control.

The recovery mechanism may use the STM32 ROM bootloader or a protected first-stage bootloader, but the externally observable behavior above is authoritative.

The USB-C attach terminations, 5 V housekeeping path, recovery-state selection, USB D+/D− routing, required clocks, and bootloader storage shall all function before and independently of application firmware. In particular, recovery-mode USB attachment shall not rely on application-controlled UCPD/USB-PD policy.

### 9.2 PD protection

The MCU UCPD peripheral handles USB-C/PD policy and messaging, but it does not eliminate the need for external:

- CC protection;
- VBUS protection;
- VBUS switching;
- reverse blocking;
- eFuse/current protection;
- ESD protection.

Exact protection ICs are TBD.

---

## 10. OI Link and OI Protocol integration

Because OI Supply 3 targets OI Stack compatibility, it shall implement OI Link and OI Protocol in accordance with the current standards.

### 10.1 OI Link physical interface

Supply 3 shall provide two electrically equivalent 8P8C OI Link ports. They are two access points to the same bus segment, not logical input and output ports.

The Draft 1 pair allocation is:

| Twisted pair | 8P8C pins | Function |
|---|---:|---|
| Pair 1 | 1 / 2 | CAN-FD |
| Pair 2 | 3 / 6 | Differential hardware trigger |
| Pair 3 | 4 / 5 | **RESERVED** |
| Pair 4 | 7 / 8 | **RESERVED** |

Supply 3 shall:

- use ordinary straight-through CAT5e or better cable;
- pass the CAN-FD and trigger conductors directly between the two ports so an unpowered Supply 3 does not interrupt either shared bus;
- tap those continuous buses with its local transceivers rather than forwarding traffic or trigger events in firmware;
- keep local CAN-FD and trigger stubs short and validate them against the eventual chain-length and bit-rate limits;
- keep both reserved pairs unconnected to project-private functions;
- never place OI Power, USB power, dedicated-input power, or DUT power on OI Link;
- provide protection appropriate to an exposed laboratory connector; and
- pursue the OI-01 objective that accidental connection to common Ethernet equipment does not normally damage either device.

OI Link is not Ethernet. The CAN-FD bit rates, final trigger PHY, signalling levels, common-mode range, termination values, chain limits, and exact PoE/misconnection tests remain open in OI-01 Draft 1 and shall not be frozen privately in this product specification.

CAN and trigger termination state shall be inspectable and configurable predictably. Manual termination is acceptable for early prototypes; automatic or software-controlled termination remains an implementation option until OI-01 freezes the mechanism.

### 10.2 Hardware-trigger behavior

Supply 3 shall include an OI Link trigger receiver. Whether it also exposes trigger-source capability remains a product decision.

If trigger-source capability is implemented:

- it shall be discoverable through OI Protocol;
- its driver shall remain disabled unless Supply 3 owns the single active trigger-source role; and
- source selection, listener configuration, arming, readiness, and release shall occur through OI Protocol before a trigger edge is driven.

The exact Supply 3 trigger actions—such as synchronized output enable, disable, or preconfigured setpoint transitions—remain TBD. Trigger execution shall use hardware timing after configuration/arming rather than depend on CAN arbitration or firmware forwarding at the event instant.

### 10.3 OI Protocol product model

Supply 3 shall expose a machine-readable device identity including at least:

- device class;
- manufacturer/project;
- model;
- hardware revision;
- unique device identifier or serial number;
- firmware version;
- supported OI Protocol version(s); and
- implemented OI standards and capabilities.

The intended human-facing identity is:

```yaml
class: power_supply
manufacturer: Electrolama
manufacturer_url: https://electrolama.com
model: OI Supply 3
```

Exact machine identifiers and field names remain subject to OI-02, but the implementation shall not replace this product identity with the Open Instrument Project name as though OI were the manufacturer.

The three programmable outputs shall be independently enumerable resources with stable identifiers. For each channel, OI Protocol shall distinguish:

1. **hardware capability** — the channel's design envelope;
2. **currently available capability** — the envelope available under the present input contract, aggregate allocation, and thermal state;
3. **requested state** — requested setpoints and enable state; and
4. **actual state** — measured output, actual enable state, CV/CC/OFF/fault state, and any reason the request was rejected, clamped, delayed, or not achieved.

Illustratively:

```yaml
channel: 1
hardware:
  voltage_max: 15 V
  current_max: 5 A
  power_max: 20 W
available:
  power_max: 12 W
  reason: input_power_limit
requested:
  voltage: 5 V
  current_limit: 3 A
  enable: true
actual:
  voltage: 0 V
  current: 0 A
  enabled: false
  reason: aggregate_power_limit
```

The device model shall also expose that the three channel resources belong to one physical Supply 3, share a common return, and compete for one aggregate output-power budget. Orchestration software must not have to infer these constraints from the model name.

Supply 3 shall expose through OI Protocol:

- three programmable channel capabilities;
- setpoints;
- measured voltage/current/power, with quantity, numeric value, unit, resource, and validity/status, plus timestamp, range, resolution, uncertainty, or calibration state where supported;
- requested and actual enable/CV/CC/OFF/fault state;
- active input source and USB-PD contract where present;
- hardware and currently available per-channel/aggregate output budgets;
- service-power and functional-power presence/state;
- structured warnings and faults with stable codes, severity/category, affected resource, latch/clear requirements, and unavailable-function reasons;
- trigger capabilities, configuration, armed/ready state, and counters/timestamps where implemented;
- current OI Link termination configuration and trigger-source role where inspectable;
- temperatures and thermal derating;
- calibration state and diagnostics;
- firmware/hardware revision plus boot, application, normal-update, and recovery state where meaningful; and
- explicit protocol-version/compatibility information.

The instrument shall remain discoverable and configurable over OI Link whenever OI Power is available, even when no output-energy source is present. It shall report the high-energy functional path as unavailable instead of hiding resources or pretending full capability exists.

The same conceptual resources and underlying state shall be exposed over OI Link and OI Standalone. The exact OI Protocol wire layer, addressing, serialization, units encoding, fault schema, USB mapping, and security model remain open in OI-02 Draft 1.

### 10.4 Grounding and isolation

The three V1 output returns are common with one another. Their relationship to the service/control domain is not yet frozen.

The candidate TPS55289 channel topology is non-isolated, so each channel return is inherently tied to the selected output-energy-bus return unless additional isolation is introduced. Before schematic freeze the project shall deliberately choose and document either:

- a common-ground architecture, including every resulting host, stack, shield, and DUT current path; or
- an isolated service/control boundary, including isolated regulator control, measurement, trigger, and communications paths as required.

The document shall not imply floating or isolated outputs unless the hardware actually provides them.

Before schematic freeze, Supply 3 shall explicitly document the relationship between:

- OI Power return;
- OI Link transceiver/control reference;
- OI Standalone USB ground;
- dedicated 24 V input return;
- chassis/shield; and
- the common DUT/output return.

Connecting USB-C, OI Link, OI Power, or the dedicated input shall not create an undocumented DUT-current path. If the intended measurement/source behavior requires separation from the shared service domain, the design shall provide appropriate isolation. Resolving and validating this grounding architecture is a prerequisite for a final OI compatibility claim.

---

## 11. Programmable regulator architecture

### 11.1 Current preferred regulator

Current candidate per channel ([manufacturer product page and data sheet](https://www.ti.com/product/TPS55289)):

```text
TI TPS55289
```

Architecture:

```text
OUTPUT VIN BUS
     │
     ▼
TPS55289
4-switch synchronous buck-boost
     │
     ▼
current-sense shunt
     │
     ▼
output protection / disconnect as required
     │
     ▼
2 mm output terminals
```

### 11.2 Current-limit shunt

Current candidate:

```text
~10 mΩ Kelvin shunt
```

Target behavior:

```text
~50 mA programmable current-limit resolution
headroom above the 5 A product limit
```

Exact shunt value shall be confirmed from the regulator transfer function, tolerance requirements, thermal behavior, and desired current-limit resolution.

### 11.3 Hardware versus firmware limits

The hardware regulator current loop is authoritative for per-channel current limiting.

Firmware is responsible for:

- validating requested setpoints;
- applying the 5 A product limit;
- applying the 20 W/channel envelope;
- applying aggregate input-power limits;
- refusing impossible enable requests;
- reporting clamping/derating.

Firmware must not be the only over-current protection mechanism.

---

## 12. Measurement architecture

Each channel requires authoritative measurement of:

- output voltage;
- output current;
- output power.

The regulator's internal monitor may be useful diagnostically but should not automatically be assumed sufficient for calibrated front-panel readback.

A separate multi-channel current/voltage monitor remains under evaluation.

Earlier candidate:

```text
INA4235
```

Concern:

- attractive feature set;
- but very small DSBGA package is undesirable for an open/reference instrument intended to remain approachable for assembly/rework.

Alternative multi-channel monitors such as PAC1954-class devices should be evaluated.

The measurement IC is **not frozen in Draft 1**.

---

## 13. Output connectors

Each channel uses its own pair of front-panel connectors:

```text
RED   positive output
BLACK common return
```

Current chosen connector system:

```text
standard 2 mm banana / test sockets
```

The exact PCB-mount jack remains a reference-design component choice rather than part of the OI standard.

All three channel returns are common in V1.

---

## 14. Front-panel product concept

### UI simulator

The current interactive front-panel prototype lives in:

[`simulator/oi-supply3-sim.html`](../simulator/oi-supply3-sim.html)

The simulator is the current reference for evaluating the user interaction model described in this document, including:

- per-channel voltage/current visibility;
- channel selection;
- encoder editing;
- output enable/disable gestures;
- selection highlighting;
- CV / CC / OFF / ERR presentation.

The simulator is a design/prototyping aid, not a normative firmware implementation. Where it differs from this specification, the specification records the current intended architecture; both should be kept synchronized as the UI evolves.

Each channel should have:

- one dedicated TFT;
- one dedicated output button;
- one red 2 mm output terminal;
- one black 2 mm return terminal;
- physical channel identification.

There is one shared push encoder.

### 14.1 Display

Current target:

```text
0.96-inch
160 × 80
ST7735S-class SPI TFT
landscape
```

Three displays share SPI clock and MOSI, with independent CS.

### 14.2 Display content

All three channels must have voltage and current visible at all times.

Each TFT should show:

```text
Voltage
Current
Status: CV / CC / OFF / ERR
```

A future `PWR` state may be considered for a channel that cannot be enabled because of the available aggregate input-power budget.

### 14.3 Display-value behavior

**Output OFF**
- show voltage setpoint;
- show current-limit setpoint.

**Output ON, channel not selected**
- show measured voltage;
- show measured current.

**Output ON, channel selected**
- show setpoint for the field being edited;
- show measured value for the other field.

The edited field is highlighted.

### 14.4 Status priority

Provisional:

```text
active/latched fault -> ERR
not actually enabled -> OFF
current loop active  -> CC
otherwise            -> CV
```

Provisional TFT status treatment:

```text
OFF -> white / very pale grey, steady
CV  -> green, steady
CC  -> amber, steady
ERR -> yellow, blinking
```

The explicit status label remains present so state is not communicated by colour or blinking alone.

---

## 15. Buttons and encoder

### 15.1 Output button behavior

**Short press**
- select channel for changing output settings.

**Long press**
- toggle that channel's output enable.

Current provisional hold time:

```text
2 seconds
```

### 15.2 Selection session

Current provisional timeout:

```text
3 seconds after initial channel selection or the most recent editing interaction
```

Test 3, 4, and 5 seconds before freezing.

### 15.3 Encoder

Current target:

```text
continuous quadrature encoder
30 detents/revolution
push switch
~32 mm machined aluminium knob
```

Current voltage increment:

```text
0.1 V/detent
```

The current simulator uses:

```text
0.05 A/detent
```

The production editing step should be revisited to match the real hardware current-limit granularity.

Likely candidates:

```text
0.05 A/detent
or
0.10 A/detent with fine adjustment
```

### 15.4 Selection and button indication

Draft 1 proposes:

```text
button LED state = electrical output state
LCD highlight    = editing/selection state
```

Provisional button LED mapping:

```text
LED off          output disabled
green            output enabled / CV
amber            output enabled / CC
yellow blinking  fault / ERR
```

Selection is shown by the TFT blue rail / edited-value highlight.

The exact switch/button part remains TBD.

---

## 16. Three-channel front-panel layout

Required elements:

```text
3 × channel TFT
3 × output button
3 × red 2 mm jack
3 × black 2 mm jack
1 × push encoder
physical OUTPUT 1 / OUTPUT 2 / OUTPUT 3 labels
OI Supply 3 product marking
```

Final encoder location and panel symmetry remain open.

---

## 17. Power-budget management

OI Supply 3 must simultaneously enforce:

1. per-channel current limit;
2. per-channel 20 W envelope;
3. 5 A product limit;
4. aggregate 60 W product limit;
5. actual input-source power limit;
6. thermal limits.

### 17.1 Dynamic available power

The instrument derives an available output budget from:

- active source;
- negotiated USB-PD contract;
- measured input voltage/current;
- conversion efficiency model;
- housekeeping draw;
- thermal state;
- safety margin.

### 17.2 Enabling channels

Running channels should not be silently derated merely because another channel is enabled.

If a new enable request would exceed the available aggregate budget:

```text
existing channels remain unchanged
new channel enable is refused
reason is reported
```

### 17.3 Input power changes

If available power decreases below the current load, the protection layer must move to a deterministic safe state.

Exact channel-priority/shutdown policy remains TBD.

---

## 18. Thermal architecture

### 18.1 Design premise

Each channel is a 20 W converter with up to 5 A output at low voltage.

The thermal design must treat:

> **the PCB as part of the converter heatsink.**

Draft 1 does not assume stick-on heatsinks on the converter IC.

### 18.2 PCB requirements

Starting point:

```text
4-layer PCB minimum
```

Each stage should include:

- exposed-pad soldering as required;
- thermal vias;
- generous copper;
- short high-current loops;
- substantial output-current copper;
- careful inductor placement;
- physical separation from adjacent hot spots.

### 18.3 Inductors

Selection must consider:

- saturation current;
- RMS current;
- DCR;
- core loss;
- physical temperature rise;
- sourceability;
- acoustic behavior.

### 18.4 Temperature measurement

Temperature monitoring near the power stages is required.

Exact sensor scheme TBD.

### 18.5 Fan provision

Rev A should provide mechanical/electrical provision for a quiet:

```text
30–40 mm fan
```

The fan may ultimately be omitted if testing proves it unnecessary.

### 18.6 Thermal qualification target

Test:

```text
CH1 = 20 W
CH2 = 20 W
CH3 = 20 W
60 W aggregate
continuous
closed representative enclosure
specified worst-case ambient
```

Test representative buck and boost operating points.

---

## 19. Safety and protection hierarchy

Conceptually:

```text
user request
    ↓
firmware validation / clamp
    ↓
hardware programmable CC loop
    ↓
converter peak/inductor/short protection
    ↓
per-channel output protection
    ↓
source-side eFuse / input protection
```

Required behaviors:

- all outputs OFF after reset by default;
- outputs safe during MCU boot;
- UI/display failure cannot disable protection;
- regulator communication failure leads to safe behavior;
- watchdog critical failure disables outputs;
- hardware-backed current limiting;
- 20 W/channel limit;
- 60 W aggregate limit;
- input-source limit;
- thermal protection;
- no source backfeed;
- no OI Power to output-energy path.

Faults to evaluate include:

- OVP;
- over-current/protection trip;
- short circuit;
- over-temperature;
- input undervoltage;
- USB-PD contract loss;
- regulator communication loss;
- measurement failure;
- calibration invalid;
- fan failure if fitted;
- aggregate power-budget violation.

---

## 20. Output OFF and discharge

The design must guarantee a meaningful `OFF` state.

Evaluate:

- residual output voltage;
- reverse-current behavior;
- discharge time;
- startup glitches;
- failure modes.

A dedicated per-channel output disconnect FET/load switch may be added if required.

Output discharge behavior must be deliberate and documented.

---

## 21. Calibration

Potential calibrated quantities:

- output-voltage programming;
- current-limit programming;
- voltage readback;
- current readback.

Calibration coefficients should be stored in nonvolatile memory.

Production calibration flow remains TBD.

---

## 22. Firmware architecture

Suggested modules:

```text
usb_pd_service
power_source_manager
channel_service
regulator_driver
measurement_service
protection_service
input_scan
ui_state
display_renderer
display_bus
led_driver
oi_link_service
oi_protocol_service
oi_trigger_service
oi_power_service
oi_standalone_service
settings_store
watchdog_service
```

Protection/control tasks must never depend on completion of TFT rendering.

---

## 23. Display bus and memory

Three displays share a write-only SPI bus:

```text
shared SCLK
shared MOSI
individual CS per TFT
```

Prefer:

- shared scratch framebuffer;
- direct drawing;
- dirty rectangles;
- DMA-backed serialized transfers.

Target display/measurement refresh:

```text
~10–20 Hz
```

with immediate redraws for user/state changes.

---

## 24. Open Instrument Project integration

### OI Stack mechanical and service-zone target

Supply 3 targets OI-05 but cannot make a completed physical-compatibility claim until OI-05 freezes its common footprint, unit height, mounting scheme, rear connector coordinates, keep-outs, and airflow convention.

The product shall track the following Stack requirements:

- select and document an integer-U height once the unit height is frozen;
- remain within the common X/Y envelope and mechanical keep-outs;
- provide two OI Link ports in the standard rear service zone;
- provide OI Power input and pass-through/output in the standard rear service zone;
- place OI Standalone USB-C and the dedicated `RECOVERY` control in their standard rear locations because Supply 3 targets OI-04;
- keep the instrument-specific 24 V XT30 input outside the standard service keep-outs and make it difficult to confuse with OI Power;
- preserve access to the front-panel controls, output terminals, service interfaces, and required power inputs when removed from a stack;
- document cooling needs and follow the common airflow convention once frozen;
- use visible, replaceable cabling rather than a proprietary electrical backplane;
- provide the required cable clearance and bend radius without obstructing adjacent service interfaces; and
- carry intended stack loads through the mechanical structure rather than through OI Link, OI Power, USB-C, or instrument-specific connectors.

OI Base is an optional reference source/gateway, not a requirement. Supply 3 shall operate with any conforming OI Power source and OI Link gateway/termination arrangement.

### Unstacked, USB-C only (OI Standalone)

- boot;
- communicate;
- negotiate PD;
- power UI;
- source outputs according to available contract.

### Unstacked, 24 V AUX only

- boot from AUX housekeeping fallback;
- provide full output-energy capability;
- operate locally.

### OI Stack, service power only

- boot;
- displays alive;
- OI Link alive;
- OI Protocol discovery/control available;
- OI Power pass-through maintained for downstream instruments;
- discoverable/configurable;
- outputs unavailable.

### OI Stack + USB-PD

- housekeeping from OI Power;
- output energy from USB-PD.

### OI Stack + 24 V AUX

- housekeeping from OI Power;
- deterministic full output energy from AUX.

---

## 25. Sourceability philosophy

OI Supply 3 follows the [Open Instrument Project](https://openinstrument.dev) sourceability principle:

> **No unicorn silicon.**

Prefer parts that are:

- active;
- publicly documented;
- quantity-one purchasable;
- available from established distributors;
- practical to assemble and rework;
- supported by durable ecosystems;
- replaceable where feasible.

---

## 26. Current candidate component summary

| Function | Current candidate | Status |
|---|---|---|
| MCU | STM32G0B1RE | Preferred |
| Channel regulator | TPS55289 | Preferred |
| Regulator current shunt | ~10 mΩ Kelvin | Target; verify |
| Output monitor | TBD; PAC1954-class under evaluation | Open |
| Display | 0.96", 160×80 ST7735S-class TFT | Prototype direction |
| Output connector | 2 mm banana/test socket | Chosen interface |
| Dedicated power input | XT30, 24 V | Current direction |
| OI Link | 2 × equivalent 8P8C, CAN-FD + trigger | Required by Stack; open PHY details track OI-01 |
| OI Protocol | Common semantics over OI Link and OI Standalone | Required by Link/Stack; wire layer TBD |
| OI Power | 12 V input + pass-through | Housekeeping only; connector/current details track OI-03 |
| OI Standalone | USB-C device + dedicated RECOVERY | Target; USB framing/update format TBD |
| OI Stack | Common envelope/service zone | Target; height and mechanical definition TBD |
| Output button | sourceable illuminated momentary switch | Open |
| Encoder | continuous push encoder, 30 detents | Prototype direction |
| Cooling | PCB copper + optional 30–40 mm fan | To characterize |

---

## 27. Draft 1 architecture diagram

```text
                                 OI SUPPLY 3

 OI LINK A [8P8C] ◄════ CAN-FD + TRIGGER ════► OI LINK B [8P8C]
                                  │
                                  ▼
                         ┌─────────────────────┐
                         │    STM32G0B1RE      │◄──── RECOVERY
 USB-C D+/D− ───────────►│                     │
 USB-C CC1/CC2 ─────────►│ USB / UCPD / CAN    │
 OI TRIGGER TAP ────────►│ UI / control        │
                         └─────────┬───────────┘
                                   │
                    ┌──────────────┼─────────────────┐
                    │              │                 │
                    ▼              ▼                 ▼
                 SPI TFTs        I²C              GPIO/UI
                 ×3             devices            buttons
                                                   encoder


                            HOUSEKEEPING POWER

 OI POWER IN ───────────────┬────────────────► OI POWER OUT
                            │
                            ▼
                   protected local branch (12 V)
                            │
 USB-C VBUS ────────────────┼──► service-power mux ─► 5 V / 3.3 V
                            │
 24 V XT30 ─────────────────┘


                              OUTPUT ENERGY

 USB-C VBUS ─► PD/eFuse/ideal diode ───────┐
                                            │
 24 V XT30 ─► eFuse/ideal diode ────────────┼──► VIN BUS
                                            │
 OI POWER ──────────────────────────────────X
                                            │
                 ┌──────────────────────────┼──────────────────────────┐
                 │                          │                          │
                 ▼                          ▼                          ▼
             TPS55289                   TPS55289                   TPS55289
                CH1                        CH2                        CH3
                 │                          │                          │
              10mΩ                       10mΩ                       10mΩ
                 │                          │                          │
             measurement                measurement                measurement
                 │                          │                          │
             disconnect?                disconnect?                disconnect?
                 │                          │                          │
           RED 2mm / BLACK          RED 2mm / BLACK          RED 2mm / BLACK
```

---

## 28. UI state summary

```text
OFF:
    TFT shows V setpoint + I limit
    TFT status = OFF in white / very pale grey
    button LED off

SELECTED:
    TFT shows blue selection rail / edited field
    button continues showing electrical state

CV:
    TFT status = CV
    button green

CC:
    TFT status = CC
    button amber

FAULT:
    TFT status = ERR, yellow blinking
    button yellow blinking
```

Input behavior:

```text
short channel-button press
    → select channel

long channel-button press
    → toggle output

encoder turn
    → change selected V or I

encoder press
    → switch V/I edit field

selection timeout
    → leave edit state
```

---

## 29. Open decisions after Draft 1

Intentionally not frozen:

1. exact 24 V input tolerance;
2. exact USB-PD/eFuse/protection components;
3. final power-path mux/ideal-diode topology;
4. exact USB-PD contract selection algorithm;
5. derating curve for lower-power PD contracts;
6. exact output measurement IC;
7. exact voltage/current accuracy targets;
8. exact current-limit shunt value;
9. exact output disconnect implementation;
10. exact button/switch part;
11. final three-channel front-panel geometry;
12. final encoder step sizes/acceleration;
13. selection timeout and long-press timings;
14. fan requirement;
15. thermal derating thresholds;
16. output ripple/noise target;
17. transient-response target;
18. startup/shutdown slew-rate behavior;
19. OVP implementation/threshold policy;
20. fault latch/clear policy;
21. calibration process;
22. exact OI Protocol schema and wire/USB mappings;
23. final PCB stack-up;
24. enclosure dimensions and airflow;
25. EMC strategy and pre-compliance plan;
26. maximum declared OI Power draw;
27. OI Power input/pass-through connector footprint, current rating, protection, and fault behavior after OI-03 freezes them;
28. OI Link CAN-FD bit rates, transceiver/protection implementation, termination mechanism, and misconnection test plan;
29. OI trigger PHY, receiver/source implementation, termination, and supported Supply 3 trigger actions;
30. relationship between OI service/control grounds, USB, the 24 V input, chassis/shield, and the common DUT output return;
31. OI Standalone USB class/framing, update format, recovery tooling, and VID/PID strategy;
32. OI Protocol identity/resource identifiers, units, structured fault schema, addressing, and security model;
33. OI Stack X/Y envelope, integer-U height, mounting pattern, rear connector coordinates, and keep-outs;
34. OI Stack airflow direction and cooling interaction with adjacent instruments;
35. explicit common-ground versus isolated service/control-domain architecture; and
36. final STM32 package/pin allocation and the application-independent USB-C attach/recovery implementation.

---

## 30. Prototype validation plan

### Electrical

- bring up one regulator channel;
- validate full 0.8–15 V range;
- validate 5 A low-voltage operation;
- characterize 20 W envelope;
- characterize CV/CC crossover;
- verify current-limit programming;
- measure efficiency;
- verify short-circuit behavior;
- verify OFF/discharge;
- validate measurement accuracy;
- validate calibration;
- document and test the relationships between service/control, USB, dedicated-input, chassis/shield, and common DUT-output grounds; and
- verify that no undocumented DUT-current path is created through an OI interface.

### OI Standalone

- validate USB 2.0 device communication and direct control;
- expose the same identity, resources, capabilities, state, measurements, and faults as OI Link;
- validate normal firmware update;
- validate the power-off/hold-RECOVERY/connect/reflash procedure with application firmware absent or corrupted;
- prove recovery from ordinary 5 V VBUS with OI Power and 24 V AUX removed and without USB-PD negotiation;
- prove USB-C attachment, boot selection, D+/D− communications, and recovery with application-controlled UCPD/PD policy unavailable;
- verify that RECOVERY is dedicated, clearly labelled, and protected from accidental use; and
- verify no backfeed among USB VBUS, OI Power, and the dedicated input.

### USB-PD

- validate USB data and PD simultaneously;
- negotiate fixed PDOs as available;
- validate 20 V / 3 A;
- validate 20 V / 4 A with compliant 5 A cable/source;
- verify 3 A cable behavior;
- verify PD loss while outputs active;
- verify no VBUS backfeed.

### OI Link and OI Protocol

- verify the Draft 1 8P8C pair allocation and that both reserved pairs remain unused;
- verify passive CAN-FD and trigger continuity between both OI Link ports while Supply 3 is unpowered;
- validate both ports as equivalent taps on one linear bus segment;
- validate the chosen CAN and trigger termination behavior at an end and in the middle of a chain;
- validate local tap lengths and signal integrity at the eventual worst-case chain length and CAN-FD rates;
- validate exposed-port protection and the accidental-Ethernet misconnection objective;
- validate trigger receive, arming, readiness, and any optional single-driver source capability;
- expose device identity, three stable channel resources, static capability, current availability, requested state, actual state, measurements, structured faults, power state, and protocol version;
- expose shared-return, physical-device membership, and aggregate-budget relationships for the three channel resources;
- expose the inspectable OI Link termination state and active/listener trigger role;
- verify that missing functional energy leaves Supply 3 discoverable while reporting outputs unavailable; and
- verify consistent OI Protocol semantics over OI Link and OI Standalone.

### OI Power

- measure and declare worst-case service-power draw;
- validate 12 V service-alive operation with no functional-energy source;
- validate input protection and prohibited-backfeed behavior;
- validate the OI Power input/pass-through path at the eventual full downstream chain current;
- verify that local off/fault states do not unnecessarily interrupt downstream service power; and
- expose service-power presence, draw/budget, and functional-power availability through OI Protocol.

### Dedicated input

- validate 24 V XT30;
- validate source priority with USB attached;
- validate reverse blocking;
- validate input transient/UV/OV behavior.

### OI Stack

- boot from OI Power only;
- verify displays/UI/OI Link/OI Protocol;
- verify outputs cannot enable without output-energy source;
- validate two OI Link ports, OI Power input/output, OI Standalone, and RECOVERY in their eventual standard rear service locations;
- validate the selected integer-U height, common envelope, mounting, keep-outs, and ordinary access outside the stack once OI-05 freezes them;
- validate airflow and thermal interaction with representative adjacent slices;
- validate stack loading, retention, cable clearance, connector strain relief, and service bend radii without using electrical connectors as structural members;
- add/remove AUX without reboot;
- add/remove USB-PD without reboot while OI Power remains valid.

### Thermal

- one channel at 20 W continuous;
- three channels at 60 W continuous;
- representative buck and boost points;
- closed enclosure;
- fan off/on characterization;
- regulator, inductor, shunt, connector and PCB temperatures.

### UI

- validate all three displays at real size;
- test short/long button gestures;
- test encoder steps;
- test selection timeout;
- test current-limit editing;
- test all status transitions;
- test power-budget refusal;
- test faults without losing electrical protection.

---

## 31. Draft 1 success criteria

Ready to advance toward schematic freeze when:

- one channel reliably delivers the complete 20 W envelope;
- 5 A low-voltage operation is thermally credible;
- hardware CC behavior is satisfactory;
- three-channel 60 W thermal strategy is demonstrated;
- 20 V / 4 A PD operation is proven;
- 24 V AUX path is proven;
- no dangerous backfeed exists;
- OI Power-only housekeeping is proven;
- maximum OI Power draw is measured and declared;
- OI Power pass-through and downstream-current requirements are designed and validated against the then-current OI-03 draft;
- OI Link pair allocation, passive continuity, termination, protection, and trigger behavior are validated;
- the OI Protocol model exposes static/current capability and requested/actual state consistently over OI Link and OI Standalone;
- the guaranteed OI Standalone recovery path works from ordinary 5 V VBUS with every other power source absent;
- service/control/DUT grounding relationships are documented and validated;
- the specification accurately describes the outputs as common-ground or implements and validates the intended isolation boundary;
- the Stack compatibility target identifies its eventual integer-U height and complies with the then-frozen OI-05 envelope/service-zone rules;
- three-channel UI remains usable;
- measurement architecture is selected;
- output OFF behavior is defined;
- protection/fault policies are defined;
- major parts pass the sourceability test.

---

## 32. Snapshot conclusion

As of **2026-08-16**, OI Supply 3 is defined as a compact three-channel, common-ground programmable electronics supply with:

```text
0.8–15 V
5 A maximum per channel
20 W maximum per channel
60 W aggregate
```

The architecture uses three separately controlled synchronous buck-boost channels with a common return, currently centered on the TPS55289 and controlled by an STM32G0B1-class MCU.

USB-C is both OI Standalone and an adaptive USB-PD power source. A 20 V / 4 A contract with a 5 A-rated electronically marked USB-C cable is the target for full USB-powered 60 W operation. A dedicated 24 V XT30 input provides deterministic full capability.

The project targets OI-01 Link, OI-02 Protocol, OI-03 Power, OI-04 Standalone, and OI-05 Stack. These are compatibility targets against the current working Draft 1 standards, not completed claims.

OI Link uses two equivalent 8P8C ports with passive CAN-FD/trigger continuity and requires shared OI Protocol semantics. The same conceptual identity, resources, capabilities, requested/actual state, measurements, and faults are exposed over OI Standalone.

OI Standalone provides direct USB-C control, normal update, and a dedicated physical recovery path that works from ordinary 5 V VBUS without application firmware or another power source.

OI Power remains part of the instrument, but only as **service/housekeeping power**. It keeps a stacked OI Supply 3 alive, discoverable and configurable without ever feeding the DUT output-energy path. Because Supply 3 targets Stack, OI Power includes a downstream-current-capable input/pass-through path.

The OI Stack height, common envelope, rear connector coordinates, mounting, keep-outs, and airflow implementation remain TBD until OI-05 freezes the corresponding mechanical definitions.

The UI direction is three permanently visible per-channel TFTs, three dedicated output buttons, three pairs of 2 mm output terminals, and one shared push encoder.

The next major engineering work is:

1. one-channel schematic and power-stage validation;
2. USB-PD/power-path design;
3. measurement architecture selection;
4. thermal prototype;
5. three-channel mechanical/UI layout;
6. protection and calibration definition;
7. OI Link/trigger physical prototype and termination validation;
8. OI Protocol resource/state model and dual-transport mapping;
9. OI Power input/pass-through and declared-draw validation;
10. OI Standalone 5 V recovery validation; and
11. grounding/isolation and Stack mechanical integration.

This document records the current architecture before those implementation choices begin to harden.
