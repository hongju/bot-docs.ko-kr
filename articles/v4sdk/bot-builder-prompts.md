---
title: 대화 상자 라이브러리를 사용하여 사용자 입력 수집 | Microsoft Docs
description: Bot Builder SDK의 대화 상자 라이브러리를 사용하여 사용자에게 입력을 요청하는 방법에 대해 알아봅니다.
keywords: prompts, dialogs, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, reprompt, validation
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 150d5f0a68d897ac278026a7cf36609aca05bb80
ms.sourcegitcommit: 984705927561cc8d6a84f811ff24c8c71b71c76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/02/2018
ms.locfileid: "50965721"
---
# <a name="use-dialog-library-to-gather-user-input"></a>대화 상자 라이브러리를 사용하여 사용자 입력 수집

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

질문을 게시하여 정보를 수집하는 것은 봇이 사용자와 상호 작용하는 주요 방법 중 하나입니다. [순서 컨텍스트](~/v4sdk/bot-builder-basics.md#defining-a-turn) 개체의 _보내기 작업_ 메서드를 사용하여 직접 이렇게 한 후 그 다음으로 들어오는 메시지를 응답으로 처리할 수 있습니다. 그러나 Bot Builder SDK는 쉽게 질문하고, 응답이 특정 데이터 형식과 일치하는지 또는 사용자 지정 유효성 검사 규칙을 충족하는지 확인할 수 있도록 설계된 메서드를 제공하는 [대화 라이브러리](bot-builder-concept-dialog.md)를 제공합니다. 이 항목에서는 프롬프트 개체를 사용하여 사용자에게 입력을 요청하는 방법을 설명합니다.

이 문서에서는 프롬프트를 만들고 대화 내에서 이를 호출하는 방법에 대해 설명합니다.
대화를 사용하지 않고 입력 프롬프트를 표시하는 방법은 [사용자 고유의 프롬프트를 사용하여 사용자에게 입력 프롬프트 표시](bot-builder-primitive-prompts.md)를 참조하세요.
일반적으로 대화를 사용하는 방법은 [대화를 사용하여 간단한 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)를 참조하세요.

## <a name="prompt-types"></a>프롬프트 형식

프롬프트는 내부적으로 두 가지 단계로 수행되는 대화입니다. 먼저 프롬프트에서 입력을 요청하며, 다음으로 유효한 값을 반환하거나 다시 프롬프트를 사용하여 처음부터 다시 시작합니다.

대화 라이브러리에서 제공하는 다양한 기본 프롬프트는 다음과 같으며, 각 프롬프트는 다양한 유형의 응답을 수집하는 데 사용됩니다.

| prompt | 설명 | 반환 |
|:----|:----|:----|
| _첨부 파일 프롬프트_ | 문서 또는 이미지와 같은 하나 이상의 첨부 파일을 요청합니다. | _첨부 파일_ 개체의 컬렉션 |
| _선택 항목 프롬프트_ | 일단의 옵션에서 선택하도록 요청합니다. | _찾은 선택 항목_ 개체 |
| _확인 프롬프트_ | 확인을 요청합니다. | 부울 값입니다. |
| _날짜-시간 프롬프트_ | 날짜-시간을 요청합니다. | _날짜-시간 확인_ 개체의 컬렉션 |
| _숫자 프롬프트_ | 숫자를 요청합니다. | 숫자 값 |
| _텍스트 프롬프트_ | 일반 텍스트 입력을 요청합니다. | 문자열입니다. |

또한 라이브러리에는 사용자를 대신하여 다른 애플리케이션에 액세스할 수 있는 _OAuth 토큰_을 가져오는 _OAuth 프롬프트_가 포함되어 있습니다. 인증에 대한 자세한 내용은 [봇에 인증을 추가하는 방법](bot-builder-authentication.md)을 참조하세요.

기본 프롬프트는 "열" 또는 "열둘"과 같은 자연어 입력을 숫자로 해석하거나 "내일" 또는 "금요일 10시"를 날짜-시간으로 해석할 수 있습니다.

## <a name="using-prompts"></a>프롬프트 사용

대화와 프롬프트가 모두 동일한 대화 집합에 있는 경우에만 대화에서 프롬프트를 사용할 수 있습니다.

1. 대화 상태에 대한 상태 속성 접근자를 정의합니다.
1. 대화 집합을 만듭니다.
1. 프롬프트를 만들고 대화 집합에 추가합니다.
1. 프롬프트를 사용할 대화를 만들고 대화 집합에 추가합니다.
1. 대화 내에서 호출을 프롬프트에 추가하고 프롬프트 결과를 검색합니다.

이 문서에서는 프롬프트를 만드는 방법과 폭포 대화에서 이를 호출하는 방법에 대해 설명합니다.
일반적으로 대화에 대한 자세한 내용은 [대화 라이브러리](bot-builder-concept-dialog.md) 문서를 참조하세요.
대화 및 프롬프트를 사용하는 완전한 봇에 대한 자세한 내용은 [대화를 사용하여 간단한 대화 흐름을 관리하는 방법](bot-builder-dialog-manage-conversation-flow.md)을 참조하세요.

대화 내의 여러 단계 및 동일한 대화 집합의 여러 대화에서 동일한 프롬프트를 사용할 수 있습니다.
그러나 사용자 지정 유효성 검사는 초기화할 때 프롬프트와 연결합니다.
따라서 동일한 유형의 프롬프트에 대해 서로 다른 유효성 검사가 필요한 경우 각각 고유한 유효성 검사 코드가 있는 프롬프트 유형의 여러 인스턴스가 필요합니다.

### <a name="create-a-prompt"></a>프롬프트 만들기

사용자에게 입력을 요청하려면 _텍스트 프롬프트_와 같은 기본 제공 클래스 중 하나를 사용하여 프롬프트를 정의하고 대화 집합에 이를 추가합니다.

* 프롬프트에는 고정 ID가 있습니다. 식별자는 대화 집합 내에서 고유해야 합니다.
* 프롬프트에는 사용자 지정 유효성 검사기가 있을 수 있습니다. [사용자 지정 유효성 검사](#custom-validation)를 참조하세요.
* 일부 프롬프트의 경우 _기본 로캘_을 지정할 수 있습니다.

일반적으로 봇을 초기화할 때 대화 집합에 프롬프트와 대화를 만들고 추가합니다. 그러면 봇에서 사용자로부터 입력을 받을 때 대화 집합에서 프롬프트의 ID를 확인할 수 있습니다.

예를 들어 다음 코드는 두 개의 텍스트 프롬프트를 만들고 기존 대화 집합에 이를 추가합니다. 두 번째 텍스트 프롬프트는 여기에 표시되지 않은 유효성 검사 메서드를 참조합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

여기서 `_dialogs`에는 기존 대화 집합이 포함되어 있으며 `NameValidator`는 유효성 검사 메서드입니다.

```csharp
_dialogs.Add(new TextPrompt("nickNamePrompt"));
_dialogs.Add(new TextPrompt("namePrompt", NameValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

여기서 `this.dialogs`에는 기존 대화 집합이 포함되어 있으며 `NameValidator`는 유효성 검사 함수입니다.

```javascript
this.dialogs.add(new TextPrompt('nickNamePrompt'));
this.dialogs.add(new TextPrompt('namePrompt', NameValidator));
```

---

#### <a name="locales"></a>Locales

로캘은 **선택 항목**, **확인**, **날짜-시간** 및 **숫자** 프롬프트의 언어별 동작을 결정하는 데 사용됩니다. 사용자가 제공한 입력은 다음과 같습니다.

* 채널이 사용자의 메시지에서 _로캘_ 속성을 제공한 경우 해당 로캘이 사용됩니다.
* 그렇지 않으면 프롬프트의 생성자를 호출하거나 나중에 설정하여 프롬프트의 _기본 로캘_이 설정되면 해당 로캘이 사용됩니다.
* 그렇지 않으면 로캘로 영어("en-us")가 사용됩니다.

> [!NOTE]
> 로캘은 언어 또는 언어 제품군을 나타내는 2, 3 또는 4개 문자의 ISO 639 코드입니다.

### <a name="call-a-prompt-from-a-waterfall-dialog"></a>폭포 대화에서 프롬프트 호출

프롬프트가 추가되면 폭포 대화의 한 단계에서 이를 호출하고, 다음 대화 단계에서 프롬프트 결과를 얻습니다.
폭포 단계 내에서 프롬프트를 호출하려면 _폭포 단계 컨텍스트_ 개체의 _prompt_ 메서드를 호출합니다. 첫 번째 매개 변수는 사용할 프롬프트의 ID이고, 두 번째 매개 변수는 사용자에게 입력을 요청하는 데 사용되는 텍스트와 같은 프롬프트에 대한 옵션을 포함합니다.

사용자가 봇과 상호 작용하고, 봇에 활성 폭포 대화가 있고, 대화의 다음 단계에서 프롬프트를 사용한다고 가정합니다.

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
      * 입력이 유효하지 않으면 입력에 대한 프롬프트가 다시 표시되고 이 단계 집합이 다음 턴에서 반복됩니다.
      * 그렇지 않으면 프롬프트가 종료되고 _대화 턴 결과_ 개체가 부모 대화에 반환됩니다. 컨트롤이 폭포 대화의 다음 단계로 전달되며, 폭포 단계 컨텍스트의 _result_ 속성에 사용할 수 있는 프롬프트 결과가 표시됩니다.

<!--
> [!NOTE]
> A waterfall step delegate takes a _waterfall step context_ parameter and returns a _dialog turn result_.
> A prompt's result is contained within the prompt's return value (a dialog turn result object) when it ends.
> The waterfall dialog provides the return value in the waterfall step context parameter when it calls the next waterfall step.
-->

프롬프트가 반환되면 폭포 단계 컨텍스트의 _result_ 속성이 프롬프트의 반환 값으로 설정됩니다.

다음 예제에서는 두 개의 연속된 폭포 단계 중 일부를 보여 줍니다. 첫 번째는 프롬프트를 사용하여 사용자에게 이름을 요청합니다. 두 번째는 프롬프트의 반환 값을 가져옵니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

여기서 `name`은 텍스트 프롬프트의 ID이고, `NameStepAsync` 및 `GreetingStepAsync`은 폭포 대화의 연속된 두 단계 대리자입니다.

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    // Prompt for the user's name.
    return await stepContext.PromptAsync(
        "name",
         new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") },
         cancellationToken);
}

private static async Task<DialogTurnResult> GreetingStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the user's name from the prompt result.
    string name = (string)stepContext.Result;
    await stepContext.Context.SendActivityAsync(
        MessageFactory.Text($"Pleased to meet you, {name}."),
         cancellationToken);

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

여기서 `name`은 텍스트 프롬프트의 ID이고, `nameStep` 및 `greetingStep`은 폭포 대화의 연속된 두 단계 함수입니다.

```javascript
async nameStep(step) {
    // ...

    return await step.prompt('name', 'Please enter your name.');
}

async greetingStep(step) {
    // Get the user's name from the prompt result.
    const name = step.result;
    await step.context.sendActivity(`Pleased to meet you, ${name}.`);

    // ...
}
```

---

### <a name="call-a-prompt-from-the-bots-turn-handler"></a>봇의 턴 처리기에서 프롬프트 호출

대화 컨텍스트의 _prompt_ 메서드를 사용하여 턴 처리기에서 프롬프트를 직접 호출할 수 있습니다.
다음 턴에서 대화 컨텍스트의 _대화 계속_ 메서드를 호출하고 반환 값, 즉 _대화 턴 결과_ 개체를 검토해야 합니다. 이렇게 수행하는 방법에 대한 예제는 프롬프트 유효성 검사 샘플([C#](https://aka.ms/cs-prompt-validation-sample) | [JS](https://aka.ms/js-prompt-validation-sample))을 참조하거나 [사용자 고유의 프롬프트를 사용하여 사용자에게 입력 프롬프트를 표시하는 방법](bot-builder-primitive-prompts.md)을 참조하세요.

## <a name="prompt-options"></a>프롬프트 옵션

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

다음 예제에서는 선택 항목 프롬프트를 사용하여 세 가지 속성을 모두 제공하는 방법을 보여 줍니다. _좋아하는 색_ 메서드는 폭포 대화의 한 단계로 사용되며, 대화 집합에는 폭포 대화와 `colorChoice`의 ID가 있는 선택 항목 프롬프트가 모두 포함됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    return await stepContext.PromptAsync(
        "colorChoice",
        new PromptOptions {
            Prompt = MessageFactory.Text("Please choose a color."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a color from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

JavaScript SDK에서는 `prompt` 및 `retryPrompt` 속성 모두에 대한 문자열을 제공할 수 있습니다. 프롬프트에서 이러한 문자열을 메시지 활동으로 변환합니다.

```javascript
async favoriteColor(step) {
    // ...

    return await step.prompt('colorChoice', {
        prompt: 'Please choose a color:',
        retryPrompt: 'Sorry, please choose a color from the list.',
        choices: [ 'red', 'green', 'blue' ]
    });
}
```

---

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

### <a name="setup"></a>설정

유효성 검사 코드를 추가하기 전에 약간의 설정을 지정해야 합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

봇의 **.cs** 파일에서 예약 정보에 대한 내부 클래스를 정의합니다.

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Date { get; set; }
}
```

**BotAccessors.cs**에서 예약 데이터에 대한 상태 속성 접근자를 추가합니다.

```csharp
public class BotAccessors
{
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "BotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "BotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<ReservationBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

**Startup.cs**에서 접근자를 설정하도록 `ConfigureServices`를 업데이트합니다.

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
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<ReservationBot.Reservation>(BotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

JavaScript에 필요한 HTTP 서비스 코드는 변경하지 않으므로 **index.js** 파일을 그대로 둘 수 있습니다.

**bot.js**에서 require 문을 업데이트하고 상태 속성 접근자에 대한 식별자를 추가합니다.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, DialogTurnStatus } = require('botbuilder-dialogs');

// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';
```

---

봇 파일에서 대화와 프롬프트에 대한 식별자를 추가합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="define-the-prompts-and-dialogs"></a>프롬프트 및 대화 정의

봇의 생성자 코드에서 대화 집합을 만들고, 프롬프트를 추가하고, 예약 대화를 추가합니다.
프롬프트를 만들 때 사용자 지정 유효성 검사를 포함시킵니다. 다음으로, 유효성 검사 함수를 구현합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public ReservationBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };
    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, partySizeValidator));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-validation-code"></a>유효성 검사 코드 구현

