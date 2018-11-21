---
title: 다이얼로그 다시 사용 | Microsoft Docs
description: Node.js 및 C#용 Bot Builder SDK에서 dialog 컨테이너를 사용하여 봇 논리를 모듈화하는 방법을 알아봅니다.
keywords: 복합 컨트롤, 모듈식 봇 논리
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5e1ddbef6181e265213ffa1fdcb65909e524be51
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332997"
---
# <a name="reuse-dialogs"></a>대화 상자 재사용

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

사용자 인사말, 저녁 식사 테이블 예약, 음식 주문, 알람 설정, 현재 날씨 표시 등과 같은 여러 가지 작업을 처리하는 호텔 봇을 만든다고 가정해 보겠습니다. dialog 개체 하나를 사용하여 봇 내에서 이러한 각각의 작업을 처리할 수 있지만 이렇게 하면 dialog 코드가 너무 커져서 어수선해질 수 있습니다.

대화 흐름의 일부를 구성 요소 대화 상자로 변환하여 큰 대화 상자 집합을 보다 관리가 용이한 여러 개의 부분으로 분할할 수 있습니다. 그러면 코드를 간소화하여 디버그하기 쉽게 만들고, 여러 팀이 봇에 대한 작업을 동시에 작업할 수 있습니다.
또한 다른 대화 상자 집합 및 봇에서 다시 사용할 구성 요소 대화 상자 라이브러리를 만들 수도 있습니다.

이 예제에서는 체크인, 모닝콜 및 테이블 예약 구성 요소 대화 상자를 결합한 호텔 봇을 만들겠습니다.

이 문서에서는 [간단한](bot-builder-dialog-manage-conversation-flow.md) 대화 흐름과 [복잡한](bot-builder-dialog-manage-complex-conversation-flow.md) 대화 흐름을 관리하는 방법에 대한 정보를 제공합니다.

## <a name="create-your-project"></a>프로젝트 만들기

에코 봇 템플릿으로 시작하고 프로젝트에 대화 상자 라이브러리를 포함시킵니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot** 템플릿으로 시작하겠습니다. 지침은 [.NET용 빠른 시작](../dotnet/bot-builder-dotnet-sdk-quickstart.md)을 참조하세요.

다이얼로그를 사용하려면 프로젝트 또는 솔루션에 대한 `Microsoft.Bot.Builder.Dialogs` NuGet 패키지를 설치합니다.
그런 다음, 필요에 따라 코드 파일의 using 문에서 다이얼로그 라이브러리를 참조합니다.

**EchoBotWithCounter.cs** 파일의 이름을 **HotelBot.cs**로 바꾸고, 클래스의 이름을 **HotelBot**으로 바꿉니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Echo** 템플릿으로 시작하겠습니다. 지침은 [JavaScript용 빠른 시작](../javascript/bot-builder-javascript-quickstart.md)을 참조하세요.

`botbuilder-dialogs` 라이브러리는 npm에서 다운로드할 수 있습니다. `botbuilder-dialogs` 라이브러리를 설치하려면 다음 npm 명령을 실행합니다.

```cmd
npm install --save botbuilder-dialogs
```

---

## <a name="managing-state"></a>상태 관리

복합적인 dialog를 사용하는 봇의 상태 관리를 설정하는 방법은 여러 가지가 있습니다. 이 봇에서 각 구성 요소 대화 상자는 개체를 대화 상자 결과로 반환합니다. 호출 컨텍스트는 반환된 값을 관리합니다. 상태 관리에 대한 자세한 내용은 [대화 및 사용자 속성을 사용하여 상태 저장](bot-builder-howto-v4-state.md)을 참조하세요.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

각 대화 상자는 일부 정보를 수집하고, 봇의 순서 처리기 또는 기본 메뉴는 이러한 정보를 사용자 상태에 저장합니다. 체크 인, 테이블 예약 및 경보 설정에 대한 구성 요소 대화 상자를 정의하겠습니다. 각 항목은 적절한 클래스의 개체를 반환합니다. 다음의 각 클래스를 프로젝트에 새로운 C# 클래스 모듈로 추가합니다.

