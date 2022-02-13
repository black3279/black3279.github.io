---
layout: post
author: Bruce Lee
title: Text Mining
---

# Text Mining
- 비정형 데이터가 많아짐에 따라 비정형화 데이터 중 하나인 텍스트 데이터의 수집량이 증가하고 있다.
- Text Mining 이란 Data Mining 과 Text Data 를 합친 용어이다.
  - Data Mining 의 주된 목적은 데이터로부터 숨겨진 의미있는 데이터를 추출하는 것이다.
  - Text Data 는 논문이나 Article, Email, News, Blog, Web page 등 의 데이터를 말한다.

## Retrieval
- 대량의 정보에서 의미있는 소량의 정보를 검색해내는 분야이다. (Filtering Information)

## Mining
- 사람들이 생각하는 유의미한 정보 (하지만 알려지지않은) 를 발견해내는 분야이다. (Discovering Knowlege)

## Organization
- 주어진 데이터를 어떻게 잘구조화하여 활용할 것이냐와 관련된 분야이다. (자연어 처리 등)

## Information Retrieval
- IR 분야는 비구조화된 데이터로부터 사용자가 원하는 소량의 데이터 (이미지, 비디오, 오디오 등) 를 어떻게 검색해낼지에 대한 분야이다.
- 대표적으로는 웹 검색 엔진을 예로 들 수 있다.

### IR Process
- 질의가 들어왔을 때, 해당 질의가 특정 문서와 어떤 관련이 있는지를 정의하는 함수를 밝혀내기 위한 것이다.
- Boolean model 은 가장 오래된 IR 모델로 문서나 질의를 단어의 집합으로 표현하는 방식이다.
- Vector space 모델은 쿼리와 질의를 임의의 단어의 집합으로 표현하되, 0 또는 1로 표현하는 Boolean 모델과 달리 단어의 중요도를 고려하여 벡터로 표현하며, 그 위에서 둘 사이의 유사도를 평가하는 measure 를 함께 활용한다.
- Language 모델은 문서로부터 질의를 만들어낸다고 했을 때 얼마나 효과적으로 질의를 생성해낼지를 측정하고 이를 기반으로 둘 간의 관련성을 계산해보겠다는 것이다.

### Boolean Model
- 문서가 있을 때, 문서를 단어의 집합으로 표현하여 단어가 있으면 1, 없으면 0 으로 표현한다.
- 여러번 등장하더라도 1 로 표현하게 된다.
- 문서들을 단어들의 term 매트릭스로 표현할 수 있고 그 안에서 포함여부를 확인할 수 있으며, 단어의 순서는 고려되지 않는다.
- 간편하게 간단하게 문서를 이해할 수 있는 방식이다.

### Vector Space Model
- 단어를 축으로 concept space 를 만든 후에 해당 concept 이 얼마나 중요한지 중요도를 표현한다.
- Boolean Model 을 개선한 모델이다.

## Text Mining
- Document Classification 은 각각의 문서를 하나 또는 다수의 클래스로 분류하는 것이 목적이다.
- 단어 혹은 간단한 문장이 들어왔을 때, 해당 대상을 하나의 클래스로 분류하는 것이 목표이다.
- Sentimental 한 관점에서 Positive 인지 Negative 인지, 특정한 토픽과 얼마나 관련성이 있는지를 알아내는 것 등이 주요 Task 라고 보면 된다.
- Topic Model 은 Clustering 목적을 달성하는데 있어서, 문서들이 어떤 토픽으로 분류되는지를 알아내는 과정에서 각각의 토픽에 어떤 단어들이 관련이 있는지에 대한 확률분포도 함께 알아낼 수 있는 모델이다.

### NLP 구성요소
- Natural language understanding (NLU) : 주어진 입력값을 자연어의 유용한 표현과 맵핑시켜 이해하기 위한 분야이다.
- Natural language generation (NLG) : 의미있는 문장과 구문을 자연어의 문맥 속에서 생성해내기 위한 분야이다.
- Lexical analysis 는 주어진 문장에서 각 단어 별로, 의미 구간 별로 어떤 의미를 갖는지 품사 분류를 하는 것이다.
- Syntatic analysis 는 전체 문장에서 주어가 어디인지, 동사, 전치사가 어디인지 트리 형태의 구조화된 형태로 표현하기 위한 분야이다.
- Semantic analysis 는 Parsing 된 각 단어들의 의미, 관련성 등을 분석하는 분야이다.

### Information Extraction
- 주어진 비정형 텍스트 데이터로부터 정형화된 정보를 추출하기 위한 것이다.
- 추출된 정보를 기반으로 정보그래프 등을 만들어내는데 활용된다.

### Named Entity Recognition
- Information extraction 의 서브 분야로, 각 단어 혹은 의미역별로 해당이 어떤 타입에 해당하는 것인지를 인식하는 것이 목적이다.
- 언급된 개체명을 미리 정의한 분류 기준에 따라 감지하여 분류하는 작업으로 인명, 단체, 기관, 장소, 시간 등을 지식 그래프 등에 저장할 수 있다.

### Word Sense Disambiguation
- 단어가 여러가지 의미를 가지고 있을 때, 한 문장에서 쓰인 단어의 의미를 밝혀내기 위한 분야이다.
- Bass 를 연주하고 있는 연주자를 fish 로 잘못 번역되는 등의 오류를 교정하기 위한 분야이다.

### Machine Translation
- 기계 번역은 심층 신경망이 등장하며 성장한 분야로, 임의의 문장이 들어왔을 때 그것을 다른 언어의 문장 혹은 단어로 바꾸기 위한 분야이다.

### Document Summarization
- 장문의 문서를 짧게 요악하는 과정을 자동으로 수행하기 위한 분야이다.
- Extract summarization 은 문장 중 어떤 문장이 주제문장인지를 찾아내는 분류 문제에 대한 분야이다.
- Abstractive summarization 은 전체 문서에서 문서를 대표할 수 있는 문장을 새롭게 만들어내는 분야이다.
