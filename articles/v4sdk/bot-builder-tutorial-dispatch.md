---
title: 다중 LUIS 및 QnA 모델 사용 | Microsoft Docs
description: 봇에서 LUIS 및 QnA Maker를 사용하는 방법에 대해 알아봅니다.
keywords: LUIS, QnA, 디스패치 도구, 여러 서비스, 의도 라우팅
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bec6f44db929eab43cfcbbd6b2920b79924b7576
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/04/2019
ms.locfileid: "55712007"
---
# <a name="use-multiple-luis-and-qna-models"></a>다중 LUIS 및 QnA 모델 사용

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

이 자습서에서는 봇이 지원하는 여러 시나리오에 대해 여러 LUIS 모델과 QnA Maker 서비스가 있을 때 디스패치 서비스를 사용하여 발화를 라우팅하는 방법을 보여 줍니다. 이 경우 홈 자동화 및 날씨 정보와 관련된 대화를 위한 여러 LUIS 모델로 Dispatch를 구성하고, QnA Maker 서비스에서 FAQ 텍스트 파일에 기반하여 입력된 질문에 답변합니다. 이 샘플에서는 다음 서비스를 결합합니다.

| Name | 설명 |
|------|------|
| HomeAutomation | 연결된 엔터티 데이터를 사용하여 홈 자동화 의도를 인식하는 LUIS 앱입니다.|
| Weather | 위치 데이터를 사용하여 `Weather.GetForecast` 및 `Weather.GetCondition` 의도를 인식하는 LUIS 앱입니다.|
| FAQ  | 봇과 관련된 간단한 질문에 대한 답변을 제공하는 QnA Maker KB입니다. |

## <a name="prerequisites"></a>필수 조건

- 이 문서의 코드는 **디스패치를 통한 NLP** 샘플을 기반으로 합니다. [C#](https://aka.ms/dispatch-sample-cs) 또는 [JS](https://aka.ms/dispatch-sample-js)로 작성된 샘플의 복사본이 필요합니다.
- [봇 기본 사항](bot-builder-basics.md), [자연어 처리](bot-builder-howto-v4-luis.md), [QnA Maker](bot-builder-howto-qna.md) 및 [.bot](bot-file-basics.md) 파일에 대한 지식이 필요합니다.
- 테스트를 위해 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)가 필요합니다.

## <a name="create-the-services-and-test-the-bot"></a>서비스를 만들고 봇 테스트

