# JinyuanFOC

Independent motor-control and integrated-actuator research for limited-scope verification.

![JinyuanFOC actuator prototype](assets/images/actuator-prototype.jpg)

![JinyuanFOC actuator exploded assembly](assets/images/actuator-exploded-view.png)

JinyuanFOC is an independently maintained experimental project built around a compact motor, reducer, custom drive electronics, magnetic sensing, and an HC32F448 controller. The repository records prototype hardware, mechanical files, host-side tools, selected firmware materials, protocol notes, and short functional demonstrations.

The current release is intended for small-scale engineering validation only. It is not a production motor drive, a certified safety system, or a complete open-source FOC distribution. The scope and availability of future firmware releases have not been committed.

> [!WARNING]
> Motor drives involve high currents, power electronics, and rapidly moving mechanisms. Disconnect the load, use a current-limited supply, secure the actuator, and keep an emergency stop available during initial testing. The documented parameters describe one prototype only and must not be treated as safe defaults for another motor, inverter, encoder, or mechanism.

## Project Scope

The prototype is used to verify the following engineering functions on a limited scale:

- Three-phase BLDC/PMSM field-oriented motor control
- Motor-side MA732 magnetic-encoder sampling
- MIT-style position, velocity, and feed-forward torque commands
- CAN and UART command paths
- Output-angle conversion through the integrated reducer
- Fault detection and output shutdown behavior
- Mechanical packaging, assembly, and dual-encoder concepts

Published videos demonstrate that individual functions were exercised on the shown prototype. They are not substitutes for calibrated measurements of accuracy, bandwidth, torque, thermal limits, efficiency, lifetime, or safety integrity.

## Repository Status

| Module | Status | Location |
| --- | --- | --- |
| Selected motor-control materials | Research snapshot; not a complete production release | [`source/`](source/) |
| Python CAN tools | Published | [`can_trans.py`](can_trans.py) |
| Control and protocol notes | Published | [`docs/`](docs/) |
| J-Link support and tool setup | Published | [`tools/`](tools/) |
| Mechanical STEP models | Published | [`hardware/mechanical/`](hardware/mechanical/) |
| Preliminary motor BOM | Published and explained | [`hardware/bom/`](hardware/bom/) |
| Images and functional demonstrations | Published | [`assets/`](assets/) |
| Schematics, PCB, and fabrication files | Planned release | [`hardware/electronics/`](hardware/electronics/) |
| Second-encoder electronics | Planned release | [`hardware/electronics/encoder-2/`](hardware/electronics/encoder-2/) |
| Reproducible quantitative datasets | Planned | [`results/experiments/`](results/experiments/) |

The repository does not currently provide a supported, directly buildable production firmware package. Users should not assume that a complete Keil project, all vendor dependencies, or future FOC implementation revisions will be published.

## Current Demonstrations

| Demonstration | Status |
| --- | --- |
| Physical integrated-actuator prototype | Assembled and photographed |
| Mechanical assembly and exploded model | Published as STEP models and images |
| MIT control mode | Short demonstration video published |
| MIT position command | Short demonstration video published |
| Velocity target at 1.2 rad/s | Short demonstration video published |
| Torque target at 5 N·cm | Short demonstration video published; calibrated torque data pending |
| Fault-protection behavior | Short demonstration video published; formal fault matrix pending |
| Position-tracking accuracy and bandwidth | Quantitative dataset pending |
| Thermal, efficiency, backlash, and load testing | Pending |

See [`assets/README.md`](assets/README.md) for photographs, renderings, and videos. Future measurements should include raw data, test conditions, instruments, firmware revision, supply settings, load, and analysis scripts under [`results/experiments/`](results/experiments/).

## Control Architecture

The validation setup follows this general command and control chain:

```text
Position / velocity / feed-forward torque target
                         │
                         ▼
MIT-style outer control and limits
                         │
                         ▼
Torque/current reference
                         │
                         ▼
dq current control → inverse Park → SVPWM → inverter → motor
                         ▲
                         └──── phase currents + rotor-angle feedback
```

The published material should be treated as prototype-specific. Motor constants, current-sense scaling, encoder alignment, controller gains, limits, dead time, and protection thresholds must be independently validated before use on another system.

## Hardware and Development Tools

### Core hardware

- Integrated motor, reducer, enclosure, bearings, fasteners, and magnetic components listed in [`hardware/bom/`](hardware/bom/)
- HC32F448-based controller and three-phase inverter
- Phase-current sensing, gate drive, and CAN transceiver
- MA732 motor-side magnetic encoder
- Current-limited DC supply and an accessible emergency stop
- J-Link or another supported SWD probe
- USB-to-UART or a compatible USB-CAN adapter for validation

### Development software

