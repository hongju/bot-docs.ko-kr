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
ms.date: 11/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 57e43e6f0ad8673634bd8faafac79636a672eefd
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225848"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>대화 상자 프롬프트를 사용하여 사용자 입력 수집

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

질문을 제시하여 정보를 수집하는 것은 봇에서 사용자와 상호 작용하는 주요 방법 중 하나입니다. *대화* 라이브러리를 사용하면 질문을 쉽게 할 수 있을 뿐만 아니라 특정 데이터 형식과 일치하거나 사용자 지정 유효성 검사 규칙을 충족하는지 확인하기 위해 응답의 유효성도 검사할 수 있습니다. 이 항목에서는 폭포 대화에서 프롬프트를 만들고 호출하는 방법에 대해 자세히 설명합니다.

## <a name="prerequisites"></a>필수 조건

- 이 문서의 코드는 **DialogPromptBot** 샘플을 기반으로 합니다. [C#](https://aka.ms/dialog-prompt-cs) 또는 [JS](https://aka.ms/dialog-prompt-js)로 작성된 샘플의 복사본이 필요합니다.
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

    public string Date { get; set; }
}
```

다음으로, 예약 데이터에 대한 상태 속성 접근자를 추가합니다.

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "DialogPromptBotAccessors.Reservation";

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

    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        // ...

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<DialogPromptBot.Reservation>(DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

JavaScript에 필요한 HTTP 서비스 코드는 변경하지 않으므로 index.js 파일을 있는 그대로 둘 수 있습니다.

bot.js에는 대화 프롬프트 봇에 필요한 `require` 문이 포함되어 있습니다. 
```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus } = require('botbuilder-dialogs');
```

상태 속성 접근자, 대화 및 프롬프트의 식별자를 추가합니다.

```javascript
// Define identifiers for state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>대화 세트 및 프롬프트 만들기

일반적으로 봇을 초기화할 때 대화 세트에 프롬프트와 대화를 만들고 추가합니다. 그러면 봇에서 사용자로부터 입력을 받을 때 대화 집합에서 프롬프트의 ID를 확인할 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`DialogPromptBot` 클래스에서 대화, 프롬프트 및 대화 세트에 대한 식별자를 정의합니다.
```csharp
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

private readonly DialogSet _dialogSet;
```

봇의 생성자에서 대화 세트를 만들고, 프롬프트를 추가하고, 예약 대화를 추가합니다. 프롬프트를 만들 때 사용자 지정 유효성 검사를 포함시킵니다. 나중에 유효성 검사 함수를 구현합니다.

```csharp
// The following code creates prompts and adds them to an existing dialog set. The DialogSet contains all the dialogs that can 
// be used at runtime. The prompts also references a validation method is not shown here.

public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
   // ...

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
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

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;
    // ...
    }
```

다음으로, 대화 세트를 만들고, 사용자 지정 유효성 검사를 포함한 프롬프트를 추가합니다.

```javascript
    // ...
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, this.partySizeValidator));
    this.dialogSet.add(new ChoicePrompt (LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
    // ...
```

그런 다음, 폭포 대화의 단계를 정의하고, 세트에 추가합니다.

```javascript
    // ...
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

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

DialogPromptBot.cs 파일에서 폭포 대화의 `PromptForPartySizeAsync`, `PromptForLocationAsync`, `PromptForReservationDateAsync` 및 `AcknowledgeReservationAsync` 단계를 구현합니다.

여기서는 폭포 대화의 연속적인 두 단계 대리자인 `PromptForPartySizeAsync` 및 `PromptForLocationAsync`만 보여 줍니다.

```csharp
private async Task<DialogTurnResult> PromptForPartySizeAsync(WaterfallStepContext stepContext)
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    // If the input is not valid, the prompt is restarted, causing it to reprompt for input
    // and this set of steps is repeated next turn. Otherwise, the prompt ends and returns a _dialog turn result_ object 
    // to the parent dialog. Control passes to the next step of your waterfall dialog, with the result of the prompt 
    // available in the waterfall step context's _result_ property.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

