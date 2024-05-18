## Лабораторная №8


# Tasks
1. Создать публичный репозиторий с названием lab08 на сервисе GitHub
2. Ознакомиться со ссылками учебного материала
3. Выполнить инструкцию учебного материала
4. Составить отчет и отправить ссылку личным сообщением в Slack

# Выполнение:
Клонируем репозиторий 6 лабораторной работы:
```bash
Команда:git clone https://github.com/kuznetsovvvv/lab06 lab08
```

Удаляем файлы, связанные с CPack.

Создаём Dockerfile и пишем в него:
```bash
FROM ubuntu:18.04

RUN apt update
RUN apt install -yy gcc g++ cmake

COPY . print/
WORKDIR print

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build

ENV LOG_PATH /home/logs/log.txt

VOLUME /home/logs

WORKDIR _install/bin

ENTRYPOINT ./demo
```

В дирректории .github/workflows удаляем старые файлы сборки, создаем CI.yml, пишем с помощью текстового редактора nano:

```bash
name: CMake

on:
 push:
  branches: [master]
 pull_request:
  branches: [master]

jobs: 
 build_Linux:

  runs-on: ubuntu-latest

  steps:
  - uses: actions/checkout@v3

  - name: Install Docker
    run: sudo apt install runc containerd docker.io

  - name: Run Docker
    run: sudo docker build -t logger .
```

*Проверяем в GithubActions результаты выполнения.












