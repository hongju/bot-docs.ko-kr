---
title: 메시지에 서식 있는 카드 첨부 파일 추가 | Microsoft Docs
description: Node.js용 Bot Framework SDK를 사용하여 서식 있는 유용한 대화형 카드를 전송하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cba67dc4da5a0b505b4f91f9cbf7fbc0a47b8974
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404799"
---
# <a name="add-rich-card-attachments-to-messages"></a>메시지에 서식 있는 카드 첨부 파일 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]


> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Skype 및 Facebook과 같은 일부 채널은 사용자가 클릭하여 작업을 시작하는 대화형 단추를 포함하는 서식 있는 그래픽 카드를 사용자에게 전송하도록 지원합니다. 이 SDK는 카드를 만들어 보내는 데 사용할 수 있는 일부 메시지 및 카드 작성기 클래스를 제공합니다. Bot Framework Connector 서비스는 채널의 기본 스키마를 통해 이러한 카드를 렌더링하여 플랫폼 간 통신을 지원합니다. SMS와 같이 채널이 카드를 지원하지 않을 경우 Bot Framework는 사용자를 위해 합리적인 환경을 렌더링하기 위해 최선을 다합니다. 

## <a name="types-of-rich-cards"></a>서식 있는 카드 종류 
Bot Framework는 현재 8가지 형식의 서식 있는 카드를 지원합니다. 

