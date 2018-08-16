---
title: 다이얼로그를 사용하여 대화 흐름 관리 | Microsoft Docs
description: Node.js용 Bot Builder SDK에서 다이얼로그를 사용하여 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 대화 흐름, 다이얼로그, 프롬프트, 폭포, 다이얼로그 집합
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/8/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 99184ba71072c159c598c7f68289c42a51926795
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302203"
---
# <a name="manage-conversation-flow-with-dialogs"></a>다이얼로그를 사용하여 대화 흐름 관리
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]


대화 흐름 관리는 봇을 빌드하는 데 필수적인 작업입니다. Node.js용 Bot Builder SDK를 사용하면 **다이얼로그**를 사용하여 대화 흐름을 관리할 수 있습니다.

다이얼로그는 프로그램의 함수와 같습니다. 대개 특정 작업을 수행하도록 디자인되며 필요한 만큼 호출할 수 있습니다. 여러 개의 다이얼로그를 함께 연결하여 봇이 처리할 대화 흐름을 처리할 수 있습니다. Node.js용 Bot Builder SDK의 **다이얼로그** 라이브러리에는 다이얼로그를 통해 대화 흐름을 관리하는 데 도움이 되는 **프롬프트** 및 **폭포**와 같은 기본 제공 기능이 포함됩니다. 프롬프트 라이브러리는 사용자에게 다양한 유형의 정보를 물어보는 데 사용할 수 있는 다양한 프롬프트를 제공합니다. 폭포는 여러 단계를 하나의 시퀀스로 조합하는 방법을 제공합니다.

이 문서에서는 다이얼로그 개체를 만들고 프롬프트 및 폭포형 단계를 다이얼로그 집합에 추가하여 간단한 대화 흐름과 복잡한 대화 흐름을 둘 다 관리하는 방법을 보여 줍니다. 

## <a name="install-the-dialogs-library"></a>다이얼로그 라이브러리 설치

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
다이얼로그를 사용하려면 프로젝트 또는 솔루션에 대한 `Microsoft.Bot.Builder.Dialogs` NuGet 패키지를 설치합니다.
그런 후 코드 파일의 using 문에서 다이얼로그 라이브러리를 참조합니다. 예: 

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
`botbuilder-dialogs` 라이브러리는 NPM에서 다운로드할 수 있습니다. `botbuilder-dialogs` 라이브러리를 설치하려면 다음 NPM 명령을 실행합니다.

```cmd
npm install --save botbuilder-dialogs
```

봇에서 **대화 상자**를 사용하려면 이를 봇 코드에 포함시킵니다. 예: 

**app.js**

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```
---

## <a name="create-a-dialog-stack"></a>다이얼로그 스택 만들기

다이얼로그를 사용하려면 먼저 *다이얼로그 집합*을 만들어야 합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Microsoft.Bot.Builder.Dialogs` 라이브러리는 `DialogSet` 클래스를 제공합니다.
다이얼로그 집합에 명명된 다이얼로그 및 다이얼로그 집합을 추가한 후 나중에 이름을 사용해서 액세스할 수 있습니다.

