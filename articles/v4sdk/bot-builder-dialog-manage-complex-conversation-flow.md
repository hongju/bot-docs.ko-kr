---
title: 다이얼로그를 사용하여 복잡한 대화 흐름 관리 | Microsoft Docs
description: Node.js용 봇 작성기 SDK에서 다이얼로그를 사용하여 복잡한 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 복잡한 대화 흐름, 반복, 루프, 메뉴, 다이얼로그, 프롬프트, 폭포, 다이얼로그 집합
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/03/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 871bfd9f8d693c5082fe1ccf38349f4d3d46ece2
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326570"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>다이얼로그를 사용하여 복잡한 대화 흐름 관리

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

마지막 문서에서는 대화 상자 라이브러리를 사용하여 간단한 대화를 관리하는 방법을 보여줍니다. [간단한 대화 흐름](bot-builder-dialog-manage-conversation-flow.md)의 경우 사용자가 *폭포*의 첫 번째 단계에서 시작하여 마지막 단계까지 계속 진행함으로써 대화형 교환을 완료합니다. 이 문서에서는 다이얼로그를 사용하여 분기하고 반복할 수 있는 부분이 있는 더 복잡한 대화를 관리합니다. 이를 위해 대화 컨텍스트 및 폭포 단계 컨텍스트에 정의된 다양한 메서드를 사용하고 대화의 여러 부분 간에 인수를 전달합니다.

대화 상자에 대한 자세한 배경 정보는 [대화 상자 라이브러리](bot-builder-concept-dialog.md)를 참조하세요.

*다이얼로그 스택*을 자세히 제어하기 위해 **다이얼로그** 라이브러리에서 _replace dialog_ 메서드를 제공합니다. 이 방법을 사용하면 대화의 상태 및 흐름을 유지하면서 현재 활성 대화 상자를 다른 대화 상자로 교환할 수 있습니다. _begin dialog_ 및 _replace dialog_ 메서드를 사용하면 필요에 따라 분기하고 반복하여 더 복잡한 상호 작용을 만들 수 있습니다. 대화의 복잡성이 증가하여 폭포 대화 상자를 관리하기 어려워지면 [구성 요소 대화 상자](bot-builder-compositcontrol.md)를 사용하거나 기본 `Dialog` 클래스에 기반을 둔 사용자 지정 대화 상자 관리 클래스를 빌드하여 조사하세요.

이 문서에서는 게스트가 호텔 레스토랑 테이블 예약, 룸 서비스로 식사 주문 등 일반적인 서비스를 이용할 때 사용할 수 있는 호텔 컨시어지 봇을 위한 샘플 대화 상자를 만듭니다.  이러한 각 기능은 기능을 서로 연결하는 메뉴와 함께 대화 상자 집합의 대화 상자로 생성됩니다.

봇의 최상위 대화 상자는 손님에게 이 두 가지 옵션을 제공합니다. 게스트가 테이블을 예약하려고 하면 최상위 대화 상자가 _begin dialog_ 비동기화 메서드를 사용하여 테이블 예약 대화 상자를 시작합니다. 게스트가 룸 서비스를 주문하려는 경우 최상위 대화 상자에서 저녁 식사 주문 대화 상자를 시작합니다.

저녁 식사 주문 대화 상자에서 먼저 게스트에게 메뉴의 음식 항목을 선택하도록 요청한 다음, 객실 번호를 요청합니다. 음식 메뉴 옵션도 _또한_ 대화 상자이고, 게스트가 저녁 식사 주문을 제출하기 전에 메뉴에서 항목을 선택할 때 여러 번 재생되도록 호출됩니다.

다음 다이어그램에서는 이 문서에서 만드는 다이얼로그와 서로의 관계를 보여 줍니다.

