---
title: LUIS 및 QnA Maker에 대해 디스패치 도구 사용 | Microsoft Docs
description: 봇에서 LUIS 및 QnA Maker를 사용하는 방법에 대해 알아봅니다.
keywords: Luis, QnA, 디스패치 도구, 여러 서비스
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6294021355f82ff53a2ea99db4fb19f44cf13029
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/19/2018
ms.locfileid: "39304643"
---
## <a name="integrate-multiple-luis-apps-and-qna-services-with-the-dispatch-tool"></a>디스패치 도구를 사용하여 여러 LUIS 앱 및 QnA 서비스 통합

이 자습서에는 디스패치 도구에서 생성된 LUIS 모델을 사용하여 봇을 여러 LUIS(Language Understanding) 앱 및 QnAMaker 서비스와 통합하는 방법을 보여줍니다. 

다음 서비스를 개발했고, 모두와 통합하는 봇을 만들려 한다고 가정합니다.

| 서비스 유형 | Name | 설명 |
|------|------|------|
| LUIS 앱 | HomeAutomation | HomeAutomation.TurnOn, HomeAutomation.TurnOff 및 HomeAutomation.None 의도를 인식합니다.|
| LUIS 앱 | Weather | Weather.GetForecast 및 Weather.GetCondition 의도를 인식합니다.|
| QnAMaker 서비스 | FAQ  | 가정 자동화 조명 시스템에 대한 질문에 답변을 제공합니다. |

먼저 앱 및 서비스를 만든 다음, 함께 통합해 보겠습니다.

> [!NOTE]
> 디스패치가 성공적으로 액세스하도록 동일한 Azure 위치 내에서 모든 세 개의 앱을 만들어야 합니다. 아래 예제 디스패치 코드에서는 미국 서부 예제 위치를 사용합니다.

## <a name="create-the-luis-apps"></a>LUIS 앱 만들기

