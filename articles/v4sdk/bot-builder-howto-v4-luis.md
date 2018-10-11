---
title: Language Understanding에 LUIS 사용 | Microsoft Docs
description: 자연어 이해를 위한 LUIS를 Bot Builder SDK와 함께 사용하는 방법을 알아봅니다.
keywords: Language Understanding, LUIS, 의도, 인식기, 엔터티, 미들웨어
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/19/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c4a7cfba6f588c95dbebf4886ffd7e432d99c3ae
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389804"
---
# <a name="using-luis-for-language-understanding"></a>Language Understanding에 LUIS 사용

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

사용자가 대화 및 컨텍스트를 통해 의미하는 바를 이해하는 기능을 구현하기가 어려울 수 있지만, 봇에 보다 자연스러운 대화 느낌을 제공할 수 있습니다. LUIS라고도 하는 Language Understanding을 사용하면 이와 같은 기능을 구현하여 봇이 사용자 메시지의 의도를 인식하고, 사용자가 보다 자연스러운 언어를 사용하고, 대화 흐름을 보다 원활하게 안내할 수 있습니다. LUIS가 봇과 통합되는 방식에 대한 자세한 배경 정보는 [봇용 Language Understanding](./bot-builder-concept-LUIS.md)을 참조하세요. 

이 토픽에서는 LUIS를 사용하여 몇 가지 의도를 인식하는 간단한 봇을 설정하는 방법을 안내합니다.

## <a name="installing-packages"></a>패키지 설치

먼저 LUIS에 필요한 패키지가 있는지 확인합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

다음과 같은 NuGet 패키지의 버전 v4에 [참조를 추가](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)합니다.


* `Microsoft.Bot.Builder.AI.LUIS`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

npm을 사용하여 프로젝트에 botbuilder 및 botbuilder-ai 패키지를 설치합니다.

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`

---

## <a name="set-up-your-luis-app"></a>LUIS 앱 설정

먼저 [luis.ai](https://www.luis.ai)에서 만든 서비스인 _LUIS 앱_을 설정합니다. LUIS 앱이 특정 의도를 인식할 수 있도록 교육할 수 있습니다. LUIS 앱을 만드는 방법에 대한 자세한 내용은 LUIS 사이트를 참조하세요.

이 예에서는 도움, 취소 및 날씨 의도를 인식할 수 있는 데모 LUIS 앱을 사용할 것이며, 앱 ID는 이미 샘플 코드에 있습니다. Cognitive Services 키가 필요합니다. 이 키는 [luis.ai](https://www.luis.ai)에 로그인하고 **사용자 설정** > **작성 키**에서 키를 복사하여 얻을 수 있습니다.

> [!NOTE]
> 이 예제에 사용되는 공용 LUIS 앱의 고유한 복사본을 만들려면 [JSON](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/FirstSimpleBotExample.json) LUIS 파일을 복사합니다. 그런 다음, LUIS 앱을 [가져오고](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app), [교육](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)하고, [게시](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)합니다. 샘플 코드의 공용 앱 ID를 새 LUIS 앱의 앱 ID로 바꿉니다.


### <a name="configure-your-bot-to-call-your-luis-app"></a>LUIS 앱을 호출하려면 봇을 구성합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

모든 전환에서 LUIS 앱을 만들고 호출할 수도 있지만 싱글톤으로 LUIS 서비스를 등록한 다음, 봇의 생성자에게 매개 변수로 전달하는 것이 더 나은 코딩 방법입니다. 해당 방법이 약간 더 복잡하므로 여기서 살펴보도록 하겠습니다.

Echo 봇 템플릿으로 시작하여 **Startup.cs**를 엽니다. 

`Microsoft.Bot.Builder.AI.LUIS`에 대한 `using` 문을 추가합니다.

```csharp
// add this
using Microsoft.Bot.Builder.AI.LUIS;
```

상태 초기화 뒤 `ConfigureServices`의 끝에 다음 코드를 추가합니다. 이렇게 하면 `appsettings.json` 파일에서 정보를 가져오지만 이러한 문자열은 이 문서의 끝에 연결되거나 테스트를 위해 하드 코딩된 샘플처럼 `.bot` 파일에서 가져올 수 있습니다.

싱글톤은 생성자에게 새 `LuisRecognizer`를 반환합니다.

```csharp
    // Create and register a LUIS recognizer.
    services.AddSingleton(sp =>
    {
        // Get LUIS information from appsettings.json.
        var section = this.Configuration.GetSection("Luis");
        var luisApp = new LuisApplication(
            applicationId: section["AppId"],
            endpointKey: section["SubscriptionKey"],
            azureRegion: section["Region"]);

        // Specify LUIS options. These may vary for your bot.
        var luisPredictionOptions = new LuisPredictionOptions
        {
            IncludeAllIntents = true,
        };

        return new LuisRecognizer(
            application: luisApp,
            predictionOptions: luisPredictionOptions,
            includeApiResults: true);
    });
