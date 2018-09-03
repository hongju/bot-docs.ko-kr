---
title: 대화 및 사용자 상태 관리 | Microsoft Docs
description: .NET용 Bot Builder SDK V4를 사용하여 데이터를 저장 및 검색하는 방법을 알아봅니다.
keywords: 대화 상태, 사용자 상태, 상태 미들웨어, 대화 흐름, 파일 저장소, azure 테이블 저장소
author: ivorb
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a74c52af0ca56b62491ca3aa39d09885c2540c18
ms.sourcegitcommit: ee63d9dc1944a6843368bdabf5878950229f61d0
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/23/2018
ms.locfileid: "42795212"
---
# <a name="manage-conversation-and-user-state"></a>대화 및 사용자 상태 관리

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇이 대화 및 사용자 상태를 저장하게 하려면 상태 관리자 미들웨어를 초기화한 다음, 대화 및 사용자 상태 속성을 사용합니다.
해당 상태가 사용되는 방법에 대한 자세한 내용은 [상태 및 저장소](./bot-builder-storage-concept.md)를 참조하세요.

## <a name="initialize-state-manager-middleware"></a>상태 관리자 미들웨어 초기화

SDK에서 대화 또는 사용자 속성 저장소를 사용하려면 먼저 상태 관리자 미들웨어를 사용하도록 봇 어댑터를 초기화해야 합니다. _대화 상태_는 대화 속성에 사용되고 _사용자 상태_는 사용자 속성에 사용됩니다. (여러 대화에서 사용자 상태 속성에 액세스할 수 있습니다.) 상태 관리자 미들웨어는 기본 저장소의 형식에 상관없이 간단한 키-값 또는 개체 저장소를 사용하여 속성에 액세스할 수 있는 추상화 계층을 제공합니다. 상태 관리자는 기본 저장소 유형이 메모리 내, 파일 저장소 또는 Azure Table Storage인지 여부에 따라 저장소에 데이터 작성 및 동시성 관리를 담당합니다.


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
`ConversationState`가 초기화되는 방식은 Microsoft.Bot.Samples.EchoBot AspNetCore 샘플의 `Startup.cs`를 참조하세요.

이 코드에 필요한 라이브러리:

```csharp
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
```

`ConversationState` 초기화 중:

```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

    IStorage dataStore = new MemoryStorage();
    options.Middleware.Add(new ConversationState<EchoState>(dataStore));
});
```

`options.Middleware.Add(new ConversationState<EchoState>(dataStore));` 줄에서 `ConversationState`는 봇에 미들웨어로 추가되는 대화 상태 관리자 개체입니다. `EchoState` 형식 매개 변수는 대화 상태 정보를 저장하는 방식을 나타내는 형식입니다. 봇은 대화 또는 사용자 상태 데이터에 모든 클래스 형식을 사용할 수 있습니다.

`EchoState`의 구현은 `EchoBot.cs`에 있습니다.
```csharp
public class EchoState
{
    public int TurnNumber { get; set; }
}
``` 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

저장소 공급자를 정의한 후 사용하려는 상태 관리자에 할당합니다.
`ConversationState` 및 `UserState` 관리 미들웨어에 동일한 저장소 공급자를 사용할 수 있습니다.
그 후 `BotStateSet` 라이브러리를 사용하여 자동으로 데이터 유지를 관리할 미들웨어 계층에 연결합니다.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware.
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Alternatively, use both conversation and user state middleware.
// const storage = new MemoryStorage;
// const conversationState = new ConverstationState(storage);
// const userState = new UserState(storage);
// adapter.use(new BotStateSet(conversationState, userState));
```    
---

> [!NOTE] 
> 메모리 내 데이터 저장소는 테스트 전용입니다. 이 저장소는 휘발성이며 임시입니다. 데이터는 봇이 다시 시작될 때마다 지워집니다. 대화 상태 및 사용자 상태에 대한 다른 기본 저장소 미디어를 설정하는 방법은 이 문서 뒷부분의 [파일 저장소](#file-storage) 및 [Azure 테이블 저장소](#azure-table-storage)를 참조하세요. 

### <a name="configuring-state-manager-middleware"></a>상태 관리자 미들웨어 구성

상태 미들웨어를 초기화할 때 선택적 _상태 설정_ 매개 변수 속성을 사용하여 속성이 저장되는 기본 동작을 변경할 수 있습니다. 기본 설정은 다음과 같습니다.

* 턴 컨텍스트의 수명 이후에도 속성 유지.
* 둘 이상의 봇 인스턴스가 속성에 쓰는 경우 봇의 마지막 인스턴스가 이전 인스턴스를 덮어쓰는 것을 허용해야 합니다.

## <a name="use-conversation-and-user-state-properties"></a>대화 및 사용자 상태 속성 사용 
<!-- middleware and message context properties -->

상태 관리자 미들웨어를 구성한 후에는 컨텍스트 개체에서 대화 상태 및 사용자 상태 속성을 가져올 수 있습니다.
<!-- Changes are written to storage before the `SendActivity()` pipeline completes. -->

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Bot Builder SDK에서 `Microsoft.Bot.Samples.EchoBot` 샘플을 사용하여 작동 방식을 볼 수 있습니다. 

`OnTurn` 처리기에서, `context.GetConversationState`는 사용자가 정의한 데이터에 액세스하기 위한 대화 상태를 가져오며, 사용자는 속성을 사용하여 상태를 수정할 수 있습니다(이 예에서는 증분 `TurnNumber`).

```csharp
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
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Yeoman 생성기 샘플에서 생성된 EchoBot을 사용하여 작동 방식을 볼 수 있습니다.

