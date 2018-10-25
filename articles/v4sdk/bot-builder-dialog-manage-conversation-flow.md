---
title: 대화 상자를 사용하여 간단한 대화 흐름 관리 | Microsoft Docs
description: Node.js용 Bot Builder SDK에서 대화 상자를 사용하여 간단한 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 간단한 대화 흐름, 대화 상자, 프롬프트, 폭포, 대화 상자 집합
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 07035c8f0dfc7473192d8c51667ed1f5cefbc555
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999400"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>대화 상자를 사용하여 간단한 대화 흐름 관리

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

대화 상자 라이브러리를 사용하여 단순 및 복합 대화 흐름을 관리할 수 있습니다. 단순 대화 흐름의 경우 사용자가 *폭포*의 첫 번째 단계에서 시작하여 마지막 단계까지 계속 진행함으로써 대화를 완료합니다. [복합 대화 흐름](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)에는 분기 및 루프가 포함됩니다.

<!-- TODO: This paragraph belongs in a conceptual topic. -->

대화 상자는 봇의 구조체로써 봇 프로그램의 함수처럼 작동합니다. 대화 상자는 봇에서 보내는 메시지를 빌드하고, 필요한 계산 작업을 수행합니다. 대화 상자는 특정 작업을 특정 순서대로 수행하도록 설계되었습니다. 사용자에 대한 응답에서 호출하거나, 외부 자극에 대한 응답에서 호출하거나, 다른 대화 상자에서 호출하는 등 다양한 방법으로 호출할 수 있습니다.

대화 상자를 사용하면 봇 개발자가 대화 흐름을 안내할 수 있습니다. 여러 대화 상자를 만들고 서로 연결하면 봇으로 처리할 대화 흐름을 만들 수 있습니다. Bot Builder SDK의 **대화 상자** 라이브러리에는 대화 흐름을 관리하는 데 도움이 되는 _프롬프트_, _폭포 대화 상자_ 및 _구성 요소 대화 상자_와 같은 기본 제공 기능이 포함됩니다. 프롬프트를 사용하여 사용자에게 다른 유형의 정보를 요청할 수 있습니다. 폭포를 사용하여 여러 단계를 하나의 시퀀스로 결합할 수 있습니다. 그리고 구성 요소 대화 상자를 사용하여 여러 하위 대화 상자가 포함된 모듈식 대화 시스템을 만들 수 있습니다.

이 문서에서는 _대화 상자 집합_을 사용하여 프롬프트 및 폭포가 모두 포함된 대화 흐름을 만듭니다. **다중 순서 프롬프트** [[C#](https://aka.ms/cs-multi-prompts-sample)|[JS](https://aka.ms/js-multi-prompts-sample)] 샘플의 코드로 그릴 것입니다.

대화 상자에 대한 개요는 [대화 상자 라이브러리](bot-builder-concept-dialog.md) 및 [대화 상자 상태](bot-builder-dialog-state.md)를 참조하세요.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

일반적인 방법으로 대화 상자를 사용하려면 프로젝트 또는 솔루션용 `Microsoft.Bot.Builder.Dialogs` NuGet 패키지가 필요합니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

일반적인 방법으로 대화 상자를 사용하려면 `botbuilder-dialogs` 라이브러리가 필요하며 NPM에서 다운로드할 수 있습니다.

---

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>다이얼로그를 사용하여 사용자에게 단계 안내

이 예제에서는 대화 상자 집합을 활용하여 사용자에게 정보를 요청하는 다단계 대화 상자를 만듭니다.

### <a name="create-a-dialog-with-waterfall-steps"></a>폭포형 단계를 사용하여 다이얼로그 만들기

**WaterfallDialog**는 사용자로부터 정보를 수집하거나 사용자에게 일련의 작업을 안내하는 데 일반적으로 사용되는 대화 상자의 구현입니다. 대화의 각 단계는 함수로 구현됩니다. 각 단계에 대해 봇은 [사용자에게 입력을 프롬프트하고](bot-builder-prompts.md), 응답을 기다린 후 결과를 다음 단계로 전달합니다. 첫 번째 함수의 결과는 그 다음 함수에 인수로 전달됩니다.

예를 들어 다음 코드 샘플은 **폭포**의 단계를 나타내는 대리자 배열을 정의합니다. 각 프롬프트 후, 봇은 사용자의 입력을 인식합니다. 대화 상자에서 수집하는 입력을 여러 가지 방법으로 유지할 수 있습니다. 옵션에 대한 설명은 [사용자 데이터 유지](bot-builder-tutorial-persist-user-inputs.md)를 참조하세요.

이 샘플은 대화 상자에서 수집되는 사용자 프로필에 직접 정보를 씁니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

이 샘플에서는 폭포 대화 상자가 봇 파일 내에 정의됩니다.

이 파일에 사용된 네임스페이스를 참조합니다.

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
```

대화 상자 집합의 인스턴스 속성을 정의합니다.

```csharp
/// <summary>
/// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
/// </summary>
private DialogSet _dialogs;
```

봇의 생성자 내에 대화 상자 집합을 만들고, 이 집합에 프롬프트 및 폭포 대화 상자를 추가합니다.

```csharp
/// <summary>
/// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
/// </summary>
/// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
public MultiTurnPromptsBot(MultiTurnPromptsBotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        NameConfirmStepAsync,
        AgeStepAsync,
        ConfirmStepAsync,
        SummaryStepAsync,
    };

    // Add named dialogs to the DialogSet. These names are saved in the dialog state.
    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
    _dialogs.Add(new NumberPrompt<int>("age"));
    _dialogs.Add(new ConfirmPrompt("confirm"));
}
```

그리고 각 단계를 별도의 메서드로 정의합니다. 람다 식을 사용하여 인라인 단계를 정의할 수도 있습니다.

```csharp
/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> NameConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Name = (string)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Thanks {stepContext.Result}."), cancellationToken);

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Would you like to give your age?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> AgeStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // User said "yes" so we will be prompting for the age.

        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
        return await stepContext.PromptAsync("age", new PromptOptions { Prompt = MessageFactory.Text("Please enter your age.") }, cancellationToken);
    }
    else
    {
        // User said "no" so we will skip the next step. Give -1 as the age.
        return await stepContext.NextAsync(-1, cancellationToken);
    }
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> ConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Age = (int)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    if (userProfile.Age == -1)
    {
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"No age given."), cancellationToken);
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your age as {userProfile.Age}."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Is this ok?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> SummaryStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // We can send messages to the user at any point in the WaterfallStep.
        if (userProfile.Age == -1)
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name}."), cancellationToken);
        }
        else
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name} and age as {userProfile.Age}."), cancellationToken);
        }
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text("Thanks. Your profile will not be kept."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is the end.
    return await stepContext.EndDialogAsync(cancellationToken: cancellationToken);
}
```

대화 상자는 봇의 순서 처리기에서 실행되며, 이 처리기는 먼저 대화 상자 컨텍스트를 만들고 대화 상자를 적절하게 계속 진행하거나 시작한 다음, 순서의 마지막에 대화 및 사용자 상태를 저장합니다.

```csharp
// Run the DialogSet - let the framework identify the current state of the dialog from
// the dialog stack and figure out what (if any) is the active dialog.
var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
var results = await dialogContext.ContinueDialogAsync(cancellationToken);