```

`<subscriptionKey>` 대신에 [luis.ai](https://www.luis.ai)의 사용자 구독 키를 붙여넣습니다. 이렇게 하면 오른쪽 위에서 계정 이름을 클릭하고 **작성 키**라고 하는 **설정**으로 이동하는 경우 즉시 알 수 있습니다.

> [!NOTE]
> 공용 LUIS 앱 대신 고유의 LUIS 앱을 사용하는 경우 [luis.ai](https://www.luis.ai)에서 ID, 구독 키 및 LUIS 앱의 URL을 가져올 수 있습니다. 이들은 사용자 앱 페이지의 **게시** 및 **설정** 탭 아래에 있을 수 있습니다.
>
>[luis.ai](https://www.luis.ai)에 로그인하고, **게시** 탭으로 이동하고, **리소스 및 키** 아래에서 **엔드포인트** 열을 살펴보면 `LuisModel`에 사용할 기본 URL을 찾을 수 있습니다. 기본 URL은 구독 ID 및 기타 매개 변수 앞에 있는 **엔드포인트 URL** 부분입니다.

다음으로, 이 LUIS 인스턴스를 봇에 제공해야 합니다. **EchoBot.cs**을 열고, 파일의 맨 위에 다음 코드를 추가합니다. 참조용으로 클래스 제목 및 상태 항목도 포함되지만 여기서 설명하지는 않겠습니다.

```csharp
public class EchoBot : IBot
{
    /// <summary>
    /// Gets the Echo Bot state.
    /// </summary>
    private IStatePropertyAccessor<EchoState> EchoStateAccessor { get; }

    /// <summary>
    /// Gets the LUIS recognizer.
    /// </summary>
    private LuisRecognizer Recognizer { get; } = null;

    public EchoBot(ConversationState state, LuisRecognizer luis)
    {
        EchoStateAccessor = state.CreateProperty<EchoState>("EchoBot.EchoState");

        // The incoming luis variable is the LUIS Recognizer we added above.
        this.Recognizer = luis ?? throw new ArgumentNullException(nameof(luis));
    }

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

먼저, JavaScript [빠른 시작](../javascript/bot-builder-javascript-quickstart.md)의 단계를 따라 봇을 만듭니다. 여기서 LUIS 정보를 봇에 하드 코딩하지만 이 문서의 끝에 연결된 샘플에서 수행된 것처럼 `.bot` 파일에서 LUIS 정보를 끌어올 수 있습니다.

새 봇에서 **app.js**를 편집하여 `LuisRecognizer` 클래스를 요구하고 LUIS 모델에 사용할 인스턴스를 만듭니다.

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

const luisApplication = {
    // This appID is for a public app that's made available for demo purposes
    // You can use it or use your own LUIS "Application ID" at https://www.luis.ai under "App Settings".
     applicationId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // Replace endpointKey with your "Subscription Key"
    // your key is at https://www.luis.ai under Publish > Resources and Keys, look in the Endpoint column
    // The "subscription-key" is embeded in the Endpoint link. 
    endpointKey: '<your subscription key>',
    // You can find your app's region info embeded in the Endpoint link as well.
    // Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    azureRegion: 'westus'
}

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
}

