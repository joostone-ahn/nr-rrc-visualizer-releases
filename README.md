# 📡 NR RRC Visualizer

A web-based tool that parses 5G NR RRC messages and visualizes PHY configuration. Built to assist engineers analyzing radio network parameters from multi-vendor chipset logs.

[![License](https://img.shields.io/badge/License-Proprietary-red.svg)](LICENSE)

---

## 💡 Why This Tool?

Analyzing 5G NR RRC messages means cross-referencing dozens of parameters scattered across MIB, SIB1, rrcSetup, and rrcReconfiguration. Each requires understanding 3GPP spec formulas and chaining values in different units. On top of that, chipset vendors (Qualcomm, Samsung, Wireshark) use completely different log formats.

This tool bridges that gap by:
- **Auto-detecting** vendor format and parsing into a unified structure
- **Merging** multiple RRC messages while tracking the source IE path of every parameter
- **Visualizing** frequency domain layout, time domain allocation, RE-level resource mapping, and protocol sequences — all from a single log input

---

## ✨ Key Features

- **Multi-Vendor Parser** — Wireshark, Qualcomm QXDM, Samsung Shannon DM (auto-detected)
- **BWP & Frequency Map** — Point A → SSB → CORESET#0 → BWP layout with GSCN raster diagram
- **TDRA / TDD** — SLIV decoding + 14-symbol slot visualization + TDD UL/DL pattern
- **Call Flow** — NR RRC / NAS message sequence diagram with encrypted NAS direction inference
- **CSI-RS / SRS / PUCCH** — RE-level resource mapping with CDM group visualization
- **Compliance Check** — Security, Power Saving, VoNR compliance dashboard (Pass/Fail/Pending)
- **Measurement Config** — Event A1–A6 trigger formulas (TS 38.331)
- **Cell Reselection** — SIB2/SIB5 priority chart (TS 38.304)
- **DM Log Support** — Upload .qmdl/.hdf/.dlf/.sdm → auto-convert to PCAP via scat
- **Session Save/Load** — Save full analysis state as .zip, restore anytime
- **Source IE Tree** — Track exactly which message and IE path each parameter came from
- **Responsive UI** — Desktop grid + mobile swipe, Light/Dark theme

---

## 🌐 Online Demo

**[Try Online Demo](https://huggingface.co/spaces/Joostone/nr-rrc-visualizer)**

---

## 💻 Run Locally

Download `run-docker.sh` (macOS/Linux) or `run-docker.bat` (Windows) from [Releases](https://github.com/joostone-ahn/nr-rrc-visualizer-releases/releases/latest).

**Prerequisite**: [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running.

**macOS/Linux:**
```bash
chmod +x run-docker.sh
./run-docker.sh
```

**Windows:**
```cmd
run-docker.bat
```

> All dependencies are bundled in the Docker image. No Node.js, tshark, or scat installation needed. Re-run the script to update.

---

## 📖 How to Use

See the User Guide for detailed instructions:
- [English](https://github.com/joostone-ahn/nr-rrc-visualizer-releases/blob/main/manual/user_guide_en_v1.0.3.md)
- [한국어](https://github.com/joostone-ahn/nr-rrc-visualizer-releases/blob/main/manual/user_guide_kr_v1.0.3.md)

---

## 👤 Author

**JUSEOK AHN (안주석)**  
**Email**: ajs3013@lguplus.co.kr  
**Organization**: LG U+

---

## 📄 License

**© 2026 JUSEOK AHN. All rights reserved.**

This software is proprietary and confidential.

### Applicable For
- Network engineers analyzing 5G SA/NSA RRC configurations
- QA teams performing device certification and compliance testing
- Researchers working with 3GPP PHY/MAC/RRC layer parameters
- Field engineers debugging mobility, VoNR, and power saving issues