![이 문서에서 사용하는 다이얼로그에 대한 그림](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>분기하는 방법

다이얼로그 컨텍스트는 _다이얼로그 스택_을 유지하고 스택의 각 다이얼로그에 대해 다음 단계를 추적합니다. _begin dialog_ 메서드는 대화 상자를 스택의 맨 위로 푸시하고, _end dialog_ 메서드는 스택에서 맨 위의 대화 상자를 삭제합니다.

분기하려면 자식 대화 상자 집합 중 하나를 선택하여 시작합니다. 이 개념에 대한 자세한 내용은 [대화 분기](bot-builder-concept-dialog.md#branch-a-conversation)를 참조하세요.

## <a name="how-to-loop"></a>반복하는 방법

대화 상자 컨텍스트의 _replace dialog_ 메서드를 사용하면 스택의 맨 위에 있는 대화 상자를 대체할 수 있습니다. 이전 다이얼로그의 상태가 삭제되고 새 다이얼로그가 처음부터 시작됩니다. 이 메서드를 사용하여 다이얼로그를 자체로 대체하여 루프를 만들 수 있습니다. 그러나 [봇이 이미 수집한 정보를 보존](bot-builder-howto-v4-state.md)하려면 _replace dialog_ 메서드를 호출할 때 다이얼로그의 새 인스턴스에 정보를 전달해야 합니다.

다음 예제에서는 룸 서비스 주문이 대화 상자 상태에 저장되고, `orderPrompt` 대화 상자가 반복될 때 새 대화 상자의 상태에서 항목을 계속 추가할 수 있도록 현재 대화 상자 상태가 전달됩니다. 이 정보를 대화 상자 외부의 봇 상태에 저장하려면 [사용자 데이터를 보존](bot-builder-tutorial-persist-user-inputs.md)하는 방법을 참조하세요.

## <a name="create-the-dialogs-for-the-hotel-bot"></a>호텔 봇용 다이얼로그 만들기

이 섹션에서는 이제까지 설명한 호텔 봇의 대화를 관리하기 위한 다이얼로그를 만들어 보겠습니다.

### <a name="install-the-dialogs-library"></a>다이얼로그 라이브러리 설치

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

기본 EchoBot 템플릿에서 시작하겠습니다. 지침은 [.NET용 빠른 시작](../dotnet/bot-builder-dotnet-sdk-quickstart.md)을 참조하세요.

다이얼로그를 사용하려면 프로젝트 또는 솔루션에 대한 `Microsoft.Bot.Builder.Dialogs` NuGet 패키지를 설치합니다.
그런 다음, 필요에 따라 코드 파일의 using 문에서 다이얼로그 라이브러리를 참조합니다.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

EchoBot 템플릿으로 시작하겠습니다. 지침은 [JavaScript용 빠른 시작](../javascript/bot-builder-javascript-quickstart.md)을 참조하세요.



`botbuilder-dialogs` 라이브러리는 npm에서 다운로드할 수 있습니다. `botbuilder-dialogs` 라이브러리를 설치하려면 다음 npm 명령을 실행합니다.

```cmd
npm install --save botbuilder-dialogs
```

봇에서 **대화 상자**를 사용하려면 이를 봇 코드에 포함시킵니다. 예를 들어 이 대화 상자를 **index.js** 파일에 추가합니다.

```javascript
const { DialogSet } = require('botbuilder-dialogs');
```

**bot.js** 파일에 다음을 추가합니다.

```javascript
const { DialogSet, NumberPrompt, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>다이얼로그 집합 만들기

이 예제에 대한 모든 다이얼로그를 추가할 _다이얼로그 집합_을 만듭니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialogs** 클래스를 만들고, 필요한 using 문을 추가합니다.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;
using Microsoft.Bot.Schema;
```

**DialogSet**에서 클래스를 파생시킵니다. 대화 상자 집합 인스턴스의 내부 상태를 관리하는 데 사용할 `IStatePropertyAccessor<DialogState>` 매개 변수를 가져오는 생성자를 포함합니다. 또한 이 대화 상자 집합에 대한 대화 상자, 프롬프트 및 상태 정보를 식별하는 데 사용할 ID와 키를 정의합니다.

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialogs : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

    public HotelDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }

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

**index.js** 파일에서 코드를 추가하여 대화 상자 상태를 관리하는 데 사용할 상태 속성 접근자를 만들고, 이를 사용하여 봇에 사용할 대화 상자 집합을 만듭니다.

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const dialogStateAccessor = conversationState.createProperty('dialogState');

// Create a dialog set for the bot.
const dialogSet = new DialogSet(dialogStateAccessor);

// Create the bot.
const bot = new MyBot(conversationState, dialogSet)
```

그런 다음, 봇 개체를 사용하도록 활동 처리 호출을 업데이트합니다.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to the bot's turn handler.
        await bot.onTurn(context);
    });
});
```

---

### <a name="add-the-prompts-to-the-set"></a>집합에 프롬프트 추가

**ChoicePrompt**를 사용하여 손님에게 저녁 식사를 주문할지, 아니면 테이블을 예약할지, 그리고 저녁 식사 메뉴에서 선택할 옵션을 묻습니다. 그리고 저녁 식사를 주문하는 경우 **NumberPrompt**를 사용하여 손님의 객실 번호를 묻습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelDialogs** 생성자에서 두 개의 프롬프트를 추가합니다.

```csharp
// Add the prompts.
Add(new ChoicePrompt(Inputs.Choice));
Add(new NumberPrompt<int>(Inputs.Number));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇 생성자를 업데이트하고 두 개의 프롬프트를 대화 상자 집합에 추가합니다.

```javascript
constructor(conversationState, dialogSet) {
    // Creates a new state accessor property.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
    this.conversationState = conversationState;
    this.dialogSet = dialogSet;

    this.dialogSet.add(new ChoicePrompt('choicePrompt'));
    this.dialogSet.add(new NumberPrompt('numberPrompt'));
}
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
    /// <summary>Gets or sets the text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>Gets or sets the ID of the associated dialog for this option.</summary>
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

    /// <summary>Gets or sets the name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>Gets or sets the price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>Gets the text to show the guest for this option.</summary>
    public string Description => double.IsNaN(Price) ? Name : $"{Name} - ${Price:0.00}";
}
```

```csharp
/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>Gets the options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static readonly List<string> _welcomeList = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the top-level dialog.</summary>
    public static IList<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(_welcomeList);

    /// <summary>Gets the reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_welcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>Gets the options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static readonly List<string> _menuList = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the food-selection dialog.</summary>
    public static IList<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(_menuList);

    /// <summary>Gets the reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_menuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bot.js** 파일에서 이 정보를 포함하는 **dinnerMenu** 상수를 만듭니다.

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

