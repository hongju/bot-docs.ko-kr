---
title: 고유한 미들웨어 만들기 | Microsoft Docs
description: 미들웨어를 직접 작성하는 방법을 이해합니다.
keywords: 미들웨어, 사용자 지정 미들웨어, 단락, 대체, 활동 처리기
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78c0afd6095cf9a10e04e1b0131d66ce6964f369
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000200"
---
# <a name="create-your-own-middleware"></a>고유한 미들웨어 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

미들웨어를 사용하면 봇을 위해 풍부한 플러그인을 작성할 수 있고, 다른 사람도 사용할 수 있습니다. 이 문서에서는 기본 미들웨어를 추가하고 구현하는 방법과 어떻게 작동하는지를 보여줍니다. v4 SDK에는 상태 관리, LUIS, QnAMaker 및 변환 등과 같은 몇 가지 미들웨어가 제공됩니다. 자세한 내용은 [.NET](https://github.com/Microsoft/botbuilder-dotnet) 또는 [JavaScript](https://github.com/Microsoft/botbuilder-js)용 Bot Builder SDK를 참조하세요.

## <a name="adding-middleware"></a>미들웨어 추가

아래 예제에서는 [시작](~/bot-service-quickstart.md) 환경을 통해 만든 기본 봇 샘플을 기반으로 두 가지 미들웨어가 각 클래스의 새 인스턴스와 함께 서비스에 추가됩니다.

> [!IMPORTANT]
> 옵션에 추가되는 순서에 따라 옵션이 실행되는 순서가 결정됩니다. 둘 이상의 미들웨어 조각을 사용하는 경우 어떻게 작동할지를 고려해야 합니다.

**Startup.cs**

# <a name="ctabcsaddmiddleware"></a>[C#](#tab/csaddmiddleware)

추가하려는 각 미들웨어 조각의 봇 서비스 옵션마다 `options.Middleware.Add(new MyMiddleware());` 메서드 호출을 추가합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<HelloBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new MyMiddleware());
        options.Middleware.Add(new MyOtherMiddleware());
    });
}
```
# <a name="javascripttabjsaddmiddleware"></a>[JavaScript](#tab/jsaddmiddleware)

추가하려는 각 미들웨어 조각의 어댑터에 `adapter.use(MyMiddleware());`를 추가합니다.

```javascript
// Create adapter
const adapter = new botbuilder.BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

adapter.use(MyMiddleware());
adapter.use(MyOtherMiddleware());
```

---


## <a name="implementing-your-middleware"></a>미들웨어 구현

각 미들웨어 조각은 미들웨어 인터페이스를 상속하고 봇에 전송되는 모든 활동에서 실행되는 프로세싱 처리기를 항상 구현합니다. 프로세싱 처리기는 파이프라인을 따라 진행하면서 다른 미들웨어나 봇 논리가 컨텍스트 개체와 상호 작용하도록 허용하기 전에 추가된 각 미들웨어 조각에 대해 컨텍스트 개체를 수정하거나 작업(예: 로깅)을 수행할 기회를 얻습니다.

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)

각 미들웨어 조각은 `IMiddleware`에서 상속되며 `OnTurn()`을 항상 구현합니다.

**ExampleMiddleware.cs**
```csharp
public class MyMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {            
        // This simple middleware reports the request type and if we responded
        await context.SendActivity($"Request type: {context.Activity.Type}");
        
        await next();            

        // Report if any responses were recorded
        string response = context.Responded ? "yes" : "no";
        await context.SendActivity($"Responded?  {response}");
    }
}

public class MyOtherMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        // simple middleware to add an additional send activity
        await context.SendActivity($"My other middleware just saying Hi before the bot logic");

        await next();
    }
}