HomeAutomation 및 날씨 LUIS 앱을 만드는 가장 빠른 방법은 [homeautomation.json][HomeAutomationJSON] 및 [weather.json][WeatherJSON] 파일을 다운로드하는 것입니다. 그런 다음, [LUIS 웹 사이트](https://www.luis.ai/home)로 이동하고 로그인합니다. **내 앱** > **새 앱 가져오기**를 클릭하고 homeautomation.json 파일을 선택합니다. 새 앱의 이름을 `homeautomation`으로 지정합니다. **내 앱** > **새 앱 가져오기**를 클릭하고 weather.json 파일을 선택합니다. 이 다른 새 앱의 이름을 `weather`로 지정합니다.

## <a name="create-the-qna-cognitive-service-in-azure"></a>Azure에서 QnA Cognitive Service 만들기

QnA Maker 서비스는 두 부분, Azure의 Cognitive Service와 Cognitive Service를 사용하여 게시하는 질문 및 답변 쌍의 기술 자료를 포함합니다.

Azure에서 Cognitive Service를 만들려면 https://portal.azure.com에서 Azure Portal에 로그인하고, 다음을 수행합니다.

1. **모든 서비스**를 클릭합니다.
1. `Cognitive`를 검색하고 **Cognitive Services**를 선택합니다.
1. **추가**를 클릭합니다.
1. `QnA`를 검색하고 **QnA Maker**를 선택합니다.
1. QnA Maker 블레이드에서 **만들기**를 클릭합니다.
1. 정보를 입력하고 QnA Maker 서비스를 만듭니다.

    <!-- TODO: Add screenshot.-->

    * 서비스의 이름을 입력합니다. 이 자습서에서는 `SmartLightQnA`를 사용합니다.
    * 사용할 구독을 선택합니다.
    * 사용할 관리 가격 책정 계층을 선택합니다. F0(무료) 계층을 사용합니다.
    * 하나를 만들거나 사용할 새 리소스 그룹을 선택합니다. 이 자습서에서는 새 `SmartLightQnA` 리소스 그룹을 만듭니다.
    * 검색 가격 책정 계층을 선택합니다. B(기본) 계층을 사용합니다.
    * 검색 위치를 선택합니다. `West US`를 사용합니다.
    * 사용할 응용 프로그램 이름을 입력합니다. 기본값 `SmartLightQnA`를 유지합니다.
    * 웹 사이트 위치를 선택합니다. `West US`를 사용합니다.
    * 앱 인사이트를 활성화하는 기본값을 유지합니다.
    * 앱 인사이트 위치를 선택합니다. `West US 2`를 사용합니다.
    * **만들기**를 클릭하여 QnA Maker 서비스를 만듭니다.
    * Azure에서 서비스를 만들고 배포를 시작합니다.

1. 서비스가 배포되면 알림을 확인하고 **리소스로 이동**을 클릭하여 서비스에 대한 블레이드로 이동합니다.
1. **키**를 클릭하여 키를 가져옵니다.

    * 서비스의 이름 및 첫 번째 키를 복사합니다. 다음 단계에서 필요합니다.
    * 서비스를 중단하지 않고 한 번에 둘 중 하나를 다시 생성할 수 있도록 두 개의 키를 가져옵니다.

## <a name="create-and-publish-the-qna-maker-knowledge-base"></a>QnA Maker 기술 자료 만들기 및 게시

[QnA Maker 웹 사이트](https://qnamaker.ai)로 이동하고 로그인합니다. **기술 자료 만들기**를 선택하고 "FAQ"라는 새 기술 자료를 만듭니다. **파일 선택** 단추를 클릭하고 [샘플 TSV 파일][FAQ_TSV]을 업로드합니다. **만들기**를 클릭하고, 서비스가 만들어지면 **게시**를 클릭합니다.

## <a name="use-the-dispatch-tool-to-create-the-dispatcher-luis-app"></a>디스패치 도구를 사용하여 디스패처 LUIS 앱 만들기

다음으로 LUIS 앱을 만들어 만든 각 서비스를 결합해 보겠습니다.

Node.js 명령 프롬프트에서 이 명령을 실행하여 [디스패치 도구][DispatchTool]를 설치합니다.

```
npm install -g botdispatch
```

다음 명령을 실행하여 `CombineWeatherAndLights`라는 이름의 디스패치 도구를 초기화합니다. `"YOUR-LUIS-AUTHORING-KEY"`에 대한 [LUIS 작성 키](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-concept-keys)를 대체합니다.

```
dispatch init -name CombineWeatherAndLights -luisAuthoringKey "YOUR-LUIS-AUTHORING-KEY" -luisAuthoringRegion westus
```

사용자가 만든 각 LUIS 앱에 대해 LUIS 앱 ID를 가져옵니다. 각 앱에 대해 [LUIS 사이트](https://www.luis.ai/home)의 **내 앱** 아래에서 찾을 수 있습니다. 앱 이름을 클릭한 다음, **설정**을 클릭하여 응용 프로그램 ID를 확인합니다. 

그런 다음, 사용자가 만든 각 LUIS 앱에 대해 `dispatch add` 명령을 실행합니다.

```
dispatch add -type luis -id "HOMEAUTOMATION-APP-ID" -name homeautomation -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"
dispatch add -type luis -id "WEATHER-APP-ID" -name weather -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"

```

사용자가 만든 QnA Maker 서비스에 대해 `dispatch add` 명령을 실행합니다. `-key` 매개 변수는 [Azure에서 QnA Cognitive Service 만들기](./bot-builder-tutorial-dispatch.md#create-the-qna-cognitive-service-in-azure)에 대한 단계에서 완료했을 때 저장한 Azure Portal의 키여야 합니다.

```
dispatch add -type qna -id "QNA-KB-ID" -name faq -key "YOUR-QNA-SUBSCRIPTION-KEY"
```

`dispatch create`를 실행합니다.

```
dispatch create
```

**CombineWeatherAndLights**라는 디스패처 LUIS 앱을 만듭니다. [https://www.luis.ai/home](https://www.luis.ai/home)에서 새 앱을 볼 수 있습니다. 

![LUIS.ai의 디스패처 앱](media/tutorial-dispatch/dispatch-app-in-luis.png)

새 앱을 클릭합니다. **의도** 아래에서 `l_homeautomation`, `l_weather` 및 `q_faq` 의도를 가진 것을 확인할 수 있습니다.

![LUIS.ai의 디스패처 의도](media/tutorial-dispatch/dispatch-intents-in-luis.png)

**학습** 단추를 클릭하여 LUIS 앱을 학습하고, **게시** 탭을 사용하여 [게시](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)합니다. **설정**을 클릭하여 봇에서 사용할 새 앱의 ID를 복사합니다.

## <a name="create-the-bot"></a>봇 만들기

이제 디스패처 앱의 의도를 원래 LUIS 앱 및 QnAMaker 서비스에 메시지를 라우팅하는 봇의 논리로 연결할 수 있습니다.

시작점으로 Bot Builder SDK에 포함된 샘플을 사용할 수 있습니다.

# <a name="ctabcsaddref"></a>[C#](#tab/csaddref)

[LUIS 디스패치 샘플][DispatchBotCS]의 코드를 사용하여 시작합니다. Visual Studio에서 다음의 최신 시험판 버전으로 [Nuget 패키지를 업데이트](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package)합니다.

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai.QnA`(QnA Maker에 필수)
* `Microsoft.Bot.Builder.Ai.Luis`(LUIS에 필수)

# <a name="javascripttabjsaddref"></a>[JavaScript](#tab/jsaddref)

[LUIS 디스패치 샘플][DispatchBotJs]을 다운로드합니다.  npm을 사용하여 LUIS 및 QnA Maker에 대한 `botbuilder-ai` 패키지를 포함하여 필요한 패키지를 설치합니다.

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


디스패처 앱을 사용하도록 샘플을 설정합니다.

# <a name="ctabcsbotconfig"></a>[C#](#tab/csbotconfig)

[LUIS 디스패치 샘플][DispatchBotCS]의 **appsettings.json**에서 다음 필드를 편집합니다.

| Name | 설명 |
|------|------|
| `Luis-SubscriptionKey` |  LUIS 구독 키입니다. [여기](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-keys)에 설명된 엔드포인트 키 또는 작성 키일 수 있습니다. | 
| `Luis-ModelId-Dispatcher` | 디스패치 도구에서 생성하는 LUIS 앱에 대한 앱 ID입니다. | 
| `Luis-ModelId-HomeAutomation` | homeautomation.json에서 만든 앱의 앱 ID  | 
| `Luis-ModelId-Weather` | weather.json에서 만든 앱의 앱 ID | 
| `QnAMaker-Endpoint-Url` | 미리 보기 QnA Maker 서비스에 대해 https://westus.api.cognitive.microsoft.com/qnamaker/v2.0으로 설정되어야 합니다. <br/>새 (GA) QnA Maker 서비스에 대해 https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker로 설정합니다.|
| `QnAMaker-SubscriptionKey` | QnA Maker 구독 키입니다. | 
| `QnAMaker-KnowledgeBaseId` | [QnAMaker 포털](https://qnamaker.ai)에서 만드는 기술 자료의 ID입니다.| 



**Startup.cs**에서 `ConfigureServices` 메서드를 살펴봅니다. 방금 생성한 LUIS 앱의 앱 ID를 사용하여 `LuisRecognizerMiddleware`를 초기화하는 코드를 포함합니다.

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(this.Configuration);
    services.AddBot<LuisDispatchBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var (luisModelId, luisSubscriptionKey, luisUri) = GetLuisConfiguration(this.Configuration, "Dispatcher");

        var luisModel = new LuisModel(luisModelId, luisSubscriptionKey, luisUri);

        // If you want to get all the intents and scores that LUIS recognized, set Verbose = true in luisOptions
        var luisOptions = new LuisRequest { Verbose = true };

        var middleware = options.Middleware;
        middleware.Add(new LuisRecognizerMiddleware(luisModel, luisOptions: luisOptions));
    });
}
```
`GetLuisConfiguration` 및 `GetQnAMakerConfiguration` 메서드는 appsettings.json에서 LUIS 및 QnA 구성을 가져옵니다.
```csharp
public static (string modelId, string subscriptionId, Uri uri) GetLuisConfiguration(IConfiguration configuration, string serviceName)
{
    var modelId = configuration.GetSection($"Luis-ModelId-{serviceName}")?.Value;
    var subscriptionId = configuration.GetSection("Luis-SubscriptionKey")?.Value;
    var uri = new Uri(configuration.GetSection("Luis-Url")?.Value);
    return (modelId, subscriptionId, uri);
}

public static (string knowledgeBaseId, string subscriptionKey, string uri) GetQnAMakerConfiguration(IConfiguration configuration)
{
    var knowledgeBaseId = configuration.GetSection("QnAMaker-KnowledgeBaseId")?.Value;
    var subscriptionKey = configuration.GetSection("QnAMaker-SubscriptionKey")?.Value;
    var uri = configuration.GetSection("QnAMaker-Endpoint-Url")?.Value;
    return (knowledgeBaseId, subscriptionKey, uri);
}
```

### <a name="dispatch-the-message"></a>메시지 디스패치

봇이 하위 구성 요소에 대한 LUIS 앱 및 QnA Maker에 메시지를 디스패치하는 **LuisDispatchBot.cs**를 살펴봅니다. 

`DispatchToTopIntent`에서 디스패처 앱이 `l_homeautomation` 또는 `l_weather` 의도를 검색하는 경우 `LuisRecognizer`를 만들어 원래 `homeautomation` 및 `weather` 앱을 호출하는 `DispatchToTopIntent` 메서드를 호출합니다. 봇이 `q_faq` 의도 또는 대체 사례로 사용되는 `none` 의도를 검색하는 경우 QnAMaker를 쿼리하는 메서드를 호출합니다.

> [!NOTE] 
> 의도 이름 `l_homeautomation`, `l_weather` 또는 `q_faq`가 디스패치를 사용하여 만든 LUIS 앱과 일치하지 않는 경우 [LUIS 포털](https://www.luis.ai)에 표시되는 의도 이름의 소문자 버전과 일치하도록 편집합니다.

```csharp
private async Task DispatchToTopIntent(ITurnContext context, (string intent, double score)? topIntent)
{
    switch (topIntent.Value.intent.ToLowerInvariant())
    {
        case "l_homeautomation":
            await DispatchToLuisModel(context, this.luisModelHomeAutomation, "home automation");
            break;
        case "l_weather":
            await DispatchToLuisModel(context, this.luisModelWeather, "weather");
            break;
        case "none":
        // You can provide logic here to handle the known None intent (none of the above).
        // In this example we fall through to the QnA intent.
        case "q_faq":
            await DispatchToQnAMaker(context, this.qnaEndpoint, "FAQ");
            break;
        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivity($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");

            break;
    }
}
```

`DispatchToQnAMaker` 메서드는 QnA Maker 서비스에 사용자의 메시지를 보냅니다. 봇을 실행하기 전에 [QnA Maker 포털](https://qnamaker.ai)에 해당 서비스를 게시했는지 확인합니다.

```csharp
private static async Task DispatchToQnAMaker(ITurnContext context, QnAMakerEndpoint qnaOptions, string appName)
{
    QnAMaker qnaMaker = new QnAMaker(qnaOptions);
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await qnaMaker.GetAnswers(context.Activity.Text.Trim()).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivity(results.First().Answer);
        }
        else
        {
            await context.SendActivity($"Couldn't find an answer in the {appName}.");
        }
    }
}
```

`DispatchToLuisModel` 메서드는 원래 `homeautomation` 및 `weather` LUIS 앱에 사용자의 메시지를 보냅니다. 봇을 실행하기 전에 [LUIS 포털](https://www.luis.ai)에 해당 LUIS 앱을 게시했는지 확인합니다.

```csharp
private static async Task DispatchToLuisModel(ITurnContext context, LuisModel luisModel, string appName)
{
    await context.SendActivity($"Sending your request to the {appName} system ...");
    var (intents, entities) = await RecognizeAsync(luisModel, context.Activity.Text);

    await context.SendActivity($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", intents)}");

    if (entities.Count() > 0)
    {
        await context.SendActivity($"The following entities were found in the message:\n\n{string.Join("\n\n", entities)}");
    }
    
    // Here, you can add code for calling the hypothetical home automation or weather service, 
    // passing in the appName and any intent or entity information that you need 
}
```

`RecognizeAsync` 메서드는 `LuisRecognizer`를 호출하여 LUIS 앱에서 결과를 가져옵니다.

```cs
private static async Task<(IEnumerable<string> intents, IEnumerable<string> entities)> RecognizeAsync(LuisModel luisModel, string text)
{
    var luisRecognizer = new LuisRecognizer(luisModel);
    var recognizerResult = await luisRecognizer.Recognize(text, System.Threading.CancellationToken.None);

    // list the intents
    var intents = new List<string>();
    foreach (var intent in recognizerResult.Intents)
    {
        intents.Add($"'{intent.Key}', score {intent.Value}");
    }

    // list the entities
    var entities = new List<string>();
    foreach (var entity in recognizerResult.Entities)
    {
        if (!entity.Key.ToString().Equals("$instance"))
        {
            entities.Add($"{entity.Key}: {entity.Value.First}");
        }
    }

    return (intents, entities);
}
```

# <a name="javascripttabjsbotconfig"></a>[JavaScript](#tab/jsbotconfig)

[디스패치 봇 샘플][DispatchBotJs]의 코드를 사용하여 시작합니다. **app.js**를 열고 필요에 따라 `appId` 필드를 사용자가 만든 LUIS 앱의 ID와 바꿉니다. 원래대로 `appId` 필드를 두는 경우 데모용으로 만든 공용 LUIS 앱을 사용하게 됩니다.

```javascript
// Create LuisRecognizers and QnAMaker
// The LUIS applications are public, meaning you can use your own subscription key to test the applications.
// For QnAMaker, users are required to create their own knowledge base.
// The exported LUIS applications and QnAMaker knowledge base can be found with the sample bot.

// The corresponding LUIS application JSON is `dispatchSample.json`
const dispatcher = new LuisRecognizer({
    appId: '0b18ab4f-5c3d-4724-8b0b-191015b48ea9',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `homeautomation.json`
const homeAutomation = new LuisRecognizer({
    appId: 'c6d161a5-e3e5-4982-8726-3ecec9b4ed8d',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `weather.json`
const weather = new LuisRecognizer({
    appId: '9d0c9e9d-ce04-4257-a08a-a612955f2fb5',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});
```

다음 코드에서 `endpointKey` 및 `knowledgeBaseID`를 QnAMaker의 키 및 기술 자료 ID로 바꿉니다. `host`는 미리 보기 QnA Maker 서비스에 대해 `https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker`으로, 새 (GA) QnA Maker 서비스에 대해 `https://westus.api.cognitive.microsoft.com/qnamaker/v2.0`로 설정되어야 합니다.

```javascript
const faq = new QnAMaker(
    {
        knowledgeBaseId: '',
        endpointKey: '',
        host: ''
    },
    {
        answerBeforeNext: true
    }
);

```

**app.js**에서 코드의 나머지는 적절한 대화 상자를 시작하여 `l_homeautomation`, `l_weather` 및 `None` 의도를 처리합니다. `q_faq` 의도의 경우 QnAMaker 서비스의 응답으로 회신합니다.

> [!NOTE] 
> 의도 이름 `l_homeautomation`, `l_weather` 또는 `q_faq`가 디스패치를 사용하여 만든 LUIS 앱과 일치하지 않는 경우 [LUIS 포털](https://www.luis.ai)에 표시되는 의도 이름과 일치하도록 편집합니다.

```javascript
// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the LUIS apps that are being dispatched to
const dialogs = new DialogSet();

function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach((entity, idx) => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}

dialogs.add('HomeAutomation_TurnOff', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOff = state.homeAutomationTurnOff ? state.homeAutomationTurnOff + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOff}: You reached the "HomeAutomation.TurnOff" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation.TurnOn" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetForecast', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather.GetForecast" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetCondition', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetCondition = state.weatherGetCondition ? state.weatherGetCondition + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetCondition}: You reached the "Weather.GetCondition" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        state.noneIntent = state.noneIntent ? state.noneIntent + 1 : 1;
        await dialogContext.context.sendActivity(`${state.noneIntent}: You reached the "None" dialog.`);
        await dialogContext.end();
    }
]);

adapter.use(dispatcher);

// Listen for incoming Activities
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our dispatcher LUIS application
            const luisResults = dispatcher.get(context);

            // Extract the top intent from LUIS and use it to select which LUIS application to dispatch to
            const topIntent = LuisRecognizer.topIntent(luisResults);

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'l_homeautomation':
                        const homeAutoResults = await homeAutomation.recognize(context);
                        const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
                        await dc.begin(topHomeAutoIntent, homeAutoResults);
                        break;
                    case 'l_weather':
                        const weatherResults = await weather.recognize(context);
                        const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
                        await dc.begin(topWeatherIntent, weatherResults);
                        break;
                    case 'q_faq':
                        await faq.answer(context);
                        break;
                    default:
                        await dc.begin('None');
                }
            }

            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dispatch bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});

```

---
## <a name="run-the-bot"></a>봇 실행

[Bot Framework Emulator](../bot-service-debug-emulator.md)를 사용하여 봇을 테스트합니다. "전등 켜기"와 같은 메시지를 보내 홈 자동화 LUIS 앱 에 메시지를 디스패치하고, "시애틀의 날씨 가져오기"와 같은 메시지를 보내 날씨 LUIS 앱에 디스패치합니다.

> [!NOTE] 
> 봇을 실행하기 전에 [LUIS 포털](https://www.luis.ai)에서 만든 모든 LUIS 앱을 게시했는지 확인하고, [QnA Maker 포털](https://qnamaker.ai)에서 QnA Maker 서비스를 게시했는지 확인합니다.

![디스패치 봇에 메시지 보내기](media/tutorial-dispatch/run-dispatch-bot.png)

## <a name="evaluate-the-dispatchers-performance"></a>디스패처의 성능 평가

경우에 따라 LUIS 앱과 QnA maker 서비스의 예로 제공되는 사용자 메시지가 있고, 디스패치가 생성하는 결합된 LUIS 앱이 해당 입력에 대해 잘 수행하지 않습니다. `eval` 옵션을 사용하여 앱의 성능을 확인합니다. 

```
dispatch eval
```

`dispatch eval`을 실행하면 새 언어 모델의 성능에 대한 통계를 제공하는 **Summary.html** 파일을 생성합니다.

> [!TIP] 
> 디스패치 도구에서 만든 LUIS 앱 뿐만 아닌 모든 LUIS 앱에서 `dispatch eval`을 실제로 수행할 수 있습니다.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>중복 및 겹침에 대해 의도 편집

**Summary.html**에 중복으로 플래그가 지정된 예제 발언을 검토하고, 비슷하거나 겹치는 예제를 제거합니다. 예를 들어 homeautomation에 대한 `homeautomation` LUIS 앱에서 QnA maker에 전달될 수 있도록 "TurnOnLights" 의도에 매핑하는 "내 전등 켜기"와 같은 요청이지만 "None" 의도에 매핑하는 "내 전등이 켜지지 않는 이유는?"과 같은 요청을 가정해 봅시다. 디스패치를 사용하여 LUIS 앱 및 QnA maker 서비스를 결합할 때 다음 중 하나를 수행해야 합니다. 

* 원래 `homeautomation` LUIS 앱에서 "None" 의도를 제거하고 해당 의도의 발언을 디스패처 앱의 "None" 의도에 추가합니다.
* 원래 LUIS 앱에서 "None" 의도를 제거하지 않는 경우 해당 의도를 QnA maker 서비스에 일치시키는 해당 메시지를 전달하도록 봇에 논리를 추가해야 합니다.

> [!TIP] 
> 언어 모델의 성능 향상에 대한 팁은 [Language Understanding의 모범 사례](./bot-builder-concept-luis.md#best-practices-for-language-understanding)를 검토하세요.

## <a name="additional-resources"></a>추가 리소스

* [대화 흐름을 위해 LUIS 사용][luis-v4-how-to]

[luis-v4-how-to]: bot-builder-howto-v4-luis.md
<!-- links -->

[HomeAutomationJSON]: https://aka.ms/dispatch-luis1
[WeatherJSON]: https://aka.ms/dispatch-luis2
[DispatchJSON]: https://aka.ms/dispatch-luis
[FAQ_TSV]: https://aka.ms/dispatch-qna-tsv

[DispatchTool]: https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch
[DispatchBotCS]: https://aka.ms/dispatch-sample-cs
[DispatchBotJs]: https://aka.ms/dispatch-sample-js



