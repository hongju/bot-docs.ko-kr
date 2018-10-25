---
title: Bot Builder SDK 내 봇 활동 | Microsoft Docs
description: Bot Builder SDK 내에서 활동 및 http가 어떻게 작동하는지 설명합니다.
keywords: 대화 흐름, 순서, 봇 대화, 대화 상자, 프롬프트, 폭포, 대화 상자 집합
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/26/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fde88929c688c25d473ce8242ebfd5d44dc3a22f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998130"
---
# <a name="understanding-how-bots-work"></a>봇 작동 방식 이해

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇은 사용자가 텍스트, 그래픽(예: 카드 또는 이미지) 또는 음성을 사용하여 대화형 방식으로 상호 작용하는 앱입니다. 사용자와 봇 간에 상호 작용이 이루어질 때마다 *활동*이 생성됩니다. Bot Service는 사용자의 봇에 연결된 앱(예: Facebook, Skype, Slack 등을 말하며, *채널*이라고 함)과 봇 간에 정보를 전송합니다. 각 채널이 전송하는 활동에 추가 정보가 포함될 수 있습니다. 봇을 만들기 전에 봇이 활동 개체를 사용하여 사용자와 소통하는 방법을 이해하는 것이 중요합니다. 먼저 간단한 에코 봇을 실행할 때 주고받는 활동을 살펴보겠습니다.

![활동 다이어그램](media/bot-builder-activity.png)

여기에 설명된 두 가지 활동은 *대화 업데이트*와 *메시지*입니다.

대화 상대가 대화에 참여하면 Bot Framework Service에서 대화 업데이트를 보낼 수 있습니다. 예를 들어 Bot Framework Emulator와의 대화를 시작하면 두 가지 대화 업데이트 활동(사용자 대화 참여 활동과 봇 참여 활동)이 표시됩니다. 이러한 대화 업데이트 활동을 구분하려면 *members added* 속성에 봇 아닌 멤버가 포함되어 있는지 확인하세요. 

메시지 활동은 대화 상대 간에 대화 정보를 전달합니다. 에코 봇 예에서는 메시지 활동이 간단한 텍스트를 전달하고 채널이 이 텍스트를 렌더링합니다. 또는 메시지 활동이 말할 텍스트, 제안된 작업 또는 표시할 카드를 전달할 수 있습니다.

이 예에서는 봇이 수신한 인바운드 메시지 활동에 대한 응답으로 메시지 활동을 만들고 전송했습니다. 하지만 봇은 수신된 메시지 활동에 다른 방식으로 응답할 수 있습니다. 봇이 메시지 활동으로 환영 텍스트를 보내 대화 업데이트 활동에 응답하는 것은 일반적이지 않습니다. 자세한 내용은 [사용자 환영](bot-builder-welcome-user.md)에서 찾을 수 있습니다.

### <a name="http-details"></a>HTTP 세부 정보

활동은 HTTP POST 요청을 통해 Bot Framework Service에서 봇에 도착합니다. 봇은 HTTP 상태 코드 200으로 인바운드 POST 요청에 응답합니다. 봇에서 채널로 전송된 활동은 별도의 HTTP POST에서 Bot Framework Service로 전송됩니다. 이는 다시 말해 200 HTTP 상태 코드로 승인이 요청됩니다.

프로토콜은 이러한 POST 요청과 승인 요청이 수행되는 순서를 지정하지 않습니다. 그러나 일반적으로 이러한 요청은 일반적인 HTTP 서비스 프레임워크에 맞추기 위해 중첩됩니다. 즉, 인바운드 HTTP 요청의 범위 내에서 봇이 아웃바운드 HTTP 요청을 수행합니다. 이 패턴은 위의 다이어그램에 나와 있습니다. 두 개의 개별적인 HTTP 연결이 연달아 수행되므로 이에 대한 보안 모델이 제공되어야 합니다.

### <a name="defining-a-turn"></a>순서 정의

봇과 관련하여 활동 도착과 관련된 모든 처리를 설명하기 위해 순서라는 용어가 사용됩니다. 

*순서 컨텍스트* 개체는 발신자 및 수신자, 채널, 활동을 처리하는 데 필요한 기타 데이터와 같은 활동에 대한 정보를 제공합니다. 또한 봇의 다양한 계층에서 순서가 처리되는 동안 정보를 추가할 수 있게 해줍니다.

