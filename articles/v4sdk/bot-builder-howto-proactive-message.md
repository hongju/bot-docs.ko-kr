---
title: 사전 대응 메시지를 사용하는 방법 | Microsoft Docs
description: 봇을 통해 자동 관리 메시지를 보내는 방법을 알아봅니다.
keywords: 사전 대응 메시지
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 032d7027db3ce83c54bbacf913c2021a22c3f356
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997590"
---
# <a name="how-to-use-proactive-messaging"></a>사전 대응 메시지를 사용하는 방법

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

일반적으로 봇이 사용자에게 직접 보내는 각각의 메시지는 사용자의 이전 입력과 관련이 있습니다.
경우에 따라 봇은 현재 대화 주제 또는 사용자가 보낸 마지막 메시지와 직접 관련이 없는 메시지를 사용자에게 보내야 할 수도 있습니다. 이러한 종류의 메시지를 _자동 관리 메시지_라고 합니다.

## <a name="uses"></a>사용

자동 관리 메시지는 다양한 시나리오에서 유용할 수 있습니다. 봇이 타이머나 미리 알림을 설정할 경우 해당 시간이 되면 사용자에게 알려야 할 수 있습니다. 또는 봇이 외부 시스템에서 알림을 받았을 때 이 정보를 사용자에게 즉시 알려야 할 수 있습니다. 예를 들어, 사용자가 이전에 제품 가격을 모니터링할 것을 봇에게 요청한 경우 봇은 제품 가격이 20%까지 떨어질 경우 사용자에게 경고할 수 있습니다. 또는 봇이 사용자 질문에 대한 응답을 컴파일할 시간이 필요한 경우 지연 사실을 알리고, 당분간 대화를 계속 진행할 수 있습니다. 봇이 질문에 대한 응답 컴파일을 완료하면 해당 정보를 사용자와 공유합니다.

봇에서 자동 관리 메시지를 구현할 때 다음이 적용됩니다.

- 짧은 시간 내에 몇 가지 자동 관리 메시지를 보내지 않습니다. 일부 채널은 봇이 사용자에게 메시지를 보낼 수 있는 빈도를 제한하며, 해당 제한을 위반할 경우 봇을 사용하지 않도록 설정합니다.
- 이전에 봇과 상호 작용하지 않았거나 이메일 또는 SMS와 같은 다른 수단을 통해 봇과의 연락을 요청하지 않은 사용자에게 자동 관리 메시지를 보내지 않습니다.

**임시 자동 관리 메시지**는 가장 간단한 형식의 자동 관리 메시지입니다.
봇은 메시지가 트리거될 때마다 사용자가 현재 봇과 다른 주제의 대화에 참여하고 있으며 대화를 변경하지 않으려고 하는지 여부에 관계없이 메시지를 간단히 대화에 삽입합니다.

알림을 더 원활하게 처리하려면 대화 상태에 플래그를 설정하거나 알림을 큐에 추가하는 것처럼 알림을 대화 흐름에 통합하는 다른 방법을 사용하는 것이 좋습니다.

## <a name="prerequisites"></a>필수 조건

자동 관리 메시지를 보내려면 봇에 유효한 앱 ID와 암호가 있어야 합니다. 그러나 Emulator의 로컬 테스트에서는 자리 표시자 앱 ID를 사용할 수 있습니다.