```csharp
/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}

/// <summary>
/// State information associated with the check-in dialog.
/// </summary>
public class GuestInfo
{
    public string Name { get; set; }
    public string Room { get; set; }
}

/// <summary>
/// State information associated with the reserve-table dialog.
/// </summary>
public class TableInfo
{
    public string Number { get; set; }
}

/// <summary>
/// State information associated with the wake-up call dialog.
/// </summary>
public class WakeUpInfo
{
    public string Time { get; set; }
}
```

**EchoBotAccessors.cs**의 이름을 **BotAccessors.cs**로 바꾸고, 클래스 이름 **BotAccessors**를 업데이트합니다.

그런 다음, 이 코드를 사용하여 파일을 업데이트합니다. 두 대화 상자 상태에 대한 접근자와 수집되는 사용자 정보에 대한 접근자가 필요합니다.

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>
/// Contains the state objects and the state property accessors for the bot.
/// </summary>
public class BotAccessors
{
    // The property accessor keys to use.
    public const string UserInfoAccessorName = "HotelBot.UserInfo";
    public const string DialogStateAccessorName = "HotelBot.DialogState";

    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    /// <summary>Gets or sets the state property accessor for the user information we're tracking.</summary>
    /// <value>Accessor for user information.</value>
    public IStatePropertyAccessor<UserInfo> UserInfoAccessor { get; set; }

    /// <summary>Gets or sets the state property accessor for the dialog state.</summary>
    /// <value>Accessor for the dialog state.</value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>Gets the conversation state for the bot.</summary>
    /// <value>The conversation state for the bot.</value>
    public ConversationState ConversationState { get; }

    /// <summary>Gets the user state for the bot.</summary>
    /// <value>The user state for the bot.</value>
    public UserState UserState { get; }
}
```

**Startup.cs** 파일에서 상태와 봇의 상태 속성 접근자를 설정하는 `ConfigureServices` 메서드의 코드를 업데이트합니다.

```csharp
using Microsoft.Bot.Builder.Dialogs;

public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        //...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create conversation and user state objects.
        options.State.Add(new ConversationState(dataStore));
        options.State.Add(new UserState(dataStore));
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            UserInfoAccessor = userState.CreateProperty<UserInfo>(BotAccessors.UserInfoAccessorName),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

각 대화 상자는 일부 정보를 수집하고, 봇의 순서 처리기 또는 기본 메뉴는 이러한 정보를 사용자 상태에 저장합니다. 체크 인, 테이블 예약 및 경보 설정에 대한 구성 요소 대화 상자를 정의하겠습니다. 각 항목은 적절한 속성이 있는 개체를 반환합니다. 봇의 사용자 상태에서 이러한 속성을 집계하겠습니다.

**bot.js** 파일에서 봇 생성자를 업데이트하여 사용자 상태 및 대화 상자 상태를 추적하는 상태 속성 접근자를 만듭니다.

```javascript
// Define the identifiers for our state property accessors.
const DIALOG_STATE_PROPERTY = 'dialogStatePropertyAccessor';
const USER_INFO_PROPERTY = 'userInfoPropertyAccessor';

constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);
}
```

**index.js** 파일에서 `botbuilder` 라이브러리에서 가져온 클래스와 상태 개체 및 봇을 만드는 코드를 업데이트합니다.

