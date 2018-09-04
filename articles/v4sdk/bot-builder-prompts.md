---
title: Dailogs 라이브러리를 사용하여 사용자에게 입력 요청 | Microsoft Docs
description: Node.js용 Bot Builder SDK의 대화 상자 라이브러리를 사용하여 사용자에게 입력을 요청하는 방법에 대해 알아봅니다.
keywords: prompts, dialogs, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, reprompt, validation
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b238ed510fd1d6fda82734af373f344b0dc28e3
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905367"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>대화 상자 라이브러리를 사용하여 사용자에게 입력 요청

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇은 종종 사용자에게 제기된 질문을 통해 해당 정보를 수집합니다. 문자열 입력을 요청하려면 [턴 컨텍스트](bot-builder-concept-activity-processing.md#turn-context) 개체의 _보내기 작업_ 메서드를 사용하여 사용자에게 표준 메시지를 보낼 수 있지만, Bot Builder SDK는 다른 유형의 정보를 요청하는 데 사용할 수 있는 **대화 상자** 라이브러리를 제공합니다. 이 항목에서는 **프롬프트**를 사용하여 사용자에게 입력을 요청하는 방법을 설명합니다.

이 문서에서는 대화 상자 내에서 프롬프트를 사용하는 방법을 설명합니다. 일반적으로 대화 상자를 사용하는 방법에 대한 정보는 [대화 상자를 사용하여 간단한 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)를 참조하세요.

## <a name="prompt-types"></a>프롬프트 형식

대화 상자 라이브러리는 각각 다양한 유형의 응답을 요청하는 많은 다양한 유형의 프롬프트를 제공합니다.

| prompt | 설명 |
|:----|:----|
| **AttachmentPrompt** | 문서 또는 이미지 같은 첨부 파일에 대해 사용자에게 확인합니다. |
| **ChoicePrompt** | 옵션 집합에서 선택하도록 사용자에게 확인합니다. |
| **ConfirmPrompt** | 사용자에게 작업을 확인하라는 메시지가 표시됩니다. |
| **DatetimePrompt** | 날짜-시간에 대해 사용자에게 확인합니다. 사용자는 "내일에 오후 8시" 또는 "금요일 오전 10시" 같은 자연어를 사용하여 응답할 수 있습니다. Bot Framework SDK는 LUIS `builtin.datetimeV2` 미리 작성된 엔터티를 사용합니다. 자세한 내용은 [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)를 참조합니다. |
| **NumberPrompt** | 숫자에 대해 사용자에게 확인합니다. 사용자는 "10" 또는 "열"을 사용하여 응답할 수 있습니다. 응답이 "열"인 경우 예를 들어 프롬프트는 응답을 숫자로 변환하고 결과로 `10`을 반환합니다. |
| **TextPrompt** | 텍스트 문자열에 대해 사용자에게 확인합니다. |

## <a name="add-references-to-prompt-library"></a>라이브러리를 확인하려면 참조 추가

봇에 **대화 상자** 패키지를 추가하여 **대화 상자** 라이브러리를 가져올 수 있습니다. [대화 상자를 사용하여 간단한 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)에서 대화 상자를 관리하지만 프롬프트에 대해 대화 상자를 사용합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

NuGet에서 **Microsoft.Bot.Builder.Dialogs** 패키지를 설치합니다.

그런 다음, 봇 코드에서 라이브러리 참조를 포함합니다.

```cs
using Microsoft.Bot.Builder.Dialogs;
```

봇 코드 파일에서 대화 상자를 클래스로 또는 인라인을 속성으로 정의할 수 있습니다.

이 문서의 코드는 클래스로 정의된 대화 상자에 대해 작성되었습니다.
다음 예제에서는 대화 상자의 생성자에 코드를 추가하는 것을 가정합니다.

대화의 기본 흐름은 단계의 컬렉션이며 ID가 지정되어야 합니다. 봇은 이 ID를 사용하여 대화를 검색하므로 이 ID를 상수로 공개하는 것이 좋습니다.

```cs
public class MyDialog : DialogSet
{
    public const string Name = "mainDialog";

    public MyDialog()
    {
        // Define your dialog's prompts and steps here.
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

NPM에서 대화 상자 패키지를 설치합니다.

```cmd
npm install --save botbuilder-dialogs@preview
```

봇에서 **대화 상자**를 사용하려면 이를 봇 코드에 포함시킵니다.

app.js 파일에서 다음을 추가합니다.

```javascript
const {DialogSet} = require("botbuilder-dialogs");
const dialogs = new DialogSet();
```

---

## <a name="prompt-the-user"></a>사용자 확인

입력에 대해 사용자에게 확인하려면 대화에 프롬프트를 추가할 수 있습니다. 예를 들어 **TextPrompt** 형식의 프롬프트를 정의하고 **textPrompt**라는 대화 ID를 지정할 수 있습니다.

프롬프트 대화 상자를 추가하면 간단한 2단계 폭포 대화 상자에서 사용하거나 다단계 폭포 대화 상자에서 여러 프롬프트를 함께 사용할 수 있습니다. *폭포* 대화 상자는 단계의 시퀀스를 정의하기 위한 방법일 뿐입니다. 자세한 내용은 [대화 상자를 사용하여 간단한 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)에서 [대화 상자 사용](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps) 섹션을 참조하세요.

첫 번째로 대화 상자는 사용자의 이름을 확인하며, 두 번째로 대화 상자는 사용자 입력을 프롬프트에 대한 응답으로 처리합니다.

예를 들어 다음 대화 상자는 사용자의 이름을 확인한 다음, 사용자를 이름으로 대합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

대화 상자에서 사용하는 각 프롬프트에는 이름이 지정되며, 대화 상자나 봇이 사용하여 프롬프트에 액세스합니다. 이러한 모든 샘플에서는 프롬프트 ID를 상수로 공개합니다.

대화 컨텍스트의 **프롬프트** 또는 **종료** 메서드에 대한 호출은 대화 단계의 끝을 알립니다. 이러한 명령문이 없으면 대화 상자는 제대로 실행되지 않습니다.

```csharp
/// <summary>Defines a simple greeting dialog that asks for the user's name.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            // Each step takes in a dialog context, arguments, and the next delegate.
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];
                await dc.Context.SendActivity($"Hi {user}!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {TextPrompt} = require("botbuilder-dialogs");
```

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('textPrompt', new TextPrompt());
dialogs.add('greetings', [
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.end();
    }
]);
```

---

> [!NOTE]
> 대화 상자를 시작하려면 대화 컨텍스트를 가져오고 해당 _시작_ 메서드를 사용합니다. 자세한 내용은 [대화 상자를 사용하여 간단한 대화 흐름 관리](./bot-builder-dialog-manage-conversation-flow.md)를 참조합니다.

## <a name="reusable-prompts"></a>다시 사용할 수 있는 프롬프트

동일한 유형의 프롬프트를 사용하여 다양한 정보를 요청하는 데 프롬프트를 다시 사용할 수 있습니다. 예를 들어 위의 샘플 코드는 텍스트 프롬프트를 정의하여 사용자의 이름을 묻는 데 사용했습니다. 예를 들어 원하는 경우 동일한 프롬프트를 사용하여 사용자에게 “직장이 어디세요?” 같은 다른 텍스트 문자열을 물을 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>Defines a simple greeting dialog that asks for the user's name and place of work.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];

                // Ask them where they work.
                await dc.Prompt(Inputs.Text, $"Hi {user}! Where do you work?");
            },
            async(dc, args, next) =>
            {
                var workplace = (string)args["Text"];

                await dc.Context.SendActivity($"{workplace} is a cool place!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);

        // Ask them where they work.
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, workPlace){
        await dc.context.sendActivity(`${workPlace} is a cool place!`);

        await dc.end();
    }
]);
```

---

그러나 프롬프트가 요구하는 예상 값에 프롬프트를 연결하려는 경우 각 프롬프트에 고유 *dialogId*를 지정할 수 있습니다. 고유 ID를 사용하여 대화 상자가 추가됩니다. 다양한 ID를 사용하여 동일한 형식의 여러 **프롬프트** 대화 상자를 만들 수도 있습니다. 예를 들어 위의 예제에 대한 두 개의 **TextPrompt** 대화 상자를 만들 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>The ID of the main dialog in the set.</summary>
public const string Name = "mainDialog";

/// <summary>Defines the IDs of the prompts in the set.</summary>
public struct Inputs
{
    /// <summary>The ID of the name prompt.</summary>
    public const string Name = "namePrompt";

    /// <summary>The ID of the work prompt.</summary>
    public const string Work = "workPrompt";
}

/// <summary>Defines the prompts and steps of the dialog.</summary>
public MyDialog()
{
    Add(Inputs.Name, new TextPrompt());
    Add(Inputs.Work, new TextPrompt());
    Add(Name, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the user's name.
            await dc.Prompt(Inputs.Name, "What is your name?");
        },
        async(dc, args, next) =>
        {
            var user = (string)args["Text"];

            // Ask them where they work.
            await dc.Prompt(Inputs.Work, $"Hi {user}! Where do you work?");
        },
        async(dc, args, next) =>
        {
            var workplace = (string)args["Text"];

            await dc.Context.SendActivity($"{workplace} is a cool place!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
dialogs.add('namePrompt', new TextPrompt());
dialogs.add('workPlacePrompt', new TextPrompt());
```

