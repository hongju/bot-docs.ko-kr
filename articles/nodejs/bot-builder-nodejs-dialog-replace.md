---
title: 다이얼로그 바꾸기 | Microsoft Docs
description: 다이얼로그를 교체하여 입력을 다시 프롬프트하고, Node.js용 Bot Builder SDK를 사용하여 대화 흐름을 관리하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d1a2eca2ebdfccf6dacdf773c48c8b41444559ca
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301731"
---
# <a name="replace-dialogs"></a>다이얼로그 바꾸기

다이얼로그를 바꾸는 기능은 대화 과정 중에 사용자 입력이 유효한지 검사하거나 작업을 반복적으로 수행해야 할 경우에 유용할 수 있습니다. Node.js용 Bot Builder SDK를 사용하면 [`session.replaceDialog`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog) 메서드를 사용하여 다이얼로그를 바꿀 수 있습니다. 이 메서드를 사용하면 호출자로 돌아가지 않고도 현재 다이얼로그를 종료한 후 새 다이얼로그로 바꿀 수 있습니다. 

## <a name="create-custom-prompts-to-validate-input"></a>입력 유효성 검사를 수행하는 사용자 지정 프롬프트 만들기

Node.js용 Bot Builder SDK에는 특정 형식의 [프롬프트](bot-builder-nodejs-dialog-prompt.md)(예: `Prompts.time` 및 `Prompts.choice`)에 대한 입력 유효성 검사가 포함되어 있습니다. `Prompts.text`에 대한 응답으로, 수신된 텍스트 입력이 유효한지 검사하려면 자체 유효성 검사 논리 및 사용자 지정 프롬프트를 만들어야 합니다. 

입력이 지정한 특정 값, 패턴, 범위 또는 조건을 준수해야 할 경우 입력이 유효한지 검사하려고 할 수 있습니다. 입력 유효성 검사가 실패하면 봇은 `session.replaceDialog` 메서드를 사용하여 해당 정보를 사용자에게 다시 물어볼 수 있습니다.

다음 코드 샘플에서는 사용자 입력에서 전화번호가 유효한지 검사하는 사용자 지정 프롬프트를 만드는 방법을 보여 줍니다.

```javascript
// This dialog prompts the user for a phone number. 
// It will re-prompt the user if the input does not match a pattern for phone number.
bot.dialog('phonePrompt', [
    function (session, args) {
        if (args && args.reprompt) {
            builder.Prompts.text(session, "Enter the number using a format of either: '(555) 123-4567' or '555-123-4567' or '5551234567'")
        } else {
            builder.Prompts.text(session, "What's your phone number?");
        }
    },
    function (session, results) {
        var matched = results.response.match(/\d+/g);
        var number = matched ? matched.join('') : '';
        if (number.length == 10 || number.length == 11) {
            session.userData.phoneNumber = number; // Save the number.
            session.endDialogWithResult({ response: number });
        } else {
            // Repeat the dialog
            session.replaceDialog('phonePrompt', { reprompt: true });
        }
    }
]);
```

이 예제에서 사용자는 처음에 해당 전화번호를 제공하도록 요구됩니다. 유효성 검사 논리는 사용자 입력에서 숫자 범위와 일치하는 정규식을 사용합니다. 입력에 10자리 또는 11자리 숫자가 포함되어 있으면 해당 숫자가 응답에 반환됩니다. 그렇지 않으면, `session.replaceDialog` 메서드가 실행되면서 `phonePrompt` 다이얼로그를 반복하여 사용자에게 다시 입력하도록 프로프트합니다. 단, 이번에는 예상되는 입력 형식에 대해 보다 구체적인 지침이 제공됩니다.

`session.replaceDialog` 메서드를 호출할 때는 반복할 다이얼로그의 이름과 인수 목록을 지정합니다. 이 예제에서 인수 목록에는 봇이 초기 프롬프트인지 재프롬프트인지에 따라 다른 프롬프트 메시지를 제공하도록 하는 `{ reprompt: true }`가 포함되어 있습니다. 그렇지만 사용자가 봇이 요구할 수 있는 인수를 지정할 수 있습니다. 

## <a name="repeat-an-action"></a>작업 반복

