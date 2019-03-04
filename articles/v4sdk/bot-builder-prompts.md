---
title: 대화 상자 프롬프트를 사용하여 사용자 입력 수집 | Microsoft Docs
description: Bot Framework SDK의 대화 상자 라이브러리를 사용하여 사용자에게 입력을 요청하는 방법에 대해 알아봅니다.
keywords: prompts, prompt, user input, dialogs, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, reprompt, validation
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 68c01b0f12790393fe0ee7ae0bd28addf2d26ae7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591124"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>대화 상자 프롬프트를 사용하여 사용자 입력 수집

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

질문을 제시하여 정보를 수집하는 것은 봇에서 사용자와 상호 작용하는 주요 방법 중 하나입니다. *대화* 라이브러리를 사용하면 질문을 쉽게 할 수 있을 뿐만 아니라 특정 데이터 형식과 일치하거나 사용자 지정 유효성 검사 규칙을 충족하는지 확인하기 위해 응답의 유효성도 검사할 수 있습니다. 이 항목에서는 폭포 대화에서 프롬프트를 만들고 호출하는 방법에 대해 자세히 설명합니다.

## <a name="prerequisites"></a>필수 조건

- 이 문서의 코드는 **DialogPromptBot** 샘플을 기반으로 합니다. [C# 샘플](https://aka.ms/dialog-prompt-cs) 또는 [JS 샘플](https://aka.ms/dialog-prompt-js)의 복사본이 필요합니다.
- [대화 라이브러리](bot-builder-concept-dialog.md)와 [대화를 관리하는 방법](bot-builder-dialog-manage-conversation-flow.md)에 대한 기본적인 이해가 필요합니다.
- 테스트를 위해 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)가 필요합니다.

## <a name="using-prompts"></a>프롬프트 사용

대화와 프롬프트가 모두 동일한 대화 집합에 있는 경우에만 대화에서 프롬프트를 사용할 수 있습니다. 대화 내의 여러 단계 및 동일한 대화 집합의 여러 대화에서 동일한 프롬프트를 사용할 수 있습니다. 그러나 사용자 지정 유효성 검사는 초기화할 때 프롬프트와 연결합니다. 따라서 동일한 유형의 프롬프트에 대해 서로 다른 유효성 검사를 사용하려면, 각각 고유한 유효성 검사 코드가 있는 프롬프트 유형의 여러 인스턴스가 필요합니다.

### <a name="define-a-state-property-accessor-for-the-dialog-state"></a>대화 상태에 대한 상태 속성 접근자 정의

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

이 문서에서 사용된 대화 프롬프트 샘플은 사용자에게 예약 정보를 묻는 메시지를 표시합니다. 파티의 크기와 날짜를 관리하기 위해 DialogPromptBot.cs 파일에서 예약 정보에 대한 내부 클래스를 정의합니다.

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Location { get; set; }

    public string Date { get; set; }
}
```

다음으로, 예약 데이터에 대한 상태 속성 접근자를 추가합니다.

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; }
        = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; }
        = "DialogPromptBotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<DialogPromptBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

Startup.cs에서 접근자를 설정하도록 `ConfigureServices` 메서드를 업데이트합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    IStorage dataStore = new MemoryStorage();
    var conversationState = new ConversationState(dataStore);

    // Create and register state accesssors.
    services.AddSingleton<DialogPromptBotAccessors>(sp =>
    {
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new DialogPromptBotAccessors(conversationState)
        {
            DialogStateAccessor =
                conversationState.CreateProperty<DialogState>(
                    DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor =
                conversationState.CreateProperty<DialogPromptBot.Reservation>(
                    DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

JavaScript에 필요한 HTTP 서비스 코드는 변경하지 않으므로 index.js 파일을 있는 그대로 둘 수 있습니다.

bot.js에는 대화 프롬프트 봇에 필요한 `require` 문이 포함되어 있습니다.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus }
    = require('botbuilder-dialogs');
```

상태 속성 접근자, 대화 및 프롬프트의 식별자를 추가합니다.

