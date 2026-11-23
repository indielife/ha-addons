# Indielife Home Assistant 애드온

RS485 월패드와 환기장치를 MQTT로 연결하는 Wallpad MQTT Bridge 애드온 저장소입니다.

## 설치

1. Home Assistant의 애드온 스토어에서 저장소 관리 화면을 엽니다.
2. `https://github.com/indielife/ha-addons`를 추가합니다.
3. 목록의 **Wallpad MQTT Bridge**를 선택하여 설치합니다.
4. MQTT 브로커와 월패드 연결 정보를 설정한 뒤 시작합니다.
5. 로그에서 기동과 MQTT 연결을 확인하고, Home Assistant의 기기 등록을 확인합니다.

설정 항목은 [애드온 설정 가이드](wallpad-mqtt-bridge/README.md)를 참고하세요.

## 소스 버전과 빌드

애드온의 `config.yaml` 버전 `0.1.0`은
[소스 저장소](https://github.com/indielife/wallpad-mqtt-bridge)의 `v0.1.0` 태그에 대응합니다.
Home Assistant가 전달하는 `BUILD_VERSION` 앞에 `v`를 붙여 해당 태그를 가져옵니다.
GitHub 릴리즈 발행 없이도 소스 태그가 있으면 빌드할 수 있습니다.

저장소 루트에서 HA와 같은 버전 인수로 빌드를 검증합니다.

```bash
docker build --build-arg BUILD_VERSION=0.1.0 \
  -t wallpad-mqtt-bridge-addon:0.1.0 ./wallpad-mqtt-bridge
```

빌드 인수를 생략하면 Dockerfile의 기본 버전 `0.1.0`을 사용합니다.
새 버전 배포 시 소스 저장소에 `v<버전>` 태그를 먼저 생성하고,
애드온 `config.yaml`과 Dockerfile 기본 버전을 함께 갱신합니다.
이미 배포한 소스 태그는 다른 커밋으로 이동하지 않습니다.

로컬 이미지 빌드는 HA의 저장소 등록·설치·정상 기동 검증과 별도로 확인해야 합니다.
