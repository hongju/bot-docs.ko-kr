---
title: 대화 및 사용자 상태 관리 | Microsoft Docs
description: .NET용 Bot Builder SDK를 사용하여 상태 데이터를 저장 및 검색하는 방법을 알아봅니다.
keywords: 대화 상태, 사용자 상태, 대화 흐름
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 21f864ba6f5beba5205e860f4a56697997048dfb
ms.sourcegitcommit: 6c2426c43cd2212bdea1ecbbf8ed245145b3c30d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/08/2018
ms.locfileid: "48852298"
---
# <a name="manage-conversation-and-user-state"></a>대화 및 사용자 상태 관리

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

좋은 봇 디자인의 핵심은 대화의 컨텍스트를 추적하여 이전 질문에 대한 답변 같은 것을 기억하도록 하는 것입니다. 봇의 용도에 따라 상태의 트랙을 유지하거나 대화의 수명보다 오랫동안 정보를 저장해야 할 수 있습니다. 봇의 *상태*는 들어오는 메시지에 적절하게 응답하기 위해 기억하는 정보입니다. Bot Builder SDK는 사용자 또는 대화와 연결된 개체로 상태 데이터를 저장 및 검색하기 위한 두 클래스를 제공합니다.

- **대화 상태**는 봇이 사용자와 갖는 현재 대화의 트랙을 유지하는 데 도움이 됩니다. 봇에서 대화 항목 간의 단계 또는 전환의 시퀀스를 완료해야 하는 경우 대화 속성을 사용하여 시퀀스의 단계를 관리하거나 현재 항목을 추적할 수 있습니다. 

- **사용자 상태**는 사용자의 이전 대화를 중단한 위치를 확인하거나 단순히 이름별로 돌아온 사용자에게 인사말을 제공하는 등의 다양한 용도로 사용될 수 있습니다. 사용자의 기본 설정을 저장하면 이 정보를 사용하여 다음에 채팅할 때 대화를 사용자 지정할 수 있습니다. 예를 들어, 관심 있는 주제에 대한 뉴스 기사를 사용자에게 알리거나 약속이 가능해질 때 사용자에게 알릴 수 있습니다. 

`ConversationState` 및 `UserState`는 저장된 개체의 범위와 수명을 제어하는 정책이 포함된 `BotState` 클래스의 특수화된 상태 클래스입니다. 상태를 저장해야 하는 구성 요소는 속성을 만들어 형식에 등록하고 속성 접근자를 사용하여 상태에 액세스합니다. 봇 상태 관리자는 메모리 저장소, CosmosDB 및 Blob Storage를 사용할 수 있습니다. 

> [!NOTE] 
> 봇 상태 관리자를 사용하여 기본 설정, 사용자 이름 또는 주문했지만 중요한 비즈니스 데이터를 저장하는 데 사용하지 못한 마지막 항목을 저장합니다. 중요한 데이터의 경우 사용자 고유의 저장소 구성 요소를 만들거나 직접 [저장소](bot-builder-howto-v4-storage.md)에 기록합니다.
> 메모리 내 데이터 저장소는 테스트 전용입니다. 이 저장소는 휘발성이며 임시입니다. 데이터는 봇이 다시 시작될 때마다 지워집니다.

## <a name="using-conversation-state-and-user-state-to-direct-conversation-flow"></a>대화 상태 및 사용자 상태를 사용하여 대화 흐름 안내
대화 흐름을 설계할 때 대화 흐름을 안내하는 상태 플래그를 정의하면 유용합니다. 이 플래그는 간단한 부울 형식일 수도 있고 현재 토픽의 이름을 포함하는 형식일 수도 있습니다. 이 플래그는 대화에서 현재 위치를 추적하는 데 도움이 됩니다. 예를 들어 부울 형식 플래그는 대화에 참여했는지 여부를 알려줄 수 있는 반면, 토픽 이름 속성은 현재 어떤 대화에 참여하고 있는지를 알려줄 수 있습니다.



