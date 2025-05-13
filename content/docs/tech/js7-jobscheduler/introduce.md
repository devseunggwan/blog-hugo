---
title: "Introduce"
description: ""
weight: 1
---

# JS7 JobScheduler Controller and Agent 개요

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>JS7 JOC Cockpit Dashboard UI</p></figcaption></figure>

### 개요

* 홈페이지(회사): [https://www.sos-berlin.com/en](https://www.sos-berlin.com/en)
* Documentation: [https://kb.sos-berlin.com/display/JS7/JS7](https://kb.sos-berlin.com/display/JS7/JS7)
* 이슈 추적: [https://change.sos-berlin.com/secure/Dashboard.jspa](https://change.sos-berlin.com/secure/Dashboard.jspa)
* 포럼: [https://sourceforge.net/p/jobscheduler/discussion/](https://sourceforge.net/p/jobscheduler/discussion/)

### [라이센스 정책](https://kb.sos-berlin.com/display/JS7/JS7+-+License)

* JS7 JobScheduler와 YADE 제품은 고객에게 오픈 소스 라이선스와 상업용 라이선스 중에서 선택할 수 있는 듀얼 라이선스 모델로 제공
  * 오픈 소스 라이선스 [GPLv3(일반 공중 라이선스)에 따라 제공](https://www.gnu.org/licenses/gpl-3.0.en.html)
  * SOS의 상용 라이센스 구매
* **유일한 예외는 기업 고객을 위한 상업적으로 이용 가능한 기능인 고가용성을 위해 JS7 제품을 클러스터링하는 운영 기능**
  * 초기 해당 제품을 사용하여 피봇팅을 하는 경우 단일 컨트롤러에 독립형 에이전트를 여러개 사용할 수 있으나, 규모가 확장되고 SPOF를 피하는 목적으로 에이전트 클러스터링을 고려할 수 있음

### 아키텍처 구성요소

* `Controller`
  * 컨트롤러는 실행할 워크플로와 작업, 실행 시기, 실행에 사용할 에이전트를 알고 있습니다.
  * 컨트롤러는 JOC Cockpit에서 작업 관련 인벤토리를 수신하고 이 정보를 각 서버에서 워크플로우와 작업을 실행하는 에이전트에 배포합니다.
  * 독립형 컨트롤러는 에이전트를 오케스트레이션하는 단일 인스턴스입니다.
  * 할당하는 에이전트의 개수는 무제한입니다.
* `Agent`
  * 에이전트은 에이전트 서버에서 실행 파일 및 명령을 호출하는 작업을 실행합니다.
  * 에이전트는 컨트롤러로부터 시작할 잡과 시작 시점에 대한 정보를 받습니다.
  * 에이전트는 실행 결과와 로그 출력을 컨트롤러에 다시 보고합니다.
  * 에이전트는 작업 실행 시점에 컨트롤러가 연결되지 않아도 자율적으로 작동할 수 있습니다.
  * 독립형 에이전트는 서로 독립적으로 작동합니다.
* `JOC Cockpit`
  * JOC Cockpit은 작업 관련 인벤토리를 관리하고 컨트롤러 및 에이전트의 워크플로 실행을 모니터링 및 제어하기 위한 사용자 인터페이스입니다.
  * 독립형 JOC Cockpit은 하나 이상의 컨트롤러를 관리하는 데 사용할 수 있는 단일 인스턴스입니다.


