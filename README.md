# DA-DAKA
### Autonomous Solar Panel Cleaning Drone
**AI Vision · Autonomous Flight · Precision Spraying · Structural Design**

태양광 패널의 위치를 자동으로 파악하고, 방문 경로를 생성한 뒤 국부 오염에 선택적으로 세척수를 분사하는 자율 청소 드론 프로젝트입니다. 영상 인식, 비행제어, 거리 센서, 분사장치를 하나의 임무 흐름으로 통합했습니다.

## 01. Final Drone | 완성 기체

DA-DAKA는 **4축 쿼드로터** 기반의 태양광 패널 청소 드론입니다. 무거운 물탱크와 펌프는 지상에 배치하고, 호스를 통해 기체에 세척수를 공급합니다. 기체는 카메라와 LiDAR를 이용해 대상 패널을 찾고, 목표 위치에 접근해 정렬한 후 물을 분사합니다.

| System | Components |
|---|---|
| Flight controller | Pixhawk 4 / PX4 Autopilot |
| Mission computer | Raspberry Pi 5 |
| AI processing | NVIDIA GPU inference system |
| Vision | Raspberry Pi IMX708 camera |
| Distance sensing | TF-Luna LiDAR |
| Position & attitude | Holybro M10 GPS / Compass, Pixhawk IMU / EKF |
| Propulsion & power | Brushless motors ×4, ESC ×4, LiPo battery, PM07 |
| Water delivery | Ground tank / pump → tethered hose |
| Spray actuator | Pixhawk AUX5 → DRV8876 → NC solenoid valve → nozzle |

## 02. Demonstration | 자율 청소 임무

**SURVEY → MAP → ROUTE → APPROACH → ALIGN → SPRAY → VERIFY → RTL / LAND**

드론은 전체 작업 구역을 탐색하고 패널의 위치를 지도화합니다. 생성된 경로에 따라 목표 패널 상공으로 이동한 뒤, LiDAR와 영상으로 작업거리 및 분사 위치를 정렬합니다. 오염이 검출되고 분사 안전조건을 만족하면 세척수를 분사하고, 이후 새로 촬영한 영상으로 결과를 재확인합니다.

## 03. Airframe Design & ANSYS | 기체 구조 설계

기체의 구조적 안정성을 검토하기 위해 ANSYS를 활용한 **프레임 암 강도 분석**을 수행하고, 모터별 추력 테스트를 함께 진행했습니다. 모터의 하중이 전달되는 부위와 구조 형상에 따른 응력 분포를 확인하여 기체 설계·제작 과정에 활용했습니다.

기체 구조 검토뿐 아니라 호스와 분사장치, 카메라·LiDAR, 전원 계통이 서로 간섭하지 않도록 설계했습니다. 특히 세척수 공급 계통을 지상에 분리해 비행체의 탑재 부담을 줄이는 구성을 적용했습니다.

## 04. Pixhawk & PID Tuning | 비행제어 안정화

### 4.1 Pixhawk 및 QGroundControl 연동

Pixhawk 4 / PX4를 기체의 비행제어기로 사용하고, **QGroundControl**을 통해 실기체 제어와 상태 모니터링을 수행했습니다. Raspberry Pi 5에서 생성한 임무 명령은 MAVLink / MAVROS를 통해 비행제어 시스템에 전달됩니다.

### 4.2 PID 응답 관찰 및 튜닝

실기체의 **PID 응답 그래프**, **PID 파라미터 조정 화면**, **네트워크·프레임 손실 상태**를 관찰하며 비행제어 동작을 점검했습니다.

PX4 / Gazebo 환경에서도 기체 안정화 PID 계수와 분사반동에 대한 제어 응답을 시뮬레이션했습니다.

### 4.3 LiDAR 기반 작업거리 제어

TF-Luna의 수신 데이터에서 프레임 checksum, 거리 범위와 신호 유효성을 검사합니다. 이후 **Median(5) → Moving Average(5)** 필터를 거친 거리값을 외부 거리 PID 제어에 사용합니다.

목표 작업거리는 **1.0 m**이며, 거리 오차에 따른 속도명령에 제한을 두어 갑작스러운 제어입력을 억제했습니다.

## 05. Autonomous Survey & Path Planning | 패널 인식과 경로계획

### 5.1 Capture-Sync 기반 Panel Map