# <a name="ctabcsharp"></a>[C#](#tab/csharp)
### <a name="conversation-and-user-state"></a>대화 및 사용자 상태
[Echo Bot With Counter 샘플](https://aka.ms/EchoBot-With-Counter)을 이 방법에 대한 시작점으로 사용할 수 있습니다. 먼저 아래와 같이 `TopicState.cs`에서 현재 대화의 토픽을 관리하려면 `TopicState` 클래스를 만듭니다.

```csharp
public class TopicState
{
   public string Prompt { get; set; } = "askName";
}
``` 
그런 다음, 사용자 상태를 관리하려면 `UserProfile.cs`에서 `UserProfile` 클래스를 만듭니다.
```csharp
public class UserProfile
{
    public string UserName { get; set; }
    public string TelephoneNumber { get; set; }
}
``` 
`TopicState` 클래스에는 대화의 현재 위치를 추적하는 플래그가 있으며 대화 상태를 사용하여 이 플래그를 저장합니다. 프롬프트를 "askName"으로 초기화하여 대화를 시작합니다. 봇이 사용자로부터 응답을 수신하면 프롬프트를 "askNumber"로 재정의하여 다음 대화를 시작하게 됩니다. `UserProfile` 클래스는 사용자 이름 및 전화 번호를 추적하여 사용자 상태에 저장합니다.

### <a name="property-accessors"></a>속성 접근자
예제에서 `EchoBotAccessors` 클래스는 싱글톤으로 생성되고 종속성 주입을 통해 `class EchoWithCounterBot : IBot` 생성자로 전달됩니다. `EchoBotAccessors` 클래스 생성자는 `EchoBotAccessors` 클래스의 새 인스턴스를 초기화합니다. 또한 `ConversationState`, `UserState` 및 연결된 `IStatePropertyAccessor`을 포함합니다. `conversationState` 개체는 사용자 프로필 정보를 저장하는 `userState` 개체 및 토픽 상태를 저장합니다. `ConversationState` 및 `UserState` 개체는 Startup.cs 파일에서 만들어집니다. 대화 및 사용자 상태 개체는 대화 및 사용자 범위에서 모든 항목을 유지하는 위치입니다. 

아래에 표시된 것처럼 `UserState`를 포함하도록 생성자를 업데이트했습니다.
```csharp
using EchoBotWithCounter;

public EchoBotAccessors(ConversationState conversationState, UserState userState)
{
    ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    UserState = userState ?? throw new ArgumentNullException(nameof(userState));
}
```
`TopicState` 및 `UserProfile` 접근자에 대한 고유한 이름을 만듭니다.
```csharp
public static string UserProfileName { get; } = $"{nameof(EchoBotAccessors)}.UserProfile";
public static string TopicStateName { get; } = $"{nameof(EchoBotAccessors)}.TopicState";
```
다음으로, 두 접근자를 정의합니다. 첫 번째 접근자는 대화의 토픽을 저장하고 두 번째 접근자는 사용자 이름 및 전화 번호를 저장합니다.
```csharp
public IStatePropertyAccessor<TopicState> TopicState { get; set; }
public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }
```

ConversationState를 가져오기 위한 속성은 이미 정의되어 있지만 아래에 표시된 것처럼 `UserState`를 저장해야 합니다.
```csharp
public ConversationState ConversationState { get; }
public UserState UserState { get; }
```
변경한 후에 파일을 저장합니다. 다음으로, 사용자 범위에서 모든 항목을 유지하려면 스타트업 클래스를 업데이트하여 `UserState` 개체를 만듭니다. `ConversationState`가 이미 있습니다. 
```csharp

services.AddBot<EchoWithCounterBot>(options =>
{
    ...

    IStorage dataStore = new MemoryStorage();
    
    var conversationState = new ConversationState(dataStore);
    options.State.Add(conversationState);
        
    var userState = new UserState(dataStore);  
    options.State.Add(userState);
});
```
`options.State.Add(ConversationState);` 및 `options.State.Add(userState);` 줄은 각각 대화 상태 및 사용자 상태를 추가합니다. 다음으로, 상태 접근자를 만들고 등록합니다. 여기서 만든 접근자는 모든 전환점에서 IBot 파생 클래스로 전달됩니다. 아래에 표시된 것처럼 코드를 수정하여 사용자 상태를 포함합니다.
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var userState = options.State.OfType<UserState>().FirstOrDefault();
    if (userState == null)
    {
        throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
    }
   ...
 });