```csharp
IDialog dialog = null;
// Initialize dialog.

DialogSet dialogs = new DialogSet();
dialogs.Add("dialog name", dialog);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`botbuilder-dialogs` 라이브러리는 `DialogSet` 클래스를 제공합니다.
**DialogSet** 클래스는 **다이얼로그 스택**을 정의하고 스택을 관리하기 위한 간단한 인터페이스를 제공합니다.
Bot Builder SDK는 스택을 배열로 구현합니다.

**DialogSet**을 만들려면 다음을 수행합니다.

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

위의 호출은 `dialogStack`라는 기본 **다이얼로그 스택**을 사용하여 **DialogSet**을 만듭니다.
스택에 이름을 지정하려는 경우 **DialogSet()** 에 매개 변수로 전달할 수 있습니다. 예: 

```javascript
const dialogs = new botbuilder_dialogs.DialogSet("myStack");
```
---

다이얼로그를 만들면 해당 집합에 다이얼로그 정의만 추가됩니다. 이 다이얼로그는 _begin_ 또는 _replace_ 메서드를 호출하여 다이얼로그 스택에 푸시해야만 실행됩니다. 

다이얼로그 이름(예: `addTwoNumbers`)은 각 다이얼로그 집합 내에서 고유해야 합니다. 각 집합 내에서 필요한 수만큼 다이얼로그를 정의할 수 있습니다.

다이얼로그 라이브러리는 다음 다이얼로그를 정의합니다.
-   **프롬프트** 다이얼로그: 다이얼로그가 사용자에게 입력을 프롬프트하는 함수와 입력을 처리하는 함수를 포함하는 2개 이상의 함수를 사용합니다.
    **폭포형** 모델을 사용하여 이러한 함수를 하나로 연결할 수 있습니다.
-   **폭포** 다이얼로그는 순서대로 실행되는 _폭포형 단계_ 시퀀스를 정의합니다.
    폭포형 다이얼로그는 단일 단계를 포함할 수 있으며 이 경우 간단한 1단계 다이얼로그로 간주될 수 있습니다.

## <a name="create-a-single-step-dialog"></a>단일 단계 다이얼로그 만들기

단일 단계 다이얼로그는 단일 선회 대화형 흐름을 캡처하는 데 유용할 수 있습니다. 이 예제에서는 사용자가 "1 + 2"와 같이 말하는지 여부를 검색하고 `addTwoNumbers` 다이얼로그를 시작하여 "1 + 2 = 3"으로 회신하는 봇을 만듭니다다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

값은 `IDictionary<string,object>` 속성 모음으로서 다이얼로그에 전달되고 다이얼로그에서 반환됩니다.

다이얼로그 집합 내에 간단한 다이얼로그를 만들려면 `Add` 메서드를 사용합니다. 다음에서는 `addTwoNumbers`라는 1단계 폭포를 추가합니다.

이 단계에서는 전달되는 다이얼로그 인수가 추가할 숫자를 나타내는 `first` 및 `second` 속성을 포함한다고 가정합니다.

EchoBot 템플릿으로 시작합니다. 그런 후 봇 클래스에 코드를 추가하여 구문에 다이얼로그를 추가합니다.
```csharp
public class EchoBot : IBot
{
    private DialogSet _dialogs;

    public EchoBot()
    {
        _dialogs = new DialogSet();
        _dialogs.Add("addTwoNumbers", new WaterfallStep[]
        {              
            async (dc, args, next) =>
            {
                double sum = (double)args["first"] + (double)args["second"];
                await dc.Context.SendActivity($"{args["first"]} + {args["second"]} = {sum}");
                await dc.End();
            }
        });
    }

    // The rest of the class definition is omitted here but would include OnTurn()
}

```

### <a name="pass-arguments-to-the-dialog"></a>다이얼로그에 인수 전달

봇의 `OnTurn` 메서드 내에서 다이얼로그를 호출하려면 다음을 포함하도록 `OnTurn`을 수정합니다.
```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // create a dialog context
        var dialogCtx = _dialogs.CreateContext(context, state);

        // Bump the turn count. 
        state.TurnCount++;

        await dialogCtx.Continue();
        if (!context.Responded)
        {
            // Call a helper function that identifies if the user says something 
            // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add            
            if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
            { 
                var dialogArgs = new Dictionary<string, object>
                {
                    ["first"] = first,
                    ["second"] = second
                };                        
                await dialogCtx.Begin("addTwoNumbers", dialogArgs);
            }
            else
            {
                // Echo back to the user whatever they typed.
                await context.SendActivity($"Turn: {state.TurnCount}. You said '{context.Activity.Text}'");
            }
        }
    }
}
```

봇 클래스에 도우미 함수를 추가합니다. 도우미 함수는 단일 정규식을 사용하여 사용자의 메시지가 2개의 숫자를 더하라는 요청인지를 감지합니다.

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number, 
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7. 
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";
    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";
    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;
    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);
    var succeeded = false;
    first = 0;
    second = 0;
    if (matches.Count == 0)
    {
        succeeded = false;
    }
    else
    {
        var matched = matches[0];
        if ( System.Double.TryParse(matched.Groups[1].Value, out first) 
            && System.Double.TryParse(matched.Groups[2].Value, out second))
        {
            succeeded = true;
        } 
    }
    return succeeded;
}
```

