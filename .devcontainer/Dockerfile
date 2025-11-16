FROM mcr.microsoft.com/devcontainers/python:3.10-bullseye

# Replace apt sources with a fast mirror in China (Aliyun)
RUN cat > /etc/apt/sources.list <<'EOF'
deb http://mirrors.aliyun.com/debian/ bullseye main contrib non-free
deb-src http://mirrors.aliyun.com/debian/ bullseye main contrib non-free
deb http://mirrors.aliyun.com/debian-security bullseye-security main
deb-src http://mirrors.aliyun.com/debian-security bullseye-security main
deb http://mirrors.aliyun.com/debian/ bullseye-updates main contrib non-free
deb-src http://mirrors.aliyun.com/debian/ bullseye-updates main contrib non-free
EOF

ENV DEBIAN_FRONTEND=noninteractive

# Install minimal system tools (git, curl, ca-certificates) using apt-get with retries
RUN set -eux; \
    retry() { n=0; until [ "$n" -ge 5 ]; do "$@" && break || n=$((n+1)) && sleep 5; done }; \
    retry apt-get update; \
    retry apt-get install -y --no-install-recommends git curl ca-certificates ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Configure pip to use a fast PyPI mirror for China
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# Install JupyterLab (with retry)
RUN set -eux; retry() { n=0; until [ "$n" -ge 5 ]; do "$@" && break || n=$((n+1)) && sleep 5; done }; \
    retry pip install --no-cache-dir jupyterlab

# Copy requirements (build context is workspace root) and install
COPY PixelRefer/requirements.txt /tmp/requirements.txt
RUN set -eux; retry() { n=0; until [ "$n" -ge 5 ]; do "$@" && break || n=$((n+1)) && sleep 5; done }; \
    retry pip install --no-cache-dir -r /tmp/requirements.txt

# Clone sam2 and install (no CUDA build) — shallow clone with retry and fallback URL environment
ARG SAM2_GIT_URL="https://github.com/facebookresearch/sam2.git"
RUN set -eux; \
    retry() { n=0; until [ "$n" -ge 5 ]; do "$@" && break || n=$((n+1)) && sleep 5; done }; \
    retry git clone --depth 1 "$SAM2_GIT_URL" /tmp/sam2 || (echo 'Primary clone failed, trying fastgit mirror' && retry git clone --depth 1 "https://hub.fastgit.org/facebookresearch/sam2.git" /tmp/sam2); \
    cd /tmp/sam2 && SAM2_BUILD_CUDA=0 pip install --no-cache-dir -e . && \
    rm -rf /tmp/sam2

# Install segment-anything with retry
RUN set -eux; retry() { n=0; until [ "$n" -ge 5 ]; do "$@" && break || n=$((n+1)) && sleep 5; done }; \
    retry pip install --no-cache-dir segment-anything

# Set workspace and default user
WORKDIR /workspace/PixelRefer
USER vscode