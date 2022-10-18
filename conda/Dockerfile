FROM continuumio/miniconda3:4.10.3p0-alpine as alpine-glibc-miniconda3-xfce-vnc

### base

ENV DISPLAY :1
# alternative 1024x768x16
ENV RESOLUTION 1920x1080x24

# setup desktop environment (xfce4), display server (xvfb), vnc server (x11vnc)
RUN apk add --no-cache \
  xfce4 \
  faenza-icon-theme \
  xvfb \
  x11vnc

# setup novnc
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing \
  novnc && \
  ln -s /usr/share/novnc/vnc.html /usr/share/novnc/index.html
RUN pip3 install --no-cache-dir websockify

# required for command expansion
RUN apk add --no-cache \
  bash
SHELL ["/bin/bash", "-c"]

# setup supervisor
COPY supervisor /tmp
RUN apk add --no-cache supervisor && \
  echo_supervisord_conf > /etc/supervisord.conf && \
  sed -i -r -f /tmp/supervisor.sed /etc/supervisord.conf && \
  mkdir -pv /etc/supervisor/conf.d /var/log/{novnc,x11vnc,xfce4,xvfb} && \
  mv /tmp/supervisor-*.ini /etc/supervisor/conf.d/ && \
  rm -fr /tmp/supervisor*

CMD ["supervisord", "-c", "/etc/supervisord.conf", "-n"]

### web

FROM alpine-glibc-miniconda3-xfce-vnc

# timezone
# https://gitlab.alpinelinux.org/alpine/aports/-/issues/5543
# /usr/share/zoneinfo/Europe/Dublin
ARG TZ='Europe/Dublin'
ENV DEFAULT_TZ ${TZ}
RUN apk upgrade --update && \
  apk add --no-cache tzdata && \
  cp /usr/share/zoneinfo/${DEFAULT_TZ} /etc/localtime && \
  date

# commons
RUN apk add --no-cache \
  git \
  vim \
  zip

# python and pip
# http://github.com/Docker-Hub-frolvlad/docker-alpine-python3
# ENV PYTHONUNBUFFERED=1
# RUN apk add --no-cache \
#   python3 && \
#   ln -sf python3 /usr/bin/python && \
#   python3 -m ensurepip && \
#   rm -r /usr/lib/python*/ensurepip && \
#   pip3 install --no-cache --upgrade pip setuptools wheel && \
#   ln -sf pip3 /usr/bin/pip

# gcc
RUN apk add --no-cache \
  musl-dev \
  gcc \
  make

# networking
RUN apk add --no-cache \
  bind-tools \
  curl \
  httpie \
  jq

# java
RUN apk add --no-cache \
  openjdk11-jdk

# golang
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/community \
  go

# rust
# RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y

# filesystem
RUN apk add --no-cache \
  exa \
  fd \
  ripgrep

# monitoring
RUN apk add --no-cache \
  ncdu \
  htop

# browsers
RUN apk add --no-cache \
  firefox-esr \
  lynx

# terminals
RUN apk add --no-cache \
  xfce4-terminal \
  tmux \
  neofetch

# image viewer
# TODO nomacs | viu | imv

# update configs
COPY .profile .bashrc .vimrc /root/
ENV ENV="/root/.profile"

WORKDIR /root