---
title: 사용자 입력을 수집하도록 고유한 메시지 만들기 | Microsoft Docs
description: Bot Framework SDK에서 기본 프롬프트를 사용하여 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 대화 흐름, 프롬프트, 대화 상태, 사용자 상태, 사용자 지정 프롬프트
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4b18cc5d32d04b69fa349d22058b51fcec0e12d7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591074"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>사용자 입력을 수집하도록 고유한 메시지 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇과 사용자 간의 대화에는 종종 사용자에게 정보를 요청하고(프롬프트), 사용자의 응답을 구문 분석한 다음, 해당 정보에 대한 작업을 수행하는 것이 포함됩니다.

봇은 대화의 컨텍스트를 추적하여 해당 동작을 관리하고 이전의 질문에 대한 답변을 기억할 수 있어야 합니다. 봇의 *상태*는 들어오는 메시지에 적절하게 응답하기 위해 추적하는 정보입니다.

## <a name="prerequisites"></a>필수 조건

- 이 문서의 코드는 **사용자 입력 프롬프트** 샘플을 기반으로 합니다. [C#](https://aka.ms/cs-primitive-prompt-sample) 또는 [JS](https://aka.ms/js-primitive-prompt-sample)로 작성된 샘플의 복사본이 필요합니다.
- [관리 상태](bot-builder-concept-state.md) 및 [사용자 및 대화 데이터를 저장](bot-builder-howto-v4-state.md)하는 방법에 대한 지식이 필요합니다.
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)를 사용하여 봇을 로컬로 테스트합니다.

## <a name="about-the-sample-code"></a>샘플 코드 정보

이 문서에서는 사용자에게 일련의 질문을 하고, 몇 가지 답변의 유효성을 검사하며, 입력을 저장합니다.
봇의 턴 처리기, 사용자 및 대화 상태 속성을 사용하여 대화 흐름과 입력 수집을 관리합니다.

1. 상태 정의 및 구성
1. 상태 속성을 사용하여 대화에 지시
   1. 봇의 턴 처리기를 업데이트합니다.
   1. 도우미 메서드를 구현하여 사용자 데이터 수집을 관리합니다.
   1. 사용자 입력에 대한 유효성 검사 메서드를 구현합니다.

## <a name="define-and-configure-state"></a>상태 정의 및 구성

다음과 같은 정보를 추적하도록 봇을 구성해야 합니다.

- 사용자 이름, 나이 및 선택한 날짜(사용자 상태에 정의함)
- 방금 사용자에게 요청한 내용(대화 상태에 정의함)

이 봇은 배포하지 않으므로 _메모리 스토리지_를 사용하도록 사용자 및 대화 상태를 모두 구성합니다. 아래에서는 구성 코드의 몇 가지 주요 측면에 대해 설명합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

다음 형식을 정의합니다.

- 봇에서 수집할 사용자 정보에 대한 `UserProfile` 클래스입니다.
- 대화 중인 위치에 대한 정보를 압정으로 표시하는 `ConversationFlow` 클래스입니다.
- 대화 중인 위치를 추적하는 `ConversationFlow.Question` 내부 열거형입니다.
- 상태 관리 정보를 번들로 묶는 `CustomPromptBotAccessors` 클래스입니다.

봇 접근자 클래스는 상태 관리 및 상태 속성 접근자 개체를 포함하며, ASP.NET Core의 종속성 주입을 통해 봇에 전달됩니다. 봇에서는 봇이 각 턴에 만들어질 때 받는 상태 속성 접근자 정보를 기록합니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

상태 관리 대상을 만들고, 봇을 만들 때 이를 전달합니다.
봇에서는 상태 속성에 대한 식별자를 정의하고, 대화 중인 위치를 추적한 다음, 상태 관리 개체를 기록하고, 봇의 생성자에 상태 속성 접근자를 만듭니다.

---

## <a name="use-state-properties-to-direct-the-conversation"></a>상태 속성을 사용하여 대화에 지시

상태 속성이 구성되면 봇에서 이를 사용할 수 있습니다.