```javascript
// Import required bot services.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();

// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

---

## <a name="define-the-check-in-component-dialog"></a>체크 인 구성 요소 대화 상자 정의

먼저 사용자에게 이름과 투숙할 객실을 질문하는 간단한 체크 인 dialog부터 시작합니다. `ComponentDialog`를 확장하는 `CheckInDialog` 클래스를 만듭니다. 이 클래스에는 루트 대화 상자의 이름을 정의하는 생성자가 있으며 이를 세 단계가 포함된 waterfall 대화 상자로 정의하겠습니다.

다음은 체크 인 대화 상자의 단계입니다.

1. 게스트의 이름을 질문합니다.
1. 투숙하려는 객실을 질문합니다.
1. 확인 메시지를 보내고 dialog를 완료합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

아래 코드를 사용하여 프로젝트에 `CheckInDialog` 클래스를 추가합니다.

이 대화 상자를 진행하는 동안, 단계 컨텍스트의 속성인 `Values`를 통해 액세스할 수 있는 로컬 상태 개체에 정보를 기록할 수 있습니다. dialog가 완료되면 로컬 상태 개체가 삭제됩니다. 따라서 게스트 정보를 포함하는 대화 상자에서 값을 반환합니다.

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class CheckInDialog : ComponentDialog
{
    private const string GuestKey = nameof(CheckInDialog);
    private const string TextPrompt = "textPrompt";

    // You can start this from the parent using the dialog's ID.
    public CheckInDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        AddDialog(new TextPrompt(TextPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
            NameStepAsync,
            RoomStepAsync,
            FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> NameStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Clear the guest information and prompt for the guest's name.
        step.Values[GuestKey] = new GuestInfo();
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What is your name?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> RoomStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the name and prompt for the room number.
        string name = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Name = name;
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text($"Hi {name}. What room will you be staying in?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the room number and "sign off".
        string room = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Room = room;

        await step.Context.SendActivityAsync(
            "Great, enjoy your stay!",
            cancellationToken: cancellationToken);

        // End the dialog, returning the guest info.
        return await step.EndDialogAsync(
            (GuestInfo)step.Values[GuestKey],
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**checkInDialog.js** 파일을 만들고 `ComponentDialog`를 확장하는 `CheckInDialog` 클래스에 추가합니다.

이 대화 상자를 진행하는 동안, 단계 컨텍스트의 속성인 `values`를 통해 액세스할 수 있는 로컬 상태 개체에 정보를 기록할 수 있습니다. dialog가 완료되면 로컬 상태 개체가 삭제됩니다. 따라서 게스트 정보를 포함하는 대화 상자에서 값을 반환합니다.

```JavaScript
const { ComponentDialog, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');

class CheckInDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new TextPrompt('textPrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(dialogId, [
            async function (step) {
                // Clear the guest information and prompt for the guest's name.
                step.values.guestInfo = {};
                return await step.prompt('textPrompt', "What is your name?");
            },
            async function (step) {
                // Save the name and prompt for the room number.
                step.values.guestInfo.userName = step.result;
                return await step.prompt('textPrompt', `Hi ${step.result}. What room will you be staying in?`);
            },
            async function (step) {
                // Save the room number and "sign off".
                step.values.guestInfo.roomNumber = step.result;
                await step.context.sendActivity(`Great! Enjoy your stay in room ${step.result}!`);

                // End the dialog, returning the guest info.
                return await step.endDialog(step.values.guestInfo);
            }
        ]));
    }
}

exports.CheckInDialog = CheckInDialog;
```

---

## <a name="define-the-reserve-table-and-wake-up-component-dialogs"></a>테이블 예약 및 부팅 구성 요소 대화 상자를 정의합니다.

구성 요소 대화 상자를 사용할 경우 이점 중 하나는 서로 다른 대화 상자를 함께 사용하는 기능입니다. 각 `DialogSet`는 `dialogs` 집합을 배타적으로 유지하기 때문에 `dialogs`를 공유하거나 상호 참조하는 것이 쉽지 않습니다. 이것이 구성 요소 dialog를 사용하는 이유입니다. 대화 상자 관리와 유지 관리를 더 수월하게 만들어 주는 구성 요소 대화 상자에서 대화 흐름의 복잡하고 미묘한 부분을 캡슐화할 수 있습니다. 대화 상자를 두 개 더 만들겠습니다. 하나는 저녁 식사를 위해 예약하려는 테이블을 질문하고 다른 하나는 모닝콜을 생성하는 dialog입니다. 다시 말하지만 각 대화 상자에 별도의 클래스를 사용하며 각 대화 상자는 기본 `ComponentDialog`를 확장합니다.

이 항목이 시작될 때 대화 상자 옵션에서 게스트 정보를 수락하도록 설계하겠습니다.

다음은 테이블 예약 대화 상자의 단계입니다.

1. 예약할 테이블을 질문합니다.
1. 확인 메시지를 보내고 대화 상자를 완성하여 테이블 번호를 반환합니다.

다음은 모닝콜 대화 상자의 단계입니다.

1. 고객을 위해 설정할 모닝콜 시간을 질문합니다.
1. 확인 메시지를 보내고 대화 상자를 완성하여 알림 시간을 반환합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

아래 코드를 사용하여 프로젝트에 `ReserveTableDialog` 클래스를 추가합니다.

대화 상자가 시작될 때 전달되는 옵션 개체에서 게스트의 이름을 가져오겠습니다. 그런 다음, 대화 상자에서 테이블 번호를 포함하는 값을 반환하겠습니다.

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class ReserveTableDialog : ComponentDialog
{
    private const string TablePrompt = "choicePrompt";

    public ReserveTableDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        AddDialog(new ChoicePrompt(TablePrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                TableStepAsync,
                FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> TableStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Welcome {guest.Name}" : "Welcome";

        string prompt = $"{greeting}, How many diners will be at your table?";
        string[] choices = new string[] { "1", "2", "3", "4", "5", "6" };
        return await step.PromptAsync(
            TablePrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(prompt),
                Choices = ChoiceFactory.ToChoices(choices),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // ChoicePrompt returns a FoundChoice object.
        string table = (step.Result as FoundChoice).Value;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Sounds great;  we will reserve a table for you for {table} diners.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the table info.
        return await step.EndDialogAsync(
            new TableInfo { Number = table },
            cancellationToken);
    }
}
```

