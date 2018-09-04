---
title: 사전 대응 메시지 보내기 | Microsoft Docs
description: 봇을 통해 사전에 메시지를 보내는 방법을 이해합니다.
keywords: 사전 대응 메시지
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/01/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c22ce6a35d4d49506360a78a439f15137c429d9d
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905137"
---
# <a name="send-proactive-messages"></a>사전 대응 메시지 보내기 

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]


봇은 _반응(reactive) 메시지_를 보내는 경우가 대부분이지만 [사전 대응 메시지](bot-builder-proactive-messages.md)를 보내야 하는 경우도 있습니다. 

사전 대응 메시지의 일반적인 사례는 봇이 확정되지 않은 시간 동안 작업을 수행하는 경우입니다. 이 경우 작업에 대한 정보를 저장하고, 사용자에게는 작업이 끝나면 봇이 돌아온다고 알리고, 대화가 진행되도록 할 수 있습니다. 작업이 완료되면 봇은 사전에 확인 메시지를 보내서 대화를 재개할 수 있습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="notes-about-this-sample"></a>이 샘플에 대한 정보

기본적인 EchoBot 샘플은 수정되고 있습니다.
- `Microsoft.Samples.Proactive`가 네임스페이스로 사용됩니다.
- 상태 파일은 `JobData.cs` 파일로 대체됩니다.
- 봇 파일은 `ProactiveBot.cs` 파일로 대체됩니다.

> [!NOTE]
> 사전 대응 메시지를 사용하려면 유효한 ApplicationID와 암호가 봇에 있어야 합니다.


## <a name="define-task-data"></a>작업 데이터 정의

이 시나리오에서는 다양한 사용자가 여러 대화에서 만들 수 있는 임의의 작업을 추적합니다. 따라서 사용자 또는 대화 상태 미들웨어 대신 일반적인 봇 상태 미들웨어를 사용합니다.

다음 클래스는 개별 작업에 사용할 데이터 구조를 정의합니다.


```csharp
using Microsoft.Bot.Schema;
using System.Collections.Generic;

namespace Microsoft.Samples.Proactive
{
    /// <summary>
    /// Class for storing job state. 
    /// </summary>
    public class JobData
    {
        /// <summary>
        /// The name to use to read and write this bot state object to storage.
        /// </summary>
        public readonly static string PropertyName = $"BotState:{typeof(Dictionary<int, JobData>).FullName}";

        public int JobNumber { get; set; } = 0;
        public bool Completed { get; set; } = false;

        /// <summary>
        /// The conversation reference to which to send status updates.
        /// </summary>
        public ConversationReference Conversation { get; set; }
    }
}
```


또한 시작 코드에 상태 미들웨어를 추가해야 합니다.


`StartUp.cs` 파일에서 `ConfigureServices` 메서드를 업데이트하여 작업 사전을 봇 상태에 추가합니다. 다음 코드에서 `options.Middleware.Add`에 대한 마지막 호출입니다.
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<ProactiveBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot. 
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facillitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent. 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(ProactiveBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // Using the base BotState here, since the job log is not necessarily tied to a
        // specific user or conversation.
        options.Middleware.Add(
            new BotState<Dictionary<int, JobData>>(
                dataStore, JobData.PropertyName, (context) => $"jobs/{typeof(Dictionary<int, JobData>)}"));
    });
}
```


## <a name="update-your-bot-to-create-and-run-jobs"></a>작업을 만들고 실행하도록 봇을 업데이트

각 턴(turn)에 `run` 또는 `run job`을 입력하여 사용자가 작업을 만들 수 있도록 합니다.

이에 대한 응답으로 봇은 해당 턴(turn) 내에서 다음 단계를 수행합니다.
- 작업을 만듭니다.
- 사전 대응 메시지를 나중에 보낼 수 있도록 현재 대화에 대한 정보를 기록합니다.
- 작업이 시작되었고 완료되면 알려줄 것이라고 사용자에게 알려줍니다.
- 비동기 작업을 시작합니다.
- 턴(turn)이 종료되도록 합니다.

시작하는 작업은 간단한 5초 타이머이며 사전 대응 메시지를 보내면 완료됩니다.
- 어댑터의 대화 계속 메서드를 호출하면 봇에 의해 시작되는 새로운 턴(turn)이 만들어집니다.
- 이 턴(turn)에는 자체적인 [턴(turn) 컨텍스트](bot-builder-concept-activity-processing.md#turn-context)가 있으며 여기에서 상태 정보를 검색합니다.
- 이 컨텍스트를 사용하여 사용자에게 사전 대응 메시지를 보냅니다.



> [!NOTE]
> `GetAppId` 메시지는 .NET SDK에서 사전 대응 메시지를 가능하게 하는 해결 방법입니다.

```csharp
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Bot.Schema;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Security.Claims;
using System.Security.Principal;
using System.Threading;
using System.Threading.Tasks;