EchoBot 템플릿을 사용하는 경우 **EchoState.cs**의 `EchoState` 클래스를 다음과 같이 수정합니다.

```cs
/// <summary>
/// Class for storing conversation state.
/// This bot only stores the turn count in order to echo it to the user
/// </summary>
public class EchoState: Dictionary<string, object>
{
    private const string TurnCountKey = "TurnCount";
    public EchoState()
    {
        this[TurnCountKey] = 0;            
    }

    public int TurnCount
    {
        get { return (int)this[TurnCountKey]; }
        set { this[TurnCountKey] = value; }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

[Bot Builder SDK v4를 사용하여 봇 만들기](../javascript/bot-builder-javascript-quickstart.md)에 설명된 JS 템플릿으로 시작합니다. **app.js**에서 `botbuilder-dialogs`를 요구하는 문을 추가합니다.
```js
const {DialogSet} = require('botbuilder-dialogs');
```

**app.js**에서 `dialogs` 집합에 속하는 `addTwoNumbers`라는 단순 다이얼로그를 정의하는 다음 코드를 추가합니다.

```javascript
const dialogs = new DialogSet("myDialogStack");

// Show the sum of two numbers.
dialogs.add('addTwoNumbers', [async function (dc, numbers){
        var sum = Number.parseFloat(numbers[0]) + Number.parseFloat(numbers[1]);
        await dc.context.sendActivity(`${numbers[0]} + ${numbers[1]} = ${sum}`);
        await dc.end();
    }]
);
```

수신 활동을 처리하기 위한 **app.js**의 모드를 다음으로 바꿉니다. 봇은 수신 메시지가 2개의 숫자를 더하라는 요청과 비슷한지 확인하기 위한 도우미 함수를 호출합니다. 비슷한 경우 인수의 숫자를 `DialogContext.begin`에 전달합니다.

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';
        if (isMessage) {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // create a dialog context
            const dc = dialogs.createContext(context, state);

            // MatchesAdd2Numbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await MatchesAdd2Numbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {    
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`); 
            }           
        } else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "what's 1+2?"`);
            }
        }
    });
});
```

**app.js**에 도우미 함수를 추가합니다. 도우미 함수는 단일 정규식을 사용하여 사용자의 메시지가 2개의 숫자를 더하라는 요청인지를 감지합니다. 정규식이 일치하는 경우 추가할 숫자를 포함하는 배열을 반환합니다.

```javascript
async function MatchesAdd2Numbers(message) {
    const ADD_NUMBERS_REGEXP = /([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))(?:\s*)\+(?:\s*)([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))/i;
    let matched = ADD_NUMBERS_REGEXP.exec(message);
    if (!matched) {
        // message wasn't a request to add 2 numbers
        return null;
    }
    else {
        var numbers = [matched[1], matched[2]];
        return numbers;
    }
}
```

---

### <a name="run-the-bot"></a>봇 실행

Bot Framework Emulator에서 봇을 실행하고, "what's 1+1?"과 같이 말합니다.

![봇 실행](./media/how-to-dialogs/bot-output-add-numbers.png)



## <a name="using-dialogs-to-guide-the-user-through-steps"></a>다이얼로그를 사용하여 사용자에게 단계 안내

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="create-a-composite-dialog"></a>복합 다이얼로그 만들기

다음 코드 조각은 botbuilder-dotnet 리포지토리의 [Microsoft.Bot.Samples.Dialog.Prompts](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/MIcrosoft.Bot.Samples.Dialog.Prompts) 코드 샘플에서 가져온 것입니다.

