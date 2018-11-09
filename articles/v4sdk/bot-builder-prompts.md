---
title: 대화 상자 라이브러리를 사용하여 사용자 입력 수집 | Microsoft Docs
description: Bot Builder SDK의 대화 상자 라이브러리를 사용하여 사용자에게 입력을 요청하는 방법에 대해 알아봅니다.
keywords: prompts, dialogs, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, reprompt, validation
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9b668ed67c34dcd0f8618852375e684b23e34408
ms.sourcegitcommit: a496714fb72550a743d738702f4f79e254c69d06
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/01/2018
ms.locfileid: "50736691"
---
# <a name="use-dialog-library-to-gather-user-input"></a>대화 상자 라이브러리를 사용하여 사용자 입력 수집

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

질문을 게시하여 정보를 수집하는 것은 봇이 사용자와 상호 작용하는 주요 방법 중 하나입니다. [순서 컨텍스트](~/v4sdk/bot-builder-basics.md#defining-a-turn) 개체의 _보내기 작업_ 메서드를 사용하여 직접 이렇게 한 후 그 다음으로 들어오는 메시지를 응답으로 처리할 수 있습니다. 하지만 Bot Builder SDK는 간단하게 질문을 하고, 응답이 특정 데이터 형식과 일치하는지 또는 사용자 지정 유효성 검사 규칙을 충족하는지 확인할 수 있도록 설계된 메서드를 제공하는 **대화 상자**를 제공합니다. 이 토픽에서는 **프롬프트**를 사용하여 사용자에게 입력을 요청하는 방법을 설명합니다.

이 문서에서는 대화 상자 내에서 프롬프트를 사용하는 방법을 설명합니다. 일반적으로 대화 상자를 사용하는 방법에 대한 정보는 [대화 상자를 사용하여 간단한 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)를 참조하세요.

## <a name="prompt-types"></a>프롬프트 형식

대화 상자 라이브러리는 다양한 종류의 프롬프트를 제공하며, 각 프롬프트는 다양한 유형의 응답을 수집하는 데 사용됩니다.

| prompt | 설명 |
|:----|:----|
| **AttachmentPrompt** | 문서 또는 이미지 같은 첨부 파일에 대해 사용자에게 확인합니다. |
| **ChoicePrompt** | 옵션 집합에서 선택하도록 사용자에게 확인합니다. |
| **ConfirmPrompt** | 사용자에게 작업을 확인하라는 메시지가 표시됩니다. |
| **DatetimePrompt** | 날짜-시간에 대해 사용자에게 확인합니다. 사용자는 "내일에 오후 8시" 또는 "금요일 오전 10시" 같은 자연어를 사용하여 응답할 수 있습니다. Bot Framework SDK는 LUIS `builtin.datetimeV2` 미리 작성된 엔터티를 사용합니다. 자세한 내용은 [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)를 참조합니다. |
| **NumberPrompt** | 숫자에 대해 사용자에게 확인합니다. 사용자는 "10" 또는 "열"을 사용하여 응답할 수 있습니다. 응답이 "열"인 경우 예를 들어 프롬프트는 응답을 숫자로 변환하고 결과로 `10`을 반환합니다. |
| **TextPrompt** | 텍스트 문자열에 대해 사용자에게 확인합니다. |

## <a name="add-references-to-prompt-library"></a>라이브러리를 확인하려면 참조 추가

봇에 **botbuilder-dialogs** 패키지를 추가하여 **대화 상자** 라이브러리를 가져올 수 있습니다. [대화 상자를 사용하여 간단한 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)에서 대화 상자를 관리하지만 프롬프트에 대해 대화 상자를 사용합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

NuGet에서 **Microsoft.Bot.Builder.Dialogs** 패키지를 설치합니다.

그런 다음, 봇 코드에서 라이브러리 참조를 포함합니다.

```cs
using Microsoft.Bot.Builder.Dialogs;
```

접근자를 통해 대화 대화 상자 상태를 설정해야 합니다. 여기서는 이 코드를 자세히 다루지 않으며, 자세한 내용은 [상태](bot-builder-howto-v4-state.md) 문서에서 확인할 수 있습니다.

**Startup.cs**의 봇 옵션 내에서, 먼저 상태 개체를 정의한 다음, 봇 생성자에 접근자 클래스를 제공하는 싱글톤을 추가합니다. `BotAccessor`에 대한 클래스는 각 항목에 대한 접근자와 함께 대화 및 사용자 상태를 저장합니다. 전체 클래스 정의는 이 문서 끝부분에 연결된 샘플에 제공됩니다. 

```cs
    services.AddBot<MultiTurnPromptsBot>(options =>
    {
        InitCredentialProvider(options);

        // Create and add conversation state.
        var convoState = new ConversationState(dataStore);
        options.State.Add(convoState);

        // Create and add user state.
        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    services.AddSingleton(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        if (options == null)
        {
            throw new InvalidOperationException("BotFrameworkOptions must be configured prior to setting up the State Accessors");
        }

        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        if (conversationState == null)
        {
            throw new InvalidOperationException("ConversationState must be defined and added before adding conversation-scoped state accessors.");
        }

        var userState = options.State.OfType<UserState>().FirstOrDefault();
        if (userState == null)
        {
            throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
        }

        // The dialogs will need a state store accessor. Creating it here once (on-demand) allows the dependency injection
        // to hand it to our IBot class that is create per-request.
        var accessors = new BotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
```

그런 다음, 봇 코드에서 대화 상자 집합에 대해 다음 개체를 정의합니다.

```cs
    private readonly BotAccessors _accessors;

    /// <summary>
    /// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
    /// </summary>
    private DialogSet _dialogs;

    /// <summary>
    /// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    public MultiTurnPromptsBot(BotAccessors accessors)
    {
        _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

        // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
        _dialogs = new DialogSet(accessors.ConversationDialogState);

        // ...
        // other constructor items
        // ...
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Echo 템플릿을 사용하여 JavaScript 봇 만들기 자세한 내용은 [JavaScript 빠른 시작](../javascript/bot-builder-javascript-quickstart.md)을 참조하세요.

npm에서 대화 상자 패키지를 설치합니다.

```cmd
npm install --save botbuilder-dialogs
```

봇에서 **대화 상자**를 사용하려면 이를 봇 코드에 포함시킵니다.

1. **bot.js** 파일에 다음을 추가합니다.

    ```javascript
    // Import components from the dialogs library.
    const { DialogSet, TextPrompt, WaterfallDialog } = require("botbuilder-dialogs");

    // Name for the dialog state property accessor.
    const DIALOG_STATE_PROPERTY = 'dialogState';

    // Define the names for the prompts and dialogs for the dialog set.
    const TEXT_PROMPT = 'textPrompt';
    const MAIN_DIALOG = 'mainDialog';
    ```

    _대화 상자 집합_에는 이 봇의 대화 상자가 포함되며, _텍스트 프롬프트_는 사용자에게 입력을 요청하는 데 사용됩니다. 또한 대화 상자 집합이 상태를 추적하는 데 사용할 수 있는 대화 상자 상태 속성 접근자도 필요합니다.

1. 봇의 생성자 코드를 업데이트합니다. 여기에는 곧 더 많은 기능이 추가될 예정입니다.

    ```javascript
      constructor(conversationState) {
        // Track the conversation state object.
        this.conversationState = conversationState;

        // Create a state property accessor for the dialog set.
        this.dialogState = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    }
    ```

---

## <a name="prompt-the-user"></a>사용자 확인

사용자에게 입력을 요청하려면 **TextPrompt** 같은 기본 제공 클래스 중 하나를 사용하여 프롬프트를 정의한 다음, 대화 집합에 추가하고 대화 상자 ID를 할당합니다.

프롬프트가 추가되면 2단계 폭포 대화 상자에 사용합니다. *폭포* 대화 상자는 단계의 순서를 정의하는 방법입니다. 여러 프롬프트를 서로 연결하여 다단계 대화를 만들 수 있습니다. 자세한 내용은 [대화 상자를 사용하여 간단한 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)에서 [대화 상자 사용](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps) 섹션을 참조하세요.

예를 들어 다음 대화 상자는 사용자에게 이름을 요청한 다음, 응답을 사용하여 사용자에게 인사합니다. 첫 번째 순서에서, 대화 상자는 사용자에게 이름을 요청합니다. 사용자의 응답이 두 번째 단계 함수에 매개 변수로 전달되면 두 번째 단계 함수는 입력을 처리하고 개인화된 인사를 보냅니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

대화 상자에서 사용하는 각 프롬프트에는 이름이 지정되며, 대화 상자나 봇이 사용하여 프롬프트에 액세스합니다. 이러한 모든 샘플에서는 프롬프트 ID를 상수로 공개합니다.

봇 생성자 내에서, 2단계 폭포에 대한 정의 및 사용할 대화 상자에 대한 프롬프트를 추가합니다. 여기서는 독립 함수로 추가할 예정이지만, 원한다면 인라인 람다로 정의할 수도 있습니다.

```csharp
 public MultiTurnPromptsBot(BotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        SayHiAsync,
    };

    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
}
```

그런 다음, 봇 내에서 2단계 폭포를 정의합니다. 텍스트 프롬프트의 경우 위에서 정의한 `TextPrompt`의 *이름* ID를 정의합니다. 메서드 이름이 위의 `WaterfallStep[]`과 일치합니다. 여기에 제공될 향후 샘플에는 코드가 포함되지 않지만, 추가 단계의 경우 해당 `WaterfallStep[]`에서 메서드 이름을 올바른 순서대로 추가해야 합니다.

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 봇의 생성자에서 대화 상자 집합을 만들고 여기에 텍스트 프롬프트와 폭포 대화 상자를 추가합니다.

    ```javascript
    // Create the dialog set, and add the prompt and the waterfall dialog.
    this.dialogs = new DialogSet(this.dialogState)
        .add(new TextPrompt(TEXT_PROMPT))
        .add(new WaterfallDialog(MAIN_DIALOG, [
            async (step) => {
                // The results of this prompt will be passed to the next step.
                return await step.prompt(TEXT_PROMPT, 'What is your name?');
            },
            async (step) => {
                // The result property contains the result from the previous step.
                const userName = step.result;
                await step.context.sendActivity(`Hi ${userName}!`);
                return await step.endDialog();
            }
        ]));
    ```

1. 대화 상자를 실행하도록 봇의 순서 처리기를 업데이트합니다.

    ```javascript
    async onTurn(turnContext) {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Create a dialog context for the dialog set.
            const dc = await this.dialogs.createContext(turnContext);
            // Continue the dialog if it's active.
            await dc.continueDialog();
            if (!turnContext.responded) {
                // Otherwise, start the dialog.
                await dc.beginDialog(MAIN_DIALOG);
            }
        } else {
            // Send a default message for activity types that we don't handle.
            await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        }
    }
    ```

---

> [!NOTE]
> 대화 상자를 시작하려면 대화 컨텍스트를 가져오고 해당 _begin dialog_ 메서드를 사용합니다. 자세한 내용은 [대화 상자를 사용하여 간단한 대화 흐름 관리](./bot-builder-dialog-manage-conversation-flow.md)를 참조합니다.

## <a name="reusable-prompts"></a>다시 사용할 수 있는 프롬프트

답변이 동일한 형식인 한, 프롬프트를 다른 질문에 다시 사용할 수 있습니다. 예를 들어 위의 샘플 코드는 텍스트 프롬프트를 정의하여 사용자의 이름을 묻는 데 사용했습니다. 동일한 프롬프트를 사용하여 사용자에게 "직장이 어디세요?" 같은 다른 텍스트 문자열로 질문할 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

이 예제에서 텍스트 프롬프트 *이름*의 ID는 코드 명확성에 도움이 되지 않습니다. 그러나 프롬프트 ID를 원하는 대로 선택할 수 있는 좋은 예입니다.

이제 사용자가 작업하는 위치를 질문하는 세 번째 단계가 메서드에 포함됩니다.

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> WorkAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Where do you work?") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"{stepContext.Result} is a cool place!");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇의 생성자에서 두 번째 질문을 물어보도록 폭포를 수정합니다.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new TextPrompt(TEXT_PROMPT))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their name.
        return await step.prompt(TEXT_PROMPT, 'What is your name?');
    },
    async (step) => {
        // Acknowledge their response and ask for their place of work.
        const userName = step.result;
        return await step.prompt(TEXT_PROMPT, `Hi ${userName}; where do you work?`);
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const workPlace = step.result;
        await step.context.sendActivity(`${workPlace} is a cool place!`);
        return await step.endDialog();
    }
    ]));
```