파티 크기 유효성 검사기를 구현합니다. 예약은 6-20명의 사용자로 제한합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>Validates whether the party size is appropriate to make a reservation.</summary>
/// <param name="promptContext">The validation context.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>Reservations can be made for groups of 6 to 20 people.
/// If the task is successful, the result indicates whether the input was valid.</remarks>
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

날짜-시간 프롬프트에서 사용자 입력과 일치하는 가능한 _날짜-시간 확인_의 목록 또는 배열을 반환합니다. 예를 들어 9:00는 오전 9시 또는 오후 9시를 의미할 수 있으며, 일요일도 모호합니다. 또한 날짜-시간 확인은 날짜, 시간, 날짜-시간 또는 범위를 나타낼 수 있습니다. 날짜-시간 프롬프트에서 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text)를 사용하여 사용자 입력을 구문 분석합니다.

예약 날짜 유효성 검사기를 구현합니다. 예약은 현재 시간에서 한 시간 이상으로 제한합니다. 이 조건과 일치하는 첫 번째 확인은 유지하고, 나머지는 지웁니다.

이 유효성 검사 코드는 완전하지 않습니다. 이는 날짜와 시간을 구문 분석하는 입력에 가장 적합합니다. 날짜-시간 프롬프트의 유효성 검사에 대한 옵션 중 일부를 보여 주며, 구현은 사용자로부터 수집하려는 정보에 따라 달라집니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>Validates whether the reservation date is appropriate.</summary>
/// <param name="promptContext">The validation context.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>Reservations must be made at least an hour in advance.
/// If the task is successful, the result indicates whether the input was valid.</remarks>
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

