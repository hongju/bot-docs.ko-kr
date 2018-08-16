---
title: 사용자 중단 처리 | Microsoft Docs
description: 사용자 중단 및 직접 대화 흐름을 처리하는 방법을 알아봅니다.
keywords: 인터럽트, 중단, 토픽 전환, 중단하기기
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/17/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 651a7410893f7a66f5941121edc7b34055807ba7
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301262"
---
# <a name="handle-user-interrupt"></a>사용자 중단 처리

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

사용자 중단을 처리하는 작업은 강력한 봇의 중요한 측면입니다. 사용자가 정의된 대화 흐름을 단계별로 따를 것이라고 생각할 수 있지만 마음을 바꾸거나, 질문에 답변하는 대신 프로세스 중간에 질문을 던질 가능성이 얼마든지 있습니다. 이러한 상황이라면 봇은 사용자 입력을 어떻게 처리할까요? 사용자 환경은 어떻게 될까요? 사용자 상태 데이터는 어떻게 유지할 수 있나요?

각 상황이 봇이 처리하도록 디자인된 시나리오에 고유하므로 이러한 질문에 대한 정답은 없습니다. 이 항목에서는 사용자 중단을 처리하는 몇 가지 일반적인 방법을 알아보고 봇에서 중단 처리를 구현하는 몇 가지 방법을 제안합니다.

## <a name="handle-expected-interruptions"></a>예상된 중단 처리

절차적 대화 흐름에는 사용자를 안내하려는 핵심적인 단계 집합이 있으며 이러한 단계를 벗어하는 사용자 작업은 중단을 가져올 수 있습니다. 일반적인 흐름에는 예상할 수 있는 중단이 있습니다.

**테이블 예약** 테이블 예약 봇에서는 핵심 단계가 사용자에게 날짜와 시간, 파티 규모 및 예약 이름을 물어보는 것일 수 있습니다. 해당 프로세스에서 예상할 수 있는 몇 가지 예상되는 중단에는 다음이 포함될 수 있습니다. 
 * `cancel`: 프로세스를 종료합니다.
 * `help`: 이 프로세스에 대한 추가 지침을 제공합니다.
 * `more info`: 힌트 및 제안을 제공하거나 테이블을 예약하는 다른 방법(예: 연락할 이메일 주소 또는 전화 번호)을 제공합니다.
 * `show list of available tables`: 옵션으로 제공되는 경우 사용자가 원하는 날짜 및 시간에 사용 가능한 테이블 목록이 표시됩니다.

**저녁 주문** 저녁 주문 봇에서 핵심 단계는 메뉴 항목 목록을 제공하고 사용자가 장바구니에 항목을 추가할 수 있도록 하는 것입니다. 이 프로세스에서 예상할 수 있는 몇 가지 예상되는 중단에는 다음이 포함될 수 있습니다. 
 * `cancel`: 주문 프로세스를 종료합니다.
 * `more info`: 각 메뉴 항목에 대한 음식 세부 정보를 제공합니다.
 * `help`: 시스템 사용 방법에 대한 도움말을 제공합니다.
 * `process order`: 주문을 처리합니다.

사용자가 보낼 수 있는 봇 인식 가능 명령을 적어도 알 수 있도록 이러한 내용은 **제안된 작업** 목록 또는 힌트로 제공할 수 있습니다.

예를 들어 저녁 주문 흐름에서 메뉴 항목과 함께 예상되는 중단을 제공할 수 있습니다. 이 경우 메뉴 항목은 `choices`의 배열로 전송됩니다.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
public class dinnerItem
{
    public string Description;
    public double Price;
}

public class dinnerMenu
{
    static public Dictionary<string, dinnerItem> dinnerChoices = new Dictionary<string, dinnerItem>
    {
        { "potato salad", new dinnerItem { Description="Potato Salad", Price=5.99 } },
        { "tuna sandwich", new dinnerItem { Description="Tuna Sandwich", Price=6.89 } },
        { "clam chowder", new dinnerItem { Description="Clam Chowder", Price=4.50 } }
    };

