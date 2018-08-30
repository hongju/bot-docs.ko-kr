---
title: Language Understanding에 LUIS 사용 | Microsoft Docs
description: 자연어 이해를 위한 LUIS를 Bot Builder SDK와 함께 사용하는 방법을 알아봅니다.
keywords: Language Understanding, LUIS, 의도, 인식기, 엔터티, 미들웨어
author: ivorb
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/30/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 88651a2c86698d55e3a429d7ea62662976d2115f
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906167"
---
# <a name="using-luis-for-language-understanding"></a>Language Understanding에 LUIS 사용

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

사용자가 대화 및 컨텍스트를 통해 의미하는 바를 이해하는 기능을 구현하기가 어려울 수 있지만, 봇에 보다 자연스러운 대화 느낌을 제공할 수 있습니다. LUIS라고도 하는 Language Understanding을 사용하면 이와 같은 기능을 구현하여 봇이 사용자 메시지의 의도를 인식하고, 사용자가 보다 자연스러운 언어를 사용하고, 대화 흐름을 보다 원활하게 안내할 수 있습니다. LUIS가 봇과 통합되는 방식에 대한 자세한 배경 정보는 [봇용 Language Understanding](./bot-builder-concept-LUIS.md)을 참조하세요. 

이 토픽에서는 LUIS를 사용하여 몇 가지 의도를 인식하는 간단한 봇을 설정하는 방법을 안내합니다.

## <a name="installing-packages"></a>패키지 설치

먼저 LUIS에 필요한 패키지가 있는지 확인합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

다음과 같은 NuGet 패키지의 시험판 버전 v4에 [참조를 추가](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)합니다.


* `Microsoft.Bot.Builder.Ai.LUIS`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

npm을 통해 프로젝트의 botbuilder 및 botbuilder-ai 패키지에 참조를 추가할 수 있습니다.

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


## <a name="set-up-middleware-to-use-your-luis-app"></a>LUIS 앱을 사용하도록 미들웨어 설정