```javascript
// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const SIZE_RANGE_PROMPT = 'rangePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>대화 세트 및 프롬프트 만들기

일반적으로 봇을 초기화할 때 대화 세트에 프롬프트와 대화를 만들고 추가합니다. 그러면 봇에서 사용자로부터 입력을 받을 때 대화 집합에서 프롬프트의 ID를 확인할 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`DialogPromptBot` 클래스에서 대화, 프롬프트 및 대화 내에서 추적할 값을 정의합니다.

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partySizePrompt";
private const string SizeRangePrompt = "sizeRangePrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

// Define keys for tracked values within the dialog.
private const string LocationKey = "location";
private const string PartySizeKey = "partySize";
```

봇의 생성자에서 대화 세트를 만들고, 프롬프트를 추가하고, 예약 대화를 추가합니다. 프롬프트를 만들 때 사용자 지정 유효성 검사를 포함시킵니다. 나중에 유효성 검사 함수를 구현합니다.

```csharp
private readonly DialogSet _dialogSet;
private readonly DialogPromptBotAccessors _accessors;

// ...

// Initializes a new instance of the <see cref="DialogPromptBot"/> class.
public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...

    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);

    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
    _dialogSet.Add(new ChoicePrompt(LocationPrompt));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForLocationAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };

    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

생성자에서 상태 접근자 속성을 만듭니다.
다음으로, 대화 세트를 만들고, 사용자 지정 유효성 검사를 포함한 프롬프트를 추가합니다.
그런 다음, 폭포 대화의 단계를 정의하고, 세트에 추가합니다.

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
    this.dialogSet.add(new ChoicePrompt(LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForLocation.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-dialog-steps"></a>대화 단계 구현

기본 봇 파일에서 폭포 대화의 각 단계를 실행합니다. 프롬프트가 추가되면 폭포 대화의 한 단계에서 이를 호출하고, 다음 대화 단계에서 프롬프트 결과를 얻습니다. 폭포 단계 내에서 프롬프트를 호출하려면 _폭포 단계 컨텍스트_ 개체의 _prompt_ 메서드를 호출합니다. 첫 번째 매개 변수는 사용할 프롬프트의 ID이고, 두 번째 매개 변수는 사용자에게 입력을 요청하는 데 사용되는 텍스트와 같은 프롬프트에 대한 옵션을 포함합니다.

이러한 메서드는 다음을 보여줍니다.

- 폭포 단계에서 프롬프트를 호출하는 방법(_프롬프트 옵션_을 전달하는 방법 포함).
- _validations_ 속성을 사용하여 사용자 지정 유효성 검사기에 추가 매개 변수를 제공하는 방법.
- _choices_ 속성을 사용하여 선택 프롬프트에 옵션을 제공하는 방법.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**DialogPromptBot.cs** 파일에서 폭포 대화의 단계를 구현합니다.

여기서는 폭포의 `PromptForPartySizeAsync` 및 `PromptForLocationAsync`라는 처음 두 단계를 보여줍니다.

```csharp
// First step of the main dialog: prompt for party size.
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        SizeRangePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
            Validations = new Range { Min = 3, Max = 8 },
        },
        cancellationToken);
}

// Second step of the main dialog: prompt for location.
private async Task<DialogTurnResult> PromptForLocationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    var size = (int)stepContext.Result;
    stepContext.Values[PartySizeKey] = size;

    // Prompt for the location.
    return await stepContext.PromptAsync(
        LocationPrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bot.js** 파일에서 폭포 대화의 단계를 구현합니다.

여기서는 폭포의 `promptForPartySize` 및 `promptForLocation`라는 처음 두 단계를 보여줍니다.

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        SIZE_RANGE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?',
            validations: { min: 3, max: 8 },
        });
}

