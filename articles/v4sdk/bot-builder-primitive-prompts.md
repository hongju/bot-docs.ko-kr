---
title: 기본 프롬프트를 사용하여 대화 흐름 관리 | Microsoft Docs
description: 봇 작성기 SDK에서 기본 프롬프트를 사용하여 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 대화 흐름, 프롬프트
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 7/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b563197c111a37cf2f0f14fef183d52f38cca66
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999167"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>사용자 고유의 프롬프트를 사용하여 사용자 입력 요청

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇과 사용자 간의 대화에는 종종 사용자에게 정보를 요청하고(프롬프트), 사용자의 응답을 구문 분석한 다음, 해당 정보에 대한 작업을 수행하는 것이 포함됩니다. [다이얼로그 라이브러리를 사용하여 사용자에게 요청하는 방법](bot-builder-prompts.md)에 관한 항목에서는 **다이얼로그** 라이브러리를 사용하여 사용자에게 입력을 요청하는 방법에 대해 자세히 설명합니다. 무엇보다도 **다이얼로그** 라이브러리는 현재 대화와 현재 질문을 추적합니다. 또한 텍스트, 숫자, 날짜 및 시간 등과 같은 다양한 유형의 정보를 요청 및 유효성을 검사하는 메서드를 제공합니다.

특정 상황에서 기본 제공 **다이얼로그**는 봇에 적합한 솔루션이 아닐 수도 있습니다. **다이얼로그**는 간단한 봇에 너무 많은 오버헤드를 추가하거나, 너무 딱딱하거나, 그렇지 않은 경우 봇에서 수행해야 작업을 달성하지 못할 수 있습니다. 이러한 경우 라이브러리를 건너뛰고 사용자 고유의 프롬프트 논리를 구현할 수 있습니다. 이 항목에서는 기본 *에코 봇*을 설정하여 사용자 고유의 프롬프트를 사용하여 대화를 관리할 수 있는 방법을 보여 줍니다.

## <a name="track-prompt-states"></a>프롬프트 상태 추적

프롬프트 기반 대화에서 현재 대화의 위치와 현재 요청 중인 질문을 추적해야 합니다. 코드에서 이는 두 개의 플래그를 관리하는 것으로 해석됩니다.

예를 들어 추적하려는 두 개의 속성을 만들 수 있습니다.

이러한 상태는 현재의 항목과 프롬프트만 계속 추적합니다. 클라우드에 배포할 때 이러한 플래그가 예상대로 작동하도록 하기 위해 [대화 상태](bot-builder-howto-v4-state.md)에 저장합니다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

상태를 추적하는 두 개의 클래스를 만듭니다. 대화형 프롬프트의 진행률을 추적하는 **TopicState** 및 사용자가 제공하는 정보를 추적하는 **UserProfile**입니다. 이후 단계에서 봇 [상태](bot-builder-howto-v4-state.md)에 이 정보를 저장할 예정입니다.

```csharp
/// <summary>
/// Contains conversation state information about the conversation in progress.
/// </summary>
public class TopicState
{
    /// <summary>
    /// Identifies the current "topic" of conversation.
    /// </summary>
    public string Topic { get; set; }

    /// <summary>
    /// Indicates whether we asked the user a question last turn, and
    /// if so, what it was.
    /// </summary>
    public string Prompt { get; set; }
}
```

```csharp
/// <summary>
/// Contains user state information for the user's profile.
/// </summary>
public class UserProfile
{
    public string UserName { get; set; }

    public int? Age { get; set; }

    public string WorkPlace { get; set; }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
const storage = new MemoryStorage(); // Volatile memory
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);
const dialogState = conversationState.createProperty('dialogState');
const userProfile = userState.createProperty('userProfile');
```

이 코드는 기본 봇 논리 내에 배치합니다.

