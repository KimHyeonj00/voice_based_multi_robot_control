# voice_based_multi_robot_control
음성 인식 기반 다중 로봇 제어 시스템 - 작업자 추종 로봇 구현

<br>

## 1. 프로젝트 개요
* **프로젝트명:** Vocal-Maestro Robot System (음성 명령 다중로봇 제어 시스템)
* **팀명:** 삼위일체
* **비전(Vision):** 작업자의 이동 최소화(Minimize Movement), 인프라 변경 불필요(No Infra Change), 유연성(Flexibility), 확장성(Expansion).
* **핵심 미션(Mission):** 사람 추종(Follow People), 음성 인식(Voice Recognition), 군집 주행(Swarm System), 상태 및 데이터베이스 디스플레이(Status & Database Display).
* **사용 장비(Hardware):** Turtlebot3 Burger (4대), Intel RealSense D435, Microphone SF-555B, Speaker HK-5007, Display.

본 프로젝트는 사용자의 음성 명령을 인식하여 여러 대의 로봇을 제어하는 다중 로봇 제어 시스템입니다.  
음성 명령을 텍스트 또는 제어 명령으로 변환한 뒤, 명령 대상 로봇과 동작을 구분하여 각 로봇에 제어 신호를 전달하는 구조로 구현하였습니다.

단순히 음성 인식 기능을 구현하는 것에 그치지 않고, 여러 로봇을 명령 대상에 따라 분리 제어할 수 있도록 시스템 흐름을 구성하는 데 중점을 두었습니다.

---

### 목표

- 음성 명령을 기반으로 로봇 제어 명령 생성
- 명령 대상 로봇과 동작 명령 분리
- 여러 로봇에 대한 개별 제어 구조 구현
- 음성 인식 결과와 로봇 제어 로직 간 연동
- 실제 로봇 또는 시뮬레이션 환경에서 제어 동작 확인

---

## 2. 사용 기술 스택 (Tech Stack)
* **Vision / Follow People:** YOLOv5nu, OpenCV, Intel RealSense SDK
* **Voice Recognition:** OpenWakeWord, Whisper (STT), Rasa, Typecast TTS
* **Swarm System / Navigation:** ROS2, Domain Bridge, Nav2, Cartographer
* **Display / DB:** Qt5, Maria DB

---

## 3. 👨‍💻 담당 개발 파트: 마스터 로봇 (사람 추종 시스템)
마스터 로봇은 작업자를 인식하고 안전 거리를 유지하며 따라다니는 핵심 역할을 수행합니다. `turtlebot_follow_and_pose_tilted_final.py` 노드를 개발하여 딥러닝 기반 객체 인식과 깊이(Depth) 데이터를 융합한 추종 로직을 구현했습니다.


---

### 🎯 핵심 구현 기능
* **YOLOv5nu 기반 실시간 사람 탐지:** * RealSense 카메라의 Color 프레임을 받아 YOLO 모델로 사람 객체를 탐지합니다.
  * 오인식을 방지하기 위해 Bounding Box의 면적(Area > 2000)과 비율(h/w > 0.8)을 계산하여 가장 적합한 작업자를 타겟으로 필터링합니다.
* **45도 틸트(Tilt) 카메라 좌표계 보정:** * 카메라가 로봇 하단에서 45도 상향(`self.tilt_angle`)으로 설치된 하드웨어적 특성을 소프트웨어로 보정했습니다.
  * `rs2_deproject_pixel_to_point`로 얻은 3D 좌표에 삼각함수 회전 변환을 적용하여, 지면 기준 로봇과 사람 간의 **실제 수평 전방 거리(`ros_x`)**를 정확히 산출합니다.
* **동적 제어 로직 (Twist/cmd_vel):**
  * YOLO 모델로 탐지한 객체의 이미지상 좌표를 추출하고, 객체의 좌표가 중심에 오도록 회전합니다.
  * **거리 제어 (Linear):** 타겟과의 거리를 항상 1.0m로 유지하도록 전진 속도를 제어하며, 급발진을 막기 위해 최대 속도를 0.2m/s로 제한했습니다.

## 4. 🛠 트러블슈팅 및 성능 최적화 (Discussion)

**1. 라즈베리 파이 리소스 한계 극복 (카메라 문제 해결)**
* **문제:** RealSense D435 Wrapper 노드 실행 시 고해상도 이미지와 전체 Depth 데이터를 동시에 발행할 때, 리소스가 제한된 라즈베리 파이 환경에서 CPU 및 네트워크 대역폭 병목 현상이 발생했습니다.
* **해결:** 전체 Raw Data를 구독하는 대신, 객체 인식 모델(YOLO)에서 추출된 영역의 중심 픽셀 좌표에 해당하는 깊이(Depth) 정보만 선택적으로 추출하도록 로직을 최적화하여 연산량을 최소화했습니다.

### 원본 레포지토리
https://github.com/sanghyeon-222/Vocal_Maestro_Robot_System
