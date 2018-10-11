---
redirect_url: /bot-framework/bot-builder-howto-v4-state
ms.openlocfilehash: e5da105e32ae748383d376f90afd9aebbf4c7aa5
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46708119"
---
<a name="--"></a><!--
---
제목: 상태 및 저장소 | Microsoft Docs 설명: 어떤 상태 관리자, 대화 상태 및 사용자 상태가 Bot Builder SDK 내에 있는지 설명합니다.
키워드: LUIS, 대화 상태, 사용자 상태, 저장소, 상태 관리 작성자: DeniseMak ms.작성자: v demak 관리자: kamrani ms.제목: article ms.prod: bot framework ms.날짜: 2018/02/15 monikerRange: 'azure-bot-service-4.0'
---

# <a name="state-and-storage"></a>상태 및 저장소
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

좋은 봇 디자인의 핵심은 대화의 컨텍스트를 추적하여 이전 질문에 대한 답변 같은 것을 기억하도록 하는 것입니다.
봇의 용도에 따라 상태의 트랙을 유지하거나 대화의 수명보다 오랫동안 정보를 저장해야 할 수 있습니다.
봇의 *상태*는 들어오는 메시지에 적절하게 응답하기 위해 기억하는 정보입니다. Bot Builder SDK는 사용자 또는 대화와 연결된 개체로 상태 데이터를 저장 및 검색하기 위한 클래스를 제공합니다.

* **대화 속성**은 봇이 사용자와 갖는 현재 대화의 트랙을 유지하도록 돕습니다. 봇에서 대화 항목 간의 단계 또는 전환의 시퀀스를 완료해야 하는 경우 대화 속성을 사용하여 시퀀스의 단계를 관리하거나 현재 항목을 추적할 수 있습니다. 대화 속성은 현재 대화의 상태를 반영하므로 일반적으로 봇에서 _대화 종료_ 작업을 받을 때 대화의 끝에서 이를 지웁니다.
* **사용자 속성**은 사용자의 이전 대화를 중단한 위치를 확인하거나 단순히 이름별로 돌아온 사용자에게 인사말을 제공하는 등의 다양한 용도로 사용할 수 있습니다. 사용자의 기본 설정을 저장하면 이 정보를 사용하여 다음에 채팅할 때 대화를 사용자 지정할 수 있습니다. 예를 들어 관심 있는 주제에 대한 뉴스 기사를 사용자에게 알리거나 약속이 가능해질 때 사용자에게 알릴 수 있습니다. 봇에서 _사용자 데이터 삭제_ 작업을 수신하는 경우 이를 지워야 합니다.

[저장소](bot-builder-howto-v4-storage.md)를 사용하여 영구 저장소에서 읽고 작성할 수 있습니다. 이를 통해 봇에서 공유 리소스 업데이트, RSVP 또는 응답 기록 또는 과거 날씨 데이터 읽기와 같은 작업을 수행할 수 있습니다. 동일한 방식으로 앱은 저장소를 사용하여 해당 목표를 달성하고, 봇은 사용자와의 대화 내에서 이를 수행할 수 있습니다.

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 

<!--
## Types of underlying storage

The SDK provides bot state manager middleware to persist conversation and user state. State can be accessed using the bot's context. This state manager can use Azure Table Storage, file storage, or memory storage as the underlying data storage. You can also create your own storage components for your bot.

Bots built using Azure Table Storage can be designed to be stateless and scalable across multiple compute nodes.

> [!NOTE] 
> File and memory storage won't scale across nodes.

## Writing directly to storage

You can also use the Bot Builder SDK to read and write data directly to storage, without using middleware or without using the bot context. This can be appropriate to data that your bot uses, that comes from a source outside your bot's conversation flow.

For example, let's say your bot allows the user to ask for the weather report, and your bot retrieves the weather report for a specified date, by reading it from an external database. The content of the weather database isn't dependent on user information or the conversation context, so you could just read it directly from storage instead of using the state manager.  See [How to write directly to storage](bot-builder-howto-v4-storage.md) for an example.

## Next steps

Next, lets get into how activities are processed, in depth, and how we respond to them.

> [!div class="nextstepaction"]
> [Activity Processing](bot-builder-concept-activity-processing.md)

## Additional resources

- [How to save state](bot-builder-howto-v4-state.md)
- [How to write directly to storage](bot-builder-howto-v4-storage.md)

-->
