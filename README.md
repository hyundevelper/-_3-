# ESP32 GPIO 입출력 실습 보고서

# 1. 개요

이번 실습을 통해 ESP32 개발 보드의 GPIO(General Purpose Input/Output) 핀을 활용하여 디지털 및 아날로그 입출력 제어 방법을 학습하였다. 이를 통해 마이크로컨트롤러의 기본적인 하드웨어 제어 방식에 대한 이해를 높였으며, 실제 회로 구성과 코드 작성을 통해 이론을 실습으로 연결해볼 수 있었다.

# 2. ESP32 GPIO 개요

# 2.1 GPIO란?
GPIO는 General Purpose Input/Output의 약자로, 마이크로컨트롤러에서 다양한 외부 장치와의 연결 및 제어를 가능하게 해주는 범용 핀이다. 입력(Input)으로는 버튼, 센서 등의 데이터를 읽어올 수 있으며, 출력(Output)으로는 LED, 모터 등을 구동할 수 있다.

# 2.2. 주요 사항

디지털 출력: HIGH(1) 또는 LOW(0) 신호를 출력할 수 있다
디지털 입력: 버튼 등의 외부 신호를 읽을 수 있다
아날로그 출력(PWM, DAC): PWM 방식으로 LED 밝기 또는 모터 속도를 제어할 수 있다
아날로그 입력(ADC): 가변 저항 등의 아날로그 센서 값을 읽을 수 있다

# 2.3 ESP32 GPIO 핀 구성
ESP32는 총 39개의 핀을 가지며, 이 중 대부분을 GPIO로 활용할 수 있다. 다만 몇몇 핀은 전원 공급용이거나 입력 전용으로 제한된다.

전원 관련 핀
5V (VIN): USB 또는 외부로부터 5V 전원 공급
3.3V (3V3): 내부 전압 레귤레이터에서 나온 출력, 주로 센서 전원 공급에 사용
GND: 공통 접지
주의사항: 3.3V와 5V를 동시에 사용 시 보드가 손상될 수 있으므로 주의 필요
입출력 관련 핀
디지털 GPIO: 거의 모든 핀에서 HIGH/LOW 제어 가능 (단 GPIO34~39은 입력 전용)
ADC: ADC1과 ADC2로 구분됨 (Wi-Fi 기능 사용 시 ADC2 충돌 가능)
DAC: GPIO25(DAC1), GPIO26(DAC2)
PWM 출력: GPIO34~39 제외하고 대부분 핀에서 가능
터치 센서: GPIO4, 0, 2, 15, 13, 12, 14, 27, 33, 32

# 3. 실습

# 3.1 디지털 출력 – LED 제어
ESP32의 디지털 출력 기능을 활용하여 LED를 주기적으로 켜고 끄는 실습을 진행하였다. 회로 구성은 D4(GPIO4) 핀에 저항을 거쳐 LED를 연결하였다.

사용 핀: GPIO4
회로 설명:

GPIO4 → 저항(330Ω) → LED → GND

코드:

const int ledPin = 4; // GPIO4

void setup() {
  pinMode(ledPin, OUTPUT); // GPIO4를 출력 모드로 설정
}

void loop() {
  digitalWrite(ledPin, HIGH); // LED ON
  delay(1000);                // 1초 대기
  digitalWrite(ledPin, LOW);  // LED OFF
  delay(1000);                // 1초 대기
}

실행 결과: LED가 1초 간격으로 켜졌다가 꺼지는 것을 반복. 디지털 출력 기능을 통해 GPIO 핀에서 전압을 공급하여 외부 장치를 제어할 수 있음을 확인.

# 3.2 디지털 입력 – 버튼을 통한 LED 제어
이번 실습에서는 버튼 입력을 통해 LED 상태를 제어하는 방법을 익혔다. 버튼이 눌리는지를 디지털 입력으로 읽어와, 그 값을 LED 출력에 반영하는 방식이다.

사용 핀:

LED: GPIO4
버튼: GPIO33
회로 설명:
버튼 한쪽은 GND, 다른 한쪽은 GPIO33에 연결

코드:

const int ledPin = 4;
const int buttonPin = 33;
int buttonState = 0;

void setup() {
  pinMode(ledPin, OUTPUT);
  pinMode(buttonPin, INPUT); // 버튼 핀을 입력 모드로 설정
}

void loop() {
  buttonState = digitalRead(buttonPin);      // 버튼 상태 읽기
  digitalWrite(ledPin, buttonState);         // 버튼 상태에 따라 LED 제어
}

실행 결과:
버튼을 누르면 LED가 켜지고, 떼면 꺼짐. 버튼이 눌려있을 때 LOW 또는 HIGH 중 어떤 값을 읽는지에 따라 회로 설계와 코드 방향이 달라질 수 있다는 점을 이해하게 됨.

# 3.2.1 디지털 입력 응용 – 풀업 저항 사용

기존 실습에서는 외부 풀다운 저항이 필요했지만, 이번에는 ESP32 내부 풀업 저항 기능을 활용해 외부 저항 없이 회로를 간소화했다. 버튼을 누르면 LED가 꺼지고, 떼면 켜지는 구조로 실습을 구성하였다.

const int ledPin = 4;
const int buttonPin = 33;
int buttonState = 0;

void setup() {
  pinMode(ledPin, OUTPUT);
  pinMode(buttonPin, INPUT_PULLUP);  // 내부 풀업 사용
}

void loop() {
  buttonState = digitalRead(buttonPin);
  digitalWrite(ledPin, buttonState);  // 버튼 누르면 LED 꺼짐
}


실행 결과:
버튼을 누르면 LED가 꺼지고, 손을 떼면 다시 켜진다. 풀업 저항의 원리를 실제로 적용해본 덕분에 하드웨어와 소프트웨어 양쪽에서의 이해도를 높일 수 있었다.

# 4. 실습을 통해 알게 된 점

이번 실습을 통해 ESP32의 GPIO 핀을 이용하여 다양한 입출력 제어를 실제로 구현해보면서 마이크로컨트롤러 프로그래밍에 대한 기초를 탄탄히 할 수 있었다. 구체적으로는 다음과 같은 내용을 학습하였다.

GPIO 핀의 기본 개념과 종류별 기능
디지털 출력으로 외부 장치(LED) 제어하는 방법
디지털 입력으로 사용자 입력(버튼) 을 읽는 방법
내부 풀업 저항을 활용하여 회로를 간소화하는 실용적인 팁
코드 작성 시 핀 번호, 입력/출력 모드, 전기적 특성 고려 필요성
또한 단순히 동작 결과를 확인하는 것에 그치지 않고, 코드의 구조를 분석하고 하드웨어 회로와의 상관 관계를 명확히 이해하는 데 중점을 두었다. 이 과정에서, 디지털 입출력의 기본 로직뿐만 아니라 실제 프로젝트에서 발생할 수 있는 전기적 특성이나 오류 상황에 대한 대비 방법까지도 생각해볼 수 있었다.


