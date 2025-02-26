# 스마트 팜 시스템 (STM32 + Raspberry Pi)

## 프로젝트 개요
이 프로젝트는 STM32 마이크로컨트롤러와 라즈베리 파이를 활용하여 스마트 팜 시스템을 구축하는 프로젝트입니다. STM32는 온습도 센서(DHT11)와 조도 센서, 그리고 수위 센서를 통해 데이터를 수집하고, 이를 UART 통신을 통해 라즈베리 파이로 전송합니다. 라즈베리 파이는 수집된 데이터를 데이터베이스에 저장하고, 이 데이터를 바탕으로 STM32에 제어 명령을 보내어 스마트 팜 환경을 조절합니다.

## 시스템 구성
### STM32
STM32는 센서 데이터를 수집하고 라즈베리 파이와의 UART 통신을 통해 데이터를 송수신하는 역할을 합니다. STM32는 또한 LCD 디스플레이를 사용하여 실시간 데이터를 표시합니다.

### 라즈베리 파이
라즈베리 파이는 STM32에서 전송된 센서 데이터를 데이터베이스에 저장하며, 이를 바탕으로 STM32에 제어 명령을 전송합니다. 라즈베리 파이와 STM32 간의 UART 통신을 통해 실시간 데이터 전송 및 제어가 이루어집니다. 또한 UI를 통해서 원격으로 제어가 가능합니다.


## 시스템 흐름
1. **STM32**는 DHT11 온습도 센서와 ADC를 통해 센서 데이터를 수집합니다.
2. 이 데이터는 **UART**를 통해 **라즈베리 파이**로 전송됩니다.
3. **라즈베리 파이**는 데이터를 **MySQL DB**에 저장하고, 필요에 따라 STM32에 제어 명령을 보냅니다.
4. **STM32**는 수신된 명령에 따라 LED, 펌프, 팬 등을 제어합니다.

## 하드웨어 구성

- **STM32**: 마이크로컨트롤러 (센서 데이터 수집, 제어 명령 처리)
- **DHT11**: 온습도 센서
- **조도 센서**: 조도 측정 센서
- **펜 모터**: 환기 시스템 표현
- **LED**: 온도 조절 및 시스템 상태 표현ㄴ
- **LCD**: 실시간 데이터를 표시
- **라즈베리 파이**: 데이터베이스에 데이터 저장 및 제어 명령 송신

## 참고

- STM32 코드 예시와 라즈베리 파이 코드 예시는 모두 제공됩니다.
- 시스템은 UART 통신을 통해 STM32와 라즈베리 파이가 실시간으로 데이터를 주고받습니다.


---

## 주요 코드 설명

### STM32 코드
STM32에서는 UART 통신을 통해 라즈베리 파이와 데이터를 주고받습니다. 아래는 주요 코드입니다.

```c
// DHT11 센서 데이터를 UART로 전송
if(dht11Read(&dht)) {
    printf("%d, %d, %d\n", dht.temperature, dht.humidity, Light);
    updateLCD();  // LCD 업데이트
} else {
    printf("DHT11 Read Failed\r\n");
    DHT_Reset();
}

```


이 코드는 DHT11 센서로부터 온도와 습도를 읽고, 이를 UART를 통해 Raspberry Pi로 전송합니다.

### Raspberry Pi 코드 (Python)
Raspberry Pi는 STM32로부터 수신한 데이터를 DB에 저장하고, 해당 데이터를 기반으로 STM32에 명령을 전송합니다.


```
import serial
import sqlite3

# SQLite DB 연결
conn = sqlite3.connect('smart_farm.db')
c = conn.cursor()

# STM32와 UART 통신을 위한 설정
ser = serial.Serial('/dev/ttyUSB0', 9600)

def store_data(data):
    c.execute("INSERT INTO sensor_data (temperature, humidity, light) VALUES (?, ?, ?)", data)
    conn.commit()

def send_command(command):
    ser.write(command.encode())

while True:
    if ser.in_waiting > 0:
        data = ser.readline().decode('utf-8').strip()
        print(f"Received: {data}")
        
        # 데이터 DB에 저장
        temp, humi, light = map(int, data.split(','))
        store_data((temp, humi, light))

        # 예: 수위가 일정 이하일 경우 펌프를 켬
        if light < 1000:
            send_command('P')  # 펌프 ON
        else:
            send_command('p')  # 펌프 OFF
```

이 코드는 STM32로부터 데이터를 수신하여 SQLite 데이터베이스에 저장하고, 저장된 데이터를 바탕으로 제어 명령을 STM32로 전송합니다.

---
### 파일 구조

```

│ ├── /Core
│ ├── /Inc
│ └── /Src
      │ ├── I2C_LCD.c                # LCD 화면에 정보 출력
      │ ├── adc.c                    # 아날로그 데이터 수집 (ADC)
      │ ├── delay_us.c               # 마이크로초 단위의 딜레이 함수
      │ ├── dht.c                    # DHT11 온습도 센서 데이터 처리
      │ ├── dma.c                    # Direct Memory Access 설정
      │ ├── gpio.c                   # GPIO 설정 및 제어
      │ ├── i2c.c                    # I2C 통신 설정 및 처리
      │ ├── main.c                   # 메인 프로그램 (센서 데이터 수집 및 통신)
      │ ├── stm32f4xx_hal_msp.c      # HAL 초기화 및 MSP 설정
      │ ├── stm32f4xx_it.c           # 인터럽트 서비스 루틴
      │ ├── syscalls.c               # 시스템 호출 처리
      │ ├── sysmem.c                 # 시스템 메모리 처리
      │ ├── system_stm32f4xx.c       # 시스템 초기화 코드
      │ ├── tim.c                    # 타이머 설정 및 관리
      │ └── usart.c                  # USART 통신 설정 및 처리
└── ...

```

---


### 설치 및 실행
## STM32
- 1. STM32CubeMX를 사용하여 프로젝트를 생성하고, UART, ADC, 타이머 등을 설정합니다.
- 2. HAL 라이브러리를 기반으로 코드 작성 후, STM32에 업로드합니다.
## Raspberry Pi
- 1. Raspberry Pi에 Python 환경을 설정하고, pyserial 라이브러리를 설치합니다.
```
pip install pyserial

```
- 2. smart_farm.py를 실행하여 UART 통신을 시작합니다.


### 데이터베이스
SQLite 데이터베이스는 자동으로 생성되며, sensor_data 테이블이 생성됩니다. 테이블 구조는 다음과 같습니다.

```
CREATE TABLE sensor_data (
    id INTEGER PRIMARY KEY,
    temperature INTEGER,
    humidity INTEGER,
    light INTEGER
);

```
---

## 결론
- 이 프로젝트는 STM32와 Raspberry Pi 간의 UART 통신을 통해 스마트 팜 시스템을 실시간으로 제어하는 방법을 다룹니다. STM32는 센서 데이터를 수집하고 제어 명령을 처리하며, Raspberry Pi는 데이터를 DB에 저장하고 분석하여 STM32에 명령을 전송합니다.
