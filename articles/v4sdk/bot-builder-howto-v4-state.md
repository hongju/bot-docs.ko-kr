---
title: 사용자 및 대화 데이터 저장 | Microsoft Docs
description: Bot Framework SDK를 사용하여 상태 데이터를 저장 및 검색하는 방법을 알아봅니다.
keywords: 대화 상태, 사용자 상태, 대화, 상태 저장, 봇 상태 관리
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/26/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4cafa3516395fb8e44d2755d0fa09e7a5bd6203c
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711947"
---
# <a name="save-user-and-conversation-data"></a>사용자 및 대화 데이터 저장

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇은 기본적으로 상태 비저장입니다. 일단 봇이 배포되면 동일한 프로세스 또는 동일한 머신에서 한 턴에서 다음 턴으로 실행되지 않을 수 있습니다. 그러나 봇은 대화의 컨텍스트를 추적하여 해당 동작을 관리하고 이전 질문에 대한 답변을 기억해야 할 수도 있습니다. SDK의 상태 및 스토리지 기능을 사용하면 상태를 봇에 추가할 수 있습니다.

## <a name="prerequisites"></a>필수 조건

- [봇 기본 사항](bot-builder-basics.md) 및 봇에서 [상태를 관리하는 방법](bot-builder-concept-state.md)에 대한 지식이 필요합니다.
- 이 문서의 코드는 **StateBot** 샘플을 기반으로 합니다. [C#](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) 또는 [JS]()로 작성된 샘플의 복사본이 필요합니다.
- 봇을 로컬로 테스트하기 위해 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)가 필요합니다.

## <a name="about-the-sample-code"></a>샘플 코드 정보

이 문서에서는 봇의 상태를 관리하는 구성 측면에 대해 설명합니다. 상태를 추가하기 위해 상태 속성, 상태 관리 및 스토리지를 구성한 다음, 봇에서 이러한 항목을 사용합니다.

- 각 _상태 속성_에는 봇에 대한 상태 정보가 포함됩니다.
- 각 상태 속성 접근자를 사용하면 연결된 상태 속성의 값을 가져오거나 설정할 수 있습니다.
- 각 상태 관리 개체는 스토리지에 연결된 상태 정보의 읽기 및 쓰기를 자동화합니다.
- 스토리지 계층은 메모리 내 스토리지(테스트용) 또는 Azure Cosmos DB 스토리지(프로덕션용)와 같이 상태에 대한 지원 스토리지에 연결됩니다.

봇은 활동을 처리할 때 런타임에 상태를 가져오고 설정할 수 있는 상태 속성 접근자를 사용하여 구성해야 합니다. 상태 속성 접근자는 상태 관리 개체를 사용하여 만들어지고, 상태 관리 개체는 스토리지 계층을 사용하여 만들어집니다. 따라서 스토리지 수준에서 작업을 시작해 보겠습니다.

## <a name="configure-storage"></a>저장소 구성

이 봇을 배포할 계획이 없으므로 _메모리 스토리지_를 사용합니다. 다음 단계에서 이 스토리지를 사용하여 사용자와 대화 상태를 모두 구성합니다.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**에서 스토리지 계층을 구성합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    IStorage storage = new MemoryStorage();
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js** 파일에서 스토리지 계층을 구성합니다.

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();
```

---

## <a name="create-state-management-objects"></a>상태 관리 개체 만들기

_사용자_ 및 _대화_ 상태를 모두 추적하고, 다음 단계에서 이러한 상태를 사용하여 _상태 속성 접근자_를 만듭니다.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**에서 상태 관리 개체를 만들 때 스토리지 계층을 참조합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    ConversationState conversationState = new ConversationState(storage);
    UserState userState = new UserState(storage);
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js** 파일에서 require 문에 `UserState`를 추가합니다.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

그런 다음, 대화 및 사용자 상태 관리 개체를 만들 때 스토리지 계층을 참조합니다.

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
```

---

## <a name="create-state-property-accessors"></a>상태 속성 접근자 만들기

상태 속성을 _선언_하기 위해 먼저 상태 관리 개체 중 하나를 사용하여 상태 속성 접근자를 만듭니다. 다음 정보를 추적하도록 봇을 구성합니다.

- 사용자 상태에서 정의할 사용자의 이름
- 사용자에게 방금 보낸 메시지에 대한 이름 및 몇 가지 추가 정보를 요청했는지의 여부

봇에서 접근자를 사용하여 턴 컨텍스트에서 상태 속성을 가져옵니다.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

먼저 각 유형의 상태에서 관리하려는 모든 정보를 포함하는 클래스를 정의합니다.

- 봇에서 수집할 사용자 정보에 대한 `UserProfile` 클래스입니다.
- 메시지가 도착한 시기와 메시지를 보낸 사람에 대한 정보를 추적하는 `ConversationData` 클래스입니다.

