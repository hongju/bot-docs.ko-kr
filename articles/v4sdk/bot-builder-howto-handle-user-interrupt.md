---
title: 사용자 중단 처리 | Microsoft Docs
description: 사용자 중단 및 직접 대화 흐름을 처리하는 방법을 알아봅니다.
keywords: 인터럽트, 중단, 토픽 전환, 중단하기기
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/18/2019
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9089334823c1c57c8ace48531c767c3f966b3355
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905016"
---
# <a name="handle-user-interruptions"></a>사용자 중단 처리

[!INCLUDE[applies-to](../includes/applies-to.md)]

중단을 처리하는 작업은 강력한 봇의 중요한 측면입니다.

사용자가 정의된 대화 흐름을 단계별로 따를 것이라고 생각할 수 있지만 마음을 바꾸거나, 질문에 답변하는 대신 프로세스 중간에 질문을 던질 가능성이 얼마든지 있습니다. 이러한 상황이라면 봇은 사용자 입력을 어떻게 처리할까요? 사용자 환경은 어떻게 될까요? 사용자 상태 데이터는 어떻게 유지할 수 있나요? 중단을 처리하는 작업은 봇이 다음과 같은 상황을 처리하도록 준비하는 것입니다.

각 상황이 봇이 처리하도록 디자인된 시나리오에 고유하므로 이러한 질문에 대한 정답은 없습니다. 이 항목에서는 사용자 중단을 처리하는 몇 가지 일반적인 방법을 알아보고 봇에서 중단 처리를 구현하는 몇 가지 방법을 제안합니다.

## <a name="handle-expected-interruptions"></a>예상된 중단 처리

절차적 대화 흐름에는 사용자를 안내하려는 핵심적인 단계 집합이 있으며 이러한 단계를 벗어하는 사용자 작업은 중단을 가져올 수 있습니다. 일반적인 흐름에는 예상할 수 있는 중단이 있습니다.

**테이블 예약** 테이블 예약 봇에서 핵심 단계는 사용자에게 날짜와 시간, 파티 규모 및 예약 이름을 물어보는 것일 수 있습니다. 해당 프로세스에서 예상할 수 있는 몇 가지 예상되는 중단에는 다음이 포함될 수 있습니다.

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

**DialogSet**의 하위 클래스로 설정된 대화 상자를 정의하겠습니다.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class OrderDinnerDialogs : DialogSet
{
    public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }
}
```

메뉴를 설명하는 몇 가지 내부 클래스를 정의하겠습니다.

```cs
/// <summary>
/// Contains information about an item on the menu.
/// </summary>
public class DinnerItem
{
    public string Description { get; set; }

    public double Price { get; set; }
}

/// <summary>
/// Describes the dinner menu, including the items on the menu and options for
/// interrupting the ordering process.
/// </summary>
public class DinnerMenu
{
    /// <summary>Gets the items on the menu.</summary>
    public static Dictionary<string, DinnerItem> MenuItems { get; } = new Dictionary<string, DinnerItem>
    {
        ["Potato salad"] = new DinnerItem { Description = "Potato Salad", Price = 5.99 },
        ["Tuna sandwich"] = new DinnerItem { Description = "Tuna Sandwich", Price = 6.89 },
        ["Clam chowder"] = new DinnerItem { Description = "Clam Chowder", Price = 4.50 },
    };

    /// <summary>Gets all the "interruptions" the bot knows how to process.</summary>
    public static List<string> Interrupts { get; } = new List<string>
    {
        "More info", "Process order", "Help", "Cancel",
    };

    /// <summary>Gets all of the valid inputs a user can make.</summary>
    public static List<string> Choices { get; }
        = MenuItems.Select(c => c.Key).Concat(Interrupts).ToList();
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

기본 EchoBot 템플릿에서 시작하겠습니다. 지침은 [JavaScript용 빠른 시작](~/javascript/bot-builder-javascript-quickstart.md)을 참조하세요.

`botbuilder-dialogs` 라이브러리는 NPM에서 다운로드할 수 있습니다. `botbuilder-dialogs` 라이브러리를 설치하려면 다음 npm 명령을 실행합니다.

```cmd
npm install --save botbuilder-dialogs
```

**bot.js** 파일에서 참조할 클래스를 참조하고 대화 상자에 사용할 식별자를 정의하겠습니다.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, ChoicePrompt, WaterfallDialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Name for the dialog state property accessor.
const DIALOG_STATE_PROPERTY = 'dialogStateProperty';

// Name of the order-prompt dialog.
const ORDER_PROMPT = 'orderingDialog';

// Name for the choice prompt for use in the dialog.
const CHOICE_PROMPT = 'choicePrompt';
```

순서 지정 대화 상자에 표시할 옵션을 정의합니다.

```javascript
// The options on the dinner menu, including commands for the bot.
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel", "More info", "Help"],
    "Potato Salad - $5.99": {
        description: "Potato Salad",
        price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        description: "Tuna Sandwich",
        price: 6.89
    },
    "Clam Chowder - $4.50": {
        description: "Clam Chowder",
        price: 4.50
    }
}
```

---

주문 논리에서 문자열 일치 또는 정규식을 사용하여 이러한 메뉴 항목을 확인할 수 있습니다.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

먼저 주문을 추적하는 도우미를 정의해야 합니다.

```cs
/// <summary>Helper class for storing the order.</summary>
public class Order
{
    public double Total { get; set; } = 0.0;

