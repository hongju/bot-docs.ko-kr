---
title: 봇에 원격 분석 추가 | Microsoft Docs
description: 새로운 원격 분석 기능과 봇을 통합하는 방법에 대해 알아봅니다.
keywords: 원격 분석, Appinsights, 모니터 봇
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b6b4d30aea493180fddaee4a7f74bef72c1992ae
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215270"
---
# <a name="add-telemetry-to-your-bot"></a>봇에 원격 분석 추가

[!INCLUDE[applies-to](../includes/applies-to.md)]

Bot Framework SDK 버전 4.2에서 원격 분석 로깅이 제품에 추가되었습니다.  이렇게 하면 봇 애플리케이션은 Application Insights와 같은 서비스에 이벤트 데이터를 보낼 수 있습니다. 첫 번째 섹션에서는 이러한 메서드를 살펴본 후에 더 광범위한 원격 분석 기능도 살펴봅니다.

이 문서에서는 새로운 원격 분석 기능과 봇을 통합하는 방법에 대해 알아봅니다. 

## <a name="basic-telemetry-options"></a>기본 원격 분석 옵션

### <a name="basic-application-insights"></a>기본 Application Insights

먼저 Application Insights를 사용하여 기본 원격 분석을 봇에 추가해 보겠습니다. 설정에 대한 자세한 내용은 [Application Insights 시작](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Getting-Started-with-Application-Insights-for-ASP.NET-Core)의 처음 몇 개의 섹션을 참조하세요.   

추가 Application Insights 특정 구성(예: 원격 분석 이니셜라이저)이 필요하지 않은 "stock" Application Insights를 원하는 경우 다음을 `ConfigureServices()` 메서드에 추가합니다.   이는 Application Insights를 초기화하고 구성하여 요청, 다른 서비스에 대한 외부 호출 및 서비스 간 이벤트 상관 관계 설정을 추적하기 시작하는 가장 쉬운 방법입니다.

아래 코드 조각에 포함된 NuGet 패키지를 추가해야 합니다.

**Startup.cs**
```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.Bot.Builder.ApplicationInsights;
using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
 
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add Application Insights services into service collection
    services.AddApplicationInsightsTelemetry();

    // Add the standard telemetry client
    services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

    // Add ASP middleware to store the HTTP body, mapped with bot activity key, in the httpcontext.items
    // This will be picked by the TelemetryBotIdInitializer
    services.AddTransient<TelemetrySaveBodyASPMiddleware>();

    // Add telemetry initializer that will set the correlation context for all telemetry items
    services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

    // Add telemetry initializer that sets the user ID and session ID (in addition to other 
    // bot-specific properties, such as activity ID)
    services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();
    ...
}

// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...
    app.UseBotApplicationInsights();
}
```

그런 다음, Application Insights 계측 키를 `appsettings.json` 파일에 저장하거나 환경 변수로 저장해야 합니다. `appsettings.json` 파일에는 봇에서 실행 중에 사용하는 외부 서비스에 대한 메타데이터가 포함되어 있습니다.  예를 들어, CosmosDB, Application Insights 및 LUIS(Language Understanding) 서비스 연결 및 메타데이터는 여기에 저장됩니다. 계측 키는 Azure Portal의 **개요** 섹션(접혀 있는 경우 해당 페이지에 있는 서비스의 `Essentials` 드롭다운 아래)에서 찾을 수 있습니다. 키 가져오기에 대한 자세한 내용은 [여기서](~/bot-service-resources-app-insights-keys.md) 확인할 수 있습니다.

프레임워크에서 올바른 형식으로 지정된 키를 찾습니다. `appsettings.json` 항목의 형식은 다음과 같습니다.

```json
    "ApplicationInsights": {
        "InstrumentationKey": "putinstrumentationkeyhere"
    },
    "Logging": {
        "LogLevel": {
            "Default": "Warning"
        }
    }
```

