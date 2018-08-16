---
title: dialog 컨테이너를 사용하여 모듈식 봇 논리 만들기 | Microsoft Docs
description: Node.js 및 C#용 Bot Builder SDK에서 dialog 컨테이너를 사용하여 봇 논리를 모듈화하는 방법을 알아봅니다.
keywords: 복합 컨트롤, 모듈식 봇 논리
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2441a32167618ebb08e6a43d68d74076c3351d8f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303922"
---
# <a name="create-modular-bot-logic-with-a-dialog-container"></a>dialog 컨테이너를 사용하여 모듈식 봇 논리 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

사용자 인사말, 저녁 식사 테이블 예약, 음식 주문, 알람 설정, 현재 날씨 표시 등과 같은 여러 가지 작업을 처리하는 호텔 봇을 만든다고 가정해 보겠습니다. dialog 개체 하나를 사용하여 봇 내에서 이러한 각각의 작업을 처리할 수 있지만 이렇게 하면 dialog 코드가 너무 커져서 봇의 기본 파일이 어수선해질 수 있습니다.

모듈화를 통해 이 문제를 해결할 수 있습니다. 모듈화는 코드를 간소화하고 디버그를 쉽게 만듭니다. 여러 섹션으로 나누면 여러 팀이 동시에 봇을 작업할 수 있습니다. dialog 컨테이너를 사용하여 여러 대화 흐름을 구성 요소로 분리하면 여러 대화 흐름을 관리하는 봇을 만들 수 있습니다. 몇 가지 기본적인 대화 흐름을 만들어서 dialog 컨테이너를 사용하여 이것을 결합하는 방법을 알아보겠습니다.

이 예제에서는 체크인, 모닝콜 및 테이블 예약 모듈을 결합한 호텔 봇을 만들겠습니다.

## <a name="managing-state"></a>상태 관리

복합적인 dialog를 사용하는 봇의 상태 관리를 설정하는 방법은 여러 가지가 있습니다. 이 봇에서는 그 중 한 가지 방법을 보여줍니다.

상태 관리에 대한 자세한 내용은 [대화 및 사용자 속성을 사용하여 상태 저장](bot-builder-howto-v4-state.md)을 참조하세요.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

각 dialog는 정보를 수집하여 사용자 상태에 저장합니다. dialog마다 클래스를 정의하고 이 클래스를 사용자 상태 클래스에서 속성으로 사용하겠습니다.

```csharp
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

/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}
```

봇 턴(turn) 내에서 dialog 집합의 `CreateContext` 메서드는 dialog 상태를 설정합니다.
메서드는 턴(turn) 컨텍스트와 상태 개체를 매개 변수로 사용합니다.

dialog에 대해 이 상태 개체는 `IDictionary<string, object>` 인터페이스를 구현해야 합니다. 봇은 대화 상태만을 사용하여 dialog 상태를 포함하기 때문에 대화 상태 클래스는 간단한 사전이 될 수 있습니다.

