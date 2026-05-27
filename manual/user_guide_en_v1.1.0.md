# NR RRC Visualizer User Guide

**Version:** v1.1.0  
**Date:** 2026-05-27  
**Author:** JUSEOK AHN <ajs3013@lguplus.co.kr>

---

## Table of Contents

- [1. How to Run](#1-how-to-run)
- [2. Screen Layout](#2-screen-layout)
- [3. Log Input](#3-log-input)
- [4. BWP & Frequency Map](#4-bwp--frequency-map)
- [5. TDRA / TDD](#5-tdra--tdd)
- [6. Call Flow](#6-call-flow)
- [7. CSI-RS](#7-csi-rs)
- [8. PUCCH](#8-pucch)
- [9. SRS](#9-srs)
- [10. Compliance Check](#10-compliance-check)
- [11. Source IE Tree Popup](#11-source-ie-tree-popup)
- [12. Session Save / Load](#12-session-save--load)
- [13. Responsive Layout](#13-responsive-layout)
- [Revision History](#revision-history)

---

## 1. How to Run

### 1.1 Docker

All dependencies (Node.js, tshark, scat) are bundled in the Docker image — no separate installation required.

**Prerequisite:** [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running.

**macOS/Linux:**
```bash
chmod +x run-docker.sh
./run-docker.sh
```

**Windows:**
```cmd
run-docker.bat
```

- The script automatically pulls the latest image, starts the server, and opens http://localhost:8333 in your browser.
- To stop: `Ctrl+C` (container is cleaned up automatically).
- To update: Re-run the script — it pulls the latest image automatically.
- Port conflict: Run `docker stop nr-rrc-visualizer && docker rm nr-rrc-visualizer` then retry.

---

## 2. Screen Layout

| Area | Description |
|------|-------------|
| TopBar | App title, theme toggle (Light/Dark), session save/load buttons |
| Sidebar | Category-grouped navigation (Protocol, PHY/L1, MAC/RRC, Mobility) |
| Main Area | Content of the selected page |

**Category Structure:**
- **Protocol**: Log Input, Call Flow
- **PHY/L1**: BWP & Frequency Map, TDRA / TDD, CSI-RS, SRS, PUCCH
- **Mobility**: Meas Config, Cell Reselection
- **Other**: Compliance Check

---

## 3. Log Input

### 3.1 Text Input

Paste RRC messages into the text area for parsing.

**Supported Formats (auto-detected):**
| Format | Characteristics |
|--------|----------------|
| Wireshark | Indentation-based `key: value` tree |
| Qualcomm QXDM | ASN.1 value notation `{ key value }` |
| Samsung Shannon DM | Tab-separated `Decoded_OTA` |

- Pasting multiple messages in sequence triggers automatic merging (MIB → SIB1 → rrcSetup → rrcReconfiguration).
- The most recent message's values take priority.

### 3.2 File Upload

| File Type | Description |
|-----------|-------------|
| `.pcap` / `.pcapng` | Parsed via tshark |
| `.qmdl`, `.hdf`, `.dlf` | Qualcomm DM log → auto-converted to PCAP via scat |
| `.sdm` | Samsung Shannon DM log → auto-converted to PCAP via scat |
| `.zip` | Restore a previously saved session |

- When uploading DM logs, the converted PCAP file is available for download.

### 3.3 Load Session

- `Load Sample` button: Start analysis immediately with built-in sample data.
- Drag & drop a `.zip` file: Restore a previously saved session.

---

## 4. BWP & Frequency Map

Visualizes the 5G NR frequency domain structure hierarchically.

### 4.1 Frequency Map (Left Panel)

- Y-axis: RB number (0 ~ carrierBandwidth)
- X-axis: Columns for SSB, CORESET#0, BWP#0, CORESET#1, BWP#1, ...
- Click any block to switch the right-side detail panel.
- Group brackets: Common (SSB + CORESET#0), BWP#N (BWP + CORESETs)
- The frequency grid uses 20 RB intervals, and labels near SSB/CORESET#0 boundaries are automatically hidden to avoid overlap.

### 4.2 MIB Decoding

Displays parameters extracted from MIB:
- `subCarrierSpacingCommon`
- `k_SSB`
- `controlResourceSetZero` (decoded per TS 38.213 Table 13-4)
- `searchSpaceZero` (decoded per TS 38.213 Table 13-11)

### 4.3 SIB1 Decoding

Displays parameters extracted from SIB1:
- `offsetToPointA`
- `carrierBandwidth`
- `subcarrierSpacing`
- Band information

### 4.4 ARFCN Toggle

- **ON**: Displays absolute frequency (MHz) based on ssbFrequency from measConfig.
- **OFF**: Displays relative frequency (ΔMHz) with SSB Center = 0.

> During SA initial access without measConfig, the ARFCN can be entered manually. If measConfig is present, it is applied automatically.

### 4.5 GSCN Raster

Visualizes the SSB Center → Point A calculation as a GSCN vs CRB grid diagram.

```
Point A = SSB Center - 120×ssbSCS - k_SSB×15kHz - offsetToPointA×180kHz
```

### 4.6 CORESET Table

Displays CORESET configuration information for each BWP in a table.

| Column | Description |
|--------|-------------|
| ID | CORESET ID |
| Duration | Number of CORESET symbols |
| CCE-REG Mapping | interleaved / nonInterleaved |
| Bitmap | 45-bit frequencyDomainResources (6-RB group granularity) |
| Source | The RRC message that configured this CORESET (SIB1, rrcSetup, rrcReconfiguration) |

> **Note:** If the same CORESET ID is configured by different RRC messages, each message appears as a separate row. This allows tracking configuration changes across messages.

#### 4.6.1 CORESET Bitmap Visualization

Inline visualization of the 45-bit CORESET bitmap for each BWP.
- Shows Duration, CCE-REG Mapping Type.
- Each bit corresponds to a 6-RB group.

### 4.7 SearchSpace Table

| Column | Description |
|--------|-------------|
| ID | SearchSpace ID |
| CORESET | Referenced CORESET ID |
| Source | The RRC message that configured this SearchSpace (SIB1, rrcSetup, rrcReconfiguration) |
| Period | Monitoring periodicity |
| DCI Format | Supported DCI formats |
| Aggregation Level | Aggregation levels |
| Purpose | Usage (SI, Paging, RA, etc.) |

> **Note:** If the same SearchSpace ID is configured by different RRC messages, each message appears as a separate row. This allows tracking which message configured each SearchSpace.

### 4.8 Source IE Tree

Click the 🌳 button to view the original IE path within the RRC message for each parameter in a tree structure.

---

## 5. TDRA / TDD

### 5.1 SLIV Decoding

Decodes PDSCH/PUSCH SLIV (Start and Length Indicator Value) and displays it as a 14-symbol slot visualization.
- K0/K2 slot offset
- Mapping Type A/B
- Start symbol (S) and length (L) display

### 5.2 TDD Pattern

Visualizes the TDD UL/DL pattern at slot level.
- D (Downlink) / U (Uplink) / F (Flexible) symbol mapping
- Pattern 1 / Pattern 2 support

### 5.3 BWP Selector

When multiple BWPs exist, select from the dropdown to view the TDRA for that BWP.

### 5.4 Common / Dedicated Toggle

- **Common**: Common TDRA from SIB1 or rrcSetup
- **Dedicated**: Dedicated TDRA from rrcReconfiguration

---

## 6. Call Flow

### 6.1 Sequence Diagram

Displays NR RRC / LTE RRC / NAS-5GS / NAS-EPS messages as a time-ordered sequence diagram.
- UE ↔ Network directional arrows
- Automatic extraction of inner messages from NAS containers (hierarchical classification)
- Automatic NAS direction inference (GSMTAP header, RRC cross-reference, Encrypted Data proximity matching)

### 6.2 Message Selection

Click a message in the sequence diagram to display its decoded content in the right-side detail panel.

### 6.3 Detail Panel

Displays the full decoded tree of the selected message.
- Hierarchical indentation for IE structure
- Search function for quickly finding specific parameters

### 6.4 Keyboard Navigation

| Key | Action |
|-----|--------|
| `↑` / `↓` | Select previous/next message |
| `Enter` | Open detail panel |

---

## 7. CSI-RS

### 7.1 NZP/ZP Resource Table

| Column | Description |
|--------|-------------|
| Type | NZP or ZP |
| ID | Resource ID |
| Symbol | firstOFDMSymbolInTimeDomain |
| Ports | Number of ports |
| Density | RE density per RB |
| CDM | CDM type (noCDM, fd-CDM2, cdm4-FD2-TD2, cdm8-FD2-TD4) |
| FD Alloc | Row type + binary bitmap |
| RBs | Number of allocated RBs |

### 7.2 RE Pattern

Visualizes the RE occupancy pattern within 1 RB (12 subcarriers × 14 symbols) based on TS 38.211 Table 7.4.1.5.3-1.
- Color-coded by CDM group
- Automatic row type detection (row1, row2, row4, other)
- frequencyDomainAllocation bitmap interpretation

### 7.3 Full RB View

View CSI-RS RE placement across the full bandwidth (14 symbols × N RBs).

### 7.4 Section Grouping by Source Message

CSI-RS resources are grouped by the RRC message they were configured in (rrcSetup, rrcReconfiguration, etc.).

---

## 8. PUCCH

### 8.1 Common / Dedicated Mode

- **Common**: Common PUCCH resources from `pucch-ConfigCommon`
  - TS 38.213 Table 9.2.1-1 popup for resource mapping reference
- **Dedicated**: Dedicated PUCCH resources from `pucch-Config`

### 8.2 Format 0–4

Visualizes RE mapping for each PUCCH Format:
- **Format 0**: 1–2 symbols, sequence-based
- **Format 1**: 4–14 symbols, sequence-based
- **Format 2**: 1–2 symbols, OFDM-based
- **Format 3**: 4–14 symbols, DFT-s-OFDM
- **Format 4**: 4–14 symbols, DFT-s-OFDM (interlaced)

### 8.3 RE Pattern

Displays DMRS and UCI (Uplink Control Information) RE positions within 1 RB.

### 8.4 RB Map

Shows PUCCH resource RB positions in the frequency domain.
- 1st hop / 2nd hop differentiation (when Frequency Hopping is applied)

### 8.5 Frequency Hopping

When Frequency Hopping is configured:
- Displays 1st hop and 2nd hop RB positions separately
- Hopping offset visualization

---

## 9. SRS

### 9.1 BWP Selector

SRS resources are displayed per UL BWP. Select a BWP from the dropdown.

### 9.2 Comb Pattern

Visualizes the SRS comb-2 or comb-4 pattern with comb offset.

### 9.3 RE Pattern

Displays SRS RE positions within 1 RB.
- RE placement based on comb spacing
- DMRS and SRS differentiation

### 9.4 Full RB View

View SRS RE placement across the full bandwidth.

---

## 10. Compliance Check

### 10.1 Donut Chart

Displays the Pass/Fail/Pending ratio of all validation items as a donut chart.

### 10.2 Verdict Criteria

| Status | Meaning |
|--------|---------|
| ✅ Pass | Configuration exists and meets requirements |
| ❌ Fail | Configuration exists but does not meet requirements |
| ⏳ Pending | Related log not available; cannot determine |

### 10.3 Validation Categories

**Security / Privacy:**
- RRC/NAS Ciphering (nea2 or higher)
- RRC/NAS Integrity (nia2 or higher)
- UP Ciphering (user data encryption)
- UP Integrity (user data integrity)
- SUCI/ECIES (privacy concealment)

**Energy Efficient:**
- C-DRX Configuration
- BWP Adaptation (timer ≤ 5s)
- RRC CONNECTED → INACTIVE

**RAT Selection:**
- SA Camping BWP (power-saving BWP to prevent SA de-prioritization)

**VoNR:**
- C-DRX, BWP Adaptation, RoHC, EPS Fallback / iRAT Handover

### 10.4 VoNR Session Auto-Detection

Detects `5QI=1` (conversational voice) in NAS PDU Session Establishment, then cross-validates with the DRB mapped to that PDU Session ID in RRC radioBearerConfig to automatically determine VoNR session presence.

---

## 11. Source IE Tree Popup

A shared popup used across all analysis views (BWP, TDRA, CSI-RS, SRS, PUCCH).

### 11.1 Opening

Click the 🌳 button in each card header.

### 11.2 Features

| Feature | Description |
|---------|-------------|
| Drag | Drag the header bar to reposition the popup (avoid blocking the analysis view) |
| Details Toggle | Expand/collapse leaf data (parameter values) |
| Copy | Copy full path + data to clipboard |

### 11.3 Tree Structure

IE paths from the same message are automatically merged into a single branch:

```
rrcReconfiguration
└─ spCellConfigDedicated
   └─ downlinkBWP-ToAddModList
      └─ [0]
         ├─ genericParameters
         └─ pdcch-Config
            ├─ searchSpaces
            └─ coresets
```

---

## 12. Session Save / Load

### 12.1 Save

1. After parsing is complete, click `Save Session` in the TopBar.
2. A `.zip` file is downloaded (contains: `frames.json`, `callflow.json`, `meta.json`).

### 12.2 Load

- Click `Load Session` in the TopBar, or drag & drop a `.zip` file onto the screen.
- The full analysis state is restored without re-parsing.

> Session save/load is only available when running locally.

---

## 13. Responsive Layout

### 13.1 Desktop (lg and above, 1024px+)

- CSS Grid-based layout
- Sidebar always visible
- Charts and tables displayed side by side in the main area

### 13.2 Mobile (below sm~md, under 640px)

- Sidebar collapses into a hamburger menu
- Main content uses scroll-snap for horizontal swipe navigation
- Bottom dot indicator shows current position

---

## Revision History

| Version | Date | Changes |
|---------|------|---------|
| v1.0.0 | 2026-05-15 | Initial release |
| v1.0.1 | 2026-05-18 | Fix VoNR C-DRX compliance validation logic |
| v1.0.2 | 2026-05-25 | BWP Map: multi-message sourceIE tracking, CORESET#0 display fix, chart grid improvements |
| v1.0.3 | 2026-05-26 | BWP Map: ARFCN auto-applied from A3 measObject, parser field scoping improvements |
| v1.1.0 | 2026-05-27 | Parser architecture refactoring, PUCCH view improvements (RB Map colors, Hop display), ARFCN toggle fix |

---

**© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr> All rights reserved.**
