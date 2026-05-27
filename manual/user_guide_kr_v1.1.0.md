# NR RRC Visualizer 사용자 가이드

**버전:** v1.1.0  
**최종 수정일:** 2026-05-27  
**작성자:** JUSEOK AHN <ajs3013@lguplus.co.kr>

---

## 목차

- [1. 실행 방법](#1-실행-방법)
- [2. 화면 구성](#2-화면-구성)
- [3. Log Input (로그 입력)](#3-log-input-로그-입력)
- [4. BWP & Frequency Map](#4-bwp--frequency-map)
- [5. TDRA / TDD](#5-tdra--tdd)
- [6. Call Flow](#6-call-flow)
- [7. CSI-RS](#7-csi-rs)
- [8. PUCCH](#8-pucch)
- [9. SRS](#9-srs)
- [10. Compliance Check](#10-compliance-check)
- [11. Source IE Tree 팝업](#11-source-ie-tree-팝업)
- [12. 세션 저장 / 불러오기](#12-세션-저장--불러오기)
- [13. 반응형 레이아웃](#13-반응형-레이아웃)
- [개정이력](#개정이력)

---

## 1. 실행 방법

### 1.1 Docker 실행

모든 의존성(Node.js, tshark, scat)이 Docker 이미지에 포함되어 있어 별도 설치가 필요 없습니다.

**사전 요구사항:** [Docker Desktop](https://www.docker.com/products/docker-desktop/) 설치 및 실행

**macOS/Linux:**
```bash
chmod +x run-docker.sh
./run-docker.sh
```

**Windows:**
```cmd
run-docker.bat
```

- 스크립트가 최신 이미지를 자동으로 다운로드하고, 서버를 시작한 뒤 http://localhost:8333 을 브라우저에서 엽니다.
- 종료: `Ctrl+C` (컨테이너 자동 정리)
- 업데이트: 스크립트를 다시 실행하면 최신 이미지를 자동으로 pull합니다.
- 포트 충돌 시: `docker stop nr-rrc-visualizer && docker rm nr-rrc-visualizer` 후 재실행

---

## 2. 화면 구성

| 영역 | 설명 |
|------|------|
| TopBar | 프로그램 제목, 테마 전환 (Light/Dark), 세션 저장/불러오기 버튼 |
| Sidebar | 카테고리별 네비게이션 (Protocol, PHY/L1, MAC/RRC, Mobility) |
| 메인 영역 | 선택된 페이지의 콘텐츠 |

**카테고리 구성:**
- **Protocol**: Log Input, Call Flow
- **PHY/L1**: BWP & Frequency Map, TDRA / TDD, CSI-RS, SRS, PUCCH
- **Mobility**: Meas Config, Cell Reselection
- **기타**: Compliance Check

---

## 3. Log Input (로그 입력)

### 3.1 텍스트 입력 (Text Input)

텍스트 영역에 RRC 메시지를 붙여넣고 파싱합니다.

**지원 포맷 (자동 감지):**
| 포맷 | 특징 |
|------|------|
| Wireshark | 들여쓰기 기반 `key: value` 트리 |
| Qualcomm QXDM | ASN.1 값 표기법 `{ key value }` |
| Samsung Shannon DM | 탭 구분 `Decoded_OTA` |

- 복수 메시지를 순서대로 붙여넣으면 자동 병합됩니다 (MIB → SIB1 → rrcSetup → rrcReconfiguration)
- 최신 메시지의 값이 우선 적용됩니다

### 3.2 파일 업로드 (File Upload)

| 파일 형식 | 설명 |
|-----------|------|
| `.pcap` / `.pcapng` | tshark로 파싱 |
| `.qmdl`, `.hdf`, `.dlf` | Qualcomm DM 로그 → scat으로 PCAP 변환 후 파싱 |
| `.sdm` | Samsung Shannon DM 로그 → scat으로 PCAP 변환 후 파싱 |
| `.zip` | 저장된 세션 복원 |

- DM 로그 업로드 시 변환된 PCAP 파일을 다운로드할 수 있습니다.

### 3.3 세션 불러오기 (Load Session)

- `Load Sample` 버튼: 내장 샘플 데이터로 즉시 분석 시작
- `.zip` 파일 드래그 앤 드롭: 이전에 저장한 세션 복원

---

## 4. BWP & Frequency Map

5G NR 주파수 도메인 구조를 계층적으로 시각화합니다.

### 4.1 Frequency Map (좌측)

- 세로축: RB 번호 (0 ~ carrierBandwidth)
- 가로축: SSB, CORESET#0, BWP#0, CORESET#1, BWP#1, ...
- 각 블록 클릭으로 우측 상세 패널 전환
- 그룹 브래킷: Common (SSB + CORESET#0), BWP#N (BWP + CORESETs)
- 주파수 그리드는 20 RB 간격으로 표시되며, SSB/CORESET#0 경계 부근의 라벨은 겹침 방지를 위해 자동으로 숨겨집니다.

### 4.2 MIB Decoding

MIB에서 추출한 파라미터를 표시합니다:
- `subCarrierSpacingCommon`
- `k_SSB`
- `controlResourceSetZero` (TS 38.213 Table 13-4 기반 디코딩)
- `searchSpaceZero` (TS 38.213 Table 13-11 기반 디코딩)

### 4.3 SIB1 Decoding

SIB1에서 추출한 파라미터를 표시합니다:
- `offsetToPointA`
- `carrierBandwidth`
- `subcarrierSpacing`
- Band 정보

### 4.4 ARFCN 토글

- **ON**: measConfig의 ssbFrequency를 기반으로 절대 주파수(MHz) 표시
- **OFF**: SSB Center = 0 기준 상대 주파수(ΔMHz) 표시

> SA 초기 접속 시 measConfig가 없으면 ARFCN을 수동 입력할 수 있습니다. measConfig가 존재하면 자동 적용됩니다.

### 4.5 GSCN Raster

SSB Center → Point A 계산 과정을 GSCN vs CRB 그리드 다이어그램으로 시각화합니다.

```
Point A = SSB Center - 120×ssbSCS - k_SSB×15kHz - offsetToPointA×180kHz
```

### 4.6 CORESET Table

각 BWP에 대해 CORESET 설정 정보를 테이블로 표시합니다.

| 컬럼 | 설명 |
|------|------|
| ID | CORESET ID |
| Duration | CORESET 심볼 수 |
| CCE-REG Mapping | interleaved / nonInterleaved |
| Bitmap | 45-bit frequencyDomainResources (6 RB 그룹 단위) |
| Source | 해당 CORESET을 설정한 RRC 메시지 (SIB1, rrcSetup, rrcReconfiguration) |

> **참고:** 동일한 CORESET ID가 서로 다른 RRC 메시지에서 설정된 경우, 각 메시지별로 별도의 행으로 표시됩니다. 이를 통해 메시지 간 설정 변경 이력을 추적할 수 있습니다.

#### 4.6.1 CORESET Bitmap 시각화

각 BWP에 대해 45-bit CORESET 비트맵을 인라인으로 시각화합니다.
- Duration, CCE-REG Mapping Type 표시
- 비트맵의 각 비트가 6 RB 그룹에 대응

### 4.7 SearchSpace Table

| 컬럼 | 설명 |
|------|------|
| ID | SearchSpace ID |
| CORESET | 참조하는 CORESET ID |
| Source | 해당 SearchSpace를 설정한 RRC 메시지 (SIB1, rrcSetup, rrcReconfiguration) |
| Period | 모니터링 주기 |
| DCI Format | 지원하는 DCI 포맷 |
| Aggregation Level | 집합 레벨 |
| Purpose | 용도 (SI, Paging, RA, etc.) |

> **참고:** 동일한 SearchSpace ID가 서로 다른 RRC 메시지에서 설정된 경우, 각 메시지별로 별도의 행으로 표시됩니다. 이를 통해 어떤 메시지가 해당 SearchSpace를 설정했는지 추적할 수 있습니다.

### 4.8 Source IE Tree

🌳 버튼 클릭으로 각 파라미터의 원본 RRC 메시지 내 IE 경로를 트리 구조로 확인할 수 있습니다.

---

## 5. TDRA / TDD

### 5.1 SLIV Decoding

PDSCH/PUSCH의 SLIV(Start and Length Indicator Value)를 디코딩하여 14-symbol 슬롯 시각화로 표시합니다.
- K0/K2 슬롯 오프셋
- Mapping Type A/B
- 시작 심볼(S)과 길이(L) 표시

### 5.2 TDD Pattern

TDD UL/DL 패턴을 슬롯 레벨로 시각화합니다.
- D (Downlink) / U (Uplink) / F (Flexible) 심볼 매핑
- Pattern 1 / Pattern 2 지원

### 5.3 BWP Selector

복수 BWP가 존재할 경우 드롭다운으로 선택하여 해당 BWP의 TDRA를 확인합니다.

### 5.4 Common / Dedicated 토글

- **Common**: SIB1 또는 rrcSetup의 공통 TDRA
- **Dedicated**: rrcReconfiguration의 전용 TDRA

---

## 6. Call Flow

### 6.1 시퀀스 다이어그램

NR RRC / LTE RRC / NAS-5GS / NAS-EPS 메시지를 시간순 시퀀스 다이어그램으로 표시합니다.
- UE ↔ Network 방향 화살표
- NAS container 내부 메시지 자동 추출 (계층적 분류)
- NAS 방향 자동 추론 (GSMTAP 헤더, RRC 교차 참조, Encrypted Data 인접 매칭)

### 6.2 메시지 선택

시퀀스 다이어그램에서 메시지를 클릭하면 우측 상세 패널에 해당 메시지의 디코딩된 내용이 표시됩니다.

### 6.3 Detail Panel

선택된 메시지의 전체 디코딩 트리를 표시합니다.
- 계층적 들여쓰기로 IE 구조 표현
- 검색 기능으로 특정 파라미터 빠르게 찾기

### 6.4 키보드 네비게이션

| 키 | 동작 |
|----|------|
| `↑` / `↓` | 이전/다음 메시지 선택 |
| `Enter` | 상세 패널 열기 |

---

## 7. CSI-RS

### 7.1 NZP/ZP Resource Table

| 컬럼 | 설명 |
|------|------|
| Type | NZP 또는 ZP |
| ID | Resource ID |
| Symbol | firstOFDMSymbolInTimeDomain |
| Ports | 포트 수 |
| Density | RB당 RE 밀도 |
| CDM | CDM 타입 (noCDM, fd-CDM2, cdm4-FD2-TD2, cdm8-FD2-TD4) |
| FD Alloc | Row type + 바이너리 비트맵 |
| RBs | 할당 RB 수 |

### 7.2 RE Pattern

TS 38.211 Table 7.4.1.5.3-1 기반으로 1 RB(12 subcarrier × 14 symbol) 내 RE 점유 패턴을 시각화합니다.
- CDM 그룹별 색상 구분
- Row type 자동 감지 (row1, row2, row4, other)
- frequencyDomainAllocation 비트맵 해석

### 7.3 Full RB View

14 symbols × N RBs 전체 대역에서의 CSI-RS RE 배치를 확인합니다.

### 7.4 소스 메시지별 그룹핑

CSI-RS 리소스가 어떤 RRC 메시지(rrcSetup, rrcReconfiguration 등)에서 설정되었는지 그룹별로 분류하여 표시합니다.

---

## 8. PUCCH

### 8.1 Common / Dedicated 모드

- **Common**: `pucch-ConfigCommon`에서 설정된 공통 PUCCH 리소스
  - TS 38.213 Table 9.2.1-1 팝업으로 리소스 매핑 확인
- **Dedicated**: `pucch-Config`에서 설정된 전용 PUCCH 리소스

### 8.2 Format 0–4

각 PUCCH Format별 RE 매핑을 시각화합니다:
- **Format 0**: 1~2 symbol, sequence-based
- **Format 1**: 4~14 symbol, sequence-based
- **Format 2**: 1~2 symbol, OFDM-based
- **Format 3**: 4~14 symbol, DFT-s-OFDM
- **Format 4**: 4~14 symbol, DFT-s-OFDM (interlaced)

### 8.3 RE Pattern

1 RB 내에서 DMRS와 UCI(Uplink Control Information)의 RE 위치를 구분하여 표시합니다.

### 8.4 RB Map

주파수 도메인에서 PUCCH 리소스의 RB 위치를 표시합니다.
- 1st hop / 2nd hop 구분 (Frequency Hopping 적용 시)

### 8.5 Frequency Hopping

Frequency Hopping이 설정된 경우:
- 1st hop과 2nd hop의 RB 위치를 각각 표시
- Hopping offset 시각화

---

## 9. SRS

### 9.1 BWP Selector

UL BWP별로 SRS 리소스를 분리하여 표시합니다. 드롭다운으로 BWP를 선택합니다.

### 9.2 Comb Pattern

SRS의 comb-2 또는 comb-4 패턴과 comb offset을 시각화합니다.

### 9.3 RE Pattern

1 RB 내에서 SRS가 점유하는 RE 위치를 표시합니다.
- Comb 간격에 따른 RE 배치
- DMRS와 SRS 구분

### 9.4 Full RB View

전체 대역에서의 SRS RE 배치를 확인합니다.

---

## 10. Compliance Check

### 10.1 Donut Chart

전체 검증 항목의 Pass/Fail/Pending 비율을 도넛 차트로 표시합니다.

### 10.2 판정 기준

| 상태 | 의미 |
|------|------|
| ✅ Pass | 설정이 존재하고 요구사항 충족 |
| ❌ Fail | 설정이 존재하지만 요구사항 미충족 |
| ⏳ Pending | 관련 로그가 없어 판정 불가 |

### 10.3 검증 카테고리

**Security / Privacy:**
- RRC/NAS Ciphering (nea2 이상)
- RRC/NAS Integrity (nia2 이상)
- UP Ciphering (사용자 데이터 암호화)
- UP Integrity (사용자 데이터 무결성)
- SUCI/ECIES (Privacy concealment)

**Energy Efficient:**
- C-DRX Configuration
- BWP Adaptation (timer ≤ 5s)
- RRC CONNECTED → INACTIVE

**RAT Selection:**
- SA Camping BWP (절전 BWP 설정으로 SA 우선순위 저하 방지)

**VoNR:**
- C-DRX, BWP Adaptation, RoHC, EPS Fallback / iRAT Handover

### 10.4 VoNR 세션 자동 감지

NAS PDU Session Establishment에서 `5QI=1` (conversational voice)을 탐지하고, RRC radioBearerConfig에서 해당 PDU Session ID와 매핑된 DRB를 교차 검증하여 VoNR 세션을 자동 판정합니다.

---

## 11. Source IE Tree 팝업

모든 분석 뷰(BWP, TDRA, CSI-RS, SRS, PUCCH)에서 공통으로 사용되는 팝업입니다.

### 11.1 열기

각 카드 헤더의 🌳 버튼을 클릭합니다.

### 11.2 기능

| 기능 | 설명 |
|------|------|
| 드래그 | 헤더 바를 드래그하여 팝업 위치 이동 (분석 화면을 가리지 않도록) |
| Details 토글 | Leaf 데이터(파라미터 값) 접기/펼치기 |
| Copy | 전체 경로 + 데이터를 클립보드에 복사 |

### 11.3 트리 구조

동일 메시지에서 온 IE 경로들이 자동으로 하나의 브랜치로 통합됩니다:

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

## 12. 세션 저장 / 불러오기

### 12.1 저장

1. 로그 파싱 완료 후 TopBar의 `Save Session` 버튼 클릭
2. `.zip` 파일이 다운로드됩니다 (포함: `frames.json`, `callflow.json`, `meta.json`)

### 12.2 불러오기

- TopBar의 `Load Session` 버튼 클릭 또는 `.zip` 파일을 화면에 드래그 앤 드롭
- 재파싱 없이 전체 분석 상태가 복원됩니다

> 세션 저장/불러오기는 로컬 실행 시에만 사용 가능합니다.

---

## 13. 반응형 레이아웃

### 13.1 데스크톱 (lg 이상, 1024px+)

- CSS Grid 기반 레이아웃
- Sidebar 고정 표시
- 메인 영역에 차트와 테이블이 나란히 배치

### 13.2 모바일 (sm~md, 640px 미만)

- Sidebar가 햄버거 메뉴로 전환
- 메인 콘텐츠는 scroll-snap 기반 좌우 스와이프
- 하단 도트 인디케이터로 현재 위치 표시

---

## 개정이력

| 버전 | 날짜 | 변경 내용 |
|------|------|-----------|
| v1.0.0 | 2026-05-15 | 최초 릴리즈 |
| v1.0.1 | 2026-05-18 | VoNR C-DRX Compliance 검증 로직 수정 |
| v1.0.2 | 2026-05-25 | BWP Map: 복수 메시지 sourceIE 누적 표시, CORESET#0 미표시 수정, 차트 grid 개선 |
| v1.0.3 | 2026-05-26 | BWP Map: ARFCN 자동 적용(A3 measObject), 파서 필드 스코프 개선 |
| v1.1.0 | 2026-05-27 | 파서 아키텍처 리팩토링, PUCCH 뷰 개선(RB Map 색상/Hop 표시), ARFCN 토글 수정 |

---

**© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr> All rights reserved.**
