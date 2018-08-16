---
title: 상태 저장 및 데이터에 액세스 | Microsoft Docs
description: 어떤 상태 관리자, 대화 상태 및 사용자 상태가 Bot Builder SDK 내에 있는지 설명합니다.
keywords: LUIS, 대화 상태, 사용자 상태, 저장소, 상태 관리
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 02/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 56814ab12a85d18e52b0d5ec83fd81682f3b9f60
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/19/2018
ms.locfileid: "39304626"
---
# <a name="save-state-and-access-data"></a>상태 저장 및 데이터에 액세스
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

좋은 봇 디자인의 핵심은 대화의 컨텍스트를 추적하여 이전 질문에 대한 답변 같은 것을 기억하도록 하는 것입니다.
봇의 용도에 따라 상태의 트랙을 유지하거나 대화의 수명보다 오랫동안 정보를 저장해야 할 수 있습니다.
봇의 *상태*는 들어오는 메시지에 적절하게 응답하기 위해 기억하는 정보입니다. Bot Builder SDK는 사용자 또는 대화와 연결된 개체로 상태 데이터를 저장 및 검색하기 위한 클래스를 제공합니다.

* **대화 속성**은 봇이 사용자와 갖는 현재 대화의 트랙을 유지하도록 돕습니다. 봇에서 대화 항목 간의 단계 또는 전환의 시퀀스를 완료해야 하는 경우 대화 속성을 사용하여 시퀀스의 단계를 관리하거나 현재 항목을 추적할 수 있습니다. 대화 속성은 현재 대화의 상태를 반영하므로 일반적으로 봇에서 _대화 종료_ 작업을 받을 때 세션의 끝에서 이를 지웁니다.
* **사용자 속성**은 사용자의 이전 대화를 중단한 위치를 확인하거나 단순히 이름별로 돌아온 사용자에게 인사말을 제공하는 등의 다양한 용도로 사용할 수 있습니다. 사용자의 기본 설정을 저장하면 이 정보를 사용하여 다음에 채팅할 때 대화를 사용자 지정할 수 있습니다. 예를 들어 관심 있는 주제에 대한 뉴스 기사를 사용자에게 알리거나 약속이 가능해질 때 사용자에게 알릴 수 있습니다. 봇에서 _사용자 데이터 삭제_ 작업을 수신하는 경우 이를 지워야 합니다.

[저장소](bot-builder-howto-v4-storage.md)를 사용하여 영구 저장소에서 읽고 작성할 수 있습니다. 이를 통해 봇에서 공유 리소스 업데이트, RSVP 또는 응답 기록 또는 과거 날씨 데이터 읽기와 같은 작업을 수행할 수 있습니다. 동일한 방식으로 앱은 저장소를 사용하여 해당 목표를 달성하고, 봇은 사용자와의 대화 내에서 이를 수행할 수 있습니다.

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 


## <a name="types-of-underlying-storage"></a>기본 저장소 유형

SDK는 대화 및 사용자 상태를 유지하도록 봇 상태 관리자 미들웨어를 제공합니다. 봇의 컨텍스트를 사용하여 상태에 액세스할 수 있습니다. 이 상태 관리자는 기본 데이터 저장소로 Azure Table Storage, 파일 저장소 또는 메모리 저장소를 사용할 수 있습니다. 봇에 대한 사용자 고유의 저장소 구성 요소를 만들 수도 있습니다.

Azure Table Storage를 사용하여 빌드된 봇은 여러 계산 노드에서 상태 비저장이고 확장 가능하도록 디자인될 수 있습니다.

> [!NOTE] 
> 파일 및 메모리 저장소는 노드에서 확장하지 않습니다.

## <a name="writing-directly-to-storage"></a>저장소에 직접 작성

미들웨어를 사용하지 않거나 봇 컨텍스트를 사용하지 않고 Bot Builder SDK를 사용하여 저장소에서 직접 데이터를 읽고 쓸 수도 있습니다. 봇의 대화 흐름 외부 소스에서 제공되는 봇에서 사용하는 데이터에 적합할 수 있습니다.

예를 들어 봇을 통해 사용자는 일기 예보를 요청하고, 봇은 외부 데이터베이스에서 읽어 지정된 날짜에 대한 일기 예보를 검색한다고 가정해 봅니다. 날씨 데이터베이스의 콘텐츠는 사용자 정보 또는 대화 컨텍스트에 따라 달라지지 않으므로 상태 관리자를 사용하는 대신 저장소에서 직접 읽을 수 있습니다.  예제는 [저장소에 직접 작성하는 방법](bot-builder-howto-v4-storage.md)을 참조하세요.

## <a name="next-steps"></a>다음 단계

다음으로 작업이 처리되는 방법 및 응답하는 방법에 대해 자세히 살펴보겠습니다.

> [!div class="nextstepaction"]
> [작업 처리](bot-builder-concept-activity-processing.md)

## <a name="additional-resources"></a>추가 리소스

- [상태를 저장하는 방법](bot-builder-howto-v4-state.md)
- [저장소에 직접 작성하는 방법](bot-builder-howto-v4-storage.md)
