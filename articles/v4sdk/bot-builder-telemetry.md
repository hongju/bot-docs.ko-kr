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
ms.date: 02/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 75e12ab44915783c33c3b2ee10775cc6f00487bb
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905036"
---
# <a name="add-telemetry-to-your-bot"></a>봇에 원격 분석 추가

[!INCLUDE[applies-to](../includes/applies-to.md)]

Bot Framework SDK 버전 4.2에서 원격 분석 로깅이 제품에 추가되었습니다.  이렇게 하면 봇 애플리케이션은 Application Insights와 같은 서비스에 이벤트 데이터를 보낼 수 있습니다.

이 문서에서는 새로운 원격 분석 기능과 봇을 통합하는 방법에 대해 알아봅니다.  

## <a name="using-bot-configuration-option-1-of-2"></a>봇 구성 사용(옵션 1/2)
봇을 구성하는 방법에는 두 가지가 있습니다.  첫 번째는 Application Insights와 통합하는 것을 가정합니다.

봇 구성 파일에는 봇이 실행 중에 사용하는 외부 서비스에 대한 메타데이터가 포함되어 있습니다.  예를 들어, CosmosDB, Application Insights 및 LUIS(Language Understanding) 서비스 연결 및 메타데이터는 여기에 저장됩니다.   

"스톡" Application Insights를 원하는 경우 Application Insights 관련 추가 구성(예: 원격 분석 이니셜라이저)이 필요 없이 초기화하는 동안 봇 구성 개체를 전달합니다.   이는 Application Insights를 초기화하고 구성하여 요청, 다른 서비스에 대한 외부 호출 및 서비스 간 이벤트 상관 관계 설정을 추적하기 시작하는 가장 쉬운 방법입니다.

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Add Application Insights - pass in the bot configuration
     services.AddBotApplicationInsights(botConfig);
     ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
     app.UseBotApplicationInsights()
                 ...
                .UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
                ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**봇 구성에 Application Insights가 포함되지 않음** 봇 구성에 Application Insights가 포함되어 있지 않으면 어떻게 되나요?  걱정할 필요가 없습니다. 메서드 호출을 하지 않는 null 클라이언트가 기본값입니다.

**여러 Application Insights** 봇 구성에 여러 개의 Application Insight 섹션이 있습니까?  봇 구성 내에서 사용하려는 Application Insights 서비스 인스턴스를 지정할 수 있습니다.

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     // Add Application Insights
     services.AddBotApplicationInsights(botConfig, "myAppInsights");
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="overriding-the-telemetry-client-option-2-of-2"></a>원격 분석 클라이언트 재정의(옵션 2/2)

Application Insights 클라이언트를 사용자 지정하거나 완전히 별도의 서비스에 로그인하려는 경우 시스템을 다르게 구성해야 합니다.

**Application Insights 구성 수정**

```csharp

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create Application Insight Telemetry Client
     // with custom configuration.
     var telemetryClient = TelemetryClient(myCustomConfiguration)
     
     // Add Application Insights
     services.AddBotApplicationInsights(new BotTelemetryClient(telemetryClient), "InstrumentationKey");
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**사용자 지정 원격 분석 사용** Bot Framework에서 생성된 원격 분석 이벤트를 완전히 별도의 시스템에 로깅하려는 경우 기본 인터페이스에서 파생된 새 클래스를 만들고 구성합니다.  

```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create my IBotTelemetryClient-based logger
     var myTelemetryClient = MyTelemetryLogger();
     
     // Add Application Insights
     services.AddBotApplicationInsights(myTelemetryClient);
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="add-custom-logging-to-your-bot"></a>봇에 사용자 지정 로깅 추가