// Create the bot that handles incoming Activities.
const luisBot = new LuisBot(luisApplication, luisPredictionOptions);
```

그런 다음, `LuisBot` 봇의 생성자에서 응용 프로그램을 가져와 LuisRecognizer 인스턴스를 만듭니다.

```javascript
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {object} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {object} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }
```

> [!NOTE] 
> 공용 LUIS 앱 대신 고유의 LUIS 앱을 사용하는 경우 [https://www.luis.ai](https://www.luis.ai)에서 ID, 구독 키 및 LUIS 앱의 지역을 가져올 수 있습니다. 이들은 사용자 앱 페이지의 게시 및 설정 탭 아래에 있을 수 있습니다.

---

LUIS Language Understanding이 이제 봇에 구성되었습니다. 다음으로, LUIS에서 의도를 가져오는 방법을 살펴보겠습니다.

## <a name="get-the-intent-by-calling-luis"></a>LUIS 호출하여 의도 가져오기

봇은 LUIS 인식기를 호출하여 LUIS에서 결과를 가져옵니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

봇이 LUIS 앱에서 검색한 의도에 따라 응답을 보내게 하려면 `LuisRecognizer`를 호출하여 `RecognizerResult`를 가져옵니다. 이 작업은 LUIS 의도를 가져와야 할 때마다 코드 내에서 수행될 수 있습니다.

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
// add this reference
using Microsoft.Bot.Builder.AI.LUIS;

namespace EchoBot
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Echo bot turn handler 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurnAsync(ITurnContext context, System.Threading.CancellationToken token)
        {            
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {
                // Call LUIS recognizer
                var result = this.Recognizer.RecognizeAsync(context, System.Threading.CancellationToken.None);
                
                var topIntent = result?.GetTopScoringIntent();

                switch ((topIntent != null) ? topIntent.Value.intent : null)
                {
                    case null:
                        await context.SendActivity("Failed to get results from LUIS.");
                        break;
                    case "None":
                        await context.SendActivity("Sorry, I don't understand.");
                        break;
                    case "Help":
                        await context.SendActivity("<here's some help>");
                        break;
                    case "Cancel":
                        // Cancel the process.
                        await context.SendActivity("<cancelling the process>");
                        break;
                    case "Weather":
                        // Report the weather.
                        await context.SendActivity("The weather today is sunny.");
                        break;
                    default:
                        // Received an intent we didn't expect, so send its name and score.
                        await context.SendActivity($"Intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
                        break;
                }
            }
        }
    }    
}

```

발언에서 인식된 의도는 의도 이름 맵으로 점수에 반환되며 `result.Intents`에서 액세스할 수 있습니다. 정적 `LuisRecognizer.topIntent()` 메서드는 결과 집합에 대한 최고점 의도를 간단하게 찾을 수 있도록 도와주기 위해 제공됩니다.

인식된 엔터티는 엔터티 이름의 맵으로 값에 반환되며 `results.entities`를 사용하여 액세스할 수 있습니다. LuisRecognizer를 만들 때 `verbose=true` 설정을 전달하여 추가 엔터티 메타데이터를 반환할 수 있습니다. 추가된 메타데이터는 `results.entities.$instance`를 사용하여 액세스할 수 있습니다.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