아래 코드를 사용하여 프로젝트에 `SetAlarmDialog` 클래스를 추가합니다.

대화 상자가 시작될 때 전달되는 옵션 개체에서 게스트의 객실 번호를 가져오겠습니다. 그런 다음, 대화 상자에서 게스트가 원하는 모닝콜 시간을 포함하는 값을 반환하겠습니다.

```csharp
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class SetAlarmDialog : ComponentDialog
{
    private const string AlarmPrompt = "dateTimePrompt";

    public SetAlarmDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        // Ideally, we'd add validation to this prompt.
        AddDialog(new DateTimePrompt(AlarmPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                AlarmStepAsync,
                FinalStepAsync,
        };

        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> AlarmStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Hi {guest.Name}" : "Hi";

        string prompt = $"{greeting}. When would you like your alarm set for?";
        return await step.PromptAsync(
            AlarmPrompt,
            new PromptOptions { Prompt = MessageFactory.Text(prompt) },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Ambiguous responses can generate multiple results.
        var resolution = (step.Result as IList<DateTimeResolution>)?.FirstOrDefault();

        // Time ranges have a start and no value.
        var alarm = resolution.Value ?? resolution.Start;
        string roomNumber = (step.Options as GuestInfo)?.Room;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Your alarm is set to {alarm} for room {roomNumber}.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the alarm info.
        return await step.EndDialogAsync(
            new WakeUpInfo { Time = alarm },
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**reserveTableDialog.js** 파일을 만들고 `ComponentDialog`를 확장하는 `ReserveTableDialog` 클래스에 추가합니다.

대화 상자가 시작될 때 전달되는 옵션 개체에서 게스트의 이름을 가져오겠습니다. 그런 다음, 대화 상자에서 테이블 번호를 포함하는 값을 반환하겠습니다.

```JavaScript
const { ComponentDialog, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class ReserveTableDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new ChoicePrompt('choicePrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(dialogId, [
            async function (step) {
                // Welcome the user and ask for their table preference.
                const greeting = step.options && step.options.userName ? `Welcome ${step.options.userName}` : `Welcome`;

                const promptOptions = {
                    prompt: `${greeting}, How many diners will be at your table?`,
                    reprompt: 'That was not a valid choice, please select a table size between 1 and 6 guests.',
                    choices: ['1', '2', '3', '4', '5', '6']
                };
                return await step.prompt('choicePrompt', promptOptions);
            },
            async function (step) {
                const choice = step.result;

                // Send a confirmation message.
                const tableNumber = choice.value;
                await step.context.sendActivity(`Sounds great, we will reserve a table for you for ${tableNumber} diners.`);

                // End the dialog, returning the table info.
                return await step.endDialog({ table: tableNumber });
            }
        ]));
    }
}

