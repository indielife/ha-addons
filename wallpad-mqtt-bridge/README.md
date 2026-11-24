# Wallpad MQTT Bridge — Home Assistant Add-on

RS485 월패드와 환기장치를 MQTT로 연결하는 Home Assistant 애드온입니다.
[Indielife 애드온 저장소](../README.md)에서 제공하는 애드온 중 하나입니다.

## 설치

1. [저장소 등록 안내](../README.md#저장소-등록)에 따라 애드온 저장소를 추가합니다.
2. 애드온 스토어에서 **Wallpad MQTT Bridge**를 선택하여 설치합니다.
3. 아래 설정 가이드에 따라 MQTT 브로커와 월패드 연결 정보를 설정한 뒤 시작합니다.
4. 로그에서 기동과 MQTT 연결을 확인하고, Home Assistant의 기기 등록을 확인합니다.

## 릴리즈와 변경 내역

현재 애드온 버전 `0.1.0`의 기능·제약·검증 근거는 [GitHub 릴리즈][release]에서 확인합니다.
HA에서 표시할 버전별 요약은 [변경 내역](CHANGELOG.md)으로 제공합니다.

난방 조절기에 CC 오류가 표시되고 벽 조절기·월패드에서도 제어할 수 없는 현상이
보고되었습니다. [원인은 조사 중][heating]입니다.

## 지원 제조사 (Supported Manufacturers)

현재 이 애드온이 지원하는 월패드 제조사는 다음과 같습니다.
- **Kocom (코콤)**

## 설정 가이드 (Configuration)

애드온 설정 탭에서 다음과 같이 기본 설정 항목을 입력합니다.

### 1. MQTT Broker
홈어시스턴트에서 사용하는 MQTT 브로커(예: Mosquitto Broker 애드온)와의 연결 정보입니다.
- **Host**: MQTT 브로커 호스트. HA 내부 Mosquitto 브로커 애드온을 사용하는 경우 `core-mosquitto`를 입력합니다. 외부 브로커는 IP 주소로 입력합니다. (기본값: `core-mosquitto`)
- **Username**: MQTT 사용자 계정 ID
- **Password**: MQTT 사용자 계정 비밀번호

### 2. Wallpad
월패드 제조사 및 연결 정보 설정입니다.
- **Manufacturer**: 월패드 제조사 명칭. 현재는 `kocom`만 지원됩니다. (기본값: `kocom`)
- **Connection Type**: 월패드와 통신하는 물리적인 연결 방식 선택 (`Serial` 또는 `Socket`)
  - `Serial`: RS485 라인이 USB 시리얼 젠더를 통해 홈어시스턴트 서버(라즈베리 파이, 미니PC 등)에 직접 연결되어 있는 경우 사용합니다.
  - `Socket`: RS485 라인이 EW11 등 시리얼-네트워크 변환기에 연결되어 네트워크를 통해 통신하는 경우 사용합니다.
- **Socket**: `Connection Type`을 `Socket`으로 설정한 경우 활성화되는 네트워크 연결 설정입니다.
  - **Host**: 시리얼-네트워크 변환기(예: Elfin EW11)의 IP 주소를 입력합니다. (예: `192.168.0.11`)
  - **Port**: 변환기에 설정한 소켓 포트 번호를 입력합니다. (기본값: `8899`)
- **Serial**: `Connection Type`을 `Serial`로 설정한 경우 활성화되는 시리얼 포트 설정입니다.
  - **Port**: 홈어시스턴트 서버에 연결된 USB 시리얼 디바이스 경로를 입력합니다. (예: `/dev/ttyUSB0`)

### 3. Enabled Devices
방 개념이 없는 집 전체 단위 기기를 활성화하는 설정입니다. (활성화할 기기는 `true`로 설정)
- **fan**: 환기팬
- **gas**: 가스 밸브
- **elevator**: 엘리베이터 호출

> **조명 · 플러그 · 난방**은 아래 `Rooms` 설정에서 방별로 구성합니다.

### 4. Rooms
방마다 설치된 기기 구성을 입력합니다. `wallpad` 섹션 하위에 위치합니다.

각 방은 다음 항목으로 구성됩니다.

| 항목 | 설명 |
|------|------|
| **name** | 방 이름 (예: `livingroom`, `bedroom`, `kitchen`) |
| **room_no** | 조명·콘센트 패킷 번호 (0부터 시작). 생략 시 조명·콘센트 없음 |
| **light_count** | 조명 개수 |
| **plug_count** | 콘센트 개수 |
| **thermo_no** | 난방 패킷 번호. 조명과 다를 수 있음. 생략 시 난방 없음 |

> **패킷 번호 확인 방법**: 월패드에서 직접 기기를 조작하면서 RS485 패킷을 캡쳐하면 방별 번호를 확인할 수 있습니다. `room_no`와 `thermo_no`가 다른 것은 코콤 배선 특성으로 정상입니다.

설정 예시:
```yaml
rooms:
  - name: livingroom
    room_no: 0       # 조명/콘센트 패킷 주소 → "00"
    light_count: 3
    plug_count: 2
    thermo_no: 0     # 난방 패킷 주소 → "00"
  - name: room1
    room_no: 3       # 조명 패킷 주소 → "03"
    light_count: 2
    plug_count: 2
    thermo_no: 2     # 난방 패킷 주소 → "02" (조명과 다름)
  - name: kitchen
    room_no: 4
    light_count: 3
    plug_count: 2
    # thermo_no 생략 → 이 방은 난방 없음
```

### 5. Ventilator

전열교환기(환기장치) 연동 설정입니다.

현재 버전(`v0.1.0`)에서 환기장치는 **Serial 연결만 지원**합니다.
월패드의 Socket 연결 지원과 별개이며, 환기장치를 활성화한 상태에서
`connection_type: Socket`으로 설정하면 설정 검증 오류로 애드온이 시작되지 않습니다.

- **Manufacturer**: 전열교환기 제조사 선택 (`None`, `Grex` 중 선택. 기본값: `None`)
- **Connection Type**: `Serial`로 설정합니다.
- **Serial**: USB 시리얼 젠더로 연결된 벽면 조절기와 환기장치 본체의 포트를 지정합니다.
  - **Controller Port**: 전열교환기 벽면 조절기 라인의 디바이스 경로 (예: `/dev/ttyUSB1`)
  - **Ventilator Port**: 전열교환기 본체 라인의 디바이스 경로 (예: `/dev/ttyUSB2`)
- **Default Speed**: 기본 환기 속도 선택 (`low`, `medium`, `high` 등. 기본값: `low`)

## 소스 버전과 빌드

이 애드온의 `config.yaml` 버전 `0.1.0`은
[소스 저장소](https://github.com/indielife/wallpad-mqtt-bridge)의 `v0.1.0` 태그에 대응합니다.
Home Assistant가 전달하는 `BUILD_VERSION` 앞에 `v`를 붙여 해당 태그를 가져옵니다.
GitHub 릴리즈 발행 없이도 소스 태그가 있으면 빌드할 수 있습니다.

`ha-addons` 저장소 루트에서 HA와 같은 버전 인수로 빌드를 검증합니다.

```bash
docker build --build-arg BUILD_VERSION=0.1.0 \
  -t wallpad-mqtt-bridge-addon:0.1.0 ./wallpad-mqtt-bridge
```

빌드 인수를 생략하면 Dockerfile의 기본 버전 `0.1.0`을 사용합니다.
새 버전은 [릴리즈 절차][release-process]에 따라 릴리즈 노트를 먼저 작성·검토하고,
최종 검증 후 소스 저장소에 `v<버전>` 태그와 GitHub 릴리즈를 발행합니다.
이후 이 애드온의 `config.yaml`과 Dockerfile 기본 버전,
태그 기준 문서 링크·`CHANGELOG.md`·릴리즈 링크를 함께 갱신합니다.
애드온 저장소 `main`에 반영한 뒤 HA 저장소를 새로고침하여 버전과 변경 내역을 확인하고,
애드온 변경이 있으면 업데이트·정상 기동을 확인합니다.
이미 배포한 소스 태그는 다른 커밋으로 이동하지 않습니다.

로컬 이미지 빌드는 HA의 저장소 등록·설치·정상 기동 검증과 별도로 확인해야 합니다.

## 문서 (Documentation)

애드온이 사용하는 소스 버전 `v0.1.0`의 상세 가이드입니다.

- [시스템 아키텍처 가이드][arch]: MQTT-RS485 브릿지의 핵심 컴포넌트와 패킷 흐름
- [하드웨어 연결 가이드][hardware]: RS485 연결 방식 및 기기별 결선 아키텍처
- [로컬 개발 가이드][development]: 가상환경 구성, 패키지 관리, 포매팅·린트 및 Docker 빌드
- [코콤 패킷 분석 가이드][protocol]: 코콤 월패드 통신 패킷 구조 및 기기별 Hex 코드 정의

[arch]: https://github.com/indielife/wallpad-mqtt-bridge/blob/v0.1.0/docs/architecture.md
[hardware]: https://github.com/indielife/wallpad-mqtt-bridge/blob/v0.1.0/docs/hardware_connection.md
[development]: https://github.com/indielife/wallpad-mqtt-bridge/blob/v0.1.0/docs/development.md
[protocol]: https://github.com/indielife/wallpad-mqtt-bridge/blob/v0.1.0/docs/packet_analysis.md
[release]: https://github.com/indielife/wallpad-mqtt-bridge/releases/tag/v0.1.0
[heating]: https://github.com/indielife/wallpad-mqtt-bridge/issues/184
[release-process]: https://github.com/indielife/wallpad-mqtt-bridge/blob/main/docs/release.md

## Credits & Acknowledgements

이 프로젝트는 [Zoolian/zooil]님의 [kocomRS485](https://github.com/zooil/kocomRS485) 프로젝트를 기반으로 새롭게 구성하고 수정한 버전입니다.
좋은 코드를 공유해 주신 원작자분께 감사드립니다.
