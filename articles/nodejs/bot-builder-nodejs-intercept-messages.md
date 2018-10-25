---
title: 메시지 가로채기 | Microsoft Docs
description: Node.js용 Bot Builder SDK를 통해 정보 교환을 가로채고 처리하여 로그 및 기타 레코드를 만드는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c6f59e72eab4c3a442ba1e024b8fd3441cac6750
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996721"
---
# <a name="intercept-messages"></a>메시지 가로채기

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introduction to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="example"></a>예

다음 코드 샘플은 Node.js용 Bot Builder SDK의 **미들웨어** 개념을 사용하여 사용자와 봇 간에 교환되는 메시지를 가로채는 방법을 보여 줍니다. 

먼저, 들어오는 메시지에 대한 처리기(`botbuilder`)를 구성한 후 나가는 메시지에 대한 처리기(`send`)를 구성합니다.

```javascript
server.post('/api/messages', connector.listen());
var bot = new builder.UniversalBot(connector);
bot.use({
    botbuilder: function (session, next) {
        myMiddleware.logIncomingMessage(session, next);
    },
    send: function (event, next) {
        myMiddleware.logOutgoingMessage(event, next);
    }
})
```

그런 다음, 가로챈 각 메시지에 대해 수행할 작업을 정의하도록 각 처리기를 구현합니다.

```javascript
module.exports = {
    logIncomingMessage: function (session, next) {
        console.log(session.message.text);
        next();
    },
    logOutgoingMessage: function (event, next) {
        console.log(event.text);
        next();
    }
}
```

이제, 모든 인바운드 메시지(사용자-봇)가 `logIncomingMessage`를 트리거하고 모든 아웃바운드 메시지(봇-사용자)가 `logOutgoingMessage`를 트리거합니다.
이 예제에서 봇은 각 메시지에 대한 일부 정보만 간단히 인쇄하지만, 필요에 따라 `logIncomingMessage` 및 `logOutgoingMessage`를 업데이트하여 각 메시지에 대해 수행할 작업을 지정할 수 있습니다. 

## <a name="sample-code"></a>샘플 코드

Node.js용 Bot Builder SDK를 사용하여 메시지를 가로채는 방법을 보여 주는 전체 샘플은 GitHub의 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/capability-middlewareLogging" target="_blank">미들웨어 및 로깅 샘플</a>을 참조하세요.
