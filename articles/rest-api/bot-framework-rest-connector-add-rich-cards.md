---
title: 메시지에 서식 있는 카드 첨부 파일 추가 | Microsoft Docs
description: Bot Connector 서비스를 사용하여 메시지에 서식 있는 카드를 추가하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: e38bb7ca93c5fc4174d67d1c5ebb0655eef68653
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997923"
---
# <a name="add-rich-card-attachments-to-messages"></a>메시지에 서식 있는 카드 첨부 파일 추가
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

일반적으로 봇 및 채널은 텍스트 문자열을 교환하지만, 일부 채널은 첨부 파일 교환도 지원하므로 이를 통해 봇에서 사용자에게 더 다양한 메시지를 보낼 수 있습니다. 예를 들어, 봇은 서식 있는 카드 및 미디어 첨부 파일(예: 이미지, 비디오, 오디오, 파일)을 보낼 수 있습니다. 이 문서에서는 Bot Connector 서비스를 사용하여 서식 있는 카드 첨부 파일을 메시지에 추가하는 방법을 설명합니다.

> [!NOTE]
> 미디어 첨부 파일을 메시지에 추가하는 방법에 대한 자세한 내용은 [메시지에 미디어 첨부 파일 추가](bot-framework-rest-connector-add-media-attachments.md)를 참조하세요.

## <a id="types-of-cards"></a> 서식 있는 카드 종류

서식 있는 카드는 제목, 설명, 링크 및 이미지로 구성됩니다. 메시지에는 목록 형식 또는 회전식 형식으로 표시되는 여러 서식 있는 카드가 포함될 수 있습니다.
Bot Framework는 현재 8가지 형식의 서식 있는 카드를 지원합니다. 