이 코드 샘플은 횟수 카운터를 대화 상태에 저장하는 방법을 보여줍니다.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
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

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="using-conversation-state-to-direct-conversation-flow"></a>대화 상태를 사용하여 대화 흐름 안내

대화 흐름을 설계할 때 대화 흐름을 안내하는 상태 플래그를 정의하면 유용합니다. 이 플래그는 간단한 **부울** 형식일 수도 있고 현재 토픽의 이름을 포함하는 형식일 수도 있습니다. 이 플래그는 대화에서 현재 위치를 추적하는 데 도움이 됩니다. 예를 들어 **부울** 형식 플래그는 사용자가 현재 대화에 있는지 여부를 알려줄 수 있습니다. 반면 토픽 이름 속성은 사용자가 현재 어떤 대화에 있는지 알려줄 수 있습니다.

다음 예에서는 봇이 사용자에게 이름을 요청하면 플래그를 지정하는 부울 _have asked name_ 속성을 사용합니다. 그 다음 메시지를 수신하면 봇이 속성을 검사합니다. 속성이 `true`로 설정된 경우 봇은 사용자에게 이름만 요청했다는 사실을 알고 들어오는 메시지를 이름으로 해석하여 사용자 속성으로 저장합니다.


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
```csharp
public class ConversationInfo
{
    public bool haveAskedNameFlag { get; set; }
    public bool haveAskedNumberFlag { get; set; }
}

public class UserInfo
{
    public string name { get; set; }
    public string telephoneNumber { get; set; }
    public bool done { get; set; }
}

public async Task OnTurn(ITurnContext context)
{
    // Get state objects. Default objects are created if they don't already exist.
    var convo = ConversationState<ConversationInfo>.Get(context);
    var user = UserState<UserInfo>.Get(context);

    if (context.Activity.Type is ActivityTypes.Message)
    {
        if (string.IsNullOrEmpty(user.name) && !convo.haveAskedNameFlag)
        {
            // Ask for the name.
            await context.SendActivity("Hello. What's your name?");

            // Set flag to show we've asked for the name. We save this out so the
            // context object for the next turn of the conversation can check haveAskedName
            convo.haveAskedNameFlag = true;
        }
        else if (!convo.haveAskedNumberFlag)
        {
            // Save the name.
            var name = context.Activity.AsMessageActivity().Text;
            user.name = name;
            convo.haveAskedNameFlag = false; // Reset flag

            // Ask for the phone number. You might want a flag to track this, too.
            await context.SendActivity($"Hello, {name}. What's your telephone number?");
            convo.haveAskedNumberFlag = true;
        }
        else if (convo.haveAskedNumberFlag)
        {
            // save the telephone number
            var telephonenumber = context.Activity.AsMessageActivity().Text;

            user.telephoneNumber = telephonenumber;
            convo.haveAskedNumberFlag = false; // Reset flag
            await context.SendActivity($"Got it. I'll call you later.");
        }
    }
}
```

`UserState<UserInfo>.Get(context)`에서 반환할 수 있도록 사용자 상태를 설정하려면 사용자 상태 미들웨어를 추가합니다. 예를 들어 ASP.NET Core EchoBot의 `Startup.cs`에서 ConfigureServices.cs의 코드를 변경합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        
        IStorage dataStore = new MemoryStorage();
        options.Middleware.Add(new ConversationState<ConversationInfo>(dataStore));
        options.Middleware.Add(new UserState<UserInfo>(dataStore));
    });
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**app.js**

