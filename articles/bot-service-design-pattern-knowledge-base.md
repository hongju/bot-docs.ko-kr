---
title: 지식 봇 설계 | Microsoft 문서
description: 사용자의 입력 또는 쿼리에 대한 정보를 찾아서 반환하는 지식 봇을 설계하는 다양한 방법을 알아봅니다.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 12/13/2017
ms.openlocfilehash: e228209b4d239a05f9c76203e9fd2fb342c14d36
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999290"
---
# <a name="design-knowledge-bots"></a>지식 봇 설계

거의 모든 토픽에 대한 정보를 제공하는 지식 봇을 설계할 수 있습니다. 예를 들어 어떤 지식 봇은 "이 회의에는 어떤 봇 이벤트가 있습니까?", "다음 레게 쇼는 언제입니까?", "Tame Impala가 누구입니까?" 등의 이벤트에 대한 질문에 답할 수 있습니다. 또 어떤 지식 봇은 "내 운영 체제를 업데이트하려면 어떻게 해야 하나요?" 또는 "암호를 재설정하려면 어디로 가야 하나요?" 같은 IT 관련 질문에 대답할 수 있습니다. 또 어떤 봇은 "John Doe는 누구인가요?" 또는 "Jane Doe의 이메일 주소는 무엇입니까?" 같은 연락처 질문에 대답할 수 있습니다. 

어떤 사용 사례를 목표로 지식 봇을 설계했든, 기본 목표는 언제나 동일합니다. 그것은 바로 SQL 데이터베이스의 관계형 데이터, 비 관계형 저장소의 JSON 데이터, 문서 저장소의 PDF 같은 데이터 본문을 활용하여 사용자가 요청한 정보를 찾아서 반환하는 것입니다. 

## <a name="search"></a>검색

검색 기능은 봇 내에서 매우 유용한 도구일 수 있습니다. 

첫 번째로, "유사 항목 검색"은 사용자에게 정확한 입력을 요구하지 않고도 봇이 사용자의 질문과 관련될 가능성이 높은 정보를 반환하게 합니다. 예를 들어 사용자가 음악 지식 봇에게 "Tame Impala" 대신 "Impala"에 대한 정보를 요청하는 경우 봇은 이 입력과 관련될 가능성이 가장 높은 정보로 응답할 수 있습니다.

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/fuzzySearch2.png)

검색 점수는 특정 검색 결과에 대한 신뢰도 수준을 나타내며, 봇이 그에 따라 결과를 정렬하거나 신뢰도 수준에 따라 통신을 조정할 수 있습니다. 예를 들어 신뢰도 수준이 높으면 봇이 "검색과 가장 일치하는 이벤트는 다음과 같습니다"로 응답할 수 있습니다.

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/searchScore2.png)

신뢰도 수준이 낮으면 봇이 "음... 이러한 이벤트를 찾고 계신가요?"로 응답할 수 있습니다.

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/searchScore1.png)

### <a name="using-search-to-guide-a-conversation"></a>검색을 사용하여 대화 안내

봇을 빌드하려는 이유가 기본 검색 엔진 기능 사용인 경우 봇이 전혀 필요 없습니다. 사용자가 웹 브라우저의 일반적인 검색 엔진에서 얻을 수 없고 대화형 인터페이스에서 얻을 수 있는 것은 무엇인가요? 

지식 봇은 일반적으로 대화를 안내하도록 설계될 때 가장 효과적입니다. 대화는 사용자와 봇 간의 교환으로 이루어지며, 이를 통해 기본 검색으로 할 수 없는 방식으로 확인 질문을 던지고 옵션을 제공하고 결과의 유효성을 검사하는 기회가 봇에 제공됩니다. 예를 들어 다음 봇은 사용자가 원하는 정보를 찾을 때까지 데이터 집합을 패싯 및 필터링하는 단계를 사용자에게 안내합니다.

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/guidedConvo1.png)

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/guidedConvo2.png)

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/guidedConvo3.png)

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/guidedConvo4.png)

