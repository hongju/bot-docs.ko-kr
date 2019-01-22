---
title: 순차적 대화 흐름 구현 | Microsoft Docs
description: Node.js용 Bot Framework SDK에서 대화 상자를 사용하여 간단한 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 간단한 대화 흐름, 순차적 대화 흐름, 대화 상자, 프롬프트, 폭포, 대화 상자 집합
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 554d040dd4c9d126fa70c24f1af5a1ac97a39204
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317623"
---
# <a name="implement-sequential-conversation-flow"></a>순차적 대화 흐름 구현

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

대화 상자 라이브러리를 사용하여 단순 및 복합 대화 흐름을 관리할 수 있습니다. 간단한 상호 작용에서 봇은 고정된 일련의 단계를 통해 실행되고 대화가 완료됩니다. 이 문서에서는 _폭포 대화 상자_, 몇 가지 _프롬프트_ 및 _대화 상자 집합_을 사용하여 사용자에게 일련의 질문을 묻는 간단한 상호 작용을 만듭니다.

## <a name="prerequisites"></a>필수 조건
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- 이 문서의 코드는 **multi-turn-prompt**(다중 턴 프롬프트) 샘플을 기반으로 합니다. [C#](https://aka.ms/cs-multi-prompts-sample) 또는 [JS](https://aka.ms/js-multi-prompts-sample)로 작성된 샘플의 복사본이 필요합니다.
- [봇 기본 사항](bot-builder-basics.md), [대화 라이브러리](bot-builder-concept-dialog.md), [대화 상태](bot-builder-dialog-state.md) 및 [.bot](bot-file-basics.md) 파일에 대한 지식이 필요합니다.


다음 섹션에서는 대부분의 봇에서 간단한 대화 상자를 구현하기 위해 수행할 단계를 반영합니다.

## <a name="configure-your-bot"></a>봇 구성

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs** 파일의 구성 코드에서 봇의 대화 상자 상태에 대한 상태 속성 접근자를 초기화합니다.

봇에 대한 상태 관리 개체 및 상태 속성 접근자가 포함되도록 `MultiTurnPromptsBotAccessors` 클래스를 정의합니다.
여기에서 코드의 일부만 호출합니다.

```csharp
public class MultiTurnPromptsBotAccessors
{
    // Initializes a new instance of the class.
    public MultiTurnPromptsBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<DialogState> ConversationDialogState { get; set; }
    public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }

    public ConversationState ConversationState { get; }
    public UserState UserState { get; }
}
```

`Startup` 클래스의 `ConfigureServices` 메서드에서 접근자 클래스를 등록합니다.
다시 코드의 일부만 호출합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<MultiTurnPromptsBotAccessors>(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new MultiTurnPromptsBotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
}
```

종속성 주입을 통해 접근자는 봇의 생성자 코드에 지원됩니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js** 파일에서 상태 관리 개체를 정의합니다.
여기에서 코드의 일부만 호출합니다.

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage, UserState } = require('botbuilder');

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the main dialog, which serves as the bot's main handler.
const bot = new MultiTurnBot(conversationState, userState);
```

봇의 생성자는 봇의 상태 속성 접근자(`this.dialogState` 및 `this.userProfile`)를 만듭니다.

---

## <a name="update-the-bot-turn-handler-to-call-the-dialog"></a>대화 상자를 호출하도록 봇의 순서 처리기 업데이트

대화 상자를 실행하기 위해 봇의 순서 처리기는 봇에 대한 대화 상자를 포함하는 대화 상자 집합의 대화 컨텍스트를 만들어야 합니다. 봇은 여러 개의 대화 세트를 정의할 수 있지만, 일반적으로 봇당 하나의 대화 세트만 정의해야 합니다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

대화 상자는 봇의 순서 처리기에서 실행됩니다. 처리기는 먼저 `DialogContext`를 만들고 활성 대화 상자를 계속하거나 적절하게 새 대화 상자를 시작합니다. 그런 다음, 처리기는 순서의 끝에서 대화 및 사용자 상태를 저장합니다.

`MultiTurnPromptsBot` 클래스에서 대화 상자 집합을 포함하는 `_dialogs` 속성을 정의했습니다. 여기서 대화 컨텍스트를 생성합니다. 다시 여기에서는 순서 처리기 코드의 일부만 보여줍니다.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // ...
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
        var results = await dialogContext.ContinueDialogAsync(cancellationToken);

        // If the DialogTurnStatus is Empty we should start a new dialog.
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync("details", null, cancellationToken);
        }
    }

    // ...
    // Save the dialog state into the conversation state.
    await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

    // Save the user profile updates into the user state.
    await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇 코드는 대화 상자 라이브러리에서 몇 가지 클래스를 사용합니다.