들어오는 활동을 수신 대기하는 코드를 편집함으로써 `LuisRecognizer`를 호출하여 `RecognizerResult`를 가져올 수 있습니다.

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;
            
            switch (topIntent) {
                case 'None':
                    await context.sendActivity("Sorry, I don't understand.")
                    break;
                case 'Cancel':
                    await context.sendActivity("<cancelling the process>")
                    break;
                case 'Help':
                    await context.sendActivity("<here's some help>");
                    break;
                case 'Weather':
                    await context.sendActivity("The weather today is sunny.");
                    break;                        
                case 'null':
                    await context.sendActivity("Failed to get results from LUIS.")
                    break;
                default:
                    // Received an intent we didn't expect, so send its name and score.
                    await context.sendActivity(`The top intent was ${topIntent}`);
            }
        }
    });
});
```

발언에서 인식된 의도는 의도 이름 맵으로 점수에 반환되며 `results.intents`에서 액세스할 수 있습니다. 정적 `LuisRecognizer.topIntent()` 메서드는 결과 집합에 대한 최고점 의도를 간단하게 찾을 수 있도록 도와주기 위해 제공됩니다.


---

Bot Framework 에뮬레이터에서 봇을 실행하여 "날씨", "도움말", "취소" 등을 봇에 말합니다.

![봇 실행](./media/how-to-luis/run-luis-bot.png)

## <a name="extract-entities"></a>엔터티 추출

의도를 인식하는 것 외에, LUIS 앱은 사용자 요청 이행에 중요한 단어에 해당하는 엔터티도 추출할 수 있습니다. 예를 들어 날씨 봇의 경우 LUIS 앱은 사용자의 메시지에서 날씨 보고서의 위치를 추출할 수 있습니다.

대화를 구성하는 일반적인 방법은 사용자의 메시지에서 엔터티를 식별하고, 찾지 못한 필수 엔터티에 대한 프롬프트를 표시하는 것입니다. 그런 다음, 후속 단계에서 해당 프롬프트에 대한 응답을 처리합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

사용자의 메시지가 "시애틀 날씨가 어때?"라고 가정해 봅시다. `LuisRecognizer`는 이 구조로 된 `Entities` 속성을 사용하여 `RecognizerResult`를 제공합니다.

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

다음 도우미 함수를 봇에 추가하여 LUIS의 `RecognizerResult`에서 엔터티를 가져올 수 있습니다. **using** 문에 추가해야 할 `Newtonsoft.Json.Linq` 라이브러리를 사용해야 합니다.

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

대화의 여러 단계에서 엔터티 같은 정보를 수집할 때 필요한 정보를 상태에 저장하면 도움이 될 수 있습니다. 엔터티를 발견한 경우 적절한 상태 필드에 추가할 수 있습니다. 대화에서 현재 단계가 이미 연결된 필드를 완료한 경우 해당 정보에 대한 프롬프트를 표시하는 단계는 건너뛸 수 있습니다.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

사용자의 메시지가 "시애틀 날씨가 어때?"라고 가정해 봅시다. `LuisRecognizer`는 이 구조로 된 `entities` 속성을 사용하여 `RecognizerResult`를 제공합니다.

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

이 `findEntities` 함수는 들어오는 `entityName`과 일치하는 LUIS 앱에서 인식한 모든 엔터티를 찾습니다.


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

대화의 여러 단계에서 엔터티 같은 정보를 수집할 때 필요한 정보를 상태에 저장하면 도움이 될 수 있습니다. 엔터티를 발견한 경우 적절한 상태 필드에 추가할 수 있습니다. 대화에서 현재 단계가 이미 연결된 필드를 완료한 경우 해당 정보에 대한 프롬프트를 표시하는 단계는 건너뛸 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

LUIS를 사용하는 샘플은 [[C#](https://aka.ms/cs-luis-sample)] 또는 [[JavaScript](https://aka.ms/js-luis-sample)]에 대한 프로젝트를 참조하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [디스패치 도구를 사용하여 LUIS 및 QnA 서비스 결합](./bot-builder-tutorial-dispatch.md)