Startup.cs:
1.  봇 이름을 `DialogContainerBot`으로 바꿉니다.
1.  봇의 대화 상태에 대한 속성 모음으로 단순 사전을 사용합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<DialogContainerBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new ConversationState<Dictionary<string, object>>(new MemoryStorage()));
    });
}
```

`EchoBot` 이름을 `DialogContainerBot`으로 바꿉니다.

`DialogContainerBot.cs`에서 프로필 다이얼로그에 대한 클래스를 정의합니다.

```csharp
public class ProfileControl : DialogContainer
{
    public ProfileControl()
        : base("fillProfile")
    {
        Dialogs.Add("fillProfile", 
            new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State = new Dictionary<string, object>();
                    await dc.Prompt("textPrompt", "What's your name?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["name"] = args["Value"];
                    await dc.Prompt("textPrompt", "What's your phone number?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["phone"] = args["Value"];
                    await dc.End(dc.ActiveDialog.State);
                }
            }
        );
        Dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
    }
}
```

그런 다음, 봇 정의 내에서 봇의 기본 다이얼로그에 대한 필드를 선언하고 봇 구문에서 초기화합니다.
봇의 기본 다이얼로그에는 프로필 다이얼로그가 포함됩니다.

```csharp
private DialogSet _dialogs;

public DialogContainerBot()
{
    _dialogs = new DialogSet();

    _dialogs.Add("getProfile", new ProfileControl());
    _dialogs.Add("firstRun",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                    await dc.Context.SendActivity("Welcome! We need to ask a few questions to get started.");
                    await dc.Begin("getProfile");
            },
            async (dc, args, next) =>
            {
                await dc.Context.SendActivity($"Thanks {args["name"]} I have your phone number as {args["phone"]}!");
                await dc.End();
            }
        }
    );
}
```

봇의 `OnTurn` 메서드:
-   대화가 시작될 때 사용자를 인사합니다.
-   사용자로부터 메시지를 받을 때마다 기본 다이얼로그를 초기화하고 _계속_합니다.

    다이얼로그가 응답을 생성하지 않을 경우 이전에 완료되었거나 아직 시작되지 않았다고 가정하고, 시작할 집합에 다이얼로그 이름을 지정하여 _시작_합니다.

```csharp
public async Task OnTurn(ITurnContext turnContext)
{
    try
    {
        switch (turnContext.Activity.Type)
        {
            case ActivityTypes.ConversationUpdate:
                foreach (var newMember in turnContext.Activity.MembersAdded)
                {
                    if (newMember.Id != turnContext.Activity.Recipient.Id)
                    {
                        await turnContext.SendActivity("Hello and welcome to the Composite Control bot.");
                    }
                }
                break;

            case ActivityTypes.Message:
                var state = ConversationState<Dictionary<string, object>>.Get(turnContext);
                var dc = _dialogs.CreateContext(turnContext, state);

                await dc.Continue();

                if (!turnContext.Responded)
                {
                    await dc.Begin("firstRun");
                }

                break;
        }
    }
    catch (Exception e)
    {
        await turnContext.SendActivity($"Exception: {e.Message}");
    }
}

```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="create-a-dialog-with-waterfall-steps"></a>폭포형 단계를 사용하여 다이얼로그 만들기

대화는 사용자와 봇 간에 교환되는 일련의 메시지로 구성됩니다. 봇의 목표가 사용자를 일련의 단계로 안내하는 것이면, **폭포형** 모델을 사용하여 대화의 단계를 정의할 수 있습니다.

**폭포**는 사용자로부터 정보를 수집하거나 사용자에게 일련의 태스크를 안내하는 데 가장 일반적으로 사용되는 특정 방식의 다이얼로그 구현입니다. 태스크는 첫 번째 함수의 결과가 다음 함수의 인수로 전달되는 방식으로 계속 진행되는 함수 배열로 구현됩니다. 각 함수는 일반적으로 전체 프로세스에서 하나의 단계를 나타냅니다. 각 단계에 대해 봇은 [사용자에게 입력을 프롬프트하고](bot-builder-prompts.md), 응답을 기다린 후 결과를 다음 단계로 전달합니다.

예를 들어, 다음 코드 샘플은 세 단계로 이루어진 **폭포**를 나타내는 배열로 3개의 함수를 정의합니다.

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, results){
        var workPlace = results;
        await dc.context.sendActivity(`${workPlace} is a fun place.`);
        await dc.end(); // Ends the dialog
    }
]);

dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```

**폭포**형 단계에 대한 시그니처는 다음과 같습니다.

| 매개 변수 | 설명 |
| ---- | ----- |
| `context` | 다이얼로그 컨텍스트입니다. |
| `args` | 선택적 항목으로, 단계에 전달되는 인수를 포함합니다. |
| `next` | 선택적 항목으로, 폭포의 다음 단계로 계속 진행할 수 있도록 하는 메서드입니다. 이 메서드를 호출할 때 *args* 인수를 제공하여 폭포의 다음 단계로 인수를 전달하도록 할 수 있습니다. |

