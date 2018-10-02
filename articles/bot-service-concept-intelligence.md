---
title: Cognitive Services | Microsoft Docs
description: Microsoft Cognitive Services를 통해 봇에 인공 지능을 추가하여 더 유용하고 매력적으로 만드는 방법에 대해 알아봅니다.
keywords: 언어 이해, 정보 추출, 음성 인식, 웹 검색
author: RobStand
ms.author: rstand
manager: rstand
ms.topic: article
ms.prod: bot-framework
ms.date: 12/17/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 05841f6d708b8d22d29fb44b217e97b06053dbb4
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707999"
---
# <a name="cognitive-services"></a>Cognitive Services

Microsoft Cognitive Services를 사용하면 컴퓨터 비전, 음성, 자연어 처리, 정보 추출 및 웹 검색 분야의 전문가에 의해 개발된, 성장하고 있는 강력한 AI 알고리즘을 활용할 수 있습니다. 서비스는 다양한 AI 기반 작업을 간소화함으로써 최신 인텔리전스 기술을 단 몇 줄의 코드만으로 봇에 추가하는 빠른 방법을 제공합니다. API는 최신 언어 및 플랫폼에 통합됩니다. 또한 API는 지속적으로 개선되고, 학습하며, 영리해지므로 환경을 항상 최신 상태로 유지합니다. 

지능형 봇은 마치 사람들이 보는 것 같이 세계를 볼 수 있는 것처럼 반응합니다. 정보를 검색하며 다양한 출처에서 정보를 추출하여 유용한 답변을 제공합니다. 무엇보다 지속적으로 기능을 개선하기 위해 더 많은 환경을 획득하면서 학습합니다. 

## <a name="language-understanding"></a>언어 이해

사용자와 봇 간 상호 작용은 대부분 자유 형식이므로 봇은 자연스럽고 상황에 맞게 언어를 이해해야 합니다. Cognitive Service Language API는 강력한 언어 모델을 제공하여 사용자가 원하는 것을 파악하고, 주어진 문장에서 개념 및 엔터티를 식별하며, 궁극적으로 봇이 적절한 동작으로 응답할 수 있도록 합니다. 5개의 API는 맞춤법 검사, 감정 분석, 언어 모델링 및 텍스트로부터의 적절하고 다양한 정보 추출과 같은 여러 텍스트 분석 기능을 지원합니다. 

Cognitive Services는 언어 이해를 위해 다음과 같은 API를 제공합니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/language-understanding-intelligent-service-luis" target="_blank">LUIS(Language Understanding Intelligent Service)</a>는 미리 빌드되거나 사용자 지정 학습된 언어 모델을 사용하여 자연어를 처리할 수 있습니다. 자세한 내용은 [봇을 위한 언어 이해](v4sdk/bot-builder-concept-luis.md)에서 확인할 수 있습니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="_blank">Text Analytics API</a>는 텍스트에서 감성, 핵심 문구, 주제 및 언어를 파악합니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-spell-check-api" target="_blank">Bing Spell Check API</a>는 강력한 맞춤법 검사 기능을 제공하며, 이름, 브랜드 이름 및 은어 간의 차이점을 인식할 수 있습니다.

- <a href="https://docs.microsoft.com/en-us/azure/machine-learning/studio/text-analytics-module-tutorial" target ="_blank">Azure Machine Learning Studio의 텍스트 분석 모델</a>를 사용하면 단어의 기본형 분석 및 텍스트 전처리와 같은 텍스트 분석 모델을 빌드하고 운용할 수 있습니다. 이러한 모델은 문서 분류 또는 정서 분석과 같은 문제를 해결하는 데 유용할 수 있습니다.

Microsoft Cognitive Services를 통한 [언어 이해][language]에 대해 자세히 알아보세요.

## <a name="knowledge-extraction"></a>정보 추출

Cognitive Services는 네 가지 정보 API를 제공합니다. 이를 통해 체계가 없는 텍스트에서 명명된 엔터티 또는 구를 파악하고, 개인별 권장 사항을 추가하고, 사용자 쿼리의 자연어 해석에 따라 추천 자동 완성을 제공하며, 학술 논문과 맞춤형 FAQ 서비스와 같은 다른 연구를 검색할 수 있습니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/entity-linking-intelligence-service" target="_blank">Entity Linking Intelligence Service</a>는 텍스트에 언급된 관련 엔터티로 체계가 없는 텍스트에 주석을 추가합니다. 컨텍스트에 따라 동일한 단어 또는 구는 다른 내용을 참조할 수 있습니다. 이 서비스는 제공된 텍스트의 컨텍스트를 이해하고 사용자 텍스트에서 각 엔터티를 파악합니다.    