[C#](https://aka.ms/dispatch-sample-readme-cs) 또는 [JS](https://aka.ms/dispatch-sample-readme-js)에 대한 **README** 지침을 따라 명령줄 인터페이스 호출을 사용하여 이 봇을 만들거나 아래 단계를 따라 Azure, LUIS 및 QnAMaker 사용자 인터페이스를 사용하여 수동으로 봇을 만들 수 있습니다.

 ### <a name="create-your-bot-using-service-ui"></a>서비스 UI를 사용하여 봇 만들기
 
수동으로 봇 만들기를 시작하려면 GitHub [BotFramework 샘플](https://aka.ms/botdispatchgitsamples) 리포지토리에 있는 4개 파일 [home-automation.json](https://aka.ms/dispatch-home-automation-json), [weather.json](https://aka.ms/dispatch-weather-json), [nlp-with-dispatchDispatch.json](https://aka.ms/dispatch-dispatch-json), [QnAMaker.tsv](https://aka.ms/dispatch-qnamaker-tsv)를 로컬 폴더에 다운로드합니다. 이 작업을 수행하는 한 가지 방법은 위의 GitHub 리포지토리 링크를 열고 **BotFramework 샘플**을 클릭한 다음, 리포지토리를 로컬 머신으로 "복제 또는 다운로드"합니다. 이러한 파일은 필수 구성 요소에 언급된 샘플이 아닌 다른 리포지토리에 있습니다.

### <a name="manually-create-luis-apps"></a>수동으로 LUIS 앱 만들기

[LUIS 웹 포털](https://www.luis.ai/)에 로그인합니다. _내 앱_ 섹션에서 _새 앱 가져오기_ 탭을 선택합니다. 다음과 같은 대화 상자가 표시됩니다.

![LUIS json 파일 가져오기](./media/tutorial-dispatch/import-new-luis-app.png)

_앱 파일 선택_ 단추를 선택하고 다운로드한 'home-automation.json' 파일을 선택합니다. 이름 필드(옵션)는 비워 두세요. _완료_를 선택합니다.

LUIS에 Home Automation 앱이 열리면 _학습_ 단추를 선택합니다. 그러면 'home-automation.json' 파일을 사용하여 방금 가져온 발화 세트를 앱이 학습합니다.

학습이 완료되면 _게시_ 단추를 선택합니다. 다음과 같은 대화 상자가 표시됩니다.

![LUIS 앱 게시](./media/tutorial-dispatch/publish-luis-app.png)

'프로덕션' 환경을 선택한 후 _게시_ 단추를 선택합니다.

새 LUIS 앱이 게시되면 _관리_ 탭을 선택합니다. '애플리케이션 정보' 페이지에서 `Application ID` 및 `Display name` 값을 기록합니다. '키 및 엔드포인트' 페이지에서 `Authoring Key` 및 `Region` 값을 기록합니다. 이러한 값은 'nlp-with-dispatch.bot' 파일에서 나중에 사용하게 됩니다.

완료되면 로컬에 다운로드한 'weather.json' 및 'nlp-with-dispatchDispatch.json' 파일 모두에 이러한 단계를 반복하여 LUIS 날씨 앱 및 LUIS 디스패치 앱을 모두 _학습_시키고 _게시_합니다.

### <a name="manually-create-qna-maker-app"></a>수동으로 QnA Maker 앱 만들기

QnA Maker 기술 자료를 설정하는 첫 번째 단계는 Azure에서 QnA Maker 서비스를 설정하는 것입니다. 이렇게 하려면 [여기](https://aka.ms/create-qna-maker)에 나와 있는 단계별 지침을 따르세요. 이제 [QnAMaker 웹 포털](https://qnamaker.ai)에 로그인합니다. 아래의 2단계로 이동합니다.

![QnA 2단계 만들기](./media/tutorial-dispatch/create-qna-step-2.png)

다음을 선택합니다.
1. Azure AD 계정.
1. Azure 구독 이름.
1. QnA Maker 서비스에 대해 만든 이름. (처음에 Azure QnA 서비스가 이 풀다운 목록에 표시되지 않을 경우 페이지를 새로 고쳐 보세요.) 

3단계로 이동합니다.

![QnA 3단계 만들기](./media/tutorial-dispatch/create-qna-step-3.png)

QnA Maker 기술 자료의 이름을 지정합니다. 이 예에서는 'sample-qna'라는 이름을 사용합니다.

4단계로 이동합니다.

![QnA 4단계 만들기](./media/tutorial-dispatch/create-qna-step-4.png)

_+ 파일 추가_ 옵션을 선택하고 다운로드된 파일 'QnAMaker.tsv'를 선택합니다.

기술 자료에 _잡담_ 퍼스낼리티를 추가하는 추가 옵션이 있지만 이 예에서는 이 옵션을 다루지 않습니다.

_저장 및 학습_을 선택하고 완료되면 _게시_ 탭을 선택하고 앱을 게시합니다.

QnA Maker 앱이 게시되면 _설정_ 탭을 선택하고 '배포 세부 정보'가 나올 때까지 아래로 스크롤합니다. _Postman_ 샘플 HTTP 요청의 다음 값을 기록합니다.

```
POST /knowledgebases/<Your_Knowledgebase_Id>/generateAnswer
Host: <Your_Hostname>
Authorization: EndpointKey <Your_Endpoint_Key>
```
이러한 값은 'nlp-with-dispatch.bot' 파일에서 나중에 사용하게 됩니다.

### <a name="manually-update-your-bot-file"></a>수동으로 .bot 파일 업데이트

모든 서비스 앱이 생성되면 각 앱에 대한 정보를 'nlp-with-dispatch.bot' 파일에 추가해야 합니다. 이전에 다운로드한 C# 또는 JS 샘플 파일 내에서 이 파일을 엽니다. "type": "luis" 또는 "type": "dispatch"의 각 섹션에 다음 값을 추가합니다.

```
"appId": "<Your_Recorded_App_Id>",
"authoringKey": "<Your_Recorded_Authoring_Key>",
"subscriptionKey": "<Your_Recorded_Authoring_Key>",
"version": "0.1",
"region": "<Your_Recorded_Region>",
```

"type": "qna" 섹션에 다음 값을 추가합니다.

```
"type": "qna",
"name": "sample-qna",
"id": "201",
"kbId": "<Your_Recorded_Knowledgebase_Id>",
"subscriptionKey": "<Your_Azure_Subscription_Key>", // Used when creating your QnA service.
"endpointKey": "<Your_Recorded_Endpoint_Key>",
"hostname": "<Your_Recorded_Hostname>"
```

모든 변경 사항을 적용했으면 이 파일을 저장하세요.

### <a name="test-your-bot"></a>봇 테스트

이제 에뮬레이터를 사용하여 샘플을 실행합니다. 에뮬레이터가 열리면 'nlp-with-dispatch.bot' 파일을 선택합니다.

참고로, 포함된 서비스에서 다루는 몇 가지 질문과 명령은 다음과 같습니다.

* QnA Maker
  * `hi`, `good morning`
  * `what are you`, `what do you do`
* LUIS(가정 자동화)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS(날씨)
  * `whats the weather in redmond washington`
  * `what's the forecast for london`
  * `show me the forecast for nebraska`

### <a name="connecting-to-the-services-from-your-bot"></a>봇에서 서비스에 연결

봇은 Dispatch, LUIS 및 QnA Maker 서비스에 연결하기 위해 **.bot** 파일에서 정보를 끌어옵니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**, `ConfigureServices`는 구성 파일을 읽고, `InitBotServices`를 해당 정보를 사용하여 서비스를 초기화합니다. 봇은 생성될 때마다 등록된 `BotServices` 개체를 통해 초기화됩니다. 이러한 두 가지 방법의 관련 부분은 다음과 같습니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-dispatch.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // ...
    
    var connectedServices = InitBotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    
    services.AddBot<NlpDispatchBot>(options =>
    {
          
          // The Memory Storage used here is for local bot debugging only. 
          // When the bot is restarted, everything stored in memory will be gone.

          Storage dataStore = new MemoryStorage();

          // ...

          // Create Conversation State object.
          // The Conversation State object is where we persist anything at the conversation-scope.

          var conversationState = new ConversationState(dataStore);
          options.State.Add(conversationState);
     });
}

```
다음 코드는 외부 서비스에 대한 봇의 참조를 초기화합니다. 예를 들어 여기서는 LUIS 및 QnaMaker 서비스가 만들어집니다. 이러한 외부 서비스는 (`.bot` 파일의 내용에 따라) `BotConfiguration` 클래스를 사용하여 구성됩니다.

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    var luisServices = new Dictionary<string, LuisRecognizer>();

    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.Luis:
                {
                    // ...
                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    luisServices.Add(luis.Name, recognizer);
                    break;
                }

            case ServiceTypes.Dispatch:
                // ...
                var dispatchApp = new LuisApplication(dispatch.AppId, dispatch.AuthoringKey, dispatch.GetEndpoint());

                // Since the Dispatch tool generates a LUIS model, we use the LuisRecognizer to resolve the
                // dispatching of the incoming utterance.
                var dispatchARecognizer = new LuisRecognizer(dispatchApp);
                luisServices.Add(dispatch.Name, dispatchARecognizer);
                break;

            case ServiceTypes.QnA:
                {
                    // ...
                    var qnaEndpoint = new QnAMakerEndpoint()
                    {
                        KnowledgeBaseId = qna.KbId,
                        EndpointKey = qna.EndpointKey,
                        Host = qna.Hostname,
                    };

                    var qnaMaker = new QnAMaker(qnaEndpoint);
                    qnaServices.Add(qna.Name, qnaMaker);
                    break;
                }
        }
    }

    return new BotServices(qnaServices, luisServices);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

샘플 코드는 미리 정의된 명명 상수를 사용하여 `.bot` 파일의 다양한 섹션을 확인합니다. _nlp-with-dispatch.bot_ 파일에서 원래 샘플 명명 방식으로 섹션 이름을 수정한 경우 **bot.js**, **homeAutomation.js**, **qna.js** 또는 **weather.js** 파일에서 관련 상수 선언을 찾아 해당 항목을 수정된 이름으로 변경하세요.  

```javascript
// In file bot.js
// this is the LUIS service type entry in the .bot file.
const DISPATCH_CONFIG = 'nlp-with-dispatchDispatch';

// In file homeAutomation.js
// this is the LUIS service type entry in the .bot file.
const LUIS_CONFIGURATION = 'Home Automation';

// In file qna.js
// Name of the QnA Maker service in the .bot file.
const QNA_CONFIGURATION = 'sample-qna';

// In file weather.js
// this is the LUIS service type entry in the .bot file.
const WEATHER_LUIS_CONFIGURATION = 'Weather';
```

**bot.js**에서 구성 파일 _nlp-with-dispatch.bot_에 포함된 정보는 디스패치 봇을 다양한 서비스에 연결하는 데 사용됩니다. 각 생성자는 위에 자세히 설명된 섹션 이름에 따라 구성 파일의 적절한 섹션을 찾아 사용합니다.

```javascript
class DispatchBot {
    constructor(conversationState, userState, botConfig) {
        //...
        this.homeAutomationDialog = new HomeAutomation(conversationState, userState, botConfig);
        this.weatherDialog = new Weather(botConfig);
        this.qnaDialog = new QnA(botConfig);

        this.conversationState = conversationState;
        this.userState = userState;

        // dispatch recognizer
        const dispatchConfig = botConfig.findServiceByNameOrId(DISPATCH_CONFIG);
        //...
```
---

### <a name="calling-the-services-from-your-bot"></a>봇에서 서비스 호출

봇 논리는 결합된 Dispatch 모델에 대한 사용자 입력을 확인합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**NlpDispatchBot.cs** 파일에서 봇의 생성자는 시작 시 등록된 `BotServices` 개체를 가져옵니다.

```csharp
private readonly BotServices _services;

public NlpDispatchBot(BotServices services)
{
    _services = services ?? throw new System.ArgumentNullException(nameof(services));

    //...
}
```

봇의 `OnTurnAsync` 메서드에서 Dispatch 모델에 들어오는 사용자의 메시지를 확인합니다.

```csharp
// Get the intent recognition result
var recognizerResult = await _services.LuisServices[DispatchKey].RecognizeAsync(context, cancellationToken);
var topIntent = recognizerResult?.GetTopScoringIntent();

if (topIntent == null)
{
    await context.SendActivityAsync("Unable to get the top intent.");
}
else
{
    await DispatchToTopIntentAsync(context, topIntent, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bot.js** `onTurn` 메서드에서 사용자로부터 수신되는 메시지를 확인합니다. _ActivityType.Message_ 유형이 수신될 경우 이 메시지는 봇의 _dispatchRecognizer_를 통해 전송됩니다.

```javascript
if (turnContext.activity.type === ActivityTypes.Message) {
    // determine which dialog should fulfill this request
    // call the dispatch LUIS model to get results.
    const dispatchResults = await this.dispatchRecognizer.recognize(turnContext);
    const dispatchTopIntent = LuisRecognizer.topIntent(dispatchResults);
    //...
 }
```
---

### <a name="working-with-the-recognition-results"></a>인식 결과 작업

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

모델에서 생성하는 결과에 발언을 가장 적절하게 처리할 수 있는 서비스가 나타납니다. 이 봇의 코드는 해당 서비스에 요청을 라우팅한 후 호출된 서비스에의 응답을 요약합니다.

```csharp
// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
private async Task DispatchToTopIntentAsync(
    ITurnContext context,
    (string intent, double score)? topIntent,
    CancellationToken cancellationToken = default(CancellationToken))
{
    const string homeAutomationDispatchKey = "l_Home_Automation";
    const string weatherDispatchKey = "l_Weather";
    const string noneDispatchKey = "None";
    const string qnaDispatchKey = "q_sample-qna";

    switch (topIntent.Value.intent)
    {
        case homeAutomationDispatchKey:
            await DispatchToLuisModelAsync(context, HomeAutomationLuisKey);

            // Here, you can add code for calling the hypothetical home automation service, passing in any entity
            // information that you need.
            break;
        case weatherDispatchKey:
            await DispatchToLuisModelAsync(context, WeatherLuisKey);

            // Here, you can add code for calling the hypothetical weather service,
            // passing in any entity information that you need
            break;
        case noneDispatchKey:
            // You can provide logic here to handle the known None intent (none of the above).
            // In this example we fall through to the QnA intent.
        case qnaDispatchKey:
            await DispatchToQnAMakerAsync(context, QnAMakerKey);
            break;

        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivityAsync($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
            break;
    }
}

// Dispatches the turn to the request QnAMaker app.
private async Task DispatchToQnAMakerAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await _services.QnAServices[appName].GetAnswersAsync(context).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivityAsync(results.First().Answer, cancellationToken: cancellationToken);
        }
        else
        {
            await context.SendActivityAsync($"Couldn't find an answer in the {appName}.");
        }
    }
}


// Dispatches the turn to the requested LUIS model.
private async Task DispatchToLuisModelAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await context.SendActivityAsync($"Sending your request to the {appName} system ...");
    var result = await _services.LuisServices[appName].RecognizeAsync(context, cancellationToken);

    await context.SendActivityAsync($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", result.Intents)}");

    if (result.Entities.Count > 0)
    {
        await context.SendActivityAsync($"The following entities were found in the message:\n\n{string.Join("\n\n", result.Entities)}");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

모델에서 생성하는 결과에 발언을 가장 적절하게 처리할 수 있는 서비스가 나타납니다. 이 봇의 코드는 해당 서비스에 요청을 라우팅합니다.

```javascript
switch (dispatchTopIntent) {
   case HOME_AUTOMATION_INTENT:
      await this.homeAutomationDialog.onTurn(turnContext);
      break;
   case WEATHER_INTENT:
      await this.weatherDialog.onTurn(turnContext);
      break;
   case QNA_INTENT:
      await this.qnaDialog.onTurn(turnContext);
      break;
   case NONE_INTENT:
      default:
      // Unknown request
       await turnContext.sendActivity(`I do not understand that.`);
       await turnContext.sendActivity(`I can help with weather forecast, turning devices on and off and answer general questions like 'hi', 'who are you' etc.`);
 }
 ```

 위치: `homeAutomation.js`
 
 ```javascript
 async onTurn(turnContext) {
    // make call to LUIS recognizer to get home automation intent + entities
    const homeAutoResults = await this.luisRecognizer.recognize(turnContext);
    const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
    // depending on intent, call turn on or turn off or return unknown
    switch (topHomeAutoIntent) {
       case HOME_AUTOMATION_INTENT:
          await this.handleDeviceUpdate(homeAutoResults, turnContext);
          break;
       case NONE_INTENT:
       default:
         await turnContext.sendActivity(`HomeAutomation dialog cannot fulfill this request.`);
    }
}
```

위치: `weather.js`

```javascript
async onTurn(turnContext) {
   // Call weather LUIS model.
   const weatherResults = await this.luisRecognizer.recognize(turnContext);
   const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
   // Get location entity if available.
   const locationEntity = (LOCATION_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_ENTITY][0] : undefined;
   const locationPatternAnyEntity = (LOCATION_PATTERNANY_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_PATTERNANY_ENTITY][0] : undefined;
   // Depending on intent, call "Turn On" or "Turn Off" or return unknown.
   switch (topWeatherIntent) {
      case GET_CONDITION_INTENT:
         await turnContext.sendActivity(`You asked for current weather condition in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case GET_FORECAST_INTENT:
         await turnContext.sendActivity(`You asked for weather forecast in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case NONE_INTENT:
      default:
         wait turnContext.sendActivity(`Weather dialog cannot fulfill this request.`);
   }
}
```

위치: `qna.js`

```javascript
async onTurn(turnContext) {
   // Call QnA Maker and get results.
   const qnaResult = await this.qnaRecognizer.generateAnswer(turnContext.activity.text, QNA_TOP_N, QNA_CONFIDENCE_THRESHOLD);
   if (!qnaResult || qnaResult.length === 0 || !qnaResult[0].answer) {
       await turnContext.sendActivity(`No answer found in QnA Maker KB.`);
       return;
    }
    // respond with qna result
    await turnContext.sendActivity(qnaResult[0].answer);
}
```
---

## <a name="edit-intents-to-improve-performance"></a>성능 향상을 위해 의도 편집

봇이 실행되면 유사하거나 겹치는 발화를 제거하여 봇의 성능을 개선할 수 있습니다. 예를 들어 `Home Automation` LUIS 앱에서 QnA maker에 전달될 수 있도록 "TurnOnLights" 의도에 매핑하는 "내 전등 켜기"와 같은 요청이지만 "None" 의도에 매핑하는 "내 전등이 켜지지 않는 이유는?"과 같은 요청을 가정해 봅시다. 디스패치를 사용하여 LUIS 앱 및 QnA Maker 서비스를 결합할 때 다음 중 하나를 수행해야 합니다.

* 원래 `Home Automation` LUIS 앱에서 "None" 의도를 제거하고 해당 의도의 발언을 디스패처 앱의 "None" 의도에 추가합니다.
* 원래 LUIS 앱에서 "None" 의도를 제거하지 않는 경우 "None" 의도를 QnA maker 서비스에 일치시키는 메시지를 전달하도록 봇에 논리를 추가해야 합니다.

위의 두 동작 중 하나로 봇이 '답변을 찾을 수 없습니다.'라는 메시지로 사용자에게 응답하는 횟수를 줄일 수 있습니다. 

## <a name="additional-resources"></a>추가 리소스

**LUIS 모델 업데이트 또는 새로 만들기:** 이 샘플은 미리 구성된 LUIS 모델을 기반으로 합니다. 이 모델을 업데이트하거나 새 LUIS 모델을 만드는 데 도움이 되는 추가 정보는 [여기](https://aka.ms/create-luis-model#updating-your-cognitive-models
)서 찾을 수 있습니다.

**리소스 삭제:** 이 샘플에서는 아래에 나열된 단계를 사용하여 삭제할 수 있는 다양한 애플리케이션과 리소스를 만들지만 *다른 애플리케이션 또는 서비스*에서 사용하는 리소스는 삭제하면 안 됩니다. 

_LUIS 리소스_
1. [luis.ai](https://www.luis.ai) 포털에 로그인합니다.
1. _내 앱_ 페이지로 이동합니다.
1. 이 샘플로 만든 앱을 선택합니다.
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. _삭제_를 클릭하고 _확인_을 클릭하여 확인합니다.

_QnA Maker 리소스_
1. [qnamaker.ai](https://www.qnamaker.ai/) 포털에 로그인합니다.
1. _내 기술 자료_ 페이지로 이동합니다.
1. `Sample QnA` 기술 자료의 삭제 단추를 클릭하고 _삭제_를 클릭하여 확인합니다.

**모범 사례:** 이 샘플에서 사용된 서비스를 향상시키려면 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) 및 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices)에 대한 모범 사례를 참조하세요.