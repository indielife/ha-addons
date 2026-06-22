FROM python:3.12-slim

ENV LANG=C.UTF-8

# Install git
RUN apt-get update && apt-get install -y --no-install-recommends git && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Build arguments
ARG BUILD_VERSION=main

# Clone wallpad-mqtt-bridge
RUN git clone --depth 1 --branch ${BUILD_VERSION} https://github.com/indielife/wallpad-mqtt-bridge.git . \
    && pip install --no-cache-dir .

# Addon config
COPY config.yaml ./
COPY translations/ ./translations/

# Entrypoint
COPY run.sh ./
RUN chmod +x run.sh

CMD ["./run.sh"]