```javascript
// Pull the state of the dialog out of the conversation state manager.
const convo = await dialogState.get(context, {
    prompt: undefined,
    topic: 'profile'
});

// Pull the user profile out of the user state manager.
const userProfile = await userProfile.get(context, {  // Define the user's profile object
        "userName": undefined,
        "age": undefined,
        "workPlace": undefined
    }
);
```

---

## <a name="manage-a-topic-of-conversation"></a>대화 항목 관리

사용자로부터 정보를 수집하는 것과 같은 순차적 대화에서는 사용자가 대화에 언제 들어 왔는지, 그리고 대화에서 사용자가 어디에 있는지를 알아야 합니다. 위에서 정의한 프롬프트 플래그를 설정하고 확인한 다음, 그에 따라 작동함으로써 기본 봇의 설정 처리기에서 이를 추적할 수 있습니다. 아래 샘플에서는 이러한 플래그를 사용하여 대화를 통해 사용자의 프로필 정보를 수집하는 방법을 보여 줍니다.

봇 코드가 여기에 표시되고, 다음 섹션에서 설명됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

ASP.NET Core의 경우 봇 및 종속성 주입을 먼저 설정해야 합니다.

파일 **EchoWithCounterBot.cs**의 이름을 **PrimitivePromptsBot.cs**로 바꾸고, 클래스 이름을 업데이트합니다. 이 클래스는 봇 논리를 포함하며, 곧 업데이트할 예정입니다.

파일 **EchoBotAccessors.cs**의 이름을 **BotAccessors.cs**로 바꾸고, 클래스 이름을 업데이트합니다. 이 클래스는 상태 관리 개체 및 봇에 대한 상태 속성 접근자를 포함합니다. 다음으로 정의를 업데이트합니다.

```csharp
using Microsoft.Bot.Builder;
using System;

/// <summary>
/// Contains the state and state property accessors for the primitive prompts bot.
/// </summary>
public class BotAccessors
{
    public const string TopicStateName = "PrimitivePrompts.TopicStateAccessor";

    public const string UserProfileName = "PrimitivePrompts.UserProfileAccessor";

    public ConversationState ConversationState { get; }

    public UserState UserState { get; }

    public IStatePropertyAccessor<TopicState> TopicStateAccessor { get; set; }

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        if (conversationState is null)
        {
            throw new ArgumentNullException(nameof(conversationState));
        }

        if (userState is null)
        {
            throw new ArgumentNullException(nameof(userState));
        }

        this.ConversationState = conversationState;
        this.UserState = userState;
    }
}
```