### <a name="implement-the-dialog-steps"></a>대화 단계 구현

대화 집합에 추가한 프롬프트를 사용합니다. 봇의 생성자에서 프롬프트를 만들 때 해당 프롬프트에 대한 유효성 검사를 추가했습니다. 프롬프트에서 처음으로 사용자 입력을 요청하면 제공된 옵션에서 _prompt_ 활동을 보냅니다. 유효성 검사에 실패하면 _다시 시도 프롬프트_ 활동을 보내 사용자에게 다른 입력을 요청합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>First step of the main dialog: prompt for party size.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

/// <summary>Second step of the main dialog: record the party size and prompt for the
/// reservation date.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> PromptForReservationDateAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        ReservationDatePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Great. When will the reservation be for?"),
            RetryPrompt = MessageFactory.Text("What time should we make your reservation for?"),
        },
        cancellationToken);
}

/// <summary>Third step of the main dialog: return the collected party size and reservation date.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> AcknowledgeReservationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve the reservation date.
    DateTimeResolution resolution = (stepContext.Result as IList<DateTimeResolution>).First();
    string time = resolution.Value ?? resolution.Start;

    // Send an acknowledgement to the user.
    await stepContext.Context.SendActivityAsync(
        "Thank you. We will confirm your reservation shortly.",
        cancellationToken: cancellationToken);

    // Return the collected information to the parent context.
    Reservation reservation = new Reservation
    {
        Date = time,
        Size = (int)stepContext.Values["size"],
    };
    return await stepContext.EndDialogAsync(reservation, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForReservationDate(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        RESERVATION_DATE_PROMPT, {
            prompt: 'Great. When will the reservation be for?',
            retryPrompt: 'What time should we make your reservation for?'
        });
}

