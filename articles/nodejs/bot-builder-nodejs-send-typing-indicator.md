---
title: 타이핑 표시기 보내기 | Microsoft Docs
description: Node.js용 Bot Framework SDK를 사용하여 봇이 요청을 처리하고 있는 것을 사용자에게 알리기 위해 "please wait"(기다려주십시오.) 표시기를 추가하는 방법에 대해 알아봅니다.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3852c0b25ea385301be11edd0a46ed5984510820
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224868"
---
# <a name="send-a-typing-indicator"></a>타이핑 표시기 보내기 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

사용자는 자신의 메시지에 제때 응답할 것을 기대합니다. 봇이 사용자의 요청을 들었다는 표시를 하지 않고 서버를 호출하거나 쿼리를 실행하는 것과 같이 장기간 실행되는 작업을 수행하는 경우, 사용자가 기다리지 못하고 추가로 메시지를 보내거나 봇에 문제가 있는 것으로 추측할 수 있습니다.
많은 채널이 메시지가 수신되어 처리되고 있다는 것을 사용자에게 보여주는 타이핑 표시 전송을 지원합니다.


## <a name="typing-indicator-example"></a>타이핑 표시기 예제

다음 예제는 [session.sendTyping()][SendTyping]을 사용하여 타이핑 표시를 보내는 방법을 보여줍니다.  Bot Framework Emulator를 사용하여 이 기능을 테스트할 수 있습니다.


```javascript

// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.sendTyping();
    setTimeout(function () {
        session.send("Hello there...");
    }, 3000);
});
```

이미지가 포함된 메시지가 잘못된 순서로 전송되는 것을 방지하기 위해 메시지 지연을 삽입하는 경우에도 타이핑 표시기가 유용합니다.

자세히 알아보려면 [리치(rich) 카드를 보내는 방법](bot-builder-nodejs-send-rich-cards.md)을 참조하세요.


## <a name="additional-resources"></a>추가 리소스

* [sendTyping][SendTyping]


[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