exports.ReserveTableDialog = ReserveTableDialog;
```

**setAlarmDialog.js** 파일을 만들고 `ComponentDialog`를 확장하는 `SetAlarmDialog` 클래스에 추가합니다.

대화 상자가 시작될 때 전달되는 옵션 개체에서 게스트의 객실 번호를 가져오겠습니다. 그런 다음, 대화 상자에서 게스트가 원하는 모닝콜 시간을 포함하는 값을 반환하겠습니다.

```JavaScript
const { ComponentDialog, DateTimePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class SetAlarmDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.dialogs.add(new DateTimePrompt('datePrompt'));

        this.dialogs.add(new WaterfallDialog(dialogId, [
            async function (step) {
                step.values.wakeUp = {};
                if (step.options && step.options.roomNumber) {
                    step.values.roomNumber = step.options.roomNumber;
                }

                const greeting = step.options && step.options.userName ? `Hi ${step.options.userName}` : `Hi`;
                return await step.prompt('datePrompt', `${greeting}. What time would you like your alarm to be set?`);
            },
            async function (step) {
                // Ambiguous responses can generate multiple results.
                const resolution = step.result[0];

                // Time ranges have a start and no value.
                const alarm = resolution.value ? resolution.value : resolution.start;
                const roomNumber = step.values.roomNumber;

                // Send a confirmation message.
                await step.context.sendActivity(`Your alarm is set to ${alarm} for room ${roomNumber}.`);

                // End the dialog, returning the alarm info.
                return await step.endDialog({ alarm: alarm });
            }]));

        // Defining the prompt used in this conversation flow
    }
}

exports.SetAlarmDialog = SetAlarmDialog;
```

---

## <a name="add-the-component-dialogs-to-the-bot"></a>봇에 구성 요소 대화 상자 추가

세 가지 구성 요소 대화 상자를 정의했으므로 봇에 사용할 수 있습니다.

- dialog 세 개가 봇의 기본 dialog 집합에 모두 추가됩니다.
- 새로 대화가 시작될 때, 활성 dialog가 없고 봇의 on-turn 논리가 작업에 착수합니다.
- 사용자의 게스트 정보가 없는 경우 체크 인 대화 상자를 시작합니다.
- 게스트 정보가 있으면 기본 대화 상자가 이를 받아서 사용자가 테이블 예약 또는 모닝콜 대화 상자를 시작하도록 반복적으로 허용합니다.

봇의 on-turn 처리기 논리를 업데이트하겠습니다.

1. 사용자 상태를 가져옵니다.
1. 모든 활성 대화 상자를 계속합니다.
1. 대화 상자가 이 순서를 완료한 경우 체크 인 대화 상자여야 합니다.
   1. 게스트 정보를 저장합니다.
   1. 기본 대화 상자를 시작합니다.
1. 봇이 사용자에게 아직 메시지를 전송하지 않은 경우 실행 중인 활성 대화 상자가 없습니다.
    1. 게스트 정보가 없는 경우 체크 인 대화 상자를 시작합니다.
    1. 그렇지 않으면 기본 대화 상자를 시작합니다.
1. 모든 상태 변경 내용을 저장합니다.

다음은 기본 대화 상자의 단계입니다.

1. 게스트에게 테이블 예약 또는 모닝콜 설정 중 무엇을 원하는지 질문합니다.
1. 해당하는 자식 대화 상자를 시작하거나 입력을 인식할 수 없다는(_input not understood_) 메시지를 보내고 첫 단계부터 새로 시작합니다.
1. 자식 대화 상자에서 반환 값을 처리하고 기본 대화 상자를 다시 시작합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**HotelBot.cs** 파일에서 using 문을 업데이트합니다.

```csharp
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
using Microsoft.Extensions.Logging;
```

초기화 코드를 업데이트하고 봇의 대화 상자 집합과 기본 대화 상자를 정의합니다.

```csharp
// Define the IDs for the dialogs in the bot's dialog set.
private const string MainDialogId = "mainDialog";
private const string CheckInDialogId = "checkInDialog";
private const string TableDialogId = "tableDialog";
private const string AlarmDialogId = "alarmDialog";