대화 과정에서 사용자에게 특정 작업을 여러 번 반복할 수 있도록 하는 다이얼로그를 반복하려는 경우가 여러 번 있을 수 있습니다. 예를 들어, 봇이 다양한 서비스를 제공하는 경우, 처음에는 서비스 메뉴를 표시하고, 서비스 요청 프로세스를 안내한 후, 서비스 메뉴를 다시 표시하여 사용자가 다른 서비스를 요청할 수 있도록 할 수 있습니다. 이렇게 하려면 ['session.endConversation'](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) 메서드를 사용하여 대화를 종료하는 대신 [`session.replaceDialog`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog) 메서드를 사용하여 서비스 메뉴를 다시 표시할 수 있습니다. 

다음 예제에서는 `session.replaceDialog` 메서드를 사용하여 이러한 유형의 시나리오를 구현하는 방법을 보여 줍니다. 먼저 서비스의 메뉴가 정의됩니다.

```javascript
// Main menu
var menuItems = { 
    "Order dinner": {
        item: "orderDinner"
    },
    "Dinner reservation": {
        item: "dinnerReservation"
    },
    "Schedule shuttle": {
        item: "scheduleShuttle"
    },
    "Request wake-up call": {
        item: "wakeupCall"
    },
}
```

기본 다이얼로그에 의해 `mainMenu` 다이얼로그가 호출되므로, 대화를 시작할 때 사용자에게 메뉴가 제공됩니다. 또한 메뉴가 사용자가 “main menu”를 입력할 때마다 제공되도록 [`triggerAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction)가 `mainMenu`에 추가됩니다. 사용자에게 메뉴가 제공되고 사용자가 옵션을 선택하면 사용자의 선택에 해당하는 다이얼로그가 `session.beginDialog` 메서드를 사용하여 호출됩니다.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a reservation bot that has a menu of offerings.
var bot = new builder.UniversalBot(connector, [
    function(session){
        session.send("Welcome to Contoso Hotel and Resort.");
        session.beginDialog("mainMenu");
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Display the main menu and start a new request depending on user input.
bot.dialog("mainMenu", [
    function(session){
        builder.Prompts.choice(session, "Main Menu:", menuItems);
    },
    function(session, results){
        if(results.response){
            session.beginDialog(menuItems[results.response.entity].item);
        }
    }
])
.triggerAction({
    // The user can request this at any time.
    // Once triggered, it clears the stack and prompts the main menu again.
    matches: /^main menu$/i,
    confirmPrompt: "This will cancel your request. Are you sure?"
});
```

이 예제에서 사용자가 옵션 1을 사용하여 객실로 저녁을 주문하는 경우 `orderDinner` 다이얼로그가 호출되고, 사용자를 저녁 주문 프로세스로 안내합니다. 이 프로세스가 끝나면 봇은 주문을 확인하고, `session.replaceDialog` 메서드를 사용하여 기본 메뉴를 다시 표시합니다.

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner to be delivered to their hotel room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: %(Description)s for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}.`;
            session.send(msg);
            session.replaceDialog("mainMenu"); // Display the menu again.
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
)
.cancelAction(
    "cancelOrder", "Type 'Main Menu' to continue.", 
    {
        matches: /^cancel$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

2개의 트리거가 `orderDinner` 다이얼로그에 연결되어 사용자가 주문 프로세스 중 언제든지 주문을 "다시 시작" 또는 "취소"할 수 있도록 합니다. 

첫 번째 트리거는 사용자가 입력 “start over”를 보내서 주문 프로세스를 다시 시작할 수 있도록 하는 [`reloadAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction)입니다. 이 트리거가 발언 "start over"와 일치하면 `reloadAction`은 다이얼로그를 처음부터 다시 시작합니다. 

