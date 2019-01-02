---
title: 사용자 데이터 유지 | Microsoft Docs
description: Bot Builder SDK의 저장소에 사용자 상태 데이터를 저장하는 방법을 알아봅니다.
keywords: 사용자 데이터 유지, 저장소, 대화 데이터
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/19/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 36e8efefb276e5b9fb45ba6243b1b472476d5046
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997950"
---
# <a name="persist-user-data"></a>사용자 데이터 유지

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇에서 사용자에게 입력을 요청하는 경우 일부 형태의 저장소에 일부 정보를 유지하려는 경우가 있습니다. Bot Builder SDK를 사용하면 *메모리 내 저장소* 또는 *CosmosDB* 같은 데이터베이스 저장소를 사용하여 사용자 입력을 저장할 수 있습니다. 로컬 저장소 형식은 봇 테스트 또는 프로토타입 제작에 주로 사용됩니다. 그러나 데이터베이스 저장소 같은 영구 저장소 형식은 프로덕션 봇에 가장 적합합니다.

이 토픽에서는 저장소 개체를 정의하고, 사용자 입력이 유지될 수 있도록 저장소 개체에 저장하는 방법을 보여줍니다. 아직 사용자 이름이 없는 경우 대화 상자를 사용하여 사용자에게 이름을 요청할 것입니다. 사용하도록 선택한 저장소 유형에 관계없이 데이터 연결 및 유지를 위한 프로세스는 동일합니다. 이 토픽의 코드는 `CosmosDB`를 저장소로 사용하여 데이터를 유지합니다.

## <a name="prerequisites"></a>필수 조건

사용하려는 개발 환경에 따라 특정 리소스가 필요합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