// Define the dialog set for the bot.
private readonly DialogSet _dialogs;

// Define the state accessors and the logger for the bot.
private readonly BotAccessors _accessors;
private readonly ILogger _logger;

/// <summary>
/// Initializes a new instance of the <see cref="HotelBot"/> class.
/// </summary>
/// <param name="accessors">Contains the objects to use to manage state.</param>
/// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    if (loggerFactory == null)
    {
        throw new System.ArgumentNullException(nameof(loggerFactory));
    }

    _logger = loggerFactory.CreateLogger<HotelBot>();
    _logger.LogTrace($"{nameof(HotelBot)} turn start.");
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Define the steps of the main dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        MenuStepAsync,
        HandleChoiceAsync,
        LoopBackAsync,
    };

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    _dialogs = new DialogSet(_accessors.DialogStateAccessor)
        .Add(new WaterfallDialog(MainDialogId, steps))
        .Add(new CheckInDialog(CheckInDialogId))
        .Add(new ReserveTableDialog(TableDialogId))
        .Add(new SetAlarmDialog(AlarmDialogId));
}

private static async Task<DialogTurnResult> MenuStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Present the user with a set of "suggested actions".
    List<string> menu = new List<string> { "Reserve Table", "Wake Up" };
    await stepContext.Context.SendActivityAsync(
        MessageFactory.SuggestedActions(menu, "How can I help you?"),
        cancellationToken: cancellationToken);
    return Dialog.EndOfTurn;
}

private async Task<DialogTurnResult> HandleChoiceAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Since the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    string choice = (stepContext.Result as string)?.Trim()?.ToLowerInvariant();
    switch (choice)
    {
        case "reserve table":
            return await stepContext.BeginDialogAsync(TableDialogId, userInfo.Guest, cancellationToken);

        case "wake up":
            return await stepContext.BeginDialogAsync(AlarmDialogId, userInfo.Guest, cancellationToken);

        default:
            // If we don't recognize the user's intent, start again from the beginning.
            await stepContext.Context.SendActivityAsync(
                "Sorry, I don't understand that command. Please choose an option from the list.");
            return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
    }
}

private async Task<DialogTurnResult> LoopBackAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Because the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Process the return value from the child dialog.
    switch (stepContext.Result)
    {
        case TableInfo table:
            // Store the results of the reserve-table dialog.
            userInfo.Table = table;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        case WakeUpInfo alarm:
            // Store the results of the set-wake-up-call dialog.
            userInfo.WakeUp = alarm;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        default:
            // We shouldn't get here, since these are no other branches that get this far.
            break;
    }

    // Restart the main menu dialog.
    return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
}
```

그리고 대화 상자 집합을 사용하도록 봇의 순서 처리기를 업데이트합니다.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Establish dialog state from the conversation state.
        DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        // Get the user's info.
        UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(turnContext, () => new UserInfo(), cancellationToken);

        // Continue any current dialog.
        DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync();

        // Process the result of any complete dialog.
        if (dialogTurnResult.Status is DialogTurnStatus.Complete)
        {
            switch (dialogTurnResult.Result)
            {
                case GuestInfo guestInfo:
                    // Store the results of the check-in dialog.
                    userInfo.Guest = guestInfo;
                    await _accessors.UserInfoAccessor.SetAsync(turnContext, userInfo, cancellationToken);
                    break;
                default:
                    // We shouldn't get here, since the main dialog is designed to loop.
                    break;
            }
        }

        // Every dialog step sends a response, so if no response was sent,
        // then no dialog is currently active.
        else if (!turnContext.Responded)
        {
            if (string.IsNullOrEmpty(userInfo.Guest?.Name))
            {
                // If we don't yet have the guest's info, start the check-in dialog.
                await dc.BeginDialogAsync(CheckInDialogId, null, cancellationToken);
            }
            else
            {
                // Otherwise, start our bot's main dialog.
                await dc.BeginDialogAsync(MainDialogId, null, cancellationToken);
            }
        }

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
        await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bot.js**라는 봇 파일에서는 SDK에서 사용할 클래스뿐만 아니라 구성 요소 대화 상자에 정의한 클래스도 사용합니다.

```JavaScript
const { ActivityTypes, MessageFactory } = require('botbuilder');
const { DialogSet, WaterfallDialog, Dialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Import our component dialogs.
const { CheckInDialog } = require("./checkInDialog");
const { ReserveTableDialog } = require("./reserveTableDialog");
const { SetAlarmDialog } = require("./setAlarmDialog");
```

또한 대화 상자 집합을 만들고 여기에 사용할 모든 대화 상자를 추가해야 합니다.

기본 대화 상자의 폭포 단계를 인라인 대신 클래스에 함수로 정의합니다. 함수 내에서 `this`가 올바르게 확인되도록 이러한 함수에 `bind()`를 사용합니다.

업데이트된 봇 생성자는 다음과 같습니다.

```JavaScript
constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new CheckInDialog('checkInDialog'))
        .add(new ReserveTableDialog('reserveTableDialog'))
        .add(new SetAlarmDialog('setAlarmDialog'))
        .add(new WaterfallDialog('mainDialog', [
            this.promptForChoice.bind(this),
            this.startChildDialog.bind(this),
            this.saveResult.bind(this)
    ]));
}
```

봇 생성자 아래에 기본 대화 상자의 단계를 구현하는 다음 코드를 추가합니다.

```JavaScript
async promptForChoice(step) {
    const menu = ["Reserve Table", "Wake Up"];
    await step.context.sendActivity(MessageFactory.suggestedActions(menu, 'How can I help you?'));
    return Dialog.EndOfTurn;
}