`IStorage` 개체를 설정한 위치에서 시작하여 **Startup.cs** 파일의 `ConfigureServices` 메서드를 업데이트합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<PrimitivePromptsBot>(options =>
    {
        // ...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        var conversationState = new ConversationState(dataStore);
        options.State.Add(conversationState);

        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            TopicStateAccessor = conversationState.CreateProperty<TopicState>(BotAccessors.TopicStateName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(BotAccessors.UserProfileName),
        };

        return accessors;
    });
}
```

마지막으로 **PrimitivePromptsBot.cs** 파일에서 봇 논리를 업데이트합니다.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

public class PrimitivePromptsBot : IBot
{
    public const string ProfileTopic = "profile";

    /// <summary>
    /// Describes a field in the user profile.
    /// </summary>
    private class UserFieldInfo
    {
        /// <summary>
        /// The ID to use for this field.
        /// </summary>
        public string Key { get; set; }

        /// <summary>
        /// The prompt to use to ask for a value for this field.
        /// </summary>
        public string Prompt { get; set; }

        /// <summary>
        /// Gets the value of the corresponding field.
        /// </summary>
        public Func<UserProfile, string> GetValue { get; set; }

        /// <summary>
        /// Sets the value of the corresponding field.
        /// </summary>
        public Action<UserProfile, string> SetValue { get; set; }
    }

    /// <summary>
    /// The prompts for the user profile, indexed by field name.
    /// </summary>
    private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
    {
        new UserFieldInfo {
            Key = nameof(UserProfile.UserName),
            Prompt = "What is your name?",
            GetValue = (profile) => profile.UserName,
            SetValue = (profile, value) => profile.UserName = value,
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.Age),
            Prompt = "How old are you?",
            GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
            SetValue = (profile, value) =>
            {
                if (int.TryParse(value, out int age))
                {
                    profile.Age = age;
                }
            },
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.WorkPlace),
            Prompt = "Where do you work?",
            GetValue = (profile) => profile.WorkPlace,
            SetValue = (profile, value) => profile.WorkPlace = value,
        },
    };

    /// <summary>
    /// The state and state accessors for the bot.
    /// </summary>
    private BotAccessors Accessors { get; }

    public PrimitivePromptsBot(BotAccessors accessors)
    {
        Accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));
    }

    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type is ActivityTypes.Message)
        {
            // Use the state property accessors to get the topic state and user profile.
            TopicState topicState = await Accessors.TopicStateAccessor.GetAsync(
                turnContext,
                () => new TopicState { Topic = ProfileTopic, Prompt = null },
                cancellationToken);
            UserProfile userProfile = await Accessors.UserProfileAccessor.GetAsync(
                turnContext,
                () => new UserProfile(),
                cancellationToken);

            // Check whether we need more information.
            if (topicState.Topic is ProfileTopic)
            {
                // If we're expecting input, record it in the user's profile.
                if (topicState.Prompt != null)
                {
                    UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }

                // Determine which fields are not yet set.
                List<UserFieldInfo> emptyFields = UserFields.Where(f => f.GetValue(userProfile) is null).ToList();

                if (emptyFields.Any())
                {
                    // If all the fields are empty, send a welcome message.
                    if (emptyFields.Count == UserFields.Count)
                    {
                        await turnContext.SendActivityAsync("Welcome new user, please fill out your profile information.");
                    }

                    // We have at least one empty field. Prompt for the next empty field,
                    // and update the prompt flag to indicate which prompt we just sent,
                    // so that the response can be captured at the beginning of the next turn.
                    UserFieldInfo field = emptyFields.First();
                    await turnContext.SendActivityAsync(field.Prompt);
                    topicState.Prompt = field.Key;
                }
                else
                {
                    // Our user profile is complete!
                    await turnContext.SendActivityAsync($"Thank you, {userProfile.UserName}. Your profile is complete.");
                    topicState.Prompt = null;
                    topicState.Topic = null;
                }
            }
            else if (turnContext.Activity.Text.Trim().Equals("hi", StringComparison.InvariantCultureIgnoreCase))
            {
                await turnContext.SendActivityAsync($"Hi. {userProfile.UserName}.");
            }
            else
            {
                await turnContext.SendActivityAsync("Hi. I'm the Contoso cafe bot.");
            }

            // Use the state property accessors to update the topic state and user profile.
            await Accessors.TopicStateAccessor.SetAsync(turnContext, topicState, cancellationToken);
            await Accessors.UserProfileAccessor.SetAsync(turnContext, userProfile, cancellationToken);

            // Save any state changes to storage.
            await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript

server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === ActivityTypes.Message);

        // Set up a list of fields that we need to collect from the user.
        const fields = {
            userName: "What is your name?",
            age: "How old are you?",
            workPlace: "Where do you work?"
        }

        // Pull the state of the dialog out of the conversation state manager.
        const convo = await dialogState.get(context, {
            topic: 'profile',
            prompt: undefined
        });

        // Pull the user profile out of the user state manager.
        const userProfile = await userProfile.get(context, {  // Define the user's profile object
                "userName": undefined,
                "age": undefined,
                "workPlace": undefined
            }
        );

        if (isMessage) {

            if (convo.topic === 'profile') {
                // If a prompt flag is set in the conversation state, use it to capture the incoming value
                // into the appropriate field of the user profile.
                if (convo.prompt) {
                    userProfile[convo.prompt] = context.activity.text;
                }

                 // Determine which fields are not yet set.
                 const empty_fields = [];
                 Object.keys(fields).forEach(function(key) {
                    if (!userProfile[key]) {
                        empty_fields.push({
                           key: key,
                           prompt: fields[key]
                        });
                     }
                 });

                 if (empty_fields.length) {

                    // If all the fields are empty, send a welcome message.
                    if (empty_fields.length == Object.keys(fields).length) {
                        await context.sendActivity('Welcome new user, please fill out your profile information.');
                    }

                    // We have at least one empty field. Prompt for the next empty field.
                    await context.sendActivity(empty_fields[0].prompt);

                    // update the flag to indicate which prompt we just sent
                    // so that the response can be captured at the beginning of the next turn.
                    convo.prompt = empty_fields[0].key;

                 } else {
                    // Our user profile is complete!
                    await context.sendActivity('Thank you. Your profile is complete.');
                    convo.prompt = null;
                    convo.topic = null;

                 }
            } else if (context.activity.text && context.activity.text.match(/hi/ig)) {
                // Check to see if the user said "hi" and respond with a greeting
                await context.sendActivity(`Hi ${ userProfile.userName }.`);
            } else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

        // End the turn by writing state changes back to storage
        await conversationState.saveChanges(context);
        await userState.saveChanges(context);
    });
});

```

