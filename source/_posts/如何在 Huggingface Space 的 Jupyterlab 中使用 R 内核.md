---
abbrlink: e592c398
categories: []
date: '2024-07-31T23:24:45.790877+08:00'
tags: []
title: 如何在 Huggingface Space 的 Jupyterlab 中使用 R 内核
updated: '2024-07-31T23:24:57.808+08:00'
---
# 前言

相信有喜欢折腾的小伙伴见到 Huggingface 的 Space 新建菜单中看到了 Docker 下有快速构建 Jupyterlab 的按钮，我也十分好稀奇，因为有 Jupyterlab 的情况下可以添加 R 核心。当然，情况稍微有一点变化。

# 正文

Huggingface Space 的 Jupyterlab 建构下的文件结构类似于这样：

```
- Dockerfile
- README.md
- login.html
- on_startup.sh
- packages.txt
- requirements.txt
- start_server.sh
```

其中，`on_startup.sh`会在 build 环节下自动执行 bash 指令，也许有人想到了可以在这里自动化安装核心，但是很可惜的是，这个执行是在 root 用户下的，并不会影响到 Jupyterlab 本身。

想必你已经注意到了，Space 托管的仓库实际上只有一个前端，而后端在 Dockerfile 中进行部署，所以你可以修改 Dockerfile 完成添加核心支持。
下面是我的 Dockerfile，你可以照抄，重构建后就会支持 R 核心了。

```plaintext
FROM nvidia/cuda:11.3.1-base-ubuntu20.04

ENV DEBIAN_FRONTEND=noninteractive \
    TZ=Europe/Paris

# Remove any third-party apt sources to avoid issues with expiring keys.
# Install some basic utilities
RUN rm -f /etc/apt/sources.list.d/*.list && \
    apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    sudo \
    git \
    wget \
    procps \
    git-lfs \
    zip \
    unzip \
    htop \
    vim \
    nano \
    bzip2 \
    libx11-6 \
    build-essential \
    libsndfile-dev \
    software-properties-common \
 && rm -rf /var/lib/apt/lists/*

RUN add-apt-repository ppa:flexiondotorg/nvtop && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends nvtop

RUN curl -sL https://deb.nodesource.com/setup_14.x  | bash - && \
    apt-get install -y nodejs && \
    npm install -g configurable-http-proxy

# Install R and IRKernel dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    r-base \
    r-base-dev \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev

# Create a working directory
WORKDIR /app

# Create a non-root user and switch to it
RUN adduser --disabled-password --gecos '' --shell /bin/bash user \
 && chown -R user:user /app
RUN echo "user ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-user
USER user

# All users can use /home/user as their home directory
ENV HOME=/home/user
RUN mkdir $HOME/.cache $HOME/.config \
 && chmod -R 777 $HOME

# Set up the Conda environment
ENV CONDA_AUTO_UPDATE_CONDA=false \
    PATH=$HOME/miniconda/bin:$PATH
RUN curl -sLo ~/miniconda.sh https://repo.continuum.io/miniconda/Miniconda3-py39_4.10.3-Linux-x86_64.sh \
 && chmod +x ~/miniconda.sh \
 && ~/miniconda.sh -b -p ~/miniconda \
 && rm ~/miniconda.sh \
 && conda clean -ya

WORKDIR $HOME/app

# Install IRKernel in the Conda environment
RUN conda install -c r r-irkernel

#######################################
# Start root user section
#######################################

USER root

# User Debian packages
## Security warning : Potential user code executed as root (build time)
RUN --mount=target=/root/packages.txt,source=packages.txt \
    apt-get update && \
    xargs -r -a /root/packages.txt apt-get install -y --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

RUN --mount=target=/root/on_startup.sh,source=on_startup.sh,readwrite \
    bash /root/on_startup.sh

RUN mkdir /data && chown user:user /data

#######################################
# End root user section
#######################################

USER user

# Python packages
RUN --mount=target=requirements.txt,source=requirements.txt \
    pip install --no-cache-dir --upgrade -r requirements.txt

# Copy the current directory contents into the container at $HOME/app setting the owner to the user
COPY --chown=user . $HOME/app

RUN chmod +x start_server.sh

COPY --chown=user login.html /home/user/miniconda/lib/python3.9/site-packages/jupyter_server/templates/login.html

# Install R packages required for IRKernel
RUN R -e "install.packages(c('repr', 'IRdisplay', 'evaluate', 'crayon', 'pbdZMQ', 'uuid', 'digest'), repos='https://cloud.r-project.org/')"
RUN R -e "IRkernel::installspec(user = FALSE)"

ENV PYTHONUNBUFFERED=1 \
    GRADIO_ALLOW_FLAGGING=never \
    GRADIO_NUM_PORTS=1 \
    GRADIO_SERVER_NAME=0.0.0.0 \
    GRADIO_THEME=huggingface \
    SYSTEM=spaces \
    SHELL=/bin/bash

CMD ["./start_server.sh"]
```

顺便，修改你的 `start_server.sh` 中`NOTEBOOK\_DIR="\~/app"`，这样可以在 Jupyterlab 启动后自动打开 Repo 的路径。
