---
title: 메시지에 입력 힌트 추가 | Microsoft Docs
description: Bot Connector 서비스를 사용하여 메시지에 입력 힌트를 추가하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 66c6bc20013ff2de82e29af76e9c99898c8b13d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303218"
---
# <a name="add-input-hints-to-messages"></a>메시지에 입력 힌트 추가
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

메시지에 대한 입력 힌트를 지정하여 메시지를 클라이언트에 전달한 후 봇이 사용자 입력을 허용, 필요 또는 무시하는지 여부를 나타낼 수 있습니다. 많은 채널에서 이 기능을 사용하여 클라이언트가 사용자 입력 컨트롤 상태를 적절히 설정할 수 있습니다. 예를 들어 메시지의 입력 힌트가 봇이 사용자 입력을 무시하고 있는 것으로 표시하면 클라이언트는 마이크를 닫고 사용자가 입력을 제공하지 못하도록 입력 상자를 비활성화할 수 있습니다.

## <a name="accepting-input"></a>입력 허용

봇이 입력에 대해 수동적으로 준비되어 있지만 사용자의 응답을 기다리지 않고 있음을 나타내려면 메시지를 나타내는 [Activity][Activity] 개체 내에서 `inputHint` 속성을 **acceptingInput**으로 설정합니다. 많은 채널에서 이렇게 하면 클라이언트의 입력 상자가 활성화되고 마이크는 닫히지만 사용자는 계속 액세스할 수 있습니다. 예를 들어 사용자가 마이크 단추를 누르고 있으면 Cortana는 마이크를 열고 사용자의 입력을 허용합니다. 

다음은 메시지를 보내고 봇이 입력을 받아들이도록 지정하는 요청을 보여주는 예제입니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

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
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Here's a picture of the house I was telling you about.",
    "inputHint": "acceptingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="expecting-input"></a>입력 필요

봇이 사용자의 응답을 기다리고 있음을 나타내려면 메시지를 나타내는 [Activity][Activity] 개체 내에서 `inputHint` 속성을 **expectingInput**으로 설정합니다. 많은 채널에서 이렇게 하면 클라이언트의 입력 상자가 활성화되고 마이크가 열립니다. 

다음은 메시지를 보내고 봇이 입력을 기다리도록 지정하는 요청을 보여주는 예제입니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

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
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "What is your favorite color?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="ignoring-input"></a>입력 무시
 
봇이 사용자의 입력을 받을 준비가 되지 않다고 나타내려면 메시지를 나타내는 [Activity][Activity] 개체 내에서 `inputHint` 속성을 **ignoringInput**으로 설정합니다. 많은 채널에서 이렇게 하면 클라이언트의 입력 상자가 비활성화되고 마이크가 닫힙니다. 

다음은 메시지를 보내고 봇이 입력을 무시하도록 지정하는 요청을 보여주는 예제입니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

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
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Please hold while I perform the calculation.",
    "inputHint": "ignoringInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="additional-resources"></a>추가 리소스

- [메시지 만들기](bot-framework-rest-connector-create-messages.md)
- [메시지 보내기 및 받기](bot-framework-rest-connector-send-and-receive-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object