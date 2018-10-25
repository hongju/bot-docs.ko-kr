---
title: 메시지 보내기 및 받기 | Microsoft Docs
description: Bot Connector 서비스를 사용하여 메시지를 보내고 받는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: d0b7b3250a62a995113bc9c7e087e2e62af0f413
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997070"
---
# <a name="send-and-receive-messages"></a>메시지 보내기 및 받기

Bot Connector 서비스를 사용하면 봇이 Skype, 메일, Slack 등의 여러 채널을 통해 통신할 수 있습니다. 봇에서 채널로, 그리고 채널에서 봇으로 [활동](bot-framework-rest-connector-activities.md)을 릴레이하여 봇과 사용자 간에 쉽게 통신할 수 있게 합니다. 모든 활동에는 메시지를 만든 사람, 메시지 컨텍스트 및 메시지 받는 사람에 대한 정보와 함께 메시지를 적절한 대상에게 라우팅하는 데 사용되는 정보가 포함되어 있습니다. 이 문서에서는 Bot Connector 서비스를 사용하여 채널에서 봇과 사용자 간에 **메시지** 활동을 교환하는 방법을 설명합니다. 

## <a id="create-reply"></a> 메시지에 회신

### <a name="create-a-reply"></a>회신 만들기 

사용자가 봇에 메시지를 보내면 봇은 **message** 형식의 [Activity][Activity] 개체로 메시지를 받습니다. 사용자 메시지에 대한 회신을 만들려면 새 [Activity][Activity] 개체를 만들고 다음 속성을 설정하여 시작합니다.

| 자산 | 값 |
|----|----|
| 대화 | 이 속성을 사용자 메시지의 `conversation` 속성 내용으로 설정합니다. |
| from | 이 속성을 사용자 메시지의 `recipient` 속성 내용으로 설정합니다. |
| locale | 지정된 경우 이 속성을 사용자 메시지의 `locale` 속성 내용으로 설정합니다. |
| recipient | 이 속성을 사용자 메시지의 `from` 속성 내용으로 설정합니다. |
| replyToId | 이 속성을 사용자 메시지의 `id` 속성 내용으로 설정합니다. |
| 형식 | 이 속성을 **message**로 설정합니다. |

사용자에게 전달하려는 정보를 지정하는 속성을 설정합니다. 예를 들어, `text` 속성을 설정하여 메시지에 표시할 텍스트를 지정하고, `speak` 속성을 설정하여 봇이 말할 텍스트를 지정하고, `attachments` 속성을 설정하여 메시지에 포함할 미디어 첨부 파일 또는 서식 있는 카드를 지정할 수 있습니다. 일반적으로 사용되는 메시지 속성에 대한 자세한 내용은 [메시지 만들기](bot-framework-rest-connector-create-messages.md)를 참조하세요.

### <a name="send-the-reply"></a>회신 보내기

들어오는 활동의 `serviceUrl` 속성을 사용하여 봇이 해당 응답을 실행하는 데 사용해야 하는 [기본 URI를 식별](bot-framework-rest-connector-api-reference.md#base-uri)합니다. 

회신을 보내려면 다음 요청을 실행합니다. 

```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

이 요청 URI에서 **{conversationId}** 를 (회신) Activity 내 `conversation` 개체의 `id` 속성 값으로 바꾸고, **{activityId}** 를 (회신) Activity 내 `replyToId` 속성 값으로 바꿉니다. 요청 본문을, 회신을 나타내기 위해 만든 [Activity][Activity] 개체로 설정합니다.

다음 예제에서는 사용자 메시지에 대한 단순한 텍스트 전용 회신을 보내는 요청을 보여 줍니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723 
Authorization: Bearer ACCESS_TOKEN 
Content-Type: application/json 
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "Pepper's News Feed"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "Convo1"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "SteveW"
    },
    "text": "My bot's reply",
    "replyToId": "5d5cdc723"
}
```

## <a id="send-message"></a> (비회신) 메시지 보내기

봇이 보내는 대부분의 메시지는 사용자로부터 받은 메시지에 대한 회신입니다. 그러나 봇이 사용자 메시지에 대한 직접 회신이 아닌 메시지를 대화에 보내야 하는 경우도 있을 수 있습니다. 예를 들어, 봇이 대화의 새 주제를 시작하거나 대화를 끝낼 때 인사 메시지를 보내야 할 수 있습니다. 

사용자 메시지에 대한 직접 회신이 아닌 메시지를 대화에 보내려면 다음 요청을 실행합니다. 

```http
POST /v3/conversations/{conversationId}/activities
```

이 요청 URI에서 **{conversationId}** 를 대화의 ID로 바꿉니다. 
    
요청 본문을, 회신을 나타내기 위해 만든 [Activity][Activity] 개체로 설정합니다.

> [!NOTE]
> Bot Framework에는 봇이 보낼 수 있는 메시지 수에 대한 제한이 없습니다. 그러나 대부분의 채널은 제한 한도를 적용하여 봇이 짧은 기간에 다수의 메시지를 보낼 수 없도록 제한합니다. 또한 봇이 여러 메시지를 빠르게 연속적으로 보내는 경우 채널이 메시지를 올바른 순서로 렌더링하지 못할 수도 있습니다.

## <a name="start-a-conversation"></a>대화 시작

봇이 한 명 이상의 사용자와 대화를 시작해야 하는 경우도 있을 수 있습니다. 채널에서 사용자와 대화를 시작하려면 봇이 해당 계정 정보 및 채널의 사용자 계정 정보를 알고 있어야 합니다. 

> [!TIP]
> 나중에 봇이 사용자와 대화를 시작해야 할 수 있는 경우 사용자 계정 정보, 기타 관련 정보(예: 사용자 기본 설정, 로캘) 및 서비스 URL(대화 시작 요청에서 기본 URI로 사용)을 캐시합니다. 

대화를 시작하려면 다음 요청을 실행합니다. 

```http
POST /v3/conversations
```

요청 본문을, 봇의 계정 정보 및 대화에 포함하려는 사용자의 계정 정보를 지정하는 [Conversation][Conversation] 개체로 설정합니다.

> [!NOTE]
> 일부 채널은 그룹 대화를 지원하지 않습니다. 채널 설명서를 참조하여 채널이 그룹 대화를 지원하는지 확인하고, 채널이 대화에 허용하는 최대 참가자 수를 파악합니다.

다음 예제에서는 대화를 시작하는 요청을 보여 줍니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

```http
POST https://smba.trafficmanager.net/apis/v3/conversations 
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "bot": {
        "id": "12345678",
        "name": "bot's name"
    },
    "isGroup": false,
    "members": [
        {
            "id": "1234abcd",
            "name": "recipient's name"
        }
    ],
    "topicName": "News Alert"
}
```

지정한 사용자와 대화가 설정되면 대화를 식별하는 ID가 응답에 포함됩니다. 

```json
{
    "id": "abcd1234"
}
```

그러면 봇이 이 대화 ID를 사용하여 대화 내에서 사용자에게 [메시지를 보낼](#send-message) 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

- [작업 개요](bot-framework-rest-connector-activities.md)
- [메시지 만들기](bot-framework-rest-connector-create-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ConversationAccount]: bot-framework-rest-connector-api-reference.md#conversationaccount-object
[Conversation]: bot-framework-rest-connector-api-reference.md#conversation-object

