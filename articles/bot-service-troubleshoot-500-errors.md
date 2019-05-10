---
title: 봇 HTTP 500 오류 해결 | Microsoft Docs
description: 배포된 봇에서 HTTP 500 오류를 해결하는 방법입니다.
keywords: 문제 해결, HTTP 500, 문제.
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/30/2019
ms.openlocfilehash: 93689b7cee1c89bd9a7079c15ddf6aa16fcacc26
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033077"
---
# <a name="troubleshoot-http-500-errors"></a>HTTP 500 오류 해결

500 오류 해결의 첫 번째 단계는 Application Insights를 사용 설정하는 것입니다.

<!-- TODO: Add links back in once there's a fresh AppInsights sample.
The luis-with-appinsights ([C# sample](https://aka.ms/cs-luis-with-appinsights-sample) / [JS sample](https://aka.ms/js-luis-with-appinsights-sample)) and qna-with-appinsights ([C# sample](https://aka.ms/qna-with-appinsights) / [JS sample](https://aka.ms/js-qna-with-appinsights-sample)) samples demonstrate bots that support Azure Application Insights.
-->
기존 봇에 Application Insights를 추가하는 방법에 대한 자세한 내용은 [대화 분석 텔레메트리](https://aka.ms/botframeworkanalytics)를 참조하세요.

## <a name="enable-application-insights-on-aspnet"></a>ASP.Net에서 Application Insights 사용

기본 Application Insights 지원의 경우 [ASP.NET 웹 사이트의 Application Insights를 설정](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net)하는 방법을 참조하세요. Bot Framework(v4.2부터)는 추가 수준의 Application Insights 텔레메트리를 제공하지만 HTTP 500 오류를 진단하는데 필수는 아닙니다.

## <a name="enable-application-insights-on-nodejs"></a>Node.js에서 Application Insights 사용

기본 Application Insights 지원의 경우 [Application Insights를 사용하여 Node.js 서비스 및 앱을 모니터링](https://docs.microsoft.com/azure/azure-monitor/learn/nodejs-quick-start)하는 방법을 참조하세요. Bot Framework(v4.2부터)는 추가 수준의 Application Insights 텔레메트리를 제공하지만 HTTP 500 오류를 진단하는데 필수는 아닙니다.

## <a name="query-for-exceptions"></a>예외 쿼리

HTTP 상태 코드 500 오류를 분석하는 가장 쉬운 방법은 예외로 시작하는 것입니다.

다음 쿼리는 가장 최근의 예외를 알려줍니다.

```sql
exceptions
| order by timestamp desc
| project timestamp, operation_Id, appName
```

첫 번째 쿼리에서 작업 ID 몇 개를 선택하고 추가 정보를 찾아보세요.

```sql
let my_operation_id = "d298f1385197fd438b520e617d58f4fb";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};

union_all
    | order by timestamp desc
```

`exceptions`만 있을 경우 세부 정보를 분석하고 코드 줄에 해당하는지 확인하세요. Channel Connector(`Microsoft.Bot.ChannelConnector`)에서 발생하는 예외만 보일 경우 [Application Insights 이벤트 없음](#no-application-insights-events)을 참조하여 Application Insights가 올바르게 설정되어 있고 코드가 이벤트를 로깅하고 있는지 확인하세요.

## <a name="no-application-insights-events"></a>Application Insights 이벤트

500 오류가 발생하였고 Application Insights 내에서 봇의 추가 이벤트가 수신되지 않을 경우 다음을 확인하세요.

### <a name="ensure-bot-runs-locally"></a>봇이 로컬에서 실행되는지 확인

먼저 에뮬레이터를 사용하여 봇이 로컬에서 실행되는지 확인하세요.

### <a name="ensure-configuration-files-are-being-copied-net-only"></a>구성 파일이 복사되는지 확인(.NET만 해당)

배포 프로세스 중에 `appsettings.json` 및 기타 구성 파일이 올바르게 패키징되어 있는지 확인하세요.

#### <a name="application-assemblies"></a>애플리케이션 어셈블리

배포 프로세스 중에 Application Insights 어셈블리가 올바르게 패키징되는지 확인하세요.

- Microsoft.ApplicationInsights
- Microsoft.ApplicationInsights.TraceListener
- Microsoft.AI.Web
- Microsoft.AI.WebServer
- Microsoft.AI.ServeTelemetryChannel
- Microsoft.AI.PerfCounterCollector
- Microsoft.AI.DependencyCollector
- Microsoft.AI.Agent.Intercept

배포 프로세스 중에 `appsettings.json` 및 기타 구성 파일이 올바르게 패키징되어 있는지 확인하세요.

#### <a name="appsettingsjson"></a>appsettings.json

`appsettings.json` 파일 내에 계측 키가 설정되어 있는지 확인하세요.

## <a name="aspnet-web-apitabdotnetwebapi"></a>[ASP.NET Web API](#tab/dotnetwebapi)

```json
{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Debug",
            "System": "Information",
            "Microsoft": "Information"
        },
        "Console": {
            "IncludeScopes": "true"
        }
    }
}
```

## <a name="aspnet-coretabdotnetcore"></a>[ASP.NET Core](#tab/dotnetcore)

```json
{
    "ApplicationInsights": {
        "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
}
```

---

### <a name="verify-config-file"></a>구성 파일 확인

구성 파일에 Application Insights 키가 포함되어 있는지 확인하세요.

```json
{
    "ApplicationInsights": {
        "type": "appInsights",
        "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "resourceGroup": "my resource group",
        "name": "my appinsights name",
        "serviceName": "my service name",
        "instrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "applicationId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "apiKeys": {},
        "id": ""
    }
},
```

### <a name="check-logs"></a>로그 확인

Bot ASP.Net 및 Node는 검사할 수 있는 서버 수준에서 로그를 내보냅니다.

#### <a name="set-up-a-browser-to-watch-your-logs"></a>브라우저를 설정하여 로그 확인

1. [Azure Portal](http://portal.azure.com/)에서 봇을 엽니다.
1. **App Service 설정/모든 App Service 설정** 페이지를 열어 모든 서비스 설정을 확인합니다.
1. App Service의 **모니터링/진단 로그** 페이지를 엽니다.
   - **애플리케이션 로깅(파일 시스템)** 이 사용하도록 설정되어 있는지 확인합니다. 이 설정을 변경할 경우 **저장**을 클릭하세요.
1. **모니터링/로그 스트림** 페이지로 전환합니다.
   - **웹 서버 로그**를 선택하고 연결된 메시지를 확인합니다. 다음과 유사해야 합니다.

     ```bash
     Connecting...
     2018-11-14T17:24:51  Welcome, you are now connected to log-streaming service.
     ```

     이 창을 계속 열어 두세요.

#### <a name="set-up-browser-to-restart-your-bot-service"></a>봇 서비스를 다시 시작하도록 브라우저 설정

1. 별도의 브라우저를 사용하여 Azure Portal에서 봇을 엽니다.
1. **App Service 설정/모든 App Service 설정** 페이지를 열어 모든 서비스 설정을 확인합니다.
1. App Service의 **개요** 페이지로 전환하고 **다시 시작**을 클릭합니다.
   - 확인 메시지가 표시되면 **예**를 선택합니다.
1. 첫 번째 브라우저 창으로 돌아가서 로그를 확인합니다.
1. 새 로그가 수신되는지 확인합니다.
   - 활동이 없을 경우 봇을 다시 배포하세요.
   - 그런 다음, **애플리케이션 로그** 페이지로 전환하고 오류가 있는지 확인하세요.
