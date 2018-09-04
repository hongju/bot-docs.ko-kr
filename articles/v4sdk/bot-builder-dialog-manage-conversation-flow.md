---
title: 대화 상자를 사용하여 간단한 대화 흐름 관리 | Microsoft Docs
description: Node.js용 Bot Builder SDK에서 대화 상자를 사용하여 간단한 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 간단한 대화 흐름, 대화 상자, 프롬프트, 폭포, 대화 상자 집합
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 8/2/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77162601f542e6faa8908bc71abc971eb99fcc93
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756562"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>대화 상자를 사용하여 간단한 대화 흐름 관리

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

다이얼로그 라이브러리를 사용하여 간단하거나 복잡한 대화 흐름을 모두 관리할 수 있습니다. 간단한 대화 흐름의 경우 사용자가 *폭포*의 첫 번째 단계에서 시작하여 마지막 단계까지 계속 진행함으로써 대화형 교환을 완료합니다. 대화 상자는 또한 [복잡한 대화 흐름](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)도 처리할 수 있습니다. 여기서 대화 상자의 일부가 분기되고 반복될 수 있습니다.

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

<!-- TODO: This paragraph belongs in a conceptual topic. --> 대화 상자는 프로그램의 함수와 같습니다. 일반적으로 특정 작업을 특정 순서로 수행하도록 디자인되었으며 필요한 만큼 호출할 수 있습니다. 대화 상자를 사용하면 봇 개발자가 대화 흐름을 안내할 수 있습니다. 여러 개의 다이얼로그를 함께 연결하여 봇이 처리할 대화 흐름을 처리할 수 있습니다. Bot Builder SDK의 **대화 상자** 라이브러리에는 대화 흐름을 관리하는 데 도움이 되는 _프롬프트_ 및 _폭포 대화 상자_와 같은 기본 제공 기능이 포함됩니다. 프롬프트를 사용하여 사용자에게 다른 유형의 정보를 요청할 수 있습니다. 폭포를 사용하여 여러 단계를 하나의 시퀀스로 결합할 수 있습니다.

이 문서에서는 _대화 상자 집합_을 사용하여 프롬프트 및 폭포 단계가 모두 포함된 대화 흐름을 만듭니다. 두 개의 예제 대화 상자가 있습니다. 첫 번째는 사용자 입력이 필요 없는 작업을 수행하는 1단계 대화 상자입니다. 두 번째는 사용자에게 몇 가지 정보를 묻는 다중 단계 대화 상자입니다.

## <a name="install-the-dialogs-library"></a>다이얼로그 라이브러리 설치

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
npm install --save botbuilder-dialogs@preview
```

봇에서 **대화 상자**를 사용하려면 **app.js** 파일에 이 명령을 포함합니다.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

## <a name="create-a-dialog-stack"></a>다이얼로그 스택 만들기

이 첫 번째 예제에서는 두 숫자를 함께 추가하고 결과를 표시할 수 있는 1단계 대화 상자를 만듭니다.

다이얼로그를 사용하려면 먼저 *다이얼로그 집합*을 만들어야 합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Microsoft.Bot.Builder.Dialogs` 라이브러리는 `DialogSet` 클래스를 제공합니다.
**AdditionDialog** 클래스를 만들고, 필요한 using 문을 추가합니다.
대화 상자 집합에 명명된 대화 상자 및 대화 상자 집합을 추가한 다음, 나중에 이름으로 액세스할 수 있습니다.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

**DialogSet**에서 클래스를 파생시키고, 이 대화 상자 집합에 대한 대화 상자 및 입력 정보를 식별하는 데 사용할 ID와 키를 정의합니다.