먼저 _LUIS 앱_을 설정합니다. 이 서비스는 [www.luis.ai](https://www.luis.ai)에서 만듭니다. LUIS 앱이 특정 의도를 인식할 수 있도록 교육할 수 있습니다. LUIS 앱을 만드는 방법에 대한 자세한 내용은 [LUIS 사이트](https://www.luis.ai)를 참조하세요.

이 예에서는 도움, 취소 및 날씨 의도를 인식할 수 있는 데모 LUIS 앱을 사용할 것이며, 앱 ID는 이미 샘플 코드에 있습니다. Cognitive Services 키가 필요합니다. 이 키는 [www.luis.ai](https://www.luis.ai)에 로그인하고 **사용자 설정** > **작성 키**에서 키를 복사하여 얻을 수 있습니다.

> [!NOTE] 
> 이 예제에 사용되는 공용 LUIS 앱의 고유한 복사본을 만들려면 [FirstSimpleBotExample.json](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/FirstSimpleBotExample.json) LUIS 파일을 복사합니다. 그리고 LUIS 앱을 [가져오고,](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) [교육](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)하고, [게시](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)합니다. 샘플 코드의 공용 앱 ID를 새 LUIS 앱의 앱 ID로 바꿉니다.

간단하게 앱을 봇의 미들웨어 스택에 추가하여 사용자에게 받은 모든 메시지에 대해 LUIS 앱을 호출하도록 봇을 구성합니다. 미들웨어는 컨텍스트 개체의 인식 결과를 저장합니다. 그러면 봇 논리에서 인식 결과에 액세스할 수 있습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
Echo 봇 템플릿으로 시작하여 **Startup.cs**를 엽니다. 

`Microsoft.Bot.Builder.Ai.LUIS`에 대한 `using` 문을 추가합니다.

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
// add this
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;
```

LUIS 앱에 연결하는 `LuisRecognizerMiddleware` 개체를 추가하도록 `Startup.cs` 파일의 `ConfigureServices` 메서드를 업데이트합니다. 

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
. 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        options.Middleware.Add(new ConversationState<EchoState>(dataStore));

        // Add LUIS recognizer as middleware
        options.Middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel(
                    // This appID is for a public app that's made available for demo purposes
                    "eb0bf5e0-b468-421b-9375-fdfb644c512e",
                    // You can use it by replacing <subscriptionKey> with your Authoring Key
                    // which you can find at https://www.luis.ai under User settings > Authoring Key
                    "<subscriptionKey>",
                    // The location-based URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
                    new Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"))));
    });
}
```

<!--TODO: DOES PUBLIC APP WORK WITH KEYS IN DIFFERENT REGIIONS? --> [https://www.luis.ai](https://www.luis.ai)의 구독 키를 `<subscriptionKey>` 자리에 붙여넣습니다.

> [!NOTE] 
> 공용 LUIS 앱 대신 고유의 LUIS 앱을 사용하는 경우 [https://www.luis.ai](https://www.luis.ai)에서 ID, 구독 키 및 LUIS 앱의 URL을 얻을 수 있습니다. 
>
>[LUIS 사이트](https://www.luis.ai)에 로그인하고, **게시** 탭으로 이동하고, **리소스 및 키** 아래에서 **엔드포인트**를 살펴보면 `LuisModel`에 사용할 기본 URL을 찾을 수 있습니다. 기본 URL은 구독 ID 및 기타 매개 변수 앞에 있는 **엔드포인트 URL** 부분입니다.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

먼저 [LuisRecognizer](https://github.com/Microsoft/botbuilder-js/tree/master/doc/botbuilder-ai/classes/botbuilder_ai.luisrecognizer.md) 클래스를 요구하고/가져오고 LUIS 모델에 사용할 인스턴스를 만듭니다.

```javascript
const { LuisRecognizer } = require('botbuilder-ai');

const model = new LuisRecognizer({
    // This appID is for a public app that's made available for demo purposes
    // You can use it by providing your LUIS subscription key
     appId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // replace subscriptionKey with your Authoring Key
    // your key is at https://www.luis.ai under User settings > Authoring Key 
    subscriptionKey: '<your subscription key>',
    // The serviceEndpoint URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com'
});
```

미들웨어 스택에 모델을 추가합니다.

```javascript
adapter.use(model);
```


> [!NOTE] 
> 공용 LUIS 앱 대신 고유의 LUIS 앱을 사용하는 경우 [https://www.luis.ai](https://www.luis.ai)에서 ID, 구독 ID 및 LUIS 앱의 URL을 얻을 수 있습니다. 

---



LUIS Language Understanding이 봇에 연결되었습니다. 다음으로, 컨텍스트 개체에 저장된 LUIS 모델에서 의도를 가져오는 방법을 살펴보겠습니다.


## <a name="get-the-intent-from-the-turn-context"></a>턴 컨텍스트에서 의도 가져오기

봇이 모든 대화 턴의 컨텍스트를 사용하여 LUIS의 결과에 액세스할 수 있습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

봇이 LUIS 앱에서 검색한 의도에 따라 응답을 보내게 하려면 `OnTurn`의 코드를 다음으로 바꿉니다.

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace Bot_Builder_Echo_Bot1
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Every Conversation turn for our EchoBot will call this method. In here
        /// the bot checks the Activty type to verify it's a message, checks the /// intent from the LUIS recognizer, and sends a reply based on the recognized intent
        /// </summary>
        /// <param name="context">Turn-scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {

                var result = context.Services.Get<RecognizerResult>
                    (LuisRecognizerMiddleware.LuisRecognizerResultKey);
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
                        // Cancel the process.
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

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const results = model.get(context);
            const topIntent = LuisRecognizer.topIntent(results);
            switch (topIntent) {

                case 'Cancel':
                    await context.sendActivity("<cancelling the process>")
                    break;
                case 'Help':
                    await context.sendActivity("<here's some help>");
                    break;
                case 'Weather':
                    await context.sendActivity("The weather today is sunny.");
                    break;
                case 'None':                    
                    await context.sendActivity("Sorry, I don't understand.")
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
인식된 엔터티는 엔터티 이름의 맵으로 값에 반환되며 `results.entities`를 통해 액세스할 수 있습니다. LuisRecognizer를 만들 때 `verbose=true` 설정을 전달하여 추가 엔터티 메타데이터를 반환할 수 있습니다. 추가된 메타데이터는 `results.entities.$instance`를 통해 액세스할 수 있습니다.

---

<!-- TODO: SHOW RUNNING THE FIRST BOT --> Bot Framework 에뮬레이터에서 봇을 실행하고, 봇에 "날씨", "도움말", "취소" 등을 말합니다.

![봇 실행](./media/how-to-luis/run-luis-bot.png)


## <a name="using-a-luis-recognizer-with-conversation-state"></a>대화 상태에 LUIS 인식기 사용
<!-- TBD, complete example --> 턴이 두 개 이상인 사용자에게 응답할 경우 대화 상태를 저장하여 대화에서 현재 있는 위치를 추적하도록 결정할 수 있습니다. LUIS의 의도를 사용하여 대화 토픽이 시작되었는지 또는 완료되었는지 여부와 같은 대화 상태 데이터를 설정할 수 있습니다.

사용자가 수신한 각 메시지의 의도를 받을 수 있도록 LUIS 인식기 미들웨어는 봇의 모든 턴에 대해 실행됩니다. 의도에 따라 다중 턴 대화 흐름을 시작하려는 경우에 사용 가능한 방법 중 하나는 현재 토픽을 완료할 때까지 토픽을 변경하는 논리를 건너뛰는 것입니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

EchoState.cs에서 EchoState를 다음으로 변경합니다.

```csharp
public class ConversationStateInfo 
{
    public bool WeatherTopicStarted  { get; set; }
    public bool HelpTopicStarted  { get; set; }
    public bool CancelTopicStarted  { get; set; }
}
```

Startup.cs에서, `ConversationStateInfo`를 사용하도록 ConversationState 초기화를 변경합니다.
```cs
    options.Middleware.Add(new ConversationState<ConversationStateInfo>(dataStore));
```

EchoBot.cs에서 `OnTurn`을 편집합니다.
```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        var text = context.Activity.Text;
        var conversationState = context.GetConversationState<ConversationStateInfo>() ?? new ConversationStateInfo();

        // Here, you can add some other logic based on the topic flags in conversation state
        // For example, if you know that a particular topic was started in a previous turn,
        // you can send the reply for that topic and bypass getting the intent from LUIS 
        if (conversationState.WeatherTopicStarted)
        {
            // Set this flag to false since this reply concludes the topic.
            conversationState.WeatherTopicStarted = false;
            // Assume that they responded to the prompt with a location.
            await context.SendActivity($"The weather in {text} is sunny.");
        }
        else
        {
            var result = context.Services.Get<RecognizerResult>
            (LuisRecognizerMiddleware.LuisRecognizerResultKey);
            var topIntent = result?.GetTopScoringIntent();
            switch ((topIntent != null) ? topIntent.Value.intent : null)
            {
                case null:
                    await context.SendActivity("Failed to get results from LUIS.");
                    break;
                case "None":
                    await context.SendActivity("Sorry, I don't understand.");
                    break;
                case "Weather":
                    conversationState.WeatherTopicStarted = true;
                    await context.SendActivity($"Looks like you want a weather forecast. What city do you want the forecast for?");

                    break;
                case "Help":
                    conversationState.HelpTopicStarted = true;
                    await context.SendActivity("<here's some help>");
                    break;
                case "Cancel":
                    // Cancel the process.
                    conversationState.CancelTopicStarted = true;
                    await context.SendActivity("<cancelling the process>");
                    break;
                default:
                    // Received an intent we didn't expect, so send its name and score.
                    await context.SendActivity($"Intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
                    break;
            }
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`LuisRecognizer`를 추가한 후 대화 상태 미들웨어를 추가합니다.

```javascript
const model = new LuisRecognizer({
    // This appID is for a public app that's made available for demo purposes
    // You can use it by providing your LUIS subscription key
     appId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // replace subscriptionKey with your Authoring Key
    // your key is at https://www.luis.ai under User settings > Authoring Key 
    subscriptionKey: '<your subscription>',
    // The serviceEndpoint URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com'
});
adapter.use(model)

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

그 후 어떤 토픽이 시작되었는지를 나타내는 상태를 저장할 수 있습니다.

```javascript
// Listen for incoming activities 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async(context) => {
        if (context.activity.type === 'message') {
            var utterance = context.activity.text;
            
            // Check topic flags in conversation state 
            if (conversationState.weatherTopicStarted) 
            {
                // Assume the user's message is a reply to the bot's prompt for a location
                await context.sendActivity(`The weather in ${utterance} is sunny.`);
                // This conversation flow is now finished. Set flag to false,
                // so that on the next turn the user can ask for another weather forecast.
                conversationState.WeatherTopicStarted = false;
            }
            // To add more steps to the other topics
            // you could check the topic flags here
            else 
            {
                const results = model.get(context);
                const topIntent = LuisRecognizer.topIntent(results);
                switch (topIntent) {
                    case 'None':
                        //Add app logic when there is no result
                        await context.sendActivity("<null case>")
                        break;
                    case 'Cancel':
                        conversationState.cancelTopicStarted = true;
                        await context.sendActivity("<cancelling the process>")
                        break;
                    case 'Help':
                        conversationState.helpTopicStarted = true;
                        await context.sendActivity("<here's some help>");
                        break;
                    case 'Weather':
                        conversationState.weatherTopicStarted = true;
                        await context.sendActivity("Looks like you want a weather forecast. What city do you want the forecast for?");
                        break;
                    default:
                        // Add app logic for the recognition results.
                        await context.sendActivity(`Received this intent: ${topIntent}`);
                }
            }
        }
    });
});
```

---

Bot Framework 에뮬레이터에서 봇을 실행합니다. 이제 "날씨 가져오기"가 2턴 대화 흐름인 것을 볼 수 있습니다.

![봇 실행](./media/how-to-luis/run-luis-bot-2step-weather.png)


## <a name="using-luis-with-dialogs"></a>대화 상자에 LUIS 사용

LUIS의 의도를 사용하여 다중 턴 대화 흐름을 트리거하는 경우 대화 상자를 사용하여 이 흐름을 캡슐화하면 도움이 될 수 있습니다. 이 예제 봇은 가정 자동화 대화 상자 또는 날씨 대화 상자를 트리거하는 데 사용되는 의도를 감지하는 LUIS 앱을 작업합니다.

> [!NOTE] 
> 이 예제에 사용되는 공용 LUIS 앱의 고유한 복사본을 만들려면 [WeatherOrHomeAutomation.json](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/WeatherOrHomeAutomation.json) LUIS 파일을 복사합니다. 그리고 LUIS 앱을 [가져오고,](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) [교육](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)하고, [게시](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)합니다. 샘플 코드의 공용 앱 ID를 새 LUIS 앱의 앱 ID로 바꿉니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

먼저 LUIS 앱에 대한 미들웨어를 추가하도록 Startup.cs에서 `ConfigureServices`를 수정합니다. `EchoBot`의 이름을 `LuisDialogBot`으로 바꿉니다.
이 예제에서 `LuisRecognizerMiddleware`로 추가된 LUIS 앱은 `homeautomation` 또는 `weather` 의도를 검색하고, 해당 이름을 가진 대화 상자를 트리거합니다. 

```csharp
public void ConfigureServices(IServiceCollection services)
{  
    // Rename EchoBot to LuisDialogBot
    services.AddBot<LuisDialogBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration); 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // Use Dictionary<string, object> for the conversation state type
        options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));

        // Add LUIS recognizer as middleware
        options.Middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel(
                    // This appID is for a public app that's made available for demo purposes
                    "428affb6-7650-46ac-9184-68c00a4f1729",
                    // You can use it by replacing <subscriptionKey> with your Authoring Key
                    // which you can find at https://www.luis.ai under User settings > Authoring Key
                    "<subscriptionKey>",
                    // The location-based URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
                    new Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"))));
    });
}
```

**EchoBot.cs**의 이름을 **LuisDialogBot.cs**로 변경하고, `EchoBot` 클래스의 이름을 `LuisDialogBot`으로 변경합니다. 그런 다음, **LuisDialogBot.cs**에서 다음 `using` 문을 추가합니다.

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
using Microsoft.Bot.Schema;
```

**LuisDialogBot.cs**에서 다음 코드를 `LuisDialogBot` 클래스에 추가합니다.

```csharp
    public class LuisDialogBot : IBot
    {
        private DialogSet _dialogs;

        public LuisDialogBot()
        {
            _dialogs = new DialogSet();

            _dialogs.Add("homeautomation", CreateHomeAutomationWaterfall());
            _dialogs.Add("weather", CreateWeatherWaterfall());
            _dialogs.Add("weather_city", new Builder.Dialogs.TextPrompt());
        }

        // App ID for a separate LUIS app used to tell if the user wants to turn the lights on or off
        // The `homeautomation` dialogs uses results from this app to determine the "on/off" argument. 
        private static LuisModel luisHomeAutomation =
            new LuisModel("76feb726-515b-44c4-acc9-adb216965a58", "SUBSCRIPTION-KEY", new System.Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"));

        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {

                // Create a dialog context
                var state = ConversationState<Dictionary<string, object>>.Get(context);
                var dc = _dialogs.CreateContext(context, state);

                // Run the next dialog step
                await dc.Continue();
                
                // Check if any dialog has responded on this turn
                if (!context.Responded)
                {

                    var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
                    var topIntent = luisResult?.GetTopScoringIntent();
                    var utterance = context.Activity.Text;
                    var dialogArgs = new Dictionary<string, object>();

                    switch ((topIntent != null) ? topIntent.Value.intent.ToLowerInvariant() : null)
                    {
                        case "homeautomation":
                            // The Homeautomation_TurnOn and Homeautomation_TurnOff dialogs 
                            // use results from a separate LUIS app.
                            // The results determine the "on/off" argument to pass to the dialog. 
                            var recognizerHomeAutomation = new LuisRecognizer(luisHomeAutomation);
                            RecognizerResult recognizerResult = await recognizerHomeAutomation.Recognize(utterance, System.Threading.CancellationToken.None);
                            var topHomeAutoIntent = recognizerResult.GetTopScoringIntent().intent;


                            dialogArgs.Add("IntentFromHomeAuto", "");
                            switch ((topHomeAutoIntent != null) ? topHomeAutoIntent.ToLowerInvariant() : null)
                            {
                                case "homeautomation_turnon":
                                    dialogArgs["Intent_HomeAutomation"] = "on";
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                                case "homeautomation_turnoff":
                                    dialogArgs["Intent_HomeAutomation"] = "off";
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                                case null:
                                    await dc.Begin("homeautomation", null);
                                    break;
                                default:
                                    dialogArgs["Intent_HomeAutomation"] = topHomeAutoIntent;
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                            }
                            break;
                        case "weather":
                            dialogArgs.Add("LuisResult", luisResult);
                            await dc.Begin("weather", dialogArgs);
                            break;
                        case null:
                            await context.SendActivity($"Couldn't get a result from LUIS. You said: {utterance}");
                            break;
                        default:
                            // The intent didn't match any case, so just display the recognition results.
                            await context.SendActivity($"you said: {utterance}");
                            await context.SendActivity($"Recognized intent: {topIntent.Value.intent}.");

                            break;

                    }

                }
            }
        }

    }


```

`OnTurn` 뒤에서, `LuisDialogBot` 클래스 내부에 대화 상자를 구현할 코드를 추가합니다.

```csharp
        // The home automation waterfall has one step
        private WaterfallStep[] CreateHomeAutomationWaterfall()
        {
            return new WaterfallStep[] {
                TurnLightsOnOrOff
            };
        }

        // The weather waterfall has two steps
        private WaterfallStep[] CreateWeatherWaterfall()
        {
            return new WaterfallStep[] {
                AskWeatherLocation,
                SendWeatherReport
            };
        }

        /// <summary>
        /// This is the first step and only of the home automation dialog.
        /// </summary>
        /// <param name="dc"></param>
        /// <param name="args">Can be "on", "off", another string for an intent name, or null.
        /// null indicates that no result was received from which to get an intent name.</param>
        /// <param name="next"></param>
        /// <returns></returns>
        private async Task TurnLightsOnOrOff(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var intentFromHomeAutomation = args["Intent_HomeAutomation"];
            if (args != null)
            {
                switch (intentFromHomeAutomation)
                {
                    case "on":
                    case "off":
                        await dc.Context.SendActivity($"Turning {intentFromHomeAutomation} your lights!");
                        break;
                    default:
                        await dc.Context.SendActivity($"Intent detected by homeautomation was: {intentFromHomeAutomation}, but the home automation system doesn't support that yet.");
                        break;
                }

            }
            else
            {
                await dc.Context.SendActivity($"You said {dc.Context.Activity.AsMessageActivity().Text}. Unable to get a result from which to determine on/off/other operation");
            }

            await dc.End();

        }

        // This is the first step of the weather dialog
        private async Task AskWeatherLocation(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            
            if (args["LuisResult"] is RecognizerResult luisResult)
            {
                var location = GetEntity<string>(luisResult, "Weather_Location");
                if (!string.IsNullOrEmpty(location))
                {
                    dialogState.Add("Location", location);
                }
            }

            // Save info back to the dialog instance
            dc.ActiveDialog.State = dialogState;

            if (!dialogState.ContainsKey("Location"))
            {
                await dc.Prompt("weather_city", "What city do you want the weather for?");
            }
            else
            {
                // We've set the location parameter for the weather report,
                // so go to the next step in the waterfall
                await dc.Continue();
            }
        }

        // This the second step of the weather dialog
        private async Task SendWeatherReport(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            if (args != null)
            {
                if (!dialogState.ContainsKey("Location"))
                {   // Interpret args as a reply to prompt only if they didn't give location
                    TextResult locationResult = (TextResult)args;
                    dialogState.Add("Location", locationResult.Text);
                }

            }

            // You can add some logic that uses the location 
            // to get the weather from a weather service instead of hard-coding it
            await dc.Context.SendActivity($"The weather forecast for '{dialogState["Location"]}' is sunny and 70 degrees F.");

            dc.ActiveDialog.State = new Dictionary<string, object>(); // clear the dialog state 
            await dc.End();
        }
```

이 도우미 함수를 추가합니다.

```cs
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

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

아직 대화 상자 라이브러리를 설치하지 않은 경우 지금 설치해야 할 수도 있습니다. 

```cmd
npm install --save botbuilder-dialogs
```

먼저 `adapter.use`를 사용하여 LUIS 앱을 만들고 봇에 추가합니다.

```javascript
const { BotFrameworkAdapter, ConversationState, MemoryStorage, TurnContext } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');
const { DialogSet } = require('botbuilder-dialogs');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Create LuisRecognizer 
// The LUIS application is public, meaning you can use your own subscription key to test the applications.
const luisRecognizer = new LuisRecognizer({
    appId: '1fefd4a7-ae5b-4e63-99f7-0cf499a1423b',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// Add the recognizer to your bot
adapter.use(luisRecognizer);

// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the intents detected by the LUIS app
const dialogs = new DialogSet();
```

다음으로 대화 상자를 추가합니다.

```javascript

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.homeAutomationTurnOn counts how many times this dialog was called 
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation_TurnOn" dialog.`);

        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetForecast', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.weatherGetForecast counts how many times this dialog was called  
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather_GetForecast" dialog.`);

        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.None counts how many times this dialog was called        
        state.None = state.None ? state.None + 1 : 1;
        await dialogContext.context.sendActivity(`${state.None}: You reached the "None" dialog.`);

        await dialogContext.end();
    }
]);
```

봇에서, 인식된 의도에 따라 각 대화 상자를 호출합니다.
```javascript
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our LUIS application
            const luisResults = luisRecognizer.get(context);

            // Extract the top intent from LUIS and use it to select which dialog to start
            // "NotFound" is the intent name for when no top intent can be found.
            const topIntent = LuisRecognizer.topIntent(luisResults, "NotFound");

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'homeautomation':                    
                        await dc.begin("HomeAutomation_TurnOn", luisResults);
                        break;
                    case 'weather':                    
                        await dc.begin("Weather_GetForecast", luisResults);
                        break;
                    case 'None':
                        await dc.begin("None");
                        break;
                    case 'NotFound':
                        await context.sendActivity(`Sorry, I didn't get any results from LUIS.`);
                        break;
                    default:
                        // handle intents for which we have no dialog
                        await context.sendActivity(`You reached the ${topIntent} intent.`);
                        break;
                }
            }
            
            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dialog bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});