namespace Microsoft.Samples.Proactive
{
    public class ProactiveBot : IBot
    {
        /// <summary>
        /// Random number generator for job numbers.
        /// </summary>
        private static Random NumberGenerator = new Random();

        /// <summary>
        /// Gets the job log from the bot state.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The job log.</returns>
        private static Dictionary<int, JobData> GetJobLog(ITurnContext context)
        {
            return context.Services.Get<Dictionary<int, JobData>>(JobData.PropertyName);
        }

        /// <summary>
        /// Workaround to get the bot's app ID.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The application ID for the bot.</returns>
        private static string GetAppId(ITurnContext context)
        {
            // The BotFrameworkAdapter sets the identity provider on the context object.
            var claimsIdentity = context.Services.Get<IIdentity>("BotIdentity") as ClaimsIdentity;

            // For requests from a channel, the app ID is in the Audience claim of the JWT token.
            // For requests from the emulator, it is in the AppId claim.
            // For unauthenticated requests, we have anonymouse identity provided auth is disabled.
            // For Activities coming from Emulator AppId claim contains the Bot's AAD AppId.
            var botAppIdClaim =
                (claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AudienceClaim)
                ?? claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AppIdClaim));

            return botAppIdClaim?.Value;
        }

        /// <summary>
        /// Every Conversation turn calls this method.
        /// When the user types "run" or "run job", the bot starts a "job".
        /// When the job finishes, the bot proactively notifies the user.
        /// </summary>
        /// <param name="context">The turn context.</param>
        /// <remarks>When our virtual job finishes, it sends a proactive message
        /// to notify the user that the job completed.</remarks>
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type is ActivityTypes.Message)
            {
                var text = context.Activity.AsMessageActivity()?.Text?.Trim().ToLower();
                switch (text)
                {
                    case "run":
                    case "run job":

                        var jobLog = GetJobLog(context);
                        var job = CreateJob(context, jobLog);
                        var appId = GetAppId(context);
                        var conversation = TurnContext.GetConversationReference(context.Activity);

                        await context.SendActivity($"We're starting job {job.JobNumber} for you. We'll notify you when it's complete.");

                        // Since the context is disposed at the end of the turn, extract and send the
                        // information we need to send the proactive message later.
                        var adapter = context.Adapter;
                        Task.Run(() =>
                        {
                            // Simulate a separate process to complete the user's job.
                            Thread.Sleep(5000);

                            // Perform bookkeeping and send the proactive message.
                            CompleteJob(adapter, appId, conversation, job.JobNumber);
                        });

                        break;

                    default:

                        await context.SendActivity("Type 'run' or 'run job' to start a new job.");

                        break;
                }
            }
        }

        /// <summary>
        /// Creates a simulated job and updates the job log.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <param name="jobLog">The job log.</param>
        /// <returns>A new job.</returns>
        private JobData CreateJob(ITurnContext context, Dictionary<int, JobData> jobLog)
        {
            // Generate a non-duplicate job number;
            int number;
            while (jobLog.ContainsKey(number = NumberGenerator.Next())) { }

            // Simulate creaing the job and logging it.
            var job = new JobData
            {
                JobNumber = number,
                Conversation = TurnContext.GetConversationReference(context.Activity)
            };
            jobLog.Add(job.JobNumber, job);

            // Return the created job.
            return job;
        }

        /// <summary>
        /// Performs bookkeeping and proactively notifies the user that their job completed.
        /// </summary>
        /// <param name="adapter">The bot adapter with which to send the message.</param>
        /// <param name="appId">The app ID of the bot to send the message from.</param>
        /// <param name="conversation">The conversation in which to put the message.</param>
        /// <param name="jobNumber">The number of the job that completed.</param>
        private async void CompleteJob(BotAdapter adapter, string appId, ConversationReference conversation, int jobNumber)
        {
            await adapter.ContinueConversation(appId, conversation, async context =>
            {
                // Get the job log from state, and retrieve the job.
                var jobLog = GetJobLog(context);
                var job = jobLog[jobNumber];

                // Perform bookkeeping.
                job.Completed = true;

                // Send the user a proactive confirmation message.
                await context.SendActivity($"Job {job.JobNumber} is complete.");
            });
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

사용자에게 사전 대응 메시지를 보내려면 그 전에 사용자가 봇에게 반응(reactive) 스타일 메시지를 하나 이상 보내야 합니다. 

메시지 하나를 봇에게 보내야 합니다. 봇이 활동 개체에 대한 참조를 가져와서 나중에 사용할 수 있도록 어딘가에 저장해야 하기 때문입니다. 활동 개체는 사용자 주소로 생각할 수 있습니다. 활동 개체에는 사용자가 들어오는 채널, 사용자 ID, 대화 ID 및 향후 메시지를 받아야 하는 서버에 대한 정보가 들어 있기 때문입니다. 이 개체는 간단한 JSON이며 변경하지 말고 전체를 저장해야 합니다.

사용자가 subscribe(구독)라고 말하면 언제든지 대화 참조를 저장하는 방법을 보여주는 간단한 코드 조각부터 시작해 보겠습니다.
```javascript
const { MemoryStorage } = require('botbuilder');

const storage = new MemoryStorage();

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const utterances = (context.activity.text || '').trim().toLowerCase()
            if (utterances === 'subscribe') {
                var userId = await saveReference(TurnContext.getConversationReference(context.activity));
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe' to start proactive message");
            }
    
        }
    });
});
```
위의 코드 조각은 사용자의 `MemoryStorage`로 참조를 저장하고 `userId`를 반환하는 `saveReference()` 함수를 호출합니다. 참조가 성공적으로 저장되고 나면 `subscribeUser()`를 호출하여 구독이 완료되었다고 사용자에게 알려줍니다. 

`subscribeUser()` 함수는 실제 구독을 설정하는 함수입니다. 2초 타이머를 시작하고 타이머가 경과하면 사용자에게 사전 대응 메시지를 보내는 간단한 구현을 살펴보겠습니다.

```javascript
// Persist info to storage
async function saveReference(reference){
    const userId = reference.activityId
    const changes = {};
    changes['reference/' + userId] = reference;
    await storage.write(changes); // Write reference info to persisted storage
    return userId;
}

// Subscribe user to a proactive call. In this case, we are using a setTimeOut() to trigger the proactive call
async function subscribeUser(userId) {
    setTimeout(async () => {
        const reference = await findReference(userId);
        if (reference) {
            await adapter.continueConversation(reference, async (context) => {
                await context.sendActivity("You have been notified");
            });
            
        }
    }, 2000); // Trigger after 2 secs
}

// Read the stored reference info from storage
async function findReference(userId){
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]

    return reference;
}
```

`subscribeUser()` 함수는 저장소에서 이 메시지를 읽어서 참조 개체를 찾는 타이머를 설정합니다. 참조 개체가 발견되면 사용자와 계속 대화할 수 있습니다. `continueConversation` 메서드는 봇이 이미 통신하고 있는 대화 또는 사용자에게 사전에 메시지를 보낼 수 있도록 합니다.

---

## <a name="test-your-bot"></a>봇 테스트

봇을 테스트하려면 Azure에 등록 전용 봇으로 배포하고 웹 채팅에서 테스트하거나 에뮬레이터를 사용하여 로컬에서 테스트하십시오.
