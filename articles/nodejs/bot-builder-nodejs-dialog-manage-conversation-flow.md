---
title: dialog를 사용하여 대화 흐름 관리 | Microsoft Docs
description: Node.js용 Bot Builder SDK에서 dialog를 사용하여 봇과 사용자 사이의 대화를 관리하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 133f085a857d1bb8bf7622e7adab19374902327d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997770"
---
# <a name="manage-conversation-flow-with-dialogs"></a>다이얼로그를 사용하여 대화 흐름 관리

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

대화 흐름 관리는 봇을 빌드하는 데 필수적인 작업입니다. 봇은 핵심적인 작업을 원활하게 수행하고 중단을 정상적으로 처리할 수 있어야 합니다. Node.js용 Bot Builder SDK를 사용하면 dialog를 사용하여 대화 흐름을 관리할 수 있습니다.

dialog는 프로그램의 함수와 같습니다. 대개 특정 작업을 수행하도록 디자인되며 필요한 만큼 호출할 수 있습니다. 여러 개의 dialog를 함께 연결하여 봇이 처리할 대화 흐름을 처리할 수 있습니다. Node.js용 Bot Builder SDK에는 대화 흐름을 관리하는 데 도움이 되는 [prompt](bot-builder-nodejs-dialog-prompt.md) 및 [waterfall](bot-builder-nodejs-dialog-waterfall.md)과 같은 기본 제공 기능이 포함됩니다.

이 문서에서는 간단한 대화 흐름과 복잡한 대화 흐름을 관리하는 방법을 설명하는 일련의 예제를 제공합니다. 여기서 봇은 dialog를 사용하여 중단을 처리하고 정상적으로 흐름을 재개할 수 있습니다. 예제는 다음 시나리오를 기반으로 합니다. 

1. 봇이 저녁 식사 예약을 받습니다.
2. 봇은 예약 도중 언제든지 "Help" 요청을 처리할 수 있습니다.
3. 봇은 현재 예약 단계의 상황에 맞는 "Help"를 처리할 수 있습니다.
4. 봇은 여러 가지 대화 주제를 처리할 수 있습니다.

## <a name="manage-conversation-flow-with-a-waterfall"></a>waterfall을 사용하여 대화 흐름 관리

[waterfall](bot-builder-nodejs-dialog-waterfall.md)은 봇이 일련의 작업을 진행하는 동안 사용자를 쉽게 안내할 수 있도록 해 주는 dialog입니다. 이 예제에서는 봇이 예약 요청을 처리할 수 있도록 예약 봇이 사용자에게 일련의 질문을 합니다. 봇은 사용자에게 다음 정보를 질문합니다.

1. 예약 날짜 및 시간
2. 파티 참가 인원
3. 사람자의 이름

다음 코드 샘플은 waterfall을 사용하여 일련의 프롬프트를 진행하면서 사용자를 안내하는 방법을 보여줍니다.

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
        builder.Prompts.number(session, "How many people are in your party?");
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

이 봇의 핵심 기능은 기본 dialog에서 발생합니다. 기본 dialog는 봇이 생성될 때 정의됩니다. 

```javascript
var bot = new builder.UniversalBot(connector, [..waterfall steps..]); 
```