---

코드 재사용을 위해 단일 `textPrompt`의 정의는 텍스트 문자열을 응답으로 요청하므로 이러한 모든 메시지에 대해 작동합니다. 그러나 대화 명명 기능이 유용한 경우는 프롬프트 입력의 유효성을 검사해야 할 때입니다. 이 경우에 프롬프트는 **TextPrompt**를 사용할 수 있지만 각 프롬프트마다 다른 값 집합을 찾습니다. `NumberPrompt`를 사용하여 프롬프트 답변의 유효성을 검사할 수 있는 방법에 대해 살펴보겠습니다.

## <a name="specify-prompt-options"></a>프롬프트 옵션 지정

대화 단계 내에서 프롬프트를 사용하는 경우 재프롬프트 문자열과 같은 프롬프트 옵션을 제공할 수도 있습니다.

재프롬프트 문자열을 지정하면 사용자 입력이 프롬프트를 충족하지 못하는 경우 유용합니다. 이는 숫자 프롬프트에 대한 “내일”과 같이 프롬프트가 구문 분석을 할 수 없는 형식이거나, 입력이 유효성 검사 조건을 충족시키지 못했기 때문입니다.

> [!TIP]
> 숫자 프롬프트를 만들 때 사용할 입력 문화권을 지정해야 합니다. 숫자 프롬프트는 "열둘" 또는 "1/4", "12" 및 "0.25" 같은 다양한 입력을 해석할 수 있습니다. 입력 문화권은 프롬프트가 사용자 입력을 더 정확하게 해석하도록 도와줍니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

