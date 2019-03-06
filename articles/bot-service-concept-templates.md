---
title: Bot Service 템플릿 | Microsoft Docs
description: Bot Service를 사용하여 봇을 만들 때 사용할 수 있는 다양한 템플릿을 살펴봅니다.
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ce7b82d6d3bffe57d758370525cdd7dcc6cddea6
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999800"
---
# <a name="bot-service-templates"></a>Bot Service 템플릿

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot Service에는 봇 빌드를 시작하는 데 도움이 되는 5가지 템플릿이 포함되어 있습니다. 이러한 템플릿은 바로 사용 가능하며 모든 기능을 갖춘 봇을 제공하므로 신속하게 시작할 수 있습니다. [봇을 만들 때](bot-service-quickstart.md) 봇에 대한 템플릿과 SDK 언어를 선택합니다.

각 템플릿은 봇의 핵심 기능을 기준으로 시작점을 제공합니다. 

## <a name="basic-bot"></a>Basic 봇
대화 상자를 사용하여 사용자 입력에 응답하는 봇을 만들려면 기본 템플릿을 선택합니다. **기본** 템플릿은 시작을 위한 최소한의 파일과 코드 집합을 갖는 봇을 만듭니다. 이 봇은 사용자가 무엇을 입력하는 다시 에코합니다. 이 템플릿을 사용하여 봇에서 대화 흐름을 빌드할 수 있습니다.

## <a name="form-bot"></a>Form 봇
단계별 대화를 통해 사용자로부터 입력을 수집하는 봇을 만들려면 **양식** 템플릿을 선택합니다. Form 봇은 사용자로부터 특정 정보 집합을 수집하도록 설계되었습니다. 예를 들어, 사용자의 샌드위치 순서를 구하도록 설계된 봇은 빵 종류, 토핑 선택, 샌드위치 크기 등의 정보를 수집해야 합니다.

C# 언어의 양식 템플릿에서 만든 봇은 [FormFlow](dotnet/bot-builder-dotnet-formflow.md)를 사용하여 양식을 관리하고 Node.js 언어의 양식 템플릿으로 만든 봇은 [waterfalls](nodejs/bot-builder-nodejs-dialog-waterfall.md)로 양식을 관리합니다.

## <a name="language-understanding-bot"></a>Language Understanding 봇
자연어 모델을 사용하여 사용자 의도를 이해하는 봇을 만들려면 **Language understanding** 템플릿을 선택합니다. 이 템플릿은 <a href="https://www.luis.ai" target="_blank">LUIS(Language Understanding)</a>를 사용하여 자연어 이해를 제공합니다.

사용자가 "가상 현실 기업 관련 뉴스를 가져와" 같은 메시지를 보내면 봇은 LUIS를 사용하여 메시지의 의미를 해석합니다. LUIS를 사용하여 사용자의 입력에 담긴 의도와(뉴스 찾기) 존재하는 주요 엔터티(가상 현실 기업)를 해석하는 HTTP 엔드포인트를 신속하게 배포할 수 있습니다. LUIS는 애플리케이션과 관련이 있는 일련의 의도 및 엔터티를 지정할 수 있고 언어 이해 애플리케이션 빌드 프로세스를 안내합니다.

Language understanding 템플릿을 사용하여 봇을 만들 때 Bot Service는 빈 해당 LUIS 애플리케이션을 만듭니다(즉 항상 `None` 반환). 사용자 입력을 해석할 수 있게 LUIS 애플리케이션 모델을 업데이트하려면 <a href="https://www.luis.ai" target="_blank">LUIS</a>에 로그인하고 **내 애플리케이션**을 선택한 다음, 나에 대해 만들어진 서비스 애플리케이션을 선택하고, 의도를 만들고, 엔터티를 지정한 다음, 애플리케이션을 교육합니다.

## <a name="question-and-answer-bot"></a>질문 및 답변 봇
질문 및 대답 쌍 같은 반 구조화된 데이터를 유용하고 구분되는 답으로 거르는 봇을 만들려면 **질문 및 답변** 템플릿을 선택합니다. 이 템플릿은 <a href="https://qnamaker.ai">QnA Maker</a> 서비스를 사용하여 질문을 구문 분석하고 답변을 제공합니다. 

질문 및 답변 템플릿으로 봇을 만들 때는 <a href="https://qnamaker.ai">QnA Maker</a>에 로그인하고 새 QnA 서비스를 만들어야 합니다. 이 QnA 서비스는**QnAKnowldegebasedId** 및 **QnASubscriptionKey**에 대한 [앱 설정](bot-service-manage-settings.md) 값을 업데이트하는 데 사용할 수 있는 기술 자료 ID와 구독 키를 제공합니다. 이 값이 설정되면 봇은 QnA 서비스가 자신의 기술 자료에 가지고 있는 모든 질문에 답할 수 있습니다.

## <a name="proactive-bot"></a>Proactive 봇
사용자에게 사전 메시지를 보내는 봇을 만들려면 **사전 템플릿**을 선택합니다. 일반적으로 봇이 사용자에게 직접 보내는 각각의 메시지는 사용자의 이전 입력과 관련이 있습니다. 그러나 일부 경우 사용자의 가장 최근 메시지와 직접 관련이 없는 사용자 정보를 봇이 보내야 할 수 있습니다. 이러한 종류의 메시지를 **사전 메시지**라고 합니다. 사전 메시지는 다양한 시나리오에서 유용할 수 있습니다. 예를 들어, 봇이 타이머나 미리 알림을 설정할 경우 해당 시간이 되면 사용자에게 알려야 할 수 있습니다. 또는 봇이 외부 이벤트에 대한 알림을 받았을 때 이 정보를 사용자에게 알려야 할 수 있습니다. 

사전 템플릿을 사용하여 봇을 만들 때 몇 가지 Azure 리소스가 자동으로 만들어져 리소스 그룹에 추가됩니다. 기본적으로 이러한 Azure 리소스는 다양한 사전 메시징 시나리오를 사용하도록 설정하기 위해 이미 구성되어 있습니다. 

| 리소스 | 설명 |
|----|----|
| Azure Storage | 큐를 만드는 데 사용됩니다. |
| Azure Function App | 큐에 메시지가 있을 때마다 트리거되는 `queueTrigger` Azure Function입니다. [Direct Line](https://docs.microsoft.com/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts)을 사용하여 Bot Service와 통신합니다. 이 함수는 봇 바인딩을 사용하여 트리거의 페이로드 일부로 메시지를 보냅니다. 이 예제 함수는 사용자의 메시지를 있는 그대로 큐로부터 전달합니다.
| Bot 서비스 | 내 봇. 사용자로부터 메시지를 수신하고, 메시지를 Azure 큐에 추가하며, Azure Function에서 트리거를 수신하고, 다시 트리거의 페이로드를 통해 수신한 메시지를 보내는 논리가 포함됩니다. |

다음 다이어그램은 사전 템플릿을 통해 봇을 만들 때 트리거된 이벤트가 어떻게 작동하는지 보여 줍니다.

![Proactive 봇 예제 개요](~/media/bot-proactive-diagram.png)

이 프로세스는 사용자가 Bot Framework 서버를 통해 봇에 메시지를 보낼 때(1단계) 시작됩니다.

## <a name="next-steps"></a>다음 단계
다양한 템플릿에 대해 알아보았으므로 봇에서의 Cognitive Services 사용에 대해 살펴보겠습니다.

> [!div class="nextstepaction"]
> [봇에 대한 Cognitive Services](bot-service-concept-intelligence.md)