---

위의 샘플 코드는 `profile` 프로필 컬렉션 대화를 시작하기 위해 _토픽_ 플래그를 초기화합니다. 봇은 _프롬프트_ 플래그를 현재 질문을 나타내는 값으로 업데이트하여 대화를 통해 앞으로 이동합니다. 적절한 값으로 설정된 이 플래그를 사용하여 봇은 사용자로부터 받은 다음 메시지를 사용하여 수행할 작업을 알고 적절하게 처리합니다.

마지막으로 봇이 정보 요청을 수행하면 플래그가 다시 설정됩니다. 그렇지 않으면 봇은 최종 질문에서 이동하지 못하고 루프에 갇히게 됩니다.

이 패턴을 확장하여 사용자 입력에 따라 다른 플래그를 정의하거나 대화를 분기하여 더 복잡한 대화 흐름을 관리할 수 있습니다.

## <a name="manage-multiple-topics-of-conversations"></a>여러 대화 항목 관리

하나의 대화 항목을 처리할 수 있으면 봇의 논리를 확장하여 여러 대화 항목을 처리할 수 있습니다. 추가 조건을 확인한 다음, 적절한 경로를 수행하여 여러 항목을 처리할 수 있습니다.

위의 예제를 확장하여 테이블 예약 또는 주문 등의 다른 기능 및 대화 항목을 허용할 수 있습니다. 대화를 추적하기 위해 필요에 따라 추가 플래그를 항목 상태에 추가합니다.

또한 코드를 독립적인 함수 또는 메서드로 분할하면 대화 흐름을 더 쉽게 수행할 수 있습니다. 일반적인 패턴은 봇에서 메시지 및 상태를 평가하게 한 다음, 기능의 세부 정보를 구현하는 함수에 대한 제어를 위임하는 것입니다.

사용자가 대화의 여러 항목으로 더 잘 이동하도록 도우려면 주 메뉴를 제공하는 것이 좋습니다. 예를 들어 [제안된 작업](bot-builder-howto-add-suggested-actions.md)을 사용하여 봇이 수행할 수 있는 작업을 추측하는 대신 사용자가 참여할 수 있는 항목을 선택할 수 있습니다.

## <a name="validate-user-input"></a>사용자 입력 유효성 검사

**다이얼로그** 라이브러리는 사용자 입력의 유효성을 검사하는 기본 제공 방법을 제공하지만, 고유한 프롬프트를 사용하여 유효성 검사를 수행할 수도 있습니다. 예를 들어 사용자의 연령을 요청하는 경우 응답으로 "Bob"과 같은 문자열이 아닌 숫자를 얻으려고 합니다.

