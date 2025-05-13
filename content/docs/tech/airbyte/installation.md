---
title: "Installation"
description: "해당 문서에서는 docker-compose를 사용하여 airbyte를 설치하는 방법에 대해서 설명합니다."
weight: 1
---

# Airbyte 설치 방법

> 해당 문서에서는 docker-compose를 사용하여 airbyte를 설치하는 방법에 대해서 설명합니다.

### 선행조건

* docker 및 docker-compose 설치
* git 설치(`sudo apt-get install git`)

### Airbyte 다운로드

```bash
git clone https://github.com/airbytehq/airbyte.git
```

### 스크립트 실행

* Airbyte에서는 설치 및 배포 스크립트를 제공합니다.
* docker-compose 및 .env (Airbyte 설정 파일) 을 다운로드 합니다.

```
options:
   -d --download    Only download files - don't run docker compose
   -r --refresh     DELETE existing assets and re-download new ones
   -h --help        Print this Help.
   -x --debug       Verbose mode.
   -b --background  Run docker compose up in detached mode.
```

```bash
cd airbyte
./run-ab-platform.sh -d
```

### Airbyte 환경 설정

* docker-compose 실행 이전, 운영하려는 환경에 대한 설정을 진행합니다.
  * Core, Secrets(Vault, GCP Secret store, AWS KMS), Database, Airbyte Service, Jobs, Connection, Logging, Monitoring, Worker, Launcher, Data Retention, docker, k8s 옵션 등 다양하게 존재하기 때문에 Reference을 보면서 설정하는 것이 좋습니다.
  * [https://docs.airbyte.com/operator-guides/configuring-airbyte](https://docs.airbyte.com/operator-guides/configuring-airbyte)

### docker-compose 실행

```bash
docker compose up -d (./run-ab-platform.sh -b)
```

### Airbyte 접속

```bash
http://localhost:8000/
```

### Reference

* https://docs.airbyte.com/operator-guides/configuring-airbyte