---

여러 다른 프롬프트를 사용해야 하는 경우 각 프롬프트에 고유한 *dialogId*를 제공해야 합니다. 대화 상자 집합에 추가된 각 대화 상자 또는 프롬프트에는 고유한 ID가 필요합니다. 동일한 형식의 **프롬프트** 대화 상자를 여러 개 만들 수도 있습니다. 예를 들어 위의 예제에 대한 두 개의 **TextPrompt** 대화 상자를 만들 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
_dialogs.Add(new WaterfallDialog("details", waterfallSteps));
_dialogs.Add(new TextPrompt("name"));
_dialogs.Add(new TextPrompt("workplace"));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

예를 들어 이를 다음과 같이

```javascript
.add(new TextPrompt(TEXT_PROMPT))
```

바꿀 수 있습니다.

```javascript
.add(new TextPrompt('namePrompt'))
.add(new TextPrompt('workPlacePrompt'))
```

그런 다음, 해당 이름으로 해당 프롬프트를 사용하도록 각 폭포 단계를 업데이트합니다.

---

코드 재사용을 위해, `TextPrompt`를 한 번만 정의하면 모든 프롬프트에 적용됩니다. 모든 프롬프트는 응답으로 텍스트를 예상하기 때문입니다. 대화 상자 이름을 지정하는 기능은 프롬프트 입력에 여러 유효성 검사 규칙을 적용해야 하는 경우에 편리합니다. `NumberPrompt`를 사용하여 프롬프트 답변의 유효성을 검사할 수 있는 방법을 살펴보겠습니다.

