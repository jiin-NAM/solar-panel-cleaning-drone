# Original source-code evidence index

이 문서는 **「팀 다닦아-제작설계서.pdf」의 30~33쪽**에 실제로 수록된 코드 캡처 및 설명만을 안내합니다.

> **Status:** 코드 이미지 및 실행 가능한 원본 소스 파일은 아직 GitHub에 첨부되지 않았습니다. 이 저장소에 존재하지 않는 코드를 '구현 코드'로 작성하거나, 별도로 생성한 예제를 원본으로 표시하지 않습니다.

| 설계서 페이지 | 실제 수록된 기능 | 설계서에 직접 등장하는 관련 식별자·설정 |
|---|---|---|
| 30 좌측 | FSM 단계별 임무 상태 관리 | `PRECISION_ALIGN → SPRAY_CONFIRM → SPRAY` |
| 30 우측 | 촬영시각 동기화(Capture-Sync)와 Panel Map | `capture_sync_tolerance_s=0.15`, `input_history_s=2.0` |
| 31 좌측 | 동적 방문경로 NN + 2-opt | Nearest-Neighbour, 2-opt |
| 31 우측 | AI 결과 ID·촬영시각·최신성 관리 | `PerceptionResult.msg`, `source_id`, `session_id`, `frame_id`, `sequence`, `result_age_s` |
| 32 좌측 | TF-Luna 신뢰성 검사와 거리 PID | `0x59 0x59`, Median(5), Moving Average(5), `/distance/raw`, `/distance/filtered` |
| 32 우측 | Fail-Closed 분사 인터록 및 PX4 Timed Output | `spray_actuator.py`, `spray_controller.yaml`, `/spray/trigger`, `output_enabled: false`, `pulse_duration_s: 3.0`, `maximum_pulse_s: 3.0` |
| 33 좌측 | 분사반동 Feed-forward 보상 | LiDAR Feedback + FF, 출력 제한 |
| 33 우측 | 주/야간 카메라 노출 프로파일 | night: `shutter_us: 50000`, `gain: 32.0`; day 프로파일 |

## 실제 코드 추가 기준

- 우선 제작설계서에 실린 **원본 코드 화면 그대로** 이미지로 수록합니다.
- 실행 가능한 소스는 실제 소스파일의 내용과 캡처를 대조한 후에만 추가합니다.
- 코드 파일명·기능·설정값을 임의 생성하거나, 동작을 구현했다고 주장하지 않습니다.
- 설계서에 없는 새 알고리즘이나 타 프로젝트 코드를 혼합하지 않습니다.

**기준 문서:** 팀 다닦아, *SW개발/HW제작 설계서* (2026.09.08), pp. 30–33.
