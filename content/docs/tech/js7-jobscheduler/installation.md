---
title: "Installation"
description: ""
weight: 2
---

# JS7 JobScheduler Controller and Agent 설치 (w/ docker-compse)

## 1. 공식 문서 참조 기준

> 현재 Alpine 기본 이미지와 OpenJDK와 함께 제공되는 Linux 기반 [OCI](https://opencontainers.org/) 호환 컨테이너 이미지 에서 제공\
> 2025.01.03 기준 2.5.0 버전으로 공식 문서에 적혀있어서 참조하였습니다.

### 1-1. dotenv 설정

1. docker-compose 내 설정 값으로 들어가야 하는 항목들을 지정합니다.

```
JS7USERID=1000
JS7GROUPID=0
JS7VERSION=2-5-0
```

### 1-2. [Agent](https://kb.sos-berlin.com/display/JS7/JS7+-+Agent+Installation+using+Docker+Compose?src=contextnavpagetreemode)

1. docker-compose.yml 을 다운로드 받습니다.
2. agent의 Volume이 마운트 되는 폴더를 생성합니다.
   1. `mkdir js7-agent-primary`
3. docker-compose를 실행합니다.
   1. dotenv를 적용해야 합니다.
   2. ```bash
      docker compose --env-file ./.env -f docker-compose.yml up -d
      ```

<details>

<summary><a href="https://kb.sos-berlin.com/download/attachments/80970104/docker-compose.yml?version=2&#x26;modificationDate=1654531801000&#x26;api=v2">docker-compose.yml</a></summary>

<pre class="language-yaml"><code class="lang-yaml"><strong>
</strong>version: '3'
 
services:
  js7-agent-primary:
    image: sosberlin/js7:agent-${JS7VERSION}
    hostname: js7-agent-primary
    volumes:
      - js7-agent-primary:/var/sos-berlin.com/js7/agent/var_4445
    networks:
      - js7
    environment:
      RUN_JS_JAVA_OPTIONS: -Xmx256m
      RUN_JS_USER_ID: "${JS7USERID}:${JS7GROUPID}"
    restart: "no"
 
networks:
  js7:
    external: true
 
volumes:
  js7-agent-primary:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/js7-agent-primary
      o: bind
</code></pre>



</details>

### 1-3. [Controller](https://kb.sos-berlin.com/display/JS7/JS7+-+Controller+Installation+using+Docker+Compose?src=contextnavpagetreemode)

1. docker-compose.yml 을 다운로드 받습니다.
2. controller의 Volume이 마운트 되는 폴더를 생성합니다.
   1. `mkdir js7-controller-primary`
3. docker-compose를 실행합니다.
   1. dotenv를 적용해야 합니다.
   2. ```bash
      docker compose --env-file ./.env -f docker-compose.yml up -d 
      ```

<details>

<summary><a href="https://kb.sos-berlin.com/download/attachments/80970088/docker-compose.yml?version=1&#x26;modificationDate=1654530751000&#x26;api=v2">docker-compose.yml</a></summary>

```yaml
version: '3'
 
services:
  js7-controller-primary:
    image: sosberlin/js7:controller-${JS7VERSION}
    hostname: js7-controller-primary
    volumes:
      - js7-controller-primary:/var/sos-berlin.com/js7/controller/var
    networks:
      - js7
    environment:
      RUN_JS_JAVA_OPTIONS: -Xmx256m
      RUN_JS_USER_ID: "${JS7USERID}:${JS7GROUPID}"
    restart: "no"
 
networks:
  js7:
    external: true
 
volumes:
  js7-controller-primary:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/js7-controller-primary
      o: bind
      o: bind
```



</details>

### 1-4. [JOC Cockpit ](https://kb.sos-berlin.com/display/JS7/JS7+-+JOC+Cockpit+Installation+using+Docker+Compose?src=contextnavpagetreemode)

> MySQL, PostgreSQL 두가지 종류의 DB를 지원합니다. \
> (이 중 MySQL을 사용한 설치 예시를 작성하였습니다.)

1. docker-compose.yml 및 hibernate.cfg.xml 을 다운로드 받습니다.
2. JOC Cockpit의 Volume이 마운트 되는 폴더를 생성합니다.
   1. ```bash
      mkdir db_data
      mkdir js7-joc-primary-config
      mkdir js7-joc-primary-logs
      ```
3. docker-compose를 실행합니다.
   1. dotenv를 적용해야 합니다.
   2. ```bash
      docker compose --env-file ./.env -f docker-compose.yml up -d
      ```
4. hibernate.cfg.yml을 `js7-joc-primary-config` 폴더로 복사합니다.
   1. ```bash
      cp -f hibernate.cfg.xml js7-joc-primary-config/
      ```
5. JS7 Scheduler의 메타 정보를 저장하는 MySQL을 초기화합니다.
   1. 초기화 진행
      1. ```bash
         docker-compose exec js7-joc-primary /bin/sh -c /opt/sos-berlin.com/js7/joc/install/joc_install_tables.sh
         ```
   2. 결과 확인
      1. ```bash
         tail js7-joc-primary-logs/install-result.log
         ```

<details>

<summary> <a href="https://kb.sos-berlin.com/download/attachments/80970015/docker-compose.yml?version=2&#x26;modificationDate=1654526940000&#x26;api=v2">docker-compose.yml</a></summary>

```yaml
version: '3'
 
services:
  db:
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"
    networks:
      - js7
    environment:
      MYSQL_ROOT_PASSWORD: js7rootpassword
      MYSQL_DATABASE: js7db
      MYSQL_USER: js7user
      MYSQL_PASSWORD: js7password
    restart: "no"
 
  js7-joc-primary:
    depends_on:
      - db
    container_name: js7-joc-primary
    image: sosberlin/js7:joc-${JS7VERSION}
    hostname: js7-joc-primary
    ports:
      - "17446:4446"
    networks:
      - js7
    volumes:
      - js7-joc-primary-config:/var/sos-berlin.com/js7/joc/resources/joc
      - js7-joc-primary-logs:/var/sos-berlin.com/js7/joc/logs
    environment:
      RUN_JS_JAVA_OPTIONS: -Xmx256m
      RUN_JS_USER_ID: "${JS7USERID}:${JS7GROUPID}"
    restart: "no"
 
networks:
  js7:
    external: true
 
volumes:
  db_data:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/db_data
      o: bind
 
  js7-joc-primary-config:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/js7-joc-primary-config
      o: bind
 
  js7-joc-primary-logs:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/js7-joc-primary-logs
      o: bind
```



</details>

<details>

<summary><a href="https://kb.sos-berlin.com/download/attachments/80970015/hibernate.cfg.xml?version=1&#x26;modificationDate=1654524637000&#x26;api=v2">hibernate.cfg.xml</a></summary>

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<hibernate-configuration>
 <session-factory>
  <property name="hibernate.connection.driver_class">org.mariadb.jdbc.Driver</property>
  <property name="hibernate.connection.password">js7password</property>
  <property name="hibernate.connection.url">jdbc:mariadb://db:3306/js7db</property>
  <property name="hibernate.connection.username">js7user</property>
  <property name="hibernate.dialect">org.hibernate.dialect.MySQLInnoDBDialect</property>
  <property name="hibernate.show_sql">false</property>
  <property name="hibernate.connection.autocommit">false</property>
  <property name="hibernate.format_sql">true</property>
  <property name="hibernate.temp.use_jdbc_metadata_defaults">false</property>
  <property name="hibernate.connection.provider_class">org.hibernate.hikaricp.internal.HikariCPConnectionProvider</property>
  <property name="hibernate.hikari.maximumPoolSize">10</property>
 </session-factory>
</hibernate-configuration>
```



</details>

### Reference

{% embed url="https://kb.sos-berlin.com/display/JS7/JS7+-+Installation+for+Containers?src=contextnavpagetreemode" %}

{% embed url="https://qiita.com/saitamanokusa/items/ffb8f05cbc8e75d435ce" %}

## 2. 설치 개선

> 공식 문서에는 2.5.0 버전으로 설치를 진행하게 되어 있지만, 최근까지 계속 메인테이너가 개발을 하고 있고 Stable Version이 2.7.3이여서 해당 버전으로 개선안을 작성했습니다.

### 2-1. Agent 컨테이너에 Python 환경 구성

기존 빌드된 컨테이너에는 Python 환경이 구성되지 않았습니다. Python을 실행하기 위해서 기존 이미지를 바탕으로 새로운 이미지를 빌드해야 했습니다. Python과 의존성을 가지는 환경을 Dockerfile로 구성했습니다. Python 환경, 파일들을 공유할 폴더도 생성하도록 추가했습니다.

> Python은 UV 기반으로 구성할 예정이여서 설치 스크립트가 추가되어 있습니다. 사용하는 환경에 따라서 Dockerfile 구성을 변경하면 됩니다.

<pre class="language-docker"><code class="lang-docker"><strong>FROM sosberlin/js7:agent-2-7-3
</strong>
RUN apk add \
    wget \
    gcc \
    make \
    zlib-dev \
    libffi-dev \
    openssl-dev \
    musl-dev

RUN apk add --no-cache python3 python3-dev py3-pip

RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
ENV PATH="$PATH:/root/.cargo/bin"

RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="$PATH:/root/.local/bin"

RUN mkdir /var/sos-berlin.com/js7/agent/workspace
</code></pre>

#### 파이썬 사용 시 주의사항

JS7 JobScheduler에서 사용한 컨테이너는 Alpine Linux 를 사용했습니다. Alpine Linux는 C 컴파일에 musl을 사용하기 때문에 Python의 특정 라이브러리를 설치 시 binary wheel 을 사용하지 못합니다. 그래서 설치시 C 코드를 컴파일해야 하는 문제점이 발생합니다. 해당 문제 때문에 설치 시간이 기존 보다 몇 (십)배는 느려지는 이슈가 있습니다. 그래서 라이브러리 사용에 제약 사항이 발생할 수 있습니다.

현재 확인한 설치가 느린 라이브러리는 [Polars](https://github.com/pola-rs/polars), [oracledb ](https://oracle.github.io/python-oracledb)가 있습니다.&#x20;

{% embed url="https://eden-do.tistory.com/64" %}

{% embed url="https://nx006.tistory.com/70" %}

### 2-2. docker-compose 통합 및 Agent 다중화

문서에는 Agent, Controller, JOC를 각각의 docker-compose 파일로 분리했었습니다. 개발 환경을 구성할     떄 편리함을 위해서 한 개의 docker-compose 파일로 통합하였습니다.&#x20;

그리고 Enterprise 버전을 사용하지 않더라도 여러 개의 Standalone Agent를 Controller에 등록할 수 있습니다. 그래서 여러 개의 Agent를 사용하도록 deploy 옵션을 추가했습니다. Replicas 옵션은 dotenv에서 수정할 수 있도록 하였습니다.&#x20;

그리고 각 Agent가 실행할 코드를 공유할 수 있도록 docker volume를 추가하였습니다. Python 코드를 Git Repo에서 Pull 방식으로 배포 후 Agent 에서 사용할 수 있도록 하기 위해 해당 방법으로 구성 하였습니다.. Rolling이나 Blue/Green 등배포 전략을 사용하기에는 Agent 를 Controller에서 등록하는 방식때문에 배포 전략을 사용하기 어려운 점도 고려했습니다.

#### dotenv

<pre class="language-sh"><code class="lang-sh"><strong>JS7USERID=1000
</strong>JS7GROUPID=0
JS7VERSION=2-7-3

JS7_AGENT_REPLICAS=3
</code></pre>

#### docker-compose.yml

```yaml
version: '3'

services:

  db:
    image: mysql:8.0
    ports:
      - "3306:3306"
    command:
      - --default-authentication-plugin=caching_sha2_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_bin
      - --skip-character-set-client-handshake
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - js7
    environment:
      MYSQL_ROOT_PASSWORD: iHateMySQL8.0!
      MYSQL_DATABASE: js7db
      MYSQL_USER: js7user
      MYSQL_PASSWORD: js7password
    restart: "no"

  js7-joc-primary:
    depends_on:
      - db
    image: sosberlin/js7:joc-${JS7VERSION}
    hostname: js7-joc-primary
    ports:
      - "17443:4446"
    volumes:
      - js7-joc-primary-config:/var/sos-berlin.com/js7/joc/resources/joc
      - js7-joc-primary-logs:/var/log/sos-berlin.com/js7/joc
    networks:
      - js7
    environment:
      RUN_JS_JAVA_OPTIONS: -Xmx256m
      RUN_JS_USER_ID: "${JS7USERID}:${JS7GROUPID}"
    restart: "no"

  js7-controller-primary:
    image: sosberlin/js7:controller-${JS7VERSION}
    hostname: js7-controller-primary
    volumes:
      - js7-controller-primary:/var/sos-berlin.com/js7/controller
    networks:
      - js7
    environment:
      RUN_JS_JAVA_OPTIONS: -Xmx256m
      RUN_JS_USER_ID: "${JS7USERID}:${JS7GROUPID}"
    restart: "no"

  js7-agent-primary:
    build:
      context: .
      dockerfile: Dockerfile.js7-agent
    entrypoint: entrypoint.sh
    hostname: js7-agent-primary
    volumes:
      - js7-agent-primary-workspace:/var/sos-berlin.com/js7/agent/workspace
    networks:
      - js7
    environment:
      RUN_JS_JAVA_OPTIONS: -Xmx256m
      RUN_JS_USER_ID: "${JS7USERID}:${JS7GROUPID}"
    restart: "no"
    deploy:
      mode: replicated
      replicas: ${JS7_AGENT_REPLICAS}
      endpoint_mode: vip

volumes:

  db_data:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/db_data
      o: bind

  js7-joc-primary-config:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/js7-joc-primary-config
      o: bind

  js7-joc-primary-logs:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/js7-joc-primary-logs
      o: bind

  js7-controller-primary:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/js7-controller-primary
      o: bind

  js7-agent-primary-workspace:
    driver: local
    driver_opts:
      type: none
      device: ${PWD}/js7-agent-primary-workspace
      o: bind


networks:
  js7:
```

### 2-3. JS7 JobScheduler 초기 개발 환경 구성 스크립트 작성

일련의 작업 과정을 자동화하기 위해 스크립트를 작성하였습니다. docker-compose up 이후 20초 정도 기다렸다가 DB 초기화 작업을 진행합니다.&#x20;

만약 DB 초기화 중 에러가 발생하였다면, 스크립트의 마지막 두 줄을 다시 실행하여야 합니다. 초기화가 완료되지 않는다면, JOC가 열리지 않습니다.&#x20;

```sh
mkdir js7-agent-workspace
mkdir js7-controller-primary
mkdir js7-joc-primary-config
mkdir js7-joc-primary-logs
mkdir db_data

docker compose --env-file ./.env -f docker-compose.yml up -d

echo "Waiting for the database to start..."
sleep 20

cp -f hibernate.cfg.xml js7-joc-primary-config/
docker-compose exec js7-joc-primary /bin/bash -c /opt/sos-berlin.com/js7/joc/install/joc_install_tables.sh

docker-compose restart js7-joc-primary
```