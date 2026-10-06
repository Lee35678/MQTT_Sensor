# MQTT_Sensor

LilyGO T-Display S3에서 DHT22(온습도)와 BH1750(조도)을 읽어 TFT 화면에 표시하고 5초마다 MQTT로 발행하는 센서 노드.

## 개요

WiFi 정보와 MQTT 브로커 주소는 코드에 적지 않고 `ConfigPortal32` 설정 포털로 입력받는다. 설정이 끝나면 WiFi에 접속하고 브로커에 연결한 뒤, 측정값을 내장 TFT에 표시하면서 개별 토픽과 JSON 토픽으로 함께 발행한다. JSON 토픽(`id/LeeDH/sensor/data`)은 `MQTT_Relay_ConfigPortal32` 펌웨어가 구독하는 토픽과 같다.

## 하드웨어

- 보드: LilyGO T-Display S3 (`lilygo-t-display-s3`), 내장 TFT 디스플레이 사용
- 센서: DHT22 온습도 센서, BH1750 조도 센서(I2C)

| 장치 | 신호 | 핀 |
|------|------|----|
| DHT22 | DATA | GPIO 18 (`DHTPIN`) |
| BH1750 | SDA | GPIO 44 (`SDA_PIN`) |
| BH1750 | SCL | GPIO 43 (`SCL_PIN`) |

## 동작 방식

### 부팅 및 설정

1. `loadConfig()`로 저장된 설정을 읽는다
2. `config` 항목이 없거나 값이 `done`이 아니면 `configDevice()`로 설정 포털 실행
   - 포털 AP 이름 접두어: `ssid_pfix`
   - 추가 입력란(`user_config_html`): MQTT 서버 주소(`broker`)
3. 저장된 `ssid`, `w_pw`로 WiFi 접속
4. DHT22, I2C(BH1750), TFT 초기화. 화면에 `Thermostat`, `Temp`, `Humi`, `Lux` 라벨 표시
5. `broker` 값으로 MQTT 서버(포트 1883)에 클라이언트 ID `MQTTSensor`로 접속, 실패 시 2초 후 재시도

### 주기적 발행

- `pubInterval` = 5000 ms마다 DHT22와 BH1750 값을 읽어 TFT 값 영역을 갱신하고 발행한다

| 토픽 | 페이로드 |
|------|----------|
| `id/LeeDH/sensor/evt/temperature` | 온도, 소수 첫째 자리 |
| `id/LeeDH/sensor/evt/humidity` | 습도, 소수 첫째 자리 |
| `id/LeeDH/sensor/evt/lux` | 조도, 소수 둘째 자리 |
| `id/LeeDH/sensor/data` | `{"temperature":<온도>,"humidity":<습도>}` |

## 개발 환경

- PlatformIO, platform `espressif32`, framework `arduino`
- build_flags: `-DARDUINO_USB_CDC_ON_BOOT=1`
- lib_deps
  - `bodmer/TFT_eSPI@^2.5.43`
  - `beegee-tokyo/DHT sensor library for ESPx@^1.19`
  - `claws/BH1750@^1.3.0`
  - `knolleary/PubSubClient@^2.8`
  - `yhur/ConfigPortal32`
- 업로드·모니터 속도 115200

## 설정

- WiFi·브로커 주소 : 코드 수정 없이 첫 부팅 시 설정 포털에서 입력
- `ssid_pfix` : 설정 포털 AP 이름 접두어
- `mqttPort`, `pubInterval` : 브로커 포트, 발행 주기
- 토픽 문자열 : `pubStatus()` 안에서 수정
- `platformio.ini`의 `upload_port`, `monitor_port` : 자신의 PC에서 보드가 잡힌 COM 포트로 변경

## 빌드 및 실행

```bash
pio run -t upload
pio device monitor -b 115200
```

## 폴더 구조

```
MQTT_Sensor/
├── platformio.ini   # T-Display S3 보드·포트·라이브러리 설정
└── src/
    └── main.cpp     # 설정 포털 + 센서 측정 + TFT 표시 + MQTT 발행
```
