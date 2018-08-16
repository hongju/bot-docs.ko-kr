---
title: Bot Builder SDK 내의 대화화 | Microsoft Docs
description: Bot Builder SDK 내의 대화를 설명합니다.
keywords: 대화 흐름, 의도 인식, 단일 순서, 여러 순서
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/11/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 679109cad2f7b0c0c5826a47884b98e1149cb380
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/19/2018
ms.locfileid: "39304635"
---
# <a name="conversation-flow"></a>대화 흐름
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇은 대화형 사용자 인터페이스로 간주할 수 있으므로 대화 흐름은 사용자와 상호 작용하는 방법이며 다양한 형태가 될 수 있습니다. 올바른 대화 흐름을 사용하면 사용자의 상호 작용 및 봇의 성능을 향상시킵니다.

봇의 대화 흐름을 디자인하려면 사용자가 무엇인가를 말할 때 봇이 응답하는 방법을 결정해야 합니다. 봇은 먼저 사용자의 메시지를 기반으로 작업 또는 대화 항목을 인식합니다. 사용자의 메시지와 관련된 작업 또는 항목(*의도*로 알려짐)을 결정하기 위해 봇은 사용자 메시지의 텍스트에서 단어 또는 패턴을 검색하거나 [LUIS(Language Understanding)](bot-builder-concept-luis.md) 및 QnA Maker와 같은 서비스를 활용할 수 있습니다. 

봇이 사용자의 의도를 인식하면 시나리오에 따라 봇은 하나의 턴에서 대화를 완료하여 단일 회신으로 사용자의 요청을 충족할 수 있거나 일련의 턴이 필요할 수 있습니다. 다중 턴 대화 흐름의 경우 Bot Builder SDK는 대화를 추적하기 위한 [상태 관리](./bot-builder-howto-v4-state.md)와 정보 요청을 위한 [프롬프트](bot-builder-prompts.md), 대화 흐름 캡슐화를 위한 [대화 상자](bot-builder-dialog-manage-conversation-flow.md)를 제공합니다. 

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



<!-- 
The EchoBot sample in the BotBuilder SDK is a single-turn bot. Here are other examples of single turn conversation flow:
* A bot for getting the weather report, that just tells the user what the weather is, when they say "What's the weather?".
* An IoT bot that responds to "turn on the lights" by calling an IoT service. -->

<!-- The following isn't always true, it's a generalization --> 가장 간단한 종류의 단일 턴(single-turn) 봇은 대화 상태를 추적할 필요가 없습니다. 메시지를 받을 때마다 지난 대화형 턴의 지식 없이 현재 들어오는 메시지의 컨텍스트에 따라 응답합니다.

![단일 턴(single-turn) 날씨 봇](./media/concept-conversation/weather-single-turn.png)

날씨 봇은 단일 턴(single-turn) 흐름을 가지며 도시 또는 날짜에 대해 질문을 주고받지 않고 사용자에게 날씨 보고서를 제공합니다. 날씨 보고서를 표시하기 위한 모든 논리는 봇이 받은 메시지를 기반으로 합니다. 대화의 각 턴에서 봇은 다음에 수행할 작업 및 대화 흐름 방식을 결정하는 데 사용할 수 있는 턴 컨텍스트를 받습니다. 

## <a name="multiple-turns"></a>다중 턴

대부분의 대화 유형은 단일 턴에서 완료될 수 없으므로 봇은 다중 턴 대화 흐름을 가질 수도 있습니다. 여러 대화형 턴이 필요한 일부 시나리오는 다음을 포함합니다.

 * 작업을 완료하는 데 필요한 추가 정보를 사용자에게 확인하는 봇 봇은 작업을 충족하기 위한 모든 매개 변수가 있는지 여부를 추적해야 합니다.
 * 사용자에게 주문과 같은 프로세스의 단계를 안내하는 봇 봇은 사용자가 단계의 시퀀스에 있는지 추적해야 합니다.

예를 들어 봇이 도시에 대해 물어 "날씨는?"에 응답하는 경우 날씨 봇은 다중 턴 흐름을 갖습니다.

![다중 턴 날씨 봇](./media/concept-conversation/weather-multi-turn.png)