봇에 사용할 앱 ID와 암호를 가져오려면 [Azure Portal](https://portal.azure.com)에 로그인하고 **봇 채널 등록** 리소스를 만들 수 있습니다. 그런 다음, 테스트를 위해 Azure에 배포할 필요 없이 봇의 앱 ID와 암호를 로컬로 사용할 수 있습니다.

> [!TIP]
> 아직 구독이 없는 경우 <a href="https://azure.microsoft.com/en-us/free/" target="_blank">체험 계정</a>으로 등록할 수 있습니다.

### <a name="required-libraries"></a>필수 라이브러리

BotBuilder 템플릿 중 하나에서 시작하면 필요한 라이브러리가 설치됩니다. 자동 관리 메시지에 필요한 특정 BotBuilder 라이브러리는 다음과 같습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Microsoft.Bot.Builder.Integration.AspNet.Core** NuGet 패키지 (이 패키지를 설치하면 **Microsoft.Bot.Builder** 패키지도 설치됩니다.)

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Microsoft.Bot.Builder** npm 패키지

---

## <a name="notes-on-the-sample-code"></a>샘플 코드에 대한 참고 사항

이 문서의 코드는 [[C#](https://aka.ms/proactive-sample-cs) | [JS](https://aka.ms/proactive-sample-js)] 자동 관리 메시지 샘플에서 가져온 것입니다.

이 샘플은 명확하지 않은 시간이 걸릴 수 있는 사용자 작업을 모델링합니다. 봇은 작업에 대한 정보를 저장하고, 작업이 완료되면 다시 돌아간다고 사용자에게 알리고, 대화를 진행합니다. 작업이 완료되면 봇은 원래 대화에서 확인 메시지를 미리 보냅니다.

## <a name="define-job-data-and-state"></a>작업 데이터 및 상태 정의

이 시나리오에서는 다양한 사용자가 여러 대화에서 만들 수 있는 임의의 작업을 추적합니다. 대화 참조와 작업 식별자를 포함하여 각 작업에 대한 정보를 저장해야 합니다.

- 적절한 대화에 자동 관리 메시지를 보낼 수 있도록 대화 참조가 필요합니다.
- 작업을 확인하는 방법이 필요합니다. 다음 예제에서는 간단한 타임스탬프를 사용합니다.
- 대화 또는 사용자 상태와 관계없이 작업 상태를 저장해야 합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

작업 데이터와 작업 상태에 대한 클래스를 정의해야 합니다. 또한 봇을 등록하고, 작업 로그에 대한 상태 속성 접근자를 설정해야 합니다.

### <a name="define-a-class-for-job-data"></a>작업 데이터에 대한 클래스 정의

**JobLog** 클래스는 작업 번호(타임스탬프)로 인덱싱된 작업 데이터를 추적합니다. 작업 데이터는 사전의 내부 클래스로 정의됩니다.

```csharp
/// <summary>Contains a dictionary of job data, indexed by job number.</summary>
/// <remarks>The JobLog class tracks all the outstanding jobs.  Each job is
/// identified by a unique key.</remarks>
public class JobLog : Dictionary<long, JobLog.JobData>
{
    /// <summary>Describes the state of a job.</summary>
    public class JobData
    {
        /// <summary>Gets or sets the time-stamp for the job.</summary>
        /// <value>
        /// The time-stamp for the job when the job needs to fire.
        /// </value>
        public long TimeStamp { get; set; } = 0;

        /// <summary>Gets or sets a value indicating whether indicates whether the job has completed.</summary>
        /// <value>
        /// A value indicating whether indicates whether the job has completed.
        /// </value>
        public bool Completed { get; set; } = false;

        /// <summary>
        /// Gets or sets the conversation reference to which to send status updates.
        /// </summary>
        /// <value>
        /// The conversation reference to which to send status updates.
        /// </value>
        public ConversationReference Conversation { get; set; }
    }
}
```

### <a name="define-a-state-middleware-class"></a>상태 미들웨어 클래스 정의

**JobState** 클래스는 대화 또는 사용자 상태와 관계없이 작업 상태를 관리합니다.

```csharp
using Microsoft.Bot.Builder;

/// <summary>A <see cref="BotState"/> for managing bot state for "bot jobs".</summary>
/// <remarks>Independent from both <see cref="UserState"/> and <see cref="ConversationState"/> because
/// the process of running the jobs and notifying the user interacts with the
/// bot as a distinct user on a separate conversation.</remarks>
public class JobState : BotState
{
    /// <summary>The key used to cache the state information in the turn context.</summary>
    private const string StorageKey = "ProactiveBot.JobState";

    /// <summary>
    /// Initializes a new instance of the <see cref="JobState"/> class.</summary>
    /// <param name="storage">The storage provider to use.</param>
    public JobState(IStorage storage)
        : base(storage, StorageKey)
    {
    }

    /// <summary>Gets the storage key for caching state information.</summary>
    /// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
    /// for processing this conversation turn.</param>
    /// <returns>The storage key.</returns>
    protected override string GetStorageKey(ITurnContext turnContext) => StorageKey;
}
```

### <a name="register-the-bot-and-required-services"></a>봇 및 필요한 서비스 등록

**Startup.cs** 파일은 봇 및 관련 서비스를 등록합니다.

1. using 문 집합은 다음 네임스페이스를 참조하도록 확장됩니다.

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    ```

1. `ConfigureServices` 메서드는 오류 처리 및 상태 관리를 포함하여 봇을 등록합니다. 또한 봇의 엔드포인트 서비스 및 작업 상태 접근자를 등록합니다.

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // ...

        // Create Job State object.
        // The Job State object is where we persist anything at the job-scope.
        // Note: It's independent of any user or conversation.
        var jobState = new JobState(dataStore);

        // Make it available to our bot
        services.AddSingleton(sp => jobState);

        // Register the proactive bot.
        services.AddBot<ProactiveBot>(options =>
        {
            var secretKey = Configuration.GetSection("botFileSecret")?.Value;
            var botFilePath = Configuration.GetSection("botFilePath")?.Value;

            // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
            var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
            services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

            // Retrieve current endpoint.
            var environment = _isProduction ? "production" : "development";
            var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
            if (!(service is EndpointService endpointService))
            {
                throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
            }

            options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

            // Creates a logger for the application to use.
            ILogger logger = _loggerFactory.CreateLogger<ProactiveBot>();

            // Catches any errors that occur during a conversation turn and logs them.
            options.OnTurnError = async (context, exception) =>
            {
                logger.LogError($"Exception caught : {exception}");
                await context.SendActivityAsync("Sorry, it looks like something went wrong.");
            };

        });

        services.AddSingleton(sp =>
        {
            var config = BotConfiguration.Load(@".\BotConfiguration.bot");
            var endpointService = (EndpointService)config.Services.First(s => s.Type == "endpoint")
                                    ?? throw new InvalidOperationException(".bot file 'endpoint' must be configured prior to running.");

            return endpointService;
        });
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="set-up-the-server-code"></a>서버 코드 설정

**index.js** 파일은 다음을 수행합니다.

- 필요한 패키지 및 서비스 포함
- 봇 클래스 및 **.bot** 파일 참조
- HTTP 서버 만들기
- 봇 어댑터 및 저장소 개체 만들기
- 봇 만들기, 서버 시작 및 봇에 활동 전달

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const restify = require('restify');
const path = require('path');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, BotState, MemoryStorage } = require('botbuilder');
const { BotConfiguration } = require('botframework-config');

const { ProactiveBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file.
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
require('dotenv').config({ path: ENV_FILE });

// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open proactive-messages.bot file in the Emulator.`);
});

