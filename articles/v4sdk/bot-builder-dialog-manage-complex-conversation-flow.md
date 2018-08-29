---
title: 다이얼로그를 사용하여 복잡한 대화 흐름 관리 | Microsoft Docs
description: Node.js용 봇 작성기 SDK에서 다이얼로그를 사용하여 복잡한 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 복잡한 대화 흐름, 반복, 루프, 메뉴, 다이얼로그, 프롬프트, 폭포, 다이얼로그 집합
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 304de6783a268140bf74f95d96cd9c24e12c4c05
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/21/2018
ms.locfileid: "40236367"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>다이얼로그를 사용하여 복잡한 대화 흐름 관리

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

다이얼로그 라이브러리를 사용하여 간단하거나 복잡한 대화 흐름을 모두 관리할 수 있습니다. [간단한 대화 흐름](bot-builder-dialog-manage-conversation-flow.md)의 경우 사용자가 *폭포*의 첫 번째 단계에서 시작하여 마지막 단계까지 계속 진행함으로써 대화형 교환을 완료합니다. 이 문서에서는 다이얼로그를 사용하여 분기하고 반복할 수 있는 부분이 있는 더 복잡한 대화를 관리합니다. 이렇게 하려면 다이얼로그 컨텍스트의 _replace_ 메서드를 사용하고 다이얼로그의 다른 부분 간에 인수를 전달합니다.

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

*다이얼로그 스택*을 자세히 제어하기 위해 **다이얼로그** 라이브러리에서 _replace_ 메서드를 제공합니다. 이 메서드를 사용하면 스택에서 현재 다이얼로그를 팝하고, 새 다이얼로그를 스택의 맨 위로 푸시하고, 새 다이얼로그를 시작할 수 있습니다. 이 기능을 통해 더 복잡한 대화를 제공할 수 있습니다. 이러한 기술은 임의의 복잡한 대화를 만드는 데 사용할 수 있습니다. 대화의 복잡성이 증가하여 다이얼로그를 관리하기 어려워지는 경우 사용자의 대화를 추적하는 사용자 고유의 제어 논리 흐름을 만들 수 있습니다.

<!-- TODO: This is probably a good place to add a link to the modular/composite dialogs topic. -->

이 문서에서는 손님이 테이블을 예약하거나 객실로 배달될 식사를 주문하는 데 사용할 수 있는 호텔 봇용 다이얼로그를 만듭니다. 최상위 다이얼로그는 손님에게 이 두 가지 옵션을 제공합니다. 테이블을 예약하려는 경우 최상위 다이얼로그에서 테이블 예약 다이얼로그를 시작합니다. 손님이 저녁 식사를 주문하려는 경우 최상위 다이얼로그에서 저녁 식사 주문 다이얼로그를 시작합니다. 저녁 식사 주문 다이얼로그에서 먼저 손님에게 메뉴의 음식 항목을 선택하도록 요청한 다음, 객실 번호를 요청합니다. 음식 항목 선택도 다이얼로그이므로 손님이 저녁 식사 주문을 처리하기 전에 여러 항목을 선택하도록 허용할 수 있습니다.

다음 다이어그램에서는 이 문서에서 만드는 다이얼로그와 서로의 관계를 보여 줍니다.

