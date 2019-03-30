---
title: 폭포를 사용하여 대화 단계 정의 | Microsoft Docs
description: Node.js용 Bot Framework SDK에서 폭포를 사용하여 대화 단계를 정의하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 526091d61f10ac0c241b994aa3ea99c1d2a70074
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225328"
---
# <a name="define-conversation-steps-with-waterfalls"></a>폭포를 사용하여 대화 단계 정의

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

대화는 일련의 사용자와 봇 간에 교환되는 일련의 메시지입니다. 봇의 목표가 사용자를 일련의 단계로 안내하는 것이면, 폭포를 사용하여 대화의 단계를 정의할 수 있습니다.

[폭포](bot-builder-nodejs-dialog-overview.md)는 사용자로부터 정보를 수집하거나 사용자에게 일련의 태스크를 안내하는 데 가장 일반적으로 사용되는 특정 방식의 다이얼로그 구현입니다. 태스크는 첫 번째 함수의 결과가 다음 함수의 입력으으로 전달되는 방식으로 계속 진행되는 함수 배열로 구현됩니다. 각 함수는 일반적으로 전체 프로세스에서 하나의 단계를 나타냅니다. 각 단계에 대해 봇은 사용자에게 입력을 프롬프트하고, 응답을 기다린 후 결과를 다음 단계로 전달합니다.

이 문서에서는 폭포 작동 방식과 폭포를 사용하여 [대화 흐름을 관리](bot-builder-nodejs-dialog-manage-conversation.md)하는 방법을 이해하는 데 도움을 줍니다.

## <a name="conversation-steps"></a>대화 단계

대화 중에는 일반적으로 사용자와 봇 사이에서 일부 프롬프트/응답 교환이 진행됩니다. 각 프롬프트/응답 교환은 대화에서 앞으로 진행되는 하나의 단계입니다. 2개 정도의 단계가 있는 폭포를 사용하여 대화를 만들 수 있습니다.

예를 들어 다음과 같은 `greetings` 다이얼로그를 고려해 보겠습니다. 폭포의 첫 번째 단계에서는 사용자의 이름을 프롬프트하고, 두 번째 단계에서는 이름을 사용해서 사용자에게 인사말로 응답을 합니다.

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

이러한 방식이 가능한 이유는 프롬프트 때문입니다. Node.js용 Bot Framework SDK는 다양한 유형의 정보를 사용자에게 묻는 데 사용할 수 있는 여러 가지 유형의 기본 제공 [프롬프트](bot-builder-nodejs-dialog-prompt.md)를 제공합니다.

다음 샘플 코드를 4단계 폭포 전체에서 프롬프트를 사용하여 사용자로부터 다양한 정보를 수집하는 다이얼로그를 보여 줍니다.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses a waterfall technique to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        builder.Prompts.text(session, "Whose name will this reservation be under?");
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 
```

이 예제에서 기본 다이얼로그에는 각각 폭포의 하나의 단계를 나타내는 4개의 함수가 있습니다. 각 단계는 사용자에게 입력을 프롬프트하고, 처리를 위해 결과를 다음 단계로 보냅니다. 이 프로세스는 마지막 단계가 실행될 때까지 계속되어 예약을 확인하고 다이얼로그를 종료합니다.

다음 스크린샷은 [Bot Framework Emulator](~/bot-service-debug-emulator.md)에서 이 봇을 실행한 결과를 보여 줍니다.

![폭포를 사용하여 대화 흐름 관리](~/media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

## <a name="create-a-conversation-with-multiple-waterfalls"></a>여러 폭포를 사용하여 대화 만들기

대화 내에 여러 폭포를 사용해서 봇에 필요한 대화 구조를 정의합니다. 예를 들어, 하나의 폭포를 사용하여 대화 흐름을 관리하고, 추가 폭포를 사용하여 사용자로부터 정보를 수집할 수 있습니다. 각 폭포는 다이얼로그에 캡슐화되며, `session.beginDialog` 함수를 호출하여 호출할 수 있습니다.

다음 코드 샘플에서는 하나의 대화에서 여러 폭포를 사용하는 방법을 보여 줍니다. `greetings` 다이얼로그 내의 폭포는 대화의 흐름을 관리하지만 `askName` 다이얼로그 내의 폭포는 사용자로부터 정보를 수집합니다.

```javascript
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog('Hello %s!', results.response);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="advance-the-waterfall"></a>폭포를 따라 이동

