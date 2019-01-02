---
title: 봇의 분석 | Microsoft Docs
description: 봇의 여러 부분과 작동 방식 분석
keywords: ''
author: ivorb
ms.author: ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 238be22eff746edff30a5446d20991dfaedc945a
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928291"
---
# <a name="anatomy-of-a-bot"></a>봇의 분석

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

가장 [기본적인 의미](bot-builder-basics.md)의 봇은 사용자가 메시지를 사용하여 통신하는 대화형 앱입니다. 관련 언어로 작성된 웹앱의 기본 구조를 따르며, 이를 기반으로 Bot Framework SDK와 함께 빌드됩니다.

봇은 여러 구성 요소로 구성되어 [메시지를 보내고](./bot-builder-howto-send-messages.md) [상태](./bot-builder-storage-concept.md)를 추적할 수 있습니다. [에뮬레이터](../bot-service-debug-emulator.md), [WebChat](../bot-service-manage-test-webchat.md) 또는 [채널](bot-concepts.md) 중 하나를 통해 프로덕션 환경에서 봇과 통신할 수 있습니다. 

여기에서는 기본적인 Echo 봇을 단계별로 살펴보고, 함께 작동하는 각 부분을 검사합니다.

## <a name="files-created-for-echo-bot"></a>Echo 봇용으로 생성된 파일

각 프로그래밍 언어는 Echo 봇에 대해 서로 다른 파일을 만듭니다. 이 예제에서는 **BasicEcho**라고 명명했으며, 이러한 파일은 시스템 섹션, 도우미 항목 및 봇의 코어라는 세 가지 섹션으로 구분됩니다. 아래 표에는 C# 및 JavaScript에 대한 기본 파일이 나열되어 있습니다.

| C# | JavaScript |
| --- | --- |
| `Properties > launchSettings.json` <br> `wwwroot > default.htm` <br> `appsettings.json` <br> `BasicEcho.bot` <br> `EchoBot.cs` <br> `EchoState.cs` <br> `Program.cs` <br> `readme.md` <br> `Startup.cs` | `.env` <br> `app.js` <br> `package.json` <br> `README.md` <br> `basicEcho.bot` |

아래에서는 이러한 파일의 몇 가지 주요 기능을 섹션별로 살펴봅니다. 이러한 파일 중 일부는 봇 전용 및 일반 프로그래밍 라이브러리인 자체 포함 집합을 가지고 있습니다. 여기에는 필요한 함수 집합뿐만 아니라 애플리케이션에 필요할 수도 있는 몇 가지 함수 집합이 포함됩니다. 이에 대한 자세한 내용은 살펴보지 않겠지만, 궁금한 부분이 있다면 Visual Studio를 사용하여 정의를 확인하고, 어떤 네임스페이스에 해당하는지 확인하세요.

## <a name="system-section"></a>시스템 섹션

시스템 섹션에는 봇 함수를 표준 애플리케이션으로 작동시키는 데 필요한 항목이 있습니다. 여기에는 구성 파일, 어댑터 및 일부 JSON 파일이 포함됩니다. 이러한 항목은 특정 정보가 필요할 수 있는 일부 구성 파일을 제외하고 그대로 둘 수 있습니다(그대로 두는 경우가 많음).

# <a name="ctabcs"></a>[C#](#tab/cs)

봇은 [ASP.NET Core 웹](https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-2.1) 애플리케이션 프레임워크의 한 유형입니다. [ASP.NET 기본 사항](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x)을 보면 아래에 설명된 appsettings.json, Program.cs 및 Startup.cs와 같은 파일에서 유사한 코드를 확인할 수 있습니다. 이러한 파일은 모든 웹앱에 필요하며 봇 전용이 아닙니다. 이러한 파일 중 일부 파일의 코드는 여기에 복사되지 않지만, 봇을 실행할 때 표시됩니다.

### <a name="appsettingsjson"></a>appsettings.json