    public List<DinnerItem> Items { get; set; } = new List<DinnerItem>();

    public bool ReadyToProcess { get; set; } = false;

    public bool OrderProcessed { get; set; } = false;
}
```

필요한 ID를 추적하는 일부 상수를 추가합니다.

```csharp
/// <summary>The ID of the top-level dialog.</summary>
public const string MainDialogId = "mainMenu";

/// <summary>The ID of the choice prompt.</summary>
public const string ChoicePromptId = "choicePrompt";

/// <summary>The ID of the order card value, tracked inside the dialog.</summary>
public const string OrderCartId = "orderCart";
```

선택 프롬프트 및 폭포 대화 상자를 추가하는 생성자를 업데이트합니다. 또한 폭포 단계를 구현하는 방법을 정의하겠습니다.

```cs
public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
    : base(dialogStateAccessor)
{
    // Add a choice prompt for the dialog.
    Add(new ChoicePrompt(ChoicePromptId));

    // Define and add the main waterfall dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptUserAsync,
        ProcessInputAsync,
    };

    Add(new WaterfallDialog(MainDialogId, steps));
}

/// <summary>
/// Defines the first step of the main dialog, which is to ask for input from the user.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> PromptUserAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Initialize order, continuing any order that was passed in.
    Order order = (stepContext.Options is Order oldCart && oldCart != null)
        ? new Order
        {
            Items = new List<DinnerItem>(oldCart.Items),
            Total = oldCart.Total,
            ReadyToProcess = oldCart.ReadyToProcess,
            OrderProcessed = oldCart.OrderProcessed,
        }
        : new Order();

    // Set the order cart in dialog state.
    stepContext.Values[OrderCartId] = order;

    // Prompt the user.
    return await stepContext.PromptAsync(
        "choicePrompt",
        new PromptOptions
        {
            Prompt = MessageFactory.Text("What would you like for dinner?"),
            RetryPrompt = MessageFactory.Text(
                "I'm sorry, I didn't understand that. What would you like for dinner?"),
            Choices = ChoiceFactory.ToChoices(DinnerMenu.Choices),
        },
        cancellationToken);
}