private async Task<DialogTurnResult> PromptForLocationAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    return await stepContext.PromptAsync(
        "locationPrompt",
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```

위의 예제에서는 선택 항목 프롬프트를 사용하여 세 가지 속성을 모두 제공하는 방법을 보여 줍니다. `PromptForLocationAsync` 메서드는 폭포 대화의 한 단계로 사용되며, 대화 세트에는 폭포 대화 및 ID가 `locationPrompt`인 선택 항목 프롬프트가 모두 포함됩니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

여기서 `PARTY_SIZE_PROMPT` 및 `LOCATION_PROMPT`는 프롬프트의 ID이고, `promptForPartySize` 및 `promptForLocation`은 폭포 대화의 연속적인 두 단계 함수입니다.

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForLocation(stepContext) {
    // Prompt for location
    return await stepContext.prompt(
        LOCATION_PROMPT, 'Select a location.', ['Redmond', 'Bellevue', 'Seattle']
    );
}
```

---

_prompt_ 메서드의 두 번째 매개 변수는 다음과 같은 속성이 있는 _프롬프트 옵션_ 개체를 사용합니다.

| 자산 | 설명 |
| :--- | :--- |
| _prompt_ | 사용자 입력을 요청하기 위해 사용자에게 보내는 초기 활동입니다. |
| _retryPrompt_ | 첫 번째 입력의 유효성을 검사하지 않은 경우 사용자에게 보내는 활동입니다. |
| _choices_ | 선택 항목 프롬프트에서 사용하기 위해 사용자가 선택할 수 있는 선택 항목 목록입니다. |

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
_dialogSet = new DialogSet(_accessors.DialogStateAccessor);
_dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
_dialogSet.Add(new ChoicePrompt(LocationPrompt));
_dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));
// ...
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// ...
this.dialogSet = new DialogSet(this.dialogStateAccessor);
this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, this.partySizeValidator));
this.dialogSet.add(new ChoicePrompt (LOCATION_PROMPT));
this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
// ...
```

---

**파티 크기 유효성 검사기**

예약을 6~20명의 파티로 제한합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private async Task<bool> PartySizeValidatorAsync(
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
    int size = promptContext.Recognized.Value;
    if (size < 6 || size > 20)
    {
        await promptContext.Context.SendActivityAsync(
            "Sorry, we can only take reservations for parties of 6 to 20.",
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async partySizeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }
    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < 6 || size > 20) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of 6 to 20.');
        return false;
    }

    return true;
}
```

---

**날짜 시간 유효성 검사**

예약 날짜 유효성 검사기에서 예약을 현재 시간보다 1시간 이상으로 제한합니다. 이 조건과 일치하는 첫 번째 확인은 유지하고, 나머지는 지웁니다. 아래의 유효성 검사 코드는 완전하지 않으며, 날짜 및 시간으로 구문 분석하는 입력에 가장 적합합니다. 날짜-시간 프롬프트의 유효성 검사를 위한 옵션 중 일부를 보여 주며, 구현은 사용자로부터 수집하려는 정보에 따라 달라집니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
// Reservations must be made at least an hour in advance.
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
    DateTime earliest = DateTime.Now.AddHours(1.0);
    DateTimeResolution value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out DateTime time) && DateTime.Compare(earliest,time) <= 0);
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

### <a name="update-the-turn-handler"></a>순서 처리기 업데이트

대화를 시작하고 대화가 완료되면 대화의 반환 값을 수락하도록 봇의 턴 처리기를 업데이트합니다. 여기서는 사용자가 봇과 상호 작용하고, 봇에 활성 폭포 대화가 있고, 대화의 다음 단계에서 프롬프트를 사용한다고 가정합니다.

1. 사용자가 봇에 메시지를 보내는 경우 다음 작업이 수행됩니다.
   1. 봇의 턴 처리기에서 대화 컨텍스트를 만들고, 해당 _continue_ 메서드를 호출합니다.
   1. 컨트롤이 활성 대화(이 경우 폭포 대화)의 다음 단계로 전달됩니다.
   1. 단계에서 사용자에게 입력을 요청하는 폭포 단계 컨텍스트의 _prompt_ 메서드를 호출합니다.
   1. 폭포 단계 컨텍스트는 프롬프트를 스택으로 푸시하고 시작합니다.
   1. 프롬프트에서 사용자에게 입력을 요청하는 활동을 보냅니다.
1. 사용자가 봇에 다음 메시지를 보내는 경우 다음 작업이 수행됩니다.
   1. 봇의 턴 처리기에서 대화 컨텍스트를 만들고, 해당 _continue_ 메서드를 호출합니다.
   1. 컨트롤이 프롬프트의 두 번째 턴인 활성 대화의 다음 단계로 전달됩니다.
   1. 프롬프트에서 사용자 입력의 유효성을 검사합니다.

**프롬프트 결과 처리**

프롬프트 결과를 사용하여 수행하는 작업은 사용자에게 정보를 요청한 이유에 따라 달라집니다. 옵션은 다음과 같습니다.

- 정보를 사용하여 사용자가 확인 또는 선택 항목 프롬프트에 응답할 때와 같이 대화의 흐름을 제어합니다.
- 폭포 단계 컨텍스트의 _values_ 속성에 값을 설정하는 것과 같은 대화의 상태 정보를 캐시한 다음, 대화가 종료되면 수집된 정보를 반환합니다.
- 정보를 봇 상태에 저장합니다. 이렇게 하려면 봇의 상태 속성 접근자에 액세스할 수 있도록 대화를 설계해야 합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            Reservation reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext, () => null, cancellationToken);

            // Generate a dialog context for our dialog set.
            DialogContext dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

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
                        $"We'll see you {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

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
                        $"Your party of {reservation.Size} is confirmed for {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            break;

        // Handle other incoming activity types as appropriate to your bot.
        default:
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
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
                        `We'll see you ${reservation.date}.`);
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
                        `confirmed for ${dialogTurnResult.result.date}.`);
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