// If the DialogTurnStatus is Empty we should start a new dialog.
if (results.Status == DialogTurnStatus.Empty)
{
    await dialogContext.BeginDialogAsync("details", null, cancellationToken);
}
```

```csharp
// Save the dialog state into the conversation state.
await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

// Save the user profile updates into the user state.
await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

이 샘플에서는 폭포 대화 상자가 **bot.js** 파일 내에 정의됩니다.

코드에 필요한 개체를 가져옵니다.

```javascript
const { ActivityTypes } = require('botbuilder');
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

봇의 생성자에 대화 상자 집합을 정의 및 생성하고, 이 집합에 프롬프트 및 폭포 대화 상자를 추가합니다.

```javascript
/**
*
* @param {ConversationState} conversationState A ConversationState object used to store the dialog state.
* @param {UserState} userState A UserState object used to store values specific to the user.
*/
constructor(conversationState, userState) {
    // Create a new state accessor property. See https://aka.ms/about-bot-state-accessors to learn more about bot state and state accessors.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty(DIALOG_STATE_PROPERTY);

    this.userProfile = this.userState.createProperty(USER_PROFILE_PROPERTY);

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts that will be used by the main dialogs.
    this.dialogs.add(new TextPrompt(NAME_PROMPT));
    this.dialogs.add(new ChoicePrompt(CONFIRM_PROMPT));
    this.dialogs.add(new NumberPrompt(AGE_PROMPT, async (prompt) => {
        if (prompt.recognized.succeeded) {
            if (prompt.recognized.value <= 0) {
                await prompt.context.sendActivity(`Your age can't be less than zero.`);
                return false;
            } else {
                return true;
            }
        }

        return false;
    }));

    // Create a dialog that asks the user for their name.
    this.dialogs.add(new WaterfallDialog(WHO_ARE_YOU, [
        this.promptForName.bind(this),
        this.confirmAgePrompt.bind(this),
        this.promptForAge.bind(this),
        this.captureAge.bind(this)
    ]));

    // Create a dialog that displays a user name after it has been collected.
    this.dialogs.add(new WaterfallDialog(HELLO_USER, [
        this.displayProfile.bind(this)
    ]));
}
```

그리고 각 단계를 별도의 메서드로 정의합니다. 람다 식을 사용하여 인라인 단계를 정의할 수도 있습니다.

```javascript
// This step in the dialog prompts the user for their name.
async promptForName(step) {
    return await step.prompt(NAME_PROMPT, `What is your name, human?`);
}

// This step captures the user's name, then prompts whether or not to collect an age.
async confirmAgePrompt(step) {
    const user = await this.userProfile.get(step.context, {});
    user.name = step.result;
    await this.userProfile.set(step.context, user);
    await step.prompt(CONFIRM_PROMPT, 'Do you want to give your age?', ['yes', 'no']);
}