```js
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter (it's ok for MICROSOFT_APP_ID and MICROSOFT_APP_PASSWORD to be blank for now)  
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Storage
const storage = new MemoryStorage();
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        const convo = conversationState.get(context);
        const user = userState.get(context);

        if (isMessage) {
            if(!user.name && !convo.haveAskedNameFlag){
                // Ask for the name.
                await context.sendActivity("What is your name?")
                // Set flag to show we've asked for the name. We save this out so the
                // context object for the next turn of the conversation can check haveAskedNameFlag
                convo.haveAskedNameFlag = true;
            } else if(convo.haveAskedNameFlag){
                // Save the name.
                user.name = context.activity.text;
                convo.haveAskedNameFlag = false; // Reset flag

                await context.sendActivity(`Hello, ${user.name}. What's your telephone number?`);
                convo.haveAskedNumberFlag = true; // Set flag
            } else if(convo.haveAskedNumberFlag){
                // save the phone number
                user.telephonenumber = context.activity.text;
                convo.haveAskedNumberFlag = false; // Reset flag
                await context.sendActivity(`Got it. I'll call you later.`);
            }
        }

        // ...
    });
});

```

---

다른 대안은 _폭포_ 모델 대화 상자를 사용하는 것입니다. 이 대화 상자는 자동으로 대화 상태를 추적하므로 상태를 추적하는 플래그를 만들 필요가 없습니다. 자세한 내용은 [대화 상자로 간단한 대화 관리](bot-builder-dialog-manage-conversation-flow.md)를 참조하세요.

## <a name="file-storage"></a>File Storage

메모리 저장소 공급자는 봇이 다시 시작되면 삭제되는 메모리 내 저장소를 사용합니다. 테스트 목적에만 적합합니다. 데이터를 유지하되, 봇을 데이터베이스에 연결하지 않으려는 경우 파일 저장소 공급자를 사용하면 됩니다. 이 공급자 역시 테스트 용도로 고안되었지만, 상태 데이터를 검사할 수 있도록 파일에 상태 데이터를 유지합니다. 데이터는 JSON 형식을 사용하여 파일에 기록됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Microsoft.Bot.Samples.EchoBot-AspNetCore 샘플에서 `Startup.cs`로 이동하여 `ConfigureServices` 메서드의 코드를 편집합니다.
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // Using file storage instead of in-memory storage.
        IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());
        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
``` 

코드를 실행하고, echobot이 입력을 몇 번 반환하도록 둡니다.

그리고 `System.IO.Path.GetTempPath()`에서 지정한 디렉터리로 이동합니다. 이름이 "conversation"으로 시작하는 파일이 보일 것입니다. 이 파일을 열고 JSON을 살펴보세요. 이 파일에는 다음과 유사한 콘텐츠가 포함됩니다.
```json
{
  "$type": "Microsoft.Bot.Samples.Echo.EchoState, Microsoft.Bot.Samples.EchoBot",
  "TurnNumber": "3",
  "eTag": "ecfe2a23566b4b52b2fe697cffc59385"
}
```