폭포는 함수가 배열에 정의된 순서로 단계별로 진행됩니다. 폭포 내의 첫 번째 함수는 다이얼로그에 전달된 인수를 받을 수 있습니다. 폭포 내의 모든 함수는 `next` 함수를 사용하여 사용자에게 입력을 프롬프트하지 않고 다음 단계를 계속 진행할 수 있습니다.

다음 코드 샘플에서는 사용자 프로파일에 대한 정보를 제공하는 프로세스를 사용자에게 안내하는 다이얼로그 내에서 `next` 함수를 사용하는 방법을 보여 줍니다. 폭포의 각 단계에서 봇은 사용자에게 정보를 프롬프트하고(필요한 경우), 폭포의 후속 단계에서 사용자 응답(있는 경우)이 처리됩니다. `ensureProfile` 다이얼로그의 폭포 최종 단계는 다이얼로그를 종료하고, 완성된 프로필 정보를 호출 다이얼로그로 반환합니다. 그러면 호출 다이얼로그는 사용자에게 개인 설정된 인사말을 보냅니다.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot ensures user's profile is up to date.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.beginDialog('ensureProfile', session.userData.profile);
    },
    function (session, results) {
        session.userData.profile = results.response; // Save user profile.
        session.send(`Hello ${session.userData.profile.name}! I love ${session.userData.profile.company}!`);
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

bot.dialog('ensureProfile', [
    function (session, args, next) {
        session.dialogData.profile = args || {}; // Set the profile or create the object.
        if (!session.dialogData.profile.name) {
            builder.Prompts.text(session, "What's your name?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results, next) {
        if (results.response) {
            // Save user's name if we asked for it.
            session.dialogData.profile.name = results.response;
        }
        if (!session.dialogData.profile.company) {
            builder.Prompts.text(session, "What company do you work for?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results) {
        if (results.response) {
            // Save company name if we asked for it.
            session.dialogData.profile.company = results.response;
        }
        session.endDialogWithResult({ response: session.dialogData.profile });
    }
]);
```

> [!NOTE]
> 이 예제에서는 두 개의 데이터 모음 `dialogData` 및 `userData`를 사용하여 데이터를 저장합니다. 봇이이 여러 계산 노드 간에 분산되면 폭포의 각 단계가 다른 노드에 의해 처리될 수 있습니다. 봇 데이터 저장에 대한 자세한 내용은 [상태 데이터 관리](bot-builder-nodejs-state.md)를 참조하세요.

## <a name="end-a-waterfall"></a>폭포 종료

폭포를 사용하여 만든 다이얼로그는 명시적으로 종료해야 합니다. 그렇지 않으면 봇이 폭포를 무한 반복합니다. 다음 방법 중 하나를 사용하여 폭포를 종료할 수 있습니다.

* `session.endDialog`: 호출 다이얼로그에 반환할 데이터가 없는 경우에는 이 메서드를 사용하여 폭포를 종료합니다.

* `session.endDialogWithResult`: 호출 다이얼로그에 반환할 데이터가 있는 경우에는 이 메서드를 사용하여 폭포를 종료합니다. 반환되는 `response` 인수는 JSON 개체 또는 JavaScript 기본 데이터 형식일 수 있습니다. 예를 들면 다음과 같습니다.
  ```javascript
  session.endDialogWithResult({
    response: { name: session.dialogData.name, company: session.dialogData.company }
  });
  ```

* `session.endConversation`: 폭포의 끝이 대화 끝을 나타내는 경우 이 메서드를 사용하여 폭포를 종료합니다.

폭포를 종료하는 이러한 세 가지 방법 중 하나를 사용하여 다이얼로그에 `endConversationAction` 트리거를 연결할 수 있습니다. 예를 들면 다음과 같습니다.

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

## <a name="next-steps"></a>다음 단계

폭포를 사용하면 *프롬프트*를 통해 사용자로부터 정보를 수집할 수 있습니다. 사용자의 입력을 프롬프트하는 방법을 알아보겠습니다.

> [!div class="nextstepaction"]
> [사용자에게 입력 프롬프트](bot-builder-nodejs-dialog-prompt.md)
