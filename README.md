# STM32F411_Project

NUCLEO-F411RE 보드 학습 정리

## 목차

- [00. 개발 환경 설정 (SETUP)](./00_SETUP_DOC) : Hardware 블록도, RCC, Clock, 프로젝트 생성
- [01. GPIO](./01_GPIO_DOC) : GPIO 레지스터 맵, 디버거로 레지스터 확인, MODER / ODR / BSRR, PA5 · PA6 실습
- [02. HC-SR04](./02_HC-SR04_DOC) : 초음파 센서 동작 원리, TIM1 µs 딜레이, TRIG 신호 오실로스코프 측정

## 프로젝트 구조

```
STM32F411_Project/
├── 00_SETUP_DOC/          개발 환경 설정 문서
├── 01_GPIO_DOC/           GPIO 문서
├── 02_HC-SR04_DOC/        HC-SR04 초음파 센서 문서
├── Core/                  사용자 코드 (main.c 등)
├── Drivers/               HAL / CMSIS
├── STM32CubeIDE/          CubeIDE 프로젝트 파일
└── STM32F411_Project.ioc  CubeMX 설정
```
