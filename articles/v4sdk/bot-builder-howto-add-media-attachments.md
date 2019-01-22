---
title: 메시지에 미디어 추가 | Microsoft Docs
description: Bot Framework SDK를 사용하여 메시지에 미디어를 추가하는 방법에 대해 알아봅니다.
keywords: 미디어, 메시지, 이미지, 오디오, 비디오, 파일, MessageFactory, 서식 있는 카드, 메시지, 적응형 카드, 영웅 카드, 제안된 작업
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/17/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1ea9daeb35033e49232d64bfe98a223807dabf75
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317603"
---
# <a name="add-media-to-messages"></a>메시지에 미디어 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

사용자와 봇 간에 교환되는 메시지에는 이미지, 비디오, 오디오 및 파일과 같은 미디어 첨부 파일이 포함될 수 있습니다. Bot Framework SDK는 사용자에게 다양한 메시지를 보내는 작업을 지원합니다. 채널(Facebook, Skype, Slack 등)에서 지원하는 다양한 메시지의 유형을 결정하려면 채널 설명서에서 제한 사항에 대한 정보를 참조하세요. 사용 가능한 카드 목록은 [사용자 환경 디자인](../bot-service-design-user-experience.md)을 참조하세요. 

## <a name="send-attachments"></a>첨부 파일 보내기

이미지 또는 비디오와 같은 사용자 콘텐츠를 보내려면 첨부 파일 또는 첨부 파일 목록을 메시지에 추가하면 됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` 개체의 `Attachments` 속성에는 메시지에 첨부된 미디어 첨부 파일과 서식 있는 카드를 나타내는 `Attachment` 개체의 배열이 포함되어 있습니다. 메시지에 미디어 첨부 파일을 추가하려면 `message` 작업에 대한 `Attachment` 개체를 만들고 `ContentType`, `ContentUrl` 및 `Name` 속성을 설정합니다.
여기서 보여 주는 소스 코드는 [첨부 파일 처리](https://aka.ms/bot-attachments-sample-code) 샘플을 기반으로 합니다. 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create an attachment.
var attachment = new Attachment
    {
        ContentUrl = "imageUrl.png",
        ContentType = "image/png",
        Name = "imageName",
    };

// Add the attachment to our reply.
reply.Attachments = new List<Attachment>() { attachment };

// Send the activity to the user.
await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

여기서 보여 주는 소스 코드는 [JS 첨부 파일 처리](https://aka.ms/bot-attachments-sample-code-js) 샘플을 기반으로 합니다.
사용자에게 이미지 또는 비디오와 같은 콘텐츠의 한 조각을 보내려면 URL에 포함된 미디어를 보내면 됩니다.

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');

// Call function to get an attachment.
const reply = { type: ActivityTypes.Message };
reply.attachments = [this.getInternetAttachment()];
reply.text = 'This is an internet attachment.';
// Send the activity to the user.
await turnContext.sendActivity(reply);

/* function getInternetAttachment - Returns an attachment to be sent to the user from a HTTPS URL */
getInternetAttachment() {
        return {
            name: 'imageName.png',
            contentType: 'image/png',
            contentUrl: 'imageUrl.png'}
}
```

---

첨부 파일이 이미지, 오디오 또는 비디오인 경우 커넥터 서비스는 [채널](bot-builder-channeldata.md)이 대화 내의 해당 첨부 파일을 렌더링할 수 있도록 채널에 첨부 파일 데이터를 전달합니다. 첨부 파일이 파일인 경우 파일 URL은 대화 내 하이퍼링크로 렌더링됩니다.

## <a name="send-a-hero-card"></a>영웅 카드 보내기

단순 이미지 또는 비디오 첨부 파일 외에도 **영웅 카드**를 첨부할 수 있습니다. 이 카드를 사용하면 이미지 및 단추를 하나의 개체로 결합하여 사용자에게 보낼 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