입력 문화권은 추가 라이브러리에서 정의됩니다.

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
```

다음 코드는 기존 대화 집합인 **대화 상자**에 숫자 프롬프트를 추가합니다.

```csharp
dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
```

대화 단계 내에서 다음 코드는 사용자에게 입력을 확인하고 해당 입력이 숫자로 해석될 수 없는 경우 사용할 재프롬프트 문자열을 제공합니다.

```csharp
await dc.Prompt("numberPrompt", "How many people are in your party?", new PromptOptions()
{
    RetryPromptString = "Sorry, please specify the number of people in your party."
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {NumberPrompt} = require("botbuilder-dialogs");
```

```javascript
await dc.prompt('numberPrompt', 'How many people in your party?', { retryPrompt: `Sorry, please specify the number of people in your party.` })
```

```javascript
dialogs.add('numberPrompt', new NumberPrompt());
```

---

특히 선택 프롬프트는 일부 추가 정보인 사용자가 사용할 수 있는 선택 목록을 요구합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

이 예제에서는 다음 네임스페이스의 형식을 사용합니다.

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
```


사용자에게 옵션 집합 중에서 선택하도록 요구하기 위해 **ChoicePrompt**를 사용하는 경우 **ChoicePromptOptions** 개체 내에서 제공된 해당 옵션 집합을 프롬프트에 제공해야 합니다. 여기에서 **ChoiceFactory**를 사용하여 옵션 목록을 적절한 형식으로 변환합니다.

또한 다시 사용자에게 입력 옵션을 제공하는 방법으로 **SuggestedActions** 작업을 재프롬프트로 사용합니다.


```csharp
/// <summary>Defines a dialog that asks for a choice of color.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the color prompt.</summary>
        public const string Color = "colorPrompt";
    }

    /// <summary>The available colors to choose from.</summary>
    public List<string> Colors = new List<string> { "Green", "Blue" };

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Color, new ChoicePrompt(Culture.English));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for a color. A choice prompt requires that you specify choice options.
                await dc.Prompt(Inputs.Color, "Please make a choice.", new ChoicePromptOptions()
                {
                    Choices = ChoiceFactory.ToChoices(Colors),
                    RetryPromptActivity =
                        MessageFactory.SuggestedActions(Colors, "Please choose a color.") as Activity
                });
            },
            async(dc, args, next) =>
            {
                var color = (FoundChoice)args["Value"];

                await dc.Context.SendActivity($"You chose {color.Value}.");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ChoicePrompt} = require("botbuilder-dialogs");
```

```javascript
dialogs.add('choicePrompt', new ChoicePrompt());
```

```javascript
// A choice prompt requires that you specify choice options.
const list = ['green', 'blue'];
await dc.prompt('choicePrompt', 'Please make a choice', list, {retryPrompt: 'Please choose a color.'});
```

---

## <a name="validate-a-prompt-response"></a>프롬프트 응답의 유효성 검사

올바른 값을 **폭포**의 다음 단계에 반환하기 전에 프롬프트 응답의 유효성을 검사할 수 있습니다. 예를 들어 **6**과 **20** 사이의 숫자 범위 내에서 **NumberPrompt**의 유효성을 검사하려면 이와 비슷한 유효성 검사 논리가 있을 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Defines a dialog that asks for the number of people in a party.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the party size prompt.</summary>
        public const string Size = "parytySize";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        // Include a validation function for the party size prompt.
        Add(Inputs.Size, new NumberPrompt<int>(Culture.English, async (context, result) =>
        {
            if (result.Value < 6 || result.Value > 20)
            {
                result.Status = PromptStatus.OutOfRange;
            }
        }));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the party size.
                await dc.Prompt(Inputs.Size, "How many people are in your party?", new PromptOptions()
                {
                    RetryPromptString = "Please specify party size between 6 and 20."
                });
            },
            async(dc, args, next) =>
            {
                var size = (int)args["Value"];

                await dc.Context.SendActivity($"Okay, {size} people!");
                await dc.End();
            }
        });
    }
}
```

유효성 검사는 자체 개인 메서드 내에서 캡슐화되고 해당 방식으로 추가될 수 있습니다.

```cs
/// <summary>Validates input for the partySize prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task PartySizeValidator(ITurnContext context, Int32Result result)
{
    if (result.Value < 6 || result.Value > 20)
    {
        result.Status = PromptStatus.OutOfRange;
    }
}
```

대화 상자 내에서 입력의 유효성을 검사하는 데 사용할 메서드를 지정합니다.

```cs
// Include a validation function for the party size prompt.
Add(Inputs.Size, new NumberPrompt<int>(Culture.English, PartySizeValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Customized prompts with validations
// A number prompt with validation for valid party size within a range.
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt( async (context, value) => {
    try {
        if(value < 6 ){
            throw new Error('Party size too small.');
        }
        else if(value > 20){
            throw new Error('Party size too big.')
        }
        else {
            return value; // Return the valid value
        }
    } catch (err) {
        await context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
        return undefined;
    }
}));
```

---

마찬가지로, 향후 시간 및 날짜에 대한 **DatetimePrompt** 응답의 유효성을 검사하려는 경우 다음과 비슷한 유효성 검사 논리가 있을 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using DateTimeResult = Microsoft.Bot.Builder.Prompts.DateTimeResult;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Validates input for the reservationTime prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task TimeValidator(ITurnContext context, DateTimeResult result)
{
    if (result.Resolution.Count == 0)
    {
        await context.SendActivity("Sorry, I did not recognize the time that you entered.");
        result.Status = PromptStatus.NotRecognized;
    }

    // Find any recognized time that is not in the past.
    var now = DateTime.Now;
    DateTime time = default(DateTime);
    var resolution = result.Resolution.FirstOrDefault(
        res => DateTime.TryParse(res.Value, out time) && time > now);

    if (resolution != null)
    {
        // If found, keep only that result.
        result.Resolution.Clear();
        result.Resolution.Add(resolution);
    }
    else
    {
        // Otherwise, flag the input as out of range.
        await context.SendActivity("Please enter a time in the future, such as \"tomorrow at 9am\"");
        result.Status = PromptStatus.OutOfRange;
    }
}
```

```csharp
Add(Inputs.Time, new DateTimePrompt(Culture.English, TimeValidator));
```

추가 예제는 [리포지토리 샘플](https://github.com/Microsoft/botbuilder-dotnet)에서 찾을 수 있습니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```JavaScript
// A date and time prompt with validation for date/time in the future.
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt( async (context, values) => {
    try {
        if (values.length < 0) { throw new Error('missing time') }
        if (values[0].type !== 'datetime') { throw new Error('unsupported type') }
        const value = new Date(values[0].value);
        if (value.getTime() < new Date().getTime()) { throw new Error('in the past') }
        return value;
    } catch (err) {
        await context.sendActivity(`Please enter a valid time in the future like "tomorrow at 9am".`);
        return undefined;
    }
}));
```

추가 예제는 [리포지토리 샘플](https://github.com/Microsoft/botbuilder-js)에서 찾을 수 있습니다.

---

> [!TIP]
> 날짜/시간 프롬프트는 사용자가 모호한 답변을 하는 경우 몇몇 다른 날짜를 확인할 수 있습니다. 사용 목적에 따라 첫 번째 해결책이 아닌 프롬프트 결과에서 제공하는 모든 해결책을 확인하고 싶을 수 있습니다.

비슷한 기법을 사용하여 프롬프트 형식 중 하나에 대한 프롬프트 응답의 유효성을 검사할 수 있습니다.

## <a name="save-user-data"></a>사용자 데이터 저장

사용자 입력에 대해 확인하는 경우 해당 입력을 처리하는 방법에 대해 여러 옵션이 있습니다. 예를 들어 입력을 사용하고 버릴 수 있으며, 글로벌 변수에 저장할 수 있고, 일시적 또는 메모리 내 저장소 컨테이너에 저장할 수 있고, 파일에 저장할 수 있거나 외부 데이터베이스에 저장할 수 있습니다. 사용자 데이터를 저장하는 방법에 대한 자세한 내용은 [사용자 데이터 관리](bot-builder-howto-v4-state.md)를 참조하세요.

## <a name="next-steps"></a>다음 단계

사용자에게 입력을 확인하는 방법에 대해 배웠으므로 대화 상자를 통해 다양한 대화 흐름을 관리하여 봇 코드 및 사용자 환경을 향상시켜보겠습니다.


