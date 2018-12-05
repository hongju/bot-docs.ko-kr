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
ms.date: 11/26/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9b0ddf5cf8af61048ba78f10824c9573da82fc08
ms.sourcegitcommit: a722f960cd0a8513d46062439eb04de3a0275346
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/27/2018
ms.locfileid: "52336273"
---
# <a name="use-multiple-luis-and-qna-models"></a>다중 LUIS 및 QnA 모델 사용

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

이 자습서에서는 봇이 지원하는 여러 시나리오에 대해 여러 LUIS 모델과 QnA Maker 서비스가 있을 때 디스패치 서비스를 사용하여 발화를 라우팅하는 방법을 보여 줍니다. 이 경우 홈 자동화 및 날씨 정보와 관련된 대화를 위한 여러 LUIS 모델로 디스패치를 구성하고, QnA Maker 서비스에서 FAQ 텍스트 파일에 기반하여 입력된 질문에 답변합니다. 이 샘플에서는 다음 서비스를 결합합니다.

| 이름 | 설명 |
|------|------|
| HomeAutomation | 연결된 엔터티 데이터를 사용하여 홈 자동화 의도를 인식하는 LUIS 앱입니다.|
| Weather | 위치 데이터를 사용하여 `Weather.GetForecast` 및 `Weather.GetCondition` 의도를 인식하는 LUIS 앱입니다.|
| FAQ  | 봇과 관련된 간단한 질문에 대한 답변을 제공하는 QnA Maker KB입니다. |

## <a name="prerequisites"></a>필수 조건

- 이 문서의 코드는 **디스패치를 통한 NLP** 샘플을 기반으로 합니다. [C#](https://aka.ms/dispatch-sample-cs) 또는 [JS](https://aka.ms/dispatch-sample-js)로 작성된 샘플의 복사본이 필요합니다.
- [봇 기본 사항](bot-builder-basics.md), [자연어 처리](bot-builder-howto-v4-luis.md), [QnA Maker](bot-builder-howto-qna.md) 및 [.bot](bot-file-basics.md) 파일에 대한 지식이 필요합니다.
- 테스트를 위해 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)가 필요합니다.

## <a name="create-the-services-and-test-the-bot"></a>서비스를 만들고 봇 테스트

[C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/14.nlp-with-dispatch/README.md) 또는 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/14.nlp-with-dispatch/README.md)에 대한 **README** 지침에 따라 에뮬레이터를 사용하여 샘플을 빌드하고 실행합니다. 

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
다음 코드는 외부 서비스에 대한 봇의 참조를 초기화합니다. 예를 들어 여기서는 LUIS 및 QnaMaker 서비스가 만들어집니다. 이러한 외부 서비스는 ".bot" 파일의 내용에 따라 `BotConfiguration` 클래스를 사용하여 구성됩니다.

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

`dispatch eval`을 실행하면 언어 모델의 예상 성능에 대한 통계를 제공하는 **Summary.html** 파일을 생성합니다. 디스패치 도구에서 만든 LUIS 앱 뿐만 아닌 모든 LUIS 앱에서 `dispatch eval`을 수행할 수 있습니다.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>중복 및 겹침에 대해 의도 편집

**Summary.html**에 중복으로 플래그가 지정된 예제 발언을 검토하고, 비슷하거나 겹치는 예제를 제거합니다. 예를 들어 `Home Automation` LUIS 앱에서 QnA maker에 전달될 수 있도록 "TurnOnLights" 의도에 매핑하는 "내 전등 켜기"와 같은 요청이지만 "None" 의도에 매핑하는 "내 전등이 켜지지 않는 이유는?"과 같은 요청을 가정해 봅시다. 디스패치를 사용하여 LUIS 앱 및 QnA Maker 서비스를 결합할 때 다음 중 하나를 수행해야 합니다.

* 원래 `Home Automation` LUIS 앱에서 "None" 의도를 제거하고 해당 의도의 발언을 디스패처 앱의 "None" 의도에 추가합니다.
* 원래 LUIS 앱에서 "None" 의도를 제거하지 않는 경우 해당 의도를 QnA maker 서비스에 일치시키는 해당 메시지를 전달하도록 봇에 논리를 추가해야 합니다.


## <a name="additional-resources"></a>추가 리소스 

**리소스 삭제:** 이 샘플에서는 아래에 나열된 단계를 사용하여 삭제할 수 있는 다양한 애플리케이션과 리소스를 만들지만 *다른 애플리케이션 또는 서비스*에서 사용하는 리소스는 삭제하면 안됩니다. 

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
