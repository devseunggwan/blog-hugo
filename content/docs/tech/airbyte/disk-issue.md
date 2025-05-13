---
title: "Airbyte 운용 시 디스크 용량이 많이 찼을 때 확인해볼 수 있는 것들"
description: ""
weight: 3
---

# Airbyte 운용 시 디스크 용량이 많이 찼을 때 확인해볼 수 있는 것들

![images](https://upload.cafenono.com/image/slashpagePost/20241110/222409_PEKV2oTgrEwYaBqdDl?q=80\&s=1280x180\&t=outside\&f=webp)

## **개요**

Airbyte를 EC2에서 운영하다보면 어느 시점에 할당한 디스크 용량이 부족한 경우가 발생할 때가 있습니다. 해당 이슈를 탐지하지 못하는 경우에 디스크 용량이 꽉차게 되어 Airbyte가 동작하지 않는 상황이 발생할 수 있습니다. 이 경우 `CDC(Change Data Capture)`가 정지되기 때문에 미리 대응하는 것이 중요합니다.

---

## **Airbyte 운용 시 디스크 용량을 많이 차지하는 요소 확인**

Airbyte는 `Source`와 `Destination`을 연결하는 커넥터를 설정 후 사용하는 방식입니다. 각각은 도커 컨테이너로 구성되어 있고 커넥터의 업데이트 시에는 컨테이너 이미지를 다운로드 받아 사용합니다. 사용자가 설정한 시간에 각 단위 별로 커넥터가 동작하는 것을 Task라 하는데, 동작하는 위치마다 로그가 발생하게 됩니다.

Task에서 발생하는 리텐션 기간이 30일로 기본값으로 잡혀있습니다. 기본적으로 리텐션 기간 이전까지는 운영 시 로그가 비교적 많이 적재됩니다. 적재 주기와 리텐션 기간을 고려하여 처음에 디스크 용량 산정 시 넉넉하게 주는 것이 좋습니다.

하지만 한달 이후로 운영하다보면 계속적으로 조금씩 Airbyte 컨테이너 용량이 증가하는 것을 확인하실 수 있습니다. 이를 확인하기 위해서 컨테이너 별 용량을 확인해야 합니다.

```
docker ps --size --format "table {{.ID}}\t{{.Image}}\t{{.Size}}"
```

용량을 확인했을 때 많이 차지하는 컨테이너는 `airbyte-server` 와 `airbyte-worker` 로 확인했습니다.

---

## **Airbyte 컨테이너 내부 확인**

이후 컨테이너 내부로 들어가서 어떤 데이터가 용량을 많이 차지하는 지 확인해야 했습니다.

```
docker exec -it airbyte-server /bin/bash
```

`airbyte-server`에 들어갔을 때 있었던 파일들은 다음과 같습니다. (`airbyte-worker` 도 상황은 동일하였습니다.)

```
ls -alh

total 200G
drwxr-xr-x 1 root root 4.0K Oct  4 02:47 .
drwxr-xr-x 1 root root 4.0K Mar  4  2024 ..
drwxr-xr-x 1 root root 4.0K Jan  2  1970 airbyte-app
-rw-r--r-- 1 root root 200G Oct  4 04:14 build.log
drwxr-xr-x 2 root root 4.0K Mar  4  2024 configs
-rw------- 1 root root  28M Dec  7  2023 dd-java-agent.jar
-rw------- 1 root root  18M Nov 18  2023 opentelemetry-javaagent.jartext
```

두 컨테이너에서 모두 `build.log` 파일이 비대하게 적재되어 있는 부분을 확인했습니다. 로그 파일에서는 내부에서 동작하는 부분에 대해서 로그를 남기고 이상 발생 시 트래킹 할 수 있도록 구성되어 있었습니다.

해당 로그는 운영에 이상을 주기 때문에 S3로 백업 후 삭제하였고, `docker compose restart` 로 컨테이너를 재부팅하여 삭제된 로그의 캐시를 제거하였습니다.

---

## **한계점**

사실 위와 같이 작업한다면, 이후 계속적으로 사람이 수작업으로 백업 — 삭제 — 리부팅을 해줘야 합니다. 운영 상 사람으로 발생하는 피해를 줄이기 위해서 자동화를 해주는 것이 좋습니다.

## **Reference**

* [Understand Airbyte](https://docs.airbyte.com/understanding-airbyte/high-level-view)
* [Docker 용량 확인 및 관리](https://velog.io/@shj5508/Ubuntu-%EC%99%80-Docker-%EC%9A%A9%EB%9F%89-%ED%99%95%EC%9D%B8-%EB%B0%8F-%EA%B4%80%EB%A6%AC)