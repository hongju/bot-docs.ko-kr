---
title: Bot Framework SDK 내의 대화 | Microsoft Docs
description: Bot Framework SDK 내의 대화를 설명합니다.
keywords: 대화 흐름, 의도 인식, 단일 순서, 여러 순서, 봇 대화
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7f35b8a135cdde6ffaf11798a5c0e4a3688d5b4f
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904826"
---
# <a name="conversation-flow"></a>대화 흐름
[!INCLUDE[applies-to](../includes/applies-to.md)]

봇의 대화 흐름을 디자인하려면 사용자가 무엇인가를 말할 때 봇이 응답하는 방법을 결정해야 합니다. 봇은 먼저 사용자의 메시지를 기반으로 작업 또는 대화 항목을 인식합니다. 사용자의 메시지와 관련된 작업 또는 항목(*의도*로 알려짐)을 결정하기 위해 봇은 사용자 메시지의 텍스트에서 단어 또는 패턴을 검색하거나 [Language Understanding](bot-builder-concept-luis.md) 및 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview)와 같은 서비스를 활용할 수 있습니다.

봇이 사용자의 의도를 인식하면 시나리오에 따라 봇은 하나의 턴에서 대화를 완료하여 단일 회신으로 사용자의 요청을 충족할 수 있거나 일련의 턴이 필요할 수 있습니다. 다중 턴 대화 흐름의 경우 Bot Framework SDK는 대화를 추적하기 위한 [상태 관리](./bot-builder-howto-v4-state.md)와 정보 요청을 위한 [프롬프트](bot-builder-prompts.md), 대화 흐름 캡슐화를 위한 [대화 상자](bot-builder-dialog-manage-conversation-flow.md)를 제공합니다.

여러 하위 시스템이 있는 복잡한 봇에서 봇의 각 하위 구성 요소에 대해 하나씩 여러 서비스를 사용하여 의도를 인식할 수 있습니다. [디스패치 도구](bot-builder-tutorial-dispatch.md)는 대화형 하위 시스템을 하나의 봇으로 결합할 때 한 곳에서 여러 서비스의 결과를 가져옵니다.

<!-- 
A conversation identifies a series of activities sent between a bot and a user on a specific channel and represents an interaction between one or more bots and either a _direct_ conversation with a specific user or a _group_ conversation with multiple users.
A bot communicates with a user on a channel by receiving activities from, and sending activities to the user.

- Each user has an ID that is unique per channel.
- Each conversation has an ID that is unique per channel.
- The channel sets the conversation ID when it starts the conversation.
- The bot cannot start a conversation; however, once it has a conversation ID, it can resume that conversation.
- Not all channels support group conversations.
-->

## <a name="single-turn-conversation"></a>단일 턴 대화

가장 간단한 대화형 흐름은 단일 턴(single-turn)입니다. 단일 턴(single-turn) 흐름에서 봇은 사용자의 하나의 메시지와 봇의 하나의 회신으로 구성하여 하나의 턴에서 해당 작업을 완료합니다.

<!-- The following isn't always true, it's a generalization -->

가장 간단한 종류의 단일 턴(single-turn) 봇은 대화 상태를 추적할 필요가 없습니다. 메시지를 받을 때마다 지난 대화형 턴의 지식 없이 현재 들어오는 메시지의 컨텍스트에 따라 응답합니다.

![단일 턴(single-turn) 날씨 봇](./media/concept-conversation/weather-single-turn.png)