```

# <a name="javascripttabjsimplementmiddleware"></a>[JavaScript](#tab/jsimplementmiddleware)

각 미들웨어 조각은 `MiddlewareSet`에서 상속되며 `onTurn()`을 항상 구현합니다.

**ExampleMiddleware.js**
```js
adapter.use({onTurn: async (context, next) =>{

    // This simple middleware reports the activity type and if we responded
    await context.sendActivity(`Activity type: ${context.activity.type}`); 
    await next();            

    // Report if any responses were recorded
    const response = context.activity.text ? "yes" : "no";
    await context.sendActivity(`Responded?  ${response}`);

}}, {onTurn: async (context, next) => {

     // simple middleware to add an additional send activity
     await context.sendActivity("My other middleware just saying Hi before the bot logic");

     await next();
}})
```

---

`next()`를 호출하면 실행이 다음 미들웨어 조각으로 계속 진행됩니다. 실행이 전달되는 시점을 선택할 수 있으면 나머지 미들웨어 스택이 실행된 **후**에 실행되는 코드를 작성할 수 있습니다. 봇 논리 및 기타 미들웨어가 실행된 다음, 프로세싱 처리기의 '후행 에지'에 대한 조치를 취할 수 있습니다.  위의 예제에서 첫 번째로 구현된 미들웨어가 바로 이 작업을 수행합니다. 즉, 실행을 파이프라인으로 되돌리기 전에 이 컨텍스트 개체에 대해 응답했는지 여부를 보고합니다.

## <a name="short-circuit-routing"></a>단락 라우팅

경우에 따라 수신한 활동에 대한 이후 프로세싱을 중지할 수도 있으며, 이것을 단락이라고 합니다. 이것은 미들웨어가 요청을 완벽하게 처리하거나, 특정 명령에 대해 쉽게 응답을 제공하거나, 봇 논리가 살펴볼 필요 없이 들어오는 요청을 처리할 수 있는 경우에 유용합니다.

사용자가 "핑(ping)"이라고 할 때마다 응답을 보내고 요청을 더 이상 라우팅하지 못하게 하는 미들웨어 조각을 만들어보겠습니다.

# <a name="ctabcsmiddlewareshortcircuit"></a>[C#](#tab/csmiddlewareshortcircuit)
```cs
public class ExampleMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        var utterance = context.Activity?.AsMessageActivity()?.Text.Trim().ToLower();

        if (utterance == "ping") 
        {
            context.SendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }
    }
}
```
# <a name="javascripttabjsmiddlewareshortcircuit"></a>[JavaScript](#tab/jsmiddlewareshortcircuit)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    const utterance = (context.activity.text || '').trim().toLowerCase();
        if (utterance == "ping") 
        {
            await context.sendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }

}})

```

---

## <a name="fallback-processing"></a>대체 프로세싱

조치가 필요할 수도 있는 또 다른 동작은 아직 응답하지 않은 요청에 응답하는 것입니다. 이 동작은 `context.Responded` 속성을 확인하고 프로세싱 처리기의 후행 에지를 사용하여 쉽게 수행할 수 있습니다. 봇이 요청을 처리하기 못하면 경우 “이해하지 못했습니다.”라고 자동으로 말하는 간단한 미들웨어 조각을 만들어보겠습니다.

# <a name="ctabcsfallback"></a>[C#](#tab/csfallback)
```cs
public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
{
    await next();

    if (!context.Responded) 
    {
        context.SendActivity("I didn't understand.");
    }
}
```
# <a name="javascripttabjsfallback"></a>[JavaScript](#tab/jsfallback)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    await next();

    if (!context.responded) 
    {
       await context.sendActivity("I didn't understand.");
    }

}})
```

---

> [!NOTE] 
> 다른 미들웨어가 사용자에게 응답할 수 있거나 봇이 메시지를 제대로 수신하고 응답하지 않는 경우 등과 같이 모든 경우에 맞지 않을 수 있습니다. “이해하지 못했습니다.”라고 응답하면 사용자에게 오해를 일으킬 수 있습니다.


