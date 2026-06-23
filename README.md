# Tomato Robot Arm

공학경진대회 참가를 위해 진행한 토마토 수확 보조 로봇팔 프로젝트입니다. 카메라 이미지에서 토마토를 탐지하고 익은 정도를 판단한 뒤, 수확 대상 토마토의 위치 정보를 로봇팔 제어 흐름으로 연결하는 것을 목표로 했습니다.

## 프로젝트 개요

이 저장소의 현재 코드는 Colab에서 작성한 `tomato.py`를 기반으로 합니다. 주요 내용은 토마토 이미지 데이터셋 탐색, 학습/검증 데이터 분할, annotation 분석, bounding box 시각화, 딥러닝 추론, 라즈베리파이 카메라 촬영 흐름입니다.

최종 목표는 아래와 같습니다.

- 토마토 이미지에서 객체 위치와 익은 정도를 판단
- 수확 대상 토마토의 중심 좌표 계산
- 라즈베리파이 또는 PC에서 추론 실행
- Arduino 기반 로봇팔 제어로 좌표 전달
- 공학경진대회 시연 가능한 자동화 프로토타입 구성

## 개발 동기

농업 현장에서는 고령화와 인력 부족으로 인해 수확 인력이 지속적으로 감소하고 있으며, 이로인해 수확 시기 지연, 생산성 저하 등의 문제가 발생하고 있습니다. 이를 해결하기 위해 딥러닝 기반 방울토마토 자동 수확 로봇팔 시스템을 개발하였습니다. 

## 현재 구현 내용

- XML annotation 기반 토마토 데이터셋 분석
- 이미지/라벨 train-test split
- bounding box 크기, 비율, 객체 수, 중심 좌표 분포 시각화
- 샘플 이미지와 annotation 시각화
- PyTorch 기반 딥러닝 추론 실험
- 웹캠 촬영 흐름 작성
- 향후 로봇팔 제어에 사용할 토마토 중심 좌표 분석

## 시스템 흐름

```text
카메라 이미지
     |
     v
토마토 탐지 / 익은 정도 판단
     |
     v
Bounding box 중심 좌표 계산
     |
     v
Raspberry Pi 또는 PC 제어 스크립트
     |
     v
Arduino / Servo motor 제어
     |
     v
로봇팔 수확 시연
```

## 저장소 파일 구성

| 파일 | 설명 |
| --- | --- |
| `tomato.py` | Colab 기반 데이터 분석, 학습 준비, 추론, 라즈베리파이 카메라 테스트 코드 |
| `README.md` | 프로젝트 소개와 정리 문서 |

## 기술 스택

- Python
- PyTorch / torchvision
- OpenCV
- PIL
- scikit-learn
- matplotlib
- Arduino / servo motor control concept

## 하드웨어 구성

- Raspberry Pi
- USB 카메라 
- Arduino 보드
- Servo motor / 로봇팔 프레임
- 모터 구동용 외부 전원

## 실행 방법

주요 Python 패키지를 설치합니다.

```bash
pip install torch torchvision opencv-python pillow scikit-learn pandas matplotlib tqdm numpy
```

라즈베리파이 카메라 테스트가 필요하면 아래 패키지를 설치합니다.

```bash
sudo apt update
sudo apt install -y python3-picamera2
```

그 다음 `tomato.py` 상단의 데이터셋 경로를 실행 환경에 맞게 수정합니다.

```python
andrewmvd_tomato_detection_path = "/path/to/tomato_dataset"
```

Colab 경로와 라즈베리파이 경로가 주석으로 남아 있으므로, 실행하는 환경에 맞춰 하나만 선택하면 됩니다.

## 데이터셋 구조

스크립트는 아래와 같은 토마토 데이터셋 구조를 가정합니다.

```text
archive/
  images/
    tomato0.png
    tomato1.png
  annotations/
    tomato0.xml
    tomato1.xml
```

XML annotation 파일을 파싱해 bounding box와 객체 라벨을 추출합니다.

## 개선 예정

- Colab 실험 코드를 재사용 가능한 스크립트로 분리
- 라즈베리파이용 추론 스크립트 추가
- 로봇팔 제어를 위한 Arduino serial communication 코드 추가
- 예시 이미지 또는 시연 영상 추가
- 배선도와 servo calibration 기록 추가
- bounding box와 중심 좌표가 표시된 탐지 결과 저장

## 배운 점

- 이미지 데이터셋은 학습 전에 annotation 품질과 bounding box 분포를 확인해야 한다는 점을 배웠습니다.
- 컴퓨터 비전 결과를 실제 로봇팔 제어에 연결하려면 좌표계 변환과 하드웨어 보정이 중요하다는 점을 경험했습니다.
- 공학경진대회 프로젝트는 코드뿐 아니라 문제 정의, 시스템 흐름, 시연 가능성, 개선 계획을 함께 정리해야 설득력이 높아진다는 점을 배웠습니다.