```csharp
using System.Collections.Generic;

/// <summary>
/// Conversation state information.
/// We are also using this directly for dialog state, which needs an <see cref="IDictionary{string, object}"/> object.
/// </summary>
public class ConversationInfo : Dictionary<string, object> { }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

사용자의 입력을 추적하기 위해 부모 dialog에서 `userState` 개체를 dialog 매개 변수로 전달합니다. 각 dialog 클래스에서 dialog는 생성자로 빌드되어 `userState`에 정보를 저장할 수 있도록 합니다. 이 dialog 전체에서, 사용자가 정보를 입력하면 `dc.activeDialog.state` 개체의 속성으로 정의된 로컬 상태 개체에 쓸 수 있습니다. dialog가 완료된 후에는 로컬 상태 개체가 삭제됩니다. 따라서 부모의 `userState`에 로컬 상태 개체를 저장하여 사용자와의 모든 대화에서 사용자에 대한 정보가 유지되도록 합니다. 

---

## <a name="define-a-modular-check-in-dialog"></a>모듈식 체크 인 dialog 정의

먼저 사용자에게 이름과 투숙할 객실을 질문하는 간단한 체크 인 dialog부터 시작합니다. 이 작업을 모듈화하기 위해 `DialogContainer`를 확장하는 `CheckIn` 클래스를 만듭니다. 이 클래스에는 루트 dialog의 이름을 정의하는 생성자가 있으며 이것은 세 단계가 포함된 *waterfall*로 정의되어 있습니다. dialog 개체의 서명 및 구성은 표준 waterfall과 정확히 같습니다.

**체크 인 dialog 단계**

1. 게스트의 이름을 질문합니다.
1. 투숙하려는 객실을 질문합니다.
1. 확인 메시지를 보내고 dialog를 완료합니다.

dialog와 waterfall에 대한 자세한 내용은 [dialog를 사용하여 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)를 참조하세요.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`CheckIn` 클래스에는 체크 인 dialog의 단계를 정의하고 단일 인스턴스를 생성하여 정적 `Instance` 속성에 노출하는 Private 생성자가 있습니다.

이 dialog를 진행하는 동안, dialog 컨텍스트의 속성인 `dc.ActiveDialog.State`를 통해 액세스할 수 있는 로컬 상태 개체에 정보를 기록할 수 있습니다. dialog가 완료되면 로컬 상태 개체가 삭제됩니다. 따라서 사용자와의 모든 대화에서 사용자에 대한 정보가 유지되도록 봇의 `userState`에 로컬 상태 개체를 저장합니다.

상태 관리에 대한 자세한 내용은 [대화 및 사용자 속성을 사용하여 상태 저장](bot-builder-howto-v4-state.md)을 참조하세요. 

**CheckIn.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;

namespace HotelBot
{
    public class CheckIn : DialogContainer
    {
        public const string Id = "checkIn";

        private const string GuestKey = nameof(CheckIn);

        public static CheckIn Instance { get; } = new CheckIn();

        // You can start this from the parent using the dialog's ID.
        private CheckIn() : base(Id)
        {
            // Define the conversation flow using a waterfall model.
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the guest information and prompt for the guest's name.
                    dc.ActiveDialog.State[GuestKey] = new GuestInfo();
                    await dc.Prompt("textPrompt", "What is your name?");
                },
                async (dc, args, next) =>
                {
                    // Save the name and prompt for the room number.
                    var name = args["Value"] as string;

                    var guestInfo = dc.ActiveDialog.State[GuestKey];
                    ((GuestInfo)guestInfo).Name = name;

                    await dc.Prompt("numberPrompt",
                        $"Hi {name}. What room will you be staying in?");
                },
                async (dc, args, next) =>
                {
                    // Save the room number and "sign off".
                    var room = (string)args["Text"];

                    var guestInfo = dc.ActiveDialog.State[GuestKey];
                    ((GuestInfo)guestInfo).Room = room;

                    await dc.Context.SendActivity("Great, enjoy your stay!");

                    // Save dialog state to user state and end the dialog.
                    var userState = UserState<UserInfo>.Get(dc.Context);
                    userState.Guest = (GuestInfo)guestInfo;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("textPrompt", new TextPrompt());
            this.Dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**checkIn.js**
```JavaScript
const { DialogContainer, DialogSet, TextPrompt, NumberPrompt } = require('botbuilder-dialogs');