순서 컨텍스트는 SDK의 가장 중요한 추상화 중 하나입니다. 모든 미들웨어 구성 요소에 대한 인바운드 활동과 응용 프로그램 논리를 전달할 뿐만 아니라 미들웨어 구성 요소와 응용 프로그램 논리가 아웃바운드 활동을 전송하는 데 사용할 수 있는 메커니즘을 제공합니다.

## <a name="the-activity-processing-stack"></a>활동 처리 스택

이전 다이어그램에서 메시지 활동의 도착을 중점적으로 살펴보도록 하겠습니다.

![활동 처리 스택](media/bot-builder-activity-processing-stack.png)

위의 예제에서 봇은 동일한 텍스트 메시지를 포함하는 다른 메시지 활동을 사용하여 메시지 활동에 응답했습니다. HTTP POST 요청부터 처리되며, 활동 정보는 JSON 페이로드로 전달되고, 웹 서버에 도착합니다. C#에서 이는 일반적으로 ASP.NET 프로젝트가 되며, JavaScript Node.js 프로젝트에서 이는 Express 또는 Restify처럼 인기 있는 프레임워크 중 하나일 가능성이 높습니다.

SDK의 통합 구성 요소인 *어댑터*는 프레임워크의 생성자 역할을 합니다. 이 서비스는 활동 정보를 사용하여 활동 개체를 만든 다음, 어댑터의 *프로세스 활동* 메서드를 호출하는 한편, 활동 개체 및 인증 정보를 전달합니다(이러한 호출은 C#용 라이브러리 내부애 래핑되지만 JavaScript로 표시됨). 어댑터는 활동을 수신하자 마자 순서 컨텍스트 개체를 만들고 [미들웨어](#middleware)를 호출합니다. 미들웨어 다음에 봇 논리가 처리되고, 파이프라인이 완료되고, 어댑터가 순서 컨텍스트 개체를 삭제합니다.

응용 프로그램 논리의 대부분을 차지하는 봇의 *순서 처리기*는 순서 컨텍스트를 인수로 사용합니다. 순서 처리기는 일반적으로 인바운드 활동의 콘텐츠를 처리하고, 하나 이상의 활동을 응답으로 생성하며, 순서 컨텍스트의 *send activity* 메서드를 사용하여 이를 전송합니다. send activity 메서드를 호출하면 처리가 중단되지 않은 경우에 한해 활동이 사용자의 채널로 전송됩니다. 이 활동은 등록된 [이벤트 처리기](#response-event-handlers)를 통과한 후 채널로 전송됩니다.

## <a name="middleware"></a>미들웨어

미들웨어는 각각 순서대로 추가되고 실행되는 선형 구성 요소 집합으로, 봇의 순서 처리기 전후로 활동에서 작동할 가능성이 있으며, 해당 활동의 순서 컨텍스트에 액세스할 수 있습니다. 미들웨어가 [단락](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting)되지 않는 한, 미들웨어 파이프라인의 최종 단계는 스택을 반환하기 전에 봇의 순서 처리기를 호출하는 콜백입니다. 미들웨어에 대한 자세한 내용은 [미들웨어 항목](~/v4sdk/bot-builder-concept-middleware.md)을 참조하세요.

## <a name="generating-responses"></a>응답 생성

순서 컨텍스트는 코드가 작업에 응답하도록 허용하는 작업 응답 메서드를 제공합니다.

* _send activity_ 및 _send activities_ 메서드는 하나 이상의 작업을 대화에 보냅니다.
* 채널에서 지원되는 경우 _update activity_ 메서드는 대화 내에서 작업을 업데이트합니다.
* 채널에서 지원되는 경우 _delete activity_ 메서드는 대화에서 작업을 제거합니다.

각 응답 메서드는 비동기 프로세스에서 실행됩니다. 작업 응답 메서드는 호출되면 처리기를 호출하기 전까지 연결된 [이벤트 처리기](#response-event-handlers) 목록을 복제합니다. 즉, 이 시점까지 추가된 모든 처리기를 포함하지만, 프로세스가 시작된 후에 추가된 항목을 전혀 포함하지 않습니다.

이는 독립적인 활동 호출의 응답 순서가 보장되지 않는다는 뜻이며, 한 작업이 다른 작업보다 복잡한 경우에 더욱 그렇습니다. 봇이 들어오는 작업에 대한 여러 응답을 생성할 수 있는 경우 사용자가 받는 순서대로 의미가 통해야 합니다. 이와 관련한 유일한 예외는 정렬된 활동 집합을 전송하는 데 사용되는 *send activities* 메서드입니다.

> [!IMPORTANT]
> 기본 봇 턴을 처리하는 스레드는 사용을 마친 컨텍스트 개체를 폐기합니다. 턴 컨텍스트의 처리 및 폐기가 완료될 때까지 기본 스레드가 생성된 작업을 기다리도록 **작업 호출을 `await`해야 합니다.** 또는 응답(처리기 포함)이 상당한 시간을 소요하고 컨텍스트 개체에 따라 작동하는 경우 `Context was disposed` 오류가 발생할 수 있습니다. 

## <a name="response-event-handlers"></a>응답 이벤트 처리기

응용 프로그램 및 미들웨어 논리 외에도, 컨텍스트 개체에 응답 처리기(이벤트 처리기 또는 작업 이벤트 처리기라고도 함)를 추가할 수 있습니다. 이러한 처리기는 현재 컨텍스트 개체에서 관련 [응답](#generating-responses)이 발생하면 실제 응답을 실행하기 전에 호출됩니다. 이러한 처리기는 실제 이벤트 전에 또는 후에 현재 응답의 나머지 부분에서 해당 형식의 모든 작업에 대해 무엇을 할 것인지 알고 있는 경우에 유용 합니다.

> [!WARNING]
> 각 응답 이벤트 처리기 내부에서 작업 응답 메서드를 호출하지 않도록 주의해야 합니다. 예를 들어 _on send activity_ 처리기 내에서 send activity 메서드를 호출하면 안 됩니다. 호출하면 무한 루프가 생성될 수 있습니다.

새 작업마다 실행할 새 스레드가 생깁니다. 작업을 처리하는 스레드가 생성되면 해당 작업에 대한 처리기 목록이 해당하는 새 스레드에 복사됩니다. 해당 시점 이후 추가된 처리기는 해당 활동 이벤트에 대해 실행되지 않습니다.

컨텍스트 개체에 등록된 처리기는 어댑터에서 [미들웨어 파이프라인](~/v4sdk/bot-builder-concept-middleware.md#the-bot-middleware-pipeline)을 관리하는 방법과 매우 비슷하게 처리됩니다. 즉, 처리기는 추가된 순서대로 호출되며, _다음_ 대리자를 호출하면 등록된 그 다음 이벤트 처리기에 컨트롤이 전달됩니다. 처리기가 다음 대리자를 호출하지 않으면 후속 이벤트 처리기는 호출되지 않으며([단락](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting) 이벤트), 어댑터는 채널에 응답을 보내지 않습니다.

## <a name="bot-structure"></a>봇 구조체

카운터를 사용하는 에코 봇[[C#](https://aka.ms/EchoBotWithStateCSharp) | [JS](https://aka.ms/EchoBotWithStateJS)] 샘플을 살펴보고, 봇의 주요 요소를 살펴보겠습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

봇은 [ASP.NET Core 웹](https://docs.microsoft.com/aspnet/core/?view=aspnetcore-2.1) 응용 프로그램의 한 유형입니다. [ASP.NET 기본 사항](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x)을 보면 Program.cs 및 Startup.cs와 같은 파일에서 유사한 코드를 확인할 수 있습니다. 이러한 파일은 모든 웹앱에 필요하며 봇 전용이 아닙니다. 이러한 파일 중 일부 파일의 코드는 여기에 복사되지 않지만 카운터를 사용하는 에코 봇 샘플을 참조할 수 있습니다.

### <a name="echowithcounterbotcs"></a>EchoWithCounterBot.cs

기본 봇 논리는 `IBot` 인터페이스에서 파생되는 `EchoWithCounterBot` 클래스에 정의됩니다. `IBot`은 단일 메서드 `OnTurnAsync`를 정의합니다. 응용 프로그램은 이 메서드를 구현해야 합니다. `OnTurnAsync`에는 수신 활동에 대한 정보를 제공하는 turnContext가 있습니다. 수신 활동은 인바운드 HTTP 요청에 해당합니다. 수신 활동은 형식이 다양할 수 있으므로 먼저 봇이 메시지를 받았는지 확인합니다. 메시지라면 순서 컨텍스트에서 대화 상태를 가져오고, 순서 카운터를 구현한 다음, 대화 상태에 새로운 순서 카운터 값을 보존합니다. 그런 다음, SendActivityAsync 호출을 사용하여 사용자에게 메시지를 다시 전송합니다. 송신 활동은 아웃바운드 HTTP 요청에 해당합니다.

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var oldState = await _accessors.CounterState.GetAsync(turnContext, () => new CounterState());

        // Bump the turn count for this conversation.
        var newState = new CounterState { TurnCount = oldState.TurnCount + 1 };

        // Set the property using the accessor.
        await _accessors.CounterState.SetAsync(turnContext, newState);

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        // Echo back to the user whatever they typed.
        var responseMessage = $"Turn {newState.TurnCount}: You sent '{turnContext.Activity.Text}'\n";
        await turnContext.SendActivityAsync(responseMessage);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

### <a name="startupcs"></a>Startup.cs

`ConfigureServices` 메소드는 [.bot](bot-builder-basics.md#the-bot-file) 파일에서 연결된 서비스를 로드하고, 대화 순서 중에 발생하는 오류를 포착하여 기록하고, 자격 증명 공급자를 설정하고, 대화 상태 개체를 만들어 메모리에 대화 데이터를 저장합니다.

```csharp
services.AddBot<EchoWithCounterBot>(options =>
{
    // Creates a logger for the application to use.
    ILogger logger = _loggerFactory.CreateLogger<EchoWithCounterBot>();

    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    BotConfiguration botConfig = null;
    try
    {
        botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    }
    catch
    {
        //...
    }

    services.AddSingleton(sp => botConfig);

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

    // Catches any errors that occur during a conversation turn and logs them.
    options.OnTurnError = async (context, exception) =>
    {
        logger.LogError($"Exception caught : {exception}");
        await context.SendActivityAsync("Sorry, it looks like something went wrong.");
    };

    // The Memory Storage used here is for local bot debugging only. When the bot
    // is restarted, everything stored in memory will be gone.
    IStorage dataStore = new MemoryStorage();

    // ...

    // Create Conversation State object.
    // The Conversation State object is where we persist anything at the conversation-scope.
    var conversationState = new ConversationState(dataStore);

    options.State.Add(conversationState);
});
```

또한 `EchoBotAccessors`를 만들고 등록하는데, 이는 **EchoBotStateAccessors.cs** 파일에 정의되고, ASP.NET Core의 종속성 주입 프레임워크를 사용하여 공용 `EchoWithCounterBot` 생성자로 전달됩니다.

```csharp
// Create and register state accessors.
// Accessors created here are passed into the IBot-derived class on every turn.
services.AddSingleton<EchoBotAccessors>(sp =>
{
    var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
    // ...
    var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
    // ...

    // Create the custom state accessor.
    // State accessors enable other components to read and write individual properties of state.
    var accessors = new EchoBotAccessors(conversationState)
    {
        CounterState = conversationState.CreateProperty<CounterState>(EchoBotAccessors.CounterStateName),
    };

    return accessors;
});
```

`Configure` 메서드는 Bot Framework 및 몇 가지 다른 파일을 사용하도록 지정하여 앱의 구성을 완료합니다. Bot Framework를 사용하는 모든 봇은 해당 구성 호출이 필요합니다. `ConfigureServices` 및 `Configure`는 앱이 시작될 때 런타임에서 호출합니다.

### <a name="counterstatecs"></a>CounterState.cs

이 파일에는 봇이 현재 상태를 유지하기 위해 사용하는 간단한 클래스가 포함됩니다. 카운터를 증분시키기 위해 사용하는 `int`만 포함됩니다.

```cs
public class CounterState
{
    public int TurnCount { get; set; } = 0;
}
```

### <a name="echobotaccessorscs"></a>EchoBotAccessors.cs

`EchoBotAccessors` 클래스는 `Startup` 클래스에 싱글톤으로 생성되어 IBot 파생 클래스로 전달됩니다. 이 예제의 경우 `public class EchoWithCounterBot : IBot`입니다. 이 봇은 접근자를 사용하여 대화 데이터를 보존합니다. `EchoBotAccessors`의 생성자는 Startup.cs 파일에 생성되는 대화 개체에 전달됩니다.

```cs
public class EchoBotAccessors
{
    public EchoBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string CounterStateName { get; } = $"{nameof(EchoBotAccessors)}.CounterState";

    public IStatePropertyAccessor<CounterState> CounterState { get; set; }

    public ConversationState ConversationState { get; }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

시스템 섹션에는 주로 **package.json**, **.env** , **index.js** 및 **README.md** 파일이 포함됩니다. 일부 파일의 코드는 여기에 복사되지 않지만, 봇을 실행할 때 표시됩니다.

### <a name="packagejson"></a>package.json

**package.json**은 봇의 종속 항목 및 관련 버전을 지정합니다. 이는 모두 템플릿과 시스템에 의해 설정됩니다.

### <a name="env-file"></a>.env 파일

**.env** 파일은 포트 번호, 앱 ID 및 암호 등과 같은 봇의 구성 정보를 지정합니다. 특정 기술을 사용하거나 프로덕션 환경에서 이 봇을 사용하는 경우, 이 구성에 특정 키 또는 URL을 추가해야 합니다. 그러나 이 Echo 봇의 경우에는 이 시점에 아무 작업도 수행할 필요가 없습니다. 이 때 앱 ID와 암호는 정의되지 않은 상태로 남겨둘 수 있습니다.

**.env** 구성 파일을 사용하려면 템플릿에 추가 패키지가 포함되어 있어야 합니다.  먼저 npm에서 `dotenv` 패키지를 가져옵니다.

`npm install dotenv`

### <a name="indexjs"></a>index.js

`index.js`는 봇 논리로 활동을 전달하는 호스팅 서비스와 봇을 설정합니다.

#### <a name="required-libraries"></a>필수 라이브러리

`index.js` 파일의 맨 위쪽에는 필요한 일련의 모듈 또는 라이브러리가 있습니다. 이러한 모듈은 사용자가 응용 프로그램에 포함하는 함수 집합에 액세스할 수 있습니다.

```javascript
// Import required packages
const path = require('path');
const restify = require('restify');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
// Import required bot configuration.
const { BotConfiguration } = require('botframework-config');

const { EchoBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
const env = require('dotenv').config({ path: ENV_FILE });
```

#### <a name="bot-configuration"></a>봇 구성

다음 부분에서는 봇 구성 파일에서 정보를 로드합니다.

```javascript
// Get the .bot file path
// See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));
let botConfig;
try {
    // Read bot configuration from .bot file.
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.`);
    console.error(`\n - See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.\n\n`);
    process.exit();
}

// For local development configuration as defined in .bot file
const DEV_ENVIRONMENT = 'development';

// Define name of the endpoint configuration section from the .bot file
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Get bot endpoint configuration by service name
// Bot configuration as defined in .bot file
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
```

#### <a name="bot-adapter-http-server-and-bot-state"></a>봇 어댑터, HTTP 서버 및 봇 상태

새로운 부분에서는 봇이 사용자와 통신하고 응답을 보낼 수 있게 해주는 서버와 어댑터를 설정합니다. 서버는 **BotConfiguration.bot** 구성 파일의 지정된 포트에서 수신 대기하거나, 에뮬레이터와의 연결을 위해 _3978_로 폴백합니다. 어댑터는 봇의 지휘자 역할을 하며 들어오고 나가는 통신, 인증 등을 지시합니다.

또한 `MemoryStorage`를 저장소 공급자로 사용하는 상태 개체를 만듭니다. 이 상태는 `ConversationState`로 정의됩니다. 이는 대화의 상태를 유지한다는 것을 의미합니다. `ConversationState`는 사용자가 관심 있어 하는 정보를 메모리에 저장합니다(이 경우에는 턴 카운터만 메모리에 저장됨).

```javascript
// Create bot adapter.
// See https://aka.ms/about-bot-adapter to learn more about bot adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.microsoftAppID,
    appPassword: endpointConfig.appPassword || process.env.microsoftAppPassword
});

// Catch-all for any unhandled errors in your bot.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
};

// Define a state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state store to persist the dialog and user state between messages.
let conversationState;

// For local development, in-memory storage is used.
// CAUTION: The Memory Storage used here is for local bot debugging only. When the bot
// is restarted, anything stored in memory will be gone.
const memoryStorage = new MemoryStorage();
conversationState = new ConversationState(memoryStorage);

// Create the main dialog.
const bot = new EchoBot(conversationState);

// Create HTTP server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator`);
    console.log(`\nTo talk to your bot, open echoBot-with-counter.bot file in the Emulator`);
});
```

#### <a name="bot-logic"></a>봇 논리

어댑터의 `processActivity`는 수신 활동을 봇 논리로 보냅니다.
`processActivity` 내의 세 번째 매개 변수는 수신된 [활동](#the-activity-processing-stack)이 어댑터에 의해 사전 처리되고 모든 미들웨어를 통해 라우팅된 후 봇의 논리를 수행하기 위해 호출되는 함수 처리기입니다. 함수 처리기에 인수로 전달된 순서 컨텍스트 변수는 수신 활동, 보낸 사람 및 받는 사람, 채널, 대화 등에 대한 정보를 제공하는 데 사용될 수 있습니다. 활동 처리가 EchoBot의 `onTurn`으로 라우팅됩니다.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="echobot"></a>EchoBot

모든 활동 처리가 이 클래스의 `onTurn` 처리기로 라우팅됩니다. 클래스가 생성되면 상태 개체가 전달됩니다. 생성자는 이 상태 개체를 사용하여 이 봇의 순서 카운터를 보존하는 `this.countProperty` 접근자를 만듭니다.

각 순서에서는 먼저 봇 메시지를 받았는지 확인합니다. 봇이 메시지를 받지 못하면 수신된 활동 유형을 다시 반환합니다. 다음으로, 봇 대화의 정보를 포함하는 상태 변수를 만듭니다. 카운트 변수가 `undefined`일 경우 1로 설정되거나(봇이 먼저 시작될 경우) 새 메시지가 있을 때마다 1씩 증분됩니다. 사용자에게 보낸 메시지와 함께 카운트를 다시 반환합니다. 마지막으로, 카운트를 설정하고 변경 내용을 상태에 저장합니다.

```javascript
const { ActivityTypes } = require('botbuilder');

// Turn counter property
const TURN_COUNTER_PROPERTY = 'turnCounterProperty';

class EchoBot {

    constructor(conversationState) {
        // Creates a new state accessor property.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors
        this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
        this.conversationState = conversationState;
    }

    async onTurn(turnContext) {
        // Handle message activity type. User's responses via text or speech or card interactions flow back to the bot as Message activity.
        // Message activities may contain text, speech, interactive cards, and binary or unknown attachments.
        // see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
        if (turnContext.activity.type === ActivityTypes.Message) {
            // read from state.
            let count = await this.countProperty.get(turnContext);
            count = count === undefined ? 1 : ++count;
            await turnContext.sendActivity(`${ count }: You said "${ turnContext.activity.text }"`);
            // increment and set turn counter.
            await this.countProperty.set(turnContext, count);
        } else {
            // Generic handler for all other activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
    }
}

exports.EchoBot = EchoBot;
```

---

### <a name="the-bot-file"></a>봇 파일

**.bot** 파일에는 엔드포인트, 앱 ID, 암호 및 봇에서 사용하는 서비스에 대한 참조를 비롯한 정보가 포함됩니다. 이 파일은 템플릿에서 봇을 빌드할 때 생성되지만 에뮬레이터 또는 다른 도구를 통해 직접 만들 수 있습니다. [에뮬레이터](../bot-service-debug-emulator.md)로 봇을 테스트할 때 사용할 .bot 파일을 지정할 수 있습니다.

```json
{
    "name": "echobot-with-counter",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "1"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

## <a name="additional-resources"></a>추가 리소스

상태 관리에 대한 자세한 내용은 [대화 및 사용자 상태를 관리하는 방법](bot-builder-howto-v4-state.md)을 참조하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [봇 만들기](~/bot-service-quickstart.md)