Application Insights를 ASP.NET Core 애플리케이션에 추가하는 방법에 대한 자세한 내용은 [이 문서](https://docs.microsoft.com/en-us/azure/azure-monitor/app/asp-net-core-no-visualstudio)를 참조하세요. 

### <a name="customize-your-telemetry-client"></a>원격 분석 클라이언트 사용자 지정

Application Insights 클라이언트를 사용자 지정하거나 완전히 별도의 서비스에 로그인하려는 경우 시스템을 다르게 구성해야 합니다. nuget를 통해 `Microsoft.Bot.Builder.ApplicationInsights` 패키지를 다운로드하거나 npm을 사용하여 `botbuilder-applicationinsights`를 설치합니다. Application Insights 키를 가져오는 방법에 대한 자세한 내용은 [여기서](~/bot-service-resources-app-insights-keys.md) 확인할 수 있습니다.

**Application Insights 구성 수정**

구성을 수정하려면 Application Insights를 추가할 때 `options`를 포함시킵니다. 그렇지 않으면 모두 위와 동일합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add Application Insights services into service collection
    services.AddApplicationInsightsTelemetry(options);
    ...
}
```

`options` 개체는 `ApplicationInsightsServiceOptions` 형식입니다. 이러한 옵션에 대한 자세한 내용은 [여기서 확인할 수 있습니다]().

**사용자 지정 원격 분석 사용** Bot Framework에서 생성된 원격 분석 이벤트를 완전히 별도의 시스템에 로깅하려면 기본 `IBotTelemetryClient` 인터페이스에서 파생되는 새 클래스를 만들고 구성합니다. 그런 다음, 위와 같이 원격 분석 클라이언트를 추가할 때 사용자 지정 클라이언트를 삽입하기만 하면 됩니다. 

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry client.
    services.AddSingleton<IBotTelemetryClient, CustomTelemetryClient>();
    ...
}
```

### <a name="add-custom-logging-to-your-bot"></a>봇에 사용자 지정 로깅 추가

