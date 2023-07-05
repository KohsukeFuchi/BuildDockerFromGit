# Dockerビルド時にプライベートリポジトリをCloneする

参考：[Dockerビルド時にプライベートリポジトリをクローンする](https://zenn.dev/ryo_f/articles/27f223203481ef)

ここでは，Dockerビルド時に，プライベートリポジトリをCloneして使用する方法を紹介する。

## 1. GitHubとSSH接続できるように設定する
ローカル環境で以下のようにして秘密鍵と公開鍵を作成する。

```bash
> mkdir ~/.ssh
> cd ~/.ssh
> ssh-keygen -t rsa
> ls
id_rsa		id_rsa.pub
```

ここで，`id_rsa`が秘密鍵，`id_rsa.pub`が公開鍵となる。
権限を変更しなければ使用できないため，権限も`chmod`コマンドで変更する。

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
```

そして，GitHubに公開鍵を登録する。
登録は，マイページの[settings] - [SSH and GPG keys] - [New SSH key]で登録できる。

## 2. Dockerfileを作成する
適当なDockerfileを作成する。

注意点として，まずrootユーザを使用しないほうがいいとある。([参考](https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html#user))
そのため，別のユーザを作成する必要がある。
```dockerfile
ARG USERNAME=test
ARG GROUPNAME=test
ARG UID=1000
ARG GID=1000
ENV HOME /home/$USER

RUN groupadd -g $GID $GROUPNAME && \
    useradd -m -s /bin/bash -u $UID -g $GID $USERNAME && \
    chown -R $GROUPNAME:$USERNAME $HOME
USER $USERNAME

WORKDIR $HOME
```

以下のようにしてGitをCloneする
```dockerfile
RUN mkdir -p -m 0700 ~/.ssh
RUN ssh-keyscan github.com > ~/.ssh/known_hosts 
RUN --mount=type=ssh,uid=1000 git clone git@github.com:[username]/[repository name].git
```

## 3. Dockerをビルドする

以下のコマンドでDockerをビルドする。
```bash
DOCKER_BUILDKIT=1 docker build -t test:1.0 . --ssh default=/Users/username/.ssh/id_rsa 
```

`--ssh`に指定する秘密鍵は**フルパスで！**