사용자가 도시에 대한 봇의 프롬프트에 응답하고 봇이 "시애틀"을 수신하는 경우 봇은 현재 메시지가 이전 프롬프트 및 날씨를 가져오는 요청의 일부에 대한 응답임을 이해하기 위해 일부 컨텍스트를 저장해야 합니다. 다중 턴 봇은 새 메시지에 적절하게 응답하기 위해 상태를 추적합니다.

<!--
```
// TBD: snippet showing receiving message and using ConversationProperties
```
-->

상태 관리에 대한 개요는 [상태 관리](bot-builder-storage-concept.md)를 참조하고, 예제는 [사용자 및 대화 속성을 사용하는 방법](bot-builder-howto-v4-state.md)을 참조하세요.

> [!NOTE]
> REST API 클라이언트가 있는 다중 턴 대화는 자신의 상태를 추적해야 합니다(예: 데이터베이스 또는 테이블 저장소에서). 

## <a name="conversation-topics"></a>대화 항목

둘 이상의 작업 유형을 처리하도록 봇을 디자인할 수 있습니다. 예를 들어 사용자에게 인사말을 제공하고, 주문, 취소 및 도움말을 얻기 위해 다양한 대화 흐름을 제공하는 봇을 가질 수 있습니다. 다양한 작업 또는 대화 항목에 대한 대화 간 이 전환을 처리하는 한 가지 방법은 현재 메시지에서 의도(사용자가 원하는 것)를 인식하는 것입니다. 

### <a name="recognize-intent"></a>의도 인식

Bot Builder SDK는 각 들어오는 메시지를 처리하여 의도를 확인하는 _인식기_를 제공하므로 봇은 적절한 대화형 흐름을 시작할 수 있습니다. _콜백을 수신_하기 전에 인식기는 사용자의 메시지 콘텐츠를 확인하여 의도를 확인한 다음, 턴 컨텍스트 개체에 **상위 의도**로 저장된 수신 콜백 내에서 턴 컨텍스트 개체를 사용하여 봇에 의도를 반환합니다. 

**상위 의도**를 확인하는 인식기는 미들웨어로 개발하는 정규식, LUIS(Language Understanding) 또는 다른 논리를 간단히 사용할 수 있습니다. 다음은 인식기의 예제일 수 있습니다.
   
* 정규식을 사용하여 사용자가 help라는 단어를 말할 때마다 검색하도록 인식기를 설정합니다.
* LUIS(Language Understanding)를 사용하여 사용자가 도움을 요청할 수 있는 방법의 예제로 서비스를 학습하고, "Help" 의도로 매핑합니다.
* 들어오는 작업을 검사하고 메시지를 다른 언어로 검색할 때마다 "translate" 의도를 반환하는 사용자 고유의 인식기 미들웨어를 만듭니다.

자세한 내용은 [LUIS를 사용하여 Language Understanding](bot-builder-concept-luis.md)을 참조하세요. <!-- TODO: ADD THIS TOPIC OR SNIPPET-->

### <a name="consider-how-to-interrupt-conversation-flow-or-change-topics"></a>대화 흐름을 중단하거나 항목을 변경하는 방법을 고려합니다.

대화에 있는지 추적하는 한 가지 방법은 [대화 상태](bot-builder-howto-v4-state.md)를 사용하여 현재 활성 항목 또는 완료된 시퀀스의 단계에 대한 정보를 저장하는 것입니다.

봇이 복잡해지면 스택에서 발생하는 대화 흐름의 시퀀스를 상상할 수도 있습니다. 예를 들어 봇은 새 주문 흐름을 호출한 다음, 제품 검색 흐름을 호출합니다. 그런 다음, 사용자는 제품을 선택하고, 제품 검색 흐름 완료를 확인한 다음, 주문을 완료합니다.

그러나 대화는 이러한 선형 논리 경로를 드물게 따릅니다. 사용자는 "스택"에서 통신하지 않고 대신 마음이 자주 바뀌는 경향이 있습니다. 다음 예제를 살펴보세요.

![사용자가 예기치 않은 것을 말하는 경우](./media/concept-conversation/interruption.png)

