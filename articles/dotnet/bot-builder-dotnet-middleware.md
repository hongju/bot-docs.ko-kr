---
title: 메시지 가로채기 | Microsoft Docs
description: .NET용 Bot Framework SDK를 사용하여 사용자와 봇 간에 메시지를 가로채는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/01/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0e8873d2914d42b928004c31c14c8d60cb35b2a3
ms.sourcegitcommit: 4139ef7ebd8bb0648b8af2406f348b147817d4c7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/19/2019
ms.locfileid: "58073769"
---
# <a name="intercept-messages"></a>메시지 가로채기

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introducton to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="intercept-and-log-messages"></a>메시지 가로채기 및 로그

다음 코드 샘플은 .NET용 Bot Framework SDK의 **미들웨어** 개념을 사용하여 사용자와 봇 간에 교환되는 메시지를 가로채는 방법을 보여줍니다. 

첫째, `DebugActivityLogger` 클래스를 만들고 `LogAsync` 메서드를 정의하여 가로챈 각 메시지에 대해 수행할 작업을 지정합니다. 이 예제에서는 각 메시지에 대한 일부 정보만 인쇄합니다.

```cs
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
    }
}
```

그런 다음, 다음 코드를 `Global.asax.cs`에 추가합니다.  사용자와 봇 간 교환되는(어느 방향으로든) 모든 메시지는 이제 `DebugActivityLogger` 클래스의 `LogAsync` 메서드를 트리거하게 됩니다. 

```cs
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency();
            builder.Update(Conversation.Container);

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }
    }
```

이 예제에서는 각 메시지에 대한 일부 정보만 간단히 인쇄하지만, `LogAsync` 메서드를 업데이트하여 각 메시지에 대해 수행할 작업을 지정할 수 있습니다. 

## <a name="sample-code"></a>샘플 코드 

.NET용 Bot Framework SDK를 사용하여 메시지를 가로채는 방법을 보여주는 전체 샘플은 GitHub의 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-Middleware" target="_blank">미들웨어 샘플</a>을 참조하세요. 

## <a name="additional-resources"></a>추가 리소스

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Framework SDK 참조</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-Middleware" target="_blank">미들웨어 샘플(GitHub)</a>
