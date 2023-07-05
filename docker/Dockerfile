# syntax = docker/dockerfile:experimental
FROM ubuntu:18.04

RUN apt-get update && \
    apt-get install -y \
    git \
    ssh

# rootユーザを使用しないようにする
ARG USERNAME=test
ARG GROUPNAME=test
ARG UID=1000
ARG GID=1000
ENV HOME /home/$USERNAME

RUN groupadd -g $GID $GROUPNAME && \
    useradd -m -s /bin/bash -u $UID -g $GID $USERNAME && \
    chown -R $GROUPNAME:$USERNAME $HOME
USER $USERNAME

WORKDIR $HOME

# プライベートリポジトリをクローン
RUN mkdir -p -m 0700 ~/.ssh
RUN ssh-keyscan github.com > ~/.ssh/known_hosts 
RUN --mount=type=ssh,uid=1000 git clone git@github.com:KohsukeFuchi/test_repository.git
# RUN ssh -T git@github.com