봇이 흐름의 스택을 논리적으로 생성했을 수 있지만, 사용자는 완전히 다른 어떤 작업을 수행하거나 현재 항목과 관련이 없을 수 있는 질문을 하려고 할 수 있습니다. 예제에서 사용자는 흐름이 예상하는 예/아니요 응답을 제공하는 대신 질문을 합니다. 흐름은 어떻게 응답해야 할까요?

* 사용자가 먼저 질문에 답변할 것을 요구합니다.
* 사용자가 이전에 수행한 모든 작업을 무시하고, 전체 흐름 스택을 다시 설정하고, 사용자의 질문에 답변함으로써 처음부터 시작합니다.
* 사용자의 질문에 답변한 다음, 해당 예/아니요 질문으로 돌아가서 다시 시작해 봅니다.

최상의 솔루션은 시나리오의 세부 정보와 사용자가 봇의 응답을 합리적으로 예상하는 방법에 따라 결정되므로 이 질문에 대한 올바른 답변은 없습니다. 자세한 내용은 [사용자 중단 처리](bot-builder-howto-handle-user-interrupt.md)를 참조하세요.

> [!TIP]
> Node.Js용 Bot Builder SDK를 사용하는 경우 [대화 상자](bot-builder-dialog-manage-conversation-flow.md)를 사용하여 대화 흐름을 관리합니다.

## <a name="conversation-lifetime"></a>대화 수명

<!-- Note: these activities are dependent on whether the channel actually sends them. Also, we should add links --> 봇은 대화에 추가되고, 다른 멤버가 대화에 추가되거나 제거되고 또는 대화 메타데이터가 변경될 때마다 _대화 업데이트_ 작업을 받습니다.
사용자에게 인사하거나 자신을 소개하여 봇이 대화 업데이트 작업에 반응하도록 할 수 있습니다.

봇은 사용자가 대화를 종료했음을 나타내기 위해 _대화 종료_ 작업을 받습니다. 봇은 사용자에게 대화가 끝나감을 나타내기 위해 _대화 종료_ 작업을 보낼 수 있습니다. 대화에 대한 정보를 저장하는 경우 대화가 종료할 때 해당 정보를 삭제할 수 있습니다.

<!--  Types of conversations

Your bot can support multi-turn interactions where it prompts users for multiple peices of information. It can be focused on a very specific task or support multiple types of tasks. 
The Bot Builder SDK has some built-in support for Language Understatnding (LUIS) and QnA Maker for adding natural language "question and answer" features to your bot.

<!--TODO: Add with links when these topics are available:
[Conversation flow] and other design articles.
[Using recognizers] [Using state and storage] and other how tos.
-->
## <a name="conversations-channels-and-users"></a>대화, 채널 및 사용자

대화는 특정 사용자와의 _직접_ 대화 또는 여러 사용자와의 _그룹_ 대화일 수 있습니다.
봇은 사용자에게서 작업을 받고, 사용자에게 작업을 보내 채널에서 사용자와 통신합니다.

- 각 사용자는 채널별로 고유한 ID를 갖습니다.
- 각 대화는 채널별로 고유한 ID를 갖습니다.
- 채널은 대화를 시작할 때 대화 ID를 설정합니다.
- 봇은 대화를 시작할 수 없습니다. 그러나 대화 ID를 가지면 해당 대화를 다시 시작할 수 있습니다.
- 일부 채널은 그룹 대화를 지원하지 않습니다.

## <a name="next-steps"></a>다음 단계

위의 강조 표시된 것의 일부와 같은 복잡한 대화의 경우 턴보다 길게 정보를 유지할 수 있어야 합니다. 다음으로 상태 및 저장소를 살펴봅시다.

> [!div class="nextstepaction"]
> [상태 및 저장소](bot-builder-storage-concept.md)

<!-- In addition, your bot can send activities back to the user, either _proactively_, in response to internal logic, or _reactively_, in response to an activity from the user or channel.-->
<!--TODO: Link to messaging how tos.-->

<!--  TODO: Change to next steps, one for each of LUIS and State
## See also

- Activities
- Adapter
- Context
- Proactive messaging
- State
-->

[QnAMaker]:(bot-builder-luis-and-qna.md#using-qna-maker)

<!-- TODO: Update when the Dispatch concept is pushed -->
[Dispatch]:(bot-builder-concept-luis.md)
