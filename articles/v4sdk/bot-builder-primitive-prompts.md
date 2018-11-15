---
title: 사용자 입력을 수집하도록 고유한 메시지 만들기 | Microsoft Docs
description: 봇 작성기 SDK에서 기본 프롬프트를 사용하여 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 대화 흐름, 프롬프트
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bc223008778f0396b9bc7ff0c2ef48eb3773a105
ms.sourcegitcommit: 0702305523f8c816b2eb95dce2ea9effb9e5ee5a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/12/2018
ms.locfileid: "51562096"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>사용자 입력을 수집하도록 고유한 메시지 만들기

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

**index.js**에서 `UserState`를 포함하도록 require 문을 업데이트합니다.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

그런 다음, 사용자 상태 관리 개체를 만들고, 봇을 만들 때 이를 전달합니다.

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

**bot.js**에서 봇의 [상태](bot-builder-howto-v4-state.md)를 관리하는 데 사용할 상태 속성 접근자에 대한 식별자를 정의합니다. 또한 사용자로부터 수집하려는 정보에 사용할 프롬프트를 정의합니다.

이 코드를 `MyBot` 클래스 외부에 추가합니다.

```javascript
// Define identifiers for our state property accessors.
const TOPIC_STATE_PROPERTY = 'topicStateProperty';
const USER_PROFILE_PROPERTY = 'userProfileProperty';

// Define the prompts to use to ask for user profile information.
const fields = {
    userName: "What is your name?",
    age: "How old are you?",
    workPlace: "Where do you work?"
}
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

**bot.js**에서 `MyBot` 클래스 정의를 업데이트합니다.

봇의 생성자에서 상태 속성 접근자(`topicStateAccessor` 및 `userProfileAccessor`)를 설정했습니다. 항목 상태는 대화 항목을 추적하고, 사용자 프로필은 사용자에 대해 수집한 정보를 추적합니다.

```javascript
constructor(conversationState, userState) {
    // Create state property accessors.
    this.topicStateAccessor = conversationState.createProperty(TOPIC_STATE_PROPERTY);
    this.userProfileAccessor = userState.createProperty(USER_PROFILE_PROPERTY);

    // Track the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

그런 다음, 봇 상태를 사용하여 대화 흐름을 제어하고 수집 된 사용자 정보를 저장하도록 턴 처리기를 업데이트합니다.

```javascript
async onTurn(turnContext) {
    // Handle only message activities from the user.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get state properties using their accessors, providing default values.
        const topicState = await this.topicStateAccessor.get(turnContext, {
            prompt: undefined,
            topic: 'profile'
        });
        const userProfile = await this.userProfileAccessor.get(turnContext, {
            "userName": undefined,
            "age": undefined,
            "workPlace": undefined
        });

        if (topicState.topic === 'profile') {
            // If a prompt flag is set in the conversation state, use it to capture the incoming value
            // into the appropriate field of the user profile.
            if (topicState.prompt) {
                userProfile[topicState.prompt] = turnContext.activity.text;
            }

            // Determine which fields are not yet set.
            const empty_fields = [];
            Object.keys(fields).forEach(function (key) {
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
                    await turnContext.sendActivity('Welcome new user, please fill out your profile information.');
                }

                // We have at least one empty field. Prompt for the next empty field.
                await turnContext.sendActivity(empty_fields[0].prompt);

                // update the flag to indicate which prompt we just sent
                // so that the response can be captured at the beginning of the next turn.
                topicState.prompt = empty_fields[0].key;

            } else {
                // Our user profile is complete!
                await turnContext.sendActivity('Thank you. Your profile is complete.');
                topicState.prompt = null;
                topicState.topic = null;

            }
        } else if (turnContext.activity.text && turnContext.activity.text.match(/hi/ig)) {
            // Check to see if the user said "hi" and respond with a greeting
            await turnContext.sendActivity(`Hi ${userProfile.userName}.`);
        } else {
            // Default message
            await turnContext.sendActivity("Hi. I'm the Contoso bot.");
        }

        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
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

숫자 또는 날짜/시간에 대한 구문 분석은 이 주제의 범위를 벗어나는 복잡한 작업입니다. 다행히도 활용할 수 있는 라이브러리가 있습니다. 이 정보를 구문 분석하기 위해 [Microsoft의 텍스트 인식기](https://github.com/Microsoft/Recognizers-Text) 라이브러리를 사용합니다. 이 패키지는 NuGet과 npm을 통해 사용할 수 있습니다. 또한 리포지토리에서 직접 다운로드할 수도 있습니다. (**다이얼로그** 라이브러리에도 포함되어 있습니다. 여기서는 사용하지 않지만 주목할 만한 가치가 있습니다.)

이 라이브러리는 날짜, 시간 또는 전화 번호와 같은 복잡한 입력을 구문 분석하는 데 특히 유용합니다. 이 샘플에서는 저녁 식사 예약 파티 크기에 대한 숫자의 유효성을 검사하지만 더 복잡한 유효성 검사 작업을 위해 동일한 개념을 확장할 수 있습니다.

다음 샘플에서는 `RecognizeNumber`만 사용합니다. 라이브러리의 다른 인식기 메서드를 사용하는 방법에 대한 자세한 내용은 해당 [리포지토리의 설명서](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md)에서 찾을 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Microsoft.Recognizers.Text.Number** 라이브러리를 사용하려면 NuGet 패키지를 포함하고 다음 using 문을 봇 파일에 추가합니다.

```csharp
using System.Linq;
using Microsoft.Recognizers.Text;
using Microsoft.Recognizers.Text.Number;
```

유효성 검사는 여러 가지 방법으로 처리할 수 있습니다. 여기서는 유효성 검사를 포함하도록 도우미 클래스를 업데이트합니다.

봇의 내부 `UserFieldInfo` 클래스에 다음 멤버를 추가합니다.

```csharp
/// <summary>Delegate for validating input.</summary>
/// <param name="turnContext">The current turn context. turnContext.Activity.Text contains the input to validate.</param>
/// <returns><code>true</code> if the input is valid; otherwise, <code>false</code>.</returns>
public delegate Task<bool> ValidatorDelegate(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken));