```csharp
// Defines a state property used to track information about the user.
public class UserProfile
{
    public string Name { get; set; }
}
```

```csharp
// Defines a state property used to track conversation data.
public class ConversationData
{
    // The time-stamp of the most recent incoming message.
    public string Timestamp { get; set; }

    // The ID of the user's channel.
    public string ChannelId { get; set; }

    // Track whether we have already asked the user's name
    public bool PromptedUserForName { get; set; } = false;
}
```

다음으로, 봇 인스턴스를 구성하는 데 필요한 상태 관리 정보를 포함하는 클래스를 정의합니다.

```csharp
public class StateBotAccessors
{
    public StateBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }
  
    public static string UserProfileName { get; } = "UserProfile";

    public static string ConversationDataName { get; } = "ConversationData";

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public IStatePropertyAccessor<ConversationData> ConversationDataAccessor { get; set; }
  
    public ConversationState ConversationState { get; }
  
    public UserState UserState { get; }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

상태 관리 개체를 봇의 생성자에 직접 전달하고, 봇에서 상태 속성 접근자를 자체적으로 만들게 합니다.

**index.js**에서 봇을 만들 때 상태 관리 개체를 제공합니다.

```javascript
// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

**bot.js**에서 상태를 관리하고 추적하는 데 필요한 식별자를 정의합니다.

```javascript
// The accessor names for the conversation data and user profile state property accessors.
const CONVERSATION_DATA_PROPERTY = 'conversationData';
const USER_PROFILE_PROPERTY = 'userProfile';
```

---

## <a name="configure-your-bot"></a>봇 구성

이제 상태 속성 접근자를 정의하고 봇을 구성할 준비가 되었습니다.
대화 상태 관리 개체는 대화 흐름 상태 속성 접근자에 사용하며,
사용자 상태 관리 개체는 사용자 프로필 상태 속성 접근자에 사용합니다.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**에서 번들로 묶은 상태 속성과 및 관리 개체를 제공하도록 ASP.NET을 구성합니다. 이는 ASP.NET Core의 종속성 주입 프레임워크를 통해 봇의 생성자에서 검색됩니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddSingleton<StateBotAccessors>(sp =>
    {
        // Create the custom state accessor.
        return new StateBotAccessors(conversationState, userState)
        {
            ConversationDataAccessor = conversationState.CreateProperty<ConversationData>(StateBotAccessors.ConversationDataName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(StateBotAccessors.UserProfileName),
        };
    });
}
```

ASP.NET에서 봇을 만들 때 봇의 생성자에서 `StateBotAccessors` 개체가 제공됩니다.

```csharp
// Defines a bot for filling a user profile.
public class StateBot : IBot
{
    private readonly StateBotAccessors _accessors;

    public StateBot(StateBotAccessors accessors, ILoggerFactory loggerFactory)
    {
        // ...
        _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));
    }

    // The bot's turn handler and other supporting code...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bot.js** 파일에 있는 봇의 생성자에서 상태 속성 접근자를 만들고 이를 봇에 추가합니다. 또한 상태 변경 내용을 저장하는 데 필요한 경우 상태 관리 개체에 대한 참조도 추가합니다.