고고도 Survey 단계에서 카메라가 촬영한 영상으로 태양광 패널을 검출합니다. 영상의 패널 중심 픽셀좌표와 촬영시각에 동기화된 **기체 Pose 및 LiDAR 정보**를 이용해 패널 관측값을 Local 좌표계로 변환합니다.

서로 다른 프레임에서 반복 검출된 패널 좌표는 중복 제거 및 클러스터링 과정을 거쳐 **Panel Map**으로 통합합니다.

### 5.2 Nearest-Neighbour + 2-opt

Panel Map을 기반으로 현장에서 방문할 패널 순서를 동적으로 생성합니다.

1. **Nearest-Neighbour:** 현재 위치에서 가까운 패널을 순차적으로 선택해 초기 방문 경로 생성
2. **2-opt:** 경로 구간의 방문 순서를 재배열해 불필요한 교차와 이동거리 개선
3. **PX4 / MAVROS:** 선택한 목표 패널의 위치를 비행제어 시스템에 전달

영상 수신 시점이 아닌 **원래 촬영 timestamp**를 기준으로 제어 데이터를 연결하여 이동 중에도 일관된 패널 위치를 계산하도록 구성했습니다.

## 06. AI Detection & Precision Spraying | 오염 인식과 선택분사

### 6.1 Two-Stage Vision

| Stage | Model | Function |
|---|---|---|
| Panel V4 | Faster R-CNN MobileNetV3-Large FPN | 태양광 패널 검출 및 bounding box 생성 |
| Dirt V3 | LR-ASPP MobileNetV3-Large | 패널 ROI의 국부 오염 segmentation |

검출한 패널 영역을 ROI로 추출하고 **384 × 640** 크기로 전처리한 다음 오염 영역을 분할합니다.

Dirt V3의 확률맵에 **threshold 0.99725**를 적용하여 Binary Mask를 생성하고, 최소 component 면적 **8 px**, 최소 면적비 **0.0001**을 기준으로 작은 오검출을 필터링합니다. 최종 오염 영역의 **Centroid**를 계산하고 카메라–노즐 오프셋을 반영해 실제 분사 목표점을 산출합니다.

### 6.2 Safety Gate 및 분사 제어

오염 검출만으로 곧바로 물을 분사하지 않습니다. **LiDAR 작업거리, Vision / Yaw 정렬, 자세·속도 안정성, AI 결과 최신성, 통신 및 구동장치 상태**가 모두 유효할 때만 분사를 허용합니다.

**분사 명령 전달:** Raspberry Pi 5 → MAVROS CommandLong → PX4 Camera Trigger → AUX5 → DRV8876 → NC 솔레노이드 밸브

1회 분사는 **3초 제한 출력**으로 제어하며, 조건 이탈 시 즉시 밸브를 차단하고 HOLD 또는 재정렬 상태로 전환하는 **Fail-Closed** 구조를 적용했습니다.

### 6.3 분사반동 Feed-forward 보상

물 분사로 발생하는 반동을 고려해 **Feed-forward 속도명령**을 LiDAR 거리 Feedback과 결합했습니다. 분사 시작 시 보상을 적용하고, 종료 후에는 보상명령을 해제하여 거리 Feedback이 잔여 오차를 보정합니다.

| Parameter | Value |
|---|---:|
| Estimated spray reaction | 0.437 N |
| Disturbance acceleration | 0.156 m/s² |
| Feed-forward command | −0.031 m/s |
| FF output limit | 0.20 m/s |
| Vertical speed limit | 0.25 m/s |

### 6.4 Fresh Frame Verification

분사 후 새로운 영상을 촬영하여 오염을 다시 검출합니다. 잔류 오염이 있으면 재정렬 및 재시도를 수행하고, 이상 상태에서는 분사를 중지하고 복귀·착륙 절차로 전환합니다.

## 07. Vision Performance | AI 인식 성능

| Evaluation | Result |
|---|---:|
| Panel V4 mAP@0.50:0.95 | **0.921** |
| Dirt V3 Dirty Recall | **100%** |
| Pixel IoU | **80.77%** |
| Mean dirt centroid error | **1.57 px** |

## 08. Development Stack

**Mechanical & Flight:** ANSYS · Pixhawk 4 · PX4 Autopilot · QGroundControl · PX4/Gazebo

**Autonomy & Vision:** Python · ROS 2 · MAVROS · PyTorch · OpenCV · Panel V4 · Dirt V3

**Hardware:** Raspberry Pi 5 · NVIDIA GPU · IMX708 · TF-Luna LiDAR · PM07 · DRV8876 · NC Solenoid Valve
