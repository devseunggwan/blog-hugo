---
title: "UnstructuredURLLoader 무한 로딩 해결"
description: ""
---

# UnstructuredURLLoader 무한 로딩 해결

## 요약

* UnstructedURLLoader에서 사용하는 라이브러리 중 [unstructured.file\_utils.filetype](https://github.com/Unstructured-IO/unstructured/blob/main/unstructured/file_utils/filetype.py) 에서 python-magic 라이브러리를 로드할 때 발생하는 이슈였습니다.
* 해당 라이브러리를 사용할 떄는 OS 별로 다르게 magic 라이브러리의 종속성을 제공해야 합니다. 그래서 Linux나 MacOS를 사용할 때는 Unstructed 기본 종속성으로 `python-magic` 이 설치되어 문제가 없지만, Windows를 사용할 떄는 `python-magic-bin` 을 사용하는 환경에 종속성을 추가해야 합니다.
  * Windows: `python-magic-bin`
  * Linux, MacOS: `python-magic`

---

## 상황 설명

* `Windows 환경`에서 [Hugging-face Open-Source AI Cookbook](https://huggingface.co/learn/cookbook/ko/advanced_ko_rag) 를 보며 LLM 스터디를 진행하고 있었습니다.
* `UnstructuredURLLoader`을 사용하여 페이지의 글자들을 가져올 떄 무한 로딩이 걸리는 이슈가 발생하였습니다. 30초 내외로 로드가 완료되어야 정상인데 2분 이상 로드가 완료되지 않는 현상이 발견되었습니다.
* Ubuntu 환경에서도 현상을 재현해보았지만 큰 문제가 없이 로드가 되어 사용할 수 있었습니다.

### 예시 코드

```python
from langchain_community.document_loaders import UnstructuredURLLoader

version = "v4.49.0"

urls = [
    f"<https://huggingface.co/docs/transformers/{version}/ko/pipeline_tutorial>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/autoclass_tutorial>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/preprocessing>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/training>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/run_scripts>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/tokenizer_summary>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/attention>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/pad_truncation>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/pipeline_webserver>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/tasks_explained>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/hpo_train>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/tasks/sequence_classification>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/tasks/token_classification>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/tasks/question_answering>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/tasks/language_modeling>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/tasks/masked_language_modeling>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/tasks/translation>",
    f"<https://huggingface.co/docs/transformers/{version}/ko/tasks/summarization>",
]
loader = UnstructuredURLLoader(urls=urls)
docs = loader.load()
```

---

## 접근 방법

1. 네트워크가 제한된 망을 사용하고 있던 상황이라 네트워크 이슈라고 생각하여 HTTP 통신을 시도해보았습니다.
   1. `httpx` 를 사용하여 URL 중 하나에 get 요청을 보내어 정상적으로 요청을 받는 지 확인하였습니다.
   2. 요청 결과는 정상적으로 받아왔었습니다.
2. 라이브러리 자체의 문제라고 생각하여 Github에 있는 코드들을 확인 및 직접 실행하여 Blocking이 걸리는 부분을 확인하였습니다.
   1. [UnstructuredURLLoader](https://github.com/langchain-ai/langchain/blob/7e62e3a137814b6813e11b602b2f78df1dec8d14/libs/community/langchain_community/document_loaders/url.py#L13C7-L13C28)
   2. [unstructured.partition.auto.partition](https://github.com/Unstructured-IO/unstructured/blob/347a4e5d9ee42f32c1186f0f0dada93bf9910778/unstructured/partition/auto.py#L30)
   3. [unstructured.file\_utils.filetype](https://github.com/Unstructured-IO/unstructured/blob/main/unstructured/file_utils/filetype.py)
   4. importlib.import\_module("magic")
      1. importlib 자체는 라이브러리 로드가 되기 떄문에 magic 문제라 생각하고 magic을 실행하였습니다.
   5. magic
3. `python-magic` 라이브러리를 Windows에서 사용할 때 발생하는 원인 등을 조사하였습니다.
   1. Windows에서는 python-magic 대신 `python-magic-bin` 을 사용하여 magic 라이브러리를 사용할 수 있도록 해야 합니다.
   2. unstructed issue에 다른 유저가 관련 이슈를 올린 것을 확인하였습니다.
      1. [https://github.com/Unstructured-IO/unstructured/issues/3438](https://github.com/Unstructured-IO/unstructured/issues/3438)
4. 최종적으로 `python-magic-bin==0.4.14` 를 설치하였을 떄 정상적으로 작동하였습니다.