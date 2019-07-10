---
title: 언어 이해 | Microsoft Docs
description: Microsoft Cognitive Services를 통해 봇에 인공 지능을 추가하여 더 유용하고 매력적으로 만드는 방법에 대해 알아봅니다.
keywords: LUIS, intent, recognizer, dispatch tool, qna, qna maker
author: ivorb
ms.author: v-ivorb
manager: rstand
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 09/19/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0c0918b0ac0a10927bd8d7c52283e74b4fd480bf
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404639"
---
# <a name="language-understanding"></a>언어 이해

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇은 구조적 및 단계별 대화에서 자유 형식 및 개방형 대화까지 다양한 대화 스타일을 사용할 수 있습니다. 봇은 사용자가 언급한 내용에 따라 대화 흐름에서 다음 작업을 결정해야 하며, 개방형 대화에서는 광범위한 사용자 응답이 있습니다.

| 단계별 | 열기 |
|------|------|
| 여행 봇입니다. 항공편 찾기, 호텔 찾기, 렌트카 찾기 옵션 중 하나를 선택합니다. | 여행 예약을 도와드릴 수 있습니다. 뭐하고 싶으세요? |
| 다른 작업이 필요하신가요? 예 또는 아니요를 클릭합니다. | 다른 작업이 필요하신가요? |

사용자와 봇 간 상호 작용은 대부분 자유 형식이며, 봇은 자연스럽고 상황에 맞게 언어를 이해해야 합니다. 이 문서에서는 Language Understanding(LUIS)이 사용자가 원하는 것을 파악하고, 문장에서 개념 및 엔터티를 식별하며, 궁극적으로 봇이 적절한 동작으로 응답할 수 있도록 도와주는 방법에 대해 설명합니다.

## <a name="recognize-intent"></a>의도 인식

[LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/home)는 말하는 내용에서 하고 싶은 것 즉, 사용자의 **의도**를 확인하여 도움을 주므로 봇이 적절하게 응답할 수 있습니다. LUIS는 봇에게 말하는 내용이 예측 가능한 구조 또는 특정 패턴을 따르지 않는 경우에 특히 유용합니다. 봇에게 사용자가 응답을 말하거나 입력하는 방식의 대화형 사용자 인터페이스가 있는 경우 사용자의 음성 또는 텍스트 입력인 *발언*에 대해 무한 변형이 있을 수 있습니다.

여행 봇의 사용자가 항공권 예약을 요청할 수 있는 많은 방법을 예로 들어 보겠습니다.

![항공편 예약에 대한 다양한 형식의 발언](media/cognitive-services-add-bot-language/cognitive-services-luis-utterances.png)

이러한 발언에는 다른 구조가 있을 수 있으며 생각해 본 적 없는 “항공권”에 대한 다양한 동의어가 포함될 수 있습니다. 봇의 경우 모든 발언과 일치하고 논리를 작성하는 것과 같은 단어가 포함된 서로 다른 의도를 구분하는 것은 어려운 과제일 수 있습니다. 또한 봇은 위치 및 시간 같이 중요한 단어인 *엔터티*를 추출해야 합니다. LUIS는 의도 및 엔터티를 문맥에 맞게 식별하여 이 프로세스를 쉽게 만들어줍니다.

