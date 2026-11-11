# GPIO

   - [Datasheet - STM32F411xC STM32F411xE : Arm Cortex-M4 32b MCU+FPU, 125 DMIPS, 512KB Flash, 128KB RAM, USB OTG FS, 11 TIMs,1 ADC,13 comm. interfaces](./doc/DS_stm32f411ce.pdf)
   - [RM0383 Reference Manual : STM32F411xC/E advanced Arm-based 32-bit MCUs](./doc/RM0383_stm32f411xce.pdf)

## Table of Contents
- [1. Architecture](#1architecture)
- [2. CLOCK](#2-clock)
- [3. MemoryMap](#3memorymap)
- [4. GPIO](#4-gpio)
- [5. 실습](#5-실습)

## 1.Architecture

<details>
<summary>F411</summary>
   
#### 메모리 및 버스 아키텍처

##### 시스템 아키텍처

* STM32F411xC/E의 메인 시스템은 32비트 다중 계층(multilayer) AHB 버스 매트릭스로 구성되어 있으며, 이를 통해 다음 요소들이 상호 연결됩니다. <br>
* 6개의 마스터(Master):
   * FPU를 포함한 Cortex-M4 코어의 I-버스(I-bus), D-버스(D-bus) 및 S-버스(S-bus)
   * DMA1 메모리 버스
   * DMA2 메모리 버스
   * DMA2 주변장치(peripheral) 버스
* 5개의 슬레이브(Slave):
   * 내부 플래시 메모리 ICode 버스
   * 내부 플래시 메모리 DCode 버스
   * 메인 내부 SRAM
   * AHB-APB 브리지 및 APB 주변장치를 포함한 AHB1 주변장치
   * AHB2 주변장치

버스 매트릭스는 마스터가 슬레이브에 접근할 수 있도록 경로를 제공하며, 여러 고속 주변장치가 동시에 작동할 때에도 동시 접근을 가능하게 하여 효율적인 동작을 지원합니다. 이러한 아키텍처는 그림 1에 나타나 있습니다.

<img src="./img/arm-arch-2.png"> <br>

#### 2.1.1 I-bus(명령 버스)
이 버스는 FPU가 포함된 Cortex®-M4 코어의 명령 버스(Instruction bus)를 BusMatrix에 연결합니다. 코어는 이 버스를 사용하여 명령을 가져옵니다(fetch). 이 버스의 대상은 코드가 저장된 메모리(내부 플래시 메모리/SRAM)입니다.

#### 2.1.2 D-bus(데이터 버스)
이 버스는 FPU가 포함된 Cortex®-M4의 데이터 버스를 BusMatrix에 연결합니다. 코어는 이 버스를 사용하여 리터럴(상수) 값을 로드하거나 디버깅을 위한 접근을 수행합니다. 이 버스의 대상은 코드나 데이터가 저장된 메모리(내부 플래시 메모리/SRAM)입니다.

#### 2.1.3 S-bus(시스템 버스)
이 버스는 FPU가 포함된 Cortex®-M4 코어의 시스템 버스를 BusMatrix에 연결합니다. 이 버스는 주변 장치(peripheral)나 SRAM에 위치한 데이터에 접근하는 데 사용됩니다. 명령을 가져오는(fetch) 용도로도 사용할 수 있지만, ICode 버스를 사용하는 것보다 효율성은 떨어집니다. 이 버스의 대상은 내부 SRAM1, AHB1 주변 장치(APB 주변 장치 포함), 그리고 AHB2 주변 장치입니다.

#### 2.1.4 DMA 메모리 버스
이 버스는 DMA 메모리 버스 마스터 인터페이스를 BusMatrix에 연결합니다. DMA가 메모리와 데이터를 주고받는(전송하는) 작업을 수행할 때 사용됩니다. 이 버스의 대상은 데이터 메모리(내부 플래시 메모리, 내부 SRAM)이며, S4의 경우 AHB1/AHB2 주변 장치(APB 주변 장치 포함)도 대상에 포함됩니다.

#### 2.1.5 DMA 주변 장치 버스
이 버스는 DMA 주변 장치 마스터 버스 인터페이스를 BusMatrix에 연결합니다. DMA가 AHB 주변 장치에 접근하거나 메모리 간(memory-to-memory) 데이터 전송을 수행할 때 사용됩니다. 이 버스의 대상은 AHB 및 APB 주변 장치와 데이터 메모리(플래시 메모리 및 내부 SRAM)입니다.

#### 2.1.6 BusMatrix(버스 매트릭스)
BusMatrix는 마스터 간의 접근 중재(arbitration)를 관리합니다. 중재 방식으로는 라운드 로빈(round-robin) 알고리즘이 사용됩니다. 

#### 2.1.7 AHB/APB 브리지(APB)
두 개의 AHB/APB 브리지인 APB1과 APB2는 AHB 버스와 두 개의 APB 버스 간에 완전 동기식 연결을 제공하며, 이를 통해 주변 장치(peripheral)의 동작 주파수를 유연하게 선택할 수 있습니다.
APB1 및 APB2의 최대 동작 주파수에 대한 자세한 내용은 해당 디바이스의 데이터시트를 참조하고, AHB 및 APB 주변 장치의 주소 매핑에 대해서는 표 1을 참조하십시오.
디바이스가 리셋될 때마다 모든 주변 장치 클록은 비활성화됩니다(SRAM 및 플래시 메모리 인터페이스는 예외). 주변 장치를 사용하려면 먼저 RCC_AHBxENR 또는 RCC_APBxENR 레지스터를 통해 해당 주변 장치의 클록을 활성화해야 합니다.

참고: APB 레지스터에 대해 16비트 또는 8비트 액세스를 수행하면, 해당 액세스는 32비트 액세스로 변환됩니다. 즉, 브리지가 16비트 또는 8비트 데이터를 복제하여 32비트 벡터를 구성합니다.
   
<img src="./img/arm-arch-1.png"> <br>

</details>

## 2. CLOCK

<details>
<summary>F411</summary>

<img src="./img/411-clk1.png"> <br>
<img src="./img/411-clk2.png"> <br>

</details>

## 3.MemoryMap

<details>

<summary>F411</summary>

### 4.2 메모리 구성

프로그램 메모리, 데이터 메모리, 레지스터 및 I/O 포트는 동일한 4GB 선형 주소 공간 내에 구성됩니다.
메모리 내의 바이트는 리틀 엔디언(little endian) 형식으로 저장됩니다. 워드(word)에서 가장 낮은 주소의 바이트는 최하위 바이트(LSB, least significant byte)로 간주되며, 가장 높은 주소의 바이트는 최상위 바이트(MSB, most significant byte)로 간주됩니다.
주변 장치(peripheral) 레지스터의 상세 매핑에 대해서는 관련 장(chapter)을 참조하십시오. 주소 지정이 가능한 메모리 공간은 각각 512MB 크기의 8개 주요 블록으로 나뉩니다.
온칩(on-chip) 메모리나 주변 장치에 할당되지 않은 모든 메모리 영역은 '예약됨(Reserved)'으로 간주됩니다. 제품 데이터시트에 있는 메모리 맵 그림을 참조하십시오.

### 4.3 메모리 맵

메모리 맵에 대한 전체적인 구성도는 사용 중인 장치의 데이터시트를 참조하십시오. 표 1은 STM32F411xC/E 장치에서 사용할 수 있는 주변 장치들의 경계 주소를 나타냅니다.

#### 2.3.1 내장 SRAM

STM32F411xC/E 디바이스는 128 K바이트의 시스템 SRAM을 갖추고 있습니다.
내장 SRAM은 바이트, 하프 워드(16비트) 또는 풀 워드(32비트) 단위로 액세스할 수 있습니다. 읽기 및 쓰기 작업은 CPU 속도로 수행되며, 대기 상태(wait state)는 0입니다.
SRAM에서 부팅하도록 설정되었거나 물리적 리맵(physical remap)이 선택된 경우, CPU는 시스템 버스 또는 I-Code/D-Code 버스를 통해 내장 SRAM에 액세스할 수 있습니다.
(SYSCFG 컨트롤러 내의 7.2.1절: SYSCFG 메모리 리맵 레지스터(SYSCFG_MEMRMP) 참조). SRAM에서 코드를 실행할 때 최대 성능을 얻으려면 물리적 리맵을 선택해야 합니다(부팅 시 또는 소프트웨어를 통해 설정 가능).

#### 2.3.2  플래시 메모리 개요
플래시 메모리 인터페이스는 CPU의 AHB I-Code 및 D-Code를 통한 플래시 메모리 액세스를 관리합니다. 이 인터페이스는 플래시 메모리의 삭제 및 프로그래밍 작업, 그리고 읽기/쓰기 보호 메커니즘을 구현합니다. 또한 명령어 프리페치(prefetch) 및 캐시 라인 시스템을 통해 코드 실행 속도를 높입니다.
플래시 메모리는 다음과 같이 구성됩니다:
• 섹터로 나뉜 메인 메모리 블록.
• 시스템 메모리 부팅 모드에서 디바이스가 부팅되는 시스템 메모리 영역.
• 사용자 데이터를 위한 512바이트의 OTP(일회성 프로그래밍 가능) 메모리.
• 읽기/쓰기 보호, BOR(Brown-Out Reset) 레벨, 워치독(소프트웨어/하드웨어), 그리고 Standby 또는 Stop 모드에서의 리셋 동작 등을 설정하기 위한 옵션 바이트(Option bytes).
자세한 내용은 3절: 내장 플래시 메모리 인터페이스를 참조하십시오.

#### 2.3.3 비트 밴딩(Bit banding)
FPU가 포함된 Cortex®-M4의 메모리 맵에는 두 개의 비트 밴드 영역이 있습니다. 이 영역들은 '앨리어스(alias) 영역'의 각 워드를 '비트 밴드 영역'의 특정 비트에 매핑합니다. 앨리어스 영역의 워드에 데이터를 쓰는 것은 비트 밴드 영역 내의 대상 비트에 대해 '읽기-수정-쓰기(read-modify-write)' 작업을 수행하는 것과 동일한 효과를 냅니다. STM32F4xx 디바이스에서는 주변 장치 레지스터와 SRAM 모두가 비트 밴드(bit-band) 영역에 매핑되어, 단일 비트 단위의 읽기 및 쓰기 작업이 가능합니다. 이러한 작업은 FPU를 사용하는 Cortex-M4를 통해서만 수행할 수 있으며, 다른 버스 마스터(예: DMA)를 통해서는 수행할 수 없습니다.
매핑 공식을 사용하면 에일리어스(alias) 영역의 각 워드를 비트 밴드 영역의 특정 비트에 대응시킬 수 있습니다. 매핑 공식은 다음과 같습니다.
bit_word_addr = bit_band_base + (byte_offset x 32) + (bit_number × 4)
여기서 각 항목의 의미는 다음과 같습니다.
– bit_word_addr: 대상 비트에 매핑되는 에일리어스 메모리 영역 내 워드의 주소
– bit_band_base: 에일리어스 영역의 시작 주소
– byte_offset: 대상 비트를 포함하고 있는 비트 밴드 영역 내 바이트의 번호(오프셋)
– bit_number: 대상 비트의 위치(0~7)

**예시**
다음 예시는 SRAM 주소 0x20000300에 위치한 바이트의 2번 비트를 별칭 영역(alias region)에 매핑하는 방법을 보여줍니다.
0x22006008 = 0x22000000 + (0x300*32) + (2*4)
주소 0x22006008에 데이터를 쓰는 것은 SRAM 주소 0x20000300에 있는 바이트의 2번 비트에 대해 '읽기-수정-쓰기(read-modify-write)' 작업을 수행하는 것과 동일한 효과를 냅니다.
주소 0x22006008을 읽으면 SRAM 주소 0x20000300에 있는 바이트의 2번 비트 값(0x01 또는 0x00)이 반환됩니다(0x01: 비트 설정됨, 0x00: 비트 해제됨).
비트 밴딩(bit-banding)에 대한 자세한 내용은 'Cortex-M4 with FPU 프로그래밍 매뉴얼'(1페이지의 '관련 문서' 참조)을 확인하십시오.

### 2.4 부팅 구성
고정된 메모리 맵에 따라 코드 영역은 주소 0x0000 0000부터 시작하며(ICode/DCode 버스를 통해 접근), 데이터 영역(SRAM)은 주소 0x2000 0000부터 시작합니다(시스템 버스를 통해 접근). Cortex-M4 with FPU CPU는 항상 ICode 버스를 통해 리셋 벡터를 가져오는데, 이는 부팅 공간이 코드 영역(일반적으로 플래시 메모리)에만 존재해야 함을 의미합니다. STM32F4xx 마이크로컨트롤러는 다른 메모리(내부 SRAM 등)에서도 부팅할 수 있도록 특별한 메커니즘을 구현하고 있습니다.
STM32F4xx에서는 표 2에 나타난 바와 같이 BOOT[1:0] 핀을 통해 세 가지 부팅 모드 중 하나를 선택할 수 있습니다. 

BOOT 핀의 값은 리셋 후 SYSCLK의 4번째 상승 에지(rising edge)에서 래치(latch)됩니다. 사용자는 리셋 후 BOOT1 및 BOOT0 핀을 설정하여 원하는 부팅 모드를 선택해야 합니다.
BOOT0는 전용 핀인 반면, BOOT1은 GPIO 핀과 공유됩니다. BOOT1 값이 샘플링되고 나면, 해당 GPIO 핀은 자유 상태가 되어 다른 용도로 사용할 수 있습니다.
장치가 대기(Standby) 모드에서 빠져나올 때도 BOOT 핀을 다시 샘플링합니다. 따라서 장치가 대기 모드에 있는 동안에도 원하는 부팅 모드 설정 상태를 유지해야 합니다. 이러한 시작 지연(startup delay)이 끝나면, CPU는 주소 0x0000 0000에서 스택 최상단(top-of-stack) 값을 가져온 다음, 0x0000 0004부터 시작되는 부팅 메모리 영역에서 코드 실행을 시작합니다.

참고: 장치가 SRAM에서 부팅되는 경우, 애플리케이션 초기화 코드 내에서 NVIC 예외 테이블과 오프셋 레지스터를 사용하여 SRAM 내의 벡터 테이블을 재배치(relocate)해야 합니다.

**내장 부트로더(Embedded bootloader)**
내장 부트로더 모드는 다음 직렬 인터페이스 중 하나를 사용하여 플래시 메모리를 재프로그래밍하는 데 사용됩니다.
• USART1 (PA9/PA10)
• USART2 (PD5/PD6)
• I2C1 (PB6/PB7)
• I2C2 (PB10/PB3)
• I2C3 (PA8/PB4)
• SPI1 (PA4/PA5/PA6/PA7)
• SPI2 (PB12/PB13/PB14/PB15)
• SPI3 (PA15/PC10/PC11/PC12)
• USB OTG FS (PA11/12) - 디바이스 모드(DFU: Device Firmware Upgrade)
USART 주변 장치는 내부 16 MHz 오실레이터(HSI) 주파수에서 작동하며, USB OTG FS는 1 MHz의 배수인 외부 클록(HSE, 4~26 MHz 범위)을 필요로 합니다. 내장 부트로더 코드는 시스템 메모리에 위치하며, 제조 과정에서 ST에 의해 프로그래밍됩니다. 자세한 내용은 애플리케이션 노트 AN2606을 참조하십시오.

**STM32F411xC/E에서의 물리적 리맵(Physical remap)**
부트 핀이 선택되면, 애플리케이션 소프트웨어는 코드 영역에서 접근 가능한 메모리를 수정할 수 있습니다(이를 통해 코드는 시스템 버스 대신 ICode 버스를 통해 실행될 수 있습니다). 이러한 수정은 SYSCFG 컨트롤러 내의 '7.2.1절: SYSCFG 메모리 리맵 레지스터(SYSCFG_MEMRMP)'를 프로그래밍함으로써 수행됩니다.
다음 메모리들을 리맵할 수 있습니다.
• 메인 플래시 메모리(Main Flash memory)
• 시스템 메모리(System memory)
• 내장 SRAM1(Embedded SRAM1)
임베디드 플래시 메모리 인터페이스

### 3.1 개요
플래시 메모리 인터페이스는 CPU의 AHB I-Code 및 D-Code를 통한 플래시 메모리 접근을 관리합니다. 이 인터페이스는 플래시 메모리의 삭제 및 프로그래밍 작업과 읽기/쓰기 보호 메커니즘을 구현합니다.
또한, 명령어 프리페치(prefetch) 및 캐시 라인 시스템을 통해 코드 실행 속도를 향상시킵니다.

### 3.2 주요 특징
• 플래시 메모리 읽기 작업
• 플래시 메모리 프로그래밍/삭제 작업
• 읽기/쓰기 보호 기능
• I-Code에 대한 프리페치(prefetch) 기능
• I-Code용 128비트 캐시 라인 64개
• D-Code용 128비트 캐시 라인 8개

그림 2는 시스템 아키텍처 내에서 플래시 메모리 인터페이스가 어떻게 연결되어 있는지 보여줍니다.

<img src="./img/007-1.png" width="120%"> <br>
<img src="./img/009-1.png" width="85%"> <br>
<img src="./img/010-1.png" width="85%"> <br>
<img src="./img/011-1.png" width="85%"> <br>
</details>

## 4. GPIO

<details>
<summary>F411</summary>
   
## 8 범용 입출력(GPIO)
STM32F411xC/E 모델에서는 GPIO F/G/H/I/J/K를 사용할 수 없습니다(GPIOH0 및 GPIOH1은 제외).

### 8.1 GPIO 개요

각 범용 입출력(GPIO) 포트에는 4개의 32비트 설정 레지스터(GPIOx_MODER, GPIOx_OTYPER, GPIOx_OSPEEDR, GPIOx_PUPDR), <br>
2개의 32비트 데이터 레지스터(GPIOx_IDR, GPIOx_ODR),  <br>
32비트 세트/리셋 레지스터(GPIOx_BSRR),  <br>
32비트 잠금 레지스터(GPIOx_LCKR),  <br>
그리고 2개의 32비트 대체 기능 선택 레지스터(GPIOx_AFRH, GPIOx_AFRL)가 포함되어 있습니다. 

### 8.2 GPIO 주요 특징

   * 최대 16개의 I/O 제어 가능
   * 출력 상태: 푸시-풀(push-pull) 또는 오픈-드레인(open-drain) + 풀업/풀다운(pull-up/down)
   * 출력 데이터 소스: 출력 데이터 레지스터(GPIOx_ODR) 또는 주변 장치(대체 기능 출력)
   * 각 I/O별 속도 설정 가능
   * 입력 상태: 플로팅(floating), 풀업/풀다운, 아날로그
   * 입력 데이터 목적지: 입력 데이터 레지스터(GPIOx_IDR) 또는 주변 장치(대체 기능 입력)
   * 비트 설정 및 리셋 레지스터(GPIOx_BSRR)를 통한 GPIOx_ODR의 비트 단위 쓰기 접근 지원
   * I/O 구성을 고정할 수 있는 잠금 메커니즘(GPIOx_LCKR) 제공
   * 아날로그 기능 지원
   * 대체 기능(Alternate Function, AF) 입출력 선택 레지스터 (I/O당 최대 16개의 AF 지원)
   * 2클럭 주기마다 상태를 변경할 수 있는 고속 토글(fast toggle) 기능
   * 유연한 핀 멀티플렉싱을 통해 I/O 핀을 범용 I/O(GPIO) 또는 다양한 주변 장치 기능으로 활용 가능

### 8.3 GPIO 기능 설명

데이터시트에 명시된 각 I/O 포트의 하드웨어 특성에 따라, 범용 I/O(GPIO) 포트의 각 비트는 소프트웨어를 통해 다음과 같은 여러 모드로 개별 설정할 수 있습니다.
   * 입력 플로팅(Input floating)
   * 입력 풀업(Input pull-up)
   * 입력 풀다운(Input pull-down)
   * 아날로그(Analog)
   * 오픈-드레인 출력 (풀업 또는 풀다운 기능 포함)
   * 푸시-풀 출력 (풀업 또는 풀다운 기능 포함)
   * 대체 기능 푸시-풀 출력 (풀업 또는 풀다운 기능 포함)
   * 대체 기능 오픈-드레인 출력 (풀업 또는 풀다운 기능 포함)

각 I/O 포트 비트는 자유롭게 프로그래밍할 수 있지만, I/O 포트 레지스터에 접근할 때는 32비트 워드, 하프 워드 또는 바이트 단위로 접근해야 합니다. GPIOx_BSRR 레지스터는 GPIO 레지스터에 대한 원자적(atomic) 읽기/수정 접근을 가능하게 합니다. 이를 통해 읽기 작업과 수정 작업 사이에 인터럽트(IRQ)가 발생할 위험을 방지할 수 있습니다. 5V 허용(5V-tolerant) I/O 포트 비트의 기본 구조를 보여줍니다. 표 26은 포트 비트의 가능한 구성들을 나타냅니다.

<img src="./img/F16.png"> <br>

#### 8.3.9 입력 구성

I/O 포트가 입력으로 설정된 경우:
   * 출력 버퍼가 비활성화됩니다.
   * 슈미트 트리거(Schmitt trigger) 입력이 활성화됩니다.
   * GPIOx_PUPDR 레지스터의 값에 따라 풀업(pull-up) 및 풀다운(pull-down) 저항이 활성화됩니다.
   * I/O 핀에 존재하는 데이터는 매 AHB 클록 주기마다 입력 데이터 레지스터로 샘플링됩니다.
   * 입력 데이터 레지스터를 읽으면 I/O 상태를 확인할 수 있습니다. 그림 18은 I/O 포트 비트의 입력 구성을 보여줍니다.

<img src="./img/F18.png"> <br>

#### 8.3.10 출력 구성
I/O 포트가 출력으로 설정된 경우:
   * 출력 버퍼가 활성화됩니다.

오픈 드레인(Open-drain) 모드: 출력 레지스터의 값이 '0'이면 N-MOS가 활성화되고, '1'이면 포트가 하이 임피던스(Hi-Z) 상태가 됩니다(P-MOS는 절대 활성화되지 않음). 푸시-풀(Push-pull) 모드: 출력 레지스터의 값이 '0'이면 N-MOS가 활성화되고, '1'이면 P-MOS가 활성화됩니다.

   * 슈미트 트리거(Schmitt trigger) 입력이 활성화됩니다.
   * 약한(weak) 풀업 및 풀다운 저항은 GPIOx_PUPDR 레지스터의 설정값에 따라 활성화 여부가 결정됩니다.
   * I/O 핀에 존재하는 데이터는 매 AHB 클록 주기마다 입력 데이터 레지스터로 샘플링됩니다.
   * 입력 데이터 레지스터를 읽으면 현재 I/O 상태를 확인할 수 있습니다.
   * 출력 데이터 레지스터를 읽으면 마지막으로 기록된 값을 확인할 수 있습니다. 그림 19는 I/O 포트 비트의 출력 구성을 보여줍니다.

<img src="./img/F19.png"> <br>

#### 8.3.11 대체 기능(Alternate function) 구성
I/O 포트가 대체 기능으로 설정된 경우:
   * 출력 버퍼는 오픈 드레인(open-drain) 또는 푸시-풀(push-pull) 방식으로 구성할 수 있습니다.
   * 출력 버퍼는 주변 장치(peripheral)에서 오는 신호(송신기 활성화 및 데이터)에 의해 구동됩니다.
   * 슈미트 트리거(Schmitt trigger) 입력이 활성화됩니다.
   * 약한(weak) 풀업 및 풀다운 저항은 GPIOx_PUPDR 레지스터의 설정값에 따라 활성화 여부가 결정됩니다.
   * I/O 핀에 존재하는 데이터는 매 AHB 클록 주기마다 입력 데이터 레지스터로 샘플링됩니다.
   * 입력 데이터 레지스터를 읽으면 현재 I/O 상태를 확인할 수 있습니다.

그림 20은 I/O 포트 비트의 대체 기능 구성을 보여줍니다.

<img src="./img/F20.png"> <br>

#### 8.3.12 아날로그 구성

I/O 포트가 아날로그 구성으로 설정된 경우:
   * 출력 버퍼가 비활성화됩니다.

참고:
   * 슈미트 트리거(Schmitt trigger) 입력이 비활성화되어, I/O 핀의 아날로그 값에 관계없이 전력 소비가 발생하지 않습니다. 슈미트 트리거의 출력은 고정된 값(0)으로 강제 설정됩니다.
   * 약한(weak) 풀업 및 풀다운 저항이 비활성화됩니다.
   * 입력 데이터 레지스터를 읽으면 값 "0"이 반환됩니다.
아날로그 구성 상태인 I/O 핀은 5V 내성(5V tolerant)을 가질 수 없습니다.
그림 21은 I/O 포트 비트의 고임피던스 아날로그 입력 구성을 보여줍니다.

<img src="./img/F21.png"> <br>

<img src="./img/841.png"> <br>
<img src="./img/842.png"> <br>
<img src="./img/843.png"> <br>
<img src="./img/844.png"> <br>
<img src="./img/845.png"> <br>
<img src="./img/846.png"> <br>
<img src="./img/847.png"> <br>
<img src="./img/848.png"> <br>
<img src="./img/849.png"> <br>
<img src="./img/8410.png"> <br>

</details>

## 5. 실습

NUCLEO-F411RE 보드에서 GPIO 레지스터를 디버거로 직접 확인하고 조작해 본 내용 정리

### 5.0 GPIO 개요

- GPIO 는 **AHB1 버스**에 연결 → 사용 전 RCC 에서 클럭을 켜야 동작 (`__HAL_RCC_GPIOA_CLK_ENABLE()`)
- 포트 하나당 최대 16개 핀, 핀마다 모드를 따로 설정
- 출력 : Push-Pull / Open-Drain, 입력 : Floating / Pull-up / Pull-down, 그 외 Alternate Function / Analog
- ODR 은 직접 쓰거나 BSRR 로 비트 단위 Set / Reset

#### I/O 포트 비트 기본 구조 (RM0383 Figure 16)

![I/O 포트 기본 구조](img/R1_gpio_structure.png)

#### 출력 구성 (RM0383 Figure 19)

![출력 구성](img/R2_gpio_output.png)

- Write → BSRR / ODR → Output control → P-MOS / N-MOS → I/O 핀
- 출력 모드에서도 입력 드라이버(슈미트 트리거)는 켜져 있어서 **IDR 로 핀 상태를 읽을 수 있음**

#### 코드에서 LED 까지 전체 흐름

![MCU Core → Bus → GPIO → LED](img/R3_mcu_core_to_led.png)

1. **Flash** : PC 가 가리키는 `HAL_GPIO_WritePin(...)` 명령을 읽음
2. **Core 레지스터** : R0 = 포트 주소(`0x4002_0000`), R1 = 핀 번호, R2 = 상태 를 담아 함수 호출 (LR 에 돌아올 주소 저장)
3. **Bus Matrix (AHB)** : Load / Store 명령으로 GPIO 레지스터(SFR) 에 접근
4. **GPIO 레지스터** : MODER 로 출력 모드, BSRR / ODR 로 출력 값 설정
5. **I/O 핀** : Output control → P-MOS / N-MOS → 핀 전압 변화 → **LD2 (PA5)** 점등

---

### 5.1 기본 LED 토글 코드

```c
  /* USER CODE BEGIN 2 */
  int counter = 0;
  /* USER CODE END 2 */

  while (1)
  {
    counter++;
    HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
    HAL_Delay(1000);
  }
```

`LD2_GPIO_Port`, `LD2_Pin` 은 `main.h` 에 정의된 매크로일 뿐이다.

```c
#define LD2_Pin        GPIO_PIN_5
#define LD2_GPIO_Port  GPIOA
```

따라서 아래 두 줄은 같은 코드

```c
HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
```

---

### 5.2 실습 1 : PA5 (LD2) 토글 측정

5.1 의 기본 코드 실행
→ LD2 가 1초마다 깜박일 때 PA5 핀을 오실로스코프로 측정

#### 5.2.1 보드 연결

![보드 연결 PA5](img/G6_board_wiring_pa5.jpg)

| 선 | 보드 쪽 | 오실로스코프 쪽 |
|---|---|---|
| 보라색 | CN5 **PA5 (D13 / SCK)** | 프로브 팁 (CH1) |
| 노란색 | CN6 **GND** | 프로브 접지 집게 |

#### 5.2.2 오실로스코프 측정 결과

![오실로스코프 PA5 토글](img/G7_scope_pa5_toggle.jpg)

- CH1 프로브를 PA5 에 연결, 트리거 CH1 상승 에지 (V/div, 시간축 값은 파형이 보이도록 맞춘 것)
- 시간축 500 ms/div 에서 High 약 2칸 ≈ **1 s**, Low 약 2칸 ≈ **1 s** → `HAL_Delay(1000)` 마다 토글
- 주기 약 2 s (0.5 Hz), High 레벨 약 **3.3 V**
- 파형이 High 일 때 LD2 켜짐, Low 일 때 꺼짐

---

### 5.3 GPIOA 레지스터 맵

실습 1 은 코드(HAL)로 LED 를 깜박였다면, 이제 **레지스터를 직접 보고 바꿔보기** 위해 GPIOA 레지스터 구조를 확인

GPIOA Base Address : `0x4002_0000`

| Offset | 주소 | 레지스터 | 역할 |
|---|---|---|---|
| 0x00 | 0x4002_0000 | MODER | 핀 모드 (입력 / 출력 / AF / 아날로그) |
| 0x04 | 0x4002_0004 | OTYPER | 출력 타입 (Push-Pull / Open-Drain) |
| 0x08 | 0x4002_0008 | OSPEEDR | 출력 속도 |
| 0x0C | 0x4002_000C | PUPDR | Pull-up / Pull-down |
| 0x10 | 0x4002_0010 | IDR | 입력 데이터 (Read Only) |
| 0x14 | 0x4002_0014 | ODR | 출력 데이터 |
| 0x18 | 0x4002_0018 | BSRR | 비트 Set / Reset |
| 0x1C | 0x4002_001C | LCKR | 설정 잠금 |
| 0x20 | 0x4002_0020 | AFR[0] (AFRL) | Alternate Function (핀 0~7) |
| 0x24 | 0x4002_0024 | AFR[1] (AFRH) | Alternate Function (핀 8~15) |

![칠판 정리](img/G2_whiteboard_registers.png)

---

### 5.4 디버거로 레지스터 확인

#### 5.4.1 처음 결과 : 모든 값이 `0xA8000000` ?

![실행 전 Expressions](img/G1_expressions_before_run.png)

- 디버그를 시작하면 `main()` 첫 줄에서 멈춰 있다 → `MX_GPIO_Init()` **실행 전**
- `0xA8000000` 은 GPIOA MODER의 **리셋값** (PA13, PA14, PA15 = SWD 디버그 핀)
- 모든 레지스터가 같은 값으로 보인 것은 Expressions 창 표시 문제 → **SFRs 창** 또는 **Memory 창**(`0x40020000`)으로 확인하는 것이 정확

#### 5.4.2 해결 : F8 (Resume)

`HAL_Delay(1000);` 줄에 브레이크포인트 → **F8** 로 실행하면 GPIO 초기화 후 값이 보인다.

| 키 | 동작 |
|---|---|
| F8 | Resume : 다음 브레이크포인트까지 실행 |
| F6 | Step Over : 한 줄 실행 (함수 안으로 안 들어감) |
| F5 | Step Into : 함수 안으로 들어감 |

#### 5.4.3 리셋값 vs 실행 후 기대값 (PA5 출력)

| 레지스터 | GPIOA 리셋값 | 실행 후 |
|---|---|---|
| MODER | `0xA800_0000` | `0xA800_0400` |
| OTYPER | `0x0000_0000` | `0x0000_0000` |
| OSPEEDR | `0x0C00_0000` | `0x0C00_0000` |
| PUPDR | `0x6400_0000` | `0x6400_0000` |
| ODR | `0x0000_0000` | `0x0000_0020` ↔ `0x0000_0000` (1초마다 토글) |

> 칠판의 `0x0000_0400` 은 **PA5 비트만** 적은 값. 실제로는 디버그 핀(PA13~15) 설정 때문에 상위 비트 `0xA8` 이 붙어 `0xA800_0400` 이 정상.

---

### 5.5 MODER / ODR 비트 계산

MODER 는 **핀 하나당 2비트** 사용 → 핀 n 은 비트 `[2n+1 : 2n]`

| 값 | 모드 |
|---|---|
| 00 | Input |
| 01 | General purpose Output |
| 10 | Alternate Function |
| 11 | Analog |

ODR 은 **핀 하나당 1비트** → 핀 n 은 비트 n

| 핀 | MODER 비트 | 출력 설정값 | ODR ON 값 |
|---|---|---|---|
| PA5 | [11:10] | `0x0000_0400` | `0x0000_0020` |
| PA6 | [13:12] | `0x0000_1000` | `0x0000_0040` |

```
PA6 MODER : 01 << (6 × 2) = 1 << 12 = 0x1000
PA6 ODR   : 1 << 6               = 0x40
```

---

### 5.6 실습 2 : PA6 으로 바꾸기

5.3 ~ 5.5 에서 확인한 레지스터를 이용해, 디버거에서 MODER / ODR 값을 직접 수정
(MODER `0x1000` → PA6 출력, ODR `0x00` / `0x40` → PA6 Low / High)

![PA6 출력](img/G3_pa6_output.png)

| 레지스터 | 값 | 해석 |
|---|---|---|
| MODER | `0xA800_10A0` | `0x1000` : PA6 출력 / `0x00A0` : PA2, PA3 AF (USART2) / `0xA800_0000` : SWD |
| OSPEEDR | `0x0C00_00F0` | PA2, PA3 Very High Speed (UART) |
| IDR | `0x0000_C00C` | PA2, PA3 (UART Idle = High), PA14, PA15 (SWD) |
| ODR | `0x0` / `0x40` | PA6 Low / High |
| AFR[0] | 30464 = `0x7700` | PA2, PA3 = AF7 (USART2) |

- ODR 을 `0x40` / `0x0` 으로 바꾸면 **오실로스코프에서 PA6 신호가 잡힌다**

#### 5.6.1 보드 연결

![보드 연결](img/G4_board_wiring.jpg)

| 선 | 보드 쪽 | 오실로스코프 쪽 |
|---|---|---|
| 보라색 | CN10 **PA6 (D12)** | 프로브 팁 (CH1) |
| 노란색 | CN7 **GND** | 프로브 접지 집게 |

#### 5.6.2 오실로스코프 측정 결과

![오실로스코프 PA6](img/G5_scope_pa6_rising.jpg)

- CH1 프로브를 PA6 에 연결
- 상승 에지를 확인하려고 트리거를 **CH1, 상승 에지** 로 맞춤 (V/div, 시간축 값은 파형이 보이도록 맞춘 것)
- 파형이 올라간 지점 = 디버거에서 **ODR 을 `0x00` → `0x40` 으로 바꾼 순간** (PA6 Low → High)
- 0 V 에서 약 1.6 칸 × 2 V ≈ **3.3 V** 로 올라감 → PA6 이 HIGH 로 출력됨
