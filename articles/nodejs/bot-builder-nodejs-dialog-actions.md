---
title: 사용자 작업 처리 | Microsoft Docs
description: 봇이 Node.js용 Bot Builder SDK를 사용하여 특정 키워드를 포함하는 사용자 입력을 수신 대기하고 처리하도록 활성화하여 사용자 작업을 처리하는 방법에 대해 알아봅니다.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 580d2b75f0db3aff8657eb453f1f3bf6301f7a4d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304226"
---
# <a name="handle-user-actions"></a>사용자 작업 처리

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-global-handlers.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-actions.md)

사용자는 일반적으로 “help”, “cancel” 또는 “start over”와 같은 키워드를 사용하여 봇의 특정 기능에 액세스하려고 합니다. 사용자는 이를 대화 도중 봇이 다른 응답을 기다리는 경우에 자주 수행합니다. **작업**을 구현하면 이러한 요청을 더 정상적으로 처리하도록 봇을 디자인할 수 있습니다. 처리기는 사용자 입력에서 “help”, “cancel” 또는 “start over”와 같은 사용자가 지정한 키워드를 검사하고 적절하게 응답합니다. 

![사용자의 대화 방법](../media/designing-bots/capabilities/trigger-actions.png)

## <a name="action-types"></a>작업 유형

대화 상자에 연결할 수 있는 작업 유형이 아래 표에 나열되어 있습니다. 각 작업 이름에 대한 링크를 통해 해당 작업에 대한 자세한 정보를 제공하는 섹션으로 이동할 수 있습니다.