자연어 입력에 대한 봇을 설계할 경우 봇이 인식해야 할 의도 및 엔터티를 결정하고 봇이 취하는 동작에 연결하는 방법에 대해 생각해야 합니다. [luis.ai](https://www.luis.ai)에서 사용자 지정 의도 및 엔터티를 정의하고, 각 의도에 대한 예제를 제공하고 예제 내에서 엔터티에 레이블을 지정함으로써 해당 동작을 지정합니다.

봇은 LUIS에서 인식하는 의도를 사용하여 대화 항목을 확인하거나 대화 흐름을 시작합니다. 예를 들어 사용자가 "항공권을 예약하고 싶습니다"라고 말하는 경우 봇은 BookFlight 의도를 감지하고 항공편 검색을 시작하기 위해 대화 흐름을 호출합니다. LUIS는 의도를 트리거하는 원래 발언에서뿐만 아니라 나중에 대화 흐름에서도 대상 도시 및 출발 날짜와 같은 엔터티를 감지합니다. 봇에 필요한 모든 정보가 갖춰지면 사용자의 의도를 수행할 수 있습니다.

![봇과의 대화는 BookFlight 의도에 의해 트리거됩니다.](media/cognitive-services-add-bot-language/cognitive-services-luis-conversation-high-level.png)

### <a name="recognize-intent-in-common-scenarios"></a>일반적인 시나리오에서 의도 인식

개발 시간을 절약하기 위해 LUIS는 봇의 일반 범주에 대한 일반적인 발언을 인식하는 미리 학습된 언어 모델을 제공합니다. 

**미리 빌드된 도메인**은 미리 학습되고 사용할 준비가 된 의도 및 엔터티의 컬렉션으로 약속, 알림, 관리, 적합성, 엔터테인먼트, 통신, 예약 등과 같은 일반적인 시나리오에 대해 잘 협력합니다. **유틸리티** 미리 빌드된 도메인은 봇이 취소, 확인, 도움말, 반복, 중지 같은 일반적인 작업을 처리하는 데 도움이 됩니다. LUIS에서 제공하는 [미리 빌드된 도메인](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-how-to-use-prebuilt-domains)을 살펴봅니다.

**미리 빌드된 엔터티**는 봇이 날짜, 시간, 숫자, 온도, 통화, 지리 및 기간 같은 일반적인 유형의 정보를 인식하는 데 도움이 됩니다. LUIS가 인식할 수 있는 형식의 배경은 [미리 빌드된 엔터티 사용](https://docs.microsoft.com/azure/cognitive-services/LUIS/pre-builtentities)을 참조하세요.

## <a name="how-your-bot-gets-messages-from-luis"></a>봇이 LUIS에서 메시지를 가져오는 방법

LUIS를 설정하고 연결하면 봇은 의도 및 엔터티를 포함하는 JSON 응답을 반환하는 메시지를 LUIS 앱으로 보낼 수 있습니다. 그런 다음, 봇의 _턴 처리기_에서 [턴 컨텍스트](~/v4sdk/bot-builder-basics.md#defining-a-turn)를 사용하여 의도에 따라 LUIS 응답에서 대화 흐름을 라우팅할 수 있습니다. 

![의도 및 엔터티를 봇에 전달하는 방법](./media/cognitive-services-add-bot-language/cognitive-services-luis-message-flow-bot-code.png)

봇에서 LUIS 앱 사용을 시작하려면 [Language Understanding에 LUIS 사용](https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0)을 확인하세요.

## <a name="best-practices-for-language-understanding"></a>Language Understanding의 모범 사례

봇용 언어 모델을 설계할 경우 다음 사례를 고려합니다.

### <a name="consider-the-number-of-intents"></a>의도 수를 고려

LUIS 앱은 발언을 여러 범주 중 하나로 분류하여 의도를 인식합니다. 많은 의도 중에서 올바른 범주를 확인하는 것이 의도를 분별할 수 있는 LUIS 앱의 역량을 줄일 수 있다는 것은 자연스러운 결과입니다.

의도의 수를 줄이는 한 가지 방법은 계층 디자인을 사용하는 것입니다. 날씨와 관련된 의도 세 개, 가정 자동화와 관련된 의도 세 개, 도움말, 취소 및 인사말의 다른 유틸리티 의도 세 개를 포함하는 개인 비서 봇의 사례를 고려합니다. 동일한 LUIS 앱에 모든 의도를 집어넣는 경우 이미 의도 9개가 있으며, 봇에 기능을 추가하면 결국 수십 개로 될 수 있습니다. 대신, 디스패처 LUIS 앱을 사용하여 사용자의 요청이 날씨, 가정 자동화 또는 유틸리티에 대한 것인지 여부를 확인한 다음, 디스패처가 확인하는 범주에 대해 LUIS 앱을 호출할 수 있습니다. 이 경우 각 LUIS 앱은 의도 3개만으로 시작합니다.

### <a name="use-a-none-intent"></a>None 의도 사용

봇의 사용자가 현재 대화 흐름과 관련이 없거나 예상치 못한 것을 말하는 경우가 종종 있습니다. _None_ 의도는 해당 메시지를 처리하기 위해 제공됩니다.

대체(fallback), 기본 또는 "위에 없음" 사례를 처리하기 위한 의도를 학습하지 않은 경우 LUIS 앱은 메시지를 정의한 의도로만 분류할 수 있습니다. 예를 들어 `HomeAutomation.TurnOn` 및 `HomeAutomation.TurnOff`, 두 의도를 포함한 LUIS 앱이 있다고 합시다. 단지 의도에 불과하고 "금요일에 약속 예약"과 같이 관계없는 것을 입력하는 경우 LUIS 앱은 해당 메시지를 HomeAutomation.TurnOn 또는 HomeAutomation.TurnOff 중 하나로 분류할 수밖에 없습니다. LUIS 앱에 몇 가지 예제를 포함한 `None` 의도가 있는 경우 예기치 않은 발언을 처리하려면 봇의 일부 대체 로직을 제공할 수 있습니다.

`None` 의도는 인식 결과를 개선하는 데 매우 유용합니다. 이 홈 자동화 시나리오에서 "금요일에 약속 예약"은 신뢰도가 낮은 `HomeAutomation.TurnOn` 의도를 생성할 수 있으며, 봇은 이를 거부해야 합니다. `None`으로 올바르게 확인할 수 있도록 `None` 의도 아래의 모델에 이러한 구문을 추가할 수 습니다.

### <a name="review-the-utterances-that-luis-app-receives"></a>LUIS 앱에서 받는 발언 검토

LUIS 앱은 사용자가 전송한 메시지를 검토하여 앱 성능을 개선하기 위한 기능을 제공합니다. 단계별 연습은 [제안된 발언](https://docs.microsoft.com/azure/cognitive-services/LUIS/label-suggested-utterances)을 참조하세요.


## <a name="integrate-multiple-luis-apps-and-qna-services-with-the-dispatch-tool"></a>디스패치 도구를 사용하여 여러 LUIS 앱 및 QnA 서비스 통합

여러 대화 항목을 이해하는 다용도 봇을 빌드하는 경우 각 함수에 대한 서비스를 개별적으로 개발하기 시작한 다음, 모두 함께 통합할 수 있습니다. 이러한 서비스에는 Language Understanding(LUIS) 앱 및 QnAMaker 서비스가 포함될 수 있습니다. 봇이 여러 LUIS 앱, 여러 QnAMaker 서비스 또는 둘의 조합을 결합할 수 있는 몇 가지 예제 시나리오는 다음과 같습니다.

* 개인 비서 봇을 사용하면 사용자는 다양한 명령을 호출할 수 있습니다. 각 범주의 명령은 별도로 개발할 수 있는 "기술"을 만들고, 각 기술마다 LUIS 앱이 있습니다.
* 봇은 많은 기술 자료를 검색하여 자주 묻는 질문(FAQ)에 대한 답변을 찾습니다.
* 비즈니스용 봇에는 고객 계정을 만들고 주문을 배치하기 위한 LUIS 앱과 함께 FAQ에 대한 QnAMaker 서비스도 있습니다.  

### <a name="the-dispatch-tool"></a>Dispatch 도구

Dispatch 도구를 사용하면 적절한 LUIS 및 QnAMaker 서비스에 메시지를 라우팅하는 새 LUIS 앱인 *디스패치 앱*을 만들어 여러 LUIS 앱 및 QnA Maker 서비스를 봇과 통합할 수 있습니다. 하나의 봇에서 여러 LUIS 앱 및 QnA Maker를 결합하는 단계별 자습서는 [디스패치 자습서](./bot-builder-tutorial-dispatch.md)를 참조하세요.

## <a name="use-luis-to-improve-speech-recognition"></a>LUIS를 사용하여 음성 인식을 개선

사용자가 대화하는 봇의 경우 LUIS와 통합하면 봇이 음성을 텍스트로 변환할 때 오해의 소지가 있는 단어를 식별할 수 있습니다.  예를 들어 체스 시나리오에서 사용자는 “기사를 A7으로 이동(Move knight to A7)”하라고 말할 수 있습니다. 사용자 의도에 대한 컨텍스트가 없으면 이 발화는 “밤 287 이동(Move night 287)”으로 잘못 인식될 수 있습니다. 체스의 말 및 좌표를 나타내는 엔터티를 만들고 발언의 엔터티에 레이블을 지정하여 엔터티를 식별하려면 음성 인식에 대한 컨텍스트를 제공합니다. Web Chat, Bot Framework Emulator, Cortana 등의 Bing Speech와 통합된 Bot Framework 채널을 사용하여 [음성 인식 초기화를 사용하도록 설정](https://docs.microsoft.com/azure/bot-service/bot-service-manage-speech-priming?view=azure-bot-service-4.0)할 수 있습니다.  

## <a name="additional-resources"></a>추가 리소스
자세한 내용은 [Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/) 설명서를 참조하세요.
