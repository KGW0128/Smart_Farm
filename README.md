# 스마트 팜 시스템 (STM32 + Raspberry Pi)

## 프로젝트 개요
본 프로젝트는 STM32 마이크로컨트롤러와 Raspberry Pi 간의 UART 통신을 통해 실시간으로 센서 데이터를 주고받고, Raspberry Pi에서 이를 데이터베이스(DB)에 저장하여 관리합니다. 저장된 데이터를 기반으로 STM32에 통제 메시지를 보내어 스마트 팜 시스템을 제어하는 프로젝트입니다.

## 시스템 구성
1. **STM32**:
   - 온도, 습도, 조도, 수위 등을 측정하는 다양한 센서를 사용하여 데이터를 수집합니다.
   - 라즈베리 파이와 UART 통신을 통해 센서 데이터를 전송하고, 라즈베리 파이로부터 명령을 받습니다.
   - 각종 장치(LED, 펌프, 팬 등)의 제어를 수행합니다.

2. **Raspberry Pi**:
   - STM32로부터 데이터를 수신하여 데이터베이스에 저장합니다.
   - 데이터 분석을 통해 STM32로 제어 명령을 전송합니다.

## 기능
- **STM32**:
  - DHT11 온습도 센서로 온도 및 습도를 측정합니다.
  - ADC를 사용하여 조도와 수위 값을 측정합니다.
  - 온습도, 조도, 수위 값을 UART를 통해 Raspberry Pi로 전송합니다.
  - Raspberry Pi로부터 제어 명령(예: 팬, 펌프, LED 상태)을 수신하여 해당 장치를 제어합니다.

- **Raspberry Pi**:
  - STM32로부터 실시간 데이터를 수신하여 데이터베이스에 저장합니다.
  - 저장된 데이터를 바탕으로 시스템 상태를 분석하고, 필요시 STM32로 제어 명령을 전송합니다.

## 시스템 동작
1. STM32는 주기적으로 DHT11 센서를 통해 온도와 습도를 측정하고, ADC를 통해 조도와 수위 값을 측정합니다.
2. 측정된 데이터는 UART를 통해 Raspberry Pi로 전송됩니다.
3. Raspberry Pi는 받은 데이터를 데이터베이스에 저장하고, 저장된 데이터를 분석하여 STM32로 제어 명령을 보냅니다.
4. STM32는 Raspberry Pi로부터 받은 명령에 따라 펌프, 팬, LED 등을 제어합니다.

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

/project_root
│
├── /stm32
│   ├── main.c                # STM32 코드
│   ├── adc.c
│   ├── uart.c
│   └── (기타 STM32 관련 파일)
│
├── /raspberry_pi
│   ├── smart_farm.py         # Raspberry Pi 코드
│   └── /database
│       └── smart_farm.db     # SQLite 데이터베이스 파일
│
└── README.md                 # 프로젝트 설명서

```

### 설치 및 실행
## STM32
1. STM32CubeMX를 사용하여 프로젝트를 생성하고, UART, ADC, 타이머 등을 설정합니다.
2. HAL 라이브러리를 기반으로 코드 작성 후, STM32에 업로드합니다.
## Raspberry Pi
1. Raspberry Pi에 Python 환경을 설정하고, pyserial 라이브러리를 설치합니다.
```
pip install pyserial

```
2. smart_farm.py를 실행하여 UART 통신을 시작합니다.


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
이 프로젝트는 STM32와 Raspberry Pi 간의 UART 통신을 통해 스마트 팜 시스템을 실시간으로 제어하는 방법을 다룹니다. STM32는 센서 데이터를 수집하고 제어 명령을 처리하며, Raspberry Pi는 데이터를 DB에 저장하고 분석하여 STM32에 명령을 전송합니다.