async promptForLocation(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for location.
    return await stepContext.prompt(LOCATION_PROMPT, {
        prompt: 'Please choose a location.',
        retryPrompt: 'Sorry, please choose a location from the list.',
        choices: ['Redmond', 'Bellevue', 'Seattle'],
    });
}
```

---

_prompt_ 메서드의 두 번째 매개 변수는 다음과 같은 속성이 있는 _프롬프트 옵션_ 개체를 사용합니다.

| 자산 | 설명 |
| :--- | :--- |
| _prompt_ | 사용자 입력을 요청하기 위해 사용자에게 보내는 초기 활동입니다. |
| _retryPrompt_ | 첫 번째 입력의 유효성을 검사하지 않은 경우 사용자에게 보내는 활동입니다. |
| _choices_ | 선택 항목 프롬프트에서 사용하기 위해 사용자가 선택할 수 있는 선택 항목 목록입니다. |
| _validations_ | 사용자 지정 유효성 검사기에 사용할 추가 매개 변수입니다. |

일반적으로 prompt 및 retryPrompt(다시 시도 프롬프트) 속성은 활동이지만, 활동이 다른 프로그래밍 언어로 처리되는 방법에는 약간의 차이가 있습니다.

사용자에게 보낼 초기 프롬프트 활동은 항상 지정해야 합니다.

다시 시도 프롬프트를 지정하는 것은 프롬프트에서 구문 분석할 수 없는 형식(예: 숫자 프롬프트에 대한 "내일")이거나 입력이 유효성 검사 조건에 실패하여 사용자 입력의 유효성을 검사하는 데 실패하는 경우에 유용합니다. 이 경우 다시 시도 프롬프트가 제공되지 않으면 프롬프트에서 초기 프롬프트 활동을 사용하여 사용자에게 다시 입력하도록 요청하는 프롬프트를 표시합니다.

선택 항목 프롬프트의 경우 항상 사용 가능한 선택 항목 목록을 제공해야 합니다.

## <a name="custom-validation"></a>사용자 지정 유효성 검사

값을 **폭포**의 다음 단계로 반환하기 전에 프롬프트 응답의 유효성을 검사할 수 있습니다. 유효성 검사기 함수는 _프롬프트 유효성 검사기 컨텍스트_ 매개 변수를 사용하며, 입력이 유효성 검사를 통과하는지 여부를 나타내는 부울을 반환합니다.

프롬프트 유효성 검사기 컨텍스트에 포함되는 속성은 다음과 같습니다.

| 자산 | 설명 |
| :--- | :--- |
| _컨텍스트_ | 봇에 대한 현재 턴 컨텍스트입니다. |
| _Recognized_ | 인식기에서 처리한 사용자 입력에 대한 정보가 포함된 _프롬프트 인식기 결과_입니다. |
| _옵션_ | 프롬프트를 시작하는 호출에 제공된 _프롬프트 옵션_을 포함합니다. |

프롬프트 인식기 결과에 포함되는 속성은 다음과 같습니다.

| 자산 | 설명 |
| :--- | :--- |
| _성공함_ | 인식기에서 입력을 구문 분석할 수 있는지 여부를 나타냅니다. |
| _값_ | 인식기에서 반환하는 값입니다. 필요한 경우 유효성 검사 코드에서 이 값을 수정할 수 있습니다. |

### <a name="implement-validation-code"></a>유효성 검사 코드 구현

초기화 시 봇의 생성자에서 프롬프트와 사용자 지정 유효성 검사를 연결하세요.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// ...
_dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
// ...
_dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));
// ...
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// ...
this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
// ...
this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
// ...
```

---

**파티 크기 유효성 검사기**