class CheckIn extends DialogContainer {
    constructor(userState) {
        // Dialog ID of 'checkIn' will start when class is called in the parent
        super('checkIn');

        // Defining the conversation flow using a waterfall model
        this.dialogs.add('checkIn', [
            async function (dc) {
                // Create a new local guestInfo state object
                dc.activeDialog.state.guestInfo = {};
                await dc.context.sendActivity("What is your name?");
            },
            async function (dc, name){
                // Save the name 
                dc.activeDialog.state.guestInfo.userName = name;
                await dc.prompt('numberPrompt', `Hi ${name}. What room will you be staying in?`);
            },
            async function (dc, room){
                // Save the room number
                dc.activeDialog.state.guestInfo.room = room
                await dc.context.sendActivity(`Great! Enjoy your stay!`);

                // Save dialog's state object to the parent's state object
                const user = userState.get(dc.context);
                user.guestInfo = dc.activeDialog.state.guestInfo;
                await dc.end();
            }
        ]);
        // Defining the prompt used in this conversation flow
        this.dialogs.add('textPrompt', new TextPrompt());
        this.dialogs.add('numberPrompt', new NumberPrompt());
    }
}
exports.CheckIn = CheckIn;
```

---

## <a name="define-modular-reserve-table-and-wake-up-dialogs"></a>모듈식 테이블 예약 및 모닝콜 dialog 정의

dialog 컨테이너를 사용하는 이점 중 하나는 dialogs를 함께 작성할 수 있는 기능입니다. 각 `DialogSet`는 `dialogs` 집합을 배타적으로 유지하기 때문에 `dialogs`를 공유하거나 상호 참조하는 것이 쉽지 않습니다. 이것이 dialog 컨테이너를 사용하는 이유입니다. dialog 컨테이너를 사용하여 dialog 전반에서 dialog 흐름을 보다 쉽게 관리할 수 있는 복합 dialog를 만들 수 있습니다. 이것을 보여주기 위해 dialog를 두 개 더 만듭니다. 하나는 저녁 식사를 위해 예약하려는 테이블을 질문하고 다른 하나는 모닝콜을 생성하는 dialog입니다. 다시 말하지만 각 dialog에 별도의 클래스를 사용하며 각 dialog는 `DialogContainer`를 확장합니다.

**테이블 예약 dialog 단계**

1. 예약할 테이블을 질문합니다.
1. 확인 메시지를 보내고 dialog를 완료합니다.

**모닝콜 dialog 단계**

1. 고객을 위해 설정할 모닝콜 시간을 질문합니다.
1. 확인 메시지를 보내고 dialog를 완료합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**ReserveTable.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System.Linq;
using Choice = Microsoft.Bot.Builder.Prompts.Choices.Choice;
using FoundChoice = Microsoft.Bot.Builder.Prompts.Choices.FoundChoice;

namespace HotelBot
{
    public class ReserveTable : DialogContainer
    {
        public const string Id = "reserveTable";

        private const string TableKey = "Table";

        public static ReserveTable Instance { get; } = new ReserveTable();

        private ReserveTable() : base(Id)
        {
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the table information and prompt for the table number.
                    dc.ActiveDialog.State[TableKey] = new TableInfo();

                    var guestInfo = UserState<UserInfo>.Get(dc.Context).Guest;

                    var prompt = $"Welcome {guestInfo.Name}, " +
                        $"which table would you like to reserve?";
                    var choices = new string[] { "1", "2", "3", "4", "5", "6" };
                    await dc.Prompt("choicePrompt", prompt,
                        new ChoicePromptOptions
                        {
                            Choices = choices.Select(s => new Choice { Value = s }).ToList()
                        });
                },
                async (dc, args, next) =>
                {
                    // Save the table number and "sign off".
                    var table = (args["Value"] as FoundChoice).Value;

                    var tableInfo = dc.ActiveDialog.State[TableKey];
                    ((TableInfo)tableInfo).Number = table;

                    await dc.Context.SendActivity(
                        $"Sounds great; we will reserve table number {table} for you.");

                    // Save dialog state to user state and end the dialog.
                    var userState = UserState<UserInfo>.Get(dc.Context);
                    userState.Table = (TableInfo)tableInfo;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("choicePrompt", new ChoicePrompt(Culture.English));
        }
    }
}
```

**WakeUp.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
using System.Linq;
using DateTimeResolution = Microsoft.Bot.Builder.Prompts.DateTimeResult.DateTimeResolution;

namespace HotelBot
{
    public class WakeUp : DialogContainer
    {
        public const string Id = "wakeUp";

        private const string WakeUpKey = "WakeUp";

        public static WakeUp Instance { get; } = new WakeUp();