- Keil MDK-ARM or another supported Arm toolchain
- Official HC32F448 packages from the [XHSC HC32F448 product page](https://www.xhsc.com.cn/product/1213.html)
- XHCode, optional, for peripheral configuration and project generation
- SEGGER J-Link software when using J-Link
- Python 3 for the supplied host utilities
- A serial terminal such as XCOM or an equivalent program

Small redistributable HC32F448 J-Link flash algorithms and device descriptions are included under [`tools/programming/jlink/hc32f448/`](tools/programming/jlink/hc32f448/). Follow the [J-Link setup instructions](tools/programming/jlink/hc32f448/README.md) to register the HC32F448 device. Third-party proprietary installers are not mirrored in this repository.

## Repository Layout

```text
JinyuanFOC/
├─ source/                     # Selected prototype firmware materials
├─ firmware/projects/keil/     # Reserved project structure
├─ hardware/
│  ├─ electronics/            # Electronics release structure
│  ├─ mechanical/             # STEP models and mechanical documentation
│  └─ bom/                    # BOM source and explanation
├─ docs/                       # Control, protocol, assembly, setup, and tuning notes
├─ tools/                      # Programming, UART, and CAN support material
├─ assets/
│  ├─ images/                 # Photographs and CAD renderings
│  └─ videos/                 # Functional validation demonstrations
├─ results/experiments/        # Future reproducible measurements
├─ can_trans.py                # USB-CAN command-line utility
└─ can_code_control_test.py    # Position-sequence validation script
```

## Quick Start

### 1. Clone the repository and retrieve large assets

```bash
git clone https://github.com/ZJYSII/JinyuanFOC.git
cd JinyuanFOC
git lfs install
git lfs pull
```

### 2. Review the public materials

- Start with [`assets/README.md`](assets/README.md) for prototype media and demonstrations.
- Review [`hardware/`](hardware/) for mechanical files and the preliminary BOM.
- Read [`docs/development-setup.md`](docs/development-setup.md) before selecting HC32 packages, a debugger, or a programming workflow.
- Treat all firmware and protocol material as prototype-specific and unsupported for production use.

### 3. Use the CAN host utility

```bash
python -m pip install -r requirements.txt
python can_trans.py --port COM5 --serial-baud 921600 --pos 10 --vel 0 --tau 0
```

Interactive or repeated transmission:

```bash
python can_trans.py --port COM5 --interactive
python can_trans.py --port COM5 --pos 10 --repeat --hz 20
```

The utility targets the serial protocol used by a Damiao USB-CAN adapter. It is not a generic `python-can` interface. See [`docs/protocol/can.md`](docs/protocol/can.md) for the documented validation frame format.

### 4. Perform only controlled validation

Before applying power:

- Secure the actuator and remove the external load.
- Confirm supply polarity, voltage, and current limiting.
- Verify phase order, encoder direction, electrical alignment, and current-sense polarity.
- Confirm CAN/UART configuration and protection behavior.
- Start with conservative current, torque, velocity, and motion limits.

Immediately remove power if the actuator kicks, runs in the wrong direction, oscillates, draws sustained high current, overheats, reports discontinuous position, or produces unusual noise.

## Roadmap

- [x] Publish mechanical STEP models and a preliminary BOM
- [x] Publish prototype photographs, CAD renderings, and short validation videos
- [x] Document the HC32F448 and J-Link development workflow
- [ ] Release drive-board and second-encoder manufacturing files when ready
- [ ] Add complete assembly and wiring documentation
- [ ] Publish calibrated position, velocity, current, torque, and thermal measurements
- [ ] Add raw logs, test fixtures, procedures, and reproducible analysis scripts
- [ ] Characterize reducer efficiency, friction, backlash, compliance, and lifetime
- [ ] Define the long-term public-release boundary for firmware materials

## Frequently Asked Questions

### Is JinyuanFOC a complete open-source motor controller?

No. The public repository is currently a limited-scope research and validation record. It does not promise a complete, supported, or production-ready FOC firmware release.

### Can the firmware be built directly after cloning?

No supported standalone build is currently guaranteed. The repository may contain selected prototype materials, but users must supply and validate the correct HC32 startup files, linker settings, CMSIS layer, DDL version, IDE configuration, and hardware-specific parameters.

### Is XHCode required?

No. It is an optional assistant for generating or inspecting HC32 peripheral configuration. It is not a compiler, debugger, or substitute for the device-support package.

### Is a dedicated XHSC programmer required?

Not for the documented development path. A correctly configured J-Link can program and debug the HC32F448 over SWD. Vendor tools may still be useful for production fixtures or recovery.

### Are proprietary tools and drivers stored here?

No. Only small redistributable support files are included. Obtain proprietary installers, drivers, serial terminals, and vendor utilities from their official publishers.

### Do the videos prove the stated setpoint was accurately achieved?

No. They are small-scale functional demonstrations. Accuracy, bandwidth, torque, thermal behavior, and reliability require calibrated instruments, documented conditions, raw data, and repeatable analysis.

### Can another motor, reducer, encoder, or inverter be used?

Possibly, but it is a new engineering port rather than a drop-in substitution. Revalidate sensing, phase order, encoder alignment, motor constants, gear ratio, PWM and gate timing, limits, protection, and controller gains.

## License

An open-source license has not yet been selected for this repository. Until a `LICENSE` file is added, public visibility alone does not grant permission to copy, modify, or redistribute the contents. Third-party components remain subject to their respective licenses.

## Project Link

- Repository: <https://github.com/ZJYSII/JinyuanFOC>
- Questions and suggestions: use this repository's GitHub Issues