이 대화 상자는 `ChoicePrompt`를 사용하여 메뉴를 표시한 다음, 사용자가 옵션을 선택할 때까지 기다립니다. 사용자가 `Order dinner` 또는 `Reserve a table`을 선택하면 적절한 대화 상자가 시작됩니다. 자식 대화 상자가 완료되면 마지막 단계에서 대화 상자를 조용히 종료해서 사용자가 무슨 일이 일어나는지 궁금하게 만들기보다는 `mainMenu` 대화 상자를 다시 시작하여 `mainMenu` 대화 상자를 반복합니다. 이 간단한 구조를 통해 봇에는 항상 메뉴가 표시되고, 사용자는 항상 자신이 선택한 항목을 알 수 있습니다.

`mainMenu` 대화 상자에는 다음과 같은 폭포 단계가 포함되어 있습니다.

* 폭포의 첫 번째 단계에서는 다이얼로그 상태를 초기화하거나 지우고, 손님을 맞이하고, 사용 가능한 `Order dinner` 또는 `Reserve a table` 옵션 중에서 선택하도록 요청합니다.
* 두 번째 단계에서는 선택 항목을 검색한 다음, 해당 선택 항목과 연결된 자식 대화 상자를 시작합니다. 자식 다이얼로그가 종료되면 이 다이얼로그가 다음 단계로 다시 시작됩니다.
* 마지막 단계에서는 _replace dialog_ 메서드를 사용하여 이 대화 상자를 자체의 새 인스턴스로 바꿉니다. 그러면 시작 다이얼로그가 반복 메뉴로 효과적으로 전환됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

생성자 내에서 폭포 대화 상자를 추가합니다. 그런 다음, 폭포 단계를 정의합니다. 여기에서는 중첩된 클래스에 이를 정의했습니다.

**HotelDialogs** 생성자 내.

```csharp
// Define the steps for and add the main welcome dialog.
WaterfallStep[] welcomeDialogSteps = new WaterfallStep[]
{
    MainDialogSteps.PresentMenuAsync,
    MainDialogSteps.ProcessInputAsync,
    MainDialogSteps.RepeatMenuAsync,
};

Add(new WaterfallDialog(MainMenu, welcomeDialogSteps));
```