- <a href="https://www.microsoft.com/cognitive-services/en-us/knowledge-exploration-service" target="_blank">Knowledge Exploration Service</a>는 사용자 쿼리의 자연어 해석을 제공하고, 주석이 추가된 해석을 반환하여 다양한 검색 및 사용자가 입력할 내용을 예측하는 자동 완성 환경을 지원합니다. 즉각적인 쿼리 완성 제안 및 예측 쿼리 구체화는 사용자 고유의 데이터와 응용 프로그램별 문법을 기반으로 하여 사용자가 빠른 쿼리를 수행할 수 있도록 합니다.    

- <a href="https://www.microsoft.com/cognitive-services/en-us/academic-knowledge-api" target="_blank">Academic Knowledge API</a>는 <a href="https://www.microsoft.com/en-us/research/project/microsoft-academic-graph/" target="_blank">Microsoft Academic Graph</a>로부터 학술 연구 논문, 작성자, 저널, 회의, 토픽 및 대학교를 반환합니다. Knowledge Exploration Service의 도메인별 예제로 구축된 Academic Knowledge API는 수억 개의 연구 관련 엔터티를 통한 검색 기능이 있는 그래픽과 유사한 대화 상자를 사용하여 기술 자료를 제공합니다. 토픽, 교수, 대학 또는 회의를 검색하고, API에서는 관련 출판물 및 관련 엔터티를 제공합니다. 또한 문법은 “Papers by Michael Jordan about machine learning after 2010(Michael Jordan이 저술한 2010년 이후 머신 러닝에 관한 논문)”과 같은 자연스러운 쿼리도 지원합니다.

- <a href="https://qnamaker.ai" target="_blank">QnA Maker</a>는 손쉽게 사용할 수 있는 무료 REST API이며 자연스러운 대화형 방식으로 사용자의 질문에 응답하도록 AI를 학습하는 웹 기반 서비스입니다. QnA Maker는 최적화된 머신 러닝 논리 및 업계 최고의 언어 처리를 통합하는 기능을 통해 질문과 답변 쌍과 같이 반구조적 데이터를 분명하고 유용한 답변으로 추출합니다.

Microsoft Cognitive Services를 통한 [정보 추출][knowledge]에 대해 자세히 알아보세요.

## <a name="speech-recognition-and-conversion"></a>음성 인식 및 전환

화자 인식과 더불어 음성-텍스트 및 텍스트-음성 변환을 위한 업계 최고의 알고리즘을 활용하도록 봇에 고급 음성 기술을 추가하려면 Speech API를 사용합니다. Speech API는 높은 정확도의 다양한 시나리오를 포함하는 기본 제공 언어 및 어쿠스틱 모델을 사용합니다. 

추가로 사용자 지정해야 하는 응용 프로그램은 CRIS(Custom Recognition Intelligent Service)를 사용할 수 있습니다. 이를 통해 응용 프로그램의 어휘나 심지어 사용자의 말하기 스타일에도 맞게 조정하여 음성 인식기의 언어 및 어쿠스틱 모델을 보정할 수 있습니다.

Cognitive Services에서 음성을 처리하거나 합성하는 데 사용할 수 있는 세 가지 Speech API는 다음과 같습니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/speech-api" target="_blank">Bing Speech API</a>는 음성-텍스트 및 텍스트-음성 변환 기능을 제공합니다.
- <a href="https://www.microsoft.com/cognitive-services/en-us/custom-recognition-intelligent-service-cris" target="_blank">CRIS(Custom Recognition Intelligent Service)</a>를 사용하면 음성-텍스트 변환을 응용 프로그램의 어휘나 사용자의 말하기 스타일에 맞게 조정하는 사용자 지정 음성 인식 모델을 만들 수 있습니다.
- <a href="https://www.microsoft.com/cognitive-services/en-us/speaker-recognition-api" target="_blank">Speaker Recognition API</a>는 음성을 통한 화자 식별 및 인증이 가능합니다.

다음 리소스는 봇에 음성 인식 기능을 추가하는 데 관한 추가 정보를 제공합니다.

