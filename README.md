<p align="center"><img src="assets/hero.png" alt="GPS 경로에서 차량 제어까지 — HL FMA 2025" width="100%"></p>

# HL FMA 2025 · GPS 경로 기반 자율주행

> 경로 좌표를 차량 기준 목표점으로 바꾸고, 위치별 주행 조건을 조향·구동 명령에 연결한 ROS 2 팀 프로젝트입니다.

**대회:** HL FMA 2025 자율주행 경진대회 · **팀:** KHUsla · **성과:** 장려상

[팀 원본 저장소](https://github.com/Khusla/HLFMA_2025) · [확인한 소스 버전](https://github.com/Khusla/HLFMA_2025/tree/48406564072bd34c5d223b7e9fb8b05a2381fa11)

## 프로젝트 한눈에 보기

이 프로젝트는 미리 준비한 GPS 경로를 따라 주행하기 위해 **위치 추정 → 경로 변환 → 목표점 선택 → 조향·구동 명령**을 ROS 2 노드로 나눈 시스템입니다. 카메라 신호등 인식과 LiDAR 장애물 검출 모듈도 함께 구성되어 있습니다.

핵심은 위도·경도 경로를 차량이 사용할 수 있는 로컬 좌표로 바꾸는 과정입니다. 경로 계획 노드는 GPS 위치와 보정한 방향을 받아 전방 경로를 만들고, 제어 노드는 목표점 방향을 단계별 조향 명령으로 변환합니다. 특정 위치 구간에서는 경사로 구동 조건과 장애물 정지 조건을 적용합니다.

| 구분 | 저장소에서 확인한 내용 |
|---|---|
| 개발 구성 | ROS 2 Humble 대상 설치 스크립트, Python 노드, 사용자 정의 ROS 메시지 |
| 위치·방향 | u-blox GNSS, IMU, 엔코더 오도메트리, `robot_localization` |
| 경로 | CSV 위도·경도 → `FromLL` 좌표 변환 → 등간격 경로 → 차량 기준 전방 경로 |
| 제어 | 목표점 방향 기반 단계 조향, 전륜·후륜 구동 명령, 위치 구간별 조건 처리 |
| 인식 | YOLOv5 신호등 분류, LiDAR 거리 조건 및 연속 검출 |

## 시스템 구성

위치·방향 입력과 경로 계획, 차량 명령이 연결되는 흐름입니다.

<p align="center"><img src="assets/architecture.png" alt="GPS와 EKF 방향을 경로 계획에 연결하고 LiDAR 정지 조건을 차량 명령에 반영하는 구조" width="100%"></p>

카메라 파이프라인은 `Green / Yellow / Red / Left / None` 상태를 발행합니다. 현재 제어 코드에서는 이 상태를 구독·저장하며, 신호등 상태를 조향·구동 명령에 적용하는 분기는 확인되지 않습니다.

## 구현에서 살펴볼 부분

### 1. GPS 경로를 차량의 목표점으로 변환

위도·경도 CSV를 `robot_localization`의 `FromLL` 서비스로 변환하고, 경로 길이에 따라 등간격으로 다시 샘플링합니다. 경로 계획 노드는 GPS 위치와 EKF 방향을 사용해 전방 구간을 선택한 뒤 차량 기준 로컬 좌표로 발행합니다.

- 시작 시 약 1 m 직진하도록 경로를 내보내고, 실제 이동 방향과 IMU 방향의 차이로 yaw 오프셋을 계산합니다.
- 경로 길이를 기준으로 lookahead 구간을 선택합니다.
- 횡방향 이탈이나 경유점 통과 지연이 발생하면 전방 경로에서 기준점을 다시 찾는 로직을 포함합니다.
- 마지막 경유점 근처에서는 빈 경로를 발행해 기본 주행 명령이 정지 상태로 전환되도록 구성합니다.

[좌표 변환·경로 생성 코드](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/gps_path/gps_path/test_path_node1.py) · [경로 계획 코드](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/decision_making_pkg/decision_making_pkg/path_planner_node.py)

### 2. 위치별 주행 조건을 제어 명령에 연결

제어 노드는 전방 목표점의 방향을 `atan2(x, y)`로 계산하고, 이를 좌우 단계별 조향 값으로 바꿉니다. 명령은 `s{조향} f{전륜} r{후륜}` 형식으로 발행되며, 시리얼 노드가 줄바꿈을 붙여 Arduino 인터페이스로 전달합니다.

경사로로 지정한 위치에 진입하면 정해진 시간 동안 전륜·후륜 구동 값을 바꾸는 분기가 있습니다. 장애물 정지는 시작점과 종료점으로 지정한 구간에서 LiDAR 검출을 받아 구동 값을 0으로 설정하는 방식이며, 현재 코드에서는 세션당 한 번 발동하도록 구성되어 있습니다.

[주행 조건·명령 생성 코드](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/decision_making_pkg/decision_making_pkg/motion_planner_node.py) · [시리얼 송신 코드](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/serial_communication_pkg/serial_communication_pkg/serial_sender_node.py)

### 3. 인식 결과를 작은 메시지로 전달

카메라 모듈은 YOLOv5 검출 결과를 사용자 정의 `DetectionArray`로 발행하고, 신호등 모듈은 관심 클래스 중 점수가 가장 높은 결과를 상태 문자열로 변환합니다. LiDAR 모듈은 거리 조건을 만족하는 검출이 연속으로 들어왔는지를 세어 장애물 여부를 발행합니다.

인식·판단·통신 사이의 경계를 ROS 메시지로 분리한 구조를 소스에서 살펴볼 수 있습니다.

[YOLOv5 노드](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/camera_perception_pkg/camera_perception_pkg/yolov5_node.py) · [신호등 상태 변환](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/camera_perception_pkg/camera_perception_pkg/traffic_light_detector_node.py) · [LiDAR 장애물 검출](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/lidar_perception_pkg/lidar_perception_pkg/lidar_obstacle_detector_node.py)

## 코드를 읽는 순서

| 순서 | 디렉터리·파일 | 읽을 내용 |
|---|---|---|
| 1 | `src/gps_pkg/config/`, `src/gps_pkg/launch/fusion.launch.py` | 위치·방향 입력과 좌표계 |
| 2 | `src/gps_path/gps_path/test_path_node1.py` | CSV를 ROS 경로로 바꾸는 과정 |
| 3 | `src/decision_making_pkg/decision_making_pkg/path_planner_node.py` | 방향 보정, lookahead, 로컬 경로 |
| 4 | `src/decision_making_pkg/decision_making_pkg/motion_planner_node.py` | 조향·구동 값과 위치별 조건 |
| 5 | `src/camera_perception_pkg/`, `src/lidar_perception_pkg/` | 인식 결과 생성 |
| 6 | `src/serial_communication_pkg/` | 실제 차량 인터페이스로 명령 전달 |
| 7 | `src/debug_pkg/` | GPS 경로·검출 결과 시각화 노드 |

## 자료 안내

[구현 범위와 실행 참고](docs/technical-notes.md) · [핵심 코드 안내](docs/code-map.md)


이 저장소는 프로젝트의 설계와 구현을 설명하는 포트폴리오입니다. 전체 팀 소스는 [KHUsla 원본 저장소](https://github.com/Khusla/HLFMA_2025)에 있습니다. 위 설명은 명시한 커밋의 코드를 기준으로 작성했으며, 장비 연결·빌드·주행 재현은 별도로 검증해야 합니다.

---

[FPGA 드론 프로젝트](https://github.com/jounget5411-lab/fpga-drone-portfolio) · [국민대 자율주행 프로젝트](https://github.com/jounget5411-lab/kookmin-autonomous-driving-portfolio)