봇에 새로운 원격 분석 로깅 지원이 구성되면 봇에 원격 분석을 추가할 수 있습니다.  `BotTelemetryClient`(C#에서는 `IBotTelemetryClient`)에는 고유한 이벤트 유형을 기록하는 여러 가지 방법이 있습니다.  적절한 이벤트 유형을 선택하면 Application Insights 기존 보고서를 이용할 수 있습니다(Application Insights를 사용하는 경우).  일반적인 시나리오 `TraceEvent`가 일반적으로 사용됩니다.  `TraceEvent`를 사용하여 기록된 데이터는 Kusto의 `CustomEvent` 테이블에 배치됩니다.

봇 내에서 대화 상자를 사용하는 경우 모든 대화 상자 기반 개체(프롬프트 포함)에는 새로운 `TelemetryClient` 속성이 포함됩니다.  이는 로깅을 수행할 수 있게 해주는 `BotTelemetryClient`입니다.  이것은 단지 편의성만을 위한 것은 아니며, 이 속성이 설정되면 이 문서의 뒷부분에서 볼 수 있고, `WaterfallDialogs`는 이벤트를 생성합니다.

### <a name="identifiers-and-custom-events"></a>식별자 및 사용자 지정 이벤트

이벤트를 Application Insights에 로깅하는 경우 생성된 이벤트는 입력할 필요가 없는 기본 속성을 포함합니다.  예를 들어, `user_id` 및 `session_id` 속성은 각 사용자 지정 이벤트(`TraceEvent` API로 생성된)에 포함됩니다.  또한, `activitiId`, `activityType` 및 `channelId`가 추가됩니다.

>참고: 사용자 지정 원격 분석 클라이언트는 이러한 값을 제공하지 않습니다.

자산 |Type | 세부 정보
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [봇 활동 ID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [봇 활동 유형](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [봇 활동 채널 ID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)

## <a name="waterfalldialog-events"></a>WaterfallDialog 이벤트

사용자 고유의 이벤트를 생성하는 것 외에도 SDK 내의 `WaterfallDialog` 개체는 이제 이벤트를 생성합니다. 다음 섹션에서는 Bot Framework 내에서 생성된 이벤트에 대해 설명합니다. `WaterfallDialog`에 `TelemetryClient` 속성을 설정하면 이러한 이벤트가 저장됩니다.

다음은 원격 분석 이벤트를 기록하기 위해 `WaterfallDialog`를 사용하는 샘플(BasicBot)을 수정하는 예제입니다.  BasicBot은 `WaterfallDialog`가 `ComponentDialog`(`GreetingDialog`) 내에 배치되는 일반적인 패턴을 사용합니다.

```csharp
// IBotTelemetryClient is direct injected into our Bot
public BasicBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
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


## <a name="events-generated-by-the-bot-framework-service"></a>Bot Framework Service에서 생성된 이벤트

봇 코드에서 이벤트를 생성하는 `WaterfallDialog` 외에도 Bot Framework 채널 서비스도 이벤트를 기록합니다.  이 기능을 사용하면 채널 또는 전체 봇 오류 문제를 진단할 수 있습니다.

### <a name="customevent-activity"></a>CustomEvent: "활동"
**로그인 출처:** 채널 서비스는 메시지 수신 시 채널 서비스에서 기록됩니다.

### <a name="exception-bot-errors"></a>예외: "봇 오류"
**로그인 출처:** 봇에 대한 호출이 비 2XX Http 응답을 반환하는 경우 채널에 의해 로깅되는 채널 서비스입니다.

## <a name="additional-events"></a>추가 이벤트

[엔터프라이즈 템플릿](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)은 자유롭게 복사할 수 있는 오픈 소스 코드입니다.  여기에는 보고 요구에 맞게 재사용 및 수정할 수 있는 여러 구성 요소가 포함되어 있습니다.

### <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
**로그인 출처:** TelemetryLoggerMiddleware(**엔터프라이즈 샘플**)

봇이 새 메시지를 수신할 때 기록됩니다.

- UserID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ConversationID ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- 채널  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityType  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- 텍스트(PII의 경우 선택 사항)
- FromId
- FromName
- RecipientId
- RecipientName
- ConversationId
- ConversationName
- 로캘

### <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
**로그인 출처:** TelemetryLoggerMiddleware(**엔터프라이즈 샘플**)

봇이 메시지를 보낼 때 기록됩니다.

- UserID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ConversationID ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- 채널  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityType  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ReplyToID
- 채널  (소스 채널 - 예: Skype, Cortana, Teams)
- RecipientId
- ConversationName
- 로캘
- 텍스트(PII의 경우 선택 사항)
- RecipientName(PII의 경우 선택 사항)

### <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
**로그인 출처:** TelemetryLoggerMiddleware는 봇에서 메시지를 업데이트할 때 기록됩니다(희귀 사례).

### <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**로그인 출처:** TelemetryLoggerMiddleware는 봇에서 메시지를 삭제할 때 기록됩니다(희귀 사례).

### <a name="customevent-luisintentinentname"></a>CustomEvent: LuisIntent.INENTName 
**로그인 출처:** TelemetryLuisRecognizer(**엔터프라이즈 샘플**)

LUIS 서비스의 결과를 기록합니다.

- UserID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ConversationID ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- 채널  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityType  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- 의도
- IntentScore
- 질문
- ConversationId
- SentimentLabel
- SentimentScore
- *LUIS 엔터티*
- **신규** DialogId

### <a name="customevent-qnamessage"></a>CustomEvent: QnAMessage
**로그인 출처:** TelemetryQnaMaker(**엔터프라이즈 샘플**)

QnA Maker 서비스의 결과를 기록합니다.

- UserID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ConversationID ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityID  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- 채널  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- ActivityType  ([원격 분석 이니셜라이저에서](https://aka.ms/telemetry-initializer))
- 사용자 이름
- ConversationId
- OriginalQuestion
- 질문
- 응답
- 점수(*선택 사항*: 정보를 찾은 경우)

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

