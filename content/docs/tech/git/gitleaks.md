---
title: "Gitleaks를 사용한 Git 비밀 유출 검사, 사용법"
description: ""
---

# Gitleaks를 사용한 git 비밀 유출 검사, 사용법

AWS 키가 Github를 통해 노출이 되서 해킹으로 천만원 넘게 과금이 되고, 이를 AWS에서 도와주는 경우를 종종 들었습니다. _**AWS 키 노출**_ 키워드로 구글에 검색하면 이에 대한 썰(?)을 어렵지 않게 찾아볼 수 있습니다. 키 노출 이슈가 개인이나 기업에서 중대하게 작용하기 때문에 이에 대한 대비책들도 존재합니다.

이외에도 개발을 하면서 서버나 DB, API 등에 접근하기 위해 비밀 키 등을 사용합니다. 이러한 비밀들이 Git 리포지토리에 커밋이 되었는 지를 확인해주는 오픈소스 도구인 Gitleaks에 대해서 소개해드리려고 합니다.

## **Gitleaks**

Gitleaks는 Git 리포지토리의 비밀번호, API 키, 토큰과 같은 하드코딩 된 비밀을 감지하고 유출을 방지하는 SAST(Static Application Security Testing, 정적 어플리케이션 보안 테스트) 도구입니다.

Go로 작성되었으며, Homebrew, Docker, Git Repo, Pre-commit, Github-Action 등으로 설치하여 사용할 수 있습니다.

아래는 Homebrew를 통해서 맥 로컬에 설치하는 방법입니다.

```
brew install gitleaks
```

Gitleaks를 통해서 찾을 수 있는 비밀은 여러가지가 있고, [Github Repo](https://github.com/gitleaks/gitleaks/blob/master/cmd/generate/config/main.go)에서 각 규칙(총 137가지)들을 확인할 수 있습니다.

> 만약, 찾는 규칙이 없다면 오픈소스에 직접 기여해보는 방법도 추천드립니다:)

Gitleaks를 만약 로컬에 설치했다면, 실행하는 방법은 다음과 같습니다.

```
gitleaks detect -v # verbose
```

```
    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

9:28PM INF 157 commits scanned.
9:28PM INF scan completed in 1.49s
9:28PM INF no leaks found
```

> 157개의 커밋된 Git Repo를 Gitleaks로 검사했을 때 1.49초만에 완료하고, 찾은 비밀은 없는 경우입니다.

## **Gitleaks 설정**

Gitleaks로 검사할 때 탬플릿에 목업으로 적어놓은 키 같이 필터링을 해서 비밀키를 찾아야 하는 경우가 있습니다. 이를 위해서 config 파일을 제공합니다. config 파일은 Git Repo가 위치한 루트 패스에 `[.gitleaks.toml](https://github.com/gitleaks/gitleaks?tab=readme-ov-file#configuration)` 로 저장하면 별도 스크립트 추가없이 실행할 때 자동으로 설정이 적용됩니다.

`.gitleaks.toml`을 통해서 설정을 했음에도 불구하고 커밋된 목업 키가 계속 찾아진다면 `.gitleaksignore` 를 통해서 검사를 무시할 위치를 추가합니다.

```
Finding:     aws_secret="AKIAIMNOJVGFDXXXE4OA"
RuleID:      aws-access-token
Secret       AKIAIMNOJVGFDXXXE4OA
Entropy:     3.65
File:        checks_test.go
Line:        37
Commit:      ec2fc9d6cb0954fb3b57201cf6133c48d8ca0d29
Author:      Zachary Rice
Email:       z@email.com
Date:        2018-01-28T17:39:00Z
Fingerprint: ec2fc9d6cb0954fb3b57201cf6133c48d8ca0d29:checks_test.go:aws-access-token:37
```