`$type`은 봇에서 대화 상태를 저장하는 데 사용하는 데이터 구조의 형식을 지정합니다. `TurnNumber` 필드는 `EchoState` 클래스의 `TurnNumber` 속성에 해당합니다. `eTag` 필드는 `IStoreItem`에서 상속되며 봇이 대화 상태를 업데이트할 때마다 자동으로 업데이트되는 고유한 값입니다.  eTag 필드는 봇이 낙관적 동시성을 사용할 수 있게 해줍니다.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`FileStorage`를 사용하려면 [대화 및 사용자 상태 속성 사용](#use-conversation-and-user-state-properties) 섹션에서 설명한 에코 봇 샘플을 업데이트합니다. `storage`를 `MemoryStorage` 대신 `FileStorage`로 설정하고 botbuilder로부터 `FileStorage`를 요구합니다. 유일하게 필요한 변경 작업입니다. 

```javascript
// Storage
const storage = new FileStorage("c:/temp");
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));
```

`FileStorage` 공급자는 매개 변수로 "경로"를 사용합니다. 경로를 지정하면 봇의 보관된 정보를 사용하여 파일을 쉽게 찾을 수 있습니다. *대화*마다 새 파일이 만들어집니다. 따라서 *경로*에서 `conversation!`로 시작하는 여러 파일 이름을 찾을 수 있습니다. 날짜순으로 정렬하면 최신 대화를 쉽게 찾을 수 있습니다. 반면, *사용자* 상태에 대한 파일은 하나밖에 없습니다. 파일 이름은 `user!`로 시작합니다. 언제든지 이러한 개체의 상태가 변경되면 상태 관리자는 변경 내용을 반영하도록 파일을 업데이트합니다.

봇을 실행하고 메시지 몇 개를 보냅니다. 그런 다음, 저장소 파일을 찾아서 엽니다. 횟수 카운터를 추적하는 에코 봇의 JSON 콘텐츠는 다음과 비슷합니다.

```json
{
  "turnNumber": "3",
  "eTag": "322"
}
```
---

## <a name="azure-table-storage"></a>Azure Table Storage

저장소 미디어로 Azure 테이블 저장소를 사용할 수도 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Microsoft.Bot.Samples.EchoBot AspNetCore 샘플에서 `Microsoft.Bot.Builder.Azure` NuGet 패키지에 대한 참조를 추가합니다.

그런 다음, `Startup.cs`로 이동하여 `using Microsoft.Bot.Builder.Azure;` 문을 추가하고, `ConfigureServices` 메서드에서 코드를 편집합니다.
```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
    // The parameters are the connection string and table name.
    // "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
    // Replace it with your own connection string if you're not using the emulator
    options.Middleware.Add(new ConversationState<EchoState>(new AzureTableStorage("UseDevelopmentStorage=true","conversationstatetable")));
    // you could also specify the cloud storage account instead of the connection string
    /* options.Middleware.Add(new ConversationState<EchoState>(
        new AzureTableStorage(WindowsAzure.Storage.CloudStorageAccount.DevelopmentStorageAccount, "conversationstatetable"))); */
    options.EnableProactiveMessages = true;
});
```
`UseDevelopmentStorage=true`는 [Azure Storage 에뮬레이터][AzureStorageEmulator]에 사용할 수 있는 연결 문자열입니다. 에뮬레이터를 사용하지 않는 경우 사용자 고유의 연결 문자열로 바꿉니다.

생성자에서 `AzureTableStorage`에 지정한 이름을 가진 테이블이 없으면 이 테이블이 생성됩니다.

<!-- 
TODO: step-by-step inspection of the stored table
-->

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

echobot 샘플의 `app.js`에서, `AzureTableStorage`를 사용하여 ConversationState를 만들 수 있습니다. 

```bash
npm install --save botbuilder-azure@preview
```

```javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState } = require('botbuilder');
const { TableStorage } = require('botbuilder-azure');

// ...

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add conversation state middleware
// The parameters are the connection string and table name.
// "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
// Replace it with your own connection string if you're not using the emulator
var azureStorage = new TableStorage({ tableName: "TestAzureTable1", storageAccountOrConnectionString: "UseDevelopmentStorage=true"})

// You can alternatively use your account name and table name
// var azureStorage = new TableStorage({tableName: "TestAzureTable2", storageAccessKey: "V3ah0go4DLkMtQKUPC6EbgFeXnE6GeA+veCwDNFNcdE6rqSVE/EQO/kjfemJaitPwtAkmR9lMKLtcvgPhzuxZg==", storageAccountOrConnectionString: "storageaccount"});

const conversationState = new ConversationState(azureStorage);
adapter.use(conversationState);

```

`UseDevelopmentStorage=true`는 [Azure Storage 에뮬레이터][AzureStorageEmulator]에 사용할 수 있는 연결 문자열입니다.
생성자에서 `AzureTableStorage`에 지정한 이름을 가진 테이블이 없으면 이 테이블이 생성됩니다.

---

저장된 대화 상태 데이터를 살펴보려면 샘플을 실행한 다음, [Azure Storage 탐색기][AzureStorageExplorer]를 사용하여 테이블을 엽니다.

![Azure Storage 탐색기의 echobot 대화 상태 데이터](media/how-to-state/echostate-azure-storage-explorer.png)


**파티션 키**는 현재 대화에 관련된 고유하게 생성된 키입니다. 봇을 다시 시작하거나 새 대화를 시작하면 새 대화는 고유한 파티션 키가 있는 자체 행을 갖게 됩니다. `EchoState` 데이터 구조는 JSON으로 직렬화되어 Azure Table의 **Json** 열에 저장됩니다. 

```json
{
    "$type":"Microsoft.Bot.Samples.Echo.AspNetCore.EchoState, Microsoft.Bot.Samples.EchoBot-AspNetCore",
    "TurnNumber":2,
    "LastMessage":"second message",
    "eTag":"*"
}
```

`$type` 및 `eTag` 필드는 BotBuilder SDK에 의해 추가됩니다. eTags에 대한 자세한 내용은 [낙관적 동시성 관리](bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)를 참조하세요.


## <a name="next-steps"></a>다음 단계

상태를 사용하여 봇 데이터를 읽고 저장소에 쓰는 방법을 알아보았으니, 데이터를 읽고 저장소에 직접 쓰는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [저장소에 직접 쓰는 방법](bot-builder-howto-v4-storage.md)

## <a name="additional-resources"></a>추가 리소스
저장소에 대한 배경 정보는 [Bot Builder SDK의 저장소](bot-builder-storage-concept.md)를 참조하세요.

<!-- Links -->
[AzureStorageEmulator]: https://docs.microsoft.com/azure/storage/common/storage-use-emulator
[AzureStorageExplorer]: https://azure.microsoft.com/features/storage-explorer/