```csharp
/// <summary>Defines a simple dialog for adding two numbers together.</summary>
public class AdditionDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Main = "additionDialog";

    /// <summary>Defines the IDs of the input arguments.</summary>
    public struct Inputs
    {
        public const string First = "first";
        public const string Second = "second";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

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

다이얼로그 이름(예: `addTwoNumbers`)은 각 다이얼로그 집합 내에서 고유해야 합니다. 각 집합 내에서 필요한 수만큼 다이얼로그를 정의할 수 있습니다. 여러 대화 상자 집합을 만든 후 모두 원활하게 작동하도록 하려면 [모듈식 봇 논리 만들기](bot-builder-compositcontrol.md)를 참조하세요.

다이얼로그 라이브러리는 다음 다이얼로그를 정의합니다.

* **프롬프트** 다이얼로그: 다이얼로그가 사용자에게 입력을 프롬프트하는 함수와 입력을 처리하는 함수를 포함하는 2개 이상의 함수를 사용합니다. **폭포형** 모델을 사용하여 이러한 함수를 하나로 연결할 수 있습니다.
* **폭포** 다이얼로그는 순서대로 실행되는 _폭포형 단계_ 시퀀스를 정의합니다. 폭포형 다이얼로그는 단일 단계를 포함할 수 있으며 이 경우 간단한 1단계 다이얼로그로 간주될 수 있습니다.

## <a name="create-a-single-step-dialog"></a>단일 단계 다이얼로그 만들기

단일 단계 다이얼로그는 단일 선회 대화형 흐름을 캡처하는 데 유용할 수 있습니다. 이 예제에서는 사용자가 "1 + 2"와 같이 말하는지 여부를 검색하고 `addTwoNumbers` 다이얼로그를 시작하여 "1 + 2 = 3"으로 회신하는 봇을 만듭니다다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

값은 `IDictionary<string,object>` 속성 모음으로서 다이얼로그에 전달되고 다이얼로그에서 반환됩니다.

다이얼로그 집합 내에 간단한 다이얼로그를 만들려면 `Add` 메서드를 사용합니다. 다음에서는 `addTwoNumbers`라는 1단계 폭포를 추가합니다.

이 단계에서는 전달되는 다이얼로그 인수가 추가할 숫자를 나타내는 `first` 및 `second` 속성을 포함한다고 가정합니다.

**AdditionDialog** 클래스에 다음 생성자를 추가합니다.

```csharp
/// <summary>Defines the steps of the dialog.</summary>
public AdditionDialog()
{
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Get the input from the arguments to the dialog and add them.
            var x =(double)args[Inputs.First];
            var y =(double)args[Inputs.Second];
            var sum = x + y;

            // Display the result to the user.
            await dc.Context.SendActivity($"{x} + {y} = {sum}");

            // End the dialog.
            await dc.End();
        }
    });
}
```

### <a name="pass-arguments-to-the-dialog"></a>다이얼로그에 인수 전달

봇 코드에서 Using 문을 업데이트합니다.

```cs
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
```

추가 대화 상자의 클래스에 정적 속성을 추가합니다.

```cs
private static AdditionDialog AddTwoNumbers { get; } = new AdditionDialog();
```

봇의 `OnTurn` 메서드 내에서 다이얼로그를 호출하려면 다음을 포함하도록 `OnTurn`을 수정합니다.

```cs
public async Task OnTurn(ITurnContext context)
{
    // Handle any message activity from the user.
    if (context.Activity.Type is ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var conversationState = context.GetConversationState<ConversationData>();

        // Generate a dialog context for the addition dialog.
        var dc = AddTwoNumbers.CreateContext(context, conversationState.DialogState);

        // Call a helper function that identifies if the user says something
        // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add.
        if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
        {
            // Start the dialog, passing in the numbers to add.
            var args = new Dictionary<string, object>
            {
                [AdditionDialog.Inputs.First] = first,
                [AdditionDialog.Inputs.Second] = second
            };
            await dc.Begin(AdditionDialog.Main, args);
        }
        else
        {
            // Echo back to the user whatever they typed.
            await context.SendActivity($"You said '{context.Activity.Text}'");
        }
    }
}
```

봇 클래스에 **TryParseAddingTwoNumbers** 도우미 함수를 추가합니다. 도우미 함수는 단일 정규식을 사용하여 사용자의 메시지가 2개의 숫자를 더하라는 요청인지를 감지합니다.

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number,
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7.
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public static bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";

    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";

    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;

    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);

    first = 0;
    second = 0;
    if (matches.Count > 0)
    {
        var matched = matches[0];
        if (double.TryParse(matched.Groups[1].Value, out first)
            && double.TryParse(matched.Groups[2].Value, out second))
        {
            return true;
        }
    }
    return false;
}
```

EchoBot 템플릿을 사용하는 경우 **EchoState** 클래스의 이름을 **ConversationData**로 변경하고 다음을 포함하도록 수정합니다.