숫자 또는 날짜/시간에 대한 구문 분석은 이 주제의 범위를 벗어나는 복잡한 작업입니다. 다행히도 활용할 수 있는 라이브러리가 있습니다. 이 정보를 구문 분석하기 위해 [Microsoft의 텍스트 인식기](https://github.com/Microsoft/Recognizers-Text) 라이브러리를 사용합니다. 이 패키지는 NuGet을 통해 사용하거나 리포지토리에서 다운로드할 수 있습니다. (**다이얼로그** 라이브러리에도 포함되어 있습니다. 여기서는 사용하지 않지만 주목할 만한 가치가 있습니다.)

이 라이브러리는 날짜, 시간 또는 전화 번호와 같은 복잡한 입력을 구문 분석하는 데 특히 유용합니다. 이 샘플에서는 저녁 식사 예약 파티 크기에 대한 숫자의 유효성을 검사하지만 더 복잡한 유효성 검사 작업을 위해 동일한 개념을 확장할 수 있습니다.

다음 샘플에서는 `RecognizeNumber`만 사용합니다. 라이브러리의 다른 인식기 메서드를 사용하는 방법에 대한 자세한 내용은 해당 [리포지토리의 설명서](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md)에서 찾을 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Microsoft.Recognizers.Text.Number** 라이브러리를 사용하려면 NuGet 패키지를 포함하고 using 문을 사용하여 추가합니다.

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq;
```

그런 다음, 유효성 검사를 실제로 수행하는 함수를 정의합니다.

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(value, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        int.TryParse(result.First().Text, out int partySize);

        if (partySize < 6)
        {
            throw new Exception("Party size too small.");
        }
        else if (partySize > 20)
        {
            throw new Exception("Party size too big.");
        }

        // If we got through this, the number is valid
        return true;
    }
    catch (Exception)
    {
        await context.SendActivityAsync("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**인식기** 라이브러리를 사용하려면 **app.js**에 해당 라이브러리가 필요합니다.

```javascript
// Required packages for this bot
var Recognizers = require('@microsoft/recognizers-text-suite');
```

그런 다음, 유효성 검사를 실제로 수행하는 함수를 정의합니다.

```javascript
// Support party size between 6 and 20 only
async function validatePartySize(context, input){
    try {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        var value = parseInt(results[0].resolution.value);

        if (value < 6) {
            throw new Error(`Party size too small.`);
        } else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    } catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

프롬프트에 대한 사용자의 응답을 처리하는 동안 다음 프롬프트로 이동하기 전에 유효성 검사 함수를 호출합니다. 유효성 검사가 실패하면 질문을 다시 요청합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
if (topicState.Prompt == "partySize")
{
    if (await ValidatePartySize(turnContext, turnContext.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which
        // is a new class we've added to our state
        // UserFieldInfo partySize;
        partySize.SetValue(userProfile, turnContext.Activity.Text);

        // Ask next question.
        topicState.Prompt = "reserveName";
        await turnContext.SendActivityAsync("Who's name will this be under?");
    }
    else
    {
        // Ask again.
        await turnContext.SendActivityAsync("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
// ...
if (convo.prompt == "partySize") {
    if (await validatePartySize(context, context.activity.text)) {
        // Save user's response
        reservationInfo.partySize = context.activity.text;

        // Ask next question
        convo.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    } else {
        // Ask again
        await context.sendActivity("How many people are in your party?");
    }
}
// ...
```

---

## <a name="next-step"></a>다음 단계

이제 프롬프트 상태를 직접 관리하는 방법을 이해했으므로 **다이얼로그** 라이브러리를 활용하여 사용자 입력을 요청하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [다이얼로그를 사용하여 사용자 입력 요청](bot-builder-prompts.md)