또한 생성 과정을 진행하는 동안 어떤 [데이터 저장소](bot-builder-nodejs-state.md)를 사용할지 설정할 수 있습니다. 예를 들어 메모리 내 저장소를 사용하려는 경우에는 다음과 같이 설정할 수 있습니다.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..]).set('storage', inMemoryStorage); // Register in-memory storage 
```

기본 dialog는 waterfall의 단계를 정의하는 함수의 배열로 만들어집니다. 예를 들어 함수가 4개 있으면 waterfall은 4단계로 구성됩니다. 각 단계는 단일 작업을 처리하고 그 결과는 다음 단계에서 처리됩니다. 이 프로세스는 마지막 단계까지 계속되어 예약이 확인되고 가 dialog가 끝납니다.

다음 스크린샷은 [Bot Framework Emulator](../bot-service-debug-emulator.md)에서 이 봇을 실행한 결과를 보여줍니다.

![waterfall을 사용하여 대화 흐름 관리](../media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

### <a name="prompt-user-for-input"></a>사용자에게 입력 요청

이 예제의 각 단계에는 사용자에게 입력을 요구하는 프롬프트가 사용됩니다. 프롬프트는 사용자에게 입력을 요청하고, 응답을 기다리고, 응답을 waterfall의 다음 단계에 반환하는 특별한 형식의 dialog입니다. 봇에서 사용할 수 있는 다양한 형식의 프롬프트에 대한 정보는 [사용자에게 입력 요청](bot-builder-nodejs-dialog-prompt.md)을 참조하세요.

이 예제에서 봇은 `Prompts.text()`를 사용하여 사용자에게 텍스트 형식의 자유형 응답을 요청합니다. 사용자는 원하는 텍스트로 응답할 수 있고 봇은 이 응답을 어떻게 처리할지 결정해야 합니다. `Prompts.time()`은 [Chrono](https://github.com/wanasit/chrono) 라이브러리를 사용하여 문자열에서 날짜와 시간 정보를 구문 분석합니다. 따라서 날짜와 시간을 지정하는 자연어를 봇이 더 잘 이해할 수 있습니다. 예: '2017년 6월 6일 오후 9시', '오늘 오후 7시 30분', '다음 주 월요일 오후 6시' 등

> [!TIP] 
> 사용자가 입력하는 시간은 봇을 호스트하는 서버의 표준 시간대에 따라 UTC 시간으로 변환됩니다. 서버가 사용자와 다른 표준 시간대에 있을 수 있으므로 표준 시간대를 고려해야 합니다. 날짜 및 시간을 사용자의 현지 시간으로 변환하려면 사용자에게 사용자의 표준 시간대를 요청하는 것이 좋습니다.

## <a name="manage-a-conversation-flow-with-multiple-dialogs"></a>여러 dialog를 사용하여 대화 흐름 관리

대화 흐름을 관리하는 또 다른 기법은 waterfall과 여러 dialog를 함께 사용하는 것입니다. waterfall을 사용하면 하나의 dialog에 여러 함수를 함께 연결할 수 있으며 dialog를 사용하면 대화를 언제든지 다시 사용할 수 있는 작은 기능 조각으로 나눌 수 있습니다.

예를 들면 저녁 식사 예약 봇이 있습니다. 다음 코드 샘플은 waterfall과 여러 dialog를 사용하도록 다시 작성된 이전 예제를 보여줍니다.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses multiple dialogs to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Dialog to ask for a date and time
bot.dialog('askForDateTime', [
    function (session) {
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);

// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
])

// Dialog to ask for the reservation name.
bot.dialog('askForReserverName', [
    function (session) {
        builder.Prompts.text(session, "Who's name will this reservation be under?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

이 봇을 실행한 결과는 waterfall만 사용했던 이전 봇과 정확히 같습니다. 하지만 프로그래밍 방식으로는 두 가지 주요한 차이점이 있습니다.

1. 기본 dialog는 대화 흐름을 관리하는 데만 사용됩니다.
2. 대화의 각 단계별 작업은 별도의 dialog로 관리됩니다. 이 경우 봇에 세 가지 정보가 필요하기 때문에 사용자에게 세 번 질문합니다. 이제 각 프롬프트가 자체 dialog에 포함되어 있습니다.

이 기법을 사용하면 대화 흐름을 작업 논리와 분리할 수 있습니다. 이렇게 하면 필요한 경우 다른 대화 흐름에 dialog를 다시 사용할 수 있습니다. 

## <a name="respond-to-user-input"></a>사용자 입력에 응답

일련의 작업을 통해 사용자를 안내하는 과정에서, 답변을 하기 전에 사용자가 질문을 하거나 추가 정보를 요청하면 어떻게 요청을 처리하시겠습니까? 예를 들어 사용자가 대화에 참여하는 곳에 관계없이 사용자가 "도움말", "지원" 또는 "취소"를 입력하면 봇이 어떻게 응답해야 할까요? 사용자가 단계에 대한 정보를 더 원하는 경우에는 어떻게 해야 할까요? 사용자가 마음을 바꿔서 현재 작업을 포기하고 완전히 다른 작업을 시작하려고 하면 어떻게 되나요?

Node.js용 Bot Builder SDK에서는 봇이 현재 dialog의 범위에서 글로벌 컨텍스트 또는 로컬 컨텍스트 내의 특정 입력을 수신 대기할 수 있습니다. 이러한 입력을 [action](bot-builder-nodejs-dialog-actions.md)(동작)이라고 하며, 봇은 `matches` 절을 기반으로 사용자 입력을 수신 대기할 수 있습니다. 특정 사용자 입력에 대응하는 방식을 결정하는 것은 봇에게 달려 있습니다.

### <a name="handle-global-action"></a>글로벌 동작 처리

대화 중 어느 시점에서든 봇이 동작을 처리할 수 있도록 하려면 `triggerAction`을 사용합니다. 트리거를 사용하면 입력한 내용이 지정된 용어와 일치하는 경우 봇이 특정 dialog를 호출할 수 있습니다. 예를 들어 글로벌 "Help" 옵션을 지원하려는 경우에는 Help dialog를 만들고 "Help"와 일치하는 입력 내용을 수신 대기하는 `triggerAction`을 연결할 수 있습니다.

다음 코드 샘플은 dialog에 `triggerAction`을 연결하여 사용자가 "help"를 입력할 때 dialog가 호출되도록 지정하는 방법을 보여줍니다.

```javascript
// The dialog stack is cleared and this dialog is invoked when the user enters 'help'.
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
});
```

기본적으로 `triggerAction`이 실행되면 dialog 스택이 지워지고 트리거된 dialog가 새로운 기본 dialog가 됩니다. 이 예제에서 `triggerAction`이 실행되면 dialog 스택이 지워지고 `help` dialog가 새 기본 dialog로 스택에 추가됩니다. 이것이 원하는 동작이 아니면 `onSelectAction` 옵션을 `triggerAction`에 추가할 수 있습니다. `onSelectAction` 옵션을 사용하면 dialog 스택을 지우지 않고도 봇이 새 dialog를 시작할 수 있습니다. 그러면 대화를 일시적으로 리디렉션했다가 나중에 중단한 곳에서 다시 시작할 수 있습니다.

다음 코드는 `help` dialog가 기존 dialog 스택에 추가될 수 있도록(dialog 스택이 지워지지 않고) `triggerAction`과 `onSelectAction` 옵션을 사용하는 방법을 보여줍니다.

```javascript
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

