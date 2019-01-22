---
title: 봇에 자연어 해석 추가 | Microsoft Docs
description: 자연어 이해를 위한 LUIS를 Bot Framework SDK와 함께 사용하는 방법을 알아봅니다.
keywords: Language Understanding, LUIS, 의도, 인식기, 엔터티, 미들웨어
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/28/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77dbf8658030a18596507129c88156601d4272e5
ms.sourcegitcommit: d385ec5fe61c469ab17e6f21b4a0d50e5110d0fd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54298310"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>봇에 자연어 해석 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

사용자가 대화 및 컨텍스트를 통해 의미하는 바를 이해하는 기능을 구현하기가 어려울 수 있지만, 봇에 보다 자연스러운 대화 느낌을 제공할 수 있습니다. LUIS라고도 하는 Language Understanding을 사용하면 이와 같은 기능을 구현하여 봇이 사용자 메시지의 의도를 인식하고, 사용자가 보다 자연스러운 언어를 사용하고, 대화 흐름을 보다 원활하게 안내할 수 있습니다. 이 토픽에서는 LUIS를 사용하여 몇 가지 의도를 인식하는 간단한 봇을 설정하는 방법을 안내합니다. 
## <a name="prerequisites"></a>필수 조건
- [luis.ai](https://www.luis.ai) 계정
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- 이 문서의 코드는 **LUIS를 통한 NLP** 샘플을 기반으로 합니다. [C#](https://aka.ms/cs-luis-sample) 또는 [JS](https://aka.ms/js-luis-sample)로 작성된 샘플의 복사본이 필요합니다. 
- [봇 기본 ](bot-builder-basics.md), [자연어 처리](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis) 및 [자연어 처리](bot-file-basics.md) 파일에 대한 지식이 필요합니다.

## <a name="create-a-luis-app-in-the-luis-portal"></a>LUIS 포털에서 LUIS 앱 만들기
LUIS 포털에 로그인하여 사용자 고유 버전의 LUIS 샘플 앱을 만듭니다. 애플리케이션은 **내 앱**에서 만들고 관리할 수 있습니다. 

1. **새 앱 가져오기**를 선택합니다. 
1. **앱 파일 선택(JSON 형식)...** 을 클릭합니다. 
1. 샘플의 `CognitiveModels` 폴더에 있는 `reminders.json` 파일을 선택합니다. **옵션 이름**에서 **LuisBot**을 입력합니다. 이 파일에는 세 가지 의도 Calendar_Add, Calendar_Find 및 없음이 포함되어 있습니다. 이러한 의도를 사용하여 사용자가 봇에 메시지를 보낼 때 의미하는 바를 이해합니다. 엔터티를 포함시키려면 이 문서의 끝 부분에 나오는 [선택 사항 섹션](#optional---extract-entities)을 참조하세요.
1. 앱을 [학습](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)합니다.
1. 앱을 *프로덕션* 환경에 [게시](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)합니다.

### <a name="obtain-values-to-connect-to-your-luis-app"></a>LUIS 앱에 연결할 값 가져오기

LUIS 앱이 게시되면 봇에서 액세스할 수 있습니다. 봇 내에서 LUIS 앱에 액세스하려면 여러 값을 기록해야 합니다. LUIS 포털을 사용하여 해당 정보를 검색할 수 있습니다.

#### <a name="retrieve-application-information-from-the-luisai-portal"></a>LUIS.ai 포털에서 애플리케이션 정보 검색
.bot 파일은 모든 서비스 참조를 한 곳에 모을 수 있는 장소입니다. 검색한 정보는 다음 섹션에서 .bot 파일에 추가됩니다. 
1. [luis.ai](https://www.luis.ai)에서 게시된 LUIS 앱을 선택합니다.
1. 게시된 LUIS 앱이 열리면 **관리** 탭을 선택합니다.
1. 왼쪽에서 **애플리케이션 정보** 탭을 선택하고 _애플리케이션 ID_에 &lt;YOUR_APP_ID&gt;로 표시되는 값을 기록합니다.
1. 왼쪽에서 **키 및 엔드포인트** 탭을 선택하고 _제작 키_에 <YOUR_AUTHORING_KEY>로 표시되는 값을 기록합니다. *구독 키*는 *제작 키*와 동일합니다. 
1. 페이지 끝까지 아래로 스크롤하여 _지역_에 대해 표시된 값을 <YOUR_REGION>으로 기록합니다.
1. _엔드포인트_에 표시된 값을 <YOUR_ENDPOINT>로 기록합니다.

#### <a name="update-the-bot-file"></a>봇 파일 업데이트
애플리케이션 ID, 제작 키, 구독 키, 엔드포인트 및 지역을 포함하여 LUIS 앱에 액세스하는 데 필요한 정보를 `nlp-with-luis.bot` 파일에 추가합니다. 이는 이전에 게시된 LUIS 앱에서 저장한 값입니다.

```json
{
    "name": "LuisBot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "166"
        },
        {
            "type": "luis",
            "name": "LuisBot",
            "appId": "<luis appid>",
            "version": "0.1",
            "authoringKey": "<luis authoring key>",
            "subscriptionKey": "<luis subscription key>",
            "region": "<luis region>",
            "id": "158"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```
# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="configure-your-bot-to-use-your-luis-app"></a>LUIS 앱을 사용하도록 봇 구성

다음으로, `.bot` 파일에서 위의 정보를 가져오는 BotService 클래스 `BotServices.cs`의 새 인스턴스를 초기화합니다. 외부 서비스는 `BotConfiguration` 클래스를 사용하여 구성됩니다.

```csharp
public class BotServices
{
    // Initializes a new instance of the BotServices class
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

    // Gets the set of LUIS Services used. LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

그런 다음, `ConfigureServices` 메서드 내에서 다음 코드를 사용하여 LUIS 앱을 싱글톤으로 `Startup.cs` 파일에 등록합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-luis.bot", secretKey);
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
        
        // ...
    });
}
```

다음으로, `Luis.cs` 파일에서 봇은 이 LUIS 인스턴스를 가져옵니다.

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    // Services configured from the ".bot" file.
    private readonly BotServices _services;

    // Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration....");
        }
    }
    // ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

이 샘플에서 시작 코드는 **index.js** 파일에 있으며, 봇 논리의 코드는 **bot.js** 파일에 있고, 추가 구성 정보는 **nlp-with-luis.bot** 파일에 있습니다.

**bot.js** 파일에서 구성 정보를 읽어들여 LUIS 서비스를 생성하고 봇을 초기화합니다.
`LUIS_CONFIGURATION` 값을 구성 파일에 표시되는 LUIS 앱의 이름으로 업데이트합니다.

```javascript
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the JavaScript code.
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

### <a name="get-the-intent-by-calling-luis"></a>LUIS 호출하여 의도 가져오기

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

### <a name="test-the-bot"></a>봇 테스트

1. 샘플을 머신에서 로컬로 실행합니다. 지침이 필요한 경우 [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/12.nlp-with-luis/README.md) 또는 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/README.md) 샘플에 대한 추가 정보 파일을 참조하세요.

1. 에뮬레이터에서 아래와 같이 메시지를 입력합니다. 

![nlp 샘플 입력 테스트](./media/nlp-luis-sample-message.png)

봇에서 상위 점수 매기기 의도(이 경우 'Calendar_Add' 의도)를 사용하여 다시 응답합니다. luis.ai 포털에서 가져온 `reminders.json` 파일에 'Calendar_Add', 'Calendar_Find' 및 'None' 의도가 정의되었음을 상기하세요. 

![nlp 샘플 응답 테스트](./media/nlp-luis-sample-response.png) 

예측 점수는 예측 결과에 대한 LUIS의 신뢰도를 나타냅니다. 예측 점수는 0(영)과 1(일) 사이입니다. 신뢰도가 높은 LUIS 점수의 예는 0.99입니다. 신뢰도 점수의 예는 0.01입니다. 

## <a name="optional---extract-entities"></a>선택 사항 - 엔터티 추출

LUIS 앱은 사용자 의도를 인식하는 것 외에도 엔터티를 반환할 수 있습니다. 엔터티는 의도와 관련된 중요한 단어이며, 때로는 사용자의 요청을 수행하거나 봇에서 더 지능적으로 동작하도록 하는 데 필수적일 수 있습니다. 

### <a name="why-use-entities"></a>엔터티를 사용하는 이유

LUIS 엔터티를 사용하면 표준 의도와 다른 특정 사물이나 이벤트를 지능적으로 인식할 수 있습니다. 이렇게 하면 사용자로부터 추가 정보를 수집할 수 있으므로 봇이 더 지능적으로 응답하거나 사용자에게 해당 정보를 요청하는 특정 질문을 건너뛸 수 있습니다. 예를 들어 날씨 봇에서 LUIS 앱은 _Location_ 엔터티를 사용하여 사용자 메시지 내에서 요청된 날씨 보고서의 위치를 추출할 수 있습니다. 이렇게 하면 사용자의 위치에 대한 질문을 건너뛰고 사용자에게 더 스마트하고 간결한 대화를 제공할 수 있습니다.

### <a name="prerequisites"></a>필수 조건

이 샘플에서 엔터티를 사용하려면 엔터티가 포함된 LUIS 앱을 만들어야 합니다. 위의 [LUIS 앱 만들기](#create-a-luis-app-in-the-luis-portal) 섹션에 나오는 단계를 따르지만, `reminders.json` 파일을 사용하는 대신 [reminders-yould-Enterities.json](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/nlp-with-luis) 파일을 사용하여 LUIS 앱을 빌드합니다. 이 파일은 동일한 의도와 세 가지 추가 엔터티 Appointment(약속), Meeting(모임) 및 Schedule(일정)을 제공합니다. 이러한 엔터티는 LUIS에서 사용자 메시지의 의도를 결정하는 데 도움이 됩니다. 

### <a name="extract-and-display-entities"></a>엔터티 추출 및 표시
다음 선택적 코드는 LUIS에서 엔터티를 사용하여 사용자의 의도를 식별할 때 엔터티 정보를 추출하고 표시하기 위해 이 샘플 앱에 추가할 수 있습니다. 

# <a name="ctabcs"></a>[C#](#tab/cs)

다음 도우미 함수를 봇에 추가하여 LUIS의 `RecognizerResult`에서 엔터티를 가져올 수 있습니다. **using** 문에 추가해야 할 `Newtonsoft.Json.Linq` 라이브러리를 사용해야 합니다. LUIS에서 반환된 JSON을 구문 분석할 때 엔터티 정보가 있으면 _DeserializeObject_ Newtonsoft 함수에서 이 JSON을 동적 개체로 변환하여 엔터티 정보에 대한 액세스를 제공합니다.

```cs
using Newtonsoft.Json.Linq;

private string ParseLuisForEntities(RecognizerResult recognizerResult)
{
   var result = string.Empty;

   // recognizerResult.Entities returns type JObject.
   foreach (var entity in recognizerResult.Entities)
   {
       // Parse JObject for a known entity types: Appointment, Meeting, and Schedule.
       var appointFound = JObject.Parse(entity.Value.ToString())["Appointment"];
       var meetingFound = JObject.Parse(entity.Value.ToString())["Meeting"];
       var schedFound = JObject.Parse(entity.Value.ToString())["Schedule"];

       // We will return info on the first entity found.
       if (appointFound != null)
       {
           // use JsonConvert to convert entity.Value to a dynamic object.
           dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
           if (o.Appointment[0] != null)
           {
              // Find and return the entity type and score.
              var entType = o.Appointment[0].type;
              var entScore = o.Appointment[0].score;
              result = "Entity: " + entType + ", Score: " + entScore + ".";
              
              return result;
            }
        }

        if (meetingFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Meeting[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Meeting[0].type;
                var entScore = o.Meeting[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }

        if (schedFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Schedule[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Schedule[0].type;
                var entScore = o.Schedule[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }
    }

    // No entities were found, empty string returned.
    return result;
}
```

그러면 검색된 이 엔터티 정보를 식별된 사용자 의도와 함께 표시할 수 있습니다. 이 정보를 표시하려면 의도 정보가 표시된 직후 샘플 코드의 _OnTurnAsync_ 작업에 다음 몇 줄의 코드를 추가합니다.

```cs
// See if LUIS found and used an entity to determine user intent.
var entityFound = ParseLuisForEntities(recognizerResult);

// Inform the user if LUIS used an entity.
if (entityFound.ToString() != string.Empty)
{
   await turnContext.SendActivityAsync($"==>LUIS Entity Found: {entityFound}\n");
}
else
{
   await turnContext.SendActivityAsync($"==>No LUIS Entities Found.\n");
}
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

다음 코드는 LUIS에서 반환된 `luisRecognizer` 결과에서 엔터티 정보를 추출하기 위해 봇에 추가할 수 있습니다. bot.js 코드 샘플 파일의 `onTurn` 처리 내에서 _topIntent_ 상수에 대한 선언 바로 뒤에 다음 줄을 추가합니다. 그러면 반환되는 모든 엔터티 정보가 캡처됩니다. 

```javascript
// Since the LuisRecognizer was configured to include the raw results, get returned entity data.
var entityData = results.luisResult.entities;

```

반환된 엔터티 정보를 사용자에게 표시하려면 topIntent가 검색되었을 때 사용자에게 알릴 수 있도록 샘플 코드 내에서 사용되는 _sendActivity_ 호출 바로 뒤에 다음 코드 줄을 추가합니다.

```javascript
// See if LUIS found and used an entity to determine user intent.
if (entityData.length > 0)
{
   if ((entityData[0].type == "Appointment") || (entityData[0].type == "Meeting") || (entityData[0].type == "Schedule") )
   {
      // inform user if LUIS used an entity.
      await turnContext.sendActivity(`LUIS Entity Found: Entity: ${entityData[0].entity}, Score: ${entityData[0].score}.`);
   }
}
else{
       await turnContext.sendActivity(`No LUIS Entities Found.`);
}
```

이 코드는 먼저 LUIS가 반환된 결과 내에서 엔터티 정보를 반환했는지 확인하고, 반환한 경우 첫 번째 엔터티와 관련된 정보를 표시합니다.

---

### <a name="test-bot-with-entities"></a>엔터티를 사용하여 봇 테스트

1. 엔터티가 포함된 봇을 테스트하려면 [위](#test-the-bot)에서 설명한 대로 샘플을 로컬로 실행합니다.

1. 에뮬레이터에서 아래 표시된 메시지를 입력합니다. 

![nlp 샘플 입력 테스트](./media/nlp-luis-sample-message.png)

이제 봇은 사용자 의도를 결정하기 위해 'Calendar_Add' 상위 점수 매기기 의도와 LUIS에서 사용한 'Meetings' 엔터티를 모두 사용하여 다시 응답합니다.

![nlp 샘플 응답 테스트](./media/nlp-luis-sample-entity-response.png) 

엔터티를 검색하면 봇의 전반적인 성능을 향상시킬 수 있습니다. 예를 들어 "Meeting" 엔터티를 검색하면 애플리케이션에서 사용자의 일정에 새 모임을 만들 수 있도록 설계된 특수 함수를 호출할 수 있습니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [QnA Maker를 사용하여 질문에 답변](./bot-builder-howto-qna.md)