// This step checks the user's response - if yes, the bot will proceed to prompt for age.
// Otherwise, the bot will skip the age step.
async promptForAge(step) {
    if (step.result && step.result.value === 'yes') {
        return await step.prompt(AGE_PROMPT, `What is your age?`,
            {
                retryPrompt: 'Sorry, please specify your age as a positive number or say cancel.'
            }
        );
    } else {
        return await step.next(-1);
    }
}

// This step captures the user's age.
async captureAge(step) {
    const user = await this.userProfile.get(step.context, {});
    if (step.result !== -1) {
        user.age = step.result;
        await this.userProfile.set(step.context, user);
        await step.context.sendActivity(`I will remember that you are ${ step.result } years old.`);
    } else {
        await step.context.sendActivity(`No age given.`);
    }
    return await step.endDialog();
}

// This step displays the captured information back to the user.
async displayProfile(step) {
    const user = await this.userProfile.get(step.context, {});
    if (user.age) {
        await step.context.sendActivity(`Your name is ${ user.name } and you are ${ user.age } years old.`);
    } else {
        await step.context.sendActivity(`Your name is ${ user.name } and you did not share your age.`);
    }
    return await step.endDialog();
}
```

대화 상자는 봇의 순서 처리기에서 실행되며, 이 처리기는 먼저 대화 상자 컨텍스트를 만들고 대화 상자를 적절하게 계속 진행하거나 시작한 다음, 순서의 마지막에 대화 및 사용자 상태를 저장합니다.

```javascript
// Create a dialog context object.
const dc = await this.dialogs.createContext(turnContext);
```

```javascript
// If the bot has not yet responded, continue processing the current dialog.
await dc.continueDialog();
```

```javascript
// Start the sample dialog in response to any other input.
if (!turnContext.responded) {
    const user = await this.userProfile.get(dc.context, {});
    if (user.name) {
        await dc.beginDialog(HELLO_USER);
    } else {
        await dc.beginDialog(WHO_ARE_YOU);
    }
}
```

```javascript
// Save changes to the user state.
await this.userState.saveChanges(turnContext);

// End this turn by saving changes to the conversation state.
await this.conversationState.saveChanges(turnContext);
```

---

## <a name="dialog-context-and-waterfall-step-context-objects"></a>대화 상자 컨텍스트 및 폭포 단계 컨텍스트 개체

대화 상자 컨텍스트 개체를 사용하여 봇의 순서 처리기 내에서 대화 상자 설정과 상호 작용합니다.
폭포 단계 컨텍스트 개체를 사용하여 폭포 단계 내에 설정된 대화 상자와 상호 작용합니다.

## <a name="to-start-a-dialog"></a>대화 상자를 시작하려면

대화 상자를 시작하려면 시작하려는 *dialogId*를 대화 상자 컨텍스트의 _beginDialog_, _prompt_ 또는 _replaceDialog_ 메서드로 전달합니다. _beginDialog_ 메서드는 대화 상자를 스택으로 푸시하고, _replaceDialog_ 메서드는 현재 대화 상자를 스택에서 삭제하고 대체 대화 상자를 스택에 푸시합니다.

대화 상자 컨텍스트의 _prompt_ 메서드는 인수를 사용하고, 프롬프트에 적절한 옵션을 생성한 다음, 프롬프트 대화 상자를 시작하는 도우미 메서드입니다. 프롬프트에 대한 자세한 내용은 [사용자에게 입력 요청](bot-builder-prompts.md)을 참조하세요.

## <a name="to-end-a-dialog"></a>대화 상자를 종료하려면

_end dialog_ 메서드는 스택에서 대화 상자를 삭제하여 대화 상자를 종료하고, 선택적 결과를 부모 대화 상자에 반환합니다.

대화 상자 끝부분에서 명시적으로 _endDialog_ 메서드를 호출하는 것이 가장 좋은 모범 사례입니다.

## <a name="to-clear-the-dialog-stack"></a>대화 상자 스택을 지우려면

모든 대화 상자를 스택에서 삭제하려면 대화 상자 컨텍스트의 _cancel all dialogs_ 메서드를 호출하여 대화 상자 스택을 지울 수 있습니다.

## <a name="to-repeat-a-dialog"></a>대화 상자를 반복하려면

대화 상자를 반복하려면 현재 대화 상자를 스택에서 삭제하고 대체 대화 상자를 스택의 맨 위로 푸시하여 해당 대화 상자를 시작하는 _replace dialog_ 메서드를 사용합니다. 이는 [복잡한 대화 흐름](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)을 처리하는 좋은 방법이며 메뉴를 관리하는 데 유용한 기술입니다.

## <a name="next-steps"></a>다음 단계

단순 대화 흐름을 관리하는 방법을 알아보았으니, 이번에는 _replace_ 메서드를 활용하여 복합 대화 흐름을 처리하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [복잡한 대화 흐름 관리](bot-builder-dialog-manage-complex-conversation-flow.md)
