---
title: 사전 대응 메시지 보내기 | Microsoft Docs
description: Node.js용 Bot Framework SDK를 사용하여 사전 대응 메시지를 통해 현재 대화 흐름을 중단하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8ca8043c5680a993fa27e2febb9740206691884c
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225578"
---
# <a name="send-proactive-messages"></a>사전 대응 메시지 보내기
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>자동 관리 메시지 형식

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>임시 사전 대응 메시지 보내기

다음 코드 샘플은 Node.js용 Bot Framework SDK를 사용하여 임시 사전 대응 메시지를 보내는 방법을 보여줍니다.

사용자에게 임시 메시지를 보낼 수 있으려면 봇이 먼저 현재 대화에서 사용자에 대한 정보를 수집하고 저장해야 합니다. 메시지의 **address** 속성에는 나중에 봇이 사용자에게 임시 메시지를 보내는 데 필요한 모든 정보가 포함됩니다. 

```javascript
bot.dialog('adhocDialog', function(session, args) {
    var savedAddress = session.message.address;

    // (Save this information somewhere that it can be accessed later, such as in a database, or session.userData)
    session.userData.savedAddress = savedAddress;

    var message = 'Hello user, good to meet you! I now know your address and can send you notifications in the future.';
    session.send(message);
})
```

> [!NOTE]
> 봇은 나중에 사용자 데이터에 액세스할 수 있다면 어떤 방식으로든 사용자 데이터를 저장할 수 있습니다.

봇은 사용자에 대한 정보를 수집한 후 언제든지 사용자에게 임시 사전 대응 메시지를 보낼 수 있습니다. 이를 위해 이전에 저장한 사용자 데이터를 검색하고, 메시지를 생성하고, 보내면 됩니다.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

var bot = new builder.UniversalBot(connector)
                .set('storage', inMemoryStorage); // Register in-memory storage 

function sendProactiveMessage(address) {
    var msg = new builder.Message().address(address);
    msg.text('Hello, this is a notification');
    msg.textLocale('en-US');
    bot.send(msg);
}
```

> [!TIP]
> HTTP 요청, 타이머, 큐와 같은 비동기 트리거에서 또는 개발자가 선택하는 다른 위치에서 시작하는 것처럼 임시 사전 대응 메시지를 시작할 수 있습니다.

## <a name="send-a-dialog-based-proactive-message"></a>대화 상자 기반 사전 대응 메시지 보내기

다음 코드 샘플은 Node.js용 Bot Framework SDK를 사용하여 다이얼로그 기반 사전 대응 메시지를 보내는 방법을 보여줍니다. [Microsoft/BotBuilder-Samples/Node/core-proactiveMessages/startNewDialog](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog) 폴더에서 전체 작업 예제를 찾을 수 있습니다.

사용자에게 대화 상자 기반 메시지를 보낼 수 있으려면 봇이 먼저 현재 대화에서 정보를 수집하고 저장해야 합니다. `session.message.address` 개체에는 봇이 사용자에게 대화 상자 기반 사전 대응 메시지를 보내는 데 필요한 모든 정보가 포함됩니다. 

```javascript
// proactiveDialog dialog
bot.dialog('proactiveDialog', function (session, args) {

    savedAddress = session.message.address;

    var message = 'Hey there, I\'m going to interrupt our conversation and start a survey in five seconds...';
    session.send(message);

    message = `You can also make me send a message by accessing: http://localhost:${server.address().port}/api/CustomWebApi`;
    session.send(message);

    setTimeout(() => {
        startProactiveDialog(savedAddress);
    }, 5000);
});
```

이제 메시지를 보낼 때 봇은 새 대화 상자를 만들고 대화 상자 스택의 맨 위에 추가합니다. 새 대화 상자가 대화를 제어하고, 사전 대응 메시지를 전달하고, 닫힌 다음, 스택의 이전 대화 상자에 제어 권한을 반환합니다. 

```javascript
// initiate a dialog proactively 
function startProactiveDialog(address) {
    bot.beginDialog(address, "*:survey");
}
```

> [!NOTE]
> 위의 코드 샘플에는 사용자 지정 파일 **botadapter.js**가 필요합니다([GitHub에서 다운로드](https://github.com/Microsoft/BotBuilder-Samples/blob/master/Node/core-proactiveMessages/startNewDialog/botadapter.js)할 수 있음).

설문 조사 대화 상자는 완료될 때까지 대화를 제어합니다. 그런 다음, 대화 상자가 닫히고(`session.endDialog()` 호출을 통해) 제어 권한을 다시 이전 대화 상자에 반환합니다. 


```javascript
// handle the proactive initiated dialog
bot.dialog('survey', function (session, args, next) {
  if (session.message.text === "done") {
    session.send("Great, back to the original conversation");
    session.endDialog();
  } else {
    session.send('Hello, I\'m the survey dialog. I\'m interrupting your conversation to ask you a question. Type "done" to resume');
  }
});
```

## <a name="sample-code"></a>샘플 코드

Node.js용 Bot Framework SDK를 사용하여 사전 대응 메시지를 보내는 방법을 보여주는 전체 샘플은 GitHub의 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages" target="_blank">Proactive Messages sample</a>(사전 대응 메시지 샘플)을 참조하세요. 사전 대응 메시지 샘플 내에서 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/simpleSendMessage" target="_blank">simpleSendMessage</a>는 임시 사전 대응 메시지를 보내는 방법을 보여주고 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog" target="_blank">startNewDialog</a>는 대화 상자 기반 사전 대응 메시지를 보내는 방법을 보여줍니다.

## <a name="additional-resources"></a>추가 리소스

- [대화 흐름 디자인](../bot-service-design-conversation-flow.md)