각 단계는 반환 전에 *next()*, *dialogs.prompt()*, *dialogs.end()*, *dialogs.begin()* 또는 *Promise.resolve()* 메서드 중 하나를 호출해야 합니다. 그렇지 않으면 봇이 해당 단계에서 중단될 수 있습니다. 즉, 함수가 이러한 메서드 중 하나를 반환하지 않으면 사용자가 입력을 하여 봇에 메시지를 보낼 때마다 이 단계가 다시 실행됩니다.

폭포 끝에 도달하면 다이얼로그가 스택에서 사라질 수 있도록 `end()` 메서드를 사용하여 반환하는 것이 가장 좋습니다. 자세한 내용은 [다이얼로그 종료](#end-a-dialog) 섹션을 참조하세요. 마찬가지로, 한 단계에서 다음 단계를 계속 진행하려면 폭포형 단계가 프롬프트로 끝나거나 폭포를 계속 진행하기 위한 `next()` 함수를 명시적으로 호출해야 합니다. 

### <a name="start-a-dialog"></a>다이얼로그 시작

다이얼로그를 시작하려면 시작하려는 *dialogId*를 `begin()`, `prompt()` 또는 `replace()` 메서드에 전달합니다. **replace** 메서드는 현재 다이얼로그를 스택에서 삭제하고 대체 다이얼로그를 스택에 푸시하지만, **begin** 메서드는 스택 위에 다이얼로그를 푸시합니다.

인수 없이 다이얼로그를 시작하려면 다음을 입력합니다.

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

인수를 사용하여 다이얼로그르 시작하려면 다음을 입력합니다.

```javascript
// Start the 'greetings' dialog with the 'userName' passed in. 
await dc.begin('greetings', userName);
```

**프롬프트** 다이얼로그를 시작하려면 다음을 입력합니다.

```javascript
// Start a 'choicePrompt' dialog with choices passed in as an array of colors to choose from.
await dc.prompt('choicePrompt', `choice: select a color`, ['red', 'green', 'blue']);
```

시작하는 프롬프트 형식에 따라, 프롬프트의 인수 시그니처가 다를 수 있습니다. **DialogSet.prompt** 메서드는 도우미 메서드입니다. 이 메서드는 인수를 사용하고, 프롬프트에 적절한 옵션을 생성합니다. 그런 후 **begin** 메서드를 호출하여 프롬프트 다이얼로그를 시작합니다.

스택의 다이얼로그를 바꾸려면:

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); // Can optionally passed in an 'args' as the second argument.
```

**replace()** 사용 방법에 대한 자세한 내용은 아래의 [다이얼로그 반복](#repeat-a-dialog) 및 [다이얼로그 루프](#dialog-loops) 섹션을 참조하세요.

## <a name="end-a-dialog"></a>다이얼로그 종료

스택에서 삭제하여 다이얼로그를 종료하고, 선택적 결과를 부모 다이얼로그에 반환합니다. 부모 다이얼로그의 **Dialog.resume()** 메서드가 반환된 결과를 사용해서 호출됩니다.

다이얼로그가 끝날 때 `end()` 메서드를 명시적으로 호출하는 것이 가장 바람직하지만, 폭포 끝에 도달할 때 스택에서 다이얼로그가 자동으로 삭제되므로 이러한 작업이 반드시 필요한 것은 아닙니다.

다이얼로그를 종료하려면 다음을 입력합니다.

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

선택적 인수를 부모 다이얼로그에 전달하고 다이얼로그를 종료하려면 다음을 입력합니다.

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end(result);
```

또는 해결된 프라미스를 반환하여 다이얼로그를 종료할 수도 있습니다.

```javascript
await Promise.resolve();
```

