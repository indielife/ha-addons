FROM python:3.12-slim

ENV LANG=C.UTF-8

# Install git
RUN apt-get update && apt-get install -y --no-install-recommends git && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# HA가 config.yaml의 version을 전달하며, 직접 빌드할 때도 같은 버전을 사용합니다.
ARG BUILD_VERSION=0.1.0

# 버전에 대응하는 소스 태그로 고정합니다.
RUN git clone --depth 1 --branch "v${BUILD_VERSION}" https://github.com/indielife/wallpad-mqtt-bridge.git . \
    && pip install --no-cache-dir .

# Addon config
COPY config.yaml ./
COPY translations/ ./translations/

# Entrypoint
COPY run.sh ./
RUN chmod +x run.sh

CMD ["./run.sh"]