![이 문서에서 사용하는 다이얼로그에 대한 그림](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>분기하는 방법

다이얼로그 컨텍스트는 _다이얼로그 스택_을 유지하고 스택의 각 다이얼로그에 대해 다음 단계를 추적합니다. _begin_ 메서드는 다이얼로그를 스택의 맨 위로 푸시하고, _end_ 메서드는 스택에서 맨 위의 다이얼로그를 팝합니다.

다이얼로그는 다이얼로그 컨텍스트의 _begin_ 메서드를 호출하고 새 다이얼로그의 ID를 제공하여 동일한 다이얼로그 집합 내에서 새 다이얼로그를 시작할 수 있습니다(즉, 분기). 원래 다이얼로그는 여전히 스택에 있지만, 다이얼로그 컨텍스트의 _continue_ 메서드에 대한 호출은 스택 맨 위에 있는 다이얼로그인 _활성 다이얼로그_에만 전송됩니다. 다이얼로그가 스택에서 팝되면 다이얼로그 컨텍스트가 중단된 스택에서 폭포의 다음 단계로 다시 시작됩니다.

따라서 잠재적인 다이얼로그 집합에서 시작하도록 다이얼로그를 조건부로 선택할 수 있는 단계를 하나의 다이얼로그에 포함하여 대화 흐름 내에 분기를 만들 수 있습니다.

## <a name="how-to-loop"></a>반복하는 방법

다이얼로그 컨텍스트의 _replace_ 메서드를 사용하면 스택의 맨 위에 있는 다이얼로그를 대체할 수 있습니다. 이전 다이얼로그의 상태가 삭제되고 새 다이얼로그가 처음부터 시작됩니다. 이 메서드를 사용하여 다이얼로그를 자체로 대체하여 루프를 만들 수 있습니다. 그러나 [상태를 유지](bot-builder-howto-v4-state.md)하려면 _replace_ 메서드를 호출할 때 다이얼로그의 새 인스턴스에 정보를 전달해야 합니다. 다음 예제에서는 저녁 식사 주문 카트가 다이얼로그 상태에 저장되고, `orderPrompt` 다이얼로그가 반복될 때 새 다이얼로그의 상태에서 항목을 계속 추가할 수 있도록 현재 다이얼로그 상태가 전달됩니다. 이러한 정보를 다른 상태에 저장하려면 [사용자 데이터 유지](bot-builder-tutorial-persist-user-inputs.md)를 참조하세요.

## <a name="create-the-dialogs-for-the-hotel-bot"></a>호텔 봇용 다이얼로그 만들기

이 섹션에서는 이제까지 설명한 호텔 봇의 대화를 관리하기 위한 다이얼로그를 만들어 보겠습니다.

### <a name="install-the-dialogs-library"></a>다이얼로그 라이브러리 설치

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

기본 EchoBot 템플릿에서 시작하겠습니다. 지침은 [.NET용 빠른 시작](~/dotnet/bot-builder-dotnet-quickstart.md)을 참조하세요.

다이얼로그를 사용하려면 프로젝트 또는 솔루션에 대한 `Microsoft.Bot.Builder.Dialogs` NuGet 패키지를 설치합니다.
그런 다음, 필요에 따라 코드 파일의 using 문에서 다이얼로그 라이브러리를 참조합니다.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`botbuilder-dialogs` 라이브러리는 NPM에서 다운로드할 수 있습니다. `botbuilder-dialogs` 라이브러리를 설치하려면 다음 NPM 명령을 실행합니다.

```cmd
npm install --save botbuilder-dialogs
```

봇에서 **대화 상자**를 사용하려면 이를 봇 코드에 포함시킵니다. 예를 들어 이 다이얼로그를 **app.js** 파일에 추가합니다.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>다이얼로그 집합 만들기

이 예제에 대한 모든 다이얼로그를 추가할 _다이얼로그 집합_을 만듭니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialog** 클래스를 만들고, 필요한 using 문을 추가합니다.

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
```

**DialogSet**에서 클래스를 파생시키고, 이 다이얼로그 집합에 대한 다이얼로그, 프롬프트 및 상태 정보를 식별하는 데 사용할 ID와 키를 정의합니다.

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialog : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

    /// <summary>Contains the IDs for the other dialogs in the set.</summary>
    private static class Dialogs
    {
        public const string OrderDinner = "orderDinner";
        public const string OrderPrompt = "orderPrompt";
        public const string ReserveTable = "reserveTable";
    }

    /// <summary>Contains the IDs for the prompts used by the dialogs.</summary>
    private static class Inputs
    {
        public const string Choice = "choicePrompt";
        public const string Number = "numberPrompt";
    }

    /// <summary>Contains the keys used to manage dialog state.</summary>
    private static class Outputs
    {
        public const string OrderCart = "orderCart";
        public const string OrderTotal = "orderTotal";
        public const string RoomNumber = "roomNumber";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇에서 **대화 상자**를 사용하려면 이를 봇 코드에 포함시킵니다.

**app.js** 파일에서 라이브러리를 참조하세요.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

그런 다음, 다이얼로그 집합을 만듭니다.

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

---

### <a name="add-the-prompts-to-the-set"></a>집합에 프롬프트 추가

**ChoicePrompt**를 사용하여 손님에게 저녁 식사를 주문할지, 아니면 테이블을 예약할지, 그리고 저녁 식사 메뉴에서 선택할 옵션을 묻습니다. 그리고 저녁 식사를 주문하는 경우 **NumberPrompt**를 사용하여 손님의 객실 번호를 묻습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialog** 생성자에서 두 개의 프롬프트를 추가합니다.

```csharp
// Add the prompts.
this.Add(Inputs.Choice, new ChoicePrompt(Culture.English));
this.Add(Inputs.Number, new NumberPrompt<int>(Culture.English));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

이 두 프롬프트를 다이얼로그 집합에 추가합니다.

```javascript
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
```

---

### <a name="define-some-of-the-supporting-information"></a>지원 정보 중 일부 정의

저녁 식사 메뉴의 각 옵션에 대한 정보가 필요하므로 지금 이 정보를 설정해 보겠습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

이 정보를 포함할 내부 정적 **Lists** 클래스를 만듭니다. 또한 각 옵션에 대한 정보를 포함하는 내부 **WelcomeChoice** 및 **MenuChoice** 클래스를 만듭니다.

여기서는 옵션 목록에 대한 정보를 최상위 수준의 시작 다이얼로그에 추가하고, 나중에 손님에게 이 정보를 요청할 때 사용할 지원 목록도 만듭니다. 앞부분에 약간의 추가 작업이 필요하지만, 다이얼로그 코드를 더 간단하게 만들겠습니다.

```csharp
/// <summary>Describes an option for the top-level dialog.</summary>
private class WelcomeChoice
{
    /// <summary>The text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>The ID of the associated dialog for this option.</summary>
    public string DialogName { get; set; }
}

/// <summary>Describes an option for the food-selection dialog.</summary>
/// <remarks>We have two types of options. One represents meal items that the guest
/// can add to their order. The other represents a request to process or cancel the
/// order.</remarks>
private class MenuChoice
{
    /// <summary>The request text for cancelling the meal order.</summary>
    public const string Cancel = "Cancel order";

    /// <summary>The request text for processing the meal order.</summary>
    public const string Process = "Process order";

    /// <summary>The name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>The price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>The text to show the guest for this option.</summary>
    public string Description => (double.IsNaN(Price)) ? Name : $"{Name} - ${Price:0.00}";
}

/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>The options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static List<string> WelcomeList { get; } = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the top-level dialog.</summary>
    public static List<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(WelcomeList);

    /// <summary>The reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(WelcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>The options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static List<string> MenuList { get; } = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the food-selection dialog.</summary>
    public static List<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(MenuList);

    /// <summary>The reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(MenuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

이 정보를 포함하는 **dinnerMenu** 상수를 만듭니다.

```javascript
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
        "Process order", "Cancel"],
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

