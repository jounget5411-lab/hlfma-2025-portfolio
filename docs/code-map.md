# 핵심 코드 안내

[프로젝트 소개로 돌아가기](../README.md)

기준 커밋: `48406564072bd34c5d223b7e9fb8b05a2381fa11` · 2025년 9월 20일

아래 링크는 공개 소스의 해당 시점으로 고정되어 있습니다. 상세 개발 기록이 설명하는 다중 경로·신호등 제어·Arduino·시뮬레이션 보관본과의 차이는 [구현 버전과 검증 범위](technical-notes.md)에 정리했습니다.

| 확인할 내용 | 원본 코드 |
|---|---|
| 위치 추정 설정 | [gps_pkg/config](https://github.com/Khusla/HLFMA_2025/tree/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/gps_pkg/config) |
| 위도·경도 CSV와 좌표 변환 | [test_path_node1.py](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/gps_path/gps_path/test_path_node1.py) |
| 방향 보정과 로컬 경로 | [path_planner_node.py](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/decision_making_pkg/decision_making_pkg/path_planner_node.py) |
| 단계 조향과 구동 조건 | [motion_planner_node.py](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/decision_making_pkg/decision_making_pkg/motion_planner_node.py) |
| 카메라 객체 검출 | [yolov5_node.py](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/camera_perception_pkg/camera_perception_pkg/yolov5_node.py) |
| 신호등 상태 메시지 | [traffic_light_detector_node.py](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/camera_perception_pkg/camera_perception_pkg/traffic_light_detector_node.py) |
| LiDAR 장애물 판정 | [lidar_obstacle_detector_node.py](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/lidar_perception_pkg/lidar_perception_pkg/lidar_obstacle_detector_node.py) |
| 명령 시리얼 송신 | [serial_sender_node.py](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/serial_communication_pkg/serial_communication_pkg/serial_sender_node.py) |
| 보정 방향과 경로 시각화 | [gps_path_visualizer_node.py](https://github.com/Khusla/HLFMA_2025/blob/48406564072bd34c5d223b7e9fb8b05a2381fa11/src/debug_pkg/debug_pkg/gps_path_visualizer_node.py) |

공개된 팀 소스는 [KHUsla 원본 저장소](https://github.com/Khusla/HLFMA_2025)에 있습니다. 이 포트폴리오에는 전체 코드·모델·데이터셋을 복제하지 않았으며, 원본의 권리·라이선스 조건은 해당 프로젝트 자료를 따릅니다.
