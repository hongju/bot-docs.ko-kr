---
title: 메시지에 음성 추가 | Microsoft Docs
description: Bot Connector Service를 사용하여 메시지에 음성을 추가하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 91385fa3e8ae1410679ca5274e40db7fe38bafea
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304219"
---
# <a name="add-speech-to-messages"></a>메시지에 음성 추가
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Cortana와 같은 음성 지원 채널을 위한 봇을 빌드하는 경우 봇에서 말하는 텍스트를 지정하는 메시지를 구성할 수 있습니다. 봇이 사용자 입력을 수락, 대기 또는 무시할지 여부를 나타내는 [입력 힌트](bot-framework-rest-connector-add-input-hints.md)를 지정하여 클라이언트의 마이크의 상태에 영향을 미칠 수 있습니다.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>봇에서 말하는 텍스트 지정

음성 지원 채널에서 봇에서 말하는 텍스트를 지정하려면 사용자의 메시지를 나타내는 [활동][Activity] 개체 내 `speak` 속성을 설정합니다. `speak` 속성을 일반 텍스트 문자열 또는 음성, 속도, 음량, 발음, 음의 높이 등과 같은 봇의 말하기에 대한 다양한 특성을 제어할 수 있는 XML 기반 표시 언어인 <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML(Speech Synthesis Markup Language)</a>로 형식이 지정된 문자열로 설정할 수 있습니다. 

다음 요청은 표시할 텍스트 및 말할 텍스트를 지정하는 메시지를 보내고 봇이 [사용자 입력을 대기](bot-framework-rest-connector-add-input-hints.md)함을 나타냅니다. 여기서는 “sure”이란 단어가 일반적인 강조의 의미로 말했음을 표시하기 위해 <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a> 양식을 사용하여 `speak` 속성을 지정합니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 발행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

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
    "text": "Are you sure that you want to cancel this transaction?",
    "speak": "Are you <emphasis level='moderate'>sure</emphasis> that you want to cancel this transaction?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="input-hints"></a>입력 힌트

음성 지원 채널에서 메시지를 전송하는 경우 봇이 사용자 입력을 수락, 대기 또는 무시할지 여부를 나타내는 입력 힌트를 포함하여 클라이언트의 마이크의 상태에 영향을 미칠 수 있습니다. 자세한 내용은 [메시지에 입력 힌트 추가](bot-framework-rest-connector-add-input-hints.md)를 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- [메시지 만들기](bot-framework-rest-connector-create-messages.md)
- [메시지 보내기 및 받기](bot-framework-rest-connector-send-and-receive-messages.md)
- [메시지에 입력 힌트 추가](bot-framework-rest-connector-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML(Speech Synthesis Markup Language)</a>

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