영웅 카드와 단추가 있는 메시지를 작성하려면 `HeroCard`를 메시지에 첨부할 수 있습니다. 여기서 보여 주는 소스 코드는 [첨부 파일 처리](https://aka.ms/bot-attachments-sample-code) 샘플을 기반으로 합니다. 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create a HeroCard with options for the user to choose to interact with the bot.
var card = new HeroCard
{
    Text = "You can upload an image or select one of the following choices",
    Buttons = new List<CardAction>()
    {
        new CardAction(ActionTypes.ImBack, title: "1. Inline Attachment", value: "1"),
        new CardAction(ActionTypes.ImBack, title: "2. Internet Attachment", value: "2"),
        new CardAction(ActionTypes.ImBack, title: "3. Uploaded Attachment", value: "3"),
    },
};

// Add the card to our reply.
reply.Attachments = new List<Attachment>() { card.ToAttachment() };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

영웅 카드와 단추가 있는 메시지를 작성하려면 `HeroCard`를 메시지에 첨부할 수 있습니다. 여기서 보여 주는 소스 코드는 [JS 첨부 파일 처리](https://aka.ms/bot-attachments-sample-code-js) 샘플을 기반으로 합니다.

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');
// build buttons to display.
const buttons = [
            { type: ActionTypes.ImBack, title: '1. Inline Attachment', value: '1' },
            { type: ActionTypes.ImBack, title: '2. Internet Attachment', value: '2' },
            { type: ActionTypes.ImBack, title: '3. Uploaded Attachment', value: '3' }
];

// construct hero card.
const card = CardFactory.heroCard('', undefined,
buttons, { text: 'You can upload an image or select one of the following choices.' });

// add card to Activity.
const reply = { type: ActivityTypes.Message };
reply.attachments = [card];

// Send hero card to the user.
await turnContext.sendActivity(reply);
```

---

## <a name="process-events-within-rich-cards"></a>서식 있는 카드 내에서 이벤트 처리

서식 있는 카드 내에서 이벤트를 처리하려면 _카드 작업_ 개체를 사용하여 사용자가 단추를 클릭하거나 카드의 섹션을 탭할 때 수행되어야 하는 작업을 지정합니다. 각 카드 동작에는 _type_ 및 _value_가 있습니다.

올바르게 작동하려면 카드의 클릭 가능한 각 항목에 작업 유형을 할당합니다. 이 표에는 사용 가능한 동작 유형 및 관련 값 속성에 필요한 항목과 관련 설명이 나와 있습니다.

| type | 설명 | 값 |
| :---- | :---- | :---- |
| openUrl | 기본 제공 브라우저에서 URL이 열립니다. | 열려는 URL입니다. |
| imBack | 봇에 메시지를 보내고 채팅에 표시 가능한 응답을 게시합니다. | 보낼 메시지의 텍스트입니다. |
| postBack | 봇에 메시지를 보내고 채팅에 표시 가능한 응답을 게시하지 않을 수 있습니다. | 보낼 메시지의 텍스트입니다. |
| 다음을 호출합니다. | 전화 통화를 시작합니다. | 전화 통화 대상입니다(`tel:123123123123` 형식). |
| playAudio | 오디오를 재생합니다. | 재생할 오디오의 URL입니다. |
| playVideo | 비디오를 재생합니다. | 재생할 비디오의 URL입니다. |
| showImage | 이미지를 표시합니다. | 표시할 이미지의 URL입니다. |
| downloadFile | 파일을 다운로드합니다. | 다운로드할 파일의 URL입니다. |
| signin | OAuth 로그인 프로세스를 시작합니다. | 시작할 OAuth 흐름의 URL입니다. |

## <a name="hero-card-using-various-event-types"></a>다양한 이벤트 유형을 사용하는 영웅 카드

다음 코드에서는 다양한 서식이 있는 카드 이벤트를 사용하는 예제를 보여 줍니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

var card = new HeroCard
{
    Buttons = new List<CardAction>()
    {
        new CardAction(title: "Much Quieter", type: ActionTypes.PostBack, value: "Shh! My Bot friend hears me."),
        new CardAction(ActionTypes.OpenUrl, title: "Azure Bot Service", value: "https://azure.microsoft.com/en-us/services/bot-service/"),
    },
};

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");

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
적응형 카드 및 MessageFactory는 사용자와 통신하기 위해 텍스트, 이미지, 비디오, 오디오 및 파일이 포함된 서식 있는 메시지를 보내는 데 사용됩니다. 하지만 이들 사이에는 약간의 차이점이 있습니다. 

첫째, 일부 채널만 적응형 카드를 지원하며, 이를 지원하는 채널은 부분적으로 적응형 카드를 지원할 수 있습니다. 예를 들어, Facebook에서 적응형 카드를 보내면 텍스트와 이미지가 제대로 작동하는 동안 단추가 작동하지 않습니다. MessageFactory는 Bot Framework SDK 내의 도우미 클래스로, 생성 단계를 자동화할 수 있으며, 대부분의 채널에서 지원됩니다. 

둘째, 적응형 카드는 카드 양식의 메시지를 전달하며, 채널은 카드의 레이아웃을 결정합니다. MessageFactory가 제공하는 메시지 양식은 채널에 따라 다르며, 적응형 카드가 첨부 파일의 일부가 아니라면 카드 양식이 아니어도 됩니다. 

적응형 카드 채널 지원에 대한 최신 정보는 <a href="http://adaptivecards.io/visualizer/">적응형 카드 시각화 도우미</a>를 참조하세요.

적응형 카드를 사용하려면 `Microsoft.AdaptiveCards` NuGet 패키지를 추가해야 합니다. 


> [!NOTE]
> 해당 채널이 적응형 카드를 지원하는지 여부를 확인하려면 봇이 사용할 채널을 통해 이 기능을 테스트해야 합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

여기서 보여 주는 소스 코드는 [적응형 카드 사용](https://aka.ms/bot-adaptive-cards-sample-code) 샘플을 기반으로 합니다.

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Creates an attachment that contains an adaptive card
// filePath is the path to JSON file
private static Attachment CreateAdaptiveCardAttachment(string filePath)
{
    var adaptiveCardJson = File.ReadAllText(filePath);
    var adaptiveCardAttachment = new Attachment()
    {
        ContentType = "application/vnd.microsoft.card.adaptive",
        Content = JsonConvert.DeserializeObject(adaptiveCardJson),
    };
    return adaptiveCardAttachment;
}

// Create adaptive card and attach it to the message 
var cardAttachment = CreateAdaptiveCardAttachment(adaptiveCardJsonFilePath);
var reply = turnContext.Activity.CreateReply();
reply.Attachments = new List<Attachment>() { cardAttachment };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

여기서 보여 주는 소스 코드는 [JS 적응형 카드 사용](https://aka.ms/bot-adaptive-cards-js-sample-code) 샘플을 기반으로 합니다.

```javascript
const { BotFrameworkAdapter } = require('botbuilder');

// Import AdaptiveCard content.
const FlightItineraryCard = require('./resources/FlightItineraryCard.json');
const ImageGalleryCard = require('./resources/ImageGalleryCard.json');
const LargeWeatherCard = require('./resources/LargeWeatherCard.json');
const RestaurantCard = require('./resources/RestaurantCard.json');
const SolitaireCard = require('./resources/SolitaireCard.json');

// Create array of AdaptiveCard content, this will be used to send a random card to the user.
const CARDS = [
    FlightItineraryCard,
    ImageGalleryCard,
    LargeWeatherCard,
    RestaurantCard,
    SolitaireCard
];
// Select a random card to send.
const randomlySelectedCard = CARDS[Math.floor((Math.random() * CARDS.length - 1) + 1)];
// Send adaptive card.
await context.sendActivity({
      text: 'Here is an Adaptive Card:',
       attachments: [CardFactory.adaptiveCard(randomlySelectedCard)]
});
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
const {MessageFactory, CardFactory} = require('botbuilder');

//  init message object
let messageWithCarouselOfCards = MessageFactory.carousel([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>추가 리소스

카드 스키마에 대한 자세한 내용은 [Bot Framework 카드 스키마](https://aka.ms/botSpecs-cardSchema)를 참조하세요.

카드의 샘플 코드는 여기에서 확인할 수 있습니다. [C#](https://aka.ms/bot-cards-sample-code)/[JS](https://aka.ms/bot-cards-js-sample-code), 적응형 카드: [C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code), 첨부 파일: [C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-sample-code-js) 및 제안된 작업: [C#](https://aka.ms/SuggestedActionsCSharp)/[JS](https://aka.ms/SuggestedActionsJS).
추가 샘플은 [GitHub](https://aka.ms/bot-samples-readme)의 Bot Builder 샘플 리포지토리를 참조하세요.