```

다음으로, `TopicState` 및 `UserProfile`을 사용하여 두 접근자를 만들어 종속성 주입을 통해 `class EchoWithCounterBot : IBot` 클래스에 전달합니다.
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var accessors = new BotAccessors(conversationState, userState)
    {
        TopicState = conversationState.CreateProperty<TopicState>("TopicState"),
        UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
     };

     return accessors;
 });
```

대화 및 사용자 상태는 `services.AddSingleton` 코드 블록을 통해 싱글톤에 연결되고 `var accessors = new BotAccessor(conversationState, userState)`로 시작하는 코드에서 상태 저장소 접근자를 통해 저장됩니다.

### <a name="use-conversation-and-user-state-properties"></a>대화 및 사용자 상태 속성 사용 
`EchoWithCounterBot : IBot` 클래스의 `OnTurnAsync` 처리기에서 코드를 수정하여 사용자 이름에 이어 전화 번호에 대한 메시지를 표시합니다. 현재 대화의 위치를 추적하려면 TopicState에서 정의된 프롬프트 속성을 사용합니다. 이 속성은 "askName"으로 초기화되었습니다. 사용자 이름을 가져온 후, "askNumber"로 설정하고 UserName을 사용자가 입력한 이름으로 설정합니다. 전화 번호를 받은 후 대화의 끝에 있으므로 확인 메시지를 보내고 프롬프트를 '확인'으로 설정합니다.

```csharp
using EchoBotWithCounter;

if (turnContext.Activity.Type == ActivityTypes.Message)
{
    // Get the conversation state from the turn context.
    var convo = await _accessors.TopicState.GetAsync(turnContext, () => new TopicState());
    
    // Get the user state from the turn context.
    var user = await _accessors.UserProfile.GetAsync(turnContext, () => new UserProfile());
    
    // Ask user name. The Prompt was initialiazed as "askName" in the TopicState.cs file.
    if (convo.Prompt == "askName")
    {
        await turnContext.SendActivityAsync("What is your name?");
        
        // Set the Prompt to ask the next question for this conversation
        convo.Prompt = "askNumber";
        
        // Set the property using the accessor
        await _accessors.TopicState.SetAsync(turnContext, convo);
        
        //Save the new prompt into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "askNumber")
    {
        // Set the UserName that is defined in the UserProfile class
        user.UserName = turnContext.Activity.Text;
        
        // Use the user name to prompt the user for phone number
        await turnContext.SendActivityAsync($"Hello, {user.UserName}. What's your telephone number?");
        
        // Set the Prompt now that we have collected all the data
        convo.Prompt = "confirmation";
                 
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfile.SetAsync(turnContext, user);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "confirmation")
    { 
        // Set the TelephoneNumber that is defined in the UserProfile class
        user.TelephoneNumber = turnContext.Activity.Text;

        await turnContext.SendActivityAsync($"Got it, {user.UserName}. I'll call you later.");

        // reset initial prompt state
        convo.Prompt = "askName"; // Reset for a new conversation.
        
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="conversation-and-user-state"></a>대화 및 사용자 상태

[Echo Bot With Counter 샘플](https://aka.ms/EchoBot-With-Counter-JS)을 이 방법에 대한 시작점으로 사용할 수 있습니다. 이 샘플은 이미 `ConversationState`를 사용하여 메시지 개수를 저장합니다. 대화 상태를 추적하려면 `TopicStates` 개체를 추가하고 `userProfile` 개체의 사용자 정보를 추적하려면 `UserState`를 추가해야 합니다. 

기본 봇의 `index.js` 파일에서 필요 목록에 `UserState`를 추가합니다.

**index.js**

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

다음으로, 저장소 공급자로 `MemoryStorage`를 사용하여 `UserState`를 만든 다음, 두 번째 인수로 `MainDialog` 클래스에 전달합니다.

**index.js**

```javascript
// Create conversation state with in-memory storage provider. 
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
// Create the main bot.
const bot = new EchBot(conversationState, userState);
```

`bot.js` 파일에서 생성자를 업데이트하여 두 번째 인수로 `userState`를 수락합니다. 그런 다음, `conversationState`에서 `topicState` 속성을 만들고 `userState`에서 `userProfile` 속성을 만듭니다.

**bot.js**

```javascript
const TOPIC_STATE = 'topic';
const USER_PROFILE = 'user';