이 파일에는 봇용 설정 정보(주로 앱 ID 및 암호)가 기본 JSON 형식으로 포함되어 있습니다.  [에뮬레이터](../bot-service-debug-emulator.md)로 테스트하는 경우에는 이러한 정보가 필요하지 않으며 비워 둘 수 있지만, 프로덕션 환경에서는 필요합니다.

### <a name="programcs"></a>Program.cs

이 파일은 ASP.NET 웹앱을 작동하는 데 필요하며, 실행할 `Startup` 클래스를 지정합니다.

### <a name="startupcs"></a>Startup.cs

여기서는 **Startup.cs**의 시스템 섹션을 살펴보겠지만, 다른 흥미로운 부분은 나중에 살펴볼 예정입니다.

`Startup`의 생성자는 앱 설정 및 환경 변수를 지정하고 웹앱에 대한 구성을 만듭니다.

`Configure` 메서드는 Bot Framework 및 몇 가지 다른 파일을 사용하도록 지정하여 앱의 구성을 완료합니다. Bot Framework를 사용하는 모든 봇은 해당 구성 호출이 필요합니다.

```cs
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace BasicEcho
{
    public class Startup
    {
        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();

            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }

        // ...
        // Definition of ConfigureServices covered in the next section
        // ...

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
        }
    }
}

```

### <a name="launchsettingsjson-and-readmemd"></a>launchSettings.json 및 readme.md

**launchSettings.json**에는 웹앱에 대한 몇 가지 설정 구성이 포함되어 있으며 **readme.md**는 echo 봇에 대한 간단한 한 줄 설명입니다. 이러한 파일의 내용은 이 봇을 이해하는 데 중요하지 않습니다. 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

시스템 섹션에는 주로 **package.json**, **.env** 파일, **app.js** 및 **README.md**가 포함됩니다. 일부 파일의 코드는 여기에 복사되지 않지만, 봇을 실행할 때 표시됩니다.

### <a name="packagejson"></a>package.json

**package.json**은 봇의 종속 항목 및 관련 버전을 지정합니다. 이는 모두 템플릿과 시스템에 의해 설정됩니다.

### <a name="env-file"></a>.env 파일

**.env** 파일은 포트 번호, 앱 ID 및 암호 등과 같은 봇의 구성 정보를 지정합니다. 특정 기술을 사용하거나 프로덕션 환경에서 이 봇을 사용하는 경우, 이 구성에 특정 키 또는 URL을 추가해야 합니다. 그러나 이 Echo 봇의 경우에는 이 시점에 아무 작업도 수행할 필요가 없습니다. 이 때 앱 ID와 암호는 정의되지 않은 상태로 남겨둘 수 있습니다. 

**.env** 구성 파일을 사용하려면 템플릿에 추가 패키지가 포함되어 있어야 합니다.  먼저 npm에서 `dotenv` 패키지를 가져옵니다.

`npm install dotenv`

그런 다음, 다른 필요한 라이브러리를 사용하여 봇에 다음 줄을 추가합니다.

```javascript
const dotenv = require('dotenv');
```

### <a name="appjs"></a>app.js

`app.js` 파일의 상단에는 봇이 사용자와 통신하고 응답을 보낼 수 있게 해주는 서버와 어댑터가 있습니다. 서버는 **.env** 구성의 지정된 포트에서 수신 대기하거나, 에뮬레이터와의 연결을 위해 3978로 폴백합니다. 어댑터는 봇의 지휘자 역할을 하며 들어오고 나가는 통신, 인증 등을 지시합니다. 

```javascript
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
``` 

### <a name="readmemd"></a>README.md

**README.md**는 기본 봇 구조 또는 *대화 상자*의 정의와 같은 봇 빌드의 몇 가지 측면에 대한 유용한 정보를 제공하지만, 봇을 이해하는 데 중요하지 않습니다.

---


## <a name="helper-items"></a>도우미 항목

