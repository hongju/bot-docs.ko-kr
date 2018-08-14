---
title: 메시지에 미디어 추가 | Microsoft Docs
description: Bot Builder SDK를 사용하여 메시지에 미디어를 추가하는 방법에 대해 알아봅니다.
keywords: 미디어, 메시지, 이미지, 오디오, 비디오, 파일, MessageFactory, 서식 있는 메시지
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/03/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e1b25a3b5c090cbb13b4c27279745a81da64e6c4
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302578"
---
# <a name="add-media-to-messages"></a>메시지에 미디어 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

사용자와 봇 간의 메시지 교환에는 이미지, 비디오, 오디오 및 파일과 같은 미디어 첨부 파일이 포함될 수 있습니다. Bot Builder SDK에는 사용자에게 서식 있는 메시지를 보내는 작업을 간소화하도록 설계된 새 `Microsoft.Bot.Builder.MessageFactory` 클래스가 포함되어 있습니다.

## <a name="send-attachments"></a>첨부 파일 보내기

이미지 또는 비디오와 같은 사용자 콘텐츠를 보내려면 첨부 파일 또는 첨부 파일 목록을 메시지에 추가하면 됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` 개체의 `Attachments` 속성에는 메시지에 첨부된 미디어 첨부 파일과 서식 있는 카드를 나타내는 `Attachment` 개체의 배열이 포함되어 있습니다. 메시지에 미디어 첨부 파일을 추가하려면 `message` 작업에 대한 `Attachment` 개체를 만들고 `ContentType`, `ContentUrl` 및 `Name` 속성을 설정합니다. 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and add an attachment.
var activity = MessageFactory.Attachment(
    new Attachment { ContentUrl = "imageUrl.png", ContentType = "image/png" }
);

// Send the activity to the user.
await context.SendActivity(activity);
```

메시지 팩터리의 `Attachment` 메서드는 첨부 파일 목록을 다른 첨부 파일 위에 겹쳐서 보낼 수도 있습니다.

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and add an attachment.
var activity = MessageFactory.Attachment(new Attachment[]
{
    new Attachment { ContentUrl = "imageUrl1.png", ContentType = "image/png" },
    new Attachment { ContentUrl = "imageUrl2.png", ContentType = "image/png" },
    new Attachment { ContentUrl = "imageUrl3.png", ContentType = "image/png" }
});

// Send the activity to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

사용자에게 이미지 또는 비디오와 같은 콘텐츠의 한 조각을 보내려면 URL에 포함된 미디어를 보내면 됩니다.

```javascript
const {MessageFactory} = require('botbuilder');
let imageOrVideoMessage = MessageFactory.contentUrl('imageUrl.png', 'image/jpeg')

// Send the activity to the user.
await context.sendActivity(imageOrVideoMessage);
```

첨부 파일 목록을 보내려면 다른 <!-- TODO: Convert the hero cards to image attachments in this example. --> 위에 하나씩 쌓아 올립니다.

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory} = require('botbuilder');
const {CardFactory} = require('botbuilder');