constructor (conversationState, userState) {
    // creates a new state accessor property.see https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors 
    this.conversationState = conversationState;
    this.topicState = this.conversationState.createProperty(TOPIC_STATE);

    // User state
    this.userState = userState;
    this.userProfile = this.userState.createProperty(USER_PROFILE);
}
```

### <a name="use-conversation-and-user-state-properties"></a>대화 및 사용자 상태 속성 사용

`MainDialog` 클래스의 `onTurn` 처리기에서 코드를 수정하여 사용자 이름에 이어 전화 번호에 대한 메시지를 표시합니다. 현재 대화의 위치를 추적하려면 `topicState`에서 정의된 `prompt` 속성을 사용합니다. 이 속성은 "askName"으로 초기화됩니다. 사용자 이름을 가져온 후, "askNumber"로 설정하고 UserName을 사용자가 입력한 이름으로 설정합니다. 전화 번호를 받은 후 대화의 끝에 있으므로 확인 메시지를 보내고 프롬프트를 `undefined`으로 설정합니다.

**dialogs/mainDialog/index.js**

```javascript
// see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
if (turnContext.activity.type === 'message') {
    // read from state and set default object if object does not exist in storage.
    let topicState = await this.topicState.get(turnContext, {
        //Define the topic state object
        prompt: "askName"
    });
    let userProfile = await this.userProfile.get(turnContext, {  
        // Define the user's profile object
        "userName": "",
        "telephoneNumber": ""
    });

    if(topicState.prompt == "askName"){
        await turnContext.sendActivity("What is your name?");

        // Set next prompt state
        topicState.prompt = "askNumber";

        // Update state
        await this.topicState.set(turnContext, topicState);
    }
    else if(topicState.prompt == "askNumber"){
        // Set the UserName that is defined in the UserProfile class
        userProfile.userName = turnContext.activity.text;

        // Use the user name to prompt the user for phone number
        await turnContext.sendActivity(`Hello, ${userProfile.userName}. What's your telephone number?`);

        // Set next prompt state
        topicState.prompt = "confirmation";

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    else if(topicState.prompt == "confirmation"){
        // Set the phone number
        userProfile.telephoneNumber = turnContext.activity.text;

        // Sent confirmation
        await turnContext.sendActivity(`Got it, ${userProfile.userName}. I'll call you later.`)

        // reset initial prompt state
        topicState.prompt = "askName"; // Reset for a new conversation

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    
    // Save state changes to storage
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
    
}
else {
    await turnContext.sendActivity(`[${context.activity.type} event detected]`);
}
```

---

## <a name="start-your-bot"></a>봇 시작
봇을 로컬로 실행합니다.

### <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결
에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 "시작" 탭에서 **봇 열기** 링크를 클릭합니다. 
2. Visual Studio 솔루션을 만든 디렉터리에서 .bot 파일을 선택합니다.

### <a name="interact-with-your-bot"></a>봇과의 상호 작용

봇에 메시지를 보내면 봇이 메시지를 통해 응답합니다.
![에뮬레이터 실행](../media/emulator-v4/emulator-running.png)

상태를 직접 관리하려는 경우 [고유한 프롬프트를 사용하여 대화 흐름 관리](bot-builder-primitive-prompts.md)를 참조하세요. 대안은 폭포 대화 상자를 사용하는 것입니다. 이 대화 상자는 자동으로 대화 상태를 추적하므로 상태를 추적하는 플래그를 만들 필요가 없습니다. 자세한 내용은 [대화 상자로 간단한 대화 관리](bot-builder-dialog-manage-conversation-flow.md)를 참조하세요.

## <a name="next-steps"></a>다음 단계
상태를 사용하여 봇 데이터를 읽고 저장소에 쓰는 방법을 알아보았으니, 데이터를 읽고 저장소에 직접 쓰는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [저장소에 직접 쓰는 방법](bot-builder-howto-v4-storage.md)