**HotelDialogs** 클래스 내에 단계 정의를 추가합니다.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class MainDialogSteps
{
    public static async Task<DialogTurnResult> PresentMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Greet the guest and ask them to choose an option.
        await stepContext.Context.SendActivityAsync(
            "Welcome to Contoso Hotel and Resort.",
            cancellationToken: cancellationToken);
        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("How may we serve you today?"),
                RetryPrompt = Lists.WelcomeReprompt,
                Choices = Lists.WelcomeChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)stepContext.Result;
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        return await stepContext.BeginDialogAsync(dialogId, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> RepeatMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Start this dialog over again.
        return await stepContext.ReplaceDialogAsync(MainMenu, null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇 생성자에서 `mainMenu` 폭포 대화 상자를 추가합니다.

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
this.dialogSet.add(new WaterfallDialog('mainMenu', [
    async function (step) {
        // Welcome the user and send a prompt.
        await step.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        return await step.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function (step) {
        // Handle the user's response to the previous prompt and branch the dialog.
        if (step.result.value.match(/order dinner/ig)) {
            return await step.beginDialog('orderDinner');
        } else if (step.result.value.match(/reserve a table/ig)) {
            return await step.beginDialog('reserveTable');
        }
    },
    async function (step) {
        // Calling replaceDialog will loop the main menu
        return await step.replaceDialog('mainMenu');
    }
]));
```

---

### <a name="create-the-order-dinner-dialog"></a>저녁 식사 주문 다이얼로그 만들기

저녁 식사 주문(order-dinner) 다이얼로그에서는 손님을 "저녁 식사 주문 서비스"로 맞이하고, 다음 섹션에서 설명하는 음식 선택(food-selection) 다이얼로그를 바로 시작합니다. 중요한 점은 손님이 자신의 주문을 처리하도록 서비스에 요청하면 음식 선택 다이얼로그에서 주문에 대한 항목 목록을 반환한다는 것입니다. 프로세스를 완료하기 위해 이 다이얼로그는 음식을 배달할 객실 번호를 요구한 다음, 확인 메시지를 보냅니다. 손님이 주문을 취소하면 이 다이얼로그에서 객실 번호를 묻지 않고 즉시 종료합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

손님의 저녁 식사 주문을 추적하는 데 사용할 수 있는 데이터 구조를 **HotelDialog** 클래스에 추가합니다. 이는 대화 상태에 보존되므로 클래스에 직렬화를 위한 기본 생성자가 필요합니다.

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice>
{
    public OrderCart() : base() { }

    public OrderCart(OrderCart other) : base(other) { }
}
```

생성자 내에서 폭포 대화 상자를 추가합니다. 그런 다음, 폭포 단계를 정의합니다. 여기에서는 중첩된 클래스에 이를 정의했습니다.

**HotelDialogs** 생성자 내.

```csharp
// Define the steps for and add the order-dinner dialog.
WaterfallStep[] orderDinnerDialogSteps = new WaterfallStep[]
{
    OrderDinnerSteps.StartFoodSelectionAsync,
    OrderDinnerSteps.GetRoomNumberAsync,
    OrderDinnerSteps.ProcessOrderAsync,
};

Add(new WaterfallDialog(Dialogs.OrderDinner, orderDinnerDialogSteps));
```

**HotelDialogs** 클래스 내에 단계 정의를 추가합니다.

* 첫 번째 단계에서는 시작 메시지를 보내고, 음식 선택 다이얼로그를 시작합니다.
* 두 번째 단계에서는 음식 선택 다이얼로그에서 주문 카트를 반환했는지 여부를 확인합니다.
  * 그렇다면 객실 번호를 묻고 카트 정보를 저장합니다.
  * 그렇지 않은 경우 주문을 취소했다고 가정하고 _end dialog_ 메서드를 호출하여 이 대화 상자를 종료합니다.
* 마지막 단계에서는 손님의 객실 번호를 기록하고 종료하기 전에 확인 메시지를 보냅니다. 이 단계는 봇에서 주문을 처리 서비스로 전달합니다.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class OrderDinnerSteps
{
    public static async Task<DialogTurnResult> StartFoodSelectionAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Welcome to our Dinner order service.",
            cancellationToken: cancellationToken);

        // Start the food selection dialog.
        return await stepContext.BeginDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> GetRoomNumberAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        if (stepContext.Result != null && stepContext.Result is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            stepContext.Values[Outputs.OrderCart] = cart;
            return await stepContext.PromptAsync(
                Inputs.Number,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your room number?"),
                    RetryPrompt = MessageFactory.Text("Please enter your room number."),
                },
                cancellationToken);
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }

    public static async Task<DialogTurnResult> ProcessOrderAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get and save the guest's answer.
        var roomNumber = (int)stepContext.Result;
        stepContext.Values[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await stepContext.Context.SendActivityAsync(
            $"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.",
            cancellationToken: cancellationToken);
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇 생성자에서 `orderDinner` 폭포 대화 상자를 추가합니다.

```javascript
// Order dinner:
// Help user order dinner from a menu
this.dialogSet.add(new WaterfallDialog('orderDinner', [
    async function (step) {
        await step.context.sendActivity("Welcome to our dinner order service.");

        return await step.beginDialog('orderPrompt', step.values.orderCart = {
            orders: [],
            total: 0
        }); // Prompt for orders
    },
    async function (step) {
        if (step.result == "Cancel") {
            return await step.endDialog();
        } else {
            return await step.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function (step) {
        await step.context.sendActivity(`Thank you. Your order will be delivered to room ${step.result} within 45 minutes.`);
        return await step.endDialog();
    }
]));
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

생성자 내에서 폭포 대화 상자를 추가합니다. 그런 다음, 폭포 단계를 정의합니다. 여기에서는 중첩된 클래스에 이를 정의했습니다.

**HotelDialogs** 생성자 내.

```csharp
// Define the steps for and add the order-prompt dialog.
WaterfallStep[] orderPromptDialogSteps = new WaterfallStep[]
{
    OrderPromptSteps.PromptForItemAsync,
    OrderPromptSteps.ProcessInputAsync,
};

Add(new WaterfallDialog(Dialogs.OrderPrompt, orderPromptDialogSteps));
```

* 첫 번째 단계에서는 다이얼로그 상태를 초기화합니다. 다이얼로그의 입력 인수에 카트 정보가 포함되어 있으면 해당 정보를 다이얼로그 상태에 저장합니다. 그렇지 않으면 빈 카트를 만들어 해당 정보를 추가합니다. 손님이 저녁 식사 메뉴에서 항목을 선택하도록 요청합니다.
* 다음 단계에서는 손님이 선택한 옵션을 살펴봅니다.
  * 주문 처리 요청인 경우 카트에 항목이 포함되어 있는지 확인합니다.
    * 그렇다면 다이얼로그를 종료하고 카트 내용을 반환합니다.
    * 그렇지 않으면 손님에게 오류 메시지를 보내고, 다이얼로그를 처음부터 다시 시작합니다.
  * 주문 취소 요청인 경우 다이얼로그를 종료하고 빈 사전을 반환합니다.
  * 저녁 식사 항목인 경우 해당 항목을 카트에 추가하고, 상태 메시지를 보내고, 다이얼로그를 시작한 다음, 현재 주문 상태를 전달합니다.

**HotelDialogs** 클래스 내에 단계 정의를 추가합니다.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-prompt dialog.
/// </summary>
private static class OrderPromptSteps
{
    public static async Task<DialogTurnResult> PromptForItemAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // First time through, options will be null.
        var cart = (stepContext.Options is OrderCart oldCart && oldCart != null)
            ? new OrderCart(oldCart) : new OrderCart();

        stepContext.Values[Outputs.OrderCart] = cart;
        stepContext.Values[Outputs.OrderTotal] = cart.Sum(item => item.Price);

        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What would you like?"),
                RetryPrompt = Lists.MenuReprompt,
                Choices = Lists.MenuChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get the guest's choice.
        var choice = (FoundChoice)stepContext.Result;
        var menuOption = Lists.MenuOptions[choice.Index];

        // Get the current order from dialog state.
        var cart = (OrderCart)stepContext.Values[Outputs.OrderCart];

        if (menuOption.Name is MenuChoice.Process)
        {
            if (cart.Count > 0)
            {
                // If there are any items in the order, then exit this dialog,
                // and return the list of selected food items.
                return await stepContext.EndDialogAsync(cart, cancellationToken);
            }
            else
            {
                // Otherwise, send an error message and restart from
                // the beginning of this dialog.
                await stepContext.Context.SendActivityAsync(
                    "Your cart is empty. Please add at least one item to the cart.",
                    cancellationToken: cancellationToken);
                return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
            }
        }
        else if (menuOption.Name is MenuChoice.Cancel)
        {
            await stepContext.Context.SendActivityAsync(
                "Your order has been cancelled.",
                cancellationToken: cancellationToken);

            // Exit this dialog, returning null.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
        else
        {
            // Add the selected food item to the order and update the order total.
            cart.Add(menuOption);
            var total = (double)stepContext.Values[Outputs.OrderTotal] + menuOption.Price;
            stepContext.Values[Outputs.OrderTotal] = total;

            await stepContext.Context.SendActivityAsync(
                $"Added {menuOption.Name} (${menuOption.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.",
                cancellationToken: cancellationToken);

            // Present the order options again, passing in the current order state.
            return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, cart);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇 생성자에서 `orderPrompt` 폭포 대화 상자를 추가합니다.

```javascript
// Helper dialog to repeatedly prompt user for orders
this.dialogSet.add(new WaterfallDialog('orderPrompt', [
    async function (step) {
        // Define a new cart of one does not exists
        step.values.orderCart = step.options;

        return await step.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function (step) {
        const choice = step.result;
        if (choice.value.match(/process order/ig)) {
            if (step.values.orderCart.orders.length > 0) {
                // Process the order
                await step.context.sendActivity("Processing your order.");
                // ...
                step.values.orderCart = undefined; // Reset cart
                return await step.endDialog();
            } else {
                await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        } else if (choice.value.match(/cancel/ig)) {
            await step.context.sendActivity("Your order has been canceled.");
            return await step.endDialog(choice.value);
        } else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if (!item) {
                await step.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            } else {
                // Add the item to cart
                step.values.orderCart.orders.push(item);
                step.values.orderCart.total += item.Price;

                await step.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${step.values.orderCart.total}`);

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        }
    }
]));
```

위의 샘플 코드에서는 기본 `orderDinner` 다이얼로그가 `orderPrompt`라는 도우미 다이얼로그를 사용하여 사용자 선택을 처리하는 과정을 보여 줍니다. `orderPrompt` 다이얼로그는 음식 항목 메뉴를 표시하고, 사용자에게 항목을 선택하도록 요청하고, 카트에 해당 항목을 추가한 다음, 루프를 통해 다시 확인합니다. 이렇게 하면 사용자가 주문에 여러 품목을 추가할 수 있습니다. 사용자가 `Process order` 또는 `Cancel`을 선택할 때까지 다이얼로그가 반복됩니다. 어떤 시점에서 실행이 부모 다이얼로그(`orderDinner` 다이얼로그)로 다시 전달됩니다. `orderDinner` 다이얼로그는 사용자가 주문을 처리하기 원할 경우 막판 조정 작업을 수행하고, 그렇지 않은 경우 종료되고 실행을 부모 다이얼로그(예: `mainMenu`)로 반환합니다. 그런 후 `mainMenu` 다이얼로그가 기본 메뉴 선택 항목을 다시 표시하는 마지막 단계를 계속 실행합니다.

---

### <a name="create-the-reserve-table-dialog"></a>테이블 예약 다이얼로그 만들기

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

이 예제를 더 짧게 유지하기 위해 여기서는 `reserveTable` 다이얼로그의 최소 구현만 보여 줍니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

생성자 내에서 폭포 대화 상자를 추가합니다. 그런 다음, 폭포 단계를 정의합니다. 여기에서는 중첩된 클래스에 이를 정의했습니다.

**HotelDialogs** 생성자 내.

```csharp
// Define the steps for and add the reserve-table dialog.
WaterfallStep[] reserveTableDialogSteps = new WaterfallStep[]
{
    ReserveTableSteps.StubAsync,
};

Add(new WaterfallDialog(Dialogs.ReserveTable, reserveTableDialogSteps));
```

**HotelDialogs** 클래스 내에 단계 정의를 추가합니다.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the reserve-table dialog.
/// </summary>
private static class ReserveTableSteps
{
    public static async Task<DialogTurnResult> StubAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Your table has been reserved.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇 생성자에서 자리 표시자 `reserveTable` 폭포 대화 상자를 추가합니다.

```javascript
// Reserve a table:
// Help the user to reserve a table
this.dialogSet.add(new WaterfallDialog('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(step){
        await step.context.sendActivity("Your table has been reserved");
        await step.endDialog();
    }
]));
```

---

### <a name="update-the-bot-code-to-call-the-dialogs"></a>대화 상자를 호출하도록 봇 코드 업데이트

대화 상자를 호출하도록 봇의 순서 처리기 코드를 업데이트합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBotAccessors.cs**의 이름을 **BotAccessors.cs**로 바꾸고, 클래스 이름을 `EchoBotAccessors`에서 `BotAccessors`로 바꿉니다. 이 봇에 필요한 상태 속성 접근자를 제공하도록 using 문과 클래스 정의를 업데이트합니다.

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
```

```csharp
/// <summary>
/// This class is created as a Singleton and passed into the IBot-derived constructor.
///  - See <see cref="EchoWithCounterBot"/> constructor for how that is injected.
///  - See the Startup.cs file for more details on creating the Singleton that gets
///    injected into the constructor.
/// </summary>
public class BotAccessors
{
    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    /// <summary>
    /// Gets the <see cref="IStatePropertyAccessor{T}"/> name used for the <see cref="DialogState"/> accessor.
    /// </summary>
    /// <remarks>Accessors require a unique name.</remarks>
    /// <value>The accessor name for the dialog state accessor.</value>
    public static string DialogStateAccessorName { get; } = $"{nameof(BotAccessors)}.DialogState";

    /// <summary>
    /// Gets or sets the DialogState property accessor.
    /// </summary>
    /// <value>
    /// The DialogState property accessor.
    /// </value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>
    /// Gets the <see cref="ConversationState"/> object for the conversation.
    /// </summary>
    /// <value>The <see cref="ConversationState"/> object.</value>
    public ConversationState ConversationState { get; }
}
```

**Startup.cs** 파일을 업데이트하여 `BotAccessors` 싱글톤을 구성합니다.

1. using 문을 업데이트합니다.

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    ```

1. 봇 상태 속성 접근자를 등록하는 `ConfigureServices` 메서드 부분을 업데이트합니다.

    ```csharp
    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
    ```

EchoWithCounterBot.cs 파일의 이름을 HotelBot.cs로 바꾸고, 클래스의 이름을 EchoWithCounterBot에서 HotelBot으로 바꿉니다.

1. 봇의 초기화 코드를 업데이트합니다.

    ```csharp
    private readonly BotAccessors _accessors;
    private readonly HotelDialogs _dialogs;
    private readonly ILogger _logger;

    /// <summary>
    /// Initializes a new instance of the <see cref="HotelBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    /// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
    public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<HotelBot>();
        _logger.LogTrace("EchoBot turn start.");
        _accessors = accessors;
        _dialogs = new HotelDialogs(_accessors.DialogStateAccessor);
    }
    ```

1. 대화 상자를 실행하도록 봇의 순서 처리기를 업데이트합니다.

    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        var dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            await dc.ContinueDialogAsync(cancellationToken);
            if (!turnContext.Responded)
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            var activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }

        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    let dc = await this.dialogSet.createContext(turnContext);

    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {

        await dc.continueDialog();

        if (!turnContext.responded) {
            await dc.beginDialog('mainMenu');
        }
    } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
        // Do we have any new members added to the conversation?
        if (turnContext.activity.membersAdded.length !== 0) {
            // Iterate over all new members added to the conversation
            for (var idx in turnContext.activity.membersAdded) {
                // Greet anyone that was not the target (recipient) of this message.
                // Since the bot is the recipient for events from the channel,
                // context.activity.membersAdded === context.activity.recipient.Id indicates the
                // bot was added to the conversation, and the opposite indicates this is a user.
                if (turnContext.activity.membersAdded[idx].id !== turnContext.activity.recipient.id) {
                    // Start the dialog.
                    await dc.beginDialog('mainMenu');
                }
            }
        }
    }

    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
```

---

## <a name="next-steps"></a>다음 단계

이 봇은 "추가 정보" 또는 "도움말"과 같은 다른 옵션을 메뉴 선택 항목에 제공하여 향상시킬 수 있습니다. 이러한 유형의 중단을 구현하는 방법에 대한 자세한 내용은 [사용자 중단 처리](bot-builder-howto-handle-user-interrupt.md)를 참조하세요.

이제 다이얼로그, 프롬프트 및 폭포를 사용하여 복잡한 대화 흐름을 관리하는 방법을 알아보았으므로, 다이얼로그(예: `orderDinner` 및 `reserveTable` 다이얼로그)를 하나의 큰 다이얼로그 집합으로 함께 묶는 대신 개별 개체로 분할하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [복합 컨트롤을 사용하여 모듈식 봇 논리 만들기](bot-builder-compositcontrol.md)