let messageWithCarouselOfCards = MessageFactory.list([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

첨부 파일이 이미지, 오디오 또는 비디오인 경우 커넥터 서비스는 [채널](~/v4sdk/bot-builder-channeldata.md)이 대화 내의 해당 첨부 파일을 렌더링할 수 있도록 채널에 첨부 파일 데이터를 전달합니다. 첨부 파일이 파일인 경우 파일 URL은 대화 내 하이퍼링크로 렌더링됩니다.

## <a name="send-a-hero-card"></a>영웅 카드 보내기

단순 이미지 또는 비디오 첨부 파일 외에도 **영웅 카드**를 첨부할 수 있습니다. 이 카드를 사용하면 이미지 및 단추를 하나의 개체로 결합하여 사용자에게 보낼 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

영웅 카드와 단추로 메시지를 작성하려면 `HeroCard`를 메시지에 첨부하면 됩니다.

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a Hero card.
var activity = MessageFactory.Attachment(
    new HeroCard(
        title: "White T-Shirt",
        images: new CardImage[] { new CardImage(url: "imageUrl.png") },
        buttons: new CardAction[]
        {
            new CardAction(title: "buy", type: ActionTypes.ImBack, value: "buy")
        })
    .ToAttachment());

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

사용자에게 카드와 단추를 보내려면 메시지에 `heroCard`를 첨부하면 됩니다.

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory} = require('botbuilder');
const {CardFactory} = require('botbuilder');

const message = MessageFactory.attachment(
     CardFactory.heroCard(
        'White T-Shirt',
        ['https://example.com/whiteShirt.jpg'],
        ['buy']
    )
 );

await context.sendActivity(message);
```

---

<!--Lifted from the RESP API documentation-->

서식 있는 카드는 제목, 설명, 링크 및 이미지로 구성됩니다. 메시지에는 목록 형식 또는 회전식 형식으로 표시되는 여러 서식 있는 카드가 포함될 수 있습니다. Bot Builder SDK는 현재 다양한 범위의 서식 있는 카드를 지원합니다. 지원되는 서식 있는 카드 및 채널 목록을 보려면 [UX 요소 설계](../bot-service-design-user-experience.md)를 참조하세요.

> [!TIP]
> 채널이 지원하는 서식 있는 카드의 형식을 확인하고 채널이 서식 있는 카드를 렌더링하는 방법을 알아보려면 [채널 검사기](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-inspector?view=azure-bot-service-4.0)를 참조하세요. 카드 콘텐츠에 대한 제한 사항 정보는 채널 설명서를 참조하세요(예: 최대 단추 수 또는 최대 제목 길이).

## <a name="process-events-within-rich-cards"></a>서식 있는 카드 내에서 이벤트 처리

서식 있는 카드 내에서 이벤트를 처리하려면 _카드 작업_ 개체를 사용하여 사용자가 단추를 클릭하거나 카드의 섹션을 탭할 때 수행되어야 하는 작업을 지정합니다.

올바르게 작동하려면 카드의 클릭 가능한 각 항목에 작업 유형을 할당합니다. 이 표에서는 카드 작업 개체의 유형 속성에 대한 유효한 값을 나열하고 각 유형에 대한 값 속성의 예상 콘텐츠를 설명합니다.

| type | 값 |
| :---- | :---- |
| openUrl | 기본 제공 브라우저에서 열리는 URL입니다. URL을 열어 탭 또는 클릭에 응답합니다. |
| imBack | 단추를 클릭하거나 카드를 탭한 사용자에서 봇으로 전송할 메시지의 텍스트. 이 메시지(사용자에서 봇으로)는 대화를 호스트하고 있는 클라이언트 응용 프로그램을 통해 모든 대화 참가자에게 표시됩니다. |
| postBack | 단추를 클릭하거나 카드를 탭한 사용자에서 봇으로 전송할 메시지의 텍스트. 일부 클라이언트 응용 프로그램은 이 텍스트를 메시지 피드로 표시할 수 있습니다. 메시지 피드에서는 텍스트가 모든 대화 참가자에게 표시됩니다. |
| 다음을 호출합니다. | 이 형식의 전화 통화 대상: `tel:123123123123` 통화를 시작하여 탭 또는 클릭에 응답합니다.|
| playAudio | 재생되는 오디오의 URL입니다. 오디오를 재생하여 탭 또는 클릭에 응답합니다. |
| playVideo | 재생되는 비디오의 URL입니다. 비디오를 재생하여 탭 또는 클릭에 응답합니다. |
| showImage | 표시되는 이미지의 URL입니다. 이미지를 표시하여 탭 또는 클릭에 응답합니다. |
| downloadFile | 다운로드되는 파일의 URL입니다.  파일을 다운로드하여 탭 또는 클릭에 응답합니다. |
| signin | 시작되는 OAuth 흐름의 URL입니다. 로그인을 시작하여 탭 또는 클릭에 응답합니다. |

## <a name="hero-card-using-various-event-types"></a>다양한 이벤트 유형을 사용하는 영웅 카드

다음 코드에서는 다양한 서식이 있는 카드 이벤트를 사용하는 예제를 보여 줍니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a Hero card.
var activity = MessageFactory.Attachment(
    new HeroCard(
        title: "Holler Back Buttons",
        images: new CardImage[] { new CardImage(url: "imageUrl.png") },
        buttons: new CardAction[]
        {
            new CardAction(title: "Shout Out Loud", type: ActionTypes.imBack, value: "You can ALL hear me!"),
            new CardAction(title: "Much Quieter", type: ActionTypes.postBack, value: "Shh! My Bot friend hears me."),
            new CardAction(title: "Show me how to Holler", type: ActionTypes.openURL, value: $"https://en.wikipedia.org/wiki/{cardContent.Key}")
        })
    .ToAttachment());

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");
```

```javascript
const hero = MessageFactory.attachment(
    CardFactory.heroCard(
        'Holler Back Buttons',
        ['https://example.com/whiteShirt.jpg'],
        [{
            type: ActionTypes.ImBack,
            title: 'ImBack',
            value: 'You can ALL hear me! Shout Out Loud'
        },
        {
            type: ActionTypes.PostBack,
            title: 'PostBack',
            value: 'Shh! My Bot friend hears me. Much Quieter'
        },
        {
            type: ActionTypes.OpenUrl,
            title: 'OpenUrl',
            value: 'https://en.wikipedia.org/wiki/{cardContent.Key}'
        }]
    )
);

await context.sendActivity(hero);

```

---

## <a name="send-an-adaptive-card"></a>적응형 카드 보내기

적응형 카드를 첨부 파일로 보낼 수도 있습니다. 현재 일부 채널은 적응형 카드를 지원하지 않습니다. 적응형 카드 채널 지원에 대한 최신 정보는 <a href="http://adaptivecards.io/visualizer/">적응형 카드 시각화 도우미</a>를 참조하세요.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
적응형 카드를 사용하려면 `Microsoft.AdaptiveCards` NuGet 패키지를 추가해야 합니다.


> [!NOTE]
> 해당 채널이 적응형 카드를 지원하는지 여부를 확인하려면 봇이 사용할 채널을 통해 이 기능을 테스트해야 합니다.

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach an Adaptive Card.
var card = new AdaptiveCard();
card.Body.Add(new TextBlock()
{
    Text = "<title>",
    Size = TextSize.Large,
    Wrap = true,
    Weight = TextWeight.Bolder
});
card.Body.Add(new TextBlock() { Text = "<message text>", Wrap = true });
card.Body.Add(new TextInput()
{
    Id = "Title",
    Value = string.Empty,
    Style = TextInputStyle.Text,
    Placeholder = "Title",
    IsRequired = true,
    MaxLength = 50
});
card.Actions.Add(new SubmitAction() { Title = "Submit", DataJson = "{ Action:'Submit' }" });
card.Actions.Add(new SubmitAction() { Title = "Cancel", DataJson = "{ Action:'Cancel'}" });

var activity = MessageFactory.Attachment(new Attachment(AdaptiveCard.ContentType, content: card));

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {CardFactory} = require("botbuilder");

const message = CardFactory.adaptiveCard({
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "version": "1.0",
    "type": "AdaptiveCard",
    "speak": "Your flight is confirmed for you from San Francisco to Amsterdam on Friday, October 10 8:30 AM",
    "body": [
        {
            "type": "TextBlock",
            "text": "Passenger",
            "weight": "bolder",
            "isSubtle": false
        },
        {
            "type": "TextBlock",
            "text": "Sarah Hum",
            "separator": true
        },
        {
            "type": "TextBlock",
            "text": "1 Stop",
            "weight": "bolder",
            "spacing": "medium"
        },
        {
            "type": "TextBlock",
            "text": "Fri, October 10 8:30 AM",
            "weight": "bolder",
            "spacing": "none"
        },
        {
            "type": "ColumnSet",
            "separator": true,
            "columns": [
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "San Francisco",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "size": "extraLarge",
                            "color": "accent",
                            "text": "SFO",
                            "spacing": "none"
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": "auto",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": " "
                        },
                        {
                            "type": "Image",
                            "url": "http://messagecardplayground.azurewebsites.net/assets/airplane.png",
                            "size": "small",
                            "spacing": "none"
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "text": "Amsterdam",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "size": "extraLarge",
                            "color": "accent",
                            "text": "AMS",
                            "spacing": "none"
                        }
                    ]
                }
            ]
        },
        {
            "type": "ColumnSet",
            "spacing": "medium",
            "columns": [
                {
                    "type": "Column",
                    "width": "1",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "Total",
                            "size": "medium",
                            "isSubtle": true
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "text": "$1,032.54",
                            "size": "medium",
                            "weight": "bolder"
                        }
                    ]
                }
            ]
        }
    ]
});

// send adaptive card as attachment 
await context.sendActivity({ attachments: [message] })
```

---

## <a name="send-a-carousel-of-cards"></a>회전식 카드 보내기

메시지는 첨부 파일을 나란히 배치하고 사용자가 스크롤할 수 있도록 하는 회전식 레이아웃에 여러 첨부 파일을 포함할 수도 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a set of Hero cards.
var activity = MessageFactory.Carousel(
    new Attachment[]
    {
        new HeroCard(
            title: "title1",
            images: new CardImage[] { new CardImage(url: "imageUrl1.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button1", type: ActionTypes.ImBack, value: "item1")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title2",
            images: new CardImage[] { new CardImage(url: "imageUrl2.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button2", type: ActionTypes.ImBack, value: "item2")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title3",
            images: new CardImage[] { new CardImage(url: "imageUrl3.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button3", type: ActionTypes.ImBack, value: "item3")
            })
        .ToAttachment()
    });

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory} = require('botbuilder');
const {CardFactory} = require('botbuilder');

//  init message object
let messageWithCarouselOfCards = MessageFactory.carousel([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

## <a name="additional-resources"></a>추가 리소스

[채널 검사기를 사용하여 기능 미리 보기](../bot-service-channel-inspector.md)

---