# 🚦 3D 무한 도시 횡단보도 & 도로 횡단 시뮬레이터 (v2.3)

> **Live GitHub Pages URL**: [https://kyum-yee.github.io/crosswalk-3d-simulator/](https://kyum-yee.github.io/crosswalk-3d-simulator/)
> 
> **소스 코드 위치**:
> - [`~/tem/index.html`](file:///Users/jakesmacair/tem/index.html) (단일 독립형 실행 파일, 총 2,748줄)
> - [`~/tem/crosswalk_3d.html`](file:///Users/jakesmacair/tem/crosswalk_3d.html)
> - [`/Users/jakesmacair/프로젝트 파일/tem/crosswalk_3d.html`](file:///Users/jakesmacair/%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%20%ED%8C%8C%EC%9D%BC/tem/crosswalk_3d.html)

---

## 🌟 구현된 핵심 기능 요약

### 1. 🦯 시각장애인 모드 (신규 추가)
- **완전 암막 상태 (Blackout Environment)**:
  - 태양광 및 기본 배경이 완전히 소등되며, 칠흑 같은 어둠(피치 블랙) 상태를 기본 시각으로 전환합니다.
- **반경 3m 실루엣 렌더링**:
  - `THREE.Fog(0x000000, 0.4, 3.2)`와 플레이어 추적형 `PointLight(radius: 3.6m)`가 결합되어 **유저 반경 3m 이내의 오브제(볼라드, 신호등 기둥, 차량, 단차, 점자블록) 윤곽선과 실루엣만 극적으로 가시화**됩니다.
  - 전용 `#blind-silhouette-overlay` 암막 방사형 비네트로 중심부 시야 집중에 맞춘 감각을 전달합니다.
- **흰지팡이(White Cane) 타격음 & 점자블록 반향음**:
  - 보행 시 발소리 대신 지팡이로 바닥을 두드리는 타격음이 재생됩니다.
  - 일반 보도/아스팔트를 두드릴 때는 둔탁한 타격음(520Hz), **노란 점자블록(유도형/점형) 위를 두드릴 때는 맑고 선명한 고음 반향음(1350Hz)**이 재생됩니다.
  - 점자블록 위에 올라서면 HUD에 `[🟡 점자블록 감지]` 토스트가 활성화되며, 어둠 속에서 점자블록이 은은하게 발광합니다.
- **횡단보도 시각장애인용 음향신호기 사운드**:
  - 횡단보도 14m 이내 접근 시 국내 실제 음향신호기와 동일한 유도음이 자동 송출됩니다.
  - **보행 녹색 신호**: 맑은 뻐꾸기/새소리("뻐~ 꾹~", 1046Hz -> 784Hz).
  - **보행 적색 신호 / 대기**: 부드러운 위치 유도 펄스음(659Hz).

---

### 2. 🛠️ 오브제 근접 멈춤(Freezing) 버그 완전 해결
- **원인 분석**:
  1. 단차(`curbZ`) 통과 검사 조건이 진행 방향을 구분하지 않고 단차 경계 주변에 서 있기만 해도 무차별 통과 시도로 판정하여, 체력 고갈 시 `{ blocked: true, newPos: currentPos }`로 위치를 영구 고정시켜 뒤로 물러나거나 좌우로 비켜서는 조작까지 전면 봉쇄되었습니다.
  2. 볼라드, 신호등 기둥, 표지판, 가로수 등 원통형 오브제 충돌 시 미끄러짐(Slide) 연산이 없어 오브제 표면에 달라붙어 정지하는 현상이 발생했습니다.
- **개선 및 해결책**:
  1. **오브제 탄성 슬라이딩 충돌(Continuous Slide Collision)**: 청크 내 모든 볼라드, 기둥, 가로수에 반경 충돌체를 부여하고, 접근 시 충돌 법선 벡터를 따라 `targetPos`를 외곽으로 밀어내어 **오브제와 맞닿아도 멈춤 없이 둥근 표면을 따라 매끄럽게 옆으로 미끄러지도록** 구현했습니다.
  2. **방향성 단차 검사 & 탈진 시에도 회피 이동 보장**: 보도에서 차도로 내려가거나 차도에서 보도로 올라오는 진입 벡터에 대해서만 단차 극복을 판정하며, 탈진으로 단차를 넘지 못하더라도 **Z축 단차선만 차단하고 X축 좌우 이동 및 뒤로 물러서기(후진)는 100% 허용**하여 멈춤 현상을 완전히 제거했습니다.

---

### 3. 👴 고령자(노인) 모드
- **뿌옇고 침침한 시야 (백내장/노안 시야 필터)**:
  - 노인 모드 활성화 시 3D 환경에 **흐림 필터(`backdrop-filter: blur(3.2px) contrast(0.86)`)**와 **주변부 호박색 색조 저하**가 실시간 적용됩니다.
- **휠체어만큼 취약한 체력 & 완만한 보행**:
  - 기본 보행 속도 `2.2 m/s`, 빠른 보행 `3.2 m/s`.
  - 조금만 걸어도 체력이 빠르게 소모되며, 10cm 단차 보행 시 관절 부담으로 **체력 25% 소모**.
  - 표준 눈높이: 허리가 약간 굽은 고령자 표준 `1.45m`.

---

### 4. ♿ 휠체어 사용자 모드 & 단차 극복 체력 소모
- **단차 극복 시 체력 급감**:
  - 휠체어로 10cm 보도 단차를 넘을 때 **체력이 즉시 50% 급감**하며, 묵직한 덜컹거림 충격(`Camera Jolt`)과 고하중 마찰음이 재생됩니다.
- **탈진 시 단차 극복 차단**:
  - 체력이 소진된 탈진 상태에서는 턱을 넘지 못하며, 후진하거나 평지에서 휴식하여 체력을 재충전해야 합니다.

---

### 5. 🏃 무한 절차적 도로 & 🚗 사륜차 / 🛵 오토바이 교통 시스템
- 전방(`-Z` 방향)으로 진행할수록 **횡단보도 블록**과 **무단횡단 도로 블록**이 실시간 절차적(Procedural)으로 무한 생성됩니다.
- 사륜차(승용차·택시)와 고속 배달 오토바이가 2개 차선에서 각기 다른 속도로 질주하며, 차도 진입 시 클랙슨 경적(11m 이내)과 충돌 판정을 수행합니다.
- `W/A/S/D` 및 **화살표 방향키(↑, ↓, ←, →)** 완벽 호환.

---

## 🗺️ 소스 코드 맵 (Code Map - index.html 총 2,748줄)

```
index.html (총 2,748줄)
├── 1. 문서 헤더 및 외부 라이브러리 (Lines 1 ~ 11)
├── 2. CSS 스타일시트 & 글래스모피즘 디자인 시스템 (Lines 12 ~ 840)
│   ├── 노인 모드 시야 저하 (백내장/노안 흐림) 필터 (Lines 50 ~ 65)
│   ├── 시각장애인 모드 암막 비네트 오버레이 (#blind-silhouette-overlay) (Lines 66 ~ 81)
│   ├── 상단 HUD 및 4가지 모드 배지 (Lines 105 ~ 175)
│   ├── 실시간 체력 게이지 바 (Lines 177 ~ 230)
│   ├── 알림 토스트 6종 (단차경고, 차단, 체력급감, 충돌, 점자감지, 구간안내) (Lines 232 ~ 410)
│   └── 오른쪽 글래스모피즘 컨트롤 패널 & 4모드 2x2 그리드 (Lines 440 ~ 730)
├── 3. HTML DOM 구조 & HUD 인터페이스 (Lines 842 ~ 1130)
│   ├── WebGL 캔버스 & 2종 특수시각 오버레이 (Lines 844 ~ 855)
│   ├── 상단 상태 배너 & 스코어 HUD (Lines 856 ~ 895)
│   ├── 알림 토스트 팝업 6종 (Lines 897 ~ 945)
│   ├── 좌하단 단축키 가이드 (WASD / 방향키 / C 모드전환) (Lines 947 ~ 970)
│   └── 우측 패널 (4개 모드 버튼, 4개 슬라이더, 토글, 신호등) (Lines 972 ~ 1115)
└── 4. JavaScript 3D 시뮬레이션 및 물리 엔진 (Lines 1132 ~ 2748)
    ├── [4-1] Web Audio API 사운드 엔진 (SoundEngine) (Lines 1135 ~ 1380)
    │   └── 지팡이 타격음(playCaneTap), 횡단보도 음향신호기(playAcousticSignal)
    ├── [4-2] 절차적 캔버스 텍스처 생성기 (Lines 1382 ~ 1490)
    ├── [4-3] Three.js 씬, 암막/일반 포그, 3m 실루엣 조명 초기화 (Lines 1495 ~ 1530)
    ├── [4-4] 플레이어 상태 & WASD/방향키 입력 제어 (Lines 1532 ~ 1615)
    ├── [4-5] 사륜차 & 이륜차(오토바이) 3D 모델링 (Lines 1617 ~ 1735)
    ├── [4-6] 무한 청크 생성 & 오브제 충돌체 관리 (RoadChunk) (Lines 1747 ~ 2035)
    ├── [4-7] UI 컨트롤러 & 4개 모드 전환 (setMode) (Lines 2038 ~ 2270)
    ├── [4-8] 오브제 슬라이딩 충돌 & 단차 물리 판정 (checkStepAndRoadConstraints) (Lines 2272 ~ 2435)
    └── [4-9] 메인 게임 & 렌더링 애니메이션 루프 (animate) (Lines 2437 ~ 2745)
```

---

### 📌 주요 코드 영역별 라인 번호 안내

| 섹션 | 라인 범위 | 핵심 내용 |
|---|---|---|
| **시각장애인 암막 필터 CSS** | [L67 ~ L81](file:///Users/jakesmacair/tem/index.html#L67-L81) | `#blind-silhouette-overlay`: 암막 방사형 비네트 그라데이션 |
| **모드 선택기 2x2 그리드 CSS** | [L534 ~ L590](file:///Users/jakesmacair/tem/index.html#L534-L590) | `.mode-selector` 2열 그리드 및 `.active-blind` 스카이블루 테마 |
| **점자블록 감지 토스트 CSS** | [L357 ~ L382](file:///Users/jakesmacair/tem/index.html#L357-L382) | `#tactile-toast`: 점자블록 밟을 때 노란색 글로우 알림 |
| **4개 모드 버튼 DOM** | [L984 ~ L1000](file:///Users/jakesmacair/tem/index.html#L984-L1000) | 보행자(`btn-mode-pedestrian`), 휠체어(`btn-mode-wheelchair`), 노인(`btn-mode-elderly`), 시각장애인(`btn-mode-blind`) |
| **접근성 오디오 신시사이저** | [L1299 ~ L1377](file:///Users/jakesmacair/tem/index.html#L1299-L1377) | `playCaneTap()`: 점자블록(1350Hz) vs 아스팔트(520Hz) 지팡이 소리, `playAcousticSignal()`: 보행녹색 뻐꾸기음 |
| **3m 실루엣 조명 & 암막 포그** | [L1498 ~ L1525](file:///Users/jakesmacair/tem/index.html#L1498-L1525) | `blindFog = THREE.Fog(0x000000, 0.4, 3.2)`, `blindPointLight(radius: 3.6m)` |
| **오브제 충돌체 등록** | [L1757 ~ L1867](file:///Users/jakesmacair/tem/index.html#L1757-L1867) | `RoadChunk.colliders`: 볼라드, 신호등 기둥, 표지판, 가로수 좌표 및 충돌 반경 등록 |
| **4개 모드 통합 전환기** | [L2069 ~ L2115](file:///Users/jakesmacair/tem/index.html#L2069-L2115) | `setMode('blind')`: 암막 포그, 햇빛 소등, 실루엣 조명 점등, 점자블록 발광, 지팡이 보행 속도(3.0m/s) 동기화 |
| **버그 수정 물리 판정** | [L2274 ~ L2350](file:///Users/jakesmacair/tem/index.html#L2274-L2350) | `checkStepAndRoadConstraints()`: **오브제 탄성 슬라이딩 충돌 & 방향성 단차 판정으로 멈춤 현상 원천 차단** |
| **메인 시뮬레이션 루프** | [L2480 ~ L2620](file:///Users/jakesmacair/tem/index.html#L2480-L2620) | 지팡이 타격 주기 동기화, 점자블록 위치 감지, 음향신호기 자동 송출, 실루엣 라이트 위치 추적 |

---

## 🎮 조작법 안내

| 키 | 기능 |
|---|---|
| **W / A / S / D** 또는 **↑ / ← / ↓ / →** | 1인칭 전후좌우 이동 (앞으로 전진하면 무한 생성) |
| **Shift** | 전력 질주 / 빠른 주행 (체력 소모 가속) |
| **마우스 이동** | 1인칭 시선 회전 (화면 클릭 시 마우스 락) |
| **C** | **보행자 ↔ 휠체어 ↔ 노인 ↔ 시각장애인 4개 모드 순환 전환** |
| **R** | 시작 위치 (0m)로 리셋 |
| **ESC** | 마우스 커서 해제 |

---

## 🌐 실행 방법

### 1. 웹 브라우저 온라인 접속 (GitHub Pages)
👉 [https://kyum-yee.github.io/crosswalk-3d-simulator/](https://kyum-yee.github.io/crosswalk-3d-simulator/)

### 2. 로컬 실행
```bash
open ~/tem/index.html
```
또는
```bash
open "/Users/jakesmacair/프로젝트 파일/tem/crosswalk_3d.html"
```