## <a name="specify-prompt-options"></a>프롬프트 옵션 지정

대화 단계 내에서 프롬프트를 사용하는 경우 재프롬프트 문자열과 같은 프롬프트 옵션을 제공할 수도 있습니다.

재프롬프트 문자열을 지정하면 사용자 입력이 프롬프트를 충족하지 못하는 경우 유용합니다. 이는 숫자 프롬프트에 대한 “내일”과 같이 프롬프트가 구문 분석을 할 수 없는 형식이거나, 입력이 유효성 검사 조건을 충족시키지 못했기 때문입니다. 숫자 프롬프트는 "열둘" 또는 "1/4", "12" 및 "0.25" 같은 다양한 입력을 해석할 수 있습니다.

로캘은 **NumberPrompt**처럼 특정 프롬프트의 선택적 매개 변수입니다. 프롬프트가 입력을 보다 정확하게 구문 분석하는 데 도움이 되지만, 필수는 아닙니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

다음 코드는 기존 대화 집합인 **_dialogs**에 숫자 프롬프트를 추가합니다.

```csharp
_dialogs.Add(new NumberPrompt<int>("age"));
```

대화 단계 내에서 다음 코드는 사용자에게 입력을 확인하고 해당 입력이 숫자로 해석될 수 없는 경우 사용할 재프롬프트 문자열을 제공합니다.