봇은 각 단계에서 사용자 입력을 처리하고 관련 옵션을 제공하여 사용자를 사용자가 찾는 정보로 안내합니다. 봇은 일단 정보를 제공한 후에는 나중에 유사한 정보를 보다 효율적으로 찾는 방법에 대한 지침을 제공할 수도 있습니다. 

![다이얼로그 구조](~/media/bot-service-design-pattern-knowledge-base/Training.png)

### <a name="azure-search"></a>Azure Search

<a href="https://azure.microsoft.com/en-us/services/search/" target="_blank">Azure Search</a>를 사용하면 봇이 쉽게 검색, 패싯 및 필터링할 수 있는 효율적인 검색 인덱스를 만들 수 있습니다. Azure Portal을 사용하여 만든 검색 인덱스를 고려해 보세요.

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/search3.PNG)

데이터 저장소의 모든 속성에 액세스할 수 있기를 원하므로 각 속성을 "retrievable"로 설정합니다. 음악가를 이름으로 검색할 수 있기를 원하므로 **Name** 속성을 "searchable"로 설정합니다. 마지막으로, 음악가의 연대에 대해 필터를 패싯할 수 있기를 원하므로 **Eras** 속성을 "facetable" 및 "filterable"로 표시합니다. 

패싯은 각 값의 크기와 함께 특정 속성의 데이터 저장소에 있는 값을 확인합니다. 예를 들어 이 스크린샷은 데이터 저장소에 5개의 고유한 연대가 있음을 보여줍니다.

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/facet.png)

필터링은 결국 특정 속성의 지정된 인스턴스만 선택합니다. 예를 들어 **Era**가 "Romantic"인 항목만 포함하도록 위의 결과 집합을 필터링할 수 있습니다. 

> [!NOTE]
> Azure Document DB, Azure Search 및 Microsoft Bot Framework를 사용하여 만든 지식 봇의 완전한 예제에 대한 <a href="https://github.com/ryanvolum/AzureSearchBot" target="_blank">샘플 봇</a>을 참조하세요.
> 
> 단순성을 위해, 위의 예제는 Azure Portal을 사용하여 만드는 검색 인덱스를 보여줍니다. 
> 인덱스를 프로그래밍 방식으로 만들 수도 있습니다.

## <a name="qna-maker"></a>QnA Maker

일부 지식 봇의 목적은 단순히 FAQ(질문과 대답)에 대답하는 것일 수 있습니다. 
<a href="https://www.microsoft.com/cognitive-services/en-us/qnamaker" target="_blank">QnA Maker</a>는 이 사용 사례를 위해 특별히 설계된 강력한 도구입니다. QnA Maker는 기존 FAQ 사이트의 질문과 대답을 스크랩하는 기능을 내장하고 있으며, 고유한 사용자 지정 질문 및 답변 목록을 수동으로 구성할 수 있습니다. QnA Maker는 자연어 처리 기능을 갖추고 있어서 예상과 약간 다르게 표시되는 질문에도 대답할 수 있습니다. 그러나 의미 체계 언어 이해 기능을 갖추고 있지는 않습니다. 예를 들어 강아지가 개의 한 종류라는 것을 확인할 수 없습니다. 

QnA Maker 웹 인터페이스를 사용하면 세 가지 질문과 답변 쌍으로 기술 자료를 구성할 수 있습니다. 

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/KnowledgeBaseConfig.png)

그런 다음, 일련의 질문을 통해 테스트할 수 있습니다. 

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/exampleQnAConvo.png)

