---
title: Bot Connector Service를 사용하여 봇 만들기 | Microsoft Docs
description: Bot Connector Service를 사용하여 봇 만들기
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 3baf5bde772e67084a6046a8d2a8e7d631b245f6
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301739"
---
# <a name="create-a-bot-with-the-bot-connector-service"></a>Bot Connector Service를 사용하여 봇 만들기
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Bot Service](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Bot Connector Service에서는 봇에서 HTTPS를 통해 업계 표준 REST 및 JSON을 사용하여 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 포털</a>에 구성된 채널과 메시지를 교환할 수 있습니다. 이 자습서에서는 Bot Framework에서 액세스 토큰을 획득하고 Bot Connector service를 사용하여 사용자와 메시지를 교환하는 과정을 안내합니다.

## <a id="get-token"></a> 액세스 토큰 가져오기

> [!IMPORTANT]
> 아직 수행하지 않은 경우 Bot Framework에 [봇을 등록](../bot-service-quickstart-registration.md)하여 해당 앱 ID 및 암호를 가져와야 합니다. 액세스 토큰을 가져오려면 봇의 AppID 및 암호가 필요합니다.

Bot Connector Service와 통신하려면 다음 형식을 사용하여 각 API 요청의 `Authorization` 헤더에 액세스 토큰을 지정해야 합니다. 

```http
Authorization: Bearer ACCESS_TOKEN
```

API 요청을 실행하여 봇에 대한 액세스 토큰을 가져올 수 있습니다.

### <a name="request"></a>요청

Bot Connector Service에서 요청을 인증하는 데 사용되는 액세스 토큰을 요청하려면 **MICROSOFT-APP-ID** 및 **MICROSOFT-APP-PASSWORD**를 Bot Framework에 봇을 [등록](../bot-service-quickstart-registration.md)할 때 획득한 앱 ID 및 암호로 바꾸어 다음 요청을 실행합니다.

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="response"></a>response

요청이 성공하면 액세스 토큰 및 해당 만료에 대한 정보를 지정하는 HTTP 200 응답을 받게 됩니다. 

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

> [!TIP]
> Bot Connector Service의 인증에 대한 자세한 내용은 [인증](bot-framework-rest-connector-authentication.md)을 참조하세요.

## <a name="exchange-messages-with-the-user"></a>사용자와 메시지 교환

대화는 일련의 사용자와 봇 간에 교환되는 일련의 메시지입니다. 

### <a name="receive-a-message-from-the-user"></a>사용자로부터 메시지 수신

사용자가 메시지를 보낼 때 Bot Framework Connector는 봇 [등록](../bot-service-quickstart-registration.md) 시 지정한 끝점으로 요청을 POST합니다. 요청의 본문은 [Activity][Activity] 개체입니다. 다음 예제에서는 사용자가 봇에 간단한 메시지를 보낼 때 봇이 수신하는 요청 본문을 보여 줍니다. 

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

### <a name="reply-to-the-users-message"></a>사용자 메시지에 회신

봇의 끝점이 사용자가 보낸 메시지를 나타내는 `POST` 요청(즉, `type` = **메시지**)을 수신하면 해당 요청의 정보를 사용하여 응답에 대한 [Activity][Activity] 개체를 만듭니다.

1. **conversation** 속성을 사용자 메시지에 있는 **conversation** 속성의 내용으로 설정합니다.
2. **from** 속성을 사용자 메시지에 있는 **recipient** 속성의 내용으로 설정합니다.
3. **recipient** 속성을 사용자 메시지에 있는 **from** 속성의 내용으로 설정합니다.
4. **text** 및 **attachments** 속성을 적절히 설정합니다.

들어오는 요청의 `serviceUrl` 속성을 사용하여 봇이 해당 응답을 실행하는 데 사용해야 하는 [기본 URI를 식별](bot-framework-rest-connector-api-reference.md#base-uri)합니다. 

응답을 보내려면 다음 예제와 같이 [Activity][Activity] 개체를 `/v3/conversations/{conversationId}/activities/{activityId}`에 `POST`합니다. 이 요청의 본문은 사용할 수 있는 약속 시간을 선택하라는 메시지를 표시하는 [Activity][Activity] 개체입니다.

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have these times available:",
    "replyToId": "bf3cc9a2f5de..."
}
```

이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요. 

> [!IMPORTANT]
> 이 예제에 표시된 것처럼 전송하는 각 API 요청의 `Authorization` 헤더에는 단어 **Bearer**와 [Bot Framework에서 가져온](#get-token) 액세스 토큰이 차례로 포함되어야 합니다.

사용자가 단추를 클릭하여 사용 가능한 약속 시간을 선택할 수 있도록 하는 다른 메시지를 전송하려면 같은 끝점으로 다른 요청을 `POST`합니다.

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "attachmentLayout": "list",
    "attachments": [
      {
        "contentType": "application/vnd.microsoft.card.thumbnail",
        "content": {
          "buttons": [
            {
              "type": "imBack",
              "title": "10:30",
              "value": "10:30"
            },
            {
              "type": "imBack",
              "title": "11:30",
              "value": "11:30"
            },
            {
              "type": "openUrl",
              "title": "See more",
              "value": "http://www.contososalon.com/scheduling"
            }
          ]
        }
      }
    ],
    "replyToId": "bf3cc9a2f5de..."
}
```   

## <a name="next-steps"></a>다음 단계

이 자습서에서는 Bot Framework에서 액세스 토큰을 획득하고 Bot Connector Service를 사용하여 사용자와 메시지를 교환하는 과정을 진행했습니다. [Bot Framework Emulator](../bot-service-debug-emulator.md)를 사용하여 봇을 테스트하고 디버그할 수 있습니다. 봇을 다른 사용자와 공유하려면 하나 이상의 채널에서 실행되도록 [구성](../bot-service-manage-channels.md)해야 합니다.


[Activity]: bot-framework-rest-connector-api-reference.md#activity-object