async acknowledgeReservation(stepContext) {
    // Retrieve the reservation date.
    const resolution = stepContext.result[0];
    const time = resolution.value || resolution.start;

    // Send an acknowledgement to the user.
    await stepContext.context.sendActivity(
        'Thank you. We will confirm your reservation shortly.');

    // Return the collected information to the parent context.
    return await stepContext.endDialog({ date: time, size: stepContext.values.size });
}
```

---

### <a name="update-the-turn-handler"></a>순서 처리기 업데이트

대화를 시작하고 대화가 완료되면 대화의 반환 값을 수락하도록 봇의 턴 처리기를 업데이트합니다.

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
        default:
            break;
    }
}
```

---

추가 예제는 [리포지토리 샘플](https://aka.ms/bot-samples-readme)에서 찾을 수 있습니다.

비슷한 기법을 사용하여 프롬프트 형식 중 하나에 대한 프롬프트 응답의 유효성을 검사할 수 있습니다.

## <a name="handling-prompt-results"></a>프롬프트 결과 처리

프롬프트 결과를 사용하여 수행하는 작업은 사용자에게 정보를 요청한 이유에 따라 달라집니다. 옵션은 다음과 같습니다.

* 정보를 사용하여 사용자가 확인 또는 선택 항목 프롬프트에 응답할 때와 같이 대화의 흐름을 제어합니다.
* 폭포 단계 컨텍스트의 _values_ 속성에 값을 설정하는 것과 같은 대화의 상태 정보를 캐시한 다음, 대화가 종료되면 수집된 정보를 반환합니다.
* 정보를 봇 상태에 저장합니다. 이렇게 하려면 봇의 상태 속성 접근자에 액세스할 수 있도록 대화를 설계해야 합니다.

이러한 시나리오를 다루는 항목 및 샘플에 대한 추가 리소스를 참조하세요.

## <a name="additional-resources"></a>추가 리소스

* [간단한 대화 흐름 관리](bot-builder-dialog-manage-conversation-flow.md)
* [복잡한 대화 흐름 관리](bot-builder-dialog-manage-complex-conversation-flow.md)
* [대화 상자의 통합된 집합 만들기](bot-builder-compositcontrol.md)
* [대화 상자에 데이터 유지](bot-builder-tutorial-persist-user-inputs.md)
* **다중 턴 프롬프트** 샘플([C#](https://aka.ms/cs-multi-prompts-sample) | [JS](https://aka.ms/js-multi-prompts-sample))

## <a name="next-steps"></a>다음 단계

사용자에게 입력을 확인하는 방법에 대해 배웠으므로 대화 상자를 통해 다양한 대화 흐름을 관리하여 봇 코드 및 사용자 환경을 향상시켜보겠습니다.

> [!div class="nextstepaction"]
> [복잡한 대화 흐름 관리](bot-builder-dialog-manage-complex-conversation-flow.md)
