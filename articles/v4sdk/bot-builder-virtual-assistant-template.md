---
title: 가상 도우미 템플릿 개요 | Microsoft Docs
description: 가상 도우미 템플릿에 대한 자세한 정보
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ce3ab86d5716250e24a44268f5e5fc39fbdd3398
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/24/2019
ms.locfileid: "66214168"
---
# <a name="virtual-assistant---template-outline"></a>가상 도우미 - 템플릿 개요

> [!NOTE]
> 이 토픽은 SDK v4 버전에 적용됩니다. 

가상 도우미 템플릿은 대화형 환경 구축을 통해 확인한 여러 우수 사례를 결합하고, Bot Framework 개발자에게 매우 유용하다고 판단한 구성 요소의 통합을 자동화합니다. 이 섹션에서는 템플릿이 작동하는 방식에 대한 이유를 설명하는 데 도움이 되는 주요 의사 결정에 대한 일부 배경에 대해 설명합니다.

기능      | 설명 |
------------ | -------------
소개 | 대화 시작 시 [적응형 카드]()가 있는 소개 메시지입니다.
입력 표시기  | 대화 중에 자동화된 시각적 입력 표시기이며, 장기 실행 작업에 대해 반복합니다.
기본 LUIS 모델  | **Cancel**, **Help**, **Escalate** 등과 같은 일반적인 의도를 지원합니다.
기본 대화 | 기본 사용자 정보 및 취소/도움말 의도에 대한 중단 논리를 캡처하는 대화 흐름입니다.
기본 응답  | 기본 의도 및 대화에 대한 텍스트 및 음성 응답입니다.
FAQ | [QnA Maker](https://www.qnamaker.ai)와 통합하여 기술 자료에서 일반적인 질문에 답변합니다. 
잡담 | 일반적인 쿼리에 대한 표준 답변을 제공하는 전문가 잡담 모델입니다([자세한 정보](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base)).
디스패처 | LUIS 또는 QnA Maker에서 지정된 발화를 처리해야 하는지 여부를 식별하는 통합 [Dispatch](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) 모델입니다.
언어 지원 | 영어, 프랑스어, 이탈리아어, 독일어, 스페인어 및 중국어로 사용할 수 있습니다.
스크립트(Transcript) | Azure Storage에 저장된 모든 대화에 대한 음성 텍스트입니다.
원격 분석  | [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/)와 통합하여 모든 대화에 대한 원격 분석을 수집합니다.
분석 | 대화 환경에 대한 인사이트를 시작할 수 있는 Power BI 대시보드 예제입니다.
자동화된 배포 | Azure ARM 템플릿을 사용하여 앞에서 언급한 모든 서비스를 쉽게 배포합니다.

## <a name="introduction-card"></a>소개 카드

많은 대화형 환경에서 중요한 문제는 최종 사용자가 시작하는 방법을 알지 못하여 봇이 응답하는 데 가장 적합하지 않을 수 있는 일반적인 질문으로 이어진다는 것입니다. 첫 인상이 중요합니다! 소개 카드는 최종 사용자에게 봇의 기능을 소개하는 기회를 제공하며, 사용자가 시작하는 데 사용할 수 있는 몇 가지 초기 질문을 제안합니다. 또한 봇의 특성을 드러내는 좋은 기회입니다.

필요에 맞게 설정할 수 있도록 간단한 소개 카드를 기본으로 제공합니다. 사용자가 온보딩 대화 상자(소개 카드의 시작하기 단추로 트리거)를 완료한 다음, 대화를 시작하면 대화 내용을 적용한 카드가 표시됩니다.

![소개 카드 예제](./media/enterprise-template/vabotintrocard.png)

## <a name="basic-language-understanding-luis-intents"></a>기본 LUIS(Language Understanding) 의도

모든 봇은 대화형 언어 이해의 기본 수준을 처리해야 합니다. 예를 들어 인사말은 모든 봇에서 쉽게 처리해야 기본적인 작업입니다. 일반적으로 개발자는 이러한 기본 의도를 만들고 초기 학습 데이터를 제공하여 시작해야 합니다. 가상 도우미 템플릿은 시작하기 위한 LU 파일 예제를 제공하여 모든 프로젝트마다 매번 이러한 파일을 만들지 않도록 하며, 기본 수준의 기능을 즉시 사용할 수 있습니다.

LU 파일에서 영어, 중국어, 프랑스어, 이탈리아어, 독일어 및 스페인어로 제공하는 의도는 다음과 같습니다.

의도       | 발화 샘플 |
-------------|-------------|
취소       |*취소*, *괜찮아*|
Escalate     |*사람에게 이야기할 수 있나요?*|
FinishTask   |*완료*, *모두 완료되었습니다*|
GoBack       |*뒤로 이동*|
도움말         |*도와주실 수 있으세요?*|
Repeat       |*다시 말할 수 있나요?*|
SelectAny    |*이러한 항목 중 하나*|
SelectItem   |*첫 번째 항목*|
SelectNone   |*없음*|
ShowNext     |*자세히 표시*|
ShowPrevious |*이전 표시*|
StartOver    |*restart*|
중지         |*stop*|

[LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 형식은 MarkDown과 비슷하여 쉽게 수정하고 소스를 제어할 수 있습니다. 그런 다음, [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 도구를 사용하여 .LU 파일을 LUIS 모델로 변환한 다음, 포털 또는 관련 [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI(명령줄) 도구를 통해 LUIS 구독에 게시할 수 있습니다.

## <a name="telemetry"></a>원격 분석

봇의 사용자 참여에 대한 인사이트를 제공하는 것은 매우 가치 있는 것으로 입증되었습니다. 이 인사이트는 봇이 대답할 수 없는 사람들의 질문과 함께 사용자 참여 수준, 사용자들이 사용 중인 봇의 기능(의도)에 대해 이해하는 데 도움이 될 수 있습니다. 예를 들어 새로운 QnA Maker 문서를 통해 해결될 수 있는 봇의 지식에 관한 차이를 강조합니다.

Application Insights의 통합은 중요한 운영/기술적 인사이트를 즉시 제공하지만, 특정 봇 관련 이벤트, 즉 LUIS 및 QnA Maker 작업과 함께 보내고 받는 메시지를 캡처하는 데에도 사용할 수 있습니다.

봇 수준의 원격 분석은 본질적으로 기술 및 운영 원격 분석에 연결되어 있어 지정된 사용자 질문에 응답한 방법과 그 반대의 방법도 검사할 수 있습니다.

QnA Maker 및 LuisRecognizer SDK 클래스의 래퍼 클래스와 결합된 미들웨어 구성 요소는 일단의 이벤트를 일관되게 수집하는 세련된 방법을 제공합니다. 그런 다음, 이러한 일관된 이벤트는 PowerBI와 같은 도구와 함께 Application Insights 도구에서 사용할 수 있습니다.

예제 PowerBI 대시보드는 Bot Framework 솔루션 github 리포지토리에 속하며, 모든 가상 도우미 템플릿에서 별도 작업 없이 즉시 작동합니다. 자세한 내용은 [Analytics](https://github.com/Microsoft/AI/blob/master/docs/readme.md#analytics) 섹션을 참조하세요.

![Analytics 예제](./media/enterprise-template/powerbi-conversationanalytics-luisintents.png)

## <a name="dispatcher"></a>디스패처

대화형 환경의 첫 번째 이동에서 좋은 효과를 가져오는 데 사용된 주요 디자인 패턴은 LUIS(Language Understanding)와 QnA Maker를 활용하는 것이었습니다. LUIS는 봇에서 최종 사용자를 위해 수행할 수 있었던 작업으로 학습되었고, QnA Maker는 더 일반적인 지식으로 학습되었습니다.

들어오는 모든 발화(질문)는 분석을 위해 LUIS로 라우팅되었습니다. 지정된 발화의 의도가 확인되지 않는 경우 의도가 [없음]으로 표시되었습니다. 그런 다음, QnA Maker를 사용하여 최종 사용자의 답변을 찾으려고 시도했습니다.

이 패턴이 잘 작동하는 동안 문제가 발생할 수 있는 두 가지 주요 시나리오가 있었습니다.

- 종종 LUIS 모델과 QnA Maker의 발화가 약간 겹치는 경우 LUIS에서 QnA Maker에 전달해야 할 때 질문을 처리하려고 시도하는 이상한 동작이 발생할 수 있었습니다.
- 둘 이상의 LUIS 모델이 있을 때 봇에서 각각을 호출하고 특정 형태의 의도 평가 비교를 수행하여 지정된 발화를 보내는 위치를 식별해야 했습니다. 공통 기준이 없어 모델 간의 점수 비교가 효과적으로 작동하지 않았으므로 사용자 환경이 좋지 않았습니다.

[Dispatcher](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig)는 구성된 각 LUIS 모델에서 발화를 추출하고, QnA Maker에서 질문을 추출하고, 중앙의 디스패치 LUIS 모델을 만들어 이러한 사용자 환경에 대한 세련된 솔루션을 제공합니다.

이렇게 하면 봇에서 발화를 처리해야 하는 LUIS 모델 또는 구성 요소를 빠르게 식별할 수 있으며, 이전과 같이 [없음] 의도뿐만 아니라 QnA Maker 데이터도 의도 처리의 최상위 수준에서 고려되도록 할 수 있습니다.

또한 이 디스패치 도구를 사용하면 배포하기 전에 문제를 강조하는 LUIS 모델과 QnA Maker 기술 자료 간에 혼동과 겹침 문제를 강조하는 평가를 수행할 수 있습니다.

디스패처는 템플릿을 사용하여 만든 각 프로젝트의 핵심에서 사용됩니다. 디스패치 모델은 `MainDialog` 클래스 내에서 대상이 LUIS 모델인지, 아니면 QnA인지를 식별하는 데 사용됩니다. LUIS의 경우 평소와 같이 보조 LUIS 모델을 호출되어 의도와 엔터티를 반환합니다. 디스패처는 중단 감지에도 사용됩니다.

![디스패치 예제](./media/enterprise-template/dispatchexample.png)

## <a name="qna-maker"></a>QnA Maker

[QnA Maker](https://www.qnamaker.ai/)는 개발자가 아닌 사용자가 일반적인 지식을 질문 및 답변의 쌍 형식으로 조정하는 기능을 제공합니다. 이 지식은 FAQ 데이터 원본, 제품 설명서 및 QnaMaker 포털 내에서 대화형으로 가져올 수 있습니다.

QnA Maker 모델의 두 예(FAQ용 모델과 잡담용 모델)는 CognitiveModels의 QnA 폴더에 [LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 파일 형식으로 제공됩니다. 그러면 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown)이 배포 스크립트의 일부로 사용되어 QnA Maker JSON 파일을 만든 다음, [QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI(명령줄) 도구에서 이 파일을 사용하여 항목을 QnA Maker 기술 자료에 게시합니다.

![QnA ChitChat 예제](./media/enterprise-template/qnachitchatexample.png)

## <a name="content-moderator"></a>Content Moderator

Content Moderator는 잠재적인 불경한 언어를 감지하고 PII(개인 식별 정보)를 확인하는 선택적 구성 요소입니다. 이는 봇에 통합되어 봇에서 불경한 언어에 대응할 수 있게 하거나 사용자가 PII 정보를 공유하는 경우에 유용합니다. 예를 들어 PII 정보가 검색되면 봇에서 사람에게 사과하고 전달하거나 원격 분석 기록을 저장하지 않을 수 있습니다.

TurnState 개체의 ```TextModeratorResult```를 통해 텍스트와 화면을 표시하는 미들웨어 구성 요소가 제공됩니다.

# <a name="next-steps"></a>다음 단계
가상 도우미를 만들고 배포하는 방법에 대한 자세한 내용은 [시작](https://github.com/Microsoft/AI/tree/master/docs#tutorials)을 참조하세요. 

# <a name="additional-resources"></a>추가 리소스
가상 도우미 템플릿의 전체 소스 코드는 [GitHub](https://github.com/Microsoft/AI/)에서 찾을 수 있습니다.