이 예제에서 사용자가 "help"를 입력하면 `help` dialog가 대화의 컨트롤을 가져갑니다. `triggerAction`에 `onSelectAction` 옵션이 포함되어 있기 때문에 `help` dialog가 기존 dialog 스택으로 푸시되고 스택이 지워지지 않습니다. `help` dialog가 끝나면 dialog 스택에서 제거되고 `help` 명령으로 중단된 지점에서 대화가 재개됩니다.

### <a name="handle-contextual-action"></a>상황에 맞는 동작 처리

이전 예제에서는 대화의 어느 지점에서든 사용자가 "help"를 입력하면 `help` dialog가 호출됩니다. 이런 경우 사용자의 help 요청에 대한 구체적인 컨텍스트가 없기 때문에 dialog는 일반적인 help 안내만 제공할 수 있습니다. 하지만 사용자가 대화의 특정 부분에 대한 도움을 요청하려면 어떻게 해야 할까요? 이런 경우 현재 dialog의 컨텍스트 내에서 `help` dialog가 트리거되야 합니다.

예를 들면 저녁 식사 예약 봇이 있습니다. 사용자가 파티 참가 인원에 대한 질문을 받고 최대 파티 규모를 알고 싶어 하면 어떻게 해야 할까요? 이런 시나리오를 처리하려면 `beginDialogAction`을 `askForPartySize` dialog에 연결하여 사용자의 "help" 입력을 수신 대기할 수 있습니다.

