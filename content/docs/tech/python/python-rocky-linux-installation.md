---
title: "폐쇄망에 Python 설치 (Rocky Linux 8.10, UV 사용)"
description: ""
---

# 폐쇄망에 Python 설치 (Rocky Linux 8.10, UV 사용)

* 폐쇄망 환경에 Python 환경을 배포하기 위한 과정을 정리하기 위해서 작성하였습니다.
* Python 가상 환경 및 Dependency 관리는 UV로 진행합니다. 그래서 UV 설치 과정이 포함되어 있습니다.&#x20;
* Rochy-Linux 8.10, Python 3.12.8, UV 0.5.18 기준으로 작성하였습니다.

## 1. dnf를 사용한 Python 설치

### 1-1. 파일 다운로드

#### Python 검색

* dnf를 사용하여 검색할 수 있는 Python 버전은 3.8, 3.9, 3.11, 3.12 입니다.
* Python 3.10은 dnf search 및 [Red Hat 문서](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/9/html/installing_and_using_dynamic_programming_languages/assembly_installing-and-using-python_installing-and-using-dynamic-programming-languages#assembly_installing-and-using-python_installing-and-using-dynamic-programming-languages)에서 확인할 수 없었습니다.

```sh
dnf search python
```

#### Python 설치 파일 다운로드

* download-path에 Python을 다운로드 받을 폴더를 지정합니다.

```sh

dnf install --downloaddir {download-path} --downloadonly python3.12 python3.12-pip python3.12-devel  -y

```

### 1-2. Python 설치

* 다운로드 받은 파일은 rpm 형태로 구성되어 있습니다. 지정한 download-path에 들어가서 설치 진행합니다.

```sh

rpm -ivh *.rpm

```

### 1-3. Python 설치 확인

* 설치가 되었다면 정상적으로 설치되었는 지 확인합니다. python  명령어로만 실행 시에는 에러가 발생합니다.

```sh
python3 -V # OK
python3.12 -V # OK
python # ERROR
```

## 2. UV 설치

### 2-1. 파일 다운로드

* Github Repo에 들어가서 UV Release를 확인하고, OS Version에 맞는 설치 파일을 다운로드 받습니다.

```sh

wget https://github.com/astral-sh/uv/releases/download/0.5.18/uv-x86_64-unknown-linux-gnu.tar.gz

```

### 2-2. UV 설치

* 다운로드 받은 UV 파일을 압축 해제하고 해제한 파일들은 shell에서 사용할 수 있도록 조치해야 합니다.
* 압축 해제 후 보안 이슈가 없다면 `/usr/bin` 에 파일들을 욺겨서 사용할 수 있도록 합니다.

> UV 스크립트로 설치하면 `$HOME/.local/bin/env`에 설치된 파일들이 추가되고 사용하는 shell에 Path 환경 변수를 등록해야 합니다.

```sh

tar -xvf uv-x86_64-unknown-linux-gnu.tar.gz

```

### 2-3. UV 설치 확인

* 설치가 완료되었는 지 확인합니다.

```sh
uv
```

## 3. (변외) Python UV 환경 구성

> 설치 뿐만 아니라 UV를 사용하여 Python 환경을 구성하는 과정도 함께 작성합니다.

### 3-1. UV 환경 초기화

```sh
uv init {project-name}
cd {project-name}
```

다음과 같이 환경이 구성됩니다.

```
{project-name}
├── README.md
├── hello.py
├── pyproject.toml
└── uv.lock
```

### 3-2. 가상환경 생성

* `uv venv`를 사용하여 가상 환경을 생성합니다.&#x20;
* UV를 사용하여 가상 환경을 생성하면, Python 가상환경을 사전에 설정하지 않더라도 pyproject.toml 을 읽고 Python을 실행시킬 수 있습니다.

#### UV 가상환경 생성

```sh
uv venv
```

#### UV를 사용하여 초기화 한 프로젝트 내 Python 파일 실행

```sh
uv run hello.py # Hello from {project-name}!
```