/// <summary>By default, evaluate all input as valid.</summary>
private static readonly ValidatorDelegate NoValidator =
    async (ITurnContext turnContext, CancellationToken cancellationToken) => true;

/// <summary>Gets or sets the validation function to use.</summary>
public ValidatorDelegate ValidateInput { get; set; } = NoValidator;
```

그런 다음, 사용할 유효성 검사를 정의하도록 봇의 `UserFields`에 있는 _age_ 항목을 업데이트합니다.
age 값을 설정하기 전에 입력의 유효성을 검사하므로 `SetValue` 함수를 간소화하고 텍스트 인식기 라이브러리를 활용할 수 있습니다.

```csharp
private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
{
    // ...
    new UserFieldInfo {
        Key = nameof(UserProfile.Age),
        Prompt = "How old are you?",
        GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
        SetValue = (profile, value) =>
        {
            // As long as the input validates, this should work.
            List<ModelResult> result = NumberRecognizer.RecognizeNumber(value, Culture.English);
            profile.Age = int.Parse(result.First().Text);
        },
        ValidateInput = async (turnContext, cancellationToken) =>
        {
            try
            {
                // Recognize the input as a number. This works for responses such as
                // "twelve" as well as "12".
                List<ModelResult> result = NumberRecognizer.RecognizeNumber(
                    turnContext.Activity.Text, Culture.English);

                // Attempt to convert the Recognizer result to an integer
                int.TryParse(result.First().Text, out int age);

                if (age < 18)
                {
                    await turnContext.SendActivityAsync(
                        "You must be 18 or older.",
                        cancellationToken: cancellationToken);
                    return false;
                }
                else if (age > 120)
                {
                    await turnContext.SendActivityAsync(
                        "You must be 120 or younger.",
                        cancellationToken: cancellationToken);
                    return false;
                }
            }
            catch
            {
                await turnContext.SendActivityAsync(
                    "I couldn't understand your input. Please enter your age in years.",
                    cancellationToken: cancellationToken);
                return false;
            }

            // If we got through this, the number is valid.
            return true;
        },
    },
    // ...
};
```

마지막으로, 속성에 값을 저장하기 전에 모든 입력의 유효성을 검사하도록 턴 처리기를 업데이트합니다.
유효성 검사는 기본적으로 모든 입력을 허용하는 NoValidator 함수를 사용합니다. 따라서 age 프롬프트에 대한 동작만 변경해야 합니다. 입력 유효성 검사에 실패하면 필드를 설정하지 않고 봇은 다음 턴에서 이 필드에 대한 입력을 다시 요청합니다.

여기서는 업데이트해야 할 턴 처리기 부분만 살펴봅니다.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type is ActivityTypes.Message)
    {
        // ...
        // Check whether we need more information.
        if (topicState.Topic is ProfileTopic)
        {
            // If we're expecting input, record it in the user's profile.
            if (topicState.Prompt != null)
            {
                UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                if (await field.ValidateInput(turnContext, cancellationToken))
                {
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }
            }

            // ...
        }
        //...
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Recognizers** 라이브러리를 사용하려면 패키지를 추가하고 **bot.js**의 봇 코드에서 이 패키지를 요청합니다.

```bash
npm i @microsoft/recognizers-text-suite --save
```

```javascript
// Required packages for this bot.
const Recognizers = require('@microsoft/recognizers-text-suite');
```

그런 다음, 텍스트 인식 및 유효성 검사 코드를 포함하도록 `fields` 메타데이터를 업데이트합니다.

```javascript
// Define the prompts to use to ask for user profile information.
const fields = {
    userName: { prompt: "What is your name?" },
    age: {
        prompt: "How old are you?",
        recognize: (turnContext) => {
            var result = Recognizers.recognizeNumber(
                turnContext.activity.text, Recognizers.Culture.English);
            return parseInt(result[0].resolution.value);
        },
        validate: async (turnContext) => {
            try {
                // Recognize the input as a number. This works for responses such as
                // "twelve" as well as "12".
                var result = Recognizers.recognizeNumber(
                    turnContext.activity.text, Recognizers.Culture.English);
                var age = parseInt(result[0].resolution.value);
                if (age < 18) {
                    await turnContext.sendActivity("You must be 18 or older.");
                    return false;
                }
                if (age > 120 ) {
                    await turnContext.sendActivity("You must be 120 or younger.");
                    return false;
                }
            } catch (_) {
                await turnContext.sendActivity(
                    "I couldn't understand your input. Please enter your age in years.");
                return false;
            }
            return true;
        }
    },
    workPlace: { prompt: "Where do you work?" }
}
```

봇의 턴 처리기에서 다음 블록을 업데이트합니다. 여기서는 사용자 입력을 기록하고 사용자에게 메시지를 표시합니다. 필드 메타데이터의 변경을 고려하여 이러한 섹션을 업데이트해야 합니다.

```javascript
async onTurn(turnContext) {
    // Handle only message activities from the user.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // ...

        if (topicState.topic === 'profile') {
            // If a prompt flag is set in the conversation state, use it to capture the incoming value
            // into the appropriate field of the user profile.
            if (topicState.prompt) {
                const field = fields[topicState.prompt];
                // If the prompt has validation, check whether the input validates.
                if (!field.validate || await field.validate(turnContext)) {
                    // Set the field, using a recognizer if one is defined.
                    userProfile[topicState.prompt] = (field.recognize)
                        ? field.recognize(turnContext)
                        : turnContext.activity.text;
                }
            }

            // ...

            if (empty_fields.length) {

                // ...

                // We have at least one empty field. Prompt for the next empty field.
                await turnContext.sendActivity(empty_fields[0].prompt.prompt);

                // ...

            } // ...
        } // ...

        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

## <a name="next-step"></a>다음 단계

이제 프롬프트 상태를 직접 관리하는 방법을 이해했으므로 **다이얼로그** 라이브러리를 활용하여 사용자 입력을 요청하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [다이얼로그를 사용하여 사용자 입력 요청](bot-builder-prompts.md)