다음 코드 샘플은 `beginDialogAction`을 사용하여 상황에 맞는 도움말을 dialog에 연결하는 방법을 보여줍니다.

```javascript
// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
       session.endDialogWithResult(results);
    }
])
.beginDialogAction('partySizeHelpAction', 'partySizeHelp', { matches: /^help$/i });

// Context Help dialog for party size
bot.dialog('partySizeHelp', function(session, args, next) {
    var msg = "Party size help: Our restaurant can support party sizes up to 150 members.";
    session.endDialog(msg);
})
```

이 예제에서는 사용자가 "help"를 입력할 때마다 봇이 `partySizeHelp` dialog를 스택으로 푸시합니다. 이 dialog는 사용자에게 도움말 메시지를 보낸 다음, dialog를 끝내고 `askForPartySize` dialog에 컨트롤을 반환하여 사용자에게 파티 규모를 다시 질문합니다.

상황에 맞는 도움말은 사용자가 `askForPartySize` dialog에 있을 때만 실행된다는 점에 유의해야 합니다. 그렇지 않으면 `triggerAction`의 일반 도움말 메시지가 대신 실행됩니다. 즉, 로컬 `matches` 절은 글로벌 `matches` 절보다 항상 우선합니다. 예를 들어 `beginDialogAction`이 **help**와 일치하면 `triggerAction`에서 **help**에 일치하는 항목이 실행되지 않습니다. 자세한 내용은 [동작 우선순위](bot-builder-nodejs-dialog-actions.md#action-precedence)를 참조하세요.

### <a name="change-the-topic-of-conversation"></a>대화의 주제 변경

기본적으로 `triggerAction`을 실행하면 dialog 스택이 지워지고 지정된 dialog부터 시작하도록 dialog가 다시 설정됩니다. 이런 동작은 봇이 특정 대화 주제에서 다른 대화 주제로 전환해야 할 때 유용한 경우가 많습니다. 예를 들어 사용자가 저녁 식사를 한참 예약하다가 중간에 마음을 바꿔서 저녁 식사를 호텔 방으로 배달하는 주문을 하기로 결정하는 경우가 그렇습니다. 

다음 예제는 사용자가 봇에게 저녁 식사를 예약하거나 저녁 식사 배달을 주문할 수 있는 이전 예제를 기반으로 합니다. 이 봇에서 기본 dialog는 사용자에게 두 가지 옵션 즉, `Dinner Reservation` 및 `Order Dinner`를 제공하는 인사말 dialog입니다.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot enables users to either make a dinner reservation or order dinner.
var bot = new builder.UniversalBot(connector, function(session){
    var msg = "Welcome to the reservation bot. Please say `Dinner Reservation` or `Order Dinner`";
    session.send(msg);
}).set('storage', inMemoryStorage); // Register in-memory storage 
```

이전 예제의 기본 dialog에 있던 저녁 예약 논리가 이제 `dinnerReservation`이라는 자체 dialog에 있습니다. `dinnerReservation`의 흐름은 앞에서 설명한 여러 dialog 버전과 동일합니다. 유일한 차이는 dialog에 `triggerAction`이 연결되어 있는 것입니다. 이 버전에서 `confirmPrompt`는 새 dialog를 호출하기 전에 사용자에게 대화 주제를 변경하고 싶은지 확인하도록 요청합니다. 이와 같은 시나리오에는 일단 dialog 스택이 지워지면, 사용자가 새로운 대화 주제로 이동하게 되고, 진행하던 대화 주제는 포기하게 되기 때문에 `confirmPrompt`를 포함하는 것이 좋습니다.

```javascript
// This dialog helps the user make a dinner reservation.
bot.dialog('dinnerReservation', [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
])
.triggerAction({
    matches: /^dinner reservation$/i,
    confirmPrompt: "This will cancel your current request. Are you sure?"
});
```

대화의 두 번째 주제는 waterfall을 사용하여 `orderDinner` dialog에 정의됩니다. 이 dialog는 저녁 식사 메뉴만 표시하고 주문이 정해지면 사용자에게 방 번호를 질문합니다. `triggerAction`은 dialog에 연결되어 사용자가 "order dinner"를 입력하면 호출되어야 한다는 것을 지정하고 대화의 주제를 바꾸고 싶어 하면 선택 사항을 확인하도록 묻는 질문이 표시되도록 합니다.

```javascript
// This dialog help the user order dinner to be delivered to their hotel room.
var dinnerMenu = {
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50":{
        Description: "Clam Chowder",
        Price: 4.50
    }
};

bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endDialog(msg);
        }
    }
])
.triggerAction({
    matches: /^order dinner$/i,
    confirmPrompt: "This will cancel your order. Are you sure?"
});
```

사용자가 대화를 시작하고 `Dinner Reservation` 또는 `Order Dinner`를 선택한 후에도 언제든지 마음을 바꿀 수 있습니다. 예를 들어 사용자가 저녁 식사를 예약하는 도중에 "order dinner(저녁 식사 주문)"을 입력하면 봇은 "This will cancel your current request. Are you sure?(그러면 현재 요청이 취소됩니다. 계속하시겠습니까?)"라고 말하면서 확인합니다. 사용자가 "no(아니오)"를 입력하면 요청이 취소되고 사용자는 저녁 식사 예약 프로세스를 계속 진행할 수 있습니다. 사용자가 "yes(예)"를 입력하면 봇은 dialog 스택을 지우고 대화의 컨트롤을 `orderDinner` dialog로 넘깁니다.

## <a name="end-conversation"></a>대화 종료

위의 예제에서 dialog는 `session.endDialog`나 `session.endDialogWithResult`를 사용하여 닫히고, 두 가지 모두 dialog를 끝내고, dialog를 스택에서 제거하고, 컨트롤을 호출한 dialog로 반환합니다. 사용자가 대화의 끝에 도달하면 `session.endConversation`을 사용하여 대화가 완료되었음을 나타내야 합니다.

[`session.endConversation`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) 메서드는 대화를 끝내고 선택적으로 사용자에게 메시지를 보냅니다. 예를 들어, 앞의 예제에서 `orderDinner` dialog는 다음 코드 샘플처럼 `session.endConversation`을 사용하여 대화를 끝낼 수 있습니다.

```javascript
bot.dialog('orderDinner', [
    //...waterfall steps...
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
]);
```

`session.endConversation`을 호출하면 dialog 스택을 지우고 [`session.conversationData`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#conversationdata) 저장소를 재설정하여 대화를 종료합니다. 데이터 저장소에 대한 자세한 내용은 [상태 데이터 관리](bot-builder-nodejs-state.md)를 참조하세요.

`session.endConversation` 호출은 봇이 설계된 대화 흐름을 사용자가 완료하면 수행해야 하는 논리적인 작업입니다. 사용자가 대화 중에 "cancel(취소)" 또는 "goodbye(안녕)"를 입력하는 상황에 `session.endConversation`을 사용하여 대화를 끝낼 수도 있습니다. 이렇게 하려면 dialog에 `endConversationAction`을 연결하고 이 트리거가 "cancel(취소)" 또는 "goodbye(안녕)"와 일치하는 입력을 수신 대기하도록 설정합니다.

다음 코드 샘플은 `endConversationAction`을 dialog에 연결하여 사용자가 "cancel(취소)" 또는 "goodbye(안녕)"를 입력하면 대화를 끝내는 방법을 보여줍니다.

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

`session.endConversation` 또는 `endConversationAction`을 사용하여 대화를 끝내면 dialog 스택이 지워지고 사용자가 처음부터 다시 시작해야 하므로 `confirmPrompt`를 포함시켜서 사용자가 정말로 원하는지 확인해야 합니다.

## <a name="next-steps"></a>다음 단계

이 문서에서는 사실상 순차적인 대화를 관리하는 방법을 알아봅니다. dialog를 반복하거나 대화에 반복 패턴을 사용하려면 어떻게 해야 할까요? 스택에서 dialog를 대체하여 수행할 수 있는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [dialog 대체](bot-builder-nodejs-dialog-replace.md)