// .bot file path
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));

// Read the bot's configuration from a .bot file identified by BOT_FILE.
// This includes information about the bot's endpoints and configuration.
let botConfig;
try {
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.\n\n`);
    process.exit();
}

const DEV_ENVIRONMENT = 'development';

// Define the name of the bot, as specified in .bot file.
// See https://aka.ms/about-bot-file to learn more about .bot files.
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Load the configuration profile specific to this bot identity.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);

// Create the adapter. See https://aka.ms/about-bot-adapter to learn more about using information from
// the .bot file when configuring your adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.MicrosoftAppId,
    appPassword: endpointConfig.appPassword || process.env.MicrosoftAppPassword
});

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create state manager with in-memory storage provider.
const botState = new BotState(memoryStorage, () => 'proactiveBot.botState');

// Create the main dialog, which serves as the bot's main handler.
const bot = new ProactiveBot(botState, adapter);

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        // Route the message to the bot's main handler.
        await bot.onTurn(turnContext);
    });
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
};
```

---

<!--TODO: (Post-Ignite) -- link to a second topic on how to write a job completion DirectLine client that will generate appropriate job completed event activities.-->

## <a name="define-the-bot"></a>봇 정의

사용자는 작업을 만들고 실행하도록 봇에 요청할 수 있습니다. 작업이 완료되면 별도의 작업 서비스에서 봇에 알릴 수 있습니다.

봇은 다음과 같이 설계되었습니다.

- 사용자의 `run` 또는 `run job` 메시지에 대한 응답으로 작업을 만듭니다.
- 사용자의 `show` 또는 `show jobs` 메시지에 대한 응답으로 등록된 모든 작업을 표시합니다.
- 완료된 작업을 식별하는 _작업 완료됨_ 이벤트에 대한 응답으로 작업을 완료합니다.
- `done <jobIdentifier>` 메시지에 대한 응답으로 작업 완료됨 이벤트를 시뮬레이션합니다.
- 작업이 완료되면 원래 대화를 사용하여 사용자에게 자동 관리 메시지를 보냅니다.

봇에 이벤트 활동을 보낼 수 있는 시스템을 구현하는 방법을 보여 주지 않습니다.
<!--TODO: DirectLine--Add back in once the DirectLine topic is added back to the TOC.
See [how to create a Direct Line bot and client](bot-builder-howto-direct-line.md) for information on how to do so.
-->

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

봇에는 다음과 같은 몇 가지 측면이 있습니다.

- 초기화 코드
- 순서 처리기
- 작업 만들기 및 완료 메서드

### <a name="declare-the-class"></a>클래스 선언

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Configuration;
using Microsoft.Bot.Schema;

namespace Microsoft.BotBuilderSamples
{
    /// <summary>
    /// For each interaction from the user, an instance of this class is called.
    /// This is a Transient lifetime service.  Transient lifetime services are created
    /// each time they're requested. For each Activity received, a new instance of this
    /// class is created. Objects that are expensive to construct, or have a lifetime
    /// beyond the single Turn, should be carefully managed.
    /// </summary>
    public class ProactiveBot : IBot
    {
        /// <summary>The name of events that signal that a job has completed.</summary>
        public const string JobCompleteEventName = "jobComplete";

        public const string WelcomeText = "Type 'run' or 'run job' to start a new job.\r\n" +
                                          "Type 'show' or 'show jobs' to display the job log.\r\n" +
                                          "Type 'done <jobNumber>' to complete a job.";
    }
}
```

### <a name="add-initialization-code"></a>초기화 코드 추가

```csharp
private readonly JobState _jobState;
private readonly IStatePropertyAccessor<JobLog> _jobLogPropertyAccessor;

public ProactiveBot(JobState jobState, EndpointService endpointService)
{
    _jobState = jobState ?? throw new ArgumentNullException(nameof(jobState));
    _jobLogPropertyAccessor = _jobState.CreateProperty<JobLog>(nameof(JobLog));

    // Validate AppId.
    // Note: For local testing, .bot AppId is empty for the Bot Framework Emulator.
    AppId = string.IsNullOrWhiteSpace(endpointService.AppId) ? "1" : endpointService.AppId;
}

private string AppId { get; }
```

### <a name="add-a-turn-handler"></a>순서 처리기 추가

모든 봇에서 순서 처리기를 구현해야 합니다. 어댑터는 이 메서드에 활동을 전달합니다.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.Activity.Type != ActivityTypes.Message)
    {
        // Handle non-message activities.
        await OnSystemActivityAsync(turnContext);
    }
    else
    {
        // Get the job log.
        // The job log is a dictionary of all outstanding jobs in the system.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Get the user's text input for the message.
        var text = turnContext.Activity.Text.Trim().ToLowerInvariant();
        switch (text)
        {
            case "run":
            case "run job":

                // Start a virtual job for the user.
                JobLog.JobData job = CreateJob(turnContext, jobLog);

                // Set the new property
                await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

                // Now save it into the JobState
                await _jobState.SaveChangesAsync(turnContext);

                await turnContext.SendActivityAsync(
                    $"We're starting job {job.TimeStamp} for you. We'll notify you when it's complete.");

                break;

            case "show":
            case "show jobs":

                // Display information for all jobs in the log.
                if (jobLog.Count > 0)
                {
                    await turnContext.SendActivityAsync(
                        "| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>" +
                        "| :--- | :---: | :---: |<br>" +
                        string.Join("<br>", jobLog.Values.Select(j =>
                            $"| {j.TimeStamp} &nbsp; | {j.Conversation.Conversation.Id.Split('|')[0]} &nbsp; | {j.Completed} |")));
                }
                else
                {
                    await turnContext.SendActivityAsync("The job log is empty.");
                }

                break;

            default:
                // Check whether this is simulating a job completed event.
                string[] parts = text?.Split(' ', StringSplitOptions.RemoveEmptyEntries);
                if (parts != null && parts.Length == 2
                    && parts[0].Equals("done", StringComparison.InvariantCultureIgnoreCase)
                    && long.TryParse(parts[1], out long jobNumber))
                {
                    if (!jobLog.TryGetValue(jobNumber, out JobLog.JobData jobInfo))
                    {
                        await turnContext.SendActivityAsync($"The log does not contain a job {jobInfo.TimeStamp}.");
                    }
                    else if (jobInfo.Completed)
                    {
                        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is already complete.");
                    }
                    else
                    {
                        await turnContext.SendActivityAsync($"Completing job {jobInfo.TimeStamp}.");

                        // Send the proactive message.
                        await CompleteJobAsync(turnContext.Adapter, AppId, jobInfo);
                    }
                }

                break;
        }

        if (!turnContext.Responded)
        {
            await turnContext.SendActivityAsync(WelcomeText);
        }
    }
}

private static async Task SendWelcomeMessageAsync(ITurnContext turnContext)
{
    foreach (var member in turnContext.Activity.MembersAdded)
    {
        if (member.Id != turnContext.Activity.Recipient.Id)
        {
            await turnContext.SendActivityAsync($"Welcome to SuggestedActionsBot {member.Name}.\r\n{WelcomeText}");
        }
    }
}

// Handles non-message activities.
private async Task OnSystemActivityAsync(ITurnContext turnContext)
{
    // On a job completed event, mark the job as complete and notify the user.
    if (turnContext.Activity.Type is ActivityTypes.Event)
    {
        var jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());
        var activity = turnContext.Activity.AsEventActivity();
        if (activity.Name == JobCompleteEventName
            && activity.Value is long timestamp
            && jobLog.ContainsKey(timestamp)
            && !jobLog[timestamp].Completed)
        {
            await CompleteJobAsync(turnContext.Adapter, AppId, jobLog[timestamp]);
        }
    }
    else if (turnContext.Activity.Type is ActivityTypes.ConversationUpdate)
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            await SendWelcomeMessageAsync(turnContext);
        }
    }
}
```

### <a name="add-job-creation-and-completion-methods"></a>작업 만들기 및 완료 메서드 추가

작업을 시작하기 위해 봇에서 작업을 만들고, 이 작업에 대한 정보와 현재 대화를 작업 로그에 기록합니다. 봇은 모든 대화에서 작업 완료됨 이벤트를 받으면 작업 ID의 유효성을 검사한 후에 코드를 호출하여 작업을 완료합니다.

작업을 완료하는 코드는 상태에서 작업 로그를 가져온 다음, 작업을 완료로 표시하고, 어댑터의 _대화 계속_ 메서드를 사용하여 자동 관리 메시지를 보냅니다.

- 대화 계속 호출은 사용자와는 관계없이 순서를 시작하라는 메시지를 채널에 표시합니다.
- 어댑터는 순서 처리기에서 봇의 정상적인 처리 대신 연결된 콜백을 실행합니다. 이 순서에는 상태 정보를 검색하고 사용자에게 자동 관리 메시지를 보내는 자체의 순서 컨텍스트가 있습니다.

```csharp
// Creates and "starts" a new job.
private JobLog.JobData CreateJob(ITurnContext turnContext, JobLog jobLog)
{
    JobLog.JobData jobInfo = new JobLog.JobData
    {
        TimeStamp = DateTime.Now.ToBinary(),
        Conversation = turnContext.Activity.GetConversationReference(),
    };

    jobLog[jobInfo.TimeStamp] = jobInfo;

    return jobInfo;
}

// Sends a proactive message to the user.
private async Task CompleteJobAsync(
    BotAdapter adapter,
    string botId,
    JobLog.JobData jobInfo,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await adapter.ContinueConversationAsync(botId, jobInfo.Conversation, CreateCallback(jobInfo), cancellationToken);
}

// Creates the turn logic to use for the proactive message.
private BotCallbackHandler CreateCallback(JobLog.JobData jobInfo)
{
    return async (turnContext, token) =>
    {
        // Get the job log from state, and retrieve the job.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Perform bookkeeping.
        jobLog[jobInfo.TimeStamp].Completed = true;

        // Set the new property
        await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

        // Now save it into the JobState
        await _jobState.SaveChangesAsync(turnContext);

        // Send the user a proactive confirmation message.
        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is complete.");
    };
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇은 **bot.js**에 정의되어 있으며, 다음과 같은 몇 가지 측면이 있습니다.

- 초기화 코드
- 순서 처리기
- 작업 만들기 및 완료 메서드

### <a name="declare-the-class-and-add-initialization-code"></a>클래스 선언 및 초기화 코드 추가

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');

const JOBS_LIST = 'jobs';

class ProactiveBot {
    constructor(botState, adapter) {
        this.botState = botState;
        this.adapter = adapter;

        this.jobsList = this.botState.createProperty(JOBS_LIST);
    }

    // ...
};

// Helper function to check if object is empty.
function isEmpty(obj) {
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            return false;
        }
    }
    return true;
};

module.exports.ProactiveBot = ProactiveBot;
```

### <a name="the-turn-handler"></a>순서 처리기

`onTurn` 및 `showJobs` 메서드는 `ProactiveBot` 클래스 내에 정의됩니다. `onTurn`은 사용자의 입력을 처리합니다. 또한 가상 작업 처리 시스템에서 이벤트 활동도 받습니다. `showJobs`는 작업 로그를 형식 지정하고 보냅니다.

```javascript
/**
    *
    * @param {TurnContext} turnContext A TurnContext object representing an incoming message to be handled by the bot.
    */
async onTurn(turnContext) {
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {
        const utterance = (turnContext.activity.text || '').trim().toLowerCase();
        var jobIdNumber;

        // If user types in run, create a new job.
        if (utterance === 'run') {
            await this.createJob(turnContext);
        } else if (utterance === 'show') {
            await this.showJobs(turnContext);
        } else {
            const words = utterance.split(' ');

            // If the user types done and a Job Id Number,
            // we check if the second word input is a number.
            if (words[0] === 'done' && !isNaN(parseInt(words[1]))) {
                jobIdNumber = words[1];
                await this.completeJob(turnContext, jobIdNumber);
            } else if (words[0] === 'done' && (words.length < 2 || isNaN(parseInt(words[1])))) {
                await turnContext.sendActivity('Enter the job ID number after "done".');
            }
        }

        if (!turnContext.responded) {
            await turnContext.sendActivity(`Say "run" to start a job, or "done <job>" to complete one.`);
        }
    } else if (turnContext.activity.type === ActivityTypes.Event && turnContext.activity.name === 'jobCompleted') {
        jobIdNumber = turnContext.activity.value;
        if (!isNaN(parseInt(jobIdNumber))) {
            await this.completeJob(turnContext, jobIdNumber);
        }
    }

    await this.botState.saveChanges(turnContext);
}

// Show a list of the pending jobs
async showJobs(turnContext) {
    const jobs = await this.jobsList.get(turnContext, {});
    if (Object.keys(jobs).length) {
        await turnContext.sendActivity(
            '| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>' +
            '| :--- | :---: | :---: |<br>' +
            Object.keys(jobs).map((key) => {
                return `${ key } &nbsp; | ${ jobs[key].reference.conversation.id.split('|')[0] } &nbsp; | ${ jobs[key].completed }`;
            }).join('<br>'));
    } else {
        await turnContext.sendActivity('The job log is empty.');
    }
}
```

### <a name="logic-to-start-a-job"></a>작업 시작 논리

`createJob` 메서드는 `ProactiveBot` 클래스 내에 정의됩니다. 사용자에 대한 새 작업을 만들고 기록합니다. 이론적으로 이 정보는 작업 처리 시스템에도 전달합니다.

```javascript
// Save job ID and conversation reference.
async createJob(turnContext) {
    // Create a unique job ID.
    var date = new Date();
    var jobIdNumber = date.getTime();

    // Get the conversation reference.
    const reference = TurnContext.getConversationReference(turnContext.activity);

    // Get the list of jobs. Default it to {} if it is empty.
    const jobs = await this.jobsList.get(turnContext, {});

    // Try to find previous information about the saved job.
    const jobInfo = jobs[jobIdNumber];

    try {
        if (isEmpty(jobInfo)) {
            // Job object is empty so we have to create it
            await turnContext.sendActivity(`Need to create new job ID: ${ jobIdNumber }`);

            // Update jobInfo with new info
            jobs[jobIdNumber] = { completed: false, reference: reference };

            try {
                // Save to storage
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job has been processed
                await turnContext.sendActivity('Successful write to log.');
            } catch (err) {
                await turnContext.sendActivity(`Write failed: ${ err.message }`);
            }
        }
    } catch (err) {
        await turnContext.sendActivity(`Read rejected. ${ err.message }`);
    }
}
```

### <a name="logic-to-complete-a-job"></a>작업 완료 논리

`completeJob` 메서드는 `ProactiveBot` 클래스 내에 정의됩니다. 일부 기록을 수행하고, 원래 대화 중이었던 사용자에게 해당 작업이 완료되었다는 자동 관리 메시지를 보냅니다.

```javascript
async completeJob(turnContext, jobIdNumber) {
    // Get the list of jobs from the bot's state property accessor.
    const jobs = await this.jobsList.get(turnContext, {});

    // Find the appropriate job in the list of jobs.
    let jobInfo = jobs[jobIdNumber];

    // If no job was found, notify the user of this error state.
    if (isEmpty(jobInfo)) {
        await turnContext.sendActivity(`Sorry no job with ID ${ jobIdNumber }.`);
    } else {
        // Found a job with the ID passed in.
        const reference = jobInfo.reference;
        const completed = jobInfo.completed;

        // If the job is not yet completed and conversation reference exists,
        // use the adapter to continue the conversation with the job's original creator.
        if (reference && !completed) {
            // Since we are going to proactively send a message to the user who started the job,
            // we need to create the turnContext based on the stored reference value.
            await this.adapter.continueConversation(reference, async (proactiveTurnContext) => {
                // Complete the job.
                jobInfo.completed = true;
                // Save the updated job.
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job is complete.
                await proactiveTurnContext.sendActivity(`Your queued job ${ jobIdNumber } just completed.`);
            });

            // Send a message to the person who completed the job.
            await turnContext.sendActivity('Job completed. Notification sent.');
        } else if (completed) { // The job has already been completed.
            await turnContext.sendActivity('This job is already completed, please start a new job.');
        };
    };
};
```

---

## <a name="test-your-bot"></a>봇 테스트

봇을 로컬로 실행하고, 두 개의 Emulator 창을 엽니다.

1. 대화 ID는 두 창에서 서로 다릅니다.
1. 첫 번째 창에서 `run`을 두 번 입력하여 몇 가지 작업을 시작합니다.
1. 두 번째 창에서 `show`를 입력하여 로그에 있는 작업의 목록을 확인합니다.
1. 두 번째 창에서 `done <jobNumber>`를 입력합니다. 여기서 `<jobNumber>`는 로그의 작업 번호 중 하나이며, 꺾쇠 괄호는 사용하지 않습니다. (봇 코드는 이를 jobComplete 이벤트인 것처럼 해석하도록 설계되었습니다.)
1. 봇은 첫 번째 창에서 사용자에게 자동 관리 메시지를 보냅니다.

<!--TODO: Recreate the screen shots once we're happy with both the C# and JS versions of the code.-->

대화는 사용자의 관점에서 다음과 같을 수 있습니다.

![사용자의 에뮬레이터 세션](~/v4sdk/media/how-to-proactive/user.png)

시뮬레이션된 작업 시스템의 관점에서는 다음과 같습니다.

![작업 시스템의 에뮬레이터 세션](~/v4sdk/media/how-to-proactive/job-system.png)

<!-- Add a next steps section. -->