예약을 수행할 수 있는 파티의 크기를 제한합니다. 유효한 범위는 파티 크기 프롬프트를 호출하는 데 사용된 _validations_ 속성으로 정의됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the party size is appropriate to make a reservation.
private async Task<bool> RangeValidatorAsync(
    PromptValidatorContext<int> promptContext,
    CancellationToken cancellationToken)
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the number of people in your party.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.Recognized.Value;
    var validRange = promptContext.Options.Validations as Range;
    if (size < validRange.Min || size > validRange.Max)
    {
        await promptContext.Context.SendActivitiesAsync(
            new Activity[]
            {
                MessageFactory.Text($"Sorry, we can only take reservations for parties " +
                    $"of {validRange.Min} to {validRange.Max}."),
                promptContext.Options.RetryPrompt,
            },
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async rangeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    else if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < promptContext.options.validations.min
        || size > promptContext.options.validations.max) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of '
            + `${promptContext.options.validations.min} to `
            + `${promptContext.options.validations.max}.`);
        await promptContext.context.sendActivity(promptContext.options.retryPrompt);
        return false;
    }

    return true;
}
```

---

**날짜 시간 유효성 검사**

예약 날짜 유효성 검사기에서 예약을 현재 시간보다 1시간 이상으로 제한합니다. 이 조건과 일치하는 첫 번째 확인은 유지하고, 나머지는 지웁니다.

이러한 유효성 검사 코드는 완전하지 않으며, 날짜 및 시간으로 구문 분석하는 입력에 가장 적합합니다. 날짜-시간 프롬프트의 유효성 검사를 위한 옵션 중 일부를 보여 주며, 구현은 사용자로부터 수집하려는 정보에 따라 달라집니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
private async Task<bool> DateValidatorAsync(
    PromptValidatorContext<IList<DateTimeResolution>> promptContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.",
            cancellationToken: cancellationToken);

        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    var earliest = DateTime.Now.AddHours(1.0);
    var value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out var time) && DateTime.Compare(earliest, time) <= 0);

    if (value != null)
    {
        promptContext.Recognized.Value.Clear();
        promptContext.Recognized.Value.Add(value);
        return true;
    }

    await promptContext.Context.SendActivityAsync(
            "I'm sorry, we can't take reservations earlier than an hour from now.",
            cancellationToken: cancellationToken);

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async dateValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.");
        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    const earliest = Date.now() + (60 * 60 * 1000);
    let value = null;
    promptContext.recognized.value.forEach(candidate => {
        // TODO: update validation to account for time vs date vs date-time vs range.
        const time = new Date(candidate.value || candidate.start);
        if (earliest < time.getTime()) {
            value = candidate;
        }
    });
    if (value) {
        promptContext.recognized.value = [value];
        return true;
    }

    await promptContext.context.sendActivity(
        "I'm sorry, we can't take reservations earlier than an hour from now.");
    return false;
}
```

---

날짜-시간 프롬프트에서 사용자 입력과 일치하는 가능한 _날짜-시간 확인_의 목록 또는 배열을 반환합니다. 예를 들어 9:00는 오전 9시 또는 오후 9시를 의미할 수 있으며, 일요일도 모호합니다. 또한 날짜-시간 확인은 날짜, 시간, 날짜-시간 또는 범위를 나타낼 수 있습니다. 날짜-시간 프롬프트에서 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text)를 사용하여 사용자 입력을 구문 분석합니다.

## <a name="update-the-turn-handler"></a>순서 처리기 업데이트

대화를 시작하고 대화가 완료되면 대화의 반환 값을 수락하도록 봇의 턴 처리기를 업데이트합니다. 여기서는 사용자가 봇과 상호 작용하고, 봇에 활성 폭포 대화가 있고, 대화의 다음 단계에서 프롬프트를 사용한다고 가정합니다.

사용자가 봇에 메시지를 보내는 경우 다음 작업이 수행됩니다.

1. 봇이 상태 정보를 검색합니다.
1. 봇이 대화 컨텍스트를 만듭니다.
    - 활성 대화가 없으면 봇이 대화를 시작합니다. 단, 사용자가 이미 예약을 수행한 경우는 예외입니다.
    - 활성 대화가 있으면 봇이 해당 대화를 계속합니다. 대화가 종료되면 예약 정보가 상태 캐시에 기록됩니다.
1. 봇이 상태 변경 내용을 저장합니다.

대화의 단계가 단계 컨텍스트의 _prompt_ 메서드를 호출할 경우:

1. 프롬프트의 새 인스턴스가 생성되고, 대화 스택에 배치되고, 시작됩니다. (계속하기 전에 주 대화 상자가 프롬프트가 종료될 때까지 대기합니다.)
1. 프롬프트에서 사용자에게 입력을 요청하는 활동을 보냅니다.

프롬프트에 입력이 전송될 경우:

1. 프롬프트에서 프롬프트의 유형(예: 숫자 프롬프트 또는 선택 프롬프트 등)에 따라 입력 처리를 시도합니다.
1. 프롬프트에 사용자 지정 유효성 검사가 포함된 경우 사용자 지정 유효성 검사 코드가 실행됩니다.
1. 입력에서 모든 유효성 검사를 전달하면 프롬프트가 종료되고 처리된 입력이 반환됩니다. 그렇지 않으면 프롬프트가 다시 시작됩니다.

**프롬프트 결과 처리**

프롬프트 결과를 사용하여 수행하는 작업은 사용자에게 정보를 요청한 이유에 따라 달라집니다. 옵션은 다음과 같습니다.

