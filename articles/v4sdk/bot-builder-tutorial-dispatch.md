---
title: 디스패치 도구와 함께 LUIS 앱 및 QnA 서비스 사용 | Microsoft Docs
description: 봇에서 LUIS 및 QnA Maker를 사용하는 방법에 대해 알아봅니다.
keywords: Luis, QnA, 디스패치 도구, 여러 서비스
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/31/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4d029dc7361ac8a7fadb61141faf60d8a62eab3c
ms.sourcegitcommit: a496714fb72550a743d738702f4f79e254c69d06
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/01/2018
ms.locfileid: "50736681"
---
# <a name="use-luis-and-qna-services-with-the-dispatch-tool"></a>디스패치 도구와 함께 LUIS 앱 및 QnA 서비스 사용

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

<!--TODO: Add the JS sections back in and update them once the JS sample is working.-->

이 자습서에는 디스패치 도구에서 생성된 LUIS 모델을 사용하여 봇을 여러 LUIS(Language Understanding) 앱 및 QnAMaker 서비스와 통합하는 방법을 보여줍니다. 이 샘플에서는 다음 서비스를 결합합니다.

| 서비스 유형 | 이름 | 설명 |
|------|------|------|
| LUIS 앱 | HomeAutomation | 연결된 엔터티 데이터를 사용하여 가정 자동화 의도를 인식합니다.|
| LUIS 앱 | Weather | 위치 데이터를 사용하여 Weather.GetForecast 및 Weather.GetCondition 의도를 인식합니다.|
| QnAMaker 서비스 | FAQ  | 봇에 대한 몇 가지 간단한 질문에 답변을 제공합니다. |