- [턴 처리기](#the-bots-turn-handler)를 정의하여 상태에 액세스하고 도우미 메서드를 호출합니다.
- [도우미 메서드](#filling-out-the-user-profile)를 구현하여 사용자 프로필 수집을 관리합니다.
- [유효성 검사 메서드](#parse-and-validate-input)를 구현하여 사용자 입력을 구문 분석하고 유효성을 검사합니다.

### <a name="the-bots-turn-handler"></a>봇의 턴 처리기

상태 속성 접근자를 사용하여 턴 컨텍스트에서 상태 속성을 가져옵니다.
사용자 프로필을 작성해야 하는 경우 도우미 메서드를 호출한 다음, 상태 변경 내용을 저장합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**CustomPromptBot.cs**에서 상태 속성을 가져오고 도우미 메서드를 호출합니다. (봇의 생성자에 `_accessors` 인스턴스 속성이 설정되어 있습니다.)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
   if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        ConversationFlow flow = await _accessors.ConversationFlowAccessor.GetAsync(turnContext, () => new ConversationFlow());
        UserProfile profile = await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());

        await FillOutUserProfileAsync(flow, profile, turnContext);

        // Update state and save changes.
        await _accessors.ConversationFlowAccessor.SetAsync(turnContext, flow);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfileAccessor.SetAsync(turnContext, profile);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bot.js**에서 상태 속성을 가져오고 도우미 메서드를 호출합니다. (봇의 생성자에 `conversationFlow` 인스턴스 속성이 설정되어 있습니다.)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    // This bot listens for message activities.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const flow = await this.conversationFlow.get(turnContext, { lastQuestionAsked: question.none });
        const profile = await this.userProfile.get(turnContext, {});

        await MyBot.fillOutUserProfile(flow, profile, turnContext);

        // Update state and save changes.
        await this.conversationFlow.set(turnContext, flow);
        await this.conversationState.saveChanges(turnContext);

        await this.userProfile.set(turnContext, profile);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

### <a name="filling-out-the-user-profile"></a>사용자 프로필 작성

먼저 정보를 수집하여 시작하겠습니다. 각각은 비슷한 인터페이스를 제공합니다.

- 반환 값은 입력이 이 질문에 대한 유효한 답변인지 여부를 나타냅니다.
- 유효성 검사를 통과하면 구문 분석되고 정규화된 값이 생성되어 저장됩니다.
- 유효성 검사가 실패하면 봇에서 해당 정보를 다시 요청할 수 있는 메시지를 생성합니다.

 [다음 섹션](#parse-and-validate-input)에서는 사용자 입력을 구문 분석하고 유효성을 검사하는 도우미 메서드를 정의합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task FillOutUserProfileAsync(ConversationFlow flow, UserProfile profile, ITurnContext turnContext)
{
    string input = turnContext.Activity.Text?.Trim();
    string message;
    switch (flow.LastQuestionAsked)
    {
        case ConversationFlow.Question.None:
            await turnContext.SendActivityAsync("Let's get started. What is your name?");
            flow.LastQuestionAsked = ConversationFlow.Question.Name;
            break;
        case ConversationFlow.Question.Name:
            if (ValidateName(input, out string name, out message))
            {
                profile.Name = name;
                await turnContext.SendActivityAsync($"Hi {profile.Name}.");
                await turnContext.SendActivityAsync("How old are you?");
                flow.LastQuestionAsked = ConversationFlow.Question.Age;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Age:
            if (ValidateAge(input, out int age, out message))
            {
                profile.Age = age;
                await turnContext.SendActivityAsync($"I have your age as {profile.Age}.");
                await turnContext.SendActivityAsync("When is your flight?");
                flow.LastQuestionAsked = ConversationFlow.Question.Date;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Date:
            if (ValidateDate(input, out string date, out message))
            {
                profile.Date = date;
                await turnContext.SendActivityAsync($"Your cab ride to the airport is scheduled for {profile.Date}.");
                await turnContext.SendActivityAsync($"Thanks for completing the booking {profile.Name}.");
                await turnContext.SendActivityAsync($"Type anything to run the bot again.");
                flow.LastQuestionAsked = ConversationFlow.Question.None;
                profile = new UserProfile();
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
    }
}


```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Manages the conversation flow for filling out the user's profile.
static async fillOutUserProfile(flow, profile, turnContext) {
    const input = turnContext.activity.text;
    let result;
    switch (flow.lastQuestionAsked) {
        // If we're just starting off, we haven't asked the user for any information yet.
        // Ask the user for their name and update the conversation flag.
        case question.none:
            await turnContext.sendActivity("Let's get started. What is your name?");
            flow.lastQuestionAsked = question.name;
            break;

        // If we last asked for their name, record their response, confirm that we got it.
        // Ask them for their age and update the conversation flag.
        case question.name:
            result = this.validateName(input);
            if (result.success) {
                profile.name = result.name;
                await turnContext.sendActivity(`I have your name as ${profile.name}.`);
                await turnContext.sendActivity('How old are you?');
                flow.lastQuestionAsked = question.age;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for their age, record their response, confirm that we got it.
        // Ask them for their date preference and update the conversation flag.
        case question.age:
            result = this.validateAge(input);
            if (result.success) {
                profile.age = result.age;
                await turnContext.sendActivity(`I have your age as ${profile.age}.`);
                await turnContext.sendActivity('When is your flight?');
                flow.lastQuestionAsked = question.date;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for a date, record their response, confirm that we got it,
        // let them know the process is complete, and update the conversation flag.
        case question.date:
            result = this.validateDate(input);
            if (result.success) {
                profile.date = result.date;
                await turnContext.sendActivity(`Your cab ride to the airport is scheduled for ${profile.date}.`);
                await turnContext.sendActivity(`Thanks for completing the booking ${profile.name}.`);
                await turnContext.sendActivity('Type anything to run the bot again.');
                flow.lastQuestionAsked = question.none;
                profile = {};
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }
    }
}
```

---

### <a name="parse-and-validate-input"></a>입력 구문 분석 및 유효성 검사

다음 조건을 사용하여 입력의 유효성을 검사합니다.

- **name**(이름)은 비어 있지 않은 문자열이어야 합니다. 공백을 잘라서 정규화합니다.
- **age**(나이)는 18~120여야 합니다. 정수를 반환하여 정규화합니다.
- **date**(날짜)는 앞으로 한 시간 이내의 날짜 또는 시간이어야 합니다.
  구문 분석된 입력의 날짜 부분만 반환하여 정규화합니다.

> [!NOTE]
> age 및 date 입력의 경우 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) 라이브러리를 사용하여 초기 구문 분석을 수행합니다.
> 샘플 코드를 제공하지만 텍스트 인식기 라이브러리의 작동 방식을 설명하지 않으며, 이는 입력을 구문 분석하는 한 가지 방법일 뿐입니다.
> 이러한 라이브러리에 대한 자세한 내용은 리포지토리의 **README**를 참조하세요.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

다음 유효성 검사 메서드를 봇에 추가합니다.

```csharp
private static bool ValidateName(string input, out string name, out string message)
{
    name = null;
    message = null;

    if (string.IsNullOrWhiteSpace(input))
    {
        message = "Please enter a name that contains at least one character.";
    }
    else
    {
        name = input.Trim();
    }

    return message is null;
}

private static bool ValidateAge(string input, out int age, out string message)
{
    age = 0;
    message = null;

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try
    {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        List<ModelResult> results = NumberRecognizer.RecognizeNumber(input, Culture.English);
        foreach (var result in results)
        {
            // result.Resolution is a dictionary, where the "value" entry contains the processed string.
            if (result.Resolution.TryGetValue("value", out object value))
            {
                age = Convert.ToInt32(value);
                if (age >= 18 && age <= 120)
                {
                    return true;
                }
            }
        }

        message = "Please enter an age between 18 and 120.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120.";
    }

    return message is null;
}

private static bool ValidateDate(string input, out string date, out string message)
{
    date = null;
    message = null;

    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try
    {
        List<ModelResult> results = DateTimeRecognizer.RecognizeDateTime(input, Culture.English);

        // Check whether any of the recognized date-times are appropriate,
        // and if so, return the first appropriate date-time. We're checking for a value at least an hour in the future.
        DateTime earliest = DateTime.Now.AddHours(1.0);
        foreach (ModelResult result in results)
        {
            // result.Resolution is a dictionary, where the "values" entry contains the processed input.
            var resolutions = result.Resolution["values"] as List<Dictionary<string, string>>;
            foreach (var resolution in resolutions)
            {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                if (resolution.TryGetValue("value", out string dateString)
                    || resolution.TryGetValue("start", out dateString))
                {
                    if (DateTime.TryParse(dateString, out var candidate)
                        && earliest < candidate)
                    {
                        date = candidate.ToShortDateString();
                        return true;
                    }
                }
            }
        }

        message = "I'm sorry, please enter a date at least an hour out.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out.";
    }

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

다음 유효성 검사 메서드를 봇에 추가합니다.

```javascript
// Validates name input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateName(input) {
    const name = input && input.trim();
    return name != undefined
        ? { success: true, name: name }
        : { success: false, message: 'Please enter a name that contains at least one character.' };
};

// Validates age input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateAge(input) {

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        const results = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "value" entry contains the processed string.
            const value = result.resolution['value'];
            if (value) {
                const age = parseInt(value);
                if (!isNaN(age) && age >= 18 && age <= 120) {
                    output = { success: true, age: age };
                    return;
                }
            }
        });
        return output || { success: false, message: 'Please enter an age between 18 and 120.' };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120."
        };
    }
}