`Promise.resolve()`를 호출하면 다이얼로그가 끝나고 스택에서 삭제됩니다. 그러나 이 메서드는 부모 다이얼로그를 호출하여 실행을 계속하지 않습니다. `Promise.resolve()` 호출 후에 실행이 중지되고, 사용자가 봇에 메시지를 전송하면 봇은 부모 다이얼로그가 중단되었던 위치에서 다시 시작합니다. 다이얼로그를 종료하는 것이 이상적인 사용자 환경이 아닐 수 있습니다. 봇이 사용자와 계속 상호 작용할 수 있도록 `end()` 또는 `replace()`를 사용하여 다이얼로그를 종료하는 것이 좋습니다.

### <a name="clear-the-dialog-stack"></a>다이얼로그 스택 지우기

모든 다이얼로그를 스택에서 삭제하려면 `dc.endAll()` 메서드를 호출하여 다이얼로그 스택을 지울 수 있습니다.

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

### <a name="repeat-a-dialog"></a>다이얼로그 반복

다이얼로그를 반복하려면 `dialogs.replace()` 메서드를 사용합니다.

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); 
```

기본적으로 기본 메뉴를 표시하려는 경우 다음 단계를 사용하여 `mainMenu` 다이얼로그를 만들 수 있습니다.

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected.
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
        else {
            // Repeat the menu
            await dc.replace('mainMenu');
        }
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);

dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
```

이 다이얼로그는 `ChoicePrompt`를 사용하여 메뉴를 표시하고, 사용자가 옵션을 선택할 때까지 기다립니다. 사용자가 `Order Dinner` 또는 `Reserve a table`을 선택하면 적절한 선택을 위한 다이얼로그를 시작하고, 태스크가 완료되면 마지막 단계에서 다이얼로그를 끝내지 않고 이 다이얼로그를 자동으로 반복합니다.

### <a name="dialog-loops"></a>다이얼로그 루프

`replace()` 메서드를 사용하는 또 다른 방법은 루프를 에뮬레이트하는 것입니다. 예를 들어 다음 시나리오를 살펴보겠습니다. 사용자가 하나의 장바구니에 여러 메뉴 항목을 추가할 수 있도록 하려면 사용자가 주문을 완료할 때까지 메뉴 선택을 반복할 수 있습니다.

```javascript
// Order dinner:
// Help user order dinner from a menu

var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
        "More info", "Process order", "Cancel", "Help"],
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

// The order cart
var orderCart = {
    orders: [],
    total: 0,
    clear: function(dc) {
        this.orders = [];
        this.total = 0;
        dc.context.activity.conversation.orderCart = null;
    }
};

dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        orderCart.clear(dc); // Clears the cart.

        await dc.begin('orderPrompt'); // Prompt for orders
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

// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                // ...
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

// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());

```

위의 샘플 코드에서는 기본 `orderDinner` 다이얼로그가 `orderPrompt`라는 도우미 다이얼로그를 사용하여 사용자 선택을 처리하는 과정을 보여 줍니다. `orderPrompt` 다이얼로그가 메뉴를 표시하고, 사용자에게 품목을 선택하도록 요구하고, 품목을 장바구니에 추가한 후 다시 프롬프트합니다. 이렇게 하면 사용자가 주문에 여러 품목을 추가할 수 있습니다. 사용자가 `Process order` 또는 `Cancel`을 선택할 때까지 다이얼로그가 반복됩니다. 특정 시점에서 실행이 부모 다이얼로그(예: `orderDinner`)로 반환됩니다. `orderDinner` 다이얼로그는 사용자가 주문을 처리하기 원할 경우 막판 조정 작업을 수행하고, 그렇지 않은 경우 종료되고 실행을 부모 다이얼로그(예: `mainMenu`)로 반환합니다. 그런 후 `mainMenu` 다이얼로그가 기본 메뉴 선택 항목을 다시 표시하는 마지막 단계를 계속 실행합니다.

---

## <a name="next-steps"></a>다음 단계

지금까지 **다이얼로그**, **프롬프트** 및 **폭포**를 사용하여 대화 흐름을 관리하는 방법을 알아보았으므로, 기본 봇 논리의 `dialogs` 개체에서 한꺼번에 사용하는 대신, 다이얼로그를 모듈식 태스크로 분할하는 방법을 알아보겠습니다.

> [!div class="nextstepaction"]
> [복합 컨트롤을 사용하여 모듈식 봇 논리 만들기](bot-builder-compositcontrol.md)