```javascript
constructor(conversationState, userState) {
    // Create the state property accessors for the conversation data and user profile.
    this.conversationData = conversationState.createProperty(CONVERSATION_DATA_PROPERTY);
    this.userProfile = userState.createProperty(USER_PROFILE_PROPERTY);

    // The state management objects for the conversation and user state.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

---

## <a name="access-state-from-your-bot"></a>봇에서 상태 액세스

앞의 섹션에서는 상태 속성 접근자를 봇에 추가하는 초기화 시간 단계에 대해 다루고 있습니다.
이제 런타임에 이러한 접근자를 사용하여 상태 정보를 읽고 쓸 수 있습니다.

1. 상태 속성을 사용하기 전에 각 접근자를 사용하여 스토리지에서 속성을 로드하고 상태 캐시에서 이를 가져옵니다.
   - 접근자를 통해 상태 속성을 가져올 때마다 기본값을 제공해야 합니다. 그렇지 않으면 null 값 오류가 발생할 수 있습니다.
1. 턴 처리기를 종료하기 전에 다음을 수행합니다.
   1. 접근자의 _set_ 메서드를 사용하여 변경 내용을 봇 상태로 푸시합니다.
   1. 상태 관리 개체의 _변경 내용 저장_ 메서드를 사용하여 이러한 변경 내용을 스토리지에 씁니다.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The bot's turn handler.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        UserProfile userProfile =
            await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());
        ConversationData conversationData =
            await _accessors.ConversationDataAccessor.GetAsync(turnContext, () => new ConversationData());

        if (string.IsNullOrEmpty(userProfile.Name))
        {
            // First time around this is set to false, so we will prompt user for name.
            if (conversationData.PromptedUserForName)
            {
                // Set the name to what the user provided
                userProfile.Name = turnContext.Activity.Text?.Trim();

                // Acknowledge that we got their name.
                await turnContext.SendActivityAsync($"Thanks {userProfile.Name}.");

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.PromptedUserForName = false;
            }
            else
            {
                // Prompt the user for their name.
                await turnContext.SendActivityAsync($"What is your name?");

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.PromptedUserForName = true;
            }

            // Save user state and save changes.
            await _accessors.UserProfileAccessor.SetAsync(turnContext, userProfile);
            await _accessors.UserState.SaveChangesAsync(turnContext);
        }
        else
        {
            // Add message details to the conversation data.
            conversationData.Timestamp = turnContext.Activity.Timestamp.ToString();
            conversationData.ChannelId = turnContext.Activity.ChannelId.ToString();

            // Display state data
            await turnContext.SendActivityAsync($"{userProfile.Name} sent: {turnContext.Activity.Text}");
            await turnContext.SendActivityAsync($"Message received at: {conversationData.Timestamp}");
            await turnContext.SendActivityAsync($"Message received from: {conversationData.ChannelId}");
        }

        // Update conversation state and save changes.
        await _accessors.ConversationDataAccessor.SetAsync(turnContext, conversationData);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const userProfile = await this.userProfile.get(turnContext, {});
        const conversationData = await this.conversationData.get(
            turnContext, { promptedForUserName: false });

        if (!userProfile.name) {
            // First time around this is undefined, so we will prompt user for name.
            if (conversationData.promptedForUserName) {
                // Set the name to what the user provided.
                userProfile.name = turnContext.activity.text;

                // Acknowledge that we got their name.
                await turnContext.sendActivity(`Thanks ${userProfile.name}.`);

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.promptedForUserName = false;
            } else {
                // Prompt the user for their name.
                await turnContext.sendActivity('What is your name?');

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.promptedForUserName = true;
            }
            // Save user state and save changes.
            await this.userProfile.set(turnContext, userProfile);
            await this.userState.saveChanges(turnContext);
        } else {
            // Add message details to the conversation data.
            conversationData.timestamp = turnContext.activity.timestamp.toLocaleString();
            conversationData.channelId = turnContext.activity.channelId;

            // Display state data.
            await turnContext.sendActivity(`${userProfile.name} sent: ${turnContext.activity.text}`);
            await turnContext.sendActivity(`Message received at: ${conversationData.timestamp}`);
            await turnContext.sendActivity(`Message received from: ${conversationData.channelId}`);
        }
        // Update conversation state and save changes.
        await this.conversationData.set(turnContext, conversationData);
        await this.conversationState.saveChanges(turnContext);
    }
}
```

---

## <a name="test-the-bot"></a>봇 테스트

1. 샘플을 머신에서 로컬로 실행합니다. 지침이 필요한 경우 [C#](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) 또는 [JS](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/js/stateBot) 샘플에 대한 README 파일을 참조하세요.
1. 아래와 같이 에뮬레이터를 사용하여 봇을 테스트합니다.

![상태 봇 샘플 테스트](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>추가 리소스

**개인 정보:** 사용자의 개인 데이터를 저장하려면 [GDPR(일반 데이터 보호 규정)](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr)을 준수해야 합니다.

**상태 관리:** 상태 관리 호출은 모두 비동기적이며, 기본적으로 최종 작성자 인정(last-writer-wins)입니다. 실제로 봇의 상태를 최대한 가깝게 가져오고, 설정하고, 저장해야 합니다.

**중요 비즈니스 데이터:** 봇 상태를 사용하여 기본 설정, 사용자 이름 또는 요청한 마지막 항목을 저장하지만, 중요 비즈니스 데이터를 저장하는 데는 사용하지 않습니다. 중요한 데이터의 경우 [사용자 고유의 스토리지 구성 요소를 만들거나](bot-builder-custom-storage.md) [스토리지](bot-builder-howto-v4-storage.md)에 직접 씁니다.

**Recognizer-Text:** 샘플에서는 Microsoft/Recognizers-Text 라이브러리를 사용하여 사용자 입력을 구문 분석하고 유효성을 검사합니다. 자세한 내용은 [개요](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview) 페이지를 참조하세요.

## <a name="next-step"></a>다음 단계

이제 스토리지에서 봇 데이터를 읽고 쓰는 데 도움이 되는 상태를 구성하는 방법을 알아보았으므로, 사용자에게 일련의 질문을 하고 답변을 확인하고 입력을 저장하는 방법을 알아보겠습니다.

> [!div class="nextstepaction"]
> [사용자 입력을 수집하도록 고유한 메시지 만들기](bot-builder-primitive-prompts.md).