- 정보를 사용하여 사용자가 확인 또는 선택 항목 프롬프트에 응답할 때와 같이 대화의 흐름을 제어합니다.
- 폭포 단계 컨텍스트의 _values_ 속성에 값을 설정하는 것과 같은 대화의 상태 정보를 캐시한 다음, 대화가 종료되면 수집된 정보를 반환합니다.
- 정보를 봇 상태에 저장합니다. 이렇게 하려면 봇의 상태 속성 접근자에 액세스할 수 있도록 대화를 설계해야 합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            var reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext,
                () => null,
                cancellationToken);

            // Generate a dialog context for our dialog set.
            var dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

            if (dc.ActiveDialog is null)
            {
                // If there is no active dialog, check whether we have a reservation yet.
                if (reservation is null)
                {
                    // If not, start the dialog.
                    await dc.BeginDialogAsync(ReservationDialog, null, cancellationToken);
                }
                else
                {
                    // Otherwise, send a status message.
                    await turnContext.SendActivityAsync(
                        $"We'll see you on {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                var dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.Status is DialogTurnStatus.Complete)
                {
                    reservation = (Reservation)dialogTurnResult.Result;
                    await _accessors.ReservationAccessor.SetAsync(
                        turnContext,
                        reservation,
                        cancellationToken);

                    // Send a confirmation message to the user.
                    await turnContext.SendActivityAsync(
                        $"Your party of {reservation.Size} is confirmed for " +
                        $"{reservation.Date} in {reservation.Location}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(
                turnContext, false, cancellationToken);
            break;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    switch (turnContext.activity.type) {
        case ActivityTypes.Message:
            // Get the current reservation info from state.
            const reservation = await this.reservationAccessor.get(turnContext, null);

            // Generate a dialog context for our dialog set.
            const dc = await this.dialogSet.createContext(turnContext);

            if (!dc.activeDialog) {
                // If there is no active dialog, check whether we have a reservation yet.
                if (!reservation) {
                    // If not, start the dialog.
                    await dc.beginDialog(RESERVATION_DIALOG);
                }
                else {
                    // Otherwise, send a status message.
                    await turnContext.sendActivity(
                        `We'll see you on ${reservation.date}.`);
                }
            }
            else {
                // Continue the dialog.
                const dialogTurnResult = await dc.continueDialog();

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.status === DialogTurnStatus.complete) {
                    await this.reservationAccessor.set(
                        turnContext,
                        dialogTurnResult.result);

                    // Send a confirmation message to the user.
                    await turnContext.sendActivity(
                        `Your party of ${dialogTurnResult.result.size} is ` +
                        `confirmed for ${dialogTurnResult.result.date} in ` +
                        `${dialogTurnResult.result.location}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        case ActivityTypes.EndOfConversation:
        case ActivityTypes.DeleteUserData:
            break;
        default:
            break;
    }
}
```

---

비슷한 기법을 사용하여 프롬프트 형식 중 하나에 대한 프롬프트 응답의 유효성을 검사할 수 있습니다.

## <a name="test-your-bot"></a>봇 테스트

1. 샘플을 머신에서 로컬로 실행합니다. 지침이 필요한 경우 [C#](https://aka.ms/dialog-prompt-cs) 또는 [JS](https://aka.ms/dialog-prompt-js) 샘플에 대한 README 파일을 참조하세요.
2. 에뮬레이터를 시작하고, 아래 그림과 같이 메시지를 보내 봇을 테스트합니다.

![대화 프롬프트 테스트 샘플](~/media/emulator-v4/test-dialog-prompt.png)

## <a name="additional-resources"></a>추가 리소스

턴 처리기에서 메시지를 직접 호출하려면 [C#](https://aka.ms/cs-prompt-validation-sample) 또는 [JS](https://aka.ms/js-prompt-validation-sample)로 작성된 _prompt-validations_(프롬프트 유효성 검사) 샘플을 참조하세요.

또한 대화 라이브러리에는 사용자를 대신하여 다른 애플리케이션에 액세스할 수 있는 _OAuth 토큰_을 가져오기 위한 _OAuth 프롬프트_도 포함되어 있습니다. 인증에 대한 자세한 내용은 [봇에 인증을 추가하는 방법](bot-builder-authentication.md)을 참조하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [분기 및 루프를 사용하여 고급 대화 흐름 만들기](bot-builder-dialog-manage-complex-conversation-flow.md)
