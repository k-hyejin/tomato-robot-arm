# Tomato Harvesting Robot Arm

공학경진대회 참가를 위해 진행한 토마토 자동 수확 로봇팔 프로젝트입니다. 카메라 영상에서 토마토를 탐지하고 익은 정도를 판단한 뒤, 수확 대상 토마토의 위치 정보를 로봇팔 제어 시스템과 연동하여 자동 수확을 수행하는 것을 목표로 하였습니다.

## 프로젝트 개요

이 저장소는 Python 기반 객체 탐지 코드(`tomato.py`)와 Arduino 기반 로봇팔 제어 코드(`robot_arduino.ino`)로 구성되어 있습니다.

Python에서는 토마토 탐지와 중심 좌표 계산을 수행하고, Arduino에서는 전달받은 좌표를 기반으로 역기구학 계산 및 로봇팔 제어를 수행합니다.

최종 목표는 다음과 같습니다.

* 토마토 객체 위치 및 익은 정도 판단
* 수확 대상 토마토 중심 좌표 계산
* Raspberry Pi 또는 PC에서 실시간 추론 수행
* Arduino 기반 로봇팔 제어
* 자동 수확 프로토타입 구현 및 시연

## 개발 동기

농업 현장에서는 고령화와 인력 부족으로 인해 수확 인력이 지속적으로 감소하고 있으며, 이로 인해 수확 시기 지연과 생산성 저하 문제가 발생하고 있습니다.

본 프로젝트는 이러한 문제를 해결하기 위해 컴퓨터 비전과 임베디드 제어 기술을 활용한 딥러닝 기반 방울토마토 자동 수확 시스템을 개발하고자 시작되었습니다.

## 담당 역할

* 팀장으로서 프로젝트 일정 관리 및 역할 분담     
* 카메라 좌표계와 로봇팔 좌표계 변환
* Raspberry Pi ↔ Arduino Serial 통신 구현
* Arduino 기반 로봇팔 제어 로직 구현
* 4자유도 로봇팔 역기구학(Inverse Kinematics) 적용
* 시스템 통합 및 테스트

## 구현 내용

### AI · Computer Vision

* XML Annotation 기반 토마토 데이터셋 분석
* 이미지/라벨 Train-Test Split
* Bounding Box 크기, 비율, 객체 수 분석
* 중심 좌표 분포 시각화
* 샘플 이미지 및 Annotation 시각화
* PyTorch 기반 딥러닝 추론 실험
* 웹캠 및 Raspberry Pi 카메라 촬영 흐름 구현
* 토마토 중심 좌표 추출

### Arduino · Robot Arm Control

* Raspberry Pi ↔ Arduino Serial 통신 구현
* 토마토 중심 좌표 기반 로봇팔 제어
* 4자유도 로봇팔 역기구학(Inverse Kinematics) 구현
* PCA9685 기반 다중 서보모터 제어
* 서보모터 보간 제어를 통한 부드러운 관절 이동
* 릴레이 기반 흡입 모터 제어
* 카메라 좌표 → 실제 작업 좌표(mm) 변환
* TB6612FNG 기반 이동 플랫폼 제어
* IR 센서를 이용한 라인트래킹 기능 구현
* 수확 완료 후 초기 위치 복귀 및 재탐색 기능 구현

## 시스템 흐름

```text
Camera Image
      │
      ▼
Tomato Detection
(Ripeness Classification)
      │
      ▼
Bounding Box Center Extraction
      │
      ▼
Raspberry Pi / Python
      │
      ▼
Serial Communication
      │
      ▼
Arduino Controller
      │
      ▼
Inverse Kinematics
      │
      ▼
Servo Motor Control
      │
      ▼
Tomato Harvesting
```

## 프로젝트 결과

* 토마토 탐지부터 로봇팔 제어까지의 전체 시스템 흐름 구현
* 객체 탐지 결과를 실제 로봇팔 동작으로 연동
* 중심 좌표 기반 목표 위치 제어 구현
* 흡입 방식 수확 메커니즘 적용
* 공학경진대회 시연 가능한 프로토타입 제작

## 저장소 파일 구성

| 파일 | 설명 |
| --- | --- |
| `tomato.py` | 데이터셋 분석, 전처리, 객체 탐지 및 토마토 중심 좌표 추출 |
| `robot_arduino.ino` | Arduino 기반 로봇팔 제어 코드 (Serial 통신, 역기구학, 서보 제어, 라인트래킹 포함) |
| `.gitignore` | 데이터셋 및 불필요한 캐시 파일을 Git 추적에서 제외하기 위한 설정 |
| `README.md` | 프로젝트 소개 및 시스템 구성 문서 |

## 기술 스택

### Software

* Python
* PyTorch
* OpenCV
* NumPy
* scikit-learn
* Matplotlib
* Arduino IDE

### Hardware

* Raspberry Pi 4
* Arduino UNO
* PCA9685 Servo Driver
* Servo Motor
* Vacuum Pump
* Relay Module
* TB6612FNG Motor Driver
* IR Line Tracking Sensor
* USB Camera

## 실행 방법

필수 패키지를 설치합니다.

```bash
pip install torch torchvision opencv-python pillow scikit-learn pandas matplotlib tqdm numpy
```

라즈베리파이 카메라 사용 시:

```bash
sudo apt update
sudo apt install -y python3-picamera2
```



## Dataset

데이터셋은 용량 문제로 업로드하지 않았습니다. 

데이터셋 구조:

tomato_dataset/
├── 기본 데이터셋/
└── 정리한 데이터셋/
    ├── remapped_tomato/
    ├── yield_label_convert/

## 배운 점

* 이미지 데이터셋은 학습 이전에 Annotation 품질과 Bounding Box 분포를 분석하는 과정이 중요함을 배웠습니다.
* 컴퓨터 비전 결과를 실제 로봇팔 제어와 연결하기 위해 좌표계 변환과 하드웨어 보정이 필수적임을 경험했습니다.
* 프로젝트에서는 코드 구현뿐 아니라 시스템 설계, 하드웨어 통합, 시연 가능성까지 고려해야 함을 배웠습니다.
