# https://hub.docker.com/r/hiyouga/pytorch/tags
ARG BASE_IMAGE=hiyouga/pytorch:th2.6.0-cu124-flashattn2.7.4-cxx11abi0-devel
FROM ${BASE_IMAGE}

# Installation arguments
# ARG PIP_INDEX=https://pypi.org/simple
# 清华镜像源 (直接在pip命令中使用硬编码URL，不再需要ARG)
# ARG PIP_INDEX=https://pypi.tuna.tsinghua.edu.cn/simple
ARG EXTRAS=metrics
ARG INSTALL_FLASHATTN=false
ARG HTTP_PROXY=""

# Define environments
ENV MAX_JOBS=16
ENV FLASH_ATTENTION_FORCE_BUILD=TRUE
ENV VLLM_WORKER_MULTIPROC_METHOD=spawn
ENV DEBIAN_FRONTEND=noninteractive
ENV NODE_OPTIONS=""
ENV PIP_ROOT_USER_ACTION=ignore
ENV http_proxy="${HTTP_PROXY}"
ENV https_proxy="${HTTP_PROXY}"

# Set Chinese locale environment variables
ENV LANG=zh_CN.UTF-8
ENV LC_ALL=zh_CN.UTF-8
ENV LANGUAGE=zh_CN:zh
ENV TZ=Asia/Shanghai

# Use Bash instead of default /bin/sh
SHELL ["/bin/bash", "-c"]

# Set the working directory
WORKDIR /app

# Install Chinese language support and fonts
RUN apt-get update && apt-get install -y \
    locales \
    fonts-noto-cjk \
    language-pack-zh-hans \
    openssh-server \
    zip \
    unzip \
    tree \
    vim \
    tzdata \
    apt-utils \
    && rm -rf /var/lib/apt/lists/*

# Generate Chinese locale and set timezone
RUN locale-gen zh_CN.UTF-8 && \
    update-locale LANG=zh_CN.UTF-8 LC_ALL=zh_CN.UTF-8 && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "export LANG=zh_CN.UTF-8" >> /etc/bash.bashrc && \
    echo "export LC_ALL=zh_CN.UTF-8" >> /etc/bash.bashrc

# Change pip source
RUN pip config set global.index-url "https://pypi.tuna.tsinghua.edu.cn/simple" && \
    pip config set global.extra-index-url "https://pypi.tuna.tsinghua.edu.cn/simple" && \
    pip install --no-cache-dir -i "https://pypi.tuna.tsinghua.edu.cn/simple" --upgrade pip packaging wheel setuptools

# Install the requirements
COPY requirements.txt /app
RUN pip install --no-cache-dir -i "https://pypi.tuna.tsinghua.edu.cn/simple" -r requirements.txt

# Copy the rest of the application into the image
COPY . /app

# Install LLaMA Factory (使用 --upgrade-strategy only-if-needed 避免重新安装 PyTorch)
RUN pip install --no-cache-dir -i "https://pypi.tuna.tsinghua.edu.cn/simple" \
    -e ".[${EXTRAS}]" --no-build-isolation --upgrade-strategy only-if-needed

# Rebuild flash attention
RUN if [ "${INSTALL_FLASHATTN}" == "true" ]; then \
        pip uninstall -y ninja && \
        pip install --no-cache-dir -i "https://pypi.tuna.tsinghua.edu.cn/simple" ninja && \
        pip install --no-cache-dir -i "https://pypi.tuna.tsinghua.edu.cn/simple" flash-attn --no-build-isolation; \
    fi

# RUN pip uninstall pillow -y && CC="cc -mavx2" pip install -U --force-reinstall pillow-simd -i "https://pypi.tuna.tsinghua.edu.cn/simple" --use-pep517

# Install additional development tools
RUN apt-get update && apt-get install -y \
    htop \
    tmux \
    curl \
    wget \
    git \
    file \
    net-tools \
    libibverbs1 \
    libibverbs-dev \
    libgl1-mesa-glx libglx-mesa0 \
    && rm -rf /var/lib/apt/lists/*

# Customize bash prompt
RUN echo 'export PS1="\\[\\e[1;31m\\]\\D{%a %m月 %d} \\[\\e[0m\\]# \\[\\e[1;32m\\]\\t \\[\\e[0m\\]# \\[\\e[1;34m\\]\\w \\[\\e[0m\\]# "' >> /root/.bashrc

# Set up volumes
# VOLUME [ "/root/.cache/huggingface", "/app/shared_data", "/app/output" ]

# Expose port 7860 for LLaMA Board
ENV GRADIO_SERVER_PORT=7860
EXPOSE 7860

# Expose port 8000 for API service
ENV API_PORT=8000
EXPOSE 8000

# unset proxy
ENV http_proxy=
ENV https_proxy=

# Reset pip config
RUN pip config unset global.index-url && \
    pip config unset global.extra-index-url
