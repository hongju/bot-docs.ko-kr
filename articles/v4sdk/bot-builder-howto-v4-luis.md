---
title: 봇에 자연어 해석 추가 | Microsoft Docs
description: 자연어 이해를 위한 LUIS를 Bot Builder SDK와 함께 사용하는 방법을 알아봅니다.
keywords: Language Understanding, LUIS, 의도, 인식기, 엔터티, 미들웨어
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/08/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: eab8e2f9d437748d0bb0fefd31c03c8fb350c6b1
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645703"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>봇에 자연어 해석 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

사용자가 대화 및 컨텍스트를 통해 의미하는 바를 이해하는 기능을 구현하기가 어려울 수 있지만, 봇에 보다 자연스러운 대화 느낌을 제공할 수 있습니다. LUIS라고도 하는 Language Understanding을 사용하면 이와 같은 기능을 구현하여 봇이 사용자 메시지의 의도를 인식하고, 사용자가 보다 자연스러운 언어를 사용하고, 대화 흐름을 보다 원활하게 안내할 수 있습니다. LUIS의 자세한 배경이 필요하면 봇에 대한 [언어 해석](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis)을 참조하세요.

## <a name="prerequisites"></a>필수 조건
이 토픽에서는 LUIS를 사용하여 몇 가지 의도를 인식하는 간단한 봇을 설정하는 방법을 안내합니다. 이 문서의 코드는 LUIS를 사용하는 NLP 샘플에 기반을 두고 [C#](https://aka.ms/cs-luis-sample) 및 [JavaScript](https://aka.ms/js-luis-sample)로 작성되었습니다.

## <a name="create-a-luis-app-in-the-luis-portal"></a>LUIS 포털에서 LUIS 앱 만들기

먼저 [여기](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-start-new-app)에 나와 있는 지침에 따라 [luis.ai](https://www.luis.ai)에서 계정에 등록하고 LUIS 포털에서 LUIS 앱을 만듭니다. 이 문서에서 사용된 샘플 LUIS 앱을 직접 만들려면 LUIS 포털 내에서 이 `LUIS.Reminders.json` 파일([C#](https://github.com/Microsoft/BotBuilder-Samples/blob/v4/samples/csharp_dotnetcore/12.nlp-with-luis/CognitiveModels/LUIS-Reminders.json) | [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/cognitiveModels/reminders.json))을 [가져와](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) LUIS 앱을 빌드한 다음, [교육](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)시키고 [게시](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)합니다.

### <a name="obtain-values-to-connect-to-your-luis-app"></a>LUIS 앱에 연결할 값 가져오기

LUIS 앱이 게시되면 봇에서 액세스할 수 있습니다. 봇 내에서 LUIS 앱에 액세스하려면 여러 값을 기록해야 합니다. LUIS 포털 또는 CLI 도구를 사용하여 해당 정보를 검색할 수 있습니다.

#### <a name="using-luis-portal"></a>LUIS 포털 사용
- [luis.ai](https://www.luis.ai)에서 게시된 LUIS 앱을 선택합니다.
- 게시된 LUIS 앱이 열리면 **관리** 탭을 선택합니다.
- 왼쪽에서 **응용 프로그램 정보** 탭을 선택하고 _응용 프로그램 ID_에 <YOUR_APP_ID>로 표시되는 값을 기록합니다.
- 왼쪽에서 **키 및 엔드포인트** 탭을 선택하고 _제작 키_에 <YOUR_AUTHORING_KEY>로 표시되는 값을 기록합니다. <YOUR_SUBSCRIPTION_KEY>가 <YOUR_AUTHORING_KEY>와 동일한지 확인합니다. 페이지 맨 아래로 스크롤하고 _지역_에 <YOUR_REGION>으로 표시되는 값을 기록하고, _엔드포인트_에 <YOUR_ENDPOINT>로 표시되는 값을 기록합니다.

#### <a name="using-cli-tools"></a>CLI 도구 사용

[luis](https://aka.ms/botbuilder-tools-luis) 및 [msbot](https://aka.ms/botbuilder-tools-msbot-readme) BotBuilder CLI 도구를 사용하여 LUIS 앱에 대한 메타데이터를 가져와서 **.bot** 파일에 추가할 수 있습니다.

1. 터미널 또는 명령 프롬프트를 열고 봇 프로젝트의 루트 디렉터리로 이동합니다.
2. `luis` 및 `msbot` 도구가 설치되었는지 확인합니다.

    ```shell
    npm install luis msbot
    ```

3. `luis init`를 실행하여 LUIS 리소스 파일(**.luisrc**)을 만듭니다. 메시지가 표시되면 LUIS 제작 키 및 지역을 제공합니다. 지금은 앱 ID를 입력할 필요가 없습니다.
4. 다음 명령을 실행하여 메타데이터를 다운로드하고 봇의 구성 파일에 추가합니다.
    구성 파일을 암호화한 경우 비밀 키를 제공하여 파일을 업데이트해야 합니다.

    ```shell
    luis get application --appId <your-app-id> --msbot | msbot connect luis --stdin [--secret <YOUR-SECRET>]
    ```

## <a name="configure-your-bot-to-use-your-luis-app"></a>LUIS 앱을 사용하도록 봇 구성

봇을 초기화하면 LUIS 앱에 대한 참조가 먼저 추가됩니다. 그러면 봇 논리 내에서 호출할 수 있습니다.

### <a name="prerequisite"></a>필수 요소

코딩을 시작하기 전에 LUIS 앱에 필요한 패키지가 있는지 확인합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

다음 [NuGet 패키지](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)를 봇에 추가합니다.

* `Microsoft.Bot.Builder.AI.Luis`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

LUIS 기능은 `botbuilder-ai` 패키지에 있습니다. npm을 통해 프로젝트에 이 패키지를 추가할 수 있습니다.

```shell
npm install --save botbuilder-ai
```

---

# <a name="ctabcs"></a>[C#](#tab/cs)

여기에 있는 [NLP LUIS 샘플 코드](https://aka.ms/cs-luis-sample)를 다운로드하여 엽니다. 필요에 따라 이 코드를 수정합니다. 

먼저 응용 프로그램 ID, 제작 키, 구독 키, 엔드포인트 및 지역 등 LUIS 앱에 액세스하는 데 필요한 정보를 `BotConfiguration.bot` 파일에 추가합니다. 이는 이전에 게시된 LUIS 앱에서 저장한 값입니다.

```csharp
{
  "name": "LuisBot",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "1"
    },
    {
      "type": "luis",
      "name": "LuisBot",
      "AppId": "<YOUR_APP_ID>",
      "SubscriptionKey": "<YOUR_SUBSCRIPTION_KEY>",
      "AuthoringKey": "<YOUR_AUTHORING_KEY>",
      "GetEndpoint": "<YOUR_ENDPOINT>",
      "Region": "<YOUR_REGION>"
    }
  ],
  "padlock": "",
  "version": "2.0"
}
```

다음으로, `.bot` 파일에서 위의 정보를 가져오는 BotService 클래스 `BotServices.cs`의 새 인스턴스를 초기화합니다. `BotServices.cs` 파일에 다음 코드를 추가합니다.

```csharp
public class BotServices
{
    /// Initializes a new instance of the BotServices class
    public BotServices(BotConfiguration botConfiguration)
    {
        foreach (var service in botConfiguration.Services)
        {
            switch (service.Type)
            {
                case ServiceTypes.Luis:
                {
                    var luis = (LuisService)service;
                    if (luis == null)
                    {
                        throw new InvalidOperationException("The LUIS service is not configured correctly in your '.bot' file.");
                    }

                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    this.LuisServices.Add(luis.Name, recognizer);
                    break;
                    }
                }
            }
        }

    /// Gets the set of LUIS Services used.
    /// LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

그런 다음, `ConfigureServices` 내에 다음 코드를 추가하여 LUIS 앱을 `Startup.cs` 파일에 싱글톤으로 등록합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Initialize Bot Connected Services clients.
    var connectedServices = new BotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    services.AddSingleton(sp => botConfig);

    services.AddBot<LuisBot>(options =>
    {
        // Retrieve current endpoint.
        var environment = _isProduction ? "production" : "development";
        var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
        if (!(service is EndpointService endpointService))
        {
            throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
        }

        options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

        // Creates a logger for the application to use.
        ILogger logger = _loggerFactory.CreateLogger<LuisBot>();

        // Catches any errors that occur during a conversation turn and logs them.
        options.OnTurnError = async (context, exception) =>
        {
            logger.LogError($"Exception caught : {exception}");
            await context.SendActivityAsync("Sorry, it looks like something went wrong.");
        };
        /// ...
    });
}
```

다음으로, 이 LUIS 인스턴스를 봇에 제공해야 합니다. `LuisBot.cs`를 열고 파일 맨 위에 다음 코드를 추가합니다.

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    /// Services configured from the ".bot" file.
    private readonly BotServices _services;

    /// Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a LUIS service named '{LuisKey}'.");
        }
    }
    /// ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

이 샘플에서 시작 코드는 **index.js** 파일에 있으며, 봇 논리의 코드는 **bot.js** 파일에 있고, 추가 구성 정보는 **nlp-with-luis.bot** 파일에 있습니다.

LUIS 앱을 만들고 **.bot** 파일을 업데이트하는 방법에 대한 지침에 따라 **nlp-with-luis.bot** 파일에 LUIS 앱의 서비스 항목도 포함되어야 합니다.

```json
{
    "name": "YOUR_LUIS_APP_NAME",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "35"
        },
        {
            "type": "luis",
            "name": "YOUR_LUIS_APP_NAME",
            "appId": "<YOUR_APP_ID>",
            "version": "0.1",
            "authoringKey": "<YOUR_AUTHORING_KEY>",
            "subscriptionKey": "<YOUR_SUBSCRIPTION_KEY>>",
            "region": "<YOUR_REGION>",
            "id": "83"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

**bot.js** 파일에서 구성 정보를 읽어들여 LUIS 서비스를 생성하고 봇을 초기화합니다.
`LUIS_CONFIGURATION` 값을 구성 파일에 표시되는 LUIS 앱의 이름으로 업데이트합니다.

```javascript
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the C# code.
const LUIS_CONFIGURATION = '<YOUR_LUIS_APP_NAME>';

// Get endpoint and LUIS configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const luisConfig = botConfig.findServiceByNameOrId(LUIS_CONFIGURATION);

// Map the contents to the required format for `LuisRecognizer`.
const luisApplication = {
    applicationId: luisConfig.appId,
    endpointKey: luisConfig.subscriptionKey || luisConfig.authoringKey,
    azureRegion: luisConfig.region
};

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
};

// Create adapter...

// Create the LuisBot.
let bot;
try {
    bot = new LuisBot(luisApplication, luisPredictionOptions);
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

그런 다음, HTTP 서버를 만들고 수신 요청을 대시하면 봇 논리에 대한 호출이 생성됩니다.

```javascript
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open nlp-with-luis.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async(turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

이제 봇에 LUIS가 구성되었습니다. 다음으로, LUIS에서 의도를 가져오는 방법을 살펴보겠습니다.

## <a name="get-the-intent-by-calling-luis"></a>LUIS 호출하여 의도 가져오기

봇은 LUIS 인식기를 호출하여 LUIS에서 결과를 가져옵니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

봇이 LUIS 앱에서 검색한 의도에 따라 응답을 보내게 하려면 `LuisRecognizer`를 호출하여 `RecognizerResult`를 가져옵니다. 이 작업은 LUIS 의도를 가져와야 할 때마다 코드 내에서 수행될 수 있습니다.

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))

{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Check LUIS model
        var recognizerResult = await _services.LuisServices[LuisKey].RecognizeAsync(turnContext, cancellationToken);
        var topIntent = recognizerResult?.GetTopScoringIntent();
        if (topIntent != null && topIntent.HasValue && topIntent.Value.intent != "None")
        {
            await turnContext.SendActivityAsync($"==>LUIS Top Scoring Intent: {topIntent.Value.intent}, Score: {topIntent.Value.score}\n");
        }
        else
        {
            var msg = @"No LUIS intents were found.
                        This sample is about identifying two user intents:
                        'Calendar.Add'
                        'Calendar.Find'
                        Try typing 'Add Event' or 'Show me tomorrow'.";
            await turnContext.SendActivityAsync(msg);
        }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            // Send a welcome message to the user and tell them what actions they may perform to use this bot
            await SendWelcomeMessageAsync(turnContext, cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected", cancellationToken: cancellationToken);
        }
}
```

발언에서 인식된 의도는 의도 이름 맵으로 점수에 반환되며 `recognizerResult.Intents`에서 액세스할 수 있습니다. 정적 `recognizerResult?.GetTopScoringIntent()` 메서드는 결과 집합에 대한 최고점 의도를 간단하게 찾을 수 있도록 도와주기 위해 제공됩니다.

인식된 엔터티는 엔터티 이름의 맵으로 값에 반환되며 `recognizerResults.entities`를 사용하여 액세스할 수 있습니다. LuisRecognizer를 만들 때 `verbose=true` 설정을 전달하여 추가 엔터티 메타데이터를 반환할 수 있습니다. 추가된 메타데이터는 `recognizerResults.entities.$instance`를 사용하여 액세스할 수 있습니다.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**bot.js** 파일에서 LUIS 인식기의 `recognize` 메서드에 사용자 입력을 전달하여 LUIS 앱에서 응답을 가져옵니다.

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from the Language Understanding (LUIS) service.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class LuisBot {
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {LuisApplication} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {LuisPredictionOptions} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions, includeApiResults) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }

    /**
     * Every conversation turn calls this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls LUIS in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;

            if (topIntent.intent !== 'None') {
                await turnContext.sendActivity(`LUIS Top Scoring Intent: ${ topIntent.intent }, Score: ${ topIntent.score }`);
            } else {
                // If the top scoring intent was "None" tell the user no valid intents were found and provide help.
                await turnContext.sendActivity(`No LUIS intents were found.
                                                \nThis sample is about identifying two user intents:
                                                \n - 'Calendar.Add'
                                                \n - 'Calendar.Find'
                                                \nTry typing 'Add Event' or 'Show me tomorrow'.`);
            }
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
            turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            // If the Activity is a ConversationUpdate, send a greeting message to the user.
            await turnContext.sendActivity('Welcome to the NLP with LUIS sample! Send me a message and I will try to predict your intent.');
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            // Respond to all other Activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.LuisBot = LuisBot;
```

LUIS 인식기는 발언이 사용 가능한 의도와 일치하는 정도에 대한 정보를 반환합니다. 결과 개체의 `luisResult.intents` 속성에는 채점된 의도의 배열이 포함됩니다. 결과 개체의 `luisResult.topScoringIntent` 속성에는 최고 점수의 의도와 점수가 포함됩니다.

---

## <a name="extract-entities"></a>엔터티 추출

의도를 인식하는 것 외에, LUIS 앱은 사용자 요청 이행에 중요한 단어에 해당하는 엔터티도 추출할 수 있습니다. 예를 들어 날씨 봇의 경우 LUIS 앱은 사용자의 메시지에서 날씨 보고서의 위치를 추출할 수 있습니다.

대화를 구성하는 일반적인 방법은 사용자의 메시지에서 엔터티를 식별하고, 찾지 못한 필수 엔터티에 대한 프롬프트를 표시하는 것입니다. 그런 다음, 후속 단계에서 해당 프롬프트에 대한 응답을 처리합니다.

<!--Snip
# [C#](#tab/cs)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult) with an [`Entities` property](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-) that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

The following helper function can be added to your bot to get entities out of the `RecognizerResult` from LUIS. It will require the use of the `Newtonsoft.Json.Linq` library, which you'll have to add to your **using** statements.

```cs
// Get entities from LUIS result
private T GetEntity<T>(RecognizerResult luisResult, string entityKey)
{
    var data = luisResult.Entities as IDictionary<string, JToken>;
    if (data.TryGetValue(entityKey, out JToken value))
    {
        return value.First.Value<T>();
    }
    return default(T);
}
```

When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

# [JavaScript](#tab/js)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult) with an `entities` property that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

This `findEntities` function looks for any entities recognized by the LUIS app that match the incoming `entityName`.

```javascript
// Helper function for finding a specified entity
// entityResults are the results from LuisRecognizer.get(context)
function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach(entity => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}
```
/Snip-->

대화의 여러 단계에서 엔터티 같은 정보를 수집할 때 필요한 정보를 상태에 저장하면 도움이 될 수 있습니다. 엔터티를 발견한 경우 적절한 상태 필드에 추가할 수 있습니다. 대화에서 현재 단계가 이미 연결된 필드를 완료한 경우 해당 정보에 대한 프롬프트를 표시하는 단계는 건너뛸 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

LUIS를 사용하는 샘플은 [[C#](https://aka.ms/cs-luis-sample)] 또는 [[JavaScript](https://aka.ms/js-luis-sample)]에 대한 프로젝트를 참조하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [디스패치 도구를 사용하여 LUIS 및 QnA 서비스 결합](./bot-builder-tutorial-dispatch.md)