두 번째 트리거는 사용자가 입력 “cancel”을 보내서 주문 프로세스를 중단할 수 있도록 하는 [`cancelAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction)입니다. 이 트리거는 기본 메뉴를 자동으로 다시 표시하지 않고, 대신 “Type ‘Main Menu’ to continue”라고 말하여 다음에 수행할 작업을 사용자에게 알려 주는 메시지를 전송합니다.

### <a name="dialog-loops"></a>다이얼로그 루프

위의 예제에서 사용자는 주문당 하나의 항목만 선택할 수 있습니다. 즉, 사용자가 메뉴에서 두 개의 항목을 주문하기 원할 경우 첫 번째 항목에 대해 전체 주문 프로세스를 완료한 후 두 번째 항목에 대해 전체 주문 프로세스를 반복해야 합니다. 

다음 예제에서는 저녁 메뉴를 별도 다이얼로그로 리팩터링하여 이전 봇을 개선하는 방법을 보여 줍니다. 이렇게 하면 봇이 저녁 메뉴를 루프로 반복할 수 있으므로, 사용자가 단일 주문 내에서 여러 항목을 선택할 수 있습니다.

첫째, 메뉴에 "Check out" 옵션을 추가합니다. 이 옵션은 사용자가 항목 선택 프로세스를 끝내고 체크 아웃 프로세스를 계속하도록 합니다.

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0 // Order total. Updated as items are added to order.
    }
};
```

다음으로, 봇이 메뉴를 반복하고 사용자가 해당 주문에 여러 항목을 추가할 수 있도록 주문 프롬프트를 자체 다이얼로그로 리팩터링합니다.

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    function(session, args){
        if(args && args.reprompt){
            session.send("What else would you like to have for dinner tonight?");
        }
        else{
            // New order
            // Using the conversationData to store the orders
            session.conversationData.orders = new Array();
            session.conversationData.orders.push({ 
                Description: "Check out",
                Price: 0
            })
        }
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else {
                var order = dinnerMenu[results.response.entity];
                session.conversationData.orders[0].Price += order.Price; // Add to total.
                var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
                session.send(msg);
                session.conversationData.orders.push(order);
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu
            }
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
);
```

이 예제에서 주문은 현재 대화 `session.conversationData.orders`로 범위가 지정된 봇 데이터 저장소에 저장됩니다. 모든 새 주문에 대해, 이 변수가 새 배열로 다시 초기화되고, 사용자가 선택한 모든 항목에 대해, 봇은 `orders` 배열에 해당 항목을 추가하고 합계에 가격을 추가합니다. 그러면 해당 값이 체크 아웃의 `Price` 변수에 저장됩니다. 사용자가 주문할 항목 선택을 마치면, "check out"이라고 말하고 나머지 주문 프로세스를 계속 진행할 수 있습니다.

> [!NOTE]
> 봇 데이터 저장소에 대한 자세한 내용은 [상태 데이터 관리](bot-builder-nodejs-state.md)를 참조하세요. 

마지막으로, `orderDinner` 다이얼로그 내의 두 번째 폭포형 단계를 업데이트하여 주문 처리를 확인합니다. 

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner and have it delivered to their room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        session.beginDialog("addDinnerItem");
    },
    function (session, results) {
        if (results.response) {
            // Display itemize order with price total.
            for(var i = 1; i < session.conversationData.orders.length; i++){
                session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
            }
            session.send(`Your total is: $${session.conversationData.orders[0].Price}`);

            // Continue with the check out process.
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}`;
            session.send(msg);
            session.replaceDialog("mainMenu");
        }
    }
])
//...attached triggers...
;
```

## <a name="cancel-a-dialog"></a>다이얼로그 취소

`session.replaceDialog` 메서드를 사용하여 *현재* 다이얼로그를 새 다이얼로그로 바꿀 수 있지만, 다이얼로그 스택에서 더 아래에 있는 다이이얼로그와 바꿀 수는 없습니다. 다이얼로그 스택 내에서 *현재* 다이얼로그가 아닌 다이얼로그를 바꾸려면 [`session.cancelDialog`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#canceldialog) 메서드를 대신 사용합니다. 

다이얼로그 스택에서의 위치에 관계없이 다이얼로그를 종료하고, 필요에 따라 해당 위치에서 새 다이얼로그를 호출하려면 `session.cancelDialog` 메서드를 사용할 수 있습니다. `session.cancelDialog` 메서드를 호출하려면 취소할 다이얼로그의 ID를 지정하고, 필요에 따라 해당 위치에서 호출할 다이얼로그의 ID를 지정합니다. 예를 들어, 이 코드 조각은 `orderDinner` 다이얼로그를 취소하고 `mainMenu` 다이얼로그와 바꿉니다.

```javascript
session.cancelDialog('orderDinner', 'mainMenu'); 
```

`session.cancelDialog` 메서드가 호출되면 다이얼로그 스택이 거꾸로 검색되고, 해당 다이얼로그가 처음 나오는 경우는 취소되므로 해당 다이얼로그와 자식 다이얼로그(있는 경우)는 다이얼로그 스택에서 제거됩니다. 그런 후에 호출 다이얼로그로 제어 권한이 반환됩니다. 그러면 이 다이얼로그는 [`ResumeReason.notCompleted`](http://docs.botframework.com/en-us/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html#notcompleted)와 같은 [`results.resumed`](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed) 코드를 확인하여 취소를 감지할 수 있습니다.

`session.cancelDialog` 메서드를 호출할 때 취소할 다이얼로그의 ID를 지정하는 대신, 취소할 다이얼로그의 0부터 시작되는 인덱스를 지정할 수 있습니다. 이 인덱스는 다이얼로그 스택에서 다이얼로그의 위치를 나타냅니다. 예를 들어, 다음 코드 조각은 현재 활성 상태인 다이얼로그(인덱스 = 0)를 종료하고 해당 위치에서 `mainMenu` 다이얼로그를 시작합니다. `mainMenu` 다이얼로그는 다이얼로그 스택의 0 위치에서 호출되므로 새로운 기본 다이얼로그가 됩니다.

```javascript
session.cancelDialog(0, 'mainMenu');
```

위의 [다이얼로그 루프](#dialog-loops)에서 논의된 샘플을 고려해보세요. 사용자가 항목 선택 메뉴에 도달하면 해당 다이얼로그(`addDinnerItem`)는 다이얼로그 스택 `[default dialog, mainMenu, orderDinner, addDinnerItem]`의 네 번째 다이얼로그가 됩니다. `addDinnerItem` 다이얼로그 내에서 사용자가 주문을 취소할 수 있도록 하려면 어떻게 해야 하나요? `cancelAction` 트리거를 `addDinnerItem` 다이얼로그에 연결하면 사용자가 다시 이전 다이얼로그(`orderDinner`)로 반환되며, 여기서 사용자는 `addDinnerItem` 다이얼로그로 바로 보내집니다.

바로 이때문에 `session.cancelDialog` 메서드가 유용한 것입니다. [다이얼로그 루프 예제](#dialog-loops)부터 “Cancel order”를 저녁 메뉴 내의 명시적 옵션으로 추가합니다.

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0      // Order total. Updated as items are added to order.
    },
    "Cancel order": { // Cancel the order and back to Main Menu
        Description: "Cancel order",
        Price: 0
    }
};
```

