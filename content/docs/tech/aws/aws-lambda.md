---
title: "AWS Lambda 사용 경험 정리"
description: ""
---

# AWS Lambda 사용 경험 정리

## 개요

* AWS Lambda 를 활용해 스크래핑 및 API를 통해 다양한 데이터를 수집하는 프로젝트를 진행하며 여러 운영 관점에서 중요한 고려 사항들을 직접 체험할 수 있었습니다. 
* Lambda를 사용하면서 마주한 비용 효율성, 리소스 관리, 모니터링, 디버깅 및 AWS의 기타 서비스와의 연계 시 고려해야 할 사항들을 정리해 봤습니다.

---

## Lambda 운영 관점에서의 고려 사항

* Lambda를 사용해서 소규모 작업 등을 자동화하여 관리할 때는 비용 및 리소스 관리 효율적입니다.
  * 다만, 파이프라인 규모가 점차 늘어나서 대규모로 작업이 되어야 한다면 EC2 운용 비용보다 비싸지는 지점이 존재합니다.
* 메모리 총 사용량을 사용자가 지정할 수 있고, 사용한 메모리량에 따라 CPU 스펙과 비용이 최종 결정됩니다.
  * 그렇기 떄문에 100만건 무료라도 많은 양의 메모리를 계속해서 사용한다면 프리티어를 금방 소진할 가능성이 높습니다.
* Lambda의 모니터링은 AWS Cloudwatch로 하기 때문에 해당 비용 관리도 신경써야 합니다.
  * 로그를 통해서 작업 상태를 기록하는 것은 중요하지만, 많은 양의 로그를 Cloudwatch로 적재하게 된다면 이 또한 비용이 많이 발생하게 됩니다.
  * 그렇기 때문에 개발 후 Lambda를 운영하면서 적절하게 Log가 기록되는 지를 검토해야 합니다.
* 만약 15분 이상 한 작업에 대해서 Lambda에서 돌려야 한다면 동시성을 증가시켜 최대한 분산 처리를 하는 것을 추천드립니다.
  * 재귀 방식으로 SQS, SNS, S3를 사용해 Lambda를 실행했을 때 최대 16번 이후로 [재귀 루프 감지](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/invocation-recursion.html)를 하게 됩니다. 이 경우 Lambda에서는 이벤트를 드랍하기 때문에 더 이상 실행할 수 없습니다. 강제로 재귀 루프 감지를 해제할 수는 있지만, 의도치 않은 상황에서 큰 리소스를 사용할 수 있기 때문입니다.
* 되도록 ARM 아키텍처를 Lambda에서 선택하여 사용하였습니다.
  * 비용 및 성능이 기본적으로 x64 아키텍처보다 좋고, 컴퓨팅 운영을 사용자가 하지 않아도 되기 때문에  다른 서비스에서의 ARM 아키텍처  사용 난이도보다 비교적 낮기 때문입니다.
  * 다만, 사용하는 언어나 비즈니스 로직에서 사용해야 하는 라이브러리 별로 제약 사항이 존재하기 떄문에 테스트 및 검토 후 사용해야 합니다.

---

## Python을 사용한 Lambda 함수 작성 시 고려 사항

* [Klayer](https://github.com/keithrozario/Klayers) 를 사용하여 Lambda에서 Python 실행 시 필요한 라이브러리를 가져왔습니다.
  * Lambda에서 Python 라이브러리를 사용할 때는 각 함수 별로 계층을 추가해서 사용해야 합니다. 매번 필요한 라이브러리를 정리해서 Lambda 계층을 만드는 것이 번거로운 작업이라 생각했습니다.
  * Lambda에서는 단순한 작업이 주로 이루어지는 것이 좋기 때문에 Klayer에서 제공해주는 라이브러리 내에서 최대한 처리하려고 하였습니다.
* Python 라이브러리를 직접 Layer에 추가하는 경우는 다음과 같았습니다.
  * 비즈니스 로직에 맞는 Klayer Layer가 없는 경우
  * Klayer Layer를 사용했는데 250MB Layer 용량 제한에 걸린 경우
* [AWS Powertools](https://docs.powertools.aws.dev/lambda/python/3.8.0/)를 사용하여 Lambda에서 로깅이나 모니터링 등 기능들을 추가할 수 있습니다.
  * Lambda를 처음 작업 시에 디버깅이 쉽지 않았던 경험이 있었습니다. 디버깅을 하기 위해 직접 코드를 작성할 필요 없이 AWS Powertool 에서 제공하는 기능을 사용하여 해결하였습니다.

---

## AWS SQS(Simple Queue Service)와 연동하여 사용 시 고려 사항

* Lambda에서 SQS 이벤트를 Consuming하는 Window Size에 따라서 처리하는 Lambda의 동시성의 차이가 발생할 수 있습니다.
  * 만약 10개의 이벤트를 SQS로 날리고 Lambda에서의 Window size가 5라면 실행 시간이 충분히 길다 가정하여 2개의 동시성을 가집니다.
  * Lambda의 동시성은 1000이기 떄문에 처리하려는 비즈니스 요구사항에 따라서 Window Size를 조정하여 동시성을 낮춰야 합니다.
  * Window size에 따른 메시지를 처리할 수 있도록 구성해야 하고, 반대로 Window size가 너무 커서 Lambda가 전부 처리하지 못해 타임아웃이 나는 사항을 고려해야 합니다.
* [공식 문서](https://docs.aws.amazon.com/ko_kr/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configure-lambda-function-trigger.html)에서도 이야기되었지만, SQS의 가시성 제한 시간은 함수 타임 아웃의 6배로 설정해야 합니다.
  * 이벤트가 실패할 경우, 재실행을 Lambda에서 할 수 있도록 설정해야 합니다. 네트워크 이슈나 원천 소스가 존재하는 서버의 이상으로 간헐적인 이슈가 발생할 수 있기 때문에 충분한 재실행으로 데이터를 수집할 수 있도록 하였습니다.

---

## Lambda 배포 관점에서 고려 사항

* Lambda를 배포할 때 Terraform을 사용하여 배포하였습니다.
  * Lambda를 사용하다 보면 여러 함수들을 생성 및 관리해야 하는 경우가 발생하였고, 이 때마다 계속 함수에 대한 설정을 GUI로 진행했는데 비효율적이였습니다.
  * AWS SAM이나 Serverless Framework 등 Lambda를 개발할 떄 사용할 수 있는 도구들이 존재하였지만 해당 도구들은 POC를 진행 후 사용하지 않기로 결정했습니다.
  * 운영했던 Lambda는 EventBridge Scheduler, SQS 등도 함께 사용하여 데이터 파이프라인을 구성했기 때문에 해당 아키텍처를 함께 배포하거나 내려야 하는 등 요구사항도 존재하여, Terraform을 채택하였습니다.
  * Python 코드와 Terraform 코드를 묶어서 관리하였고, 모노레포로 관리하여 효율성을 높혔습니다. 이렇게 작업했을 경우, Lambda 함수 및 플랫폼이 Git을 통한 버저닝과 CI/CD를 연결이 가능했습니다.
