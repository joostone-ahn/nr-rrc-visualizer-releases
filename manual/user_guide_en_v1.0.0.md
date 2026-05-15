# NR RRC Visualizer User Guide

**Version:** v1.0.0  
**Date:** 2026-05-15  
**Author:** JUSEOK AHN <ajs3013@lguplus.co.kr>

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

---

## 2. Screen Layout

| Area | Description |
|------|-------------|
| TopBar | App title, theme toggle (Light/Dark), session save/load buttons |
| Sidebar | Category-grouped navigation (Protocol, PHY/L1, MAC/RRC, Mobility) |
| Main Area | Content of the selected page |

**Category Structure:**
- **Protocol**: Log Input, Call Flow, Radio Bearers
- **PHY/L1**: BWP & Frequency Map, TDRA / TDD, CSI-RS, SRS, PUCCH
- **MAC/RRC**: Compliance Check, C-DRX
- **Mobility**: Meas Config, Cell Reselection

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

### 4.6 CORESET Bitmap

Inline visualization of the 45-bit CORESET bitmap for each BWP.
- Shows Duration, CCE-REG Mapping Type.
- Each bit corresponds to a 6-RB group.

### 4.7 SearchSpace Table

| Column | Description |
|--------|-------------|
| ID | SearchSpace ID |
| CORESET | Referenced CORESET ID |
| Source | PDCCH config source (Common/Dedicated) |
| Period | Monitoring periodicity |
| DCI Format | Supported DCI formats |
| Aggregation Level | Aggregation levels |
| Purpose | Usage (SI, Paging, RA, etc.) |

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

## 8. SRS

### 8.1 BWP Selector

SRS resources are displayed per UL BWP. Select a BWP from the dropdown.

### 8.2 Comb Pattern

Visualizes the SRS comb-2 or comb-4 pattern with comb offset.

### 8.3 RE Pattern

Displays SRS RE positions within 1 RB.
- RE placement based on comb spacing
- DMRS and SRS differentiation

### 8.4 Full RB View

View SRS RE placement across the full bandwidth.

---

## 9. PUCCH

### 9.1 Common / Dedicated Mode

- **Common**: Common PUCCH resources from `pucch-ConfigCommon`
  - TS 38.213 Table 9.2.1-1 popup for resource mapping reference
- **Dedicated**: Dedicated PUCCH resources from `pucch-Config`

### 9.2 Format 0–4

Visualizes RE mapping for each PUCCH Format:
- **Format 0**: 1–2 symbols, sequence-based
- **Format 1**: 4–14 symbols, sequence-based
- **Format 2**: 1–2 symbols, OFDM-based
- **Format 3**: 4–14 symbols, DFT-s-OFDM
- **Format 4**: 4–14 symbols, DFT-s-OFDM (interlaced)

### 9.3 RE Pattern

Displays DMRS and UCI (Uplink Control Information) RE positions within 1 RB.

### 9.4 RB Map

Shows PUCCH resource RB positions in the frequency domain.
- 1st hop / 2nd hop differentiation (when Frequency Hopping is applied)

### 9.5 Frequency Hopping

When Frequency Hopping is configured:
- Displays 1st hop and 2nd hop RB positions separately
- Hopping offset visualization

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

## 11. Measurement Config

### 11.1 Event A1–A6 Trigger Formulas

Displays measurement event trigger formulas defined in TS 38.331 5.5.4:

| Event | Condition |
|-------|-----------|
| A1 | Serving > Threshold |
| A2 | Serving < Threshold |
| A3 | Neighbor > Serving + Offset |
| A4 | Neighbor > Threshold |
| A5 | Serving < Threshold1 AND Neighbor > Threshold2 |
| A6 | Neighbor > SCell + Offset |

### 11.2 Parameters

For each event, the following parameters are displayed:
- Hysteresis
- Offset (a1/a2/a3/a4/a5/a6-Offset)
- TimeToTrigger
- Threshold values

---

## 12. Cell Reselection

### 12.1 SIB2 / SIB5

Displays cell reselection parameters from SIB2 (intra-frequency) and SIB5 (inter-frequency).
- s-IntraSearchP, s-NonIntraSearchP
- qRxLevMin, qQualMin
- cellReselectionPriority