```

Bot Framework 에뮬레이터에서 봇을 실행하고, 봇에 "라이트 켜기", "시애틀의 날씨 가져오기" 등을 말합니다.

![봇 실행](./media/how-to-luis/run-luis-bot-js-single-step-dialog.png)

---

## <a name="extract-entities"></a>엔터티 추출

의도 인식 외에도, LUIS 앱은 사용자의 요청을 수행하는 데 있어서 중요한 단어인 엔터티를 추출할 수 있습니다. 예를 들어 날씨 대화 상자 예제에서, LUIS 앱은 사용자의 메시지에서 날씨 보고서의 위치를 추출할 수 있습니다. 

대화 상자를 사용하는 일반적인 방법은 사용자의 메시지에서 엔터티를 식별하고, 찾지 못한 필수 엔터티에 대한 프롬프트를 표시하는 것입니다. 그런 다음, 후속 대화 상자 단계에서 프롬프트에 대한 응답을 처리합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

사용자의 메시지가 "시애틀 날씨가 어때?"라고 가정해 봅시다. 이 예제에 나오는 봇의 경우 [LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer)는 다음과 같은 구조의 [`Entities` 속성](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-)이 있는 [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult)를 제공합니다.

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

`LuisDialogBot` 클래스에 추가된 다음 도우미 함수는 LUIS의 `RecognizerResult`에서 엔터티를 가져옵니다.

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

대화 상자 내 여러 단계의 엔터티 같은 정보를 수집할 때 필요한 정보를 대화 상자별 상태에 저장하면 도움이 될 수 있습니다. 예를 들어 `AskWeatherLocation` 대화 상자에서는 대화 상자로 전달된 LUIS 결과에서 엔터티가 추출됩니다. `Weather_Location` 엔터티가 발견되면 대화 상자 상태에 추가됩니다.

```cs

        // This is the first step of the weather dialog
        private async Task AskWeatherLocation(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            
            if (args["LuisResult"] is RecognizerResult luisResult)
            {
                var location = GetEntity<string>(luisResult, "Weather_Location");
                if (!string.IsNullOrEmpty(location))
                {
                    dialogState.Add("Location", location);
                }
            }

            // Save info back to the dialog instance
            dc.ActiveDialog.State = dialogState;

            if (!dialogState.ContainsKey("Location"))
            {
                await dc.Prompt("weather_city", "What city do you want the weather for?");
            }
            else
            {
                // We've set the location parameter for the weather report,
                // so go to the next step in the waterfall
                await dc.Continue();
            }
        }