### <a name="create-the-welcome-dialog"></a>시작 다이얼로그 만들기

이 다이얼로그는 `ChoicePrompt`를 사용하여 메뉴를 표시하고, 사용자가 옵션을 선택할 때까지 기다립니다. 사용자가 `Order dinner` 또는 `Reserve a table` 중 하나를 선택하면 적절한 선택을 위한 다이얼로그가 시작되고, 다이얼로그가 완료되면 마지막 단계에서 다이얼로그를 종료하고 사용자가 발생하는 상황에 대해 궁금해 하는 대신 이 `mainMenu` 다이얼로그가 자체적으로 `mainMenu` 다이얼로그를 다시 시작하여 반복됩니다. 이 간단한 구조를 통해 봇에는 항상 메뉴가 표시되고, 사용자는 항상 자신이 선택한 항목을 알 수 있습니다.

**MainMenu** 다이얼로그에는 다음과 같은 폭포 단계가 포함되어 있습니다.

* 폭포의 첫 번째 단계에서는 다이얼로그 상태를 초기화하거나 지우고, 손님을 맞이하고, 사용 가능한 `Order dinner` 또는 `Reserve a table` 옵션 중에서 선택하도록 요청합니다.
* 두 번째 단계에서는 선택 항목을 검색한 다음, 해당 선택 항목과 연결된 자식 다이얼로그를 시작합니다. 자식 다이얼로그가 종료되면 이 다이얼로그가 다음 단계로 다시 시작됩니다.
* 마지막 단계에서는 **DialogContext.Replace** 메서드를 사용하여 이 다이얼로그를 자체의 새 인스턴스로 바꿉니다. 그러면 시작 다이얼로그가 반복 메뉴로 효과적으로 전환됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the main welcome dialog.
this.Add(MainMenu, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Greet the guest and ask them to choose an option.
        await dc.Context.SendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.Prompt(Inputs.Choice, "How may we serve you today?", new ChoicePromptOptions
        {
            Choices = Lists.WelcomeChoices,
            RetryPromptActivity = Lists.WelcomeReprompt,
        });
    },
    async (dc, args, next) =>
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)args["Value"];
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        await dc.Begin(dialogId, dc.ActiveDialog.State);
    },
    async (dc, args, next) =>
    {
        // Start this dialog over again.
        await dc.Replace(MainMenu);
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
dialogs.add('mainMenu', [
    async function(dc){
        await dc.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function(dc, result){
        if(result.value.match(/order dinner/ig)){
            await dc.begin('orderDinner');
        }
        else if(result.value.match(/reserve a table/ig)){
            await dc.begin('reserveTable');
        }
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);
```

---

### <a name="create-the-order-dinner-dialog"></a>저녁 식사 주문 다이얼로그 만들기

저녁 식사 주문(order-dinner) 다이얼로그에서는 손님을 "저녁 식사 주문 서비스"로 맞이하고, 다음 섹션에서 설명하는 음식 선택(food-selection) 다이얼로그를 바로 시작합니다. 중요한 점은 손님이 자신의 주문을 처리하도록 서비스에 요청하면 음식 선택 다이얼로그에서 주문에 대한 항목 목록을 반환한다는 것입니다. 전체 프로세스를 완료하기 위해 이 다이얼로그는 음식을 배달할 객실 번호를 요구한 다음, 종료하기 전에 확인 메시지를 보냅니다. 손님이 주문을 취소하면 이 다이얼로그에서 객실 번호를 묻지 않고 즉시 종료합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

손님의 저녁 식사 주문을 추적하는 데 사용할 수 있는 데이터 구조를 **HotelDialog** 클래스에 추가합니다.

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice> { }
```

**HotelDialog** 생성자에서 저녁 식사 주문 다이얼로그를 추가합니다.

* 첫 번째 단계에서는 시작 메시지를 보내고, 음식 선택 다이얼로그를 시작합니다.
* 두 번째 단계에서는 음식 선택 다이얼로그에서 주문 카트를 반환했는지 여부를 확인합니다.
  * 그렇다면 객실 번호를 묻고 카트 정보를 저장합니다.
  * 그렇지 않은 경우 주문을 취소했다고 가정하고 **DialogContext.End**를 호출하여 이 다이얼로그를 종료합니다.
* 마지막 단계에서는 손님의 객실 번호를 기록하고 종료하기 전에 확인 메시지를 보냅니다. 이 단계는 봇에서 주문을 처리 서비스로 전달합니다.

```csharp
// Add the order-dinner dialog.
this.Add(Dialogs.OrderDinner, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        await dc.Context.SendActivity("Welcome to our Dinner order service.");

        // Start the food selection dialog.
        await dc.Begin(Dialogs.OrderPrompt);
    },
    async (dc, args, next) =>
    {
        if (args.TryGetValue(Outputs.OrderCart, out object arg) && arg is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            dc.ActiveDialog.State[Outputs.OrderCart] = cart;
            await dc.Prompt(Inputs.Number, "What is your room number?", new PromptOptions
            {
                RetryPromptString = "Please enter your room number."
            });
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            await dc.End();
        }
    },
    async (dc, args, next) =>
    {
        // Get and save the guest's answer.
        var roomNumber = args["Text"] as string;
        dc.ActiveDialog.State[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await dc.Context.SendActivity($"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.");
        await dc.End();
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Order dinner:
// Help user order dinner from a menu
dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        
        await dc.begin('orderPrompt', dc.activeDialog.state.orderCart); // Prompt for orders
    },
    async function (dc, result) {
        if(result == "Cancel"){
            await dc.end();
        }
        else { 
            await dc.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function(dc, result){
        await dc.context.sendActivity(`Thank you. Your order will be delivered to room ${result} within 45 minutes.`);
        await dc.end();
    }
]);
```

---

### <a name="create-the-order-prompt-dialog"></a>주문 확인 다이얼로그 만들기

음식 선택 다이얼로그에서는 주문할 수 있는 저녁 식사 항목과 두 가지 주문 처리 요청이 모두 포함된 옵션 목록을 손님에게 제공합니다. 이 다이얼로그는 손님이 봇 프로세스를 선택하거나 주문을 취소할 때까지 반복됩니다.

* 손님이 저녁 식사 항목을 선택하면 해당 _카트에 추가_됩니다.
* 손님이 자신의 주문을 처리하도록 선택하면 먼저 카트가 비어 있는지 확인합니다.
  * 비어 있으면 오류 메시지를 보내고 반복을 계속합니다.
  * 그렇지 않으면 이 다이얼로그를 종료하고 카트 정보를 부모 다이얼로그로 반환합니다.
* 손님이 주문을 취소하도록 선택하면 다이얼로그가 종료되고 카트 정보가 반환되지 않습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialog** 생성자에서 음식 선택 다이얼로그를 추가합니다.

* 첫 번째 단계에서는 다이얼로그 상태를 초기화합니다. 다이얼로그의 입력 인수에 카트 정보가 포함되어 있으면 해당 정보를 다이얼로그 상태에 저장합니다. 그렇지 않으면 빈 카트를 만들어 해당 정보를 추가합니다. 손님이 저녁 식사 메뉴에서 항목을 선택하도록 요청합니다.
* 다음 단계에서는 손님이 선택한 옵션을 살펴봅니다.
  * 주문 처리 요청인 경우 카트에 항목이 포함되어 있는지 확인합니다.
    * 그렇다면 다이얼로그를 종료하고 카트 내용을 반환합니다.
    * 그렇지 않으면 손님에게 오류 메시지를 보내고, 다이얼로그를 처음부터 다시 시작합니다.
  * 주문 취소 요청인 경우 다이얼로그를 종료하고 빈 사전을 반환합니다.
  * 저녁 식사 항목인 경우 해당 항목을 카트에 추가하고, 상태 메시지를 보내고, 다이얼로그를 시작한 다음, 현재 주문 상태를 전달합니다.

```csharp
// Add the food-selection dialog.
this.Add(Dialogs.OrderPrompt, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            if (args is null || !args.ContainsKey(Outputs.OrderCart))
            {
                // First time through, initialize the order state.
                dc.ActiveDialog.State[Outputs.OrderCart] = new OrderCart();
                dc.ActiveDialog.State[Outputs.OrderTotal] = 0.0;
            }
            else
            {
                // Otherwise, set the order state to that of the arguments.
                dc.ActiveDialog.State = new Dictionary<string, object>(args);
            }

            await dc.Prompt(Inputs.Choice, "What would you like?", new ChoicePromptOptions
            {
                Choices = Lists.MenuChoices,
                RetryPromptActivity = Lists.MenuReprompt,
            });
        },
        async (dc, args, next) =>
        {
            // Get the guest's choice.
            var choice = (FoundChoice)args["Value"];
            var option = Lists.MenuOptions[choice.Index];

            // Get the current order from dialog state.
            var cart = (OrderCart)dc.ActiveDialog.State[Outputs.OrderCart];

            if (option.Name is MenuChoice.Process)
            {
                if (cart.Count > 0)
                {
                    // If there are any items in the order, then exit this dialog,
                    // and return the list of selected food items.
                    await dc.End(new Dictionary<string, object>
                    {
                        [Outputs.OrderCart] = cart
                    });
                }
                else
                {
                    // Otherwise, send an error message and restart from
                    // the beginning of this dialog.
                    await dc.Context.SendActivity(
                        "Your cart is empty. Please add at least one item to the cart.");
                    await dc.Replace(Dialogs.OrderPrompt);
                }
            }
            else if (option.Name is MenuChoice.Cancel)
            {
                await dc.Context.SendActivity("Your order has been cancelled.");

                // Exit this dialog, returning an empty property bag.
                dc.ActiveDialog.State.Clear();
                await dc.End(new Dictionary<string, object>());
            }
            else
            {
                // Add the selected food item to the order and update the order total.
                cart.Add(option);
                var total = (double)dc.ActiveDialog.State[Outputs.OrderTotal] + option.Price;
                dc.ActiveDialog.State[Outputs.OrderTotal] = total;

                await dc.Context.SendActivity($"Added {option.Name} (${option.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.");

                // Present the order options again, passing in the current order state.
                await dc.Replace(Dialogs.OrderPrompt, dc.ActiveDialog.State);
            }
        },
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc, orderCart){
        // Define a new cart of one does not exists
        if(!orderCart){
            // Initialize a new cart
            // convoState = conversationState.get(dc.context);
            dc.activeDialog.state.orderCart = {
                orders: [],
                total: 0
            };
        }
        else {
            dc.activeDialog.state.orderCart = orderCart;
        }
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        // Get state object
        // convoState = conversationState.get(dc.context);

        if(choice.value.match(/process order/ig)){
            if(dc.activeDialog.state.orderCart.orders.length > 0) {
                // Process the order
                // ...
                dc.activeDialog.state.orderCart = undefined; // Reset cart
                await dc.context.sendActivity("Processing your order.");
                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            //dc.activeDialog.state.orderCart = undefined; // Reset cart
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!item){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                dc.activeDialog.state.orderCart.orders.push(item);
                dc.activeDialog.state.orderCart.total += item.Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${dc.activeDialog.state.orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt', dc.activeDialog.state.orderCart);
            }
        }
    }
]);
```

위의 샘플 코드에서는 기본 `orderDinner` 다이얼로그가 `orderPrompt`라는 도우미 다이얼로그를 사용하여 사용자 선택을 처리하는 과정을 보여 줍니다. `orderPrompt` 다이얼로그는 음식 항목 메뉴를 표시하고, 사용자에게 항목을 선택하도록 요청하고, 카트에 해당 항목을 추가한 다음, 루프를 통해 다시 확인합니다. 이렇게 하면 사용자가 주문에 여러 품목을 추가할 수 있습니다. 사용자가 `Process order` 또는 `Cancel`을 선택할 때까지 다이얼로그가 반복됩니다. 어떤 시점에서 실행이 부모 다이얼로그(`orderDinner` 다이얼로그)로 다시 전달됩니다. `orderDinner` 다이얼로그는 사용자가 주문을 처리하기 원할 경우 막판 조정 작업을 수행하고, 그렇지 않은 경우 종료되고 실행을 부모 다이얼로그(예: `mainMenu`)로 반환합니다. 그런 후 `mainMenu` 다이얼로그가 기본 메뉴 선택 항목을 다시 표시하는 마지막 단계를 계속 실행합니다.

---
### <a name="create-the-reserve-table-dialog"></a>테이블 예약 다이얼로그 만들기

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

이 예제를 더 짧게 유지하기 위해 여기서는 `reserveTable` 다이얼로그의 최소 구현만 보여 줍니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the table-reservation dialog.
this.Add(Dialogs.ReserveTable, new WaterfallStep[]
    {
        // Replace this waterfall with your reservation steps.
        async (dc, args, next) =>
        {
            await dc.Context.SendActivity("Your table has been reserved.");
            await dc.End();
        }
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(dc, args, next){
        await dc.context.sendActivity("Your table has been reserved");
        await dc.end();
    }
]);
```

---

## <a name="next-steps"></a>다음 단계

이 봇은 "추가 정보" 또는 "도움말"과 같은 다른 옵션을 메뉴 선택 항목에 제공하여 향상시킬 수 있습니다. 이러한 유형의 중단을 구현하는 방법에 대한 자세한 내용은 [사용자 중단 처리](bot-builder-howto-handle-user-interrupt.md)를 참조하세요.

이제 다이얼로그, 프롬프트 및 폭포를 사용하여 복잡한 대화 흐름을 관리하는 방법을 알아보았으므로, 다이얼로그(예: `orderDinner` 및 `reserveTable` 다이얼로그)를 하나의 큰 다이얼로그 집합으로 함께 묶는 대신 개별 개체로 분할하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [복합 컨트롤을 사용하여 모듈식 봇 논리 만들기](bot-builder-compositcontrol.md)