다른 프로그램과 마찬가지로 도우미 메서드와 다른 정의를 사용하면 코드를 간소화할 수 있습니다. `.bot` 파일과 같이 해당 범주에 속하는 봇의 몇 가지 부분은 봇에 중요하지만, 반드시 핵심 봇 논리의 일부는 아닙니다.

### <a name="bot-file"></a>.bot 파일

`.bot` 파일에는 [채널](bot-concepts.md)을 봇에 연결할 수 있게 해주는 엔드포인트, 앱 ID 및 암호 등의 정보가 포함되어 있습니다. 이 파일은 템플릿에서 봇을 빌드할 때 생성되지만 에뮬레이터 또는 다른 도구를 통해 직접 만들 수 있습니다.

파일 컨텐츠는 여기에 표시된 내용과 일치해야 합니다. 단, 이름은 *BasicEcho*로 지정해야 합니다. 봇의 사용 방법에 따라 이 정보를 변경하고 업데이트해야 할 수도 있지만, 이 Echo 봇을 실행하는 경우에는 지금 당장 변경 작업을 수행할 필요가 없습니다.

```json
{
  "name": "BasicEcho",
  "secretKey": "",
  "services": [
    {
      "appId": "",
      "id": "http://localhost:3978/api/messages",
      "type": "endpoint",
      "appPassword": "",
      "endpoint": "http://localhost:3978/api/messages",
      "name": "BasicEcho"
    }
  ]
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="echostatecs"></a>EchoState.cs

**EchoState.cs**에는 봇이 현재 상태를 유지하기 위해 사용하는 간단한 클래스가 포함됩니다. 턴 카운터를 증가시키기 위해 사용하는 `int`만 포함됩니다. 자세한 내용은 다음 섹션에서 살펴볼 예정이며, 지금은 `EchoState`가 턴 카운트를 포함하는 클래스임을 이해하기만 하면 됩니다.

```cs
namespace BasicEcho
{
    /// <summary>
    /// Class for storing conversation state.
    /// </summary>
    public class EchoState
    {
        public int TurnCount { get; set; } = 0;
    }
}
```

### <a name="defaulthtm"></a>default.htm

**default.htm**은 봇을 실행할 때 표시되는 웹 페이지로, `html` 형식으로 작성됩니다. 이 페이지에는 봇에 연결하는 방법과 상호 작용하는 방법에 대한 유용한 정보가 표시되지만, 페이지의 내용은 봇의 동작에 영향을 주지 않습니다. 봇을 실행할 때 코드 팝업이 표시됩니다.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="required-libraries"></a>필수 라이브러리

`app.js` 파일의 맨 위쪽에는 필요한 일련의 모듈 또는 라이브러리가 있습니다. 이러한 모듈은 사용자가 애플리케이션에 포함하는 함수 집합에 액세스할 수 있습니다. 

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');
```

---

## <a name="core-of-the-bot"></a>봇의 코어

마지막으로, 사용자가 정말 관심 있어 하는 부분을 살펴보겠습니다. 봇의 코어는 미들웨어 및 봇 논리 등 사용자와 상호 작용하는 방법을 정의합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="middleware"></a>미들웨어

**Startup.cs** 내에는 `ConfigureServices`라는 하나의 메서드가 있습니다. 이 메서드는 자격 증명 공급자를 설정하고 미들웨어를 추가합니다.

여기에 추가된 [미들웨어](bot-builder-concept-middleware.md)는 두 가지 작업을 수행합니다. 첫 번째는 봇이 정상적으로 실패하도록 하는 예외 처리입니다. 이는 람다 식으로 인라인 정의되며, 간단하게 터미널에 예외를 출력하고 사용자에게 오류가 발생되었음을 알립니다.

두 번째 미들웨어는 바로 앞에 정의된 `MemoryStorage`를 사용하여 상태를 유지합니다. 이 상태는 `ConversationState`로 정의됩니다. 이는 대화의 상태를 유지한다는 것을 의미합니다. 사용자의 턴 카운터에 정의된 클래스인 `EchoState`를 사용하여 관심 있는 정보를 저장합니다.