async startChildDialog(step) {
    // Get the user's info.
    const user = await this.userInfoAccessor.get(step.context);
    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    switch (step.result) {
        case "Reserve Table":
            return await step.beginDialog('reserveTableDialog', user.guestInfo);
            break;
        case "Wake Up":
            return await step.beginDialog('setAlarmDialog', user.guestInfo);
            break;
        default:
            await step.context.sendActivity("Sorry, I don't understand that command. Please choose an option from the list.");
            return await step.replaceDialog('mainDialog');
            break;
    }
}

async saveResult(step) {
    // Process the return value from the child dialog.
    if (step.result) {
        const user = await this.userInfoAccessor.get(step.context);
        if (step.result.table) {
            // Store the results of the reserve-table dialog.
            user.table = step.result.table;
        } else if (step.result.alarm) {
            // Store the results of the set-wake-up-call dialog.
            user.alarm = step.result.alarm;
        }
        await this.userInfoAccessor.set(step.context, user);
    }
    // Restart the main menu dialog.
    return await step.replaceDialog('mainDialog'); // Show the menu again
}
```

이제 봇의 순서 처리기를 업데이트합니다.

```JavaScript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        const user = await this.userInfoAccessor.get(turnContext, {});
        const dc = await this.dialogs.createContext(turnContext);
        const dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            user.guestInfo = dialogTurnResult.result;
            await this.userInfoAccessor.set(turnContext, user);
            await dc.beginDialog('mainDialog');
        } else if (!turnContext.responded) {
            if (!user.guestInfo) {
                await dc.beginDialog('checkInDialog');
            } else {
                await dc.beginDialog('mainDialog');
            }
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

보이는 것처럼 구성 요소 대화 상자는 대화 상자에 [프롬프트](bot-builder-prompts.md)를 추가하는 것과 유사한 방식으로 봇의 기본 대화 상자에 추가됩니다. 자식 dialog는 기본 dialog에 필요한 만큼 추가할 수 있습니다. 각 모듈은 봇이 사용자에게 제공할 수 있는 추가적인 기능과 서비스를 추가하게 됩니다.

## <a name="next-steps"></a>다음 단계

구성 요소 대화 상자를 사용하는 방법을 이해했으면 Language Understanding(LUIS)을 사용하여 봇이 dialog를 시작할 시기를 결정할 수 있도록 만드는 방법을 알아보겠습니다.

> [!div class="nextstepaction"]
> [Language Understanding에 대해 LUIS 사용](./bot-builder-howto-v4-luis.md)