```javascript
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

대화 상자는 봇의 순서 처리기에서 실행됩니다. 처리기는 먼저 `DialogContext`(`dc`)를 만들고 활성 대화 상자를 계속하거나 적절하게 새 대화 상자를 시작합니다. 그런 다음, 처리기는 순서의 끝에서 대화 및 사용자 상태를 저장합니다.

`MultiTurnBot` 클래스는 **bot.js** 파일에서 정의됩니다. 이 클래스의 생성자는 대화 상자 집합의 `dialogs` 속성을 추가합니다. 여기에서 대화 컨텍스트를 생성합니다. 이 봇은 `WHO_ARE_YOU` 대화 상자를 사용하여 사용자 데이터를 한 번 수집합니다. 사용자 프로필이 채워지면 봇은 `HELLO_USER` 대화 상자를 사용하여 응답합니다. 다시 여기에서는 순서 처리기 코드의 일부만 보여줍니다.

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Create a dialog context object.
        const dc = await this.dialogs.createContext(turnContext);

        const utterance = (turnContext.activity.text || '').trim().toLowerCase();

        // ...
        // If the bot has not yet responded, continue processing the current dialog.
        await dc.continueDialog();

        // Start the sample dialog in response to any other input.
        if (!turnContext.responded) {
            const user = await this.userProfile.get(dc.context, {});
            if (user.name) {
                await dc.beginDialog(HELLO_USER);
            } else {
                await dc.beginDialog(WHO_ARE_YOU);
            }
        }
    }

    // ...
    // Save changes to the user state.
    await this.userState.saveChanges(turnContext);

    // End this turn by saving changes to the conversation state.
    await this.conversationState.saveChanges(turnContext);
}
```

---

봇의 순서 처리기에서 대화 상자 집합의 대화 상자 컨텍스트를 만듭니다. 대화 컨텍스트는 효과적으로 대화의 마지막 순서가 다루지 않은 위치를 기억하여 봇의 상태 캐시에 액세스합니다.

활성 대화 상자가 있으면 대화 상자 컨텍스트의 _continue dialog_ 메서드가 이 순서를 트리거한 사용자의 입력을 사용하여 해당 대화 상자를 진행합니다. 그렇지 않으면, 봇이 대화 상자 컨텍스트의 _begin dialog_ 메서드를 호출하여 대화 상자를 시작합니다.

마지막으로, 상태 관리 개체에서 _save changes_ 메서드를 호출하여 이 순서에 발생한 변경 내용을 유지합니다.

### <a name="about-dialog-and-bot-state"></a>대화 상자 및 봇 상태 정보

이 봇에서 두 개의 상태 속성 접근자를 정의했습니다.

* 대화 상자 상태 속성에 대해 대화 상태 내에서 만든 접근자입니다. 대화 상자 상태는 대화 상자 집합의 대화 상자 내에서 사용자를 추적합니다. 또한 시작 대화 상자를 호출하거나 대화 상자 메서드를 계속하는 경우 대화 상자 컨텍스트로 업데이트합니다.
* 사용자 프로필 속성에 대한 사용자 상태 내에서 만든 접근자입니다. 봇은 이 접근자를 사용하여 사용자에 대해 아는 정보를 추적합니다. 본사는 봇 코드에서 명시적으로 이 상태를 관리합니다.

