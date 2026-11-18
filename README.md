# DA-DAKA | Autonomous Solar Panel Cleaning Drone

### AI Vision · Autonomous Flight · Precision Water Spraying

**DA-DAKA**는 태양광 패널의 위치와 오염을 자동으로 감지하고, 방문 경로를 생성해 목표 위치로 이동한 뒤 물을 선택적으로 분사하는 **자율 청소 드론**입니다.

기체 설계부터 구조해석, Pixhawk 비행제어, 영상 인식 및 세척장치 연동까지 하나의 시스템으로 구현했습니다.

## 01. Final Drone | 완성 기체

**4축 쿼드로터 기반 자율 청소 플랫폼**

| 구분 | 구성 |
|---|---|
| Flight Controller | Pixhawk 4 / PX4 |
| Mission Computer | Raspberry Pi 5 |
| AI Processing | NVIDIA GPU |
| Camera | Raspberry Pi IMX708 |
| Distance Sensor | TF-Luna LiDAR |
| Position & Attitude | Holybro M10 GPS / Compass, IMU / EKF |
| Water Supply | 지상 펌프 + 공급 호스 |
| Spray Actuator | PX4 AUX5 → DRV8876 → NC Solenoid |
| Software | ROS 2, MAVROS, Python, PyTorch, OpenCV |

무거운 물탱크와 펌프를 지상에 분리하고, 호스를 통해 세척수를 공급하도록 설계했습니다. 카메라와 LiDAR를 이용해 목표 위치를 인식하고 드론이 정렬한 뒤 세척수를 분사합니다.

## 02. Autonomous Cleaning | 드론은 어떻게 움직일까?

\`\`\`mermaid
flowchart LR
    A[이륙 / 탐색] --> B[패널 위치 지도]
    B --> C[경로 계획]
    C --> D[목표 패널 접근]
    D --> E[오염 검출 / 정렬]
    E --> F[안전조건 확인]
    F --> G[선택분사]
    G --> H[새 영상으로 재검증]
\`\`\`

**Survey → Map → Route → Approach → Align → Spray → Verify**

먼저 높은 위치에서 패널 영역을 관측하고, 각 패널의 좌표를 계산합니다. 가까운 패널부터 이동하도록 방문 경로를 만든 다음 목표 패널에 접근합니다. 오염 영역이 확인되면 분사 목표점과 노즐 방향을 정렬하고, 안전조건을 검사한 뒤 물을 분사합니다. 마지막으로 새 영상을 촬영해 남은 오염을 확인합니다.

## 03. Mechanical Design & ANSYS | 기체 설계

기체 암에 작용하는 모터 하중과 응력 분포를 ANSYS로 확인하고, 구조와 장착부 형상을 검토했습니다. 기체 구조뿐 아니라 카메라·LiDAR·분사 노즐, 지상 급수 호스의 간섭도 고려했습니다.

**개발 과정**

- 초기 기체 및 모터 암 형상 설계
- ANSYS로 모터 체결부 주변 응력 집중 확인
- 기체 암 및 장착 구조 개선
- 전장 부품과 세척 장치의 실제 기체 통합

구조해석 화면에서는 **모터 체결부 주변 최대 응력 345 MPa**가 표시되며, 특정 위치의 응력 집중이 나타납니다.

## 04. Pixhawk & PID | 호버링 제어

Pixhawk 4 / PX4를 비행제어기로 사용하고 QGroundControl에서 비행 상태를 모니터링했습니다.

### Actuators & Motor Configuration

4개의 모터와 Actuator 출력을 설정하고, 모터 배열과 구동 출력을 확인했습니다.

### PID Tuning

PID 튜닝 화면에서 목표값과 실제 응답을 비교하고, 비행제어 파라미터를 조정했습니다. PX4 / Gazebo 환경에서는 기체 안정화 및 분사반동에 따른 제어 응답도 점검했습니다.

### LiDAR Distance Control

TF-Luna LiDAR의 수신값을 **checksum 검사 → Median(5) → Moving Average(5)** 순으로 처리합니다. 필터링한 데이터를 외부 거리 PID에 사용하여 패널과 **1.0m**의 작업거리를 유지하도록 구성했습니다.

## 05. Autonomous Mapping | 패널 좌표와 경로 생성

### Capture-Sync

드론이 움직이면서 촬영한 영상의 패널 위치를 계산할 때, AI 결과의 도착시각이 아니라 **영상 촬영시각**에 맞는 기체 Pose와 LiDAR 거리값을 사용합니다.

### Panel Map & Route Planning

1. 영상에서 태양광 패널을 검출합니다.
2. 영상 중심좌표를 기체 기준 위치좌표로 변환합니다.
3. 중복 관측을 합쳐 **Panel Map**을 생성합니다.
4. **Nearest-Neighbour(NN)** 로 가까운 패널부터 초기 방문 순서를 계산합니다.
5. **2-opt**로 경로를 개선한 뒤 PX4에 목표 위치를 전달합니다.

## 06. AI Detection & Precision Spraying | 오염 인식부터 분사까지

### AI가 오염을 찾는 방법

| 처리 단계 | 사용 기술 |
|---|---|
| 패널 찾기 | Panel V4 / Faster R-CNN MobileNetV3-Large FPN |
| 패널 영역 분리 | ROI 추출 |
| 오염 영역 찾기 | Dirt V3 / LR-ASPP MobileNetV3-Large |
| 오염 위치 계산 | Segmentation Mask + Centroid |
| 노즐 조준 | Camera–Nozzle Offset + Vision / Yaw |

오염 mask에서 중심좌표를 계산하고, 카메라와 노즐 사이의 장착 오프셋을 반영해 실제 분사 위치를 맞춥니다.

### 분사 안전조건(Safety Gate)

오염을 찾더라도 바로 물이 나오지는 않습니다. LiDAR 거리, 기체 자세와 속도, 영상·Yaw 정렬, AI 결과 최신성, 통신 및 구동상태가 모두 유효할 때만 분사를 허용합니다.

**분사 제어:** MAVROS CommandLong → PX4 Camera Trigger → AUX5 → DRV8876 → NC Solenoid Valve

1회 분사는 **3초**로 제한되며, 이상 조건에서는 밸브를 즉시 차단하도록 구성했습니다.

### 분사반동 보상(Feed-forward)

노즐 분사 시 발생하는 반동을 예상해 LiDAR 거리 Feedback에 Feed-forward 속도명령을 더합니다. 분사 종료 후에는 Feed-forward를 해제하고 정상 거리제어를 계속합니다.

### Fresh Frame Verification

분사 후 새 영상을 다시 촬영하고 Dirt V3로 오염이 남았는지 검증합니다. 필요하면 재정렬·재분사한 뒤 다음 패널로 이동합니다.

## 07. AI Performance | 검출 성능

| 평가 지표 | 결과 |
|---|---:|
| Panel V4 mAP@0.50:0.95 | **0.921** |
| Dirt V3 Dirty Recall | **100%** |
| Pixel IoU | **80.77%** |
| Mean Dirt Centroid Error | **1.57px** |

## 08. How It Works | 실제 코드의 동작 원리

각 기능의 구체적인 데이터 흐름과 실제 설정을 [**핵심 코드 이해하기**](docs/code-guide.md)에 정리했습니다.

**FSM 임무제어 → Capture-Sync → NN + 2-opt → AI 최신성 검사 → TF-Luna 거리 PID → Safety Gate → 3초 분사 → Fresh Verify**

---

**DA-DAKA — AI Vision & Fluid Reaction Control-Based Solar Panel Cleaning Drone**