```csharp
return await stepContext.PromptAsync(
    "age",
    new PromptOptions {
        Prompt = MessageFactory.Text("Please enter your age."),
        RetryPrompt = MessageFactory.Text("I didn't get that. Please enter a valid age."),
    },
    cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

대화 상자 라이브러리에서 `NumberPrompt` 클래스를 가져옵니다.

```javascript
const { NumberPrompt } = require("botbuilder-dialogs");
```

폭포 대화 상자에서 숫자 프롬프트를 사용하여 초기 및 재시도 프롬프트 문자열을 지정합니다.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new NumberPrompt('partySize'))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their party size.
        return await step.prompt('partySize', {
            prompt: 'How many people in your party?',
            retryPrompt: 'Sorry, please specify the number of people in your party.'
        });
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const partySize = step.result;
        await step.context.sendActivity(`That's a party of ${partySize}, thanks.`);
        return await step.endDialog();
    }
]));
```

---

선택 프롬프트에는 추가적인 필수 매개 변수인 사용자가 선택 가능한 목록이 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

사용자에게 옵션 집합 중에서 선택하도록 요청하기 위해 **ChoicePrompt**를 사용할 때 **PromptOptions** 개체 내에서 제공된 해당 옵션 집합을 프롬프트에 제공해야 합니다. 여기에서 **ChoiceFactory**를 사용하여 옵션 목록을 적절한 형식으로 변환합니다.

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

    return await stepContext.PromptAsync(
        "color",
        new PromptOptions {
            Prompt = MessageFactory.Text("What's your favorite color?"),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

대화 상자 라이브러리에서 `NumberPrompt` 클래스를 가져옵니다.

```javascript
const { ChoicePrompt } = require("botbuilder-dialogs");
```

폭포 대화 상자에서 옵션 프롬프트를 사용하여 사용 가능한 옵션을 지정합니다.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
const list = ['green', 'blue', 'red', 'yellow'];
this.dialogs = new DialogSet(this.dialogState)
    .add(new ChoicePrompt('choicePrompt'))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their party size.
        return await step.prompt('choicePrompt', {
            prompt: 'Please choose a color:',
            retryPrompt: 'Sorry, please choose a color from the list.',
            choices: list
        });
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const choice = step.result;
        await step.context.sendActivity(`That's ${choice.value}, thanks.`);
        return await step.endDialog();
    }
]));
```

---

## <a name="validate-a-prompt-response"></a>프롬프트 응답의 유효성 검사

값을 **폭포**의 다음 단계로 반환하기 전에 프롬프트 응답의 유효성을 검사할 수 있습니다. 예를 들어 **6**과 **20** 사이의 숫자 범위 내에서 **NumberPrompt**의 유효성을 검사하려면 다음과 비슷한 유효성 검사 함수를 포함해야 합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

유효성 검사기 함수를 포함하도록 프롬프트가 대화 상자 집합에 추가되는 시기를 변경

```cs
_dialogs.Add(new NumberPrompt<int>("partySize", PartySizeValidatorAsync));
```

그런 다음, 유효성 검사를 고유한 메서드로 정의하여 유효성 검사 통과 여부에 따라 false 또는 true를 나타냅니다. false가 반환되면 사용자에게 다시 요청합니다.

```cs
private Task<bool> PartySizeValidatorAsync(PromptValidatorContext<int> promptContext, CancellationToken cancellationToken)
{
    var result = promptContext.Recognized.Value;

    if (result < 6 || result > 20)
    {
        return Task.FromResult(false);
    }

    return Task.FromResult(true);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

프롬프트를 만들 때 유효성 검사 메서드를 추가합니다.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new NumberPrompt('partySizePrompt', async (promptContext) =>                                                 {
        // Check to make sure a value was recognized.
        if (promptContext.recognized.succeeded) {
            const value = promptContext.recognized.value;
            try {
                if (value < 6) {
                    throw new Error('Party size too small.');
                } else if (value > 20) {
                    throw new Error('Party size too big.')
                } else {
                    return true; // Indicate that this is a valid value.
                }
            } catch (err) {
                await promptContext.context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
                return false; // Indicate that this is invalid.
            }
        } else {
            return false;
        }
    }))
    .add(new WaterfallDialog(MAIN_DIALOG, [
        async (step) => {
            // Ask the user for their party size.
            return await step.prompt('partySizePrompt', {
                prompt: 'How large is your party?',
                retryPrompt: 'Sorry, please specify a size between 6 and 20.'
            });
        },
        async (step) => {
            // Acknowledge their response and exit the dialog.
            const size = step.result;
            await step.context.sendActivity(`That's a party of ${size}, thanks.`);
            return await step.endDialog();
        }
    ]));