상태 속성 접근자의 _get_ 및 _set_ 메서드는 상태 관리 개체의 캐시에서 속성의 값을 가져오고 설정합니다. 처음으로 상태 속성의 값이 순서대로 요청되는 경우 캐시가 채워지지만 명시적으로 유지되어야 합니다. 이러한 상태 속성의 변경 내용을 모두 유지하기 위해 해당하는 상태 관리 개체의 _save changes_ 메서드를 호출합니다.

## <a name="initialize-your-bot-and-define-your-dialog"></a>봇 초기화 및 대화 상자 정의

간단한 대화는 사용자에게 제기된 일련의 질문으로 모델링됩니다. C# 및 JavaScript 버전의 단계는 약간 다릅니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. 해당하는 이름을 묻습니다.
1. 연령을 입력하려는지 묻습니다.
1. 그렇다면 연령을 묻습니다. 그렇지 않으면, 이 단계를 건너뜁니다.
1. 수집된 정보가 올바른지 묻습니다.
1. 상태 메시지를 보내고 종료합니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`who_are_you` 대화 상자의 경우:

1. 해당하는 이름을 묻습니다.
1. 연령을 입력하려는지 묻습니다.
1. 그렇다면 연령을 묻습니다. 그렇지 않으면, 이 단계를 건너뜁니다.
1. 상태 메시지를 보내고 종료합니다.

`hello_user` 대화 상자의 경우:

1. 봇이 수집한 사용자 정보를 표시합니다.

---

고유한 폭포 단계를 정의할 때 고려해야 할 몇 가지 사항은 다음과 같습니다.

* 각 봇 순서는 사용자의 입력을 반영하고 봇의 응답으로 이어집니다. 따라서 폭포 단계의 끝에서 사용자에게 입력하도록 요청하고, 다음 폭포 단계에서 해당 응답을 수신하게 됩니다.
* 각 프롬프트는 사실상 "유효한" 입력을 받을 때까지 해당 프롬프트를 표시하고 반복하는 2단계 대화 상자입니다. 

이 샘플에서 대화 상자는 봇 파일 내에서 정의되고 봇의 생성자에서 초기화됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

대화 상자 집합의 인스턴스 속성을 정의합니다.

```csharp
// The DialogSet that contains all the Dialogs that can be used at runtime.
private DialogSet _dialogs;
```

봇의 생성자 내에 대화 상자 집합을 만들고, 이 집합에 프롬프트 및 폭포 대화 상자를 추가합니다.

```csharp
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

이 샘플에서 각 단계를 별도의 메서드로 정의합니다. 람다 식을 사용하여 생성자에서 인라인 단계를 정의할 수도 있습니다.

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

이 샘플에서는 폭포 대화 상자가 **bot.js** 파일 내에 정의됩니다.

상태 속성 접근자, 프롬프트를 및 대화 상자에 사용할 식별자를 정의합니다.

```javascript
const DIALOG_STATE_PROPERTY = 'dialogState';
const USER_PROFILE_PROPERTY = 'user';

const WHO_ARE_YOU = 'who_are_you';
const HELLO_USER = 'hello_user';

const NAME_PROMPT = 'name_prompt';
const CONFIRM_PROMPT = 'confirm_prompt';
const AGE_PROMPT = 'age_prompt';
```

봇의 생성자에 대화 상자 집합을 정의하고 생성하여 이 집합에 프롬프트 및 폭포 대화 상자를 추가합니다.
`NumberPrompt`에는 사용자가 0보다 큰 연령을 입력하도록 하는 사용자 지정 유효성 검사가 포함됩니다.

```javascript
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