그런 다음, "cancel order" 요청을 확인하도록 `addDinnerItem` 다이얼로그를 업데이트합니다. "취소"가 감지되면 `session.cancelDialog` 메서드를 사용하여 기본 다이얼로그(즉, 스택의 인덱스 0에 있는 다이얼로그)를 취소하고 해당 위치에 `mainMenu` 다이얼로그를 호출합니다. 

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else if(results.response.entity.match(/^cancel/i)){
                // Cancel the order and start "mainMenu" dialog.
                session.cancelDialog(0, "mainMenu");
            }
            else {
                //...add item to list and prompt again...
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu.
            }
        }
    }
])
//...attached triggers...
;
```

이러한 방식으로 `session.cancelDialog` 메서드를 사용하여 봇이 요구하는 대화 흐름을 구현할 수 있습니다.

## <a name="next-steps"></a>다음 단계

살펴본 것처럼, 스택의 다이얼로그를 바꾸려면 다양한 형식의 **작업**을 사용하여 해당 태스크를 완료합니다. 작업은 대화 흐름을 관리할 때 유연성을 높여줍니다. **작업** 및 사용자 작업을 더 잘 처리하는 방법을 알아보겠습니다.

> [!div class="nextstepaction"]
> [사용자 작업 처리](bot-builder-nodejs-dialog-actions.md)
