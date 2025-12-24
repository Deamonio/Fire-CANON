<div align="center">

# 🔥 Fire-CANON

### Auto Fire Suppression Robot

*An automated fire detection and extinguishing system designed to respond quickly and efficiently*

![C++](https://img.shields.io/badge/C++-100%25-00599C?style=flat&logo=c%2B%2B&logoColor=white)
![Arduino](https://img.shields.io/badge/Arduino-00979D?style=flat&logo=arduino&logoColor=white)
![DYNAMIXEL](https://img.shields.io/badge/DYNAMIXEL-FF6F00?style=flat)

![GitHub Stars](https://img.shields.io/github/stars/Deamonio/Fire-CANON?style=social)
![GitHub Forks](https://img.shields.io/github/forks/Deamonio/Fire-CANON?style=social)

**Topics:** `arduino` `fire` `robot` `opencv` `video-processing` `mist` `hexagon` `binary-hive`

---

### 📸 System Preview

<div align="center">
  <!-- <img src="product_1.jpg" alt="Fire-CANON Robot" width="45%" /> -->
  <img src="product_2.jpg" alt="Fire-CANON Control System" width="100%" />
</div>

*자동 화재 진압 로봇 - 열화상 센서 & 물분무 시스템*

</div>

---

## 📋 목차

- [프로젝트 소개](#-프로젝트-소개)
- [시스템 아키텍처](#-시스템-아키텍처)
- [주요 기능](#-주요-기능)
- [하드웨어 구성](#-하드웨어-구성)
- [코드 분석](#-코드-분석)
- [설치 방법](#-설치-방법)
- [사용 방법](#-사용-방법)
- [알고리즘 설명](#-알고리즘-설명)

---

## 🎯 프로젝트 소개

**Fire-CANON**은 열화상 센서를 활용하여 화재를 자동으로 감지하고, 2축 팬-틸트 시스템으로 화재 지점을 정확히 조준한 후 물분무를 통해 진압하는 **지능형 소방 로봇**입니다. 

### 💡 배경

> **문제점**
> 초기 화재 대응 지연으로 인한 피해 확대

> **해결책**
> 24시간 무인 감시 → 자동 화재 감지 → 즉각 진압

### 🌟 특징

- 🔥 **실시간 화재 감지**:  AMG88xx 8×8 열화상 센서
- 🎯 **정밀 조준**:  DYNAMIXEL 모터 기반 팬-틸트
- 💧 **자동 진압**: 워터 펌프 물분무 시스템
- 🧠 **사분면 알고리즘**: 빠른 화재 위치 탐색
- 🔄 **연속 모니터링**: 화재 진압 후 재감시

---

## 🏗️ 시스템 아키텍처

### 전체 구조

```
AMG88xx 열화상 센서 (8x8 픽셀)
- 온도 범위: 0~80°C
- 시야각: 60°
- 화재 임계값: 28°C
         |
         | I2C 통신
         ↓
Arduino 메인 컨트롤러
1. 열화상 데이터 읽기 (64픽셀)
2. 화재 감지 (온도 임계값 비교)
3. 사분면 판단 (4구역)
4. 모터 제어 명령 생성
         |
    -----+-----
    |         |
    | TTL     | GPIO
    ↓         ↓
DYNAMIXEL   워터 펌프
- ID 6: 좌우   - Pin 12
- ID 0: 상하   - 2. 5초 분무
    |         |
    -----+-----
         ↓
2축 팬-틸트 메커니즘 + 노즐
- 수평 회전: ±180°
- 수직 회전: ±90°
- 화재 지점 정밀 조준
```

---

## ✨ 주요 기능

### 1. 🔥 열화상 화재 감지

**AMG88xx 센서 (8×8 = 64픽셀)**

```cpp
// 열화상 데이터 읽기
float pixels[64];
amg. readPixels(pixels);

// 화재 감지 (28°C 이상)
int temp = 28;
for (int i = 0; i < 64; i++) {
    if (pixels[i] >= temp) {
        // 화재 감지! 
        detectFire(i);
    }
}
```

**센서 레이아웃 (8×8 픽셀):**
```
픽셀 인덱스: 
 0  1  2  3  4  5  6  7
 8  9 10 11 12 13 14 15
16 17 18 19 20 21 22 23
24 25 26 27 28 29 30 31
32 33 34 35 36 37 38 39
40 41 42 43 44 45 46 47
48 49 50 51 52 53 54 55
56 57 58 59 60 61 62 63
```

**화재 감지 예시:**
- Pixel[25] = 35°C → 화재!  (2사분면)
- Pixel[50] = 22°C → 정상
- Pixel[42] = 30°C → 화재!  (3사분면)

---

### 2. 🎯 사분면 기반 위치 판단

**4개 구역으로 분할:**

```
8x8 센서를 4개 사분면으로 분할

좌측 상단 (2사분면)    우측 상단 (1사분면)
0-3, 8-11            4-7, 12-15
16-19, 24-27         20-23, 28-31

좌측 하단 (3사분면)    우측 하단 (4사분면)
32-35, 40-43         36-39, 44-47
48-51, 56-59         52-55, 60-63
```

**사분면별 동작:**

```cpp
// 2사분면 (좌상) - 왼쪽으로 이동 & 위로 조준
if (i >= 0 && i <= 3 || i >= 8 && i <= 11 || 
    i >= 16 && i <= 19 || i >= 24 && i <= 27) {
    moveLeft();   // 좌회전
    moveUp();     // 상향
    fireMist();   // 물분무
}

// 1사분면 (우상) - 오른쪽으로 이동 & 위로 조준
if (i >= 4 && i <= 7 || i >= 12 && i <= 15 || 
    i >= 20 && i <= 23 || i >= 28 && i <= 31) {
    moveRight();  // 우회전
    moveUp();     // 상향
    fireMist();
}

// 3사분면 (좌하) - 왼쪽으로 이동 & 아래로 조준
if (i >= 32 && i <= 35 || i >= 40 && i <= 43 || 
    i >= 48 && i <= 51 || i >= 56 && i <= 59) {
    moveLeft();
    moveDown();   // 하향
    fireMist();
}

// 4사분면 (우하) - 오른쪽으로 이동 & 아래로 조준
if (i >= 36 && i <= 39 || i >= 44 && i <= 47 || 
    i >= 52 && i <= 55 || i >= 60 && i <= 63) {
    moveRight();
    moveDown();
    fireMist();
}
```

---

### 3. 🎮 DYNAMIXEL 모터 제어

**2축 팬-틸트 시스템:**

```cpp
// 모터 ID 정의
const uint8_t DXL_ID_LR = 6;  // 좌우 (Left-Right)
const uint8_t DXL_ID_UD = 0;  // 상하 (Up-Down)

// 좌회전 (10% 속도)
dxl.setGoalVelocity(DXL_ID_LR, 10.0, UNIT_PERCENT);

// 우회전 (90% 속도)
dxl.setGoalVelocity(DXL_ID_LR, -90.0, UNIT_PERCENT);

// 상향 (45% 속도)
dxl.setGoalVelocity(DXL_ID_UD, -45.0, UNIT_PERCENT);

// 하향 (40% 속도)
dxl.setGoalVelocity(DXL_ID_UD, 40.0, UNIT_PERCENT);

// 정지
dxl.setGoalVelocity(DXL_ID_LR, 0);
dxl.setGoalVelocity(DXL_ID_UD, 0);
```

**중심선 정렬:**
```cpp
// 좌우 중심선 (세로 중앙)
int centerLR[] = {4, 12, 20, 28, 36, 44, 52, 60};

// 상하 중심선 (가로 중앙)
int centerUD[] = {32, 33, 34, 35, 36, 37, 38, 39};

// 중심선 도달 시 정지
if (pixels[centerLR[i]] >= temp) {
    dxl.setGoalVelocity(DXL_ID_LR, 0);
    break;
}
```

---

### 4. 💧 워터 펌프 제어

**물분무 시스템:**

```cpp
// 워터 펌프 핀 (Pin 12)
pinMode(12, OUTPUT);
digitalWrite(12, LOW);  // 초기 OFF

// 화재 진압
void fireMist() {
    Serial.println("fire check");
    
    // 펌프 ON (2.5초 분무)
    digitalWrite(12, HIGH);
    delay(2500);
    
    // 펌프 OFF
    digitalWrite(12, LOW);
    delay(1000);  // 대기
}
```

**분무 시퀀스:**
```
1. 화재 감지
2. 화재 지점 조준 (팬-틸트 이동)
3. 중심선 정렬 확인
4. 워터 펌프 ON (2.5초)
5. 워터 펌프 OFF
6. 재감시 모드
```

---

## 🛠️ 하드웨어 구성

### BOM (Bill of Materials)

| 부품 | 수량 | 용도 | 가격 (KRW) |
|---|---|---|---|
| Arduino Mega | 1 | 메인 컨트롤러 | 15,000 |
| AMG88xx | 1 | 열화상 센서 | 25,000 |
| DYNAMIXEL AX-12A | 2 | 팬-틸트 모터 | 60,000 |
| DYNAMIXEL Shield | 1 | 모터 드라이버 | 30,000 |
| 워터 펌프 | 1 | 물분무 시스템 | 10,000 |
| 릴레이 모듈 | 1 | 펌프 제어 | 2,000 |
| 노즐 | 1 | 분무 헤드 | 5,000 |
| 물탱크 | 1 | 물 저장 | 8,000 |
| 전원 공급 | 1 | 12V 어댑터 | 15,000 |
| 프레임 | 1 | 로봇 구조체 | 20,000 |
| **총합** | | | **190,000** |

---

### 🔌 핀 연결도

**Arduino Mega 연결:**

```
Arduino Mega
├── I2C (SDA/SCL) → AMG88xx 열화상 센서
├── Serial3 (TX/RX) → DYNAMIXEL Shield
├── Digital Pin 12 → 릴레이 모듈 → 워터 펌프
├── 5V → 센서 전원
└── GND → 공통 접지

DYNAMIXEL Shield
├── TTL Port → DYNAMIXEL ID 6 (좌우 모터)
├── TTL Port → DYNAMIXEL ID 0 (상하 모터)
└── 12V Power → 외부 전원
```

---

### 🏗️ 회로도

**시스템 연결:**

```
Arduino Mega
    |
    +-- I2C --> AMG88xx (SDA, SCL)
    |
    +-- Serial3 --> DYNAMIXEL Shield
    |
    +-- D12 --> Relay --> Water Pump
    |
    +-- 5V --> Sensor VCC
    |
    +-- GND --> Common GND

DYNAMIXEL Shield
    |
    +-- TTL1 --> Motor ID 6 (LR)
    |
    +-- TTL2 --> Motor ID 0 (UD)
    |
    +-- 12V --> External Power
```

---

## 💻 코드 분석

### 1. Setup (초기화)

```cpp
void setup() {
    // 1. 워터 펌프 핀 설정
    pinMode(12, OUTPUT);
    digitalWrite(12, LOW);
    
    // 2. 시리얼 통신 시작
    Serial.begin(9600);
    
    // 3. AMG88xx 센서 초기화
    if (!amg.begin()) {
        Serial.println("센서 연결 실패!");
        while (1);  // 무한 루프 (에러)
    }
    
    // 4. DYNAMIXEL 초기화
    dxl. begin(9600);
    dxl.setPortProtocolVersion(1. 0);
    
    // 5. 모터 핑 테스트
    dxl. ping(DXL_ID_LR);
    dxl.ping(DXL_ID_UD);
    
    // 6. 모터 모드 설정 (속도 제어 모드)
    dxl.torqueOff(DXL_ID_LR);
    dxl.setOperatingMode(DXL_ID_LR, OP_VELOCITY);
    dxl.torqueOn(DXL_ID_LR);
    
    dxl.torqueOff(DXL_ID_UD);
    dxl.setOperatingMode(DXL_ID_UD, OP_VELOCITY);
    dxl.torqueOn(DXL_ID_UD);
}
```

---

### 2. Loop (메인 루프)

```cpp
void loop() {
    // 1. 열화상 데이터 읽기
    int temp = 28;  // 화재 임계값
    float pixels[64];
    amg.readPixels(pixels);
    
    // 2. 64개 픽셀 순회
    for (int i = 0; i < 64; i++) {
        // 3. 화재 감지
        if (pixels[i] >= temp) {
            // 4. 사분면 판단 & 대응
            if (/* 2사분면 */) {
                moveLeftAndUp();
                fireMist();
            } 
            else if (/* 1사분면 */) {
                moveRightAndUp();
                fireMist();
            }
            else if (/* 3사분면 */) {
                moveLeftAndDown();
                fireMist();
            }
            else if (/* 4사분면 */) {
                moveRightAndDown();
                fireMist();
            }
        }
    }
}
```

---

### 3. 화재 진압 알고리즘

**2사분면 예시 (좌상):**

```cpp
// Step 1: 좌우 중심선 정렬
while (true) {
    amg.readPixels(pixels);
    
    // 중심선 도달 체크
    if (pixels[4] >= temp || pixels[12] >= temp || 
        pixels[20] >= temp || pixels[28] >= temp || 
        pixels[36] >= temp || pixels[44] >= temp || 
        pixels[52] >= temp || pixels[60] >= temp) {
        dxl.setGoalVelocity(DXL_ID_LR, 0);  // 정지
        break;
    }
    
    // 왼쪽으로 이동 (10% 속도)
    dxl.setGoalVelocity(DXL_ID_LR, 10.0, UNIT_PERCENT);
}

delay(1000);  // 안정화

// Step 2: 상하 중심선 정렬
while (true) {
    amg.readPixels(pixels);
    
    // 중심선 도달 체크
    if (pixels[32] >= temp || pixels[33] >= temp || 
        pixels[34] >= temp || pixels[35] >= temp || 
        pixels[36] >= temp || pixels[37] >= temp || 
        pixels[38] >= temp || pixels[39] >= temp) {
        dxl.setGoalVelocity(DXL_ID_UD, 0);  // 정지
        break;
    }
    
    // 위로 이동 (45% 속도)
    dxl.setGoalVelocity(DXL_ID_UD, -45.0, UNIT_PERCENT);
}

// Step 3: 물분무
Serial.println("fire check");
digitalWrite(12, HIGH);  // 펌프 ON
delay(2500);             // 2.5초 분무
digitalWrite(12, LOW);   // 펌프 OFF
delay(1000);             // 대기
```

---

## 📦 설치 방법

### 1. Arduino IDE 설치

```bash
# Arduino IDE 다운로드
https://www.arduino.cc/en/software
```

---

### 2. 라이브러리 설치

**필수 라이브러리:**

1. **Dynamixel2Arduino**
   ```
   스케치 → 라이브러리 포함하기 → 라이브러리 관리
   검색:  "Dynamixel2Arduino"
   설치
   ```

2. **Adafruit AMG88xx**
   ```
   검색: "Adafruit AMG88xx"
   설치 (+ Adafruit GFX, Adafruit BusIO 자동 설치)
   ```

3. **Wire** (내장)
   ```
   I2C 통신 라이브러리 (별도 설치 불필요)
   ```

---

### 3. 코드 업로드

```bash
# 1. 저장소 클론
git clone https://github.com/Deamonio/Fire-CANON.git
cd Fire-CANON

# 2. Arduino IDE에서 main/canon_pj.ino 열기

# 3. 보드 설정
도구 → 보드 → Arduino Mega 2560

# 4. 포트 선택
도구 → 포트 → COM3 (Windows) / /dev/ttyUSB0 (Linux)

# 5. 업로드
스케치 → 업로드 (Ctrl+U)
```

---

### 4. 하드웨어 연결

**순서:**

1. **AMG88xx 센서**
   - VCC → 5V
   - GND → GND
   - SDA → SDA (Arduino Mega Pin 20)
   - SCL → SCL (Arduino Mega Pin 21)

2. **DYNAMIXEL Shield**
   - Arduino Mega 위에 장착
   - 모터 ID 6, 0 연결
   - 12V 전원 연결

3. **워터 펌프**
   - 릴레이 IN → Pin 12
   - 릴레이 VCC → 5V
   - 릴레이 GND → GND
   - 펌프 → 릴레이 NO (상시개방)

---

## 🚀 사용 방법

### 1️⃣ 시스템 시작

```bash
# 1. 물탱크 물 채우기
# 2. 전원 연결 (12V 어댑터)
# 3. Arduino 전원 ON
```

**시작 시퀀스:**
```
1. AMG88xx 센서 초기화
2. DYNAMIXEL 모터 Ping 테스트
3. 모터 속도 제어 모드 설정
4. 화재 감시 시작
```

---

### 2️⃣ 화재 시뮬레이션

**테스트 방법:**

```bash
# 1. 라이터/히터 준비
# 2. 센서 시야각 (60°) 내에 열원 배치
# 3. 로봇이 자동으로 감지 & 조준
# 4. 물분무 작동 확인
```

**시리얼 모니터 출력:**
```
Arduino IDE → 도구 → 시리얼 모니터 (9600 baud)

출력 예시:
25. 1, 24.8, 26.3, 28.5, ...  (열화상 데이터)
mode UD
fire check
```

---

### 3️⃣ 사분면별 테스트

**각 구역 테스트:**

```
좌측 상단 (2사분면)   우측 상단 (1사분면)
좌+상                우+상

좌측 하단 (3사분면)   우측 하단 (4사분면)
좌+하                우+하

1.  2사분면:  왼쪽 위에 열원 → 좌회전 & 상향
2. 1사분면: 오른쪽 위에 열원 → 우회전 & 상향
3. 3사분면: 왼쪽 아래에 열원 → 좌회전 & 하향
4. 4사분면: 오른쪽 아래에 열원 → 우회전 & 하향
```

---

## 🧮 알고리즘 설명

### 1. 화재 감지 알고리즘

**단계:**

```
1. 열화상 센서 읽기 (64픽셀)
   ↓
2. 각 픽셀 온도 체크
   ↓
3. 임계값(28°C) 이상 → 화재! 
   ↓
4. 픽셀 인덱스로 사분면 판단
   ↓
5. 해당 사분면 대응 알고리즘 실행
```

**의사 코드:**

```
FOR each pixel in 64:
    IF pixel_temp >= 28°C:
        quadrant = getQuadrant(pixel_index)
        
        IF quadrant == 2:    // 좌상
            moveLeft()
            moveUp()
            fireMist()
        
        ELSE IF quadrant == 1:  // 우상
            moveRight()
            moveUp()
            fireMist()
        
        ELSE IF quadrant == 3:  // 좌하
            moveLeft()
            moveDown()
            fireMist()
        
        ELSE IF quadrant == 4:  // 우하
            moveRight()
            moveDown()
            fireMist()
```

---

### 2. 중심선 정렬 알고리즘

**좌우 정렬 (Left-Right):**

```
세로 중심선 (Column 4)
 0  1  2  3 [4] 5  6  7
 8  9 10 11[12]13 14 15
16 17 18 19[20]21 22 23
24 25 26 27[28]29 30 31
32 33 34 35[36]37 38 39
40 41 42 43[44]45 46 47
48 49 50 51[52]53 54 55
56 57 58 59[60]61 62 63

WHILE not aligned:
    IF centerLine[i] >= 28°C:
        STOP
        break
    ELSE:
        MOVE(direction)
```

**상하 정렬 (Up-Down):**

```
가로 중심선 (Row 4)
 0  1  2  3  4  5  6  7
 8  9 10 11 12 13 14 15
16 17 18 19 20 21 22 23
24 25 26 27 28 29 30 31
----중심선----
32 33 34 35 36 37 38 39
----중심선----
40 41 42 43 44 45 46 47
48 49 50 51 52 53 54 55
56 57 58 59 60 61 62 63
```

---

### 3. 속도 제어 전략

**모터 속도 설정:**

| 동작 | 모터 | 속도 | 이유 |
|---|---|---|---|
| 좌회전 | LR | 10% | 정밀 제어 |
| 우회전 | LR | 90% | 빠른 이동 |
| 상향 | UD | 45% | 중간 속도 |
| 하향 | UD | 40% | 안정적 |

**속도 차이 이유:**
- 좌회전 (10%): 화재가 왼쪽에 있을 때 천천히 조준
- 우회전 (90%): 화재가 오른쪽에 있을 때 빠르게 이동
- 상향 (45%): 위쪽 화재는 중간 속도로 접근
- 하향 (40%): 아래쪽 화재는 안정적으로 조준

---

## 🔧 문제 해결

### 1. AMG88xx 센서 인식 실패

**증상:**
```
Could not find a valid AMG88xx sensor, check wiring! 
```

**해결:**
```bash
# 1. I2C 주소 확인
도구 → I2C Scanner 실행
# AMG88xx 주소: 0x69

# 2. 배선 확인
SDA → Pin 20 (Arduino Mega)
SCL → Pin 21 (Arduino Mega)
VCC → 5V
GND → GND

# 3. 센서 전원 재연결
5V 핀 뽑았다가 다시 꽂기
```

---

### 2. DYNAMIXEL 모터 응답 없음

**증상:**
```
Ping failed for ID 6 or ID 0
```

**해결:**
```bash
# 1. 보드 레이트 확인
# DYNAMIXEL 기본:  9600 (코드와 일치해야 함)

# 2.  DYNAMIXEL Wizard 사용
# ID 확인: 6, 0
# Baud Rate 확인: 9600

# 3. 전원 확인
12V 어댑터 연결 확인
DYNAMIXEL Shield 전원 LED 확인
```

---

### 3. 워터 펌프 작동 안 함

**증상:**
```
화재 감지했지만 물분무 없음
```

**해결:**
```bash
# 1. 릴레이 연결 확인
IN → Pin 12
COM → 펌프 +
NO → 전원 +

# 2. 릴레이 LED 확인
Pin 12 HIGH 시 LED 켜짐? 

# 3. 펌프 전원 확인
별도 12V 전원 필요
```

---

### 4. 화재 오감지

**증상:**
```
화재 없는데 자꾸 감지됨
```

**해결:**
```cpp
// 임계값 조정 (28°C → 35°C)
int temp = 35;  // 더 높은 온도로 설정

// 또는 연속 감지 조건 추가
int fireCount = 0;
for (int i = 0; i < 64; i++) {
    if (pixels[i] >= temp) {
        fireCount++;
    }
}
if (fireCount >= 5) {  // 5개 이상 픽셀이 뜨거우면
    // 화재 판정
}
```

---

## 📊 성능 지표

### 시스템 성능

| 항목 | 성능 |
|---|---|
| 화재 감지 시간 | 약 1초 |
| 조준 시간 | 2-5초 |
| 물분무 시간 | 2. 5초 |
| 총 대응 시간 | 5-10초 |
| 감지 범위 | 60도 |
| 감지 거리 | 약 7m |
| 온도 정확도 | ±2. 5°C |

---

## 🎯 활용 사례

### 1. 실내 화재 예방

```
주택/사무실 천장 설치
→ 24시간 모니터링
→ 초기 화재 자동 진압
```

### 2. 산업 현장

```
공장 위험 구역 배치
→ 화학 물질 화재 대응
→ 인명 피해 최소화
```

### 3. 교육/연구

```
로봇 공학 교육 자료
→ 센서 융합 학습
→ 자율 시스템 연구
```

---

## 🚀 향후 개선 사항

- [ ] AI 객체 인식 (OpenCV)
- [ ] 다중 화재 동시 대응
- [ ] 연기 감지 센서 추가
- [ ] 자율 주행 기능 (화재 추적)
- [ ] IoT 연동 (화재 알림 앱)
- [ ] 소화제 자동 교체 시스템

---

## 🤝 기여하기

기여는 언제나 환영합니다! 🔥

### 기여 방법

1. Fork 이 저장소
2. Feature 브랜치 생성:  `git checkout -b feature/AmazingFeature`
3. 변경사항 커밋: `git commit -m 'Add some AmazingFeature'`
4. 브랜치에 Push: `git push origin feature/AmazingFeature`
5. Pull Request 생성

---

## 📜 라이선스

이 프로젝트는 MIT License 하에 배포됩니다.

---

## 📞 연락처

<div align="center">

### 프로젝트 관리자:  Deamonio

[![Email](https://img.shields.io/badge/Email-hyun0810d@gmail.com-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:hyun0810d@gmail.com)
[![GitHub](https://img.shields.io/badge/GitHub-Deamonio-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Deamonio)

**프로젝트 링크**:  [https://github.com/Deamonio/Fire-CANON](https://github.com/Deamonio/Fire-CANON)

</div>

---

## 🙏 감사의 말

| Arduino | DYNAMIXEL | AMG88xx | Adafruit |
|---|---|---|---|
| 플랫폼 | 모터 시스템 | 열화상 센서 | 라이브러리 |

**특별 감사:**
- 🔥 **소방 안전 전문가** - 시스템 설계 자문
- 🤖 **ROBOTIS** - DYNAMIXEL 기술 지원
- 📚 **Arduino Community** - 풍부한 예제
- 🛠️ **Adafruit** - 센서 라이브러리

---

<div align="center">

## ⭐ 이 프로젝트가 마음에 드셨다면 Star를 눌러주세요!

[![Star History Chart](https://api.star-history.com/svg?repos=Deamonio/Fire-CANON&type=Date)](https://star-history.com/#Deamonio/Fire-CANON&Date)

---

**Made with 🔥 by Deamonio**

*"Protecting lives with intelligent fire suppression"*

---

**© 2025 Deamonio. All rights reserved.**

[⬆ 맨 위로 돌아가기](#-fire-canon)

</div>
