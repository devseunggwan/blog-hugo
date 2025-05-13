---
title: "VSCode Python 익스텐션 에러 트러블 슈팅 (Failed to resolve env)"
description: ""
---

# VSCode Python 익스텐션 에러 트러블 슈팅 (Failed to resolve env)

### 문제 상황

* VSCode 내부에서 Intelisense와 Python이 충돌하였다고 알림이 계속 나왔었고 파이썬을 로드하는 상태였습니다.
* 이로 인해서 Python 관련 VScode 익스텐션이 제대로 동작하지 않았습니다.

### 문제 해결 방식

* &#x20;먼저 VSCode Python 출력창에서 문제가 발생하는 내용을 확인하고 키워드를 통해서 구글링하였습니다.
  * `Python Extension: interpreterManager.refresh [l [Error]: Failed to resolve env`&#x20;
* 구글링을 했을 때 많은 사람들은 `Python.Locator`을 `native`에서 `js`로 변경하는방식으로 해결을 많은 사람들이 하였지만 변경하여도 문제는 계속 발생 하였습니다.
  * [https://github.com/microsoft/vscode-python/issues/23956](https://github.com/microsoft/vscode-python/issues/23956)
* 출력창의 Python 익스텐션 에러를 분석했을 때, 이전에 가상환경으로 사용하고 지웠는데 계속 파이썬 익스텐션에서 로드하고 있었고 이로 인해서 에러를 뱉어내고 있었습니다.&#x20;
* 그래서 `Python: 캐시 지우기 및 창 다시 로드(Python: Clear Cache and Reload Window)` 기능을 사용하여 캐싱을 지웠습니다. 캐싱을 지우니까 정상적으로 익스텐션들을 로드하는 것을 확인할 수 있었습니다.

### 문제 발생 원인

* 당시 Github Codespace에서 Attention 관련 논문을 읽고 샘플 코드들을 작성하면서 학습하는 과정이였습니다.&#x20;
* 그 과정에서 코드에 사용되는 Python 의존성들을 다운받는 과정을 거쳤습니다. 그런데 그 과정에서 Codespace 용량(32G)을 생각하지 않고 pip install을 진행하였습니다. 그래서 다운로드를 계속 받는데 용량을 가득 채우는 상황이 발생하였고, 급하게 종료를 눌렸습니다.&#x20;
* 그 과정에서 Python 익스텐션의 내부 동작까지 확인할 수는 없지만, 가상 환경이 계속 캐싱이 되어 있었던 상황이였던거 같고 이후 Codespace를 삭제하였습니다. Codespace로 작업한 동일 Repo를 로드할 때부터 에러는 계속 발생하였습니다.

### 개선 방향

* VSCode 익스텐션에서 에러가 발생하는 경우가 있으면, 캐싱이 되어 있을 수 있다는 선택지를 생각해야 한다고 생각합니다. 익스텐션에서 캐싱을 지우는 방법을 제공한다면 지우고 다시 확인해보아야 합니다.
* 인공지능 관련 개발 시 버전 관리는 Github에 저장하되, Google Colab을 활용하여 실험을 할 수 있도록 하는 것이 합리적입니다. 인공지능 관련 작업을 할 때 GPU 리소스를 구글에서 무료로 사용할 수 있게 제공해주기 때문입니다.&#x20;
* Codespace는 제한된 리소스 내에서 개발, 테스트를 해볼 수 있는 상황에서 더 활용하는 것이 좋다고 생각합니다. 많은 리소스를 제공하는 것은 아니지만, 무료로독립된 환경에서 간편하게 사용하고 종료할 수 있기 때문에 리소스가 많이 들어가지 않는 개발을 할 때 Codespace를 사용하는게 합리적입니다.
