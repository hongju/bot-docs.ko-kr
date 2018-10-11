---
title: 엔터프라이즈 봇 템플릿의 자세한 개요 | Microsoft Docs
description: 엔터프라이즈 봇 템플릿에서 디자인을 결정하는 방법 알아보기
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6f295794ca7d3cc17688337e70df2a52cdb665ed
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46708830"
---
# <a name="enterprise-template---detailed-overview"></a>엔터프라이즈 템플릿 - 자세한 개요

> [!NOTE]
> 이 토픽은 SDK의 v4 버전에 적용됩니다. 

엔터프라이즈 봇 템플릿은 대화형 환경을 빌드하면서 식별한 몇 가지 모범 사례를 함께 제공하며, Azure Bot Service 개발자에게 매우 유용한 구성 요소 통합을 자동화합니다. 이 섹션에서는 템플릿이 왜 그렇게 작동하는지를 설명하는 데 도움이 되도록 중요 결정에 대한 일부 배경에 대해 다룹니다.

## <a name="introduction-card"></a>소개 카드

많은 대화형 환경의 주요 문제점은 최종 사용자가 시작하는 방법을 몰라 봇이 응답하기에 최적이 아닐 수도 있는 일반적인 질문으로 이어진다는 점입니다. 첫 인상이 중요합니다! 소개 카드는 최종 사용자에게 봇의 기능을 소개하는 기회를 제공하며, 사용자가 시작하는 데 사용할 수 있는 몇 가지 초기 질문을 제안합니다. 봇의 성격을 드러내기에도 좋은 기회입니다.

간단한 소개 카드는 필요에 따라 조정할 수 있는 표준으로 제공됩니다.

## <a name="basic-language-understanding-luis-intents"></a>기본 LUIS(Language Understanding) 의도

모든 봇은 대화형 언어 이해의 기본 수준을 처리해야 합니다. 예를 들어 인사말은 모든 봇이 쉽게 처리해야 기본적인 것입니다. 일반적으로 개발자는 이러한 기본 의도를 만들고 시작하기 위한 초기 학습 데이터를 제공해야 합니다. 엔터프라이즈 봇 템플릿은 시작하기 위한 예제 LU 파일을 제공하여 모든 프로젝트마다 매번 이러한 파일을 만들지 않도록 하며, 기본 기능을 즉시 제공합니다.

LU 파일은 다음 의도를 영어, 프랑스어, 이탈리아어, 독일어 및 스페인어로 제공합니다.

> Greeting, Help, Cancel, Restart, Escalate, ConfirmYes, ConfirmNo, ConfirmMore, Next, Goodbye

[LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 형식은 손쉬운 수정 및 원본 제어를 가능하게 하는 MarkDown과 유사합니다. [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 도구는 .LU 파일을 LUIS 모델로 변환하는 데 사용됩니다. 그러면 포털이나 연결된 [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI(명령줄) 도구를 통해 LUIS 구독에 게시할 수 있습니다.

## <a name="content-moderator"></a>Content Moderator

Content Moderator는 잠재적인 욕설 감지를 활성화하며 PII(개인 식별 정보) 확인에 도움이 됩니다. 이는 봇이 욕설이나 사용자가 PII 정보를 공유하는 경우에 대응할 수 있도록 봇에 통합하면 유용할 수 있습니다. 예를 들어, 봇은 사과하고 사람에게 전달하거나, PII 정보가 검색되면 원격 분석 레코드를 저장하지 않을 수 있습니다.

미들웨어 구성 요소는 화면에 텍스트로 표시되고 TurnState 개체에서 ```TextModeratorResult```를 통해 나타납니다.

## <a name="telemetry"></a>원격 분석

봇의 사용자 참여에 정보를 제공하는 것은 가치가 높은 것으로 입증되었습니다. 이 정보는 봇이 대답할 수 없는 사람들의 질문과 함께 사용자 참여 수준, 사용자들이 사용 중인 봇의 기능(의도)에 대해 이해하는 데 도움이 될 수 있습니다. 예를 들어 새로운 QnAMaker 문서를 통해 해결될 수 있는 봇의 지식에 관한 차이를 강조합니다.

Application Insights의 통합은 중요한 운영적/기술적 정보를 즉시 제공하지만, 이는 LUIS 및 QnAMaker 작업과 함께 전송되고 수신되는 메시지와 같이 특정 봇 관련 이벤트를 캡처하는 데도 사용될 수 있습니다.

봇 수준 원격 분석은 본질적으로 기술적 및 운영적 원격 분석과 관련되어 주어진 사용자 질문에 대한 답변 방법을 검사하고 그 반대로도 할 수 있습니다.

QnAMaker 및 LuisRecognizer SDK 클래스의 래퍼 클래스와 결합된 미들웨어 구성 요소는 일관성 있는 일련의 이벤트를 수집하는 세련된 방법을 제공합니다. 그런 다음, 이러한 일관성 있는 이벤트를 PowerBI와 같은 도구와 함께 Applicatin Insights 도구에서 사용할 수 있습니다.

예제 PowerBI 대시보드가 엔터프라이즈 봇 템플릿을 사용하여 생성된 각 프로젝트와 함께 제공됩니다. 자세한 내용은 [PowerBI](bot-builder-enterprise-template-powerbi.md) 섹션을 참조하세요.

## <a name="dispatcher"></a>디스패처

대화형 환경의 첫 번째 물결에 좋은 영향을 미치는 데 사용된 주요 디자인 패턴은 LUIS(Language Understanding) 및 QnAMaker를 활용하는 것이었습니다. LUIS는 봇이 최종 사용자를 위해 수행할 수 있는 태스크로 학습되고, QnAMaker는 더 일반적인 정보로 학습되었습니다.

들어오는 모든 발언(질문)은 분석을 위해 LUIS로 라우팅되었습니다. 지정된 발언의 의도가 명확하지 않은 경우 None 의도로 표시되었습니다. 그런 다음, QnAMaker는 최종 사용자를 위한 답변을 찾는 데 사용되었습니다.

이 패턴이 잘 작동하던 중에 문제가 발생할 수 있는 두 가지 주요 시나리오가 있었습니다.

- 종종 LUIS 모델 및 QnAMaker에서 발언이 약간 중복되는 경우 QnaMaker로 이동되어야 할 때 LUIS가 질문을 처리하려고 시도할 수도 있는 생소한 동작이 발생할 수 있었습니다.
- 둘 이상의 LUIS 모델이 있는 경우 봇은 각 모델을 호출하고, 주어진 발언을 보내는 대상을 확인하기 위해 일종의 의도 평가 비교를 수행해야 했습니다. 일반적인 기준이 없었기 때문에 모델 간 점수 비교는 효과적으로 작동하지 않아 사용자 환경의 저하로 이어졌습니다.

[디스패처](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig)는 각 구성된 LUIS 모델의 발언 및 QnAMaker의 질문을 추출하고, 중앙 디스패치 LUIS 모델을 만들어서 이에 대한 세련된 솔루션을 제공합니다.

이렇게 하면 어떤 LUIS 모델 또는 구성 요소가 주어진 발언을 처리해야 하는지를 봇이 신속하게 식별할 수 있으며, 앞서와 같이 None 의도만이 아니라 QnAMaker 데이터도 최상위 수준에서 고려합니다.

또한 이 디스패치 도구는 LUIS 모델 및 QnAMaker 기술 자료에서 혼동 및 겹침을 강조 표시하는 평가를 활성화하여 배포 전에 문제를 강조 표시합니다.

디스패처는 엔터프라이즈 봇 템플릿을 사용하여 생성되는 각 프로젝트의 핵심에서 사용됩니다. 디스패치 모델은 `MainDialog` 클래스 내에서 대상이 LUIS 모델 또는 QnA인지 여부를 식별하는 데 사용됩니다. LUIS의 경우 보조 LUIS 모델이 호출되어 의도 및 엔터티를 정상적으로 반환합니다.

## <a name="qnamaker"></a>QnAMaker

[QnAMaker](https://www.qnamaker.ai/)는 개발자가 아닌 사용자가 일반 지식을 질문 및 답변 쌍의 형식으로 큐레이팅할 수 있는 기능을 제공합니다. 이 지식은 FAQ 데이터 원본, 제품 설명서 및 QnaMaker 포털 내에서 대화식으로 가져올 수 있습니다.

QnA 항목의 예제 집합은 CogSvcModels의 QnA 폴더 내에 [LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 파일 형식으로 제공됩니다. [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown)은 배포 스크립트의 일부로 사용되어 [QnAMaker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI(명령줄) 도구가 QnAMaker 기술 자료에 항목을 게시하는 데 사용하는 QnAMaker JSON 파일을 생성합니다.