봇에 새로운 원격 분석 로깅 지원이 구성되면 봇에 원격 분석을 추가할 수 있습니다.  `BotTelemetryClient`(C#에서는 `IBotTelemetryClient`)에는 고유한 이벤트 유형을 기록하는 여러 가지 방법이 있습니다.  적절한 이벤트 유형을 선택하면 Application Insights 기존 보고서를 이용할 수 있습니다(Application Insights를 사용하는 경우).  일반적인 시나리오 `TraceEvent`가 일반적으로 사용됩니다.  `TraceEvent`를 사용하여 기록된 데이터는 Kusto의 `CustomEvent` 테이블에 배치됩니다.

봇 내에서 대화 상자를 사용하는 경우 모든 대화 상자 기반 개체(프롬프트 포함)에는 새로운 `TelemetryClient` 속성이 포함됩니다.  이는 로깅을 수행할 수 있게 해주는 `BotTelemetryClient`입니다.  이것은 단지 편의성만을 위한 것은 아니며, 이 속성이 설정되면 이 문서의 뒷부분에서 볼 수 있고, `WaterfallDialogs`는 이벤트를 생성합니다.

#### <a name="identifiers-and-custom-events"></a>식별자 및 사용자 지정 이벤트

이벤트를 Application Insights에 로깅하는 경우 생성된 이벤트는 입력할 필요가 없는 기본 속성을 포함합니다.  예를 들어, `user_id` 및 `session_id` 속성은 각 사용자 지정 이벤트(`TraceEvent` API로 생성된)에 포함됩니다.  또한, `activitiId`, `activityType` 및 `channelId`가 추가됩니다.

>참고: 사용자 지정 원격 분석 클라이언트는 이러한 값을 제공하지 않습니다.

자산 |Type | 세부 정보
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [봇 활동 ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [봇 활동 유형](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [봇 활동 채널 ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)

## <a name="in-depth-telemetry"></a>자세한 원격 분석

SDK 버전 4.4에 추가된 새 구성 요소 세 가지가 있습니다.  모든 구성 요소는 사용자 지정 구현으로 재정의할 수 있는 `IBotTelemetryClient`(또는 node.js의 `BotTelemetryClient`) 인터페이스를 사용하여 로깅됩니다.

- 메시지가 수신, 전송, 업데이트 또는 삭제되면 로깅되는 Bot Framework Middleware 구성 요소(*TelemetryLoggerMiddleware*). 사용자 지정 로깅으로 재정의할 수 있습니다.
- *LuisRecognizer* 클래스.  호출(속성 추가/바꾸기) 또는 파생된 클래스별로 사용자 지정 로깅으로 재정의할 수 있습니다.
- *QnAMaker* 클래스.  호출(속성 추가/바꾸기) 또는 파생된 클래스별로 사용자 지정 로깅으로 재정의할 수 있습니다.

### <a name="telemetry-middleware"></a>원격 분석 미들웨어

|C#  | JavaScript |
|:-----|:------------|
|**Microsoft.Bot.Builder.TelemetryLoggerMiddleware** | **botbuilder-core** |

#### <a name="out-of-box-usage"></a>바로 사용 가능

TelemetryLoggerMiddleware는 수정하지 않고 추가할 수 있는 Bot Framework 구성 요소이며, 로깅을 수행하여 Bot Framework SDK에 기본 제공되는 보고서를 사용하도록 설정합니다. 

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, TelemetryLoggerMiddleware>();

// Create the Bot Framework Adapter with error handling enabled.
services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();
```

#### <a name="adding-properties"></a>속성 추가
속성을 추가하려는 경우 TelemetryLoggerMiddleware 클래스를 파생할 수 있습니다.  예를 들어 "MyImportantProperty" 속성을 `BotMessageReceived` 이벤트에 추가하려는 경우가 있습니다.  사용자가 봇에 메시지를 보내면 `BotMessageReceived`가 로깅됩니다.  속성을 추가하는 방법은 다음과 같습니다.

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task OnReceiveActivityAsync(
                  Activity activity,
                  CancellationToken cancellation)
    {
        // Fill in the "standard" properties for BotMessageReceived
        // and add our own property.
        var properties = FillReceiveEventProperties(activity, 
                    new Dictionary<string, string>
                    { {"MyImportantProperty", "myImportantValue" } } );
                    
        // Use TelemetryClient to log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgReceiveEvent,
                        properties);
    }
    ...
}
```

그리고 Startup에서 새 클래스를 추가합니다.

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, MyTelemetryMiddleware>();
```

#### <a name="completely-replacing-properties--additional-events"></a>속성을 완전히 바꾸기/추가 이벤트

로깅되는 속성을 완전히 바꾸려는 경우 `TelemetryLoggerMiddleware` 클래스를 파생할 수 있습니다(위의 속성을 확장하는 경우와 유사함).   새 이벤트를 로깅하는 방법도 마찬가지입니다.

예를 들어 `BotMessageSend` 속성을 완전히 바꾸고 여러 이벤트를 보내려는 경우 이를 수행하는 방법을 설명하는 다음 내용을 참조하세요.

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task<RecognizerResult> OnLuisRecognizeAsync(
                  Activity activity,
                  string dialogId = null,
                  CancellationToken cancellation)
    {
        // Override properties for BotMsgSendEvent
        var botMsgSendProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        // Log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgSendEvent,
                        botMsgSendProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("activityId",
                                   activity.Id);
        secondEventProperties.Add("MyImportantProperty",
                                   "myImportantValue");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
    }
    ...
}
```
참고: 표준 속성이 로깅되지 않을 경우 제품에 기본 제공되는 보고서가 작동을 멈추게 됩니다.

#### <a name="events-logged-from-telemetry-middleware"></a>원격 분석 미들웨어에서 로깅된 이벤트
[BotMessageSend](#customevent-botmessagesend)
[BotMessageReceived](#customevent-botmessagereceived)
[BotMessageUpdate](#customevent-botmessageupdate)
[BotMessageDelete](#customevent-botmessagedelete)

### <a name="telemetry-support-luis"></a>원격 분석 지원 LUIS 

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.Luis** | **botbuilder-ai** |

#### <a name="out-of-box-usage"></a>바로 사용 가능
LuisRecognizer는 기존의 Bot Framework 구성 요소이며, `luisOptions`를 통해 IBotTelemetryClient 인터페이스를 전달하여 원격 분석을 사용하도록 설정할 수 있습니다.  로깅되는 기본 속성을 재정의할 수 있으며, 필요에 따라 새 이벤트를 로깅할 수 있습니다.

`luisOptions` 구성 중에는 `IBotTelemetryClient` 개체를 제공해야 해당 항목이 작동합니다.

```csharp
var luisOptions = new LuisPredictionOptions(
      ...
      telemetryClient,
      false); // Log personal information flag. Defaults to false.

var client = new LuisRecognizer(luisApp, luisOptions);
```

#### <a name="adding-properties"></a>속성 추가
속성을 추가하려는 경우 `LuisRecognizer` 클래스를 파생할 수 있습니다.  예를 들어 "MyImportantProperty" 속성을 `LuisResult` 이벤트에 추가하려는 경우가 있습니다.  `LuisResult`는 LUIS 예측 호출이 수행되면 로깅됩니다.  속성을 추가하는 방법은 다음과 같습니다.

```csharp
class MyLuisRecognizer : LuisRecognizer 
{
   ...
   override protected Task OnRecognizerResultAsync(
           RecognizerResult recognizerResult,
           ITurnContext turnContext,
           Dictionary<string, string> properties = null,
           CancellationToken cancellationToken = default(CancellationToken))
   {
       var luisEventProperties = FillLuisEventProperties(result, 
               new Dictionary<string, string>
               { {"MyImportantProperty", "myImportantValue" } } );
        
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResultEvent,
                        luisEventProperties);
        ..
   }    
   ...
}
```

#### <a name="add-properties-per-invocation"></a>호출당 속성 추가
호출 중에 속성을 추가해야 하는 경우가 있습니다.
```csharp
var additionalProperties = new Dictionary<string, string>
{
   { "dialogId", "myDialogId" },
   { "conversationInfo", "myConversationInfo" },
};

var result = await recognizer.RecognizeAsync(turnContext,
     additionalProperties,
     CancellationToken.None).ConfigureAwait(false);
```

#### <a name="completely-replacing-properties--additional-events"></a>속성을 완전히 바꾸기/추가 이벤트
로깅되는 속성을 완전히 바꾸려는 경우 `LuisRecognizer` 클래스를 파생할 수 있습니다(위의 속성을 확장하는 경우와 유사함).   새 이벤트를 로깅하는 방법도 마찬가지입니다.

예를 들어 `LuisResult` 속성을 완전히 바꾸고 여러 이벤트를 보내려는 경우 이를 수행하는 방법을 설명하는 다음 내용을 참조하세요.

```csharp
class MyLuisRecognizer : LuisRecognizer
{
    ...
    override protected Task OnRecognizerResultAsync(
             RecognizerResult recognizerResult,
             ITurnContext turnContext,
             Dictionary<string, string> properties = null,
             CancellationToken cancellationToken = default(CancellationToken))
    {
        // Override properties for LuisResult event
        var luisProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        
        // Log event
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResult,
                        luisProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("MyImportantProperty2",
                                   "myImportantValue2");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
        ...
    }
    ...
}
```
참고: 표준 속성이 로깅되지 않을 경우 제품에 기본 제공되는 Application Insights 보고서가 작동을 멈추게 됩니다.

#### <a name="events-logged-from-telemetryluisrecognizer"></a>TelemetryLuisRecognizer에서 로깅된 이벤트
[LuisResult](#customevent-luisevent)

### <a name="telemetry-qna-recognizer"></a>원격 분석 QnA 인식기

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.QnA** | **botbuilder-ai** |


#### <a name="out-of-box-usage"></a>바로 사용 가능
QnAMaker 클래스는 기존의 Bot Framework 구성 요소이며, 로깅을 수행하여 Bot Framework SDK에 기본 제공되는 보고서를 사용하도록 설정하는 생성자 매개 변수 두 개를 추가합니다. 새 `telemetryClient`는 로깅을 수행하는 `IBotTelemetryClient` 인터페이스를 참조합니다.  

```csharp
var qna = new QnAMaker(endpoint, options, client, 
                       telemetryClient: telemetryClient,
                       logPersonalInformation: true);
```
#### <a name="adding-properties"></a>속성 추가 
속성을 추가하려는 경우 두 가지 방법이 있습니다. QnA 호출 중에 답변을 검색하거나 `QnAMaker` 클래스에서 파생합니다.  

다음은 `QnAMaker` 클래스에서 파생하는 방법을 보여주는 예입니다.  이 예에서는 `QnAMessage` 이벤트에 "MyImportantProperty" 속성을 추가하는 방법을 보여줍니다.  `QnAMessage` 이벤트는 QnA `GetAnswers` 호출이 수행되면 로깅됩니다.  또한 두 번째 이벤트 "MySecondEvent"를 로깅합니다.

```csharp
class MyQnAMaker : QnAMaker 
{
   ...
   protected override Task OnQnaResultsAsync(
                 QueryResult[] queryResults, 
                 ITurnContext turnContext, 
                 Dictionary<string, string> telemetryProperties = null, 
                 Dictionary<string, double> telemetryMetrics = null, 
                 CancellationToken cancellationToken = default(CancellationToken))
   {
            var eventData = await FillQnAEventAsync(queryResults, turnContext, telemetryProperties, telemetryMetrics, cancellationToken).ConfigureAwait(false);

            // Add my property
            eventData.Properties.Add("MyImportantProperty", "myImportantValue");

            // Log QnaMessage event
            TelemetryClient.TrackEvent(
                            QnATelemetryConstants.QnaMsgEvent,
                            eventData.Properties,
                            eventData.Metrics
                            );

            // Create second event.
            var secondEventProperties = new Dictionary<string, string>();
            secondEventProperties.Add("MyImportantProperty2",
                                       "myImportantValue2");
            TelemetryClient.TrackEvent(
                            "MySecondEvent",
                            secondEventProperties);       }    
    ...
}
```

#### <a name="adding-properties-during-getanswersasync"></a>GetAnswersAsync 중 속성 추가
런타임 중에 추가해야 하는 속성이 있는 경우 `GetAnswersAsync` 메서드로 속성 및/또는 메트릭을 제공하여 이벤트에 추가할 수 있습니다.

예를 들어 이벤트에 `dialogId`를 추가하려는 경우 다음과 같이 할 수 있습니다.
```csharp
var telemetryProperties = new Dictionary<string, string>
{
   { "dialogId", myDialogId },
};

var results = await qna.GetAnswersAsync(context, opts, telemetryProperties);
```
`QnaMaker` 클래스는 PersonalInfomation 속성을 비롯한 속성을 재정의하는 기능을 제공합니다.

#### <a name="completely-replacing-properties--additional-events"></a>속성을 완전히 바꾸기/추가 이벤트
로깅되는 속성을 완전히 바꾸려는 경우 `TelemetryQnAMaker` 클래스를 파생할 수 있습니다(위의 속성을 확장하는 경우와 유사함).   새 이벤트를 로깅하는 방법도 마찬가지입니다.

예를 들어 `QnAMessage` 속성을 완전히 바꾸려는 경우 이를 수행하는 방법을 설명하는 다음 내용을 참조하세요.

```csharp
class MyLuisRecognizer : TelemetryQnAMaker
{
    ...
    protected override Task OnQnaResultsAsync(
         QueryResult[] queryResults, 
         ITurnContext turnContext, 
         Dictionary<string, string> telemetryProperties = null, 
         Dictionary<string, double> telemetryMetrics = null, 
         CancellationToken cancellationToken = default(CancellationToken))
    {
        // Add properties from GetAnswersAsync
        var properties = telemetryProperties ?? new Dictionary<string, string>();
        // GetAnswerAsync properties overrides - don't add if already present.
        properties.TryAdd("MyImportantProperty", "myImportantValue");

        // Log event
        TelemetryClient.TrackEvent(
                           QnATelemetryConstants.QnaMsgEvent,
                            properties);
    }
    ...
}
```
참고: 표준 속성이 로깅되지 않을 경우 제품에 기본 제공되는 보고서가 작동을 멈추게 됩니다.

#### <a name="events-logged-from-telemetryluisrecognizer"></a>TelemetryLuisRecognizer에서 로깅된 이벤트
[QnAMessage](#customevent-qnamessage)


## <a name="waterfalldialog-events"></a>WaterfallDialog 이벤트

사용자 고유의 이벤트를 생성하는 것 외에도 SDK 내의 `WaterfallDialog` 개체는 이제 이벤트를 생성합니다. 다음 섹션에서는 Bot Framework 내에서 생성된 이벤트에 대해 설명합니다. `WaterfallDialog`에 `TelemetryClient` 속성을 설정하면 이러한 이벤트가 저장됩니다.

다음은 원격 분석 이벤트를 기록하기 위해 `WaterfallDialog`를 사용하는 샘플(CoreBot)을 수정하는 예제입니다.  CoreBot은 `ComponentDialog`(`GreetingDialog`) 내에 `WaterfallDialog`가 배치되는 경우 사용되는 일반적인 패턴을 사용합니다.

```csharp
// IBotTelemetryClient is direct injected into our Bot
public CoreBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
...

// The IBotTelemetryClient passed to the GreetingDialog
...
Dialogs = new DialogSet(_dialogStateAccessor);
Dialogs.Add(new GreetingDialog(_greetingStateAccessor, telemetryClient));
...

// The IBotTelemetryClient to the WaterfallDialog
...
AddDialog(new WaterfallDialog(ProfileDialog, waterfallSteps) { TelemetryClient = telemetryClient });
...

```

`WaterfallDialog`에 `IBotTelemetryClient`가 구성되면 이벤트 로깅이 시작됩니다.

## <a name="events-generated-by-the-bot-framework-service"></a>Bot Framework Service에서 생성된 이벤트

봇 코드에서 이벤트를 생성하는 `WaterfallDialog` 외에도 Bot Framework 채널 서비스도 이벤트를 기록합니다.  이 기능을 사용하면 채널 또는 전체 봇 오류 문제를 진단할 수 있습니다.

### <a name="customevent-activity"></a>CustomEvent: "활동"
**로그인 출처:** 채널 서비스는 메시지 수신 시 채널 서비스에서 기록됩니다.

### <a name="exception-bot-errors"></a>예외: "봇 오류"
**로그인 출처:** 봇에 대한 호출이 비 2XX Http 응답을 반환하는 경우 채널에 의해 로깅되는 채널 서비스입니다.

## <a name="all-events-generated"></a>생성된 모든 이벤트

### <a name="customevent-waterfallstart"></a>CustomEvent: "WaterfallStart" 

WaterfallDialog가 시작되면 `WaterfallStart` 이벤트가 기록됩니다.

- `user_id`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `session_id` ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId`(이것은 폭포에 전달된 dialogId(문자열)입니다.  이를 "폭포 유형"으로 고려할 수 있습니다.)
- `customDimensions.InstanceID`(대화 상자의 인스턴스마다 고유)

### <a name="customevent-waterfallstep"></a>CustomEvent: "WaterfallStep" 

폭포 대화 상자에서 개별 단계를 기록합니다.

- `user_id`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `session_id` ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId`(이것은 폭포에 전달된 dialogId(문자열)입니다.  이를 "폭포 유형"으로 고려할 수 있습니다.)
- `customDimensions.StepName`(메서드 이름 또는 람다의 경우 `StepXofY`)
- `customDimensions.InstanceID`(대화 상자의 인스턴스마다 고유)

### <a name="customevent-waterfalldialogcomplete"></a>CustomEvent: "WaterfallDialogComplete"

폭포 대화 상자가 완료되면 기록됩니다.

- `user_id`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `session_id` ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId`(이것은 폭포에 전달된 dialogId(문자열)입니다.  이를 "폭포 유형"으로 고려할 수 있습니다.)
- `customDimensions.InstanceID`(대화 상자의 인스턴스마다 고유)

### <a name="customevent-waterfalldialogcancel"></a>CustomEvent: "WaterfallDialogCancel" 

폭포 대화 상자가 취소되면 기록됩니다.

- `user_id`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `session_id` ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType`  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId`(이것은 폭포에 전달된 dialogId(문자열)입니다.  이를 "폭포 유형"으로 고려할 수 있습니다.)
- `customDimensions.StepName`(메서드 이름 또는 람다의 경우 `StepXofY`)
- `customDimensions.InstanceID`(대화 상자의 인스턴스마다 고유)

### <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
봇이 사용자로부터 새 메시지를 수신할 때 로깅됩니다.

재정의되지 않을 경우, 이 이벤트는 `Microsoft.Bot.Builder.TelemetryLoggerMiddleware`에서 `Microsoft.Bot.Builder.IBotTelemetry.TrackEvent()` 메서드를 사용하여 로깅됩니다.

- 세션 식별자  
  - Application Insights를 사용하는 경우 이는 `TelemetryBotIdInitializer`에서 Application Insights와 함께 사용되는 **세션** 식별자(*Temeletry.Context.Session.Id*)로 로깅됩니다.  
  - Bot Framework 프로토콜에 정의된 [대화 ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)에 해당합니다.
  - 로깅되는 속성 이름은 `session_id`입니다.

- 사용자 ID
  - Application Insights를 사용하는 경우 이는 `TelemetryBotIdInitializer`에서 Application Insights와 함께 사용되는 **사용자** ID(*Telemetry.Context.User.Id*)로 로깅됩니다.  
  - 이 속성의 값은 Bot Framework 프로토콜에 정의된 [채널 식별자](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)와 [사용자 ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)(함께 연결) 속성의 조합입니다.
  - 로깅되는 속성 이름은 `user_id`입니다.

- ActivityID 
  - Application Insights를 사용하는 경우 `TelemetryBotIdInitializer`에서 ActivityID가 이벤트에 대한 속성으로 로깅됩니다.
  - Bot Framework 프로토콜에 정의된 [작업 ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#Id)에 해당합니다.
  - 속성 이름은 `activityId`입니다.

- 채널 식별자
  - Application Insights를 사용하는 경우 `TelemetryBotIdInitializer`에서 ActivityID가 이벤트에 대한 속성으로 로깅됩니다.  
  - Bot Framework 프로토콜의 [채널 식별자](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)에 해당합니다.
  - 로깅되는 속성 이름은 `channelId`입니다.

- ActivityType 
  - Application Insights를 사용하는 경우 `TelemetryBotIdInitializer`에서 ActivityID가 이벤트에 대한 속성으로 로깅됩니다.  
  - Bot Framework 프로토콜의 [작업 유형](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#type)에 해당합니다.
  - 로깅되는 속성 이름은 `activityType`입니다.

- 텍스트
  - `logPersonalInformation` 속성이 `true`로 설정된 경우 **필요에 따라** 로깅됩니다.
  - Bot Framework 프로토콜의 [작업 텍스트](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#text) 필드에 해당합니다.
  - 로깅되는 속성 이름은 `text`입니다.

- Speak

  - `logPersonalInformation` 속성이 `true`로 설정된 경우 **필요에 따라** 로깅됩니다.
  - Bot Framework 프로토콜의 [작업 음성](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#speak) 필드에 해당합니다.
  - 로깅되는 속성 이름은 `speak`입니다.

  - 

- FromId
  - Bot Framework 프로토콜의 [받는 사람 식별자](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) 필드에 해당합니다.
  - 로깅되는 속성 이름은 `fromId`입니다.

- FromName
  - `logPersonalInformation` 속성이 `true`로 설정된 경우 **필요에 따라** 로깅됩니다.
  - Bot Framework 프로토콜의 [받는 사람 이름](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) 필드에 해당합니다.
  - 로깅되는 속성 이름은 `fromName`입니다.

- RecipientId
  - Bot Framework 프로토콜의 [받는 사람 이름](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) 필드에 해당합니다.
  - 로깅되는 속성 이름은 `fromName`입니다.

- RecipientName
  - Bot Framework 프로토콜의 [받는 사람 이름](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) 필드에 해당합니다.
  - 로깅되는 속성 이름은 `fromName`입니다.

- ConversationId
  - Bot Framework 프로토콜의 [받는 사람 이름](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) 필드에 해당합니다.
  - 로깅되는 속성 이름은 `fromName`입니다.

- ConversationName
  - Bot Framework 프로토콜의 [받는 사람 이름](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) 필드에 해당합니다.
  - 로깅되는 속성 이름은 `fromName`입니다.

- 로캘
  - Bot Framework 프로토콜의 [받는 사람 이름](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from) 필드에 해당합니다.
  - 로깅되는 속성 이름은 `fromName`입니다.

### <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
**로그인 출처:** TelemetryLoggerMiddleware 

봇이 메시지를 보낼 때 기록됩니다.

- UserID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- SessionID([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- 채널  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityType  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ReplyToID
- RecipientId
- ConversationName
- 로캘
- RecipientName(PII의 경우 선택 사항)
- 텍스트(PII의 경우 선택 사항)
- Speak(PII의 경우 선택 사항)


### <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
**로그인 출처:** TelemetryLoggerMiddleware는 봇에서 메시지를 업데이트할 때 기록됩니다(희귀 사례).
- UserID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- SessionID([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityID([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- 채널  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityType  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName
- 로캘
- 텍스트(PII의 경우 선택 사항)


### <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**로그인 출처:** TelemetryLoggerMiddleware는 봇에서 메시지를 삭제할 때 기록됩니다(희귀 사례).
- UserID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- SessionID([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- 채널  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityType  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName

### <a name="customevent-luisevent"></a>CustomEvent: LuisEvent
**로그인 출처:** LuisRecognizer

LUIS 서비스의 결과를 기록합니다.

- UserID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- SessionID([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityID([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- Channel([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityType([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ApplicationId
- 의도
- IntentScore
- Intent2 
- IntentScore2 
- FromId
- SentimentLabel
- SentimentScore
- Entities(JSON 형식)
- Question(PII의 경우 선택 사항)

## <a name="customevent-qnamessage"></a>CustomEvent: QnAMessage
**로그인 출처:** QnAMaker

QnA Maker 서비스의 결과를 기록합니다.

- UserID([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- SessionID([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityID([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- Channel([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityType  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- Username(PII의 경우 선택 사항)
- Question(PII의 경우 선택 사항)
- MatchedQuestion
- QuestionId
- 응답
- Score
- ArticleFound

## <a name="querying-the-data"></a>데이터 쿼리
Application Insights를 사용하는 경우 모든 데이터는 서로 상관 관계가 지정됩니다(서비스 간에도 마찬가지).  성공적인 요청을 쿼리하여 확인할 수 있고 해당 요청에 대해 관련된 모든 이벤트를 볼 수 있습니다.  
다음 쿼리는 가장 최근의 요청을 알려줍니다.
```sql
requests 
| where timestamp > ago(3d) 
| where resultCode == 200
| order by timestamp desc
| project timestamp, operation_Id, appName
| limit 10
```

첫 번째 쿼리에서 `operation_Id` 몇 개를 선택한 다음, 추가 정보를 찾아보세요.

```sql
let my_operation_id = "<OPERATION_ID>";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};
union_all
    | order by timestamp asc
    | project itemType, name, performanceBucket
```

이렇게 하면 각 호출의 기간 버킷을 사용하여 단일 요청을 시간순으로 분석할 수 있습니다.
![호출 예](media/performance_query.png)

> 참고: 이러한 이벤트는 비동기적으로 기록되기 때문에 "활동" `customEvent` 이벤트 타임스탬프는 잘못되었습니다.

## <a name="create-a-dashboard"></a>대시보드 만들기

가장 쉬운 테스트 방법은 [Azure Portal의 템플릿 배포 페이지](https://portal.azure.com/#create/Microsoft.Template)를 사용하여 대시보드를 만드는 것입니다.  
- ["편집기에서 사용자 고유의 템플릿을 빌드합니다."]를 클릭합니다.
- 대시보드를 만드는 데 도움이 되도록 제공된 .json 파일 중 하나를 복사하여 붙여넣습니다.
  - [시스템 상태 대시보드](https://aka.ms/system-health-appinsights)
  - [대화 상태 대시보드](https://aka.ms/conversation-health-appinsights)
- "저장"을 클릭합니다.
- `Basics` 채우기: 
   - 구독: <your test subscription>
   - 리소스 그룹: <a test resource group>
   - 위치: <such as West US>
- `Settings` 채우기:
   - 인사이트 구성 요소 이름: <`core672so2hw`와 같음>
   - 인사이트 구성 요소 리소스 그룹: <`core67`과 같음>
   - 대시보드 이름:  <`'ConversationHealth'` 또는 `SystemHealth`와 같음>
- `I agree to the terms and conditions stated above`을 클릭합니다.
- `Purchase`을 클릭합니다.
- 유효성 검사
   - [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups)를 클릭합니다.
   - 위의 리소스 그룹(`core67`과 같은)을 선택합니다.
   - 새 리소스가 표시되지 않으면 "배포"를 확인하여 실패한 리소스가 있는지 확인합니다.
   - 일반적으로 실패는 다음과 같습니다.
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template/#parameters for usage details.'\"\r\n }\r\n}"}]}
```

데이터를 보려면 Azure Portal로 이동합니다. 왼쪽의 **대시보드**를 클릭한 다음, 드롭다운에서 만든 대시보드를 선택합니다.

## <a name="additional-resources"></a>추가 리소스
원격 분석을 구현하는 이러한 샘플을 참조할 수 있습니다.
- C#
  - [AppInsights가 포함된 LUIS](https://aka.ms/luis-with-appinsights-cs)
  - [AppInsights가 포함된 QnA](https://aka.ms/qna-with-appinsights-cs)
- JS
  - [AppInsights가 포함된 LUIS](https://aka.ms/luis-with-appinsights-js)
  - [AppInsights가 포함된 QnA](https://aka.ms/qna-with-appinsights-js)