날씨 봇은 단일 턴(single-turn) 흐름을 가질 수 있으며 도시 또는 날짜에 대해 질문을 주고받지 않고 사용자에게 날씨 보고서를 제공합니다. 날씨 보고서를 표시하기 위한 모든 논리는 봇이 받은 메시지를 기반으로 합니다. 대화의 각 턴에서 봇은 다음에 수행할 작업 및 대화 흐름 방식을 결정하는 데 사용할 수 있는 [턴 컨텍스트](bot-builder-concept-activity-processing.md#turn-context)를 받습니다.

## <a name="multiple-turns"></a>다중 턴

대부분의 대화 유형은 단일 턴에서 완료될 수 없으므로 봇은 다중 턴 대화 흐름을 가질 수도 있습니다. 여러 대화형 턴이 필요한 일부 시나리오는 다음을 포함합니다.

* 작업을 완료하는 데 필요한 추가 정보를 사용자에게 확인하는 봇 봇은 작업을 충족하기 위한 모든 매개 변수가 있는지 여부를 추적해야 합니다.
* 사용자에게 주문과 같은 프로세스의 단계를 안내하는 봇 봇은 사용자가 단계의 시퀀스에 있는지 추적해야 합니다.

예를 들어 봇이 도시에 대해 물어 "날씨는?"에 응답하는 경우 날씨 봇은 다중 턴 흐름을 갖습니다.

![다중 턴 날씨 봇](./media/concept-conversation/weather-multi-turn.png)

사용자가 도시에 대한 봇의 프롬프트에 응답하고 봇이 "시애틀"을 수신하는 경우 봇은 현재 메시지가 이전 프롬프트 및 날씨를 가져오는 요청의 일부에 대한 응답임을 이해하기 위해 일부 컨텍스트를 저장해야 합니다. 다중 턴 봇은 새 메시지에 적절하게 응답하기 위해 상태를 추적합니다.

자세한 내용은 [대화 및 사용자 상태를 관리하는 방법](bot-builder-howto-v4-state.md)을 참조하세요.

> [!NOTE]
> REST API 클라이언트가 있는 다중 턴 대화는 자신의 상태를 추적해야 합니다(예: 데이터베이스 또는 Table Storage에서).

## <a name="conversation-topics"></a>대화 항목

둘 이상의 작업 유형을 처리하도록 봇을 디자인할 수 있습니다. 예를 들어 사용자에게 인사말을 제공하고, 주문, 취소 및 도움말을 얻기 위해 다양한 대화 흐름을 제공하는 봇을 가질 수 있습니다. 다양한 작업 또는 대화 항목에 대한 대화 간 이 전환을 처리하는 한 가지 방법은 현재 메시지에서 의도(사용자가 원하는 것)를 인식하는 것입니다.

### <a name="recognize-intent"></a>의도 인식

Bot Framework SDK는 메시지를 처리하여 의도를 확인할 수 있는 _인식기_를 제공하므로 봇은 적절한 대화형 흐름을 시작할 수 있습니다. 인식기의 _인식_ 비동기 메서드를 호출하여 해당 메시지 콘텐츠에서 사용자의 의도를 확인합니다. 그런 다음, 인식기의 상위 예측을 가져오도록 결과에서 _상위 점수 매기기 의도 가져오기_ 메서드를 호출할 수 있습니다.

인식기는 정규식, 언어 이해 또는 개발하는 다른 논리를 사용할 수 있습니다. 다음은 가능한 인식기의 예제입니다.

* 사용자가 기술 자료에서 다루는 문제를 질문하는 경우를 감지하는 데 QnA Maker를 사용하는 인식기
* 사용자가 도움을 요청할 수 있는 방법의 예제로 서비스를 학습하고, `Help` 의도로 매핑하는 데 LUIS(Language Understanding)를 사용하는 인식기
* 명령을 찾는 데 정규식을 사용하는 사용자 지정 인식기
* 입력을 변환하는 데 서비스를 사용하는 사용자 지정 인식기

### <a name="consider-how-to-interrupt-conversation-flow-or-change-topics"></a>대화 흐름을 중단하거나 항목을 변경하는 방법을 고려합니다.

대화에 있는지 추적하는 한 가지 방법은 [대화 상태](bot-builder-howto-v4-state.md)를 사용하여 현재 활성 항목 또는 완료된 시퀀스의 단계에 대한 정보를 저장하는 것입니다.

봇이 복잡해지면 스택에서 발생하는 대화 흐름의 시퀀스를 상상할 수도 있습니다. 예를 들어 봇은 새 주문 흐름을 호출한 다음, 제품 검색 흐름을 호출합니다. 그런 다음, 사용자는 제품을 선택하고, 제품 검색 흐름 완료를 확인한 다음, 주문을 완료합니다.

그러나 대화는 이러한 선형 논리 경로를 드물게 따릅니다. 사용자는 "스택"에서 통신하지 않고 대신 마음이 자주 바뀌는 경향이 있습니다. 다음 예제를 살펴보세요.

![사용자가 예기치 않은 것을 말하는 경우](./media/concept-conversation/interruption.png)

봇이 흐름의 스택을 논리적으로 생성했을 수 있지만, 사용자는 완전히 다른 어떤 작업을 수행하거나 현재 항목과 관련이 없을 수 있는 질문을 하려고 할 수 있습니다. 예제에서 사용자는 흐름이 예상하는 예/아니요 응답을 제공하는 대신 질문을 합니다. 흐름은 어떻게 응답해야 할까요?

* 사용자가 먼저 질문에 답변할 것을 요구합니다.
* 사용자가 이전에 수행한 모든 작업을 무시하고, 전체 흐름 스택을 다시 설정하고, 사용자의 질문에 답변함으로써 처음부터 시작합니다.
* 사용자의 질문에 답변한 다음, 해당 예/아니요 질문으로 돌아가서 다시 시작해 봅니다.

최상의 솔루션은 시나리오의 세부 정보와 사용자가 봇의 응답을 합리적으로 예상하는 방법에 따라 결정되므로 이 질문에 대한 올바른 답변은 없습니다. [대화 상자를 사용](bot-builder-dialog-manage-conversation-flow.md)하고 [중단을 처리하는](bot-builder-howto-handle-user-interrupt.md) 방법을 참조하여 대화 흐름을 관리합니다.

## <a name="conversation-lifetime"></a>대화 수명

<!-- Note: these activities are dependent on whether the channel actually sends them. Also, we should add links -->
봇은 대화에 추가되거나, 다른 멤버가 대화에 추가 또는 제거되거나, 대화 메타데이터가 변경될 때마다 _대화 업데이트_ 작업을 받습니다.
사용자에게 인사하거나 자신을 소개하여 봇이 대화 업데이트 작업에 반응하도록 할 수 있습니다.

봇은 사용자가 대화를 종료했음을 나타내기 위해 _대화 종료_ 작업을 받습니다. 봇은 대화가 끝나감을 나타내기 위해 _대화 종료_ 작업을 보낼 수 있습니다.
대화에 대한 정보를 저장하는 경우 대화가 종료할 때 해당 정보를 삭제할 수 있습니다.

<!--  Types of conversations -->

봇은 사용자에게 정보의 여러 부분에 대한 메시지를 표시하는 다중 턴 상호 작용을 지원할 수 있습니다. 매우 구체적인 작업에 초점을 맞추거나 여러 유형의 작업을 지원할 수 있습니다.
Bot Framework SDK에는 봇에 자연 언어 "질문 및 답변" 기능을 추가하기 위한 LUIS(Language Understanding) 및 QnA Maker에 대한 일부 기본 제공 지원이 있습니다.

## <a name="conversations-channels-and-users"></a>대화, 채널 및 사용자

대화는 특정 사용자와의 _직접_ 대화 또는 여러 사용자와의 _그룹_ 대화일 수 있습니다.
봇은 사용자에게서 작업을 받고, 사용자에게 작업을 보내 채널에서 사용자와 통신합니다.

* 각 사용자는 채널별로 고유한 ID를 갖습니다.
* 각 대화는 채널별로 고유한 ID를 갖습니다.
* 채널은 대화를 시작할 때 대화 ID를 설정합니다.
* 봇은 대화를 시작할 수 없습니다. 그러나 대화 ID를 가지면 해당 대화를 다시 시작할 수 있습니다.
* 일부 채널은 그룹 대화를 지원하지 않습니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [대화 상자를 사용하여 간단한 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)

<!-- In addition, your bot can send activities back to the user, either _proactively_, in response to internal logic, or _reactively_, in response to an activity from the user or channel.-->
<!--TODO: Link to messaging how tos.-->