    static public string[] choices = new string[] {"Potato Salad", "Tuna Sandwich", "Clam Chowder", "more info", "Process order", "help", "Cancel"};
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
            "more info", "Process order", "Cancel"],
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50": {
        Description: "Clam Chowder",
        Price: 4.50
    }
}
```

---

주문 논리에서 문자열 일치 또는 정규식을 사용하여 이러한 메뉴 항목을 확인할 수 있습니다.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

먼저 주문을 추적하는 도우미를 정의해야 합니다.

```cs
// Helper class for storing the order in the dictionary
public class Orders
{
    public double total;
    public string order;
    public bool processOrder;

    // Initialize order values
    public Orders()
    {
        total = 0;
        order = "";
        processOrder = false;
    }
}
```

그런 후 봇에 다이얼로그를 추가합니다.

```cs
dialogs.Add("orderPrompt", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt the user
        await dc.Prompt("choicePrompt",
            "What would you like for dinner?",
            new ChoicePromptOptions
            {
                Choices = dinnerMenu.choices.Select( s => new Choice { Value = s }).ToList(),
                RetryPromptString = "I'm sorry, I didn't understand that. What would you " +
                    "like for dinner?"
            });
    },
    async(dc, args, next) =>
    {
        var convo = ConversationState<Dictionary<string,object>>.Get(dc.Context);

        // Get the user's choice from the previous prompt
        var response = (args["Value"] as FoundChoice).Value.ToLower();

        if(response == "process order")
        {
            try 
            {
                var order = convo["order"];

                await dc.Context.SendActivity("Order is on it's way!");
                
                // In production, you may want to store something more helpful, 
                // such as send order off to be made
                (order as Orders).processOrder = true;

                // Once it's submitted, clear the current order
                convo.Remove("order");
                await dc.End();
            }
            catch
            {
                await dc.Context.SendActivity("Your order is empty, please add your order choice");
                // Ask again
                await dc.Replace("orderPrompt");
            }
        }
        else if(response == "cancel" )
        {
            // Get rid of current order
            convo.Remove("order");
            await dc.Context.SendActivity("Your order has been canceled");
            await dc.End();
        }
        else if(response == "more info")
        {
            // Send more information about the options
            var msg = "More info: <br/>" +
                "Potato Salad: contains 330 calaries per serving. Cost: 5.99 <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. Cost: 6.89 <br/>"
                + "Clam Chowder: contains 650 calaries per serving. Cost: 4.50";
            await dc.Context.SendActivity(msg);

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else if(response == "help")
        {
            // Provide help information
            await dc.Context.SendActivity("To make an order, add as many items to your cart as " +
                "you like then choose the \"Process order\" option to check out.");

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else
        {
            // Unlikely to get past the prompt verification, but this will catch 
            // anything that isn't a valid menu choice
            if(!dinnerMenu.dinnerChoices.ContainsKey(response))
            {
                await dc.Context.SendActivity("Sorry, that is not a valid item. " +
                    "Please pick one from the menu.");
    
                // Ask again
                await dc.Replace("orderPrompt");
            }
            else {
                // Add the item to cart
                Orders currentOrder;

                // If there is a current order going, add to it. If not, start a new one
                try
                {
                    currentOrder = convo["order"] as Orders;
                }
                catch
                {
                    convo["order"] = new Orders();
                    currentOrder = convo["order"] as Orders;
                }

                // Add to the current order
                currentOrder.order += (dinnerMenu.dinnerChoices[$"{response}"].Description) + ", ";
                currentOrder.total += (double)dinnerMenu.dinnerChoices[$"{response}"].Price;

                // Save back to the conversation state
                convo["order"] = currentOrder;

                await dc.Context.SendActivity($"Added to cart. Current order: " +
                    $"{currentOrder.order} " +
                    $"<br/>Current total: ${currentOrder.total}");

                // Ask again to allow user to add more items or process order
                await dc.Replace("orderPrompt");
            }
        }
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                dc.context.activity.conversation.dinnerOrder = orderCart;

                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            orderCart.clear(context);
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else if(choice.value.match(/more info/ig)){
            var msg = "More info: <br/>Potato Salad: contains 330 calaries per serving. <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. <br/>" 
                + "Clam Chowder: contains 650 calaries per serving."
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else if(choice.value.match(/help/ig)){
            var msg = `Help: <br/>To make an order, add as many items to your cart as you like then choose the "Process order" option to check out.`
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else {
            var choice = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!choice){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                orderCart.orders.push(choice);
                orderCart.total += dinnerMenu[choice.value].Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt');
            }
        }
    }
]);
```

---

## <a name="handle-unexpected-interruptions"></a>예기치 않은 중단 처리

디자인된 봇 작업 범위를 벗어나는 중단이 있습니다.
모든 중단을 예상할 수는 없지만 봇이 처리하도록 프로그래밍할 수 있는 중단 패턴이 있습니다.

### <a name="switching-topic-of-conversations"></a>대화의 주제 전환
하나의 대화가 진행되는 중간에 다른 대화로 전환하려면 어떻게 할까요? 예를 들어, 봇으로 테이블을 예약하고 저녁을 주문할 수 있습니다.
사용자는 _테이블 예약_ 흐름에 있는 동안, "How many people are in your party?" 질문에 답변하는 대신, “order dinner” 메시지를 보냅니다. 이 경우 사용자는 마음이 바뀌었으며 대신, 저녁 주문 대화에 참여하고 싶어진 것입니다. 이러한 중단은 어떻게 처리하나요? 

저녁 주문 흐름으로 주제를 전환하거나, 숫자를 입력해야 한다는 사실을 사용자에게 알려서 스티커를 지정하고 다시 프롬프트할 수 있습니다. 주제를 전환할 수 있도록 허용할 경우 사용자가 중단했던 위치에서 선택할 수 있도록 진행 상태를 저장할지 여부를 선택해야 합니다. 그렇지 않으면 사용자가 다음 번에 테이블을 예약하고 싶을 때 해당 프로세스를 완전히 다시 시작해야 하도록 수집한 정보를 모두 삭제할 수 있습니다. 사용자 상태 데이터를 관리하는 방법에 대한 자세한 내용은 [대화 및 사용자 속성을 사용하여 상태 저장](bot-builder-howto-v4-state.md)을 참조하세요.

### <a name="apply-artificial-intelligence"></a>인공 지능 적용
범위를 벗어난 중단의 경우, 사용자 의도를 추측해볼 수 있습니다. QnAMaker, LUIS 또는 사용자 지정 논리와 같은 AI 서비스를 사용하여 사용자 의도를 추측한 후, 사용자가 원할 것으로 봇이 생각하는 작업에 대한 제안을 제공할 수 있습니다. 

예를 들어, 예약 테이블 흐름의 중간에 사용자는 "I want to order a burger"라고 말할 수 있습니다. 봇은 이 대화 흐름에서 처리 방법을 알지 못합니다. 현재 흐름은 주문과 관련이 없으며 봇의 다른 대화 명령은 “order dinner”이므로 봇은 이 입력에 대해 어떻게 할지 알지 못합니다. 예를 들어, LUIS를 적용하는 경우 모델을 학습하여 음식을 주문하려고 한다는 사실을 인식할 수 있습니다(예: LUIS에서 "orderFood" 의도를 반환할 수 있음). 따라서 봇은 "It seems you want to order food. Would you like to switch to our order dinner process instead?"로 응답할 수 있습니다. LUIS를 학습하고 사용자 의도를 검색하는 방법에 대한 자세한 내용은 [언어 이해를 위한 사용자 LUIS](bot-builder-howto-v4-luis.md)를 참조하세요.

### <a name="default-response"></a>기본 응답
모든 작업이 실패할 경우 아무 작업도 수행하지 않고 사용자가 무엇을 해야 할지 고민하는 상황을 만들지 말고, 제네릭 기본 응답을 전송할 수 있습니다. 기본 응답은 사용자가 다시 정상적인 프로세스를 진행할 수 있도록 봇이 이해하는 명령을 사용자에게 알려주어야 합니다.

봇 논리 끝에서 컨텍스트 **responded** 플래그를 확인하여 봇이 해당 차례 동안 사용자로 응답을 다시 전송했는지 검토할 수 있습니다. 봇이 사용자 입력을 처리하지만 응답하지 않은 경우, 입력으로 무슨 작업을 해야 할지 잘 모를 확률이 높습니다. 이러한 경우 해당 상황인지 파악하고 사용자에게 기본 메시지를 보낼 수 있습니다.

이 봇에 대한 기본 동작은 사용자에게 `mainMenu` 다이얼로그를 제시하는 것입니다. 이러한 상황에서 사용자가 봇에서 어떤 경험을 하게 될지는 봇 디자이너가 결정해야 합니다.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
if(!context.Responded)
{
    await dc.EndAll().Begin("mainMenu");
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.endAll().begin('mainMenu');
}
```

---

