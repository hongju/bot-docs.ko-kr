---
title: 메시지에 미디어 첨부 파일 추가 | Microsoft Docs
description: Bot Connector service를 사용하여 메시지에 미디어 첨부 파일을 추가하는 방법에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/25/2018
ms.openlocfilehash: 3fad5b66f5137cd4098087e1b01d1f2493800994
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032605"
---
# <a name="add-media-attachments-to-messages"></a>메시지에 미디어 첨부 파일 추가
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

일반적으로 봇 및 채널은 텍스트 문자열을 교환하지만, 일부 채널은 첨부 파일 교환도 지원하므로 이를 통해 봇에서 사용자에게 더 다양한 메시지를 보낼 수 있습니다. 예를 들어, 봇은 미디어 첨부 파일(예: 이미지, 비디오, 오디오, 파일) 및 [서식 있는 카드](bot-framework-rest-connector-add-rich-cards.md)를 보낼 수 있습니다. 이 문서에서는 Bot Connector 서비스를 사용하여 미디어 첨부 파일을 메시지에 추가하는 방법을 설명합니다.

> [!TIP]
> 채널에서 지원하는 첨부 파일의 형식과 수 및 채널에서 첨부 파일을 렌더링하는 방법을 확인하려면 [Channel Inspector][ChannelInspector]를 참조하세요.

## <a name="add-a-media-attachment"></a>미디어 첨부 파일 추가  

메시지에 미디어 첨부 파일을 추가하려면 [Attachment][Attachment] 개체를 만들고 `name` 속성을 설정한 후 미디어 파일의 URL에 `contentUrl` 속성을 설정한 후 `contentType` 속성을 적절한 미디어 형식(예: **image/png**, **audio/wav**, **video/mp4**)으로 설정합니다. 그런 후 메시지를 나타내는 [Activity][Activity] 개체 내에서 `attachments` 배열 내에 [Attachment][Attachment] 개체를 지정합니다. 

다음 예제에서는 텍스트와 단일 이미지 첨부 파일을 포함하는 메시지를 보내는 요청을 보여 줍니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "https://aka.ms/DuckOnARock",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

이미지의 인라인 이진 파일을 지원하는 채널의 경우 `Attachment`의 `contentUrl` 속성을 이미지의 base64 이진으로 설정할 수 있습니다(예: **data:image/png;base64,iVBORw0KGgo…**). 채널은 메시지의 텍스트 문자열 옆에 이미지 또는 있는 이미지의 URL을 표시합니다.

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "data:image/png;base64,iVBORw0KGgo…",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

이미지 파일에 대해 위에서 설명한 것과 동일한 프로세스를 사용하여 비디오 파일 또는 오디오 파일을 메시지에 첨부할 수 있습니다. 채널에 따라, 비디오 및 오디오가 인라인으로 재생되거나 링크로 표시될 수 있습니다.

> [!NOTE] 
> 또한 봇에서 미디어 첨부 파일을 포함하는 메시지도 받을 수 있습니다.
> 예를 들어, 채널에서 사용자가 분석할 사진 또는 저장할 문서를 업로드하도록 허용할 경우 봇이 수신하는 메시지에 첨부 파일이 포함될 수 있습니다.

## <a name="add-an-audiocard-attachment"></a>AudioCard 첨부 파일 추가

[AudioCard](bot-framework-rest-connector-api-reference.md#audiocard-object) 또는 [VideoCard](bot-framework-rest-connector-api-reference.md#videocard-object) 첨부 파일을 추가하는 것은 미디어 첨부 파일을 추가하는 것과 같습니다. 예를 들어, 다음 JSON은 미디어 첨부 파일에 오디오 카드를 추가하는 방법을 보여 줍니다.

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
    "attachments": [
    {
      "contentType": "application/vnd.microsoft.card.audio",
      "content": {
        "title": "Allegro in C Major",
        "subtitle": "Allegro Duet",
        "text": "No Image, No Buttons, Autoloop, Autostart, Sharable",
        "duration": "PT2M55S",
        "media": [
          {
            "url": "https://contoso.com/media/AllegrofromDuetinCMajor.mp3"
          }
        ],
        "shareable": true,
        "autoloop": true,
        "autostart": true,
        "value": {
            // Supplementary parameter for this card
        }
      }
    }],
    "replyToId": "5d5cdc723"
}
```

채널이 이 첨부 파일을 받으면 오디오 파일 재생이 시작됩니다. 예를 들어, 사용자가 **일시 중지** 단추를 클릭하여 오디오와 상호 작용하면 채널은 다음과 같은 JSON을 사용하여 봇에 콜백을 전송합니다.

```json
{
    ...
    "type": "event",
    "name": "media/pause",
    "value": {
        "url": // URL for media
        "cardValue": {
            // Supplementary parameter for this card
        }
    }
}
```

미디어 이벤트 이름 **media/pause**가 `activity.name` 필드에 표시됩니다. 모든 미디어 이벤트 이름 목록은 아래 표를 참조하세요.

| 행사 | 설명 |
| ---- | ---- |
| **media/next** | 클라이언트가 다음 미디어로 건너뜀 |
| **media/pause** | 클라이언트가 미디어 재생을 일시 중지함 |
| **media/play** | 클라이언트가 미디어 재생을 시작함 |
| **media/previous** | 클라이언트가 이전 미디어로 건너뜀 |
| **media/resume** | 클라이언트가 미디어 재생을 다시 시작함 |
| **media/stop** | 클라이언트가 미디어 재생을 중지함 |

## <a name="additional-resources"></a>추가 리소스

- [메시지 만들기](bot-framework-rest-connector-create-messages.md)
- [메시지 보내기 및 받기](bot-framework-rest-connector-send-and-receive-messages.md)
- [메시지에 서식 있는 카드 추가](bot-framework-rest-connector-add-rich-cards.md)
- [Bot Framework 카드 스키마](https://aka.ms/botSpecs-cardSchema)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object