```

---

마찬가지로, 향후 시간 및 날짜에 대한 **DatetimePrompt** 응답의 유효성을 검사하려는 경우 다음과 비슷한 유효성 검사 논리가 있을 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
    private Task<bool> DateTimeValidatorAsync(PromptValidatorContext<IList<DateTimeResolution>> prompt, CancellationToken cancellationToken)
    {
        if (prompt.Recognized.Succeeded)
        {
            var resolution = prompt.Recognized.Value.First();

            // Verify that the Timex received is within the desired bounds, compared to today.
            var now = DateTime.Now;
            DateTime.TryParse(resolution.Value, out var time);

            if (time < now)
            {
                return Task.FromResult(false);
            }

            return Task.FromResult(true);
        }

        return Task.FromResult(false);
    }
```

```csharp
_dialogs.Add(new DateTimePrompt("date", DateTimeValidatorAsync));
```

추가 예제는 [리포지토리 샘플](https://aka.ms/bot-samples-readme)에서 찾을 수 있습니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const { DateTimePrompt } = require("botbuilder-dialogs");
```

```JavaScript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new DateTimePrompt('dateTimePrompt', async (promptContext) => {
        try {
            if (!promptContext.recognized.succeeded) { throw new Error('Value not recognized.') }
            const values = promptContext.recognized.value;
            if (!Array.isArray(values) || values.length < 0) { throw new Error('Value missing.'); }
            if ((values[0].type !== 'datetime') && (values[0].type !== 'date')) { throw new Error('Unsupported type.'); }
            const now = new Date();
            const value = new Date(values[0].value);
            if (value.getTime() < now.getTime()) { throw new Error('Value in the past.') }

            // update the return value of the prompt to be a real date object
            promptContext.recognized.value = [value];
            return true; // indicate valid
        } catch (err) {
            await promptContext.context.sendActivity(`${err} Please specify a date or a date and time in the future, like tomorrow at 9am.`);
            return false; // indicate invalid
        }
    }))
    .add(new WaterfallDialog(MAIN_DIALOG, [
        async (step) => {
            // Ask the user for their party size.
            return await step.prompt('dateTimePrompt', 'When would you like to schedule that for?');
        },
        async (step) => {
            // Acknowledge their response and exit the dialog.
            const time = step.result;
            await step.context.sendActivity(`That's ${time}, thanks.`);
            return await step.endDialog();
        }
    ]));