``` cs
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot.
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facilitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent.
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // The File data store, shown here, is suitable for bots that run on 
        // a single machine and need durable state across application restarts.

        // IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());

        // For production bots use the Azure Table Store, Azure Blob, or 
        // Azure CosmosDB storage provides, as seen below. To include any of 
        // the Azure based storage providers, add the Microsoft.Bot.Builder.Azure 
        // Nuget package to your solution. That package is found at:
        //      https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/

        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureTableStorage("AzureTablesConnectionString", "TableName");
        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureBlobStorage("AzureBlobConnectionString", "containerName");

        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
```

### <a name="bot-logic"></a>봇 논리

주 봇 논리는 봇의 `OnTurn` 처리기 내에 정의됩니다. `OnTurn`에는 들어오는 [활동](bot-builder-concept-activity-processing.md), 대화 등에 대한 정보를 제공하는 인수인 [컨텍스트](./bot-builder-concept-activity-processing.md#turn-context) 변수가 제공됩니다.

들어오는 활동은 [형식이 다양](../bot-service-activities-entities.md#activity-types)할 수 있으므로 먼저 봇이 메시지를 받았는지 확인합니다. 메시지인 경우 봇 대화의 현재 정보를 포함하는 상태 변수를 만듭니다. 그런 다음, 사용자에게 보낸 메시지와 함께 카운트를 다시 반환하기 전에 턴 카운트를 추적하는 `int`를 증가시킵니다.

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace BasicEcho
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Every Conversation turn for our EchoBot will call this method. In here
        /// the bot checks the Activty type to verify it's a message, bumps the 
        /// turn conversation 'Turn' count, and then echoes the users typing
        /// back to them. 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {
                // Get the conversation state from the turn context
                var state = context.GetConversationState<EchoState>();

                // Bump the turn count. 
                state.TurnCount++;

                // Echo back to the user whatever they typed.
                await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
            }
        }
    }    
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="middleware"></a>미들웨어

**app.js** 내에서 상단의 `botbuilder`에 필요한 모듈 중 하나로 정의된 `MemoryStorage`를 통해 이 봇의 [미들웨어](bot-builder-concept-middleware.md)를 사용하여 상태를 유지합니다. 이 상태는 `ConversationState`로 정의됩니다. 이는 대화의 상태를 유지한다는 것을 의미합니다. `ConversationState`는 사용자가 관심 있어 하는 정보를 메모리에 저장합니다(이 경우에는 턴 카운터만 메모리에 저장됨).

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

### <a name="bot-logic"></a>봇 논리

*processActivity* 내의 세 번째 매개 변수는 수신된 [활동](bot-builder-concept-activity-processing.md)이 어댑터에 의해 사전 처리되고 모든 미들웨어를 통해 라우팅된 후 봇의 논리를 수행하기 위해 호출되는 함수 처리기입니다. 함수 처리기에 인수로 전달된 [context](./bot-builder-concept-activity-processing.md#turn-context) 변수는 수신 활동, 보낸 사람 및 받는 사람, 채널, 대화 등에 대한 정보를 제공하는 데 사용될 수 있습니다.

먼저 봇 메시지를 받았는지 확인합니다. 봇이 메시지를 받지 못하면 수신된 활동 유형을 다시 반환합니다. 다음으로, 봇 대화의 정보를 포함하는 상태 변수를 만듭니다. 그런 다음, 정의되지 않은 경우(봇을 시작할 때 발생) 카운트 변수를 0으로 설정하거나 새 메시지마다 증가시킵니다. 마지막으로, 사용자에게 보낸 메시지와 함께 카운트를 다시 반환합니다.

```javascript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {

            // Get the conversation state
            const state = conversationState.get(context);

            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---