* [앱 비디오를 위한 봇 대화 개요](https://channel9.msdn.com/events/Build/2017/P4114)
* [UWP 또는 Xamarin 앱을 위한 Microsoft.Bot.Client 라이브러리](https://aka.ms/BotClient)
* [봇 클라이언트 라이브러리 샘플](https://aka.ms/BotClientSample)
* [음성 지원 웹 채팅 클라이언트](https://aka.ms/BFWebChat)

Microsoft Cognitive Services를 사용한 [음성 인식 및 변환][speech]에 대해 자세히 알아보세요.

## <a name="web-search"></a>Web Search

Bing Search API를 사용하면 봇에 지능형 웹 검색 기능을 추가할 수 있습니다. 코드 몇 줄로 수십억 개의 웹 페이지, 이미지, 비디오, 뉴스 및 다른 결과 형식에 액세스할 수 있습니다. 더 나은 관련성을 위해 지리적 위치, 시장 또는 언어별로 결과를 반환하도록 API를 구성할 수 있습니다. 성인 콘텐츠를 필터링하는 유해 정보 차단 및 특정 날짜에 따라 결과를 반환하는 새로 고침과 같이 지원되는 검색 매개 변수를 사용하여 검색을 추가로 사용자 지정할 수 있습니다.

Cognitive Services에서 사용할 수 있는 5가지 Bing Search API가 있습니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-web-search-api" target="_blank">Web Search API</a>는 단일 API 호출을 사용하여 웹, 이미지, 비디오, 뉴스 및 관련 검색 결과를 제공합니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-image-search-api" target="_blank">Image Search API</a>는 향상된 메타데이터(주요 색상, 이미지 종류 등)를 포함한 이미지 결과를 반환하며, 결과를 사용자 지정하는 여러 이미지 필터를 지원합니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-video-search-api" target="_blank">Video Search API</a>는 다양한 메타데이터(비디오 크기, 품질, 가격 등)를 포함한 비디오 결과를 검색하며, 결과를 사용자 지정하는 여러 비디오 필터를 지원합니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-news-search-api" target="_blank">News Search API</a>는 검색 쿼리와 일치하거나 인터넷에서 현재 유행하는 전세계 뉴스 기사를 찾습니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-autosuggest-api" target="_blank">Autosuggest API</a>는 즉각적인 쿼리 완성 제안을 제공하여 사용자가 입력할 글자 수를 줄여 주면서 더 빠르게 검색 쿼리를 완성합니다. 

Microsoft Cognitive Services를 통한 [웹 검색][search]에 대해 자세히 알아보세요.

## <a name="image-and-video-understanding"></a>이미지 및 비디오 이해

Vision API는 고급 이미지 및 비디오 이해 기술을 봇에 제공합니다. 최신 알고리즘을 사용하면 이미지 또는 비디오를 처리하고 작업으로 다시 변환할 수 있는 정보를 다시 가져올 수 있습니다. 예를 들어, 개체, 사람의 얼굴, 연령, 성별 또는 심지어 감정까지 인식할 수 있습니다. 

Vision API는 다양한 이미지 이해 기능을 지원합니다. 성인용 콘텐츠를 식별하고, 색상을 예측 및 강조하며, 이미지의 콘텐츠를 분류하고, 광학 문자 인식을 수행하며, 전체 영어 문장을 사용하여 이미지를 설명할 수 있습니다. Vision API는 또한 이미지 또는 비디오 썸네일의 지능적 생성이나 비디오 출력의 안정화 같은 여러 이미지 및 비디오 처리 기능을 지원합니다.

Cognitive Services는 이미지 또는 비디오를 처리하는 데 사용할 수 있는 4개의 API를 제공합니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/computer-vision-api" target="_blank">Computer Vision API</a>는 이미지에 대한 다양한 정보(예: 개체 또는 사용자)를 추출하고, 이미지에 성인용 콘텐츠가 포함되었는지 여부를 판단하며, 이미지에서 텍스트를 처리합니다(OCR 사용).

- <a href="https://www.microsoft.com/cognitive-services/en-us/emotion-api" target="_blank">Emotion API</a>는 사람 얼굴을 분석하고 그들의 감정을 인간 감정의 가능한 8가지 범주로 인식합니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/face-api" target="_blank">Face API</a>는 사람 얼굴을 감지하고 유사한 얼굴과 비교하여 시각적 유사성에 따라 그룹으로 사용자를 구성할 수도 있습니다.

- <a href="https://www.microsoft.com/cognitive-services/en-us/video-api" target="_blank">Video API</a>는 비디오 출력을 안정화하기 위해 비디오를 분석 및 처리하고, 움직임을 감지하고, 얼굴을 추적하며, 비디오의 동작 썸네일 요약을 생성할 수 있습니다.

Microsoft Cognitive Services를 통한 [이미지 및 비디오 이해][vision]에 대해 자세히 알아보세요.

## <a name="additional-resources"></a>추가 리소스

<a href="https://docs.microsoft.com/azure/cognitive-services" target="_blank">Cognitive Services 설명서</a>에서 각 제품 및 해당 API 참조에 관한 포괄적인 설명서를 확인할 수 있습니다.

[language]: https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home
[search]: https://docs.microsoft.com/en-us/azure/cognitive-services/bing-web-search/search-the-web
[vision]: https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/home
[knowledge]: https://docs.microsoft.com/en-us/azure/cognitive-services/kes/overview
[speech]: https://docs.microsoft.com/en-us/azure/cognitive-services/speech/home
[location]: https://docs.microsoft.com/en-us/azure/cognitive-services/