// Validates date input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateDate(input) {
    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "today at 9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try {
        const results = Recognizers.recognizeDateTime(input, Recognizers.Culture.English);
        const now = new Date();
        const earliest = now.getTime() + (60 * 60 * 1000);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "values" entry contains the processed input.
            result.resolution['values'].forEach(function (resolution) {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                const datevalue = resolution['value'] || resolution['start'];
                // If only time is given, assume it's for today.
                const datetime = resolution['type'] === 'time'
                    ? new Date(`${now.toLocaleDateString()} ${datevalue}`)
                    : new Date(datevalue);
                if (datetime && earliest < datetime.getTime()) {
                    output = { success: true, date: datetime.toLocaleDateString() };
                    return;
                }
            });
        });
        return output || { success: false, message: "I'm sorry, please enter a date at least an hour out." };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out."
        };
    }
}
```

---

## <a name="test-the-bot-locally"></a>로컬로 봇 테스트
1. 샘플을 머신에서 로컬로 실행합니다. 지침이 필요한 경우 [C#](https://aka.ms/cs-primitive-prompt-sample) 또는 [JS](https://aka.ms/js-primitive-prompt-sample) 샘플에 대한 README 파일을 참조하세요.
1. 아래와 같이 에뮬레이터를 사용하여 테스트합니다.

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>추가 리소스

[대화 라이브러리](bot-builder-concept-dialog.md)는 대화 관리의 다양한 측면을 자동화하는 클래스를 제공합니다. 

## <a name="next-step"></a>다음 단계

> [!div class="nextstepaction"]
> [순차적 대화 흐름 구현](bot-builder-dialog-manage-conversation-flow.md)