        private WakeUp() : base(Id)
        {
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the wake up call information and prompt for the alarm time.
                   dc.ActiveDialog.State[WakeUpKey] = new WakeUpInfo();

                    var guestInfo = UserState<UserInfo>.Get(dc.Context).Guest;

                    await dc.Prompt("datePrompt", $"Hi {guestInfo.Name}, " +
                        $"what time would you like your alarm set for?");
                },
                async (dc, args, next) =>
                {
                    // Save the alarm time and "sign off".
                    var resolution = (List<DateTimeResolution>)args["Resolution"];

                    var wakeUp = dc.ActiveDialog.State[WakeUpKey];
                    ((WakeUpInfo)wakeUp).Time = resolution?.FirstOrDefault()?.Value;

                    var userState = UserState<UserInfo>.Get(dc.Context);
                    await dc.Context.SendActivity(
                        $"Your alarm is set to {((WakeUpInfo)wakeUp).Time}" +
                        $" for room {userState.Guest.Room}.");

                    // Save dialog state to user state and end the dialog.
                    userState.WakeUp = (WakeUpInfo)wakeUp;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("datePrompt", new DateTimePrompt(Culture.English));
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**reserveTable.js**
```JavaScript
const { DialogContainer, DialogSet, ChoicePrompt } = require('botbuilder-dialogs');

class ReserveTable extends DialogContainer {
    constructor(userState) {
        // Dialog ID of 'reserve_table' will start when class is called in the parent
        super('reserve_table'); 

        // Defining the conversation flow using a waterfall model
        this.dialogs.add('reserve_table', [
            async function (dc, args) {
                // Get the user state from context
                const user = userState.get(dc.context);

                // Create a new local reserveTable state object
                dc.activeDialog.state.reserveTable = {};

                const prompt = `Welcome ${user.guestInfo.userName}, which table would you like to reserve?`;
                const choices = ['1', '2', '3', '4', '5', '6'];
                await dc.prompt('choicePrompt', prompt, choices);
            },
            async function(dc, choice){
                // Save the table number
                dc.activeDialog.state.reserveTable.tableNumber = choice.value;
                await dc.context.sendActivity(`Sounds great, we will reserve table number ${choice.value} for you.`);
                
                // Get the user state from context
                const user = userState.get(dc.context);
                // Persist dialog's state object to the parent's state object
                user.reserveTable = dc.activeDialog.state.reserveTable;

                // End the dialog
                await dc.end();
            }
        ]);

        // Defining the prompt used in this conversation flow
        this.dialogs.add('choicePrompt', new ChoicePrompt());
    }
}
exports.ReserveTable = ReserveTable;
```

**wakeUp.js**
```JavaScript
const { DialogContainer, DialogSet, DatetimePrompt } = require('botbuilder-dialogs');

class WakeUp extends DialogContainer {
    constructor(userState) {
        // Dialog ID of 'wakeup' will start when class is called in the parent
        super('wakeUp');

        this.dialogs.add('wakeUp', [
            async function (dc, args) {
                // Get the user state from context
                const user = userState.get(dc.context); 

                // Create a new local reserveTable state object
                dc.activeDialog.state.wakeUp = {};  
                             
                await dc.prompt('datePrompt', `Hello, ${user.guestInfo.userName}. What time would you like your alarm to be set?`);
            },
            async function (dc, time){
                // Get the user state from context
                const user = userState.get(dc.context);

                // Save the time
                dc.activeDialog.state.wakeUp.time = time[0].value

                await dc.context.sendActivity(`Your alarm is set to ${time[0].value} for room ${user.guestInfo.room}`);
                
                // Save dialog's state object to the parent's state object
                user.wakeUp = dc.activeDialog.state.wakeUp;

                // End the dialog
                await dc.end();
            }]);

        // Defining the prompt used in this conversation flow
        this.dialogs.add('datePrompt', new DatetimePrompt());
    }
}
exports.WakeUp = WakeUp;
```

---

## <a name="add-modular-dialogs-to-a-bot"></a>봇에 모듈식 dialog 추가

기본 봇 파일은 모듈화된 dialog 세 개를 하나의 봇으로 묶습니다.

- dialog 세 개가 봇의 기본 dialog 집합에 모두 추가됩니다.
- 테이블 예약 및 모닝콜 dialog는 기본 dialog의 대화 흐름에 내장되어 있습니다.
- 새로 대화가 시작될 때, 활성 dialog가 없고 봇의 on-turn 논리가 작업에 착수합니다.

**봇의 on-turn 처리기**

봇 턴이 활성 dialog 외부에 있을 때면 항상 사용자 상태를 확인합니다.
1. 게스트 정보가 이미 있는 경우 기본 dialog를 시작합니다.
1. 그렇지 않으면 기본 dialog의 체크 인 자식 dialog를 시작합니다.

**기본 dialog 단계**

1. 게스트에게 테이블 예약 또는 모닝콜 설정 중 무엇을 원하는지 질문합니다.
1. 해당하는 자식 dialog를 시작하거나 입력을 인식할 수 없다는(_input not understood_) 메시지를 보내고 다음 단계로 건너뜁니다.
1. 건너뛰어 dialog의 시작 부분으로 돌아갑니다.


# <a name="ctabcsharp"></a>[C#](#tab/csharp)

dialog 흐름은 dialog 컨텍스트의 `Continue` 메서드를 사용하여 업데이트됩니다. 이 메서드는 dialog 스택에 있는 waterfall의 다음 단계를 실행합니다.

**HotelBot.cs**
```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

namespace HotelBot
{
    public class HotelBot : IBot
    {
        private const string MainMenuId = "mainMenu";

        private DialogSet _dialogs { get; } = ComposeMainDialog();

        /// <summary>
        /// Every Conversation turn for our bot calls this method. 
        /// </summary>
        /// <param name="context">The current turn context.</param>        
        public async Task OnTurn(ITurnContext context)
        {
            if (context.Activity.Type is ActivityTypes.Message)
            {
                // Get the user and conversation state from the turn context.
                var userInfo = UserState<UserInfo>.Get(context);
                var conversationInfo = ConversationState<ConversationInfo>.Get(context);

                // Establish dialog state from the conversation state.
                var dc = _dialogs.CreateContext(context, conversationInfo);

                // Continue any current dialog.
                await dc.Continue();

                // Every turn sends a response, so if no response was sent,
                // then there no dialog is currently active.
                if (!context.Responded)
                {
                    // If we don't yet have the guest's info, start the check-in dialog.
                    if (string.IsNullOrEmpty(userInfo?.Guest?.Name))
                    {
                        await dc.Begin(CheckIn.Id);
                    }
                    // Otherwise, start our bot's main dialog.
                    else
                    {
                        await dc.Begin(MainMenuId);
                    }
                }
            }
        }

        /// <summary>
        /// Composes a main dialog for our bot.
        /// </summary>
        /// <returns>A new main dialog.</returns>
        private static DialogSet ComposeMainDialog()
        {
            var dialogs = new DialogSet();

            dialogs.Add(MainMenuId, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    var menu = new List<string> { "Reserve Table", "Wake Up" };
                    await dc.Context.SendActivity(MessageFactory.SuggestedActions(menu, "How can I help you?"));
                },
                async (dc, args, next) =>
                {
                    // Decide which dialog to start.
                    var result = (args["Activity"] as Activity)?.Text?.Trim().ToLowerInvariant();
                    switch (result)
                    {
                        case "reserve table":
                            await dc.Begin(ReserveTable.Id);
                            break;
                        case "wake up":
                            await dc.Begin(WakeUp.Id);
                            break;
                        default:
                            await dc.Context.SendActivity("Sorry, I don't understand that command. Please choose an option from the list below.");
                            await next();
                            break;
                    }
                },
                async (dc, args, next) =>
                {
                    // Show the main menu again.
                    await dc.Replace(MainMenuId);
                }
            });

            // Add our child dialogs.
            dialogs.Add(CheckIn.Id, CheckIn.Instance);
            dialogs.Add(ReserveTable.Id, ReserveTable.Instance);
            dialogs.Add(WakeUp.Id, WakeUp.Instance);

            return dialogs;
        }
    }
}
```

마지막으로 `StartUp` 클래스의 `ConfigureServices` 메서드를 업데이트하여 봇을 연결하고 상태 미들웨어를 추가해야 합니다.

**Startup.cs**
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(HotelBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // Add state management middleware for conversation and user state.
        var path = Path.Combine(Path.GetTempPath(), nameof(HotelBot));
        if (!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
        }
        IStorage storage = new FileStorage(path);

        options.Middleware.Add(new ConversationState<ConversationInfo>(storage));
        options.Middleware.Add(new UserState<UserInfo>(storage));
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

dialog 흐름은 dialog 컨텍스트의 `continue` 메서드를 사용하여 업데이트됩니다. 이 메서드는 dialog 스택에 있는 waterfall의 다음 단계를 실행합니다.

**app.js**
```JavaScript
const {BotFrameworkAdapter, FileStorage, ConversationState, UserState, BotStateSet, MessageFactory} = require("botbuilder");
const {DialogSet} = require("botbuilder-dialogs");
const restify = require("restify");
var azure = require('botbuilder-azure'); 

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

//Memory Storage
const storage = new FileStorage("c:/temp");
// ConversationState lasts for the entirety of a conversation then gets disposed of
const convoState = new ConversationState(storage);

// UserState persists information about the user across all of the conversations you have with that user
const userState  = new UserState(storage);

adapter.use(new BotStateSet(convoState, userState));

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';

        // State will store all of your information 
        const convo = convoState.get(context);
        const user = userState.get(context); // userState will not be used in this example

        const dc = dialogs.createContext(context, convo);
        // Continue the current dialog if one is currently active
        await dc.continue(); 

        // Default action
        if (!context.responded && isMessage) {

            // Getting the user info from the state
            const userinfo = userState.get(dc.context); 
            // If guest info is undefined prompt the user to check in
            if(!userinfo.guestInfo){
                await dc.begin('checkInPrompt');
            }else{
                await dc.begin('mainMenu'); 
            }           
        }
    });
});

const dialogs = new DialogSet();
dialogs.add('mainMenu', [
    async function (dc, args) {
        const menu = ["Reserve Table", "Wake Up"];
        await dc.context.sendActivity(MessageFactory.suggestedActions(menu));    
    },
    async function (dc, result){
        // Decide which module to start
        switch(result){
            case "Reserve Table":
                await dc.begin('reservePrompt');
                break;
            case "Wake Up":
                await dc.begin('wakeUpPrompt');
                break;
            default:
                await dc.context.sendActivity("Sorry, i don't understand that command. Please choose an option from the list below.");
                break;            
        }
    },
    async function (dc, result){
        await dc.replace('mainMenu'); // Show the menu again
    }

]);

// Importing the dialogs 
const checkIn = require("./checkIn");
dialogs.add('checkInPrompt', new checkIn.CheckIn(userState));

const reserve_table = require("./reserveTable");
dialogs.add('reservePrompt', new reserve_table.ReserveTable(userState));

const wake_up = require("./wake_up");
dialogs.add('wakeUpPrompt', new wake_up.WakeUp(userState));
```

---
<!-- TODO: These dialogs are not fully modularized, as there are cross dependencies:
    - Importantly, the dialogs need to know details about the bot's user state.
-->

보이는 것처럼 모듈식 dialog는 dialog에 [프롬프트](bot-builder-prompts.md)를 추가하는 것과 유사한 방식으로 봇의 기본 dialog에 추가됩니다. 자식 dialog는 기본 dialog에 필요한 만큼 추가할 수 있습니다. 각 모듈은 봇이 사용자에게 제공할 수 있는 추가적인 기능과 서비스를 추가하게 됩니다.

## <a name="next-steps"></a>다음 단계

dialog를 사용하여 봇 논리를 모듈화하는 방법을 이해했으면 Language Understanding(LUIS)을 사용하여 봇이 dialog를 시작할 시기를 결정할 수 있도록 만드는 방법을 알아보겠습니다.

> [!div class="nextstepaction"]
> [Language Understanding에 대해 LUIS 사용](./bot-builder-howto-v4-luis.md)