```

대화 상자의 두 번째 단계에서는 위치를 묻는 프롬프트에 대한 사용자의 응답뿐 아니라 이전 단계에서 저장한 정보를 대화 상자 인스턴스의 상태에서 가져올 수 있으며, 이 정보를 사용하여 사용자에게 날씨 보고서를 보낼 수 있습니다.

```cs

// This the second step of the weather dialog
private async Task SendWeatherReport(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
{
    var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
    if (args != null)
    {
        if (!dialogState.ContainsKey("Location"))
        {   // Interpret args as a reply to prompt only if they didn't give location
            TextResult locationResult = (TextResult)args;
            dialogState.Add("Location", locationResult.Text);
        }

    }

    // You can add some logic that uses the location and date
    // to get the weather from a weather service instead of hard-coding it
    await dc.Context.SendActivity($"The weather forecast for '{dialogState["Location"]}' is sunny and 70 degrees F.");
    
    // clear the dialog state 
    dc.ActiveDialog.State = new Dictionary<string, object>(); 
    await dc.End();
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

예를 들어 사용자의 메시지가 "시애틀 날씨가 어때?"라고 가정해 봅시다. 이 예제에 나오는 봇의 경우 [LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer)는 다음과 같은 구조의 `entities` 속성이 있는 [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult)를 제공합니다.

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

이 `findEntities` 함수는 LUIS 앱에서 인식한 `Weather_Location` 엔터티를 찾습니다.

<!-- TODO: Turn into a waterfall -->

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

`Weather_GetForecast` 대화 상자에서 `findEntities`를 호출합니다.

```javascript
// Pass the LUIS recognizer result to the args parameter
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
```

`dc.begin`에 전달된 LUIS 결과에서 엔터티가 추출됩니다.

```javascript
await dc.begin("Weather_GetForecast", luisResults);
```
---

## <a name="additional-resources"></a>추가 리소스

LUIS의 자세한 배경 정보는 [Language Understanding](./bot-builder-concept-luis.md)을 참조하세요.

이러한 예제에 사용된 앱을 빌드하는 방법은 [Bot Builder용 LUIS 앱](https://aka.ms/luis-bot-examples)을 참조하세요.

LUIS는 기타 Cognitive Services와 결합하여 봇을 훨씬 더 강력하게 만들 할 수 있습니다. 봇에서 QnA와 LUIS(Language Understanding)를 결합하는 방법은 [디스패치 도구를 사용하여 LUIS 앱 및 QnA 서비스 결합](./bot-builder-tutorial-dispatch.md)을 참조하세요.

## <a name="next-steps"></a>다음 단계


> [!div class="nextstepaction"]
> [강력한 형식의 LUIS 결과 생성](./bot-builder-howto-v4-luisgen.md)