```

추가 예제는 [리포지토리 샘플](https://aka.ms/bot-samples-readme)에서 찾을 수 있습니다.

---

> [!TIP]
> 날짜/시간 프롬프트는 사용자가 모호한 답변을 하는 경우 몇몇 다른 날짜를 확인할 수 있습니다. 사용 목적에 따라 첫 번째 해결책이 아닌 프롬프트 결과에서 제공하는 모든 해결책을 확인하고 싶을 수 있습니다.

비슷한 기법을 사용하여 프롬프트 형식 중 하나에 대한 프롬프트 응답의 유효성을 검사할 수 있습니다.

## <a name="save-user-data"></a>사용자 데이터 저장

사용자 입력에 대해 확인하는 경우 해당 입력을 처리하는 방법에 대해 여러 옵션이 있습니다. 예를 들어 입력을 사용하고 버릴 수 있으며, 글로벌 변수에 저장할 수 있고, 일시적 또는 메모리 내 저장소 컨테이너에 저장할 수 있고, 파일에 저장할 수 있거나 외부 데이터베이스에 저장할 수 있습니다. 사용자 데이터를 저장하는 방법에 대한 자세한 내용은 [사용자 데이터 관리](bot-builder-howto-v4-state.md)를 참조하세요.

## <a name="additional-resources"></a>추가 리소스

이러한 프롬프트를 사용하는 완전한 샘플은 [C#](https://aka.ms/cs-multi-prompts-sample) 또는 [JavaScript](https://aka.ms/js-multi-prompts-sample)용 다중 순서 프롬프트 봇을 참조하세요.

## <a name="next-steps"></a>다음 단계

사용자에게 입력을 확인하는 방법에 대해 배웠으므로 대화 상자를 통해 다양한 대화 흐름을 관리하여 봇 코드 및 사용자 환경을 향상시켜보겠습니다.

> [!div class="nextstepaction"]
> [대화 상자를 사용하여 간단한 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)