### 12.2 Priority Chart

Visualizes reselection priority per frequency as a chart.
- References TS 38.304 reselection decision criteria.

---

## 13. Radio Bearers

### 13.1 SRB / DRB

Displays Signaling Radio Bearer (SRB) and Data Radio Bearer (DRB) configurations.

### 13.2 Protocol Stack

Visualizes the protocol stack for each bearer:
- **SDAP**: QoS Flow ↔ DRB mapping
- **PDCP**: Ciphering, integrity, header compression (RoHC)
- **RLC**: AM/UM mode, retransmission configuration

### 13.3 Cell Information

Displays cell information with hex/dec parsing.

---

## 14. C-DRX

### 14.1 Timing Diagram

Visualizes the Connected Mode DRX timing diagram:
- On Duration Timer
- Inactivity Timer
- DRX Cycle (Long/Short)
- HARQ RTT Timer
- Retransmission Timer

---

## 15. Source IE Tree Popup

A shared popup used across all analysis views (BWP, TDRA, CSI-RS, SRS, PUCCH).

### 15.1 Opening

Click the 🌳 button in each card header.

### 15.2 Features

| Feature | Description |
|---------|-------------|
| Drag | Drag the header bar to reposition the popup (avoid blocking the analysis view) |
| Details Toggle | Expand/collapse leaf data (parameter values) |
| Copy | Copy full path + data to clipboard |

### 15.3 Tree Structure

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

## 16. Session Save / Load

### 16.1 Save

1. After parsing is complete, click `Save Session` in the TopBar.
2. A `.zip` file is downloaded (contains: `frames.json`, `callflow.json`, `meta.json`).

### 16.2 Load

- Click `Load Session` in the TopBar, or drag & drop a `.zip` file onto the screen.
- The full analysis state is restored without re-parsing.

> Session save/load is only available when running locally.

---

## 17. Responsive Layout

### 17.1 Desktop (lg and above, 1024px+)

- CSS Grid-based layout
- Sidebar always visible
- Charts and tables displayed side by side in the main area

### 17.2 Mobile (below sm~md, under 640px)

- Sidebar collapses into a hamburger menu
- Main content uses scroll-snap for horizontal swipe navigation
- Bottom dot indicator shows current position

---

## 18. FAQ

**Q: File upload is not working.**  
A: File upload is only available when running locally (Native or Docker). Use `Load Sample` in the online demo.

**Q: I uploaded a DM log file (.qmdl, .hdf, .sdm) but got an error.**  
A: Verify that scat is installed: `pip install signalcat[fastcrc]`. When running via Docker, it is included automatically.

**Q: ARFCN is not displayed.**  
A: During SA initial access without measConfig, the ARFCN is unknown. Set the ARFCN toggle to OFF to display relative frequencies. You can also enter a known ARFCN manually.

**Q: Some NAS message directions show as "unknown" in Call Flow.**  
A: NAS messages encrypted after Security Mode may fail direction inference. The system attempts automatic inference via RRC frame cross-referencing when possible.

**Q: There are many Pending items in Compliance Check.**  
A: This means the relevant RRC/NAS messages are not present in the log. Adding more messages will update the verdicts.

**Q: What happens when I paste multiple messages in sequence?**  
A: They are automatically merged in order (MIB → SIB1 → rrcSetup → rrcReconfiguration). For duplicate parameters, the most recent message's value takes priority.

**Q: I get a "port 8333 already in use" error when running Docker.**  
A: Clean up the existing container: `docker stop nr-rrc-visualizer && docker rm nr-rrc-visualizer`

**Q: Can I change the theme?**  
A: Use the theme toggle button in the TopBar to switch between Light and Dark mode.

**Q: Can I use this on mobile?**  
A: Yes. The app supports responsive layout. On mobile, swipe horizontally to navigate between content sections.

**Q: How do I find the original location of a parameter in the Source IE Tree?**  
A: Click the 🌳 button to display all parameters from that view as a tree with their original IE paths within the RRC message.

---

**© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr> All rights reserved.**