| 카드 형식 | 설명 |
|----|----|
| <a href="/adaptive-cards/get-started/bots">AdaptiveCard</a> | 텍스트, 음성, 이미지, 단추 및 입력 필드의 조합을 포함할 수 있는 사용자 지정 가능한 카드입니다. [채널당 지원](/adaptive-cards/get-started/bots#channel-status)을 참조하세요.  |
| [AnimationCard][animationCard] | 애니메이션 GIF 또는 짧은 비디오를 재생할 수 있는 카드입니다. |
| [AudioCard][audioCard] | 오디오 파일을 재생할 수 있는 카드입니다. |
| [HeroCard][heroCard] | 일반적으로 하나의 큰 이미지, 하나 이상의 단추 및 텍스트를 포함하는 카드입니다. |
| [ThumbnailCard][thumbnailCard] | 일반적으로 하나의 미리 보기 이미지, 하나 이상의 단추 및 텍스트를 포함하는 카드입니다. |
| [ReceiptCard][receiptCard] | 봇이 사용자에게 확인 메일을 제공할 수 있는 카드입니다. 일반적으로 확인 메일, 세금 및 총 정보에 포함할 항목 목록과 기타 텍스트를 포함합니다. |
| [SignInCard][signinCard] | 봇이 사용자가 로그인하도록 요청할 수 있는 카드입니다. 일반적으로 사용자가 로그인 프로세스를 시작하기 위해 클릭할 수 있는 하나 이상의 단추와 텍스트를 포함합니다. |
| [VideoCard][videoCard] | 비디오를 재생할 수 있는 카드입니다. |

> [!TIP]
> 채널이 지원하는 서식 있는 카드의 형식을 확인하고 채널이 서식 있는 카드를 렌더링하는 방법을 알아보려면 [채널 검사기][ChannelInspector]를 참조하세요. 카드 콘텐츠에 대한 제한 사항 정보는 채널 문서를 참조하세요(예: 최대 단추 수 또는 최대 제목 길이).

## <a name="process-events-within-rich-cards"></a>서식 있는 카드 내에서 이벤트 처리

서식 있는 카드 내에서 이벤트를 처리하려면 [CardAction][CardAction] 개체를 사용하여 사용자가 단추를 클릭하거나 카드의 섹션을 탭할 때 수행되어야 하는 작업을 지정합니다. 각 [CardAction][CardAction] 개체는 다음 속성을 포함합니다.

| 자산 | type | 설명 | 
|----|----|----|
| 형식 | string | 작업 형식(아래 표에 지정된 값 중 하나) |
| title | string | 단추 제목 |
| 이미지 | string | 단추의 이미지 URL |
| 값 | string | 지정된 형식의 작업을 수행하는 데 필요한 값 |

> [!NOTE]
> 적응형 카드 내의 단추를 만들 때는 `CardAction` 개체를 사용하는 대신, <a href="http://adaptivecards.io" target="_blank">적응형 카드</a>에서 정의한 스키마를 사용합니다. 적응형 카드에 단추를 추가하는 방법을 보여 주는 예제는 [메시지에 적응형 카드 추가](#adaptive-card)를 참조하세요.

이 표에서는 [CardAction][CardAction] 개체의 `type` 속성에 대한 유효한 값을 나열하고 각 형식에 대한 `value` 속성의 예상 콘텐츠를 설명합니다.

| 형식 | 값 | 
|----|----|
| openUrl | 기본 제공 브라우저에서 열리는 URL |
| imBack | 단추를 클릭하거나 카드를 탭한 사용자에서 봇으로 전송할 메시지의 텍스트. 이 메시지(사용자에서 봇으로)는 대화를 호스트하고 있는 클라이언트 애플리케이션을 통해 모든 대화 참가자에게 표시됩니다. |
| postBack | 단추를 클릭하거나 카드를 탭한 사용자에서 봇으로 전송할 메시지의 텍스트. 일부 클라이언트 애플리케이션은 이 텍스트를 메시지 피드로 표시할 수 있습니다. 메시지 피드에서는 텍스트가 모든 대화 참가자에게 표시됩니다. |
| 다음을 호출합니다. | 이 형식의 전화 통화 대상: **tel:123123123123** |
| playAudio | 재생되는 오디오의 URL |
| playVideo | 재생되는 비디오의 URL |
| showImage | 표시되는 이미지의 URL |
| downloadFile | 다운로드되는 파일의 URL |
| signin | 시작되는 OAuth 흐름의 URL |

## <a name="add-a-hero-card-to-a-message"></a>메시지에 Hero Card 추가

서식 있는 카드 첨부 파일을 메시지에 추가하려면 먼저 메시지에 추가할 [카드 종류](#types-of-cards)에 해당하는 개체를 만듭니다. 그런 다음, [Attachment][Attachment] 개체를 만들고, 해당 `contentType` 속성을 카드의 미디어 형식으로 설정하고, 해당 `content` 속성을 카드를 표시하기 위해 만든 개체로 설정합니다. 메시지의 `attachments` 배열 내에서 [Attachment][Attachment] 개체를 지정합니다.

> [!TIP]
> 서식 있는 카드 첨부 파일을 포함하는 메시지는 일반적으로 `text`를 지정하지 않습니다.

일부 채널을 사용하면 메시지 내의 `attachments` 배열에 여러 서식 있는 카드를 추가할 수 있습니다. 사용자에게 여러 옵션을 제공하려는 시나리오에서는 이 기능이 유용할 수 있습니다. 예를 들어, 사용자가 봇을 통해 호텔 객실을 예약할 수 있는 경우 봇은 예약 가능한 객실 유형을 보여 주는 서식 있는 카드 목록을 사용자에게 제공합니다. 각 카드는 객실 유형에 해당하는 편의 시설의 그림 및 목록을 포함할 수 있고 사용자는 카드를 탭하거나 단추를 클릭하여 객실 유형을 선택할 수 있습니다.

> [!TIP]
> 여러 서식 있는 카드를 목록 형식으로 표시하려면 [Activity][Activity] 개체의 `attachmentLayout` 속성을 “list”로 설정합니다. 여러 서식 있는 카드를 회전식 형식으로 표시하려면 [Activity][Activity] 개체의 `attachmentLayout` 속성을 “carousel”로 설정합니다. 채널이 회전식 형식을 지원하지 않으면 `attachmentLayout` 속성을 “carousel”로 지정된 경우에도 서식 있는 카드가 목록 형식으로 표시됩니다.

다음 예제에서는 단일 Hero Card 첨부 파일을 포함하는 메시지를 보내는 요청을 보여 줍니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.hero",
            "content": {
                "title": "title goes here",
                "subtitle": "subtitle goes here",
                "text": "descriptive text goes here",
                "images": [
                    {
                        "url": "http://aka.ms/Fo983c",
                        "alt": "picture of a duck",
                        "tap": {
                            "type": "playAudio",
                            "value": "url to an audio track of a duck call goes here"
                        }
                    }
                ],
                "buttons": [
                    {
                        "type": "playAudio",
                        "title": "Duck Call",
                        "value": "url to an audio track of a duck call goes here"
                    },
                    {
                        "type": "openUrl",
                        "title": "Watch Video",
                        "image": "http://aka.ms/Fo983c",
                        "value": "url goes here of the duck in flight"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

## <a id="adaptive-card"></a> 메시지에 적응형 카드 추가

적응형 카드는 텍스트, 음성, 이미지, 단추 및 입력 필드의 모든 조합을 포함할 수 있습니다. 적응형 카드는 카드 콘텐츠 및 형식에 대한 모든 권한을 제공하는, <a href="http://adaptivecards.io" target="_blank">적응형 카드</a>에 지정된 JSON 형식을 사용하여 만듭니다. 

<a href="http://adaptivecards.io" target="_blank">적응형 카드</a> 사이트 내의 정보를 이용하여 적응형 카드 스키마를 이해하고, 적응형 카드 요소를 살펴보고, 다양한 컴퍼지션 및 복잡성이 포함된 카드를 만드는 데 사용할 수 있는 JSON 샘플을 확인합니다. 또한 대화형 시각화 도우미를 사용하여 적응형 카드 페이로드를 디자인하고 카드 출력을 미리 볼 수 있습니다.

다음 예제에서는 일정 미리 알림에 대한 단일 적응형 카드를 포함하는 메시지를 보내는 요청을 보여 줍니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.adaptive",
            "content": {
                "type": "AdaptiveCard",
                "body": [
                    {
                        "type": "TextBlock",
                        "text": "Adaptive Card design session",
                        "size": "large",
                        "weight": "bolder"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Conf Room 112/3377 (10)"
                    },
                    {
                        "type": "TextBlock",
                        "text": "12:30 PM - 1:30 PM"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Snooze for"
                    },
                    {
                        "type": "Input.ChoiceSet",
                        "id": "snooze",
                        "style": "compact",
                        "choices": [
                            {
                                "title": "5 minutes",
                                "value": "5",
                                "isSelected": true
                            },
                            {
                                "title": "15 minutes",
                                "value": "15"
                            },
                            {
                                "title": "30 minutes",
                                "value": "30"
                            }
                        ]
                    }
                ],
                "actions": [
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Snooze"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "I'll be late"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Dismiss"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

결과 카드에는 세 개의 텍스트 블록, 입력 필드(선택 목록) 및 세 개의 단추가 포함됩니다.

![적응형 카드 일정 미리 알림](../media/adaptive-card-reminder.png)


## <a name="additional-resources"></a>추가 리소스

- [메시지 만들기](bot-framework-rest-connector-create-messages.md)
- [메시지 보내기 및 받기](bot-framework-rest-connector-send-and-receive-messages.md)
- [메시지에 미디어 첨부 파일 추가](bot-framework-rest-connector-add-media-attachments.md)
- [채널 검사기][ChannelInspector]
- <a href="http://adaptivecards.io" target="_blank">적응형 카드</a>

[ChannelInspector]: ../bot-service-channel-inspector.md

[animationCard]: bot-framework-rest-connector-api-reference.md#animationcard-object
[audioCard]: bot-framework-rest-connector-api-reference.md#audiocard-object
[heroCard]: bot-framework-rest-connector-api-reference.md#herocard-object
[thumbnailCard]: bot-framework-rest-connector-api-reference.md#thumbnailcard-object
[receiptCard]: bot-framework-rest-connector-api-reference.md#receiptcard-object
[signinCard]: bot-framework-rest-connector-api-reference.md#signincard-object
[videoCard]: bot-framework-rest-connector-api-reference.md#videocard-object

[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object