| 카드 형식 | 설명 |
|------|------|
| <a href="/adaptive-cards/get-started/bots">적응형 카드</a> | 텍스트, 음성, 이미지, 단추 및 입력 필드의 조합을 포함할 수 있는 사용자 지정 가능한 카드입니다.  [채널별 지원](/adaptive-cards/get-started/bots#channel-status)을 참조하세요. |
| [애니메이션 카드][animationCard] | 애니메이션 GIF 또는 짧은 비디오를 재생할 수 있는 카드입니다. |
| [오디오 카드][audioCard] | 오디오 파일을 재생할 수 있는 카드입니다. |
| [영웅 카드][heroCard] | 일반적으로 하나의 큰 이미지, 하나 이상의 단추 및 텍스트를 포함하는 카드입니다. |
| [썸네일 카드][thumbnailCard] | 일반적으로 하나의 썸네일 이미지, 하나 이상의 단추 및 텍스트를 포함하는 카드입니다.|
| [수신 확인 카드][receiptCard] | 봇이 사용자에게 수신 확인을 제공할 수 있는 카드입니다. 일반적으로 수신 확인, 세금 및 합계 정보에 포함할 항목 목록과 기타 텍스트를 포함합니다. |
| [로그인 카드][signinCard] | 봇이 사용자 로그인을 요청하는 데 사용되는 카드입니다. 일반적으로 사용자가 로그인 프로세스를 시작하기 위해 클릭할 수 있는 하나 이상의 단추와 텍스트를 포함합니다. |
| [비디오 카드][videoCard] | 비디오를 재생할 수 있는 카드입니다. |

## <a name="send-a-carousel-of-hero-cards"></a>회전식 Hero 카드 보내기
다음 예제에서는 가상의 티셔츠 회사에 대한 봇을 보여 주고, “show shirts”라는 사용자 음성에 대한 반응으로 회전식 카드를 표시하는 방법을 보여 줍니다. 

```javascript
// Create your bot with a function to receive messages from the user
// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.send("Hi... We sell shirts. Say 'show shirts' to see our products.");
});

// Add dialog to return list of shirts available
bot.dialog('showShirts', function (session) {
    var msg = new builder.Message(session);
    msg.attachmentLayout(builder.AttachmentLayout.carousel)
    msg.attachments([
        new builder.HeroCard(session)
            .title("Classic White T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/whiteshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic white t-shirt", "Buy")
            ]),
        new builder.HeroCard(session)
            .title("Classic Gray T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/grayshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic gray t-shirt", "Buy")
            ])
    ]);
    session.send(msg).endDialog();
}).triggerAction({ matches: /^(show|list)/i });
```
이 예제에서는 [Message][Message] 클래스를 사용하여 회전식 보기를 빌드합니다.  
회전식 보기는 항목 구매를 유도하는 이미지, 텍스트 및 단일 단추가 포함된 [HeroCard][heroCard] 목록으로 구성됩니다.  
**Buy** 단추를 클릭하면 메시지 전송이 트리거되므로 단추 클릭을 가져오는 두 번째 다이얼로그를 추가해야 합니다. 

## <a name="handle-button-input"></a>단추 입력 처리

`buyButtonClick` 다이얼로그는 “buy” 또는 “add”로 시작하고 뒤에 단어 “shirt”가 있는 항목이 포함된 메시지가 수신될 때마다 트리거됩니다. 이 다이얼로그는 사용자가 요청한 색상 및 선택적 크기의 셔츠를 찾기 위한 몇 개의 정규식을 사용하여 시작됩니다.
이와 같이 개선된 유연성을 통해 단추 클릭과 “please add a large gray shirt to my cart”와 같은 사용자의 자연어 메시지가 둘 다 지원될 수 있습니다.
색상은 유효하지만 크기를 알 수 없는 경우 봇은 품목을 장바구니에 추가하기 전에 목록에서 크기를 선택하라는 메시지를 표시합니다. 봇이 필요한 모든 정보를 입수하게 되면 **session.userData**를 사용하여 유지되는 장바구니에 품목을 추가하고 사용자에게 확인 메시지를 보냅니다.

```javascript
// Add dialog to handle 'Buy' button click
bot.dialog('buyButtonClick', [
    function (session, args, next) {
        // Get color and optional size from users utterance
        var utterance = args.intent.matched[0];
        var color = /(white|gray)/i.exec(utterance);
        var size = /\b(Extra Large|Large|Medium|Small)\b/i.exec(utterance);
        if (color) {
            // Initialize cart item
            var item = session.dialogData.item = { 
                product: "classic " + color[0].toLowerCase() + " t-shirt",
                size: size ? size[0].toLowerCase() : null,
                price: 25.0,
                qty: 1
            };
            if (!item.size) {
                // Prompt for size
                builder.Prompts.choice(session, "What size would you like?", "Small|Medium|Large|Extra Large");
            } else {
                //Skip to next waterfall step
                next();
            }
        } else {
            // Invalid product
            session.send("I'm sorry... That product wasn't found.").endDialog();
        }   
    },
    function (session, results) {
        // Save size if prompted
        var item = session.dialogData.item;
        if (results.response) {
            item.size = results.response.entity.toLowerCase();
        }

        // Add to cart
        if (!session.userData.cart) {
            session.userData.cart = [];
        }
        session.userData.cart.push(item);

        // Send confirmation to users
        session.send("A '%(size)s %(product)s' has been added to your cart.", item).endDialog();
    }
]).triggerAction({ matches: /(buy|add)\s.*shirt/i });
```

<!-- 

> [!NOTE]
> When sending a message that contains images, keep in mind that some channels download images before displaying a message to the user.   
> As a result, a message containing an image followed immediately by a message without images may sometimes be flipped in the user's feed.
> For information on how to avoid messages being sent out of order, see [Message ordering][MessageOrder].  

-->
## <a name="add-a-message-delay-for-image-downloads"></a>이미지 다운로드를 위한 메시지 지연 추가
일부 채널은 사용자에게 메시지를 표시하기 전에 이미지를 다운로드하려고 하므로, 이미지와 이미지 없는 메시지를 차례대로 포함하는 메시지를 전송할 경우 때때로 사용자의 피드에서 메시지가 뒤집어져 표시됩니다. 이러한 가능성을 최소화하기 위해 이미지를 CDN(Content Delivery Network)에서 가져오도록 하고 지나치게 큰 이미지의 사용은 피할 수 있습니다. 극단적인 경우지만 이미지가 있는 메시지와 다음 메시지 사이에 1-2초의 지연을 삽입해야 할 수도 있습니다. 이 경우 **session.sendTyping()** 을 호출하여 지연 시작 전에 입력 표시기를 전송함으로써 사용자가 이러한 지연을 좀 더 자연스럽게 느끼도록 할 수 있습니다. 

<!-- 
To learn more about sending a typing indicator, see [How to send a typing indicator](bot-builder-nodejs-send-typing-indicator.md).
-->

Bot Framework는 봇의 여러 메시지가 뒤죽박죽 표시되는 경우를 방지하기 위해 일괄 처리를 구현합니다. <!-- Unfortunately, not all channels can guarantee this. --> 봇이 사용자에게 여러 응답을 보내면 개별 메시지가 자동으로 일괄 처리로 그룹화되어 사용자에게 세트로 전달되므로 메시지의 원래 순서를 유지할 수 있습니다. 이러한 자동 일괄 처리는 **session.send()** 를 호출할 때마다 기본적으로 250ms 동안 대기했다가 다음 **send()** 호출을 시작합니다.

메시지 일괄 처리 지연은 구성할 수 있습니다. SDK의 자동 일괄 처리 논리를 사용하지 않도록 설정하려면 기본 지연을 큰 수로 설정하고, 콜백으로 **sendBatch()** 를 수동으로 호출하여 일괄 처리가 전달된 후에 호출되도록 합니다.

## <a name="send-an-adaptive-card"></a>적응형 카드 보내기

적응형 카드는 텍스트, 음성, 이미지, 단추 및 입력 필드의 모든 조합을 포함할 수 있습니다. 적응형 카드는 카드 콘텐츠 및 형식에 대한 모든 권한을 제공하는, <a href="http://adaptivecards.io" target="_blank">적응형 카드</a>에 지정된 JSON 형식을 사용하여 만듭니다. 

Node.js를 사용하여 적응형 카드를 만들려면 <a href="http://adaptivecards.io" target="_blank">적응형 카드</a> 사이트 내의 정보를 이용하여 적응형 카드 스키마를 이해하고, 적응형 카드 요소를 살펴보고, 다양한 컴퍼지션 및 복잡성이 포함된 카드를 만드는 데 사용할 수 있는 JSON 샘플을 확인합니다. 또한 대화형 시각화 도우미를 사용하여 적응형 카드 페이로드를 디자인하고 카드 출력을 미리 볼 수 있습니다.

이 코드 예제에서는 일정 미리 알림을 위한 적응형 카드를 포함하는 메시지를 만드는 방법을 보여 줍니다. 

[!code-javascript[Add Adaptive Card attachment](../includes/code/node-send-card-buttons.js#addAdaptiveCardAttachment)]

결과 카드에는 세 개의 텍스트 블록, 입력 필드(선택 목록) 및 세 개의 단추가 포함됩니다.

![적응형 카드 일정 미리 알림](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>추가 리소스

* [채널 검사기를 사용하여 기능 미리 보기][inspector]
* <a href="http://adaptivecards.io" target="_blank">적응형 카드</a>
* [AnimationCard][animationCard]
* [AudioCard][audioCard]
* [HeroCard][heroCard]
* [ThumbnailCard][thumbnailCard]
* [ReceiptCard][receiptCard]
* [SigninCard][signinCard]
* [VideoCard][videoCard]
* [메시지][Message]
* [첨부 파일을 보내는 방법](bot-builder-nodejs-send-receive-attachments.md)

[MessageOrder]: bot-builder-nodejs-manage-conversation-flow.md#message-ordering
[Message]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[animationCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.animationcard.html 

[audioCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.audiocard.html 

[heroCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html

[thumbnailCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html 

[receiptCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html 

[signinCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.signincard.html 

[videoCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.videocard.html

[inspector]: ../bot-service-channel-inspector.md