/// <summary>
/// Defines the second step of the main dialog, which is to process the user's input, and
/// repeat or exit as appropriate.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> ProcessInputAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the order cart from dialog state.
    Order order = stepContext.Values[OrderCartId] as Order;

    // Get the user's choice from the previous prompt.
    string response = (stepContext.Result as FoundChoice).Value;

    if (response.Equals("process order", StringComparison.InvariantCultureIgnoreCase))
    {
        order.ReadyToProcess = true;

        await stepContext.Context.SendActivityAsync(
            "Your order is on it's way!",
            cancellationToken: cancellationToken);

        // In production, you may want to store something more helpful.
        // "Process" the order and exit.
        order.OrderProcessed = true;
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
    {
        // Cancel the order.
        await stepContext.Context.SendActivityAsync(
            "Your order has been canceled",
            cancellationToken: cancellationToken);

        // Exit without processing the order.
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("more info", StringComparison.InvariantCultureIgnoreCase))
    {
        // Send more information about the options.
        string message = "More info: <br/>" +
            "Potato Salad: contains 330 calories per serving. Cost: 5.99 <br/>"
            + "Tuna Sandwich: contains 700 calories per serving. Cost: 6.89 <br/>"
            + "Clam Chowder: contains 650 calories per serving. Cost: 4.50";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else if (response.Equals("help", StringComparison.InvariantCultureIgnoreCase))
    {
        // Provide help information.
        string message = "To make an order, add as many items to your cart as " +
            "you like. Choose the `Process order` to check out. " +
            "Choose `Cancel` to cancel your order and exit.";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }

    // We've checked for expected interruptions. Check for a valid item choice.
    if (!DinnerMenu.MenuItems.ContainsKey(response))
    {
        await stepContext.Context.SendActivityAsync("Sorry, that is not a valid item. " +
            "Please pick one from the menu.");

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else
    {
        // Add the item to cart.
        DinnerItem item = DinnerMenu.MenuItems[response];
        order.Items.Add(item);
        order.Total += item.Price;

        // Acknowledge the input.
        await stepContext.Context.SendActivityAsync(
            $"Added `{response}` to your order; your total is ${order.Total:0.00}.",
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

봇 생성자에서 대화 상자를 정의합니다. 코드는 _먼저_ 중단을 확인하고 처리한 다음, 다음 논리적 단계를 진행합니다.

```javascript
constructor(conversationState) {
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.conversationState = conversationState;

    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new ChoicePrompt(CHOICE_PROMPT))
        .add(new WaterfallDialog(ORDER_PROMPT, [
            async (step) => {
                if (step.options && step.options.orders) {
                    // If an order cart was passed in, continue to use it.
                    step.values.orderCart = step.options;
                } else {
                    // Otherwise, start a new cart.
                    step.values.orderCart = {
                        orders: [],
                        total: 0
                    };
                }
                return await step.prompt(CHOICE_PROMPT, "What would you like?", dinnerMenu.choices);
            },
            async (step) => {
                const choice = step.result;
                if (choice.value.match(/process order/ig)) {
                    if (step.values.orderCart.orders.length > 0) {
                        // If the cart is not empty, process the order by returning the order to the parent context.
                        await step.context.sendActivity("Your order has been processed.");
                        return await step.endDialog(step.values.orderCart);
                    } else {
                        // Otherwise, prompt again.
                        await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                        return await step.replaceDialog(ORDER_PROMPT);
                    }
                } else if (choice.value.match(/cancel/ig)) {
                    // Cancel the order, and return "cancel" to the parent context.
                    await step.context.sendActivity("Your order has been canceled.");
                    return await step.endDialog("cancelled");
                } else if (choice.value.match(/more info/ig)) {
                    // Provide more information, and then continue the ordering process.
                    var msg = "More info: <br/>Potato Salad: contains 330 calories per serving. <br/>"
                        + "Tuna Sandwich: contains 700 calories per serving. <br/>"
                        + "Clam Chowder: contains 650 calories per serving."
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else if (choice.value.match(/help/ig)) {
                    // Provide help information, and then continue the ordering process.
                    var msg = `Help: <br/>`
                        + `To make an order, add as many items to your cart as you like then choose `
                        + `the "Process order" option to check out.`
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else {
                    // The user has chosen a food item from the menu. Add the item to cart.
                    var item = dinnerMenu[choice.value];
                    step.values.orderCart.orders.push(item.description);
                    step.values.orderCart.total += item.price;

                    await step.context.sendActivity(`Added ${item.description} to your cart. <br/>`
                        + `Current total: $${step.values.orderCart.total}`);

                    // Continue the ordering process.
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                }
            }
        ]));
}
```

대화 상자를 호출하고 순서 지정 프로세스의 결과를 표시하는 처리기를 차례로 업데이트합니다.

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        let dc = await this.dialogs.createContext(turnContext);
        let dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            // The dialog completed this turn.
            const result = dialogTurnResult.result;
            if (!result || result === "cancelled") {
                await turnContext.sendActivity('You cancelled your order.');
            } else {
                await turnContext.sendActivity(`Your order came to $${result.total}`);
            }
        } else if (!turnContext.responded) {
            // No dialog was active.
            await turnContext.sendActivity("Let's order dinner...");
            await dc.cancelAllDialogs();
            await dc.beginDialog(ORDER_PROMPT);
        } else {
            // The dialog is active.
        }
    } else {
        await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
    }
    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
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

범위를 벗어난 중단의 경우, 사용자 의도를 추측해볼 수 있습니다. QnA Maker, LUIS 또는 사용자 지정 논리와 같은 AI 서비스를 사용하여 사용자 의도를 추측한 다음, 사용자가 원할 것으로 봇이 생각하는 작업에 대한 제안을 제공할 수 있습니다.

예를 들어, 예약 테이블 흐름의 중간에 사용자는 "I want to order a burger"라고 말할 수 있습니다. 봇은 이 대화 흐름에서 처리 방법을 알지 못합니다. 현재 흐름은 주문과 관련이 없으며 봇의 다른 대화 명령은 “order dinner”이므로 봇은 이 입력에 대해 어떻게 할지 알지 못합니다. 예를 들어, LUIS를 적용하는 경우 모델을 학습하여 음식을 주문하려고 한다는 사실을 인식할 수 있습니다(예: LUIS에서 "orderFood" 의도를 반환할 수 있음). 따라서 봇은 "It seems you want to order food. Would you like to switch to our order dinner process instead?"로 응답할 수 있습니다. LUIS를 학습하고 사용자 의도를 검색하는 방법에 대한 자세한 내용은 [언어 이해를 위한 LUIS 사용](bot-builder-howto-v4-luis.md)을 참조하세요.

### <a name="default-response"></a>기본 응답

모든 작업이 실패할 경우 아무 작업도 수행하지 않고 사용자가 무엇을 해야 할지 고민하는 상황을 만들지 말고, 기본 응답을 전송해야 합니다. 기본 응답은 사용자가 다시 정상적인 프로세스를 진행할 수 있도록 봇이 이해하는 명령을 사용자에게 알려주어야 합니다.

봇 논리 끝에서 컨텍스트 **responded** 플래그를 확인하여 봇이 해당 차례 동안 사용자로 응답을 다시 전송했는지 검토할 수 있습니다. 봇이 사용자 입력을 처리하지만 응답하지 않은 경우, 입력으로 무슨 작업을 해야 할지 잘 모를 확률이 높습니다. 이러한 경우 해당 상황인지 파악하고 사용자에게 기본 메시지를 보낼 수 있습니다.

사용자에게 `mainMenu` 대화를 제공하는 것이 봇의 기본값일 경우 이 상황에서 사용자가 경험하게 될 봇 환경을 결정해야 합니다.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check whether we replied. If not then clear the dialog stack and present the main menu.
if (!turnContext.Responded)
{
    await dc.CancelAllDialogsAsync(cancellationToken);
    await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.cancelAllDialogs();
    await step.beginDialog('mainMenu');
}
```

---

## <a name="handling-global-interruptions"></a>글로벌 중단 처리

위의 예제에서는 특정 대화 상자에서 특정 순서로 발생할 수 있는 중단을 처리합니다. 언제든지 발생할 수 있는 글로벌 중단을 처리하려고 합니다.

봇의 기본 처리기에서 들어오는 `turnContext`를 처리하는 중단 처리 함수를 배치하여 수행하고 수행할 작업을 결정할 수 있습니다.

아래 예제에서 봇이 _먼저 수행할 작업_은 사용자가 도움이 필요하거나 취소하려는 기호에 대해 들어오는 메시지 텍스트를 확인하는 것입니다. 이 두 가지는 봇에서 발생할 수 있는 매우 일반적인 중단입니다. 이 검사를 완료하면 봇은 `dc.continueDialog()`를 호출하여 보류 중인 활성 대화 상자를 처리합니다.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check for top-level interruptions.
string utterance = turnContext.Activity.Text.Trim().ToLowerInvariant();

if (utterance == "help")
{
    // Start a general help dialog. Dialogs already on the stack remain and will continue
    // normally if the help dialog exits normally.
    await dc.BeginDialogAsync(OrderDinnerDialogs.HelpDialogId, null, cancellationToken);
}
else if (utterance == "cancel")
{
    // Cancel any dialog on the stack.
    await turnContext.SendActivityAsync("Canceled.", cancellationToken: cancellationToken);
    await dc.CancelAllDialogsAsync(cancellationToken);
}
else
{
    await dc.ContinueDialogAsync(cancellationToken);

    // Check whether we replied. If not then clear the dialog stack and present the main menu.
    if (!turnContext.Responded)
    {
        await dc.CancelAllDialogsAsync(cancellationToken);
        await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

대화 상자에 사용자 응답을 보내기 전에 중단을 처리할 수 있습니다.

이 코드 조각은 대화 상자에 `helpDialog` 및 `mainMenu`가 설정되었다고 가정합니다.

```javascript
const utterance = (context.activity.text || '').trim();

// Let's look for some interruptions first!
if (utterance.match(/help/ig)) {
    // Launch a new help dialog if the user asked for help
    await dc.beginDialog('helpDialog');
} else if (utterance.match(/cancel/ig)) {
    // Cancel any active dialogs if the user says cancel
    await dc.context.sendActivity('Canceled.');
    await dc.cancelAllDialogs();
}

// If the bot hasn't yet responded...
if (!context.responded) {
    // Continue any active dialog, which might send a response...
    await dc.continueDialog();

    // Finally, if the bot still hasn't sent a response, send instructions.
    if (!context.responded) {
        await dc.cancelAllDialogs();
        await context.sendActivity("Starting the main menu...");
        await dc.beginDialog('mainMenu');
    }
}
```

---