| 조치 | 범위 | 설명 |
|------|------| ---- |
| [triggerAction](#bind-a-triggeraction) | 전역 | 대화 상자 스택을 지우고 해당 개체를 스택 맨 아래로 푸시하는 대화 상자에 작업을 바인딩합니다. 이 기본 동작을 재정의하려면 `onSelectAction` 옵션을 사용합니다. |
| [customAction](#bind-a-customaction) | 전역 | 대화 상자 스택에 영향을 주지 않고 정보를 처리하거나 조치를 취할 수 있는 봇에 사용자 지정 작업을 바인딩합니다. 이 작업의 기능을 사용자 지정하려면 `onSelectAction` 옵션을 사용합니다. |
[beginDialogAction](#bind-a-begindialogaction) | 컨텍스트 | 트리거될 때 다른 대화 상자를 시작하는 대화 상자에 작업을 바인딩합니다. 시작하는 대화 상자가 스택으로 푸시되면 종료될 때 사라집니다. |
[reloadAction](#bind-a-reloadaction) | 컨텍스트 | 트리거될 때 대화 상자가 다시 로드되도록 하는 대화 상자에 작업을 바인딩합니다. `reloadAction`을 사용하여 “start over”와 같은 사용자 발언을 처리할 수 있습니다. |
[cancelAction](#bind-a-cancelaction) | 컨텍스트 | 트리거될 때 대화 상자를 취소하는 대화 상자에 작업을 바인딩합니다. `cancelAction`을 사용하여 “cancel” 또는 “nevermind”과 같은 사용자 발언을 처리할 수 있습니다. |
[endConversationAction](#bind-an-endconversationaction) | 컨텍스트 | 트리거될 때 사용자와 대화를 종료하는 대화 상자에 작업을 바인딩합니다. `endConversationAction`을 사용하여 “goodbye”와 같은 사용자 발언을 처리할 수 있습니다. |

## <a name="action-precedence"></a>작업 우선순위 

봇이 사용자로부터 발언을 받으면 발언을 대화 상자 스택에 있는 모든 등록된 작업에 대해 확인합니다. 일치는 스택의 맨 위부터 시작되어 스택의 맨 아래로 이어집니다. 아무것도 일치하지 않으면 발언을 모든 글로벌 작업의 `matches` 옵션에 대해 확인합니다.

작업 우선순위는 컨텍스트 차이가 있는 동일한 명령을 사용하는 경우 중요합니다. 예를 들어, 일반적인 도움말로 봇에 대해 “Help” 명령을 사용할 수 있습니다. 또한 각 작업에 대해 “Help”를 사용할 수도 있지만, 이러한 help 명령은 각 작업의 문맥과 관련되어 있습니다. 여기서 더 자세히 설명하는 작업 샘플은 [사용자 입력에 응답](bot-builder-nodejs-dialog-manage-conversation-flow.md#respond-to-user-input)을 참조하세요.

## <a name="bind-actions-to-dialog"></a>대화 상자에 작업 바인딩

사용자 발언 또는 단추 클릭으로 [대화 상자](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html)에 연결된 작업을 *트리거*할 수 있습니다.
*일치*가 지정되면 작업은 사용자가 작업을 트리거하는 단어나 구를 말할 때까지 수신 대기합니다.  `matches` 옵션은 [인식기][RecognizeIntent]의 이름 또는 정규식을 사용할 수 있습니다.
단추 클릭에 동작을 바인딩하려면 [CardAction.dialogAction()][CardAction]을 사용하여 작업을 트리거합니다.

작업은 *연결이 가능*하여 원하는 만큼 많은 작업을 대화 상자에 바인딩할 수 있습니다.

### <a name="bind-a-triggeraction"></a>triggerAction 바인딩

대화 상자에 [triggerAction][triggerAction]을 바인딩하려면 다음을 수행합니다.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will clear the dialog stack and pushes
// the 'orderDinner' dialog onto the bottom of stack.
.triggerAction({
    matches: /^order dinner$/i
});
```

대화 상자에 `triggerAction`을 바인딩하면 봇에 등록됩니다. 트리거되면 `triggerAction`은 대화 상자 스택을 지우고 트리거된 대화 상자를 스택으로 푸시합니다. 이 작업은 [대화의 주제](bot-builder-nodejs-dialog-manage-conversation-flow.md#change-the-topic-of-conversation)를 전환하거나 사용자가 임의의 독립 실행형 작업을 요청하는 데 가장 적합하게 사용됩니다. 이 동작이 대화 상자 스택을 지우는 동작을 재정의하려면 `onSelectAction` 옵션을 `triggerAction`에 추가합니다.

아래 코드 조각에는 대화 상자 스택을 지우지 않으면서 글로벌 컨텍스트에서 일반 도움말을 제공하는 방법을 보여 줍니다.

```javascript
bot.dialog('help', function (session, args, next) {
    //Send a help message
    session.endDialog("Global help menu.");
})
// Once triggered, will start a new dialog as specified by
// the 'onSelectAction' option.
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the top of the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

이 경우 `triggerAction`이 `help` 대화 상자 자체에 연결되어 있습니다(`orderDinner` 대화 상자와 대조적). `onSelectAction` 옵션을 사용하면 대화 상자 스택을 지우지 않으면서 이 대화 상자를 시작할 수 있습니다. 이렇게 하면 “help”, “about”, “support” 등과 같은 글로벌 요청을 처리할 수 있습니다. `onSelectAction` 옵션은 명시적으로 `session.beginDialog` 메서드를 호출하여 트리거된 대화 상자를 시작한다는 점을 유의하세요. 트리거된 대화 상자의 ID는 `args.action`을 통해 제공됩니다. 대화 상자 ID(예: ‘help’)를 이 메서드에 핸드코드하지 마십시오. 그렇지 않으면 런타임 오류가 발생할 수 있습니다. `orderDinner` 작업 자체에 대해 상황에 맞는 도움말 메시지를 트리거하려는 경우 대신 `beginDialogAction`을 `orderDinner` 대화 상자에 연결하는 것을 고려해 보세요.

### <a name="bind-a-customaction"></a>customAction 바인딩

다른 작업 유형과 달리 `customAction`에는 정의된 기본 작업이 없습니다. 이 작업이 무엇을 수행하는지는 사용자가 정의하기 나름입니다. `customAction` 사용의 이점은 대화 상자 스택을 조작하지 않고 사용자 요청을 처리하는 옵션이 있다는 점입니다. `customAction`이 트리거되는 경우 `onSelectAction` 옵션은 새 대화 상자를 스택으로 푸시하지 않고 요청을 처리할 수 있습니다. 작업이 완료되면 컨트롤이 스택의 위쪽에 있는 대화 상자에 다시 전달되고 봇은 계속할 수 있습니다.

`customAction`을 사용하여 “What is the temperature outside right now?(현재 바깥 온도는 몇 도인가?)”, “What time is it right now in Paris?(현재 파리 시각은?)”, “Remind me to buy milk at 5pm today.(오늘 오후 5시에 우유 사라고 알려줘.)” 등과 같은 일반적이고 빠른 작업 요청을 제공할 수 있습니다. 이러한 항목은 봇이 스택을 조작하지 않고도 수행할 수 있는 일반적인 작업입니다.

`customAction`과의 또 다른 주요 차이점은 대화 상자가 아닌 봇에 바인딩한다는 점입니다.

다음 코드 샘플은 `customAction`을 미리 알림을 설정하는 요청을 수신 대기하는 `bot`에 바인딩하는 방법을 보여 줍니다.

```javascript
bot.customAction({
    matches: /remind|reminder/gi,
    onSelectAction: (session, args, next) => {
        // Set reminder...
        session.send("Reminder is set.");
    }
})
```

### <a name="bind-a-begindialogaction"></a>beginDialogAction 바인딩

대화 상자에 `beginDialogAction`을 바인딩하면 대화 상자에 작업이 등록됩니다. 이 메서드는 트리거될 때 다른 대화 상자를 시작합니다. 이 작업의 동작은 [beginDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#begindialog) 메서드 호출과 유사합니다. 새 대화 상자가 대화 상자 스택의 맨 위로 푸시되어 현재 작업이 자동으로 종료되지 않습니다. 현재 작업은 새 대화 상자가 종료되면 계속됩니다. 

다음 코드 조각은 [beginDialogAction][beginDialogAction]을 대화 상자에 바인딩하는 방법을 보여 줍니다.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i
});

// Show dinner items in cart
bot.dialog('showDinnerCart', function(session){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    // End this dialog
    session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
});
```

새 대화 상자에 추가 인수를 전달해야 하는 경우 [`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) 옵션을 작업에 추가할 수 있습니다.

위의 샘플을 사용하여 `dialogArgs`를 통해 전달된 인수를 수락하도록 수정할 수 있습니다.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i,
    dialogArgs: {
        showTotal: true;
    }
});

// Show dinner items in cart with the option to show total or not.
bot.dialog('showDinnerCart', function(session, args){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    if(args && args.showTotal){
        // End this dialog with total.
        session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
    }
    else{
        session.endDialog(); // Ends without a message.
    }
});
```

### <a name="bind-a-reloadaction"></a>reloadAction 바인딩

대화 상자에 `reloadAction`을 바인딩하면 대화 상자에 작업이 등록됩니다. 대화 상자에 이 작업을 바인딩하면 작업이 트리거될 때 대화 상자가 다시 시작됩니다. 이 작업을 트리거하는 것은 [replaceDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#replacedialog) 메서드를 호출하는 것과 유사합니다. 이는 “start over”와 같은 사용자 발언을 처리하거나 [loops](bot-builder-nodejs-dialog-replace.md#repeat-an-action)를 만들도록 논리를 구현하는 데 유용합니다.

다음 코드 조각은 [reloadAction][reloadAction]을 대화 상자에 바인딩하는 방법을 보여 줍니다.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i
});
```

다시 로드된 대화 상자에 추가 인수를 전달해야 하는 경우 [`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) 옵션을 작업에 추가할 수 있습니다. 이 옵션은 `args` 매개 변수로 전달됩니다. 다시 로드 작업에서 인수를 받기 위해 위의 샘플 코드를 다시 작성하면 다음과 같이 표시됩니다.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    function(session, args, next){
        if(args && args.isReloaded){
            // Reload action was triggered.
        }

        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    }
    //...other waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i,
    dialogArgs: {
        isReloaded: true;
    }
});
```

### <a name="bind-a-cancelaction"></a>cancelAction 바인딩

`cancelAction`을 바인딩하면 대화 상자에 등록됩니다. 트리거되면 이 작업은 대화 상자를 갑자기 종료합니다. 대화 상자가 종료되면 부모 대화 상자는 `canceled`되었음을 표시하는 다시 시작된 코드와 함께 다시 시작됩니다. 이 작업을 사용 하면 “nevermind” 또는 “cancel”과 같은 발언을 처리할 수 있습니다. 대신 프로그래밍 방식으로 대화 상자를 취소해야 할 경우 [대화 상자 취소](bot-builder-nodejs-dialog-replace.md#cancel-a-dialog)를 참조하세요. *다시 시작된 코드*에 대한 자세한 내용은 [프롬프트 결과](bot-builder-nodejs-dialog-prompt.md#prompt-results)를 참조하세요. 

다음 코드 조각은 [cancelAction][cancelAction]을 대화 상자에 바인딩하는 방법을 보여 줍니다.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
//Once triggered, will end the dialog.
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i
});
```

### <a name="bind-an-endconversationaction"></a>endConversationAction 바인딩

`endConversationAction`을 바인딩하면 대화 상자에 등록됩니다. 트리거되면 이 작업은 사용자와 대화를 종료합니다. 이 작업을 트리거하는 것은 [endConversation](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#endconversation) 메서드를 호출하는 것과 유사합니다. 대화가 종료되면 Node.js용 Bot Builder SDK는 대화 스택 및 유지된 상태 데이터를 지웁니다. 지속된 상태 데이터에 대한 자세한 내용은 [상태 데이터 관리](bot-builder-nodejs-state.md)를 참조하세요.

다음 코드 조각은 [endConversationAction][endConversationAction]을 대화 상자에 바인딩하는 방법을 보여 줍니다.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will end the conversation.
.endConversationAction('endConversationAction', 'Ok, goodbye!', {
    matches: /^goodbye$/i
});
```

## <a name="confirm-interruptions"></a>중단 확인

모두는 아닐지라도 대부분 이러한 모든 작업은 대화의 정상 흐름을 중단시킵니다. 많은 대화가 중단되며, 신중하게 처리되어야 합니다. 예를 들어 `triggerAction`, `cancelAction` 또는 `endConversationAction`은 대화 상자 스택을 지웁니다. 사용자가 이러한 작업들을 실수로 트리거한 경우 작업을 다시 시작해야 합니다. `confirmPrompt` 옵션을 이러한 작업에 추가하면 사용자가 실제로 이러한 작업을 트리거할 의도인지 확인할 수 있습니다. `confirmPrompt`는 사용자가 현재 작업을 취소 또는 종료할 것인지를 묻습니다. 사용자는 이때 마음을 바꾸거나 프로세스를 계속할 수 있습니다.

아래 코드 조각에는 [confirmPrompt](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#confirmprompt)를 통해 사용자가 정말로 주문 프로세스를 취소하려는지 확인하는 [cancelAction][cancelAction]이 나와 있습니다.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Confirm before triggering the action.
// Once triggered, will end the dialog. 
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i,
    confirmPrompt: "Are you sure?"
});
```

이 작업이 트리거되면 사용자에게 “정말로 진행하시겠습니까?”라고 묻는 메시지가 표시됩니다. 사용자는 “Yes(예)”라고 대답하여 해당 작업을 진행하거나 “No(아니오)”라고 대답하여 작업을 취소하고 원래 하던 곳에서 계속할 수 있습니다.

## <a name="next-steps"></a>다음 단계

**작업**을 사용하면 사용자 요청을 예상하고 봇이 해당 요청을 정상적으로 처리하도록 할 수 있습니다. 이러한 작업의 대부분은 현재 대화 흐름을 중단시킵니다. 사용자가 대화 흐름을 전환하고 다시 시작하도록 허용하려는 경우 전환하기 전에 사용자 상태를 저장해야 합니다. 사용자 상태를 저장하고 상태 데이터를 관리하는 방법에 대해 더 자세히 살펴보겠습니다.

> [!div class="nextstepaction"]
> [상태 데이터 관리](bot-builder-nodejs-state.md)


[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[cancelAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction

[reloadAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction

[beginDialogAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#begindialogaction

[endConversationAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#endconversationaction

[RecognizeIntent]: bot-builder-nodejs-recognize-intent-messages.md

[CardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.cardaction#dialogaction