이 문서의 코드는**Dispatch를 사용하는 NLP** 샘플[[C#](https://aka.ms/dispatch-sample-cs)]에서 가져온 것입니다.

<!-- | [JS](https://aka.ms/dispatch-sample-js)-->

언어 서비스에 대한 개요는 [언어 이해](bot-builder-concept-luis.md)를 참조하세요. 이를 봇에 구현하는 방법은 [LUIS](bot-builder-howto-v4-luis.md) 및 [QnA Maker](bot-builder-howto-qna.md)에 대한 방법 문서를 참조하세요.

샘플의 README에 나와 있는 지침을 따라 봇을 설정하고 테스트하거나 [코드에 대한 메모](#notes-about-the-code)로 바로 넘어갈 수 있습니다.

## <a name="create-the-services-and-test-the-bot"></a>서비스를 만들고 봇 테스트

샘플의 README 지침을 따르세요. CLI 도구를 사용하여 이러한 서버를 생성 및 게시하고 구성(**.bot**) 파일에 이에 대한 정보를 업데이트합니다.

1. 샘플 리포지토리를 복제하거나 끌어옵니다.
1. BotBuilder CLI 도구를 설치합니다.
1. 필요한 서비스를 수동으로 구성합니다.

### <a name="test-your-bot"></a>봇 테스트

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Visual Studio 또는 Visual Studio Code를 사용하여 봇을 시작합니다.

<!--
# [JavaScript](#tab/javascript)
-->

---

Bot Framework Emulator를 사용하여 봇에 연결합니다.

다음은 포함된 서비스에서 처리하는 입력의 일부입니다.

* QnA Maker
  * `hi`, `good morning`
  * `what are you`, `what do you do`
* LUIS(가정 자동화)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS(날씨)
  * `whats the weather in chennai india`
  * `what's the forecast for bangalore`
  * `show me the forecast for nebraska`

## <a name="notes-about-the-code"></a>코드에 대한 메모

### <a name="packages"></a>패키지

이 샘플에서는 다음 패키지를 사용합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

이러한 [NuGet 패키지](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package)의 최신 v4 버전.

* `Microsoft.Bot.Builder`
* `Microsoft.Bot.Builder.AI.Luis`
* `Microsoft.Bot.Builder.AI.QnA`
* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Configuration`

<!--
# [JavaScript](#tab/javascript)

Download the [LUIS Dispatch sample][DispatchBotJs].  Install the required packages, including the `botbuilder-ai` package for LUIS and QnA Maker, using npm:

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`
-->

---

### <a name="botbuilder-cli-tools"></a>BotBuilder CLI 도구

이 샘플에서는 이러한 [BotBuilder CLI 도구](https://aka.ms/botbuilder-tools-readme)(npm을 통해 사용 가능)를 사용하여 LUIS, QnA Maker 및 Dispatch 서비스를 생성, 교육 및 게시하고 봇의 구성(**.bot**) 파일에 이러한 서비스에 대한 정보를 기록합니다.

* [Dispatch](https://aka.ms/botbuilder-tools-dispatch)
* [LUDown](https://aka.ms/botbuilder-tools-ludown-readme)
* [LUIS](https://aka.ms/botbuilder-tools-luis)
* [MSBot](https://aka.ms/botbuilder-tools-msbot-readme)
* [QnAMaker](https://aka.ms/botbuilder-tools-qnaMaker)

> [!TIP]
> npm 및 이러한 CLI 도구의 최신 버전이 있는지 확인하려면 다음 명령을 실행합니다.
>
> ```shell
> npm i -g npm dispatch ludown luis-apis msbot qnamaker
> ```

이 도구를 사용하여 서비스를 설정했다면 이 샘플의 **.bot** 파일이 이와 유사할 것입니다. (`msbot secret -n`을 실행하여 이 파일의 민감한 값을 암호화할 수 있습니다.)

```json
{
    "name": "NLP-With-Dispatch-Bot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "luis",
            "name": "Home Automation",
            "appId": "<your-home-automation-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "110"
        },
        {
            "type": "luis",
            "name": "Weather",
            "appId": "<your-weather-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "92"
        },
        {
            "type": "qna",
            "name": "Sample QnA",
            "kbId": "<your-qna-knowledge-base-id>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "endpointKey": "<your-qna-endpoint-key>",
            "hostname": "<your-qna-host-name>",
            "id": "184"
        },
        {
            "type": "dispatch",
            "name": "NLP-With-Dispatch-BotDispatch",
            "appId": "<your-dispatch-app-id>",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "version": "Dispatch",
            "region": "westus",
            "serviceIds": [
                "110",
                "92",
                "184"
            ],
            "id": "27"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

### <a name="connecting-to-the-services-from-your-bot"></a>봇에서 서비스에 연결

봇은 Dispatch, LUIS 및 QnA Maker 서비스에 연결하기 위해 **.bot** 파일에서 정보를 끌어옵니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**, `ConfigureServices`는 구성 파일을 읽고, `InitBotServices`를 해당 정보를 사용하여 서비스를 초기화합니다. 봇은 생성될 때마다 등록된 `BotServices` 개체를 통해 초기화됩니다. 이러한 두 가지 방법의 관련 부분은 다음과 같습니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    var connectedServices = InitBotServices(botConfig);

    services.AddSingleton(sp => connectedServices);
    //...
}
```

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

### <a name="working-with-the-recognition-results"></a>인식 결과 작업

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

모델에서 생성하는 결과에 발언을 가장 적절하게 처리할 수 있는 서비스가 나타납니다. 이 봇의 코드는 해당 서비스에 요청을 라우팅한 후 호출된 서비스에의 응답을 요약합니다.

```csharp
/// <summary>
/// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the request QnAMaker app.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the requested LUIS model.
/// </summary>
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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

## <a name="evaluate-the-dispatchers-performance"></a>디스패처의 성능 평가

경우에 따라 LUIS 앱과 QnA maker 서비스의 예로 제공되는 사용자 메시지가 있고, 디스패치가 생성하는 결합된 LUIS 앱이 해당 입력에 대해 잘 수행하지 않습니다. `eval` 옵션을 사용하여 앱의 성능을 확인할 수 있습니다.

```shell
dispatch eval
```

`dispatch eval`을 실행하면 언어 모델의 예상 성능에 대한 통계를 제공하는 **Summary.html** 파일을 생성합니다.

> [!TIP]
> 디스패치 도구에서 만든 LUIS 앱 뿐만 아닌 모든 LUIS 앱에서 `dispatch eval`을 수행할 수 있습니다.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>중복 및 겹침에 대해 의도 편집

**Summary.html**에 중복으로 플래그가 지정된 예제 발언을 검토하고, 비슷하거나 겹치는 예제를 제거합니다. 예를 들어 `Home Automation` LUIS 앱에서 QnA maker에 전달될 수 있도록 "TurnOnLights" 의도에 매핑하는 "내 전등 켜기"와 같은 요청이지만 "None" 의도에 매핑하는 "내 전등이 켜지지 않는 이유는?"과 같은 요청을 가정해 봅시다. 디스패치를 사용하여 LUIS 앱 및 QnA Maker 서비스를 결합할 때 다음 중 하나를 수행해야 합니다.

* 원래 `Home Automation` LUIS 앱에서 "None" 의도를 제거하고 해당 의도의 발언을 디스패처 앱의 "None" 의도에 추가합니다.
* 원래 LUIS 앱에서 "None" 의도를 제거하지 않는 경우 해당 의도를 QnA maker 서비스에 일치시키는 해당 메시지를 전달하도록 봇에 논리를 추가해야 합니다.

> [!TIP]
> 언어 모델의 성능 향상에 대한 팁은 [Language Understanding의 모범 사례](./bot-builder-concept-luis.md#best-practices-for-language-understanding)를 검토하세요.

## <a name="to-clean-up-resources-from-this-sample"></a>이 샘플에서 리소스를 정리하려면

이 샘플에서는 여러 응용 프로그램 및 리소스를 만듭니다. 이는 다음 지침을 따라 삭제할 수 있습니다.

### <a name="luis-resources"></a>LUIS 리소스

1. [luis.ai](https://www.luis.ai) 포털에 로그인합니다.
1. **내 앱** 페이지로 이동합니다.
1. 이 샘플로 만든 앱을 선택합니다.
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. **삭제**를 클릭하고 **확인**을 클릭하여 확인합니다.

### <a name="qna-maker-resources"></a>QnA Maker 리소스

1. [qnamaker.ai](https://www.qnamaker.ai/) 포털에 로그인합니다.
1. **내 기술 자료** 페이지로 이동합니다.
1. `Sample QnA` 기술 자료의 삭제 단추를 클릭하고 **삭제**를 클릭하여 확인합니다.

### <a name="azure-resources"></a>Azure 리소스

> [!WARNING]
> 다른 모든 앱 또는 서비스가 의존하는 리소스는 삭제하지 마세요.

1. [Azure Portal](https://portal.azure.com/)에 로그인합니다.
1. 샘플용으로 생성한 Cognitive Services 리소스의 **개요** 페이지로 이동합니다.
1. **삭제**를 클릭하고 **예**를 클릭하여 확인합니다.

## <a name="additional-resources"></a>추가 리소스

* [언어 이해](bot-builder-concept-luis.md)
* [지식 봇 설계](../bot-service-design-pattern-knowledge-base.md)
* [음성 초기화 구성](../bot-service-manage-speech-priming.md)
* [Language Understanding에 LUIS 사용](bot-builder-howto-v4-luis.md)
* [QnA Maker를 사용하여 질문에 답변](bot-builder-howto-qna.md)