```cs
using System.Collections.Generic;

/// <summary>
/// Class for storing conversation state.
/// </summary>
public class ConversationData
{
    /// <summary>Property for storing dialog state.</summary>
    public Dictionary<string, object> DialogState { get; set; } = new Dictionary<string, object>();
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

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
        // State will store all of your information
        const convoState = conversationState.get(context);
        const dc = dialogs.createContext(context, convoState);

        if (isMessage) {
            // TryParseAddingTwoNumbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await TryParseAddingTwoNumbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                const count = (convoState.count === undefined ? convoState.count = 0 : ++convoState.count);
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`);
            }
        }
        else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }

        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "What's 2+3?"`);
            }
        }
    });
});

```

**app.js**에 도우미 함수를 추가합니다. 도우미 함수는 단일 정규식을 사용하여 사용자의 메시지가 2개의 숫자를 더하라는 요청인지를 감지합니다. 정규식이 일치하는 경우 추가할 숫자를 포함하는 배열을 반환합니다.

```javascript
async function TryParseAddingTwoNumbers(message) {
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

다음 예제에서는 사용자에게 정보를 묻는 다중 단계 대화 상자를 만듭니다.

### <a name="create-a-dialog-with-waterfall-steps"></a>폭포형 단계를 사용하여 다이얼로그 만들기

**폭포**는 사용자로부터 정보를 수집하거나 사용자에게 일련의 태스크를 안내하는 데 가장 일반적으로 사용되는 특정 방식의 다이얼로그 구현입니다. 태스크는 첫 번째 함수의 결과가 다음 함수의 인수로 전달되는 방식으로 계속 진행되는 함수 배열로 구현됩니다. 각 함수는 일반적으로 전체 프로세스에서 하나의 단계를 나타냅니다. 각 단계에 대해 봇은 [사용자에게 입력을 프롬프트하고](bot-builder-prompts.md), 응답을 기다린 후 결과를 다음 단계로 전달합니다.

예를 들어, 다음 코드 샘플은 세 단계로 이루어진 **폭포**를 나타내는 배열로 세 개의 함수를 정의합니다. 각 프롬프트 후 봇은 사용자의 입력을 인식하지만 입력 내용은 저장하지 않습니다. 사용자 입력을 유지하려면 [사용자 데이터 유지](bot-builder-tutorial-persist-user-inputs.md)를 참조하세요.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

이 인사말 대화 상자의 생성자에서는 **GreetingDialog**가 **DialogSet**에서 파생되고, **Inputs.Text**에  **TextPrompt** 개체에 사용 중인 ID가 포함되어 있으며, **Main**에 인사말 대화 상자 자체의 ID가 포함되어 있습니다.

```csharp
public GreetingDialog()
{
    // Include a text prompt.
    Add(Inputs.Text, new TextPrompt());

    // Define the dialog logic for greeting the user.
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Ask for their name.
            await dc.Prompt(Inputs.Text, "What is your name?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var name = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"Hi, {name}!");

            // Ask where they work.
            await dc.Prompt(Inputs.Text, "Where do you work?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var work = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"{work} is a fun place.");

            // End the dialog.
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

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

---

**폭포**형 단계에 대한 시그니처는 다음과 같습니다.

| 매개 변수 | 설명 |
| :---- | :----- |
| `dc` | 다이얼로그 컨텍스트입니다. |
| `args` | 선택 사항으로, 단계에 전달되는 인수를 포함합니다. |
| `next` | 선택 사항으로, 프롬프트 없이 폭포의 다음 단계로 계속 진행할 수 있도록 하는 메서드입니다. 이 메서드를 호출할 때 *args* 인수를 제공할 수 있습니다. 이렇게 하면 폭포의 다음 단계로 인수를 전달할 수 있습니다. |

각 단계는 반환 전에 *next()* 대리자 또는 *begin*, *end*, *prompt* 또는 *replace* 대화 상자 컨텍스트 메서드 중 하나를 호출해야 합니다. 그렇지 않으면 봇이 해당 단계에서 중단될 수 있습니다. 즉, 함수가 이러한 메서드 중 하나로 완료되지 않으면 모든 사용자 입력으로 인해 사용자가 봇에 메시지를 보낼 때마다 이 단계가 다시 실행됩니다.

폭포 끝에 도달하면 대화 상자가 스택에서 사라질 수 있도록 _end_ 메서드를 사용하여 반환하는 것이 가장 좋습니다. 자세한 내용은 아래의 [대화 상자 종료](#end-a-dialog) 섹션을 참조하세요. 마찬가지로, 한 단계에서 다음 단계를 계속 진행하려면 폭포 단계가 프롬프트로 끝나거나 폭포를 계속 진행하기 위한 _next_ 대리자를 명시적으로 호출해야 합니다.

## <a name="start-a-dialog"></a>다이얼로그 시작

대화를 시작하려면 시작하려는 *dialogId*를 대화 상자 컨텍스트의 _begin_, _prompt_ 또는 _replace_ 메서드로 전달합니다. _replace_ 메서드는 현재 대화 상자를 스택에서 삭제하고 대체 대화 상자를 스택에 푸시하지만, _begin_ 메서드는 스택 위에 다이얼로그를 푸시합니다.

인수 없이 다이얼로그를 시작하려면 다음을 입력합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog.
await dc.Begin("greetings");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

---

인수를 사용하여 다이얼로그르 시작하려면 다음을 입력합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog, passing in a property bag.
await dc.Begin("greetings", args);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog with the 'userName' passed in.
await dc.begin('greetings', userName);
```

---

**프롬프트** 다이얼로그를 시작하려면 다음을 입력합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

여기서, **Inputs.Text**에는 동일한 대화 상자 집합에 있는 **TextPrompt**의 ID가 포함됩니다.

```csharp
// Ask a user for their name.
await dc.Prompt(Inputs.Text, "What is your name?");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Ask a user for their name.
await dc.prompt('textPrompt', "What is your name?");
```

---

시작하는 프롬프트 형식에 따라, 프롬프트의 인수 시그니처가 다를 수 있습니다. **DialogSet.prompt** 메서드는 도우미 메서드입니다. 이 메서드는 인수를 사용하고, 프롬프트에 적절한 옵션을 생성합니다. 그런 후 **begin** 메서드를 호출하여 프롬프트 다이얼로그를 시작합니다. 프롬프트에 대한 자세한 내용은 [사용자에게 입력 요청](bot-builder-prompts.md)을 참조하세요.

## <a name="end-a-dialog"></a>다이얼로그 종료

_end_ 메서드는 스택에서 삭제하여 대화 상자를 종료하고, 선택적 결과를 부모 대화 상자에 반환합니다.

대화 상자가 끝날 때 _end_ 메서드를 명시적으로 호출하는 것이 가장 바람직하지만, 폭포 끝에 도달할 때 스택에서 대화 상자가 자동으로 삭제되므로 이러한 작업이 반드시 필요한 것은 아닙니다.

다이얼로그를 종료하려면 다음을 입력합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog by popping it off the stack.
await dc.End();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

---

대화 상자를 종료하고 정보를 부모 대화 상자에 반환하려면 속성 모음 인수를 포함합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and return information to the parent dialog.
await dc.end(new Dictionary<string, object>
    {
        ["property1"] = value1,
        ["property2"] = value2
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end({
    "property1": value1,
    "property2": value2
});
```

---

## <a name="clear-the-dialog-stack"></a>다이얼로그 스택 지우기

모든 대화 상자를 스택에서 삭제하려면 _end all_ 메서드를 호출하여 대화 상자 스택을 지울 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Pop all dialogs from the current stack.
await dc.EndAll();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

---

## <a name="repeat-a-dialog"></a>다이얼로그 반복

대화 상자를 반복하려면 _replace_ 메서드를 사용합니다. 대화 상자 컨텍스트의 *replace* 메서드는 현재 대화 상자를 스택에서 삭제하고 대체 대화 상자를 스택의 맨 위로 푸시하여 해당 대화 상자를 시작합니다. 이는 [복잡한 대화 흐름](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)을 처리하는 좋은 방법이며 메뉴를 관리하는 데 유용한 기술입니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and start the main menu dialog.
await dc.Replace("mainMenu");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu');
```

---

## <a name="next-steps"></a>다음 단계

이제 간단한 대화 흐름을 관리하는 방법을 배웠으므로 _replace_ 메서드를 활용하여 복잡한 대화 흐름을 처리하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [복잡한 대화 흐름 관리](bot-builder-dialog-manage-complex-conversation-flow.md)