* [Visual Studio 2015 이상을 설치](https://www.visualstudio.com/downloads/)합니다.
* [BotBuilder V4 템플릿을 설치](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)합니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

* [Visual Studio Code를 설치](https://www.visualstudio.com/downloads/)합니다.
* [Node.js v8.5 이상을 설치](https://nodejs.org/en/)합니다.
* [Yeoman을 설치](http://yeoman.io/)합니다.
* Node.js v4 Botbuilder 템플릿 생성기를 설치합니다.

    ```shell
    npm install generator-botbuilder
    ```

---

이 자습서에서는 다음 패키지를 사용합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

기본 EchoBot 템플릿에서 시작하겠습니다. 지침은 [.NET용 빠른 시작](~/dotnet/bot-builder-dotnet-quickstart.md)을 참조하세요.

NuGet 패킷 관리자에서 다음과 같은 추가 패키지를 설치합니다.

* **Microsoft.Bot.Builder.Azure**
* **Microsoft.Bot.Builder.Dialogs**

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

기본 EchoBot 템플릿에서 시작하겠습니다. 지침은 [JavaScript용 빠른 시작](~/javascript/bot-builder-javascript-quickstart.md)을 참조하세요.

다음과 같은 추가 npm 패키지를 설치합니다.

```cmd
npm install --save botbuilder-dialogs
npm install --save botbuilder-azure
```

---

이 자습서에서 만든 봇을 테스트하려면 [BotFramework Emulator](https://github.com/Microsoft/BotFramework-Emulator)를 설치해야 합니다.

## <a name="create-a-cosmosdb-service-and-update-your-application-settings"></a>CosmosDB 서비스를 만들고 애플리케이션 설정 업데이트

CosmosDB 서비스 및 데이터베이스를 설정하려면 [CosmosDB 사용](bot-builder-howto-v4-storage.md#using-cosmos-db)의 지침을 따릅니다. 단계는 아래에 요약되어 있습니다.

1. 새 브라우저 창에서 <a href="http://portal.azure.com/" target="_blank">Azure Portal</a>에 로그인합니다.
1. **리소스 만들기 > 데이터베이스 > Azure Cosmos DB**를 클릭합니다.
1. **새 계정 페이지**의 **ID** 필드에 고유한 이름을 입력합니다. **API**의 경우 **SQL**을 선택하고 **구독**, **위치** 및 **리소스 그룹** 정보를 제공합니다.
1. **만들기**를 클릭합니다.

그런 다음, 이 봇에 사용할 컬렉션을 해당 서비스에 추가합니다.

컬렉션을 추가하는 데 사용한 데이터베이스 ID 및 컬렉션 ID와 컬렉션 키 설정의 URI 및 기본 키를 기록해 둡니다. 봇을 서비스에 연결하려면 필요합니다.

### <a name="update-your-application-settings"></a>애플리케이션 설정 업데이트

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

CosmosDB에 대한 연결 정보를 포함하도록 **appsettings.json** 파일을 업데이트합니다.

```csharp
{
  // Settings for CosmosDB.
  "CosmosDB": {
    "DatabaseID": "<your-database-identifier>",
    "CollectionID": "<your-collection-identifier>",
    "EndpointUri": "<your-CosmosDB-endpoint>",
    "AuthenticationKey": "<your-primary-key>"
  }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

프로젝트 폴더에서 **.env** 파일을 찾은 다음, Cosmos 관련 데이터를 사용하여 다음 항목을 이 파일에 추가합니다.

**.env**

```text
DB_SERVICE_ENDPOINT=<your-CosmosDB-endpoint>
AUTH_KEY=<authentication key>
DATABASE=<your-primary-key>
COLLECTION=<your-collection-identifier>
```

그런 다음, 봇의 주 **index.js** 파일에서 `MemoryStorage` 대신 `CosmosDbStorage`를 사용하도록 `storage`를 바꿉니다. 런타임 중에 환경 변수를 끌어와서 이러한 필드가 채워집니다.

```javascript
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT,
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
});
```

---

## <a name="create-storage-state-manager-and-state-property-accessor-objects"></a>저장소, 상태 관리자 및 상태 속성 접근자 개체 만들기

봇은 상태 관리 및 저장소 개체를 사용하여 상태를 관리하고 유지합니다. 관리자는 기본 저장소의 유형에 관계없이 상태 속성 접근자를 사용하여 상태 속성에 액세스할 수 있는 추상화 레이어를 제공합니다. 상태 관리자를 사용하여 저장소에 데이터를 씁니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="define-a-class-for-your-user-data"></a>사용자 데이터에 대한 클래스 정의

**CounterState.cs** 파일 이름을 **UserData.cs**로 바꾸고, **CounterState** 클래스 이름을 **UserData**로 바꿉니다.

수집하는 데이터를 보존하도록 이 클래스를 업데이트합니다.

```csharp
/// <summary>
/// Class for storing persistent user data.
/// </summary>
public class UserData
{
    public string Name { get; set; }
}
```

### <a name="define-a-class-for-your-state-and-state-property-accessor-objects"></a>상태 및 상태 속성 접근자 개체에 대한 클래스 정의

**EchoBotAccessors.cs** 파일 이름을 **BotAccessors.cs**로 바꾸고, **EchoBotAccessors** 클래스 이름을 **BotAccessors**로 바꿉니다.

봇에 필요한 상태 개체 및 상태 속성 접근자를 저장하도록 이 클래스를 업데이트합니다.

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using System;

public class BotAccessors
{
    public UserState UserState { get; }

    public ConversationState ConversationState { get; }

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    public IStatePropertyAccessor<UserData> UserDataAccessor { get; set; }

    public BotAccessors(UserState userState, ConversationState conversationState)
    {
        this.UserState = userState
            ?? throw new ArgumentNullException(nameof(userState));

        this.ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }
}
```

### <a name="update-the-startup-code-for-your-bot"></a>봇의 시작 코드 업데이트

**Startup.cs** 파일에서 using 문을 업데이트합니다.

```csharp
using System;
using System.Linq;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Integration;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Configuration;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
```

`ConfigureServices` 메서드에서, 백업 저장소 개체를 만드는 위치부터 시작하여 봇 추가 호출을 업데이트한 다음, 봇 접근자 개체를 등록합니다.

대화 상자 상태를 추적하려면 `DialogState` 개체에 대한 대화 상태가 필요합니다. 대화 상자 상태 속성 접근자 및 봇에서 사용할 대화 상자 집합에 대한 싱글톤을 등록할 것입니다. 봇은 사용자 상태에 대한 고유의 상태 속성 접근자를 만듭니다.

`BotAccessors` 접근자는 봇의 여러 상태 개체에 대한 저장소를 효율적으로 관리하는 방법입니다.

```cs
public void ConfigureServices(IServiceCollection services)
{
    // Register your bot.
    services.AddBot<UserDataBot>(options =>
    {
        // ...

        // Use persistent storage and create state management objects.
        var cosmosSettings = Configuration.GetSection("CosmosDB");
        IStorage storage = new CosmosDbStorage(
            new CosmosDbStorageOptions
            {
                DatabaseId = cosmosSettings["DatabaseID"],
                CollectionId = cosmosSettings["CollectionID"],
                CosmosDBEndpoint = new Uri(cosmosSettings["EndpointUri"]),
                AuthKey = cosmosSettings["AuthenticationKey"],
            });
        options.State.Add(new ConversationState(storage));
        options.State.Add(new UserState(storage));
    });

    // Register the bot's state and state property accessor objects.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var userState = options.State.OfType<UserState>().FirstOrDefault();
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        return new BotAccessors(userState, conversationState)
        {
            UserDataAccessor = userState.CreateProperty<UserData>("UserDataBot.UserData"),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>("UserDataBot.DialogState"),
        };
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="update-your-server-code"></a>서버 코드 업데이트

프로젝트의 **index.js** 파일에서 다음 require 문을 업데이트합니다.

```javascript
// Import required bot services.
const { BotFrameworkAdapter, ConversationState, UserState } = require('botbuilder');
const { CosmosDbStorage } = require('botbuilder-azure');
```

`UserState`를 사용하여 이 자습서의 데이터를 저장할 것입니다. 새 `userState` 개체를 만들고 `MainDialog` 클래스에 두 번째 매개 변수를 전달하도록 이 코드 줄을 업데이트해야 합니다.

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);

// Create the main dialog.
const bot = new MyBot(conversationState, userState);
```

일반 오류가 발생할 경우 대화와 사용자 상태를 모두 지웁니다.

```javascript
// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${error}`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.load(context);
    await conversationState.clear(context);
    await userState.load(context);
    await userState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
    await userState.saveChanges(context);
};
```

그리고 봇 개체를 호출하도록 HTTP 서버 루프를 업데이트합니다.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="update-your-bot-logic"></a>봇 논리 업데이트

`MyBot` 클래스에서, 봇이 작동하는 데 필요한 라이브러리를 요구합니다. 이 자습서에서는 **대화 상자** 라이브러리를 사용하겠습니다.

```javascript
// Required packages for this bot
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, TextPrompt, NumberPrompt } = require('botbuilder-dialogs');

```

두 번째 매개 변수 `userState`를 수락하도록 `MyBot` 클래스의 생성자를 업데이트합니다. 또한 이 자습서에 필요한 상태, 대화 상자 및 프롬프트를 정의하도록 생성자를 업데이트합니다. 이 예에서는 _1단계_에서 사용자에게 이름을 요청하고 _2단계_에서 사용자의 응답을 반환하는 2단계 폭포를 정의합니다. 해당 정보의 유지 여부는 봇의 기본 논리에 달렸습니다.

```javascript
constructor(conversationState, userState) {

    // creates a new state accessor property.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty('dialogState');
    this.userDataAccessor = this.userState.createProperty('userData');

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts
    this.dialogs.add(new TextPrompt('textPrompt'));

    // Add a waterfall dialog to collect and return the user's name.
    this.dialogs.add(new WaterfallDialog('greetings', [
        async function (step) {
            return await step.prompt('textPrompt', "What is your name?");
        },
        async function (step) {
            return await step.endDialog(step.result);
        }
    ]));
}
```

---

사용자 데이터를 저장할 때 선택할 수 있습니다. SDK는 범위를 선택할 수 있는 몇 가지 상태 개체를 제공합니다. 여기서는 대화 상태를 사용하여 대화 상자 상태 개체를 관리하고 사용자 상태를 사용하여 사용자 데이터를 관리할 것입니다.

저장소 및 상태에 대한 일반적인 내용은 [저장소](bot-builder-howto-v4-storage.md) 및 [대화 및 사용자 상태를 관리하는 방법](bot-builder-howto-v4-state.md)을 참조하세요.

## <a name="create-a-greeting-dialog"></a>인사 대화 상자 만들기

대화 상자를 사용하여 사용자 이름을 수집할 수 있습니다. 이 시나리오를 간단하게 유지하기 위해, 대화 상자는 사용자 이름을 반환하고, 봇은 사용자 데이터 개체 및 상태를 관리합니다.

**GreetingsDialog** 클래스를 만들고 다음 코드를 포함합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>Defines a dialog for collecting a user's name.</summary>
public class GreetingsDialog : DialogSet
{
    /// <summary>The ID of the main dialog.</summary>
    public const string MainDialog = "main";

    /// <summary>The ID of the text prompt to use in the dialog.</summary>
    private const string TextPrompt = "textPrompt";

    /// <summary>Creates a new instance of this dialog set.</summary>
    /// <param name="dialogState">The dialog state property accessor to use for dialog state.</param>
    public GreetingsDialog(IStatePropertyAccessor<DialogState> dialogState)
        : base(dialogState)
    {
        // Add the text prompt to the dialog set.
        Add(new TextPrompt(TextPrompt));

        // Define the main dialog and add it to the set.
        Add(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            async (stepContext, cancellationToken) =>
            {
                // Ask the user for their name.
                return await stepContext.PromptAsync(TextPrompt, new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your name?"),
                }, cancellationToken);
            },
            async (stepContext, cancellationToken) =>
            {
                // Assume that they entered their name, and return the value.
                return await stepContext.EndDialogAsync(stepContext.Result, cancellationToken);
            },
        }));
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`MainDialog`의 생성자 내에 대화 상자를 만드는 위의 섹션을 참조하세요.

---

대화 상자에 대한 자세한 내용은 [입력을 요청하는 방법](bot-builder-prompts.md) 및 [단순 대화 흐름을 관리하는 방법](bot-builder-dialog-manage-conversation-flow.md)을 참조하세요.

## <a name="update-your-bot-to-use-the-dialog-and-user-state"></a>대화 상자 및 사용자 상태를 사용하도록 봇 업데이트

봇을 만들고 사용자 입력을 별도로 관리하는 방법을 알아보겠습니다.

### <a name="add-the-dialog-and-a-user-state-accessor"></a>대화 상자 및 사용자 상태 접근자 추가

대화 상자 인스턴스 및 사용자 데이터에 대한 상태 속성 접근자를 추적해야 합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

봇을 초기화하는 코드를 추가합니다.

```cs
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

/// <summary>Defines the bot for the persisting user data tutorial.</summary>
public class UserDataBot : IBot
{
    /// <summary>The bot's state and state property accessor objects.</summary>
    private BotAccessors Accessors { get; }

    /// <summary>The dialog set that has the dialog to use.</summary>
    private GreetingsDialog GreetingsDialog { get; }

    /// <summary>Create a new instance of the bot.</summary>
    /// <param name="options">The options to use for our app.</param>
    /// <param name="greetingsDialog">An instance of the dialog set.</param>
    public UserDataBot(BotAccessors botAccessors)
    {
        // Retrieve the bot's state and accessors.
        Accessors = botAccessors;

        // Create the greetings dialog.
        GreetingsDialog = new GreetingsDialog(Accessors.DialogStateAccessor);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`MainDialog`의 생성자 내에 이러한 상태 접근자가 정의되는 위의 섹션을 참조하세요.

---

### <a name="update-the-turn-handler"></a>순서 처리기 업데이트

순서 처리기는 사용자가 처음으로 대화에 참여하면 사용자에게 인사하고, 사용자가 봇에 메시지를 보내면 사용자에게 대답합니다. 봇은 사용자의 이름을 모르는 경우 인사 대화 상자를 시작하여 사용자의 이름을 묻습니다. 대화 상자가 완료되면 상태 속성 접근자가 생성한 상태 개체를 사용하여 사용자 이름을 사용자 상태에 저장할 것입니다. 순서가 종료되면 접근자 및 상태 관리자는 개체 변경 내용을 저장소에 씁니다.

_사용자 데이터 삭제_ 작업을 위한 지원도 추가될 예정입니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

봇의 `OnTurnAsync` 메서드를 업데이트합니다.

```cs
/// <summary>Handles incoming activities to the bot.</summary>
/// <param name="turnContext">The context object for the current turn.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve user data from state.
    UserData userData = await Accessors.UserDataAccessor.GetAsync(turnContext, () => new UserData());

    // Establish context for our dialog from the turn context.
    DialogContext dc = await GreetingsDialog.CreateContextAsync(turnContext);

    // Handle conversation update, message, and delete user data activities.
    switch (turnContext.Activity.Type)
    {
        case ActivityTypes.ConversationUpdate:

            // Greet any user that is added to the conversation.
            IConversationUpdateActivity activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                if (userData.Name is null)
                {
                    // If we don't already have their name, start a dialog to collect it.
                    await turnContext.SendActivityAsync("Welcome to the User Data bot.");
                    await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
                }
                else
                {
                    // Otherwise, greet them by name.
                    await turnContext.SendActivityAsync($"Hi {userData.Name}! Welcome back to the User Data bot.");
                }
            }

            break;

        case ActivityTypes.Message:

            // If there's a dialog running, continue it.
            if (dc.ActiveDialog != null)
            {
                var dialogTurnResult = await dc.ContinueDialogAsync();
                if (dialogTurnResult.Status == DialogTurnStatus.Complete
                    && dialogTurnResult.Result is string name
                    && !string.IsNullOrWhiteSpace(name))
                {
                    // If it completes successfully and returns a valid name, save the name and greet the user.
                    userData.Name = name;
                    await turnContext.SendActivityAsync($"Pleased to meet you {userData.Name}.");
                }
            }
            else if (userData.Name is null)
            {
                // Else, if we don't have the user's name yet, ask for it.
                await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
            }
            else
            {
                // Else, echo the user's message text.
                await turnContext.SendActivityAsync($"{userData.Name} said, '{turnContext.Activity.Text}'.");
            }

            break;

        case ActivityTypes.DeleteUserData:

            // Delete the user's data.
            userData.Name = null;
            await turnContext.SendActivityAsync("I have deleted your user data.");

            break;
    }

    // Update the user data in the turn's state cache.
    await Accessors.UserDataAccessor.SetAsync(turnContext, userData, cancellationToken);

    // Persist any changes to storage.
    await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇의 `onTurn` 처리기를 업데이트합니다.

```javascript
async onTurn(turnContext) {
    const dc = await this.dialogs.createContext(turnContext); // Create dialog context
    const userData = await this.userDataAccessor.get(turnContext, {});

    switch (turnContext.activity.type) {
        case ActivityTypes.ConversationUpdate:
            if (turnContext.activity.membersAdded[0].name !== 'Bot') {
                if (userData.name) {
                    await turnContext.sendActivity(`Hi ${userData.name}! Welcome back to the User Data bot.`);
                }
                else {
                    // send a "this is what the bot does" message
                    await turnContext.sendActivity('Welcome to the User Data bot.');
                    await dc.beginDialog('greetings');
                }
            }
            break;
        case ActivityTypes.Message:
            // If there is an active dialog running, continue it
            if (dc.activeDialog) {
                var turnResult = await dc.continueDialog();
                if (turnResult.status == "complete" && turnResult.result) {
                    // If it completes successfully and returns a value, save the name and greet the user.
                    userData.name = turnResult.result;
                    await this.userDataAccessor.set(turnContext, userData);
                    await turnContext.sendActivity(`Pleased to meet you ${userData.name}.`);
                }
            }
            // Else, if we don't have the user's name yet, ask for it.
            else if (!userData.name) {
                await dc.beginDialog('greetings');
            }
            // Else, echo the user's message text.
            else {
                await turnContext.sendActivity(`${userData.name} said, ${turnContext.activity.text}.`);
            }
            break;
        case ActivityTypes.DeleteUserData:
            // Delete the user's data.
            // Note: You can use the Emulator to send this activity.
            userData.name = null;
            await this.userDataAccessor.set(turnContext, userData);
            await turnContext.sendActivity("I have deleted your user data.");
            break;
    }

    // Save changes to the conversation and user states.
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
}
```

---

## <a name="start-your-bot-in-visual-studio"></a>Visual Studio에서 봇 시작

애플리케이션을 빌드하고 실행합니다.

## <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결

에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 "시작" 탭에서 **봇 열기** 링크를 클릭합니다.
2. Visual Studio 솔루션을 만든 디렉터리에서 .bot 파일을 선택합니다.

## <a name="interact-with-your-bot"></a>봇과의 상호 작용

봇에 메시지를 보내면 봇이 메시지를 통해 응답합니다.
![에뮬레이터 실행](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [대화 및 사용자 상태 관리](bot-builder-howto-v4-state.md)