위처럼 테스트에서 사용하는 키가 Gitleaks 검사 시 나왔고 해당 키는 검사에는 제외하고 싶습니다. 이런 경우, `.gitleaksignore`에는 Fingerprint의 값을 추가하면 다음 검사부터는 해당 커밋의 키는 검사하지 않게 됩니다.

```
# .gitleaksignore
ec2fc9d6cb0954fb3b57201cf6133c48d8ca0d29:checks_test.go:aws-access-token:37
```

gitleaks로 검사를 했을 때 나온 파일들이 이상이 없는 경우, report로 나온 파일을 사용하여서 기준점을 만들 수 있습니다. (해당 기능을 사용하여 리포팅도 가능합니다.)

```
gitleaks detect --report-path gitleaks-report.json
```

재 검사 시 기준점 이후 나온 항목들에 대해서만 추출하는 방식으로 검사할 수 있습니다.

```
gitleaks detect --baseline-path gitleaks-report.json --report-path findings.json
```

## **Case Study: Apache Airflow에 Gitleaks 써보기**

Github에는 수많은 레포들이 존재합니다. Case Study를 위해 클론하여 테스트 해 볼 레포는 데이터 파이프라인을 구축할 때 자주 사용하는 [Apache Airflow](https://github.com/apache/airflow)로 정했습니다. 커밋의 수가 많고, 프로젝트 기간이 오래되어서 확인해보기 좋을 것 같았습니다.

Gitleaks를 사용하기 위해 먼저 Airflow Repo를 클론합니다.

```
git clone https://github.com/apache/airflow.git
```

> 작성일 당시 main 브랜치가 25,000 커밋 정도 되었기 때문에 Repo 크기가 큽니다. 클론 시간이 조금 걸릴 수 있습니다.

이후, CLI(Bash, Zsh…)에서 Gitleaks를 실행합니다.

```
○
    │╲
    │ ○
    ○ ░
    ░    gitleaks
```

```
10:05PM INF 33588 commits scanned.
10:05PM INF scan completed in 21.2s
10:05PM WRN leaks found: 178
```

* 브랜치 포함 총 33588 커밋에서 178개의 leaks를 발견했습니다.
* Verbose 옵션(-v)를 주고 실행하면, 178개에 해당되는 내용이 전부 CLI로 나옵니다.
* CLI로 나오는 내용을 정리하려면, `— report-path` 를 주어 json, csv 등으로 출력 가능합니다.

### **출력한 Report 분석**

저는 Airflow 검사한 내용을 `— report-path` 를 사용하여 reports.json으로 출력하고, 분석을 위해 `Pandas`를 사용하여 Json을 로드하였습니다.

**gitleaks 확인**

```
gitleaks detect --report-path gitleaks-report.json
```

나온 Json을 확인해보면 다음과 같은 형태로 나옵니다. (reports.json)

```
[
 {
  "Description": "Detected a Generic API Key, potentially exposing access to various services and sensitive operations.",
  "StartLine": 33,
  "EndLine": 33,
  "StartColumn": 30,
  "EndColumn": 57,
  "Match": "Key\": \"cm9tIGlzIHRoZSBraW5n\"",
  "Secret": "cm9tIGlzIHRoZSBraW5n",
  "File": "helm_tests/other/test_git_ssh_key_secret.py",
  "SymlinkFile": "",
  "Commit": "f81abd6145d20305148e0a91e8a6ff5026927b70",
  "Entropy": 4.221928,
  "Author": "rom sharon",
  "Email": "33751805+romsharon98@users.noreply.github.com",
  "Date": "2024-06-19T06:00:41Z",
  "Message": "add git-sync-ssh secret template (#39936)",
  "Tags": [],
  "RuleID": "generic-api-key",
  "Fingerprint": "f81abd6145d20305148e0a91e8a6ff5026927b70:helm_tests/other/test_git_ssh_key_secret.py:generic-api-key:33"
 },
  ...
}
```

**Pandas로 Json 로드**

```
import pandas as pd

df = pd.read_json("./data/gitleaks-report.json").to_df()
df.head()
```

각각의 항목 값을 집계하여 확인해봅니다.

```
RuleID
generic-api-key      149
aws-access-token      13
slack-bot-token        7
slack-webhook-url      5
private-key            4
Name: count, dtype: int64
```

![RuleID 빈도 집계](https://upload.cafenono.com/image/slashpagePost/20241110/195016_lVVKdQ2pj38JraM0kz?q=80\&s=1280x180\&t=outside\&f=webp)

> generic-api-key 항목이 많았고, 다음으로 aws-access-token, slack-bot-token, slack-webhook-url, private-key 순으로 있었습니다.

Airflow Repo에서 확인한 비밀들의 커밋 메시지나 파일들을 보면 대부분 탬플릿이나 예시 파일들에서 사용한 목업 비밀들인 것을 확인할 수 있습니다.

```
df.loc[df["File"].str.contains("test")]["RuleID"].value_counts()

## 출력 결과
RuleID
generic-api-key      121
slack-webhook-url      3
private-key            3
slack-bot-token        3
aws-access-token       3
Name: count, dtype: int64
```

> 검사로 발견한 178개 중 133개의 항목이 파일이름에 test 가 붇어 있습니다.

대부분 테스트를 위해서 사용한 목업 형태의 비밀들 임을 확인하였고, 테스트가 붇지 않은 경우에도 howto 나 template, docs, example 등 파일 이름에서 확인할 수 있었습니다.

## **Gitleaks 자동화**

Gitleaks를 사용한 비밀 검사를 자동하기 위해서 Pre-commit과 Github-Action을 사용합니다.

* Pre-commit은 로컬에서 작업 후 커밋에서 코드를 스캔하여 비밀을 감지합니다.
* Github-Action은 CI/CD 과정에서 커밋이나 PR에서 등록된 비밀을 감지합니다.

아래는 Pre-commit과 Github-Action을 각각 설정하기 위한 코드입니다.

### **.pre-commit-config.yaml**

```
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.16.1
    hooks:
      - id: gitleaks
```

### **Github-Action**

```
name: gitleaks
on: [pull_request, push, workflow_dispatch]
jobs:
  scan:
    name: gitleaks
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITLEAKS_LICENSE: ${{ secrets.GITLEAKS_LICENSE}} # Only required for Organizations, not personal accounts.
```

## **마치며**

`gitleaks`를 사용하는 것도 좋지만, 평소에 비밀에 대한 관리가 더 중요하다고 생각합니다. 그렇기 때문에 `dotenv` 등을 사용해서 내부 환경에 비밀을 로드하여 사용하고, 코드에 비밀이 커밋이 되지 않도록 습관을 들이는 것이 중요합니다.

그래도 아무리 조심한다 한들 사람이기 때문에 안전장치를 추가하는 것도 좋습니다. gitleaks를 자동화하여 비밀 유출에 개개인이 잘 신경쓰면 좋겠습니다.

읽어주셔서 감사합니다:)

## **더 읽어 보기**

* Homepage: [https://gitleaks.io/](https://gitleaks.io/)
* Github: [https://github.com/gitleaks/gitleaks](https://github.com/gitleaks/gitleaks)
* Maintainer: [https://blog.gitleaks.io/gitleaks-llc-announcement-d7d06a52e801](https://blog.gitleaks.io/gitleaks-llc-announcement-d7d06a52e801)
* [https://medium.com/@tarikyegen35/gitleaks-a-secret-scanner-tool-5cd9b6f1d367](https://medium.com/@tarikyegen35/gitleaks-a-secret-scanner-tool-5cd9b6f1d367)
* [https://akashchandwani.medium.com/what-is-gitleaks-and-how-to-use-it-a05f2fb5b034](https://akashchandwani.medium.com/what-is-gitleaks-and-how-to-use-it-a05f2fb5b034)