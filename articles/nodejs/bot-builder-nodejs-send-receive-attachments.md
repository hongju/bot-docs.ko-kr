---
title: 첨부 파일 보내기 및 받기 | Microsoft Docs
description: Node.js용 Bot Framework SDK를 사용하여 첨부 파일을 포함하는 메시지를 보내고 받는 방법을 알아봅니다.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1911a5b0f8e8f8b53de6f661c0a939767df1efbb
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224698"
---
# <a name="send-and-receive-attachments"></a>첨부 파일 보내기 및 받기

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

사용자와 봇 간의 메시지 교환에는 이미지, 비디오, 오디오 및 파일과 같은 미디어 첨부 파일이 포함될 수 있습니다. 전송될 수 있는 첨부 파일의 형식은 채널에 따라 다르지만 기본 형식은 다음과 같습니다.

* **미디어 및 파일**: **contentType**을 [IAttachment 개체][IAttachment]의 MIME 형식으로 설정한 다음, **contentUrl**에서 파일에 대한 링크를 전달하여 이미지, 오디오 및 비디오와 같은 파일을 보낼 수 있습니다.
* **카드**: **contentType**을 원하는 카드의 형식으로 설정한 다음, 카드에 대한 JSON을 전달하여 다양한 세트의 시각적 카드 <!-- and custom keyboards -->를 보낼 수 있습니다. **HeroCard**와 같은 다양한 카드 작성기 클래스 중 하나를 사용하는 경우 첨부 파일은 자동으로 채워집니다. 이 예제는 [다양한 카드 보내기](bot-builder-nodejs-send-rich-cards.md)를 참조하세요.

## <a name="add-a-media-attachment"></a>미디어 첨부 파일 추가
메시지 개체는 [IMessage][IMessage]의 인스턴스일 것으로 예상되며 이미지와 같은 첨부 파일을 포함하려는 경우 사용자에게 개체로 메시지를 보내는 데 가장 유용합니다. [session.send()][SessionSend] 메서드를 사용하여 JSON 개체의 양식으로 메시지를 보냅니다. 

## <a name="example"></a>예

다음 예제에서는 사용자가 첨부 파일을 보냈는지 확인하고, 보낸 경우 첨부 파일에 포함된 이미지를 다시 에코하는지 확인합니다. 봇에게 이미지를 전송하여 Bot Framework 에뮬레이터로 테스트할 수 있습니다.

```javascript
// Create your bot with a function to receive messages from the user
var bot = new builder.UniversalBot(connector, function (session) {
    var msg = session.message;
    if (msg.attachments && msg.attachments.length > 0) {
     // Echo back attachment
     var attachment = msg.attachments[0];
        session.send({
            text: "You sent:",
            attachments: [
                {
                    contentType: attachment.contentType,
                    contentUrl: attachment.contentUrl,
                    name: attachment.name
                }
            ]
        });
    } else {
        // Echo back users text
        session.send("You said: %s", session.message.text);
    }
});
```
## <a name="additional-resources"></a>추가 리소스

* [채널 검사기를 사용하여 기능 미리 보기][inspector]
* [IMessage][IMessage]
* [다양한 카드 보내기][SendRichCard]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[SendRichCard]: bot-builder-nodejs-send-rich-cards.md
[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send
[IAttachment]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html
[inspector]: ../bot-service-channel-inspector.md