대화 상자 단계 메서드가 인스턴스 속성을 참조하므로 `this` 개체가 각 단계 메서드 내에서 올바르게 확인되도록 `bind` 메서드를 사용해야 합니다.

이 샘플에서 각 단계를 별도의 메서드로 정의합니다. 람다 식을 사용하여 생성자에서 인라인 단계를 정의할 수도 있습니다.

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

---

이 샘플은 대화 상자 내에서 사용자 프로필 상태를 업데이트합니다. 이 방법은 간단한 봇에 사용할 수 있지만 봇을 통해 대화 상자를 다시 사용하려는 경우 작동하지 않습니다.

대화 상자 단계와 봇 상태를 분리하는 다양한 옵션이 있습니다. 예를 들어, 대화 상자가 완전한 정보를 수집하면 다음을 수행할 수 있습니다.

* _end dialog_ 메서드를 사용하여 수집된 데이터를 부모 컨텍스트에 다시 반환 값으로 제공합니다. 이것은 봇의 순서 처리기 또는 대화 상자 스택의 초기 활성 대화 상자일 수 있습니다. 프롬프트 클래스를 디자인하는 방법입니다.
* 적절한 서비스에 요청을 생성합니다. 봇이 대규모 서비스에 대한 프론트 엔드의 역할을 하는 경우 제대로 작동할 수 있습니다.

## <a name="test-your-dialog"></a>대화 상자 테스트

봇을 로컬로 빌드하고 실행한 다음, 에뮬레이터를 사용하여 봇과 상호 작용합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. 봇은 사용자가 대화에 추가된 대화 업데이트 작업에 대한 응답에서 초기 인사말 메시지를 보냅니다.
1. `hi` 또는 기타 입력을 입력합니다. 이 순서에 활성 대화 상자가 없으므로 봇은 `details` 대화 상자를 시작합니다.
   * 봇은 대화의 첫 번째 프롬프트를 보내고 자세한 입력을 기다립니다.
1. 봇이 요청하면 질문에 응답하고 대화 상자를 통해 진행합니다.
1. 대화 상자의 마지막 단계는 입력에 따라 `Thanks` 메시지를 보냅니다.
   * 대화가 종료되면 대화 스택에서 제거되고 봇에는 더 이상 활성 대화 상자가 없게 됩니다.
1. `hi` 또는 다른 입력을 입력하여 대화 상자를 다시 시작합니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 봇은 사용자가 대화에 추가된 대화 업데이트 작업에 대한 응답에서 초기 인사말 메시지를 보냅니다.
1. `hi` 또는 기타 입력을 입력합니다. 이 순서에 활성 대화 상자가 없고 사용자 프로필이 없으므로 봇은 `who_are_you` 대화 상자를 시작합니다.
   * 봇은 대화의 첫 번째 프롬프트를 보내고 자세한 입력을 기다립니다.
1. 봇이 요청하면 질문에 응답하고 대화 상자를 통해 진행합니다.
1. 대화 상자의 마지막 단계는 간단한 확인 메시지를 보냅니다.
1. `hi` 또는 기타 입력을 입력합니다.
   * 봇은 1단계 `hello_user` 대화 상자를 시작합니다. 여기에서 수집된 데이터의 정보를 표시하고 즉시 종료합니다.

---

## <a name="additional-resources"></a>추가 리소스
여기서 설명한 대로 프롬프트의 각 유형에 대한 기본 제공 유효성 검사를 사용할 수 있거나, 사용자 고유의 사용자 지정 유효성 검사를 프롬프트에 추가할 수 있습니다. 자세한 내용은 [대화 프롬프트를 사용하여 사용자 입력 수집](bot-builder-prompts.md)을 참조하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [분기 및 루프를 사용하여 고급 대화 흐름 만들기](bot-builder-dialog-manage-complex-conversation-flow.md)