봇은 기술 자료에 구성된 질문과 직접 매핑되는 질문에 올바르게 대답합니다. 그러나 "내 차를 가져가도 될까요?"라는 질문에 올바르게 대답하지 않습니다. 왜냐하면 이 질문은 그 구조가"내 보드카를 가져가도 될까요?"라는 질문과 가장 비슷하기 때문입니다. QnA Maker가 잘못된 답변을 제공하는 이유는 기본적으로 단어의 의미를 이해하지 못하기 때문입니다. "차"가 무알콜 음료의 일종이라는 사실을 모릅니다. 따라서 "주류는 허용되지 않습니다"라고 답변합니다.

> [!TIP]
> 고유한 QnA 쌍을 만든 후 대화 아래에 있는 "검사" 단추를 사용하여 제공된 잘못된 답변을 대신하는 대안 답변을 선택하여 봇을 테스트 및 재교육합니다. 

## <a name="luis"></a>LUIS

일부 지식 봇은 사용자의 메시지를 분석하여 사용자의 의도를 파악할 수 있도록 NLP(자연어 처리) 기능이 필요합니다. 
[LUIS(Language Understanding)](https://www.luis.ai)는 NLP 기능을 봇에 추가하는 빠르고 효과적인 수단을 제공합니다. LUIS는 Bing 및 Cortana의 미리 작성된 기존 모델이 요구 사항을 충족하는 경우 이 모델을 사용할 수 있을 뿐 아니라 사용자 고유의 특수 모델을 만들 수 있습니다. 

거대한 데이터 집합을 처리할 때 반드시 한 엔터티의 모든 변형을 사용하여 NLP 모델을 교육할 수 있는 것은 아닙니다. 예를 들어 음악 재생 봇에서 사용자가 "레게 재생", "Bob Marley 재생" 또는 "One Love 재생"이라는 메시지를 보낼 수 있습니다. 봇이 모든 음악가, 장르 및 곡 이름을 학습하지 않고도 이러한 각 메시지를 의도 "playMusic"에 매핑할 수 있지만, NLP 모델은 엔터티가 장르인지, 음악가인지, 곡인지 식별할 수 없습니다. NLP 모델을 사용하여 "음악" 형식의 일반 엔터티를 식별하면 봇이 해당 엔터티의 데이터 저장소를 검색하고 그 위치에서 계속 진행할 수 있습니다. 

## <a name="combining-search-qna-maker-andor-luis"></a>Search, QnA Maker 및/또는 LUIS 결합

Search, QnA Maker 및 LUIS는 그 자체로도 강력한 도구이지만, 결합하면 둘 이상의 기능을 처리하는 지식 봇을 빌드할 수 있습니다.

### <a name="luis-and-search"></a>LUIS 및 Search

[앞에서 설명한](#search) 음악 축제 봇 예제에서, 봇은 라인업을 나타내는 단추를 표시하여 대화를 안내합니다. 그러나 이 봇은 LUIS를 사용하여 "Romit Girdhar는 어떤 종류의 음악을 재생하나요?" 같은 질문의 의도 및 엔터티를 확인함으로써 자연어 이해를 통합할 수도 있습니다. 그러면 봇은 음악가 이름을 사용하여 Azure Search 인덱스를 검색할 수 있습니다. 
 
잠재적인 값이 너무 많기 때문에 모델에 가능한 모든 음악가 이름을 교육할 수는 없지만, LUIS가 가까운 엔터티를 올바르게 식별할 수 있도록 대표적인 예제를 제공할 수 있습니다.  예를 들어 음악가 예제를 제공하여 모델을 교육하는 방법을 생각해 볼 수 있습니다. 

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/answerGenre.png)
![대화 구조](~/media/bot-service-design-pattern-knowledge-base/answerGenreOneWord.png)

"what kind of music do the beatles play?" 같은 새로운 발언을 사용하여 모델을 테스트할 때, LUIS는 성공적으로 의도 "answerGenre"를 확인하고 엔터티 "the beatles"를 식별합니다. 그러나 "what kind of music does the devil makes three play?"처럼 긴 질문을 제출하면 LUIS는 "the devil"을 엔터티로 식별합니다.

![대화 구조](~/media/bot-service-design-pattern-knowledge-base/devilMakesThreeScore.png)

기본 데이터 집합을 대표하는 예제 엔터티로 모델을 교육하면 봇의 언어 이해도를 높일 수 있습니다. 

> [!TIP]
> 일반적으로 모델이 엔터티 인식에서 단어를 너무 적게 인식하는 것보다는 너무 많이 식별하여 오류를 범하는 것이 좋습니다. 예를 들어 "Call John Smith please" 발언에서 "John"을 식별하는 것보다는 "Call John Smith please" 발언에서 "John Smith please"를 식별하는 것이 좋습니다. 검색 인덱스는 "John Smith please" 문구에서 "please"처럼 관련이 없는 단어를 무시합니다. 

### <a name="luis-and-qna-maker"></a>LUIS 및 QnA Maker

일부 지식 봇은 QnA Maker를 LUIS와 함께 사용하여 기본 질문에 대답함으로써 의도를 확인하고, 엔터티를 추출하고, 보다 정교한 대화 상자를 호출합니다. 예를 들어 간단한 IT 지원 센터 봇을 생각해 보세요. 이 봇은 QnA Maker를 사용하여 Windows 또는 Outlook에 대한 기본적인 질문에 대답할 수 있지만, 의도 인식 및 사용자와 봇 간의 통신이 필요한 암호 재설정 같은 시나리오도 지원해야 합니다. 봇이 LUIS와 QnA Maker의 하이브리드를 구현할 수 있는 몇 가지 방법이 있습니다.

1. QnA Maker와 LUIS를 동시에 호출하고, 특정 임계값의 점수를 먼저 반환하는 쪽의 정보를 사용하여 사용자에게 응답합니다. 
2. 먼저 LUIS를 호출하고, 특정 임계값을 충족하는 의도가 없으면, 다시 말해서 "None" 의도가 트리거되면 QnA Maker를 호출합니다. 또는 QnA Maker에 대한 LUIS 의도를 만들고, LUIS 모델에 "QnAIntent"로 매핑되는 예제 QnA 질문을 제공합니다. 
3. 먼저 QnA Maker를 호출하고, 특정 임계값을 충족하는 답변이 없으면 LUIS를 호출합니다. 

Bot Builder SDK는 LUIS 및 QnA Maker를 기본적으로 지원합니다. 따라서 각 도구에 대한 사용자 지정 호출을 구현할 필요 없이 LUIS 및/또는 QnA Maker를 사용하여 대화 상자를 트리거하거나 자동으로 질문에 대답할 수 있습니다. 자세한 내용은 [Bot Builder Dispatch 도구 자습서](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0)를 참조하세요.

> [!TIP]
> LUIS, QnA Maker 및/또는 Azure Search 조합을 구현할 때 각 모델의 임계값 점수를 확인하는 도구를 사용하여 입력을 테스트하세요. LUIS, QnA Maker 및 Azure Search는 서로 다른 채점 기준을 사용하여 점수를 생성하므로 이러한 도구를 통해 생성된 점수를 직접 비교할 수 없습니다. 또한 LUIS와 QnA Maker는 점수를 정규화합니다. 특정 점수가 어떤 LUIS 모델에서는 '좋음'으로 간주되고 다른 모델에서는 그렇지 않을 수 있습니다. 

## <a name="sample-code"></a>샘플 코드

- .NET용 Bot Builder SDK를 사용하여 기본 지식 봇을 만드는 방법을 보여주는 샘플은 GitHub의 <a href="https://aka.ms/qna-with-appinsights" target="_blank">기술 봇 샘플</a>을 참조하세요. 
<!-- TODO: Do not have a current bot sample to work with this
- For a sample that shows how to create more complex knowledge bots using the Bot Builder SDK for .NET, see the <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search" target="_blank">Search-powered Bots sample</a> in GitHub.
-->

[qnamakerTemplate]: https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle
