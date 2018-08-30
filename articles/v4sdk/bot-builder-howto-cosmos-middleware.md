---
title: Cosmos DB를 사용하여 미들웨어 만들기 | Microsoft Docs
description: Cosmos DB에 기록하는 사용자 지정 미들웨어를 만드는 방법을 알아봅니다.
keywords: 미들웨어, cosmos db, 데이터베이스, azure, onTurn
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3f36b00a91644859445676676374f7d321087800
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906104"
---
# <a name="create-middleware-that-logs-in-cosmos-db"></a>Cosmos DB에 기록하는 미들웨어 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

SDK에는 유용한 미들웨어가 제공되지만, 원하는 목표를 달성하기 위해 사용자 고유의 미들웨어를 구현해야 하는 경우가 있습니다.

이 샘플에서는 받은 메시지와 다시 보낸 회신을 모두 기록하기 위해 Cosmos DB에 연결되는 미들웨어 계층을 만들어보겠습니다. 여기에 표시되는 코드는 [샘플](../dotnet/bot-builder-dotnet-samples.md)과 함께 제공될 경우 전체 소스 코드로 사용 가능합니다.

## <a name="set-up"></a>설정

코드를 살펴보기 전에 몇 가지 작업을 완료해야 합니다.

### <a name="create-your-database"></a>데이터베이스 만들기

먼저 데이터베이스를 저장할 위치를 준비해야 합니다.  이 작업을 Azure 계정을 통해 수행할 수 이지만, 테스트 목적인 경우 [Cosmos DB 에뮬레이터](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)를 사용할 수 있습니다. 어떤 방법을 선택하든, 데이터베이스에 대한 끝점 URI 및 키가 필요합니다.

이 샘플과 여기에 제공되는 코드는 에뮬레이터를 사용합니다. 끝점 및 키는 Cosmos DB 테스트 계정용으로 제공되는 것으로, 에뮬레이터 설명서에 제공되는 일반적인 값이며, 프로덕션 환경에서는 사용할 수 없습니다.

실제 Azure Cosmos DB를 사용하려는 경우 [Cosmos DB 시작](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-get-started) 항목의 1단계를 수행합니다. 이 작업이 끝나면 끝점 URI와 키 둘 다 데이터베이스 설정의 **키** 탭에서 사용할 수 있습니다. 이러한 값은 아래의 구성 파일에 필요합니다.

### <a name="build-a-basic-bot"></a>기본 봇 빌드

이 항목의 나머지 부분에서는 개요에서 빌드된 HelloBot과 같은 기본 봇을 빌드합니다. [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)를 사용하여 봇에 연결하고, 봇과 대화하고, 봇을 테스트할 수 있습니다.

Bot Framework를 사용하는 표준 봇에 필요한 참조 외에도, 다음 NuGet 패키지에 대한 참조를 추가해야 합니다.

* Microsoft.Azure.Documentdb.Core
* System.Configuration.ConfigurationManager

이러한 라이브러리 각각은 데이터베이스와의 통신과 구성 파일을 사용하도록 설정하는 데 사용됩니다.

### <a name="add-configuration-file"></a>구성 파일 추가

구성 파일을 사용하는 것은 몇 가지 이유로 적절한 방법이므로, 여기에서도 구성 파일을 사용하겠습니다. 제공되는 구성 데이터는 짧고 간단하지만, 봇이 좀 더 복잡해지면 다른 구성 설정을 추가할 수 있습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
1. `app.config`라는 프로젝트에 파일을 추가합니다.
2. 다음 정보를 저장합니다. 끝점 및 키는 로컬 에뮬레이터용으로 정의되어 있지만 Cosmos DB 인스턴스용으로 업데이트할 수 있습니다.
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="EmulatorDbUrl" value="https://localhost:8081"/>
            <add key="EmulatorDbKey" value="C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="/>
        </appSettings>
    </configuration>
    ```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
1. `config.js`라는 프로젝트에 파일을 추가합니다.
2. 다음 정보를 저장합니다. 끝점 및 키는 로컬 에뮬레이터용으로 정의되어 있지만 Cosmos DB 인스턴스용으로 업데이트할 수 있습니다.
    ```javascript
        var config = {}

        config.EmulatorServiceEndpoint = "https://localhost:8081";
        config.EmulatorAuthKey = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWE+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
        config.database = {
            "id": "Tasks"
        };
        config.collection = {
            "id": "Items"
        };

        module.exports = config;
    ```

---

실제 Cosmos DB를 사용하는 경우 위의 두 키 값을 Cosmos DB 설정에 있는 값으로 바꿀 수 있습니다. 또는 다음과 같이 구성에 2개의 키를 더 추가하여 테스트용 에뮬레이터와 실제 Cosmos DB 간을 전환할 수 있습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
```
    <add key="ActualDbUrl" value="<your database URI>"/>
    <add key="ActualDbKey" value="<your database key>"/>
```
 두 개의 키를 더 추가하고 실제 데이터베이스를 사용하려는 경우 아래의 구문에 `EmulatorDbUrl` 및 `EmulatorDbKey` 대신 올바른 키를 지정해야 합니다.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```
    config.ActualServiceEndpoint = "your database URI;
    config.ActualAuthKey = "your database key";
```

두 개의 키를 더 추가하고 실제 데이터베이스를 사용하려는 경우 아래의 구문에 `EmulatorServiceEndpoint` 및 `EmulatorAuthKey` 대신 올바른 키를 지정해야 합니다.

---



## <a name="creating-your-middleware"></a>미들웨어 만들기

# <a name="ctabcs"></a>[C#](#tab/cs)
실제 미들웨어 논리 작성을 시작하기 전에 미들웨어에 대한 봇 프로젝트에서 새 클래스를 만듭니다. 해당 클래스를 만든 후 네임스페이스를 봇의 나머지 부분에서 사용할 `Microsoft.Bot.Samples`로 변경합니다. 다음에서 몇 가지 새 라이브러리에 대한 참조를 추가했다는 사실을 확인할 수 있습니다.

* `Newtonsoft.Json;`
* `Microsoft.Azure.Documents;`
* `Microsoft.Azure.Documents.Client;`
* `Microsoft.Azure.Documents.Linq;`

다양한 `Microsoft.Azure.Documents.*`는 데이터베이스와의 통신을 구성하는 여러 부분에 대한 것이며, `Newtonsoft.Json`은 데이터베이스 간을 이동하는 JSON을 구문 분석하는 데 도움이 됩니다.

마지막으로, 모든 미들웨어는 올바른 작동에 필요한 적절한 처리기를 정의하도록 요구하는 `IMiddleware`를 구현합니다.

```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Configuration;
using Newtonsoft.Json;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using Microsoft.Azure.Documents;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;

namespace Microsoft.Bot.Samples
{
    public class CosmosMiddleware : IMiddleware
    {
        ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
실제 미들웨어 논리를 작성하기 전에 `onTurn()`으로 시작하는 사용자 지정 미들웨어를 만들어야 합니다. 

```javascript
adapter.use({onTurn: async (context, next) =>{
    // Middleware logic here...
}})
    
```

---

#### <a name="defining-local-variables"></a>지역 변수 정의

# <a name="ctabcs"></a>[C#](#tab/cs)
다음으로, 데이터베이스를 조작하기 위한 지역 변수와 로깅하려는 정보를 저장하는 클래스가 필요합니다. `Log`라고 하는 이러한 정보 클래스는 각 멤버와 연결된 JSON 속성에 지정되는 이름을 정의합니다. 이 내용은 나중에 좀 더 살펴보겠습니다.

```cs
    private string Endpoint;
    private string Key;
    private static readonly string Database = "BotData";
    private static readonly string Collection = "BotCollection";
    public DocumentClient docClient;

    public class Log
    {
        [JsonProperty("Time")]
        public string Time;

        [JsonProperty("MessageReceived")]
        public string Message;

        [JsonProperty("ReplySent")]
        public string Reply;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
다음으로, 로깅하려는 정보를 저장하기 위한 지역 변수가 필요합니다. 미들웨어에서는 Cosmos DB를 저장소 공급자로 갖는 conversationState에 액세스해야 합니다. 변수 `info`는 읽고 쓸 상태 개체가 됩니다. 

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(storage);
adapter.use(conversationState);

adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);
    // More middleware logic 
}})
```

--- 


#### <a name="initialize-database-connection"></a>데이터베이스 연결 초기화

# <a name="ctabcs"></a>[C#](#tab/cs)
이 미들웨어의 구문은 위에 정의된 구성 파일에서 필요한 정보를 끌어오고, 데이터베이스에서 읽고 쓸 수 있도록 하는 `DocumentClient`를 만드는 과정을 진행합니다. 여기서는 데이터베이스 및 컬렉션이 설정되어 있는지도 확인할 것입니다.

새로운 `DocumentClient`의 적절한 사용에는 삭제 과정도 필요합니다. 이 작업은 소멸자가 수행합니다.

데이터베이스 만들기는 첫 번째 데이터베이스의 기본 읽기가 시도된 후 해당 데이터베이스 내의 컬렉션에 대한 읽기 시도를 통해 진행됩니다. `NotFound` 예외가 발생할 경우 해당 예외를 catch하고, 스택으로 throw하는 대신, 데이터베이스 또는 컬렉션을 만듭니다. 대부분의 경우에서 이러한 항목이 미리 만들어지지만, 이 샘플의 경우에는 처음 실행 전에 해당 데이터베이스가 만들어지지 않으므로 걱정할 필요가 없습니다. 샘플 코드에서 데이터베이스와 컬렉션은 편의상 두 개의 함수로 분할되어 있지만 아래 코드 조각에서는 결합되어 있습니다.

CosmosDB에 대한 자세한 내용은 해당 [웹 사이트](https://docs.microsoft.com/en-us/azure/cosmos-db/)를 참조하세요. 이 항목의 나머지 부분에서는 Cosmos DB 자체에 대한 세부 정보를 살펴보고 봇이 해당 DB를 사용하는 데 필요한 요건을 집중적으로 알아봅니다.

`Key`, `Endpoint` 및 `docClient`의 정의는 위의 코드 조각에 포함되어 있으며 명확한 이해를 위해 여기에 복사되었습니다.

```cs
    private string Endpoint;
    private string Key;
    // ...
    public DocumentClient docClient;
    // ...

    public CosmosMiddleware()
    {
        Endpoint = ConfigurationManager.AppSettings["EmulatorDbUrl"];
        Key = ConfigurationManager.AppSettings["EmulatorDbKey"];
        docClient = new DocumentClient(new Uri(Endpoint), Key);
        CreateDatabaseAndCollection().ConfigureAwait(false);
    }

    ~CosmosMiddleware()
    {
        docClient.Dispose();
    }

    private async Task CreateDatabaseAndCollection()
    {
        try
        {
            await docClient.ReadDatabaseAsync(UriFactory.CreateDatabaseUri(Database));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDatabaseAsync(new Database { Id = Database });
            }
            else
            {
                throw;
            }
        }

        try
        {
            await docClient.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri
                (Database, Collection));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDocumentCollectionAsync(
                    UriFactory.CreateDatabaseUri(Database),
                    new DocumentCollection { Id = Collection });
            }
            else
            {
                throw;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
데이터베이스 만들기는 첫 번째 데이터베이스의 기본 읽기가 시도된 후 해당 데이터베이스 내의 컬렉션에 대한 읽기 시도를 통해 진행됩니다. `undefined`를 만나면 이를 catch하고 빈 배열로 설정되는 `log`라는 개체를 만듭니다. 대부분의 경우에서 `log`가 미리 만들어지지만, 이 샘플의 경우에는 처음 실행 전에 해당 데이터베이스가 만들어지지 않으므로 걱정할 필요가 없습니다. 샘플 코드에서 `info.log`가 정의되어 있지 않은지 확인합니다. 정의되어 있지 않으면 빈 배열로 설정합니다.

```javascript
adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    // More bot logic below
}})
```
---

#### <a name="read-from-database-logic"></a>데이터베이스 논리에서 읽기

마지막 도우미 함수는 데이터베이스에서 읽고, 지정된 수의 가장 최근 레코드를 반환합니다. 여기에서 사용하는 방법 외에 데이터를 검색하는 더 나은 방법이 있는지 고려해보는 것이 좋으며, 데이터 저장소가 상당히 큰 경우에는 특히 그렇습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task<string> ReadFromDatabase(int numberOfRecords)
    {
        var documents = docClient.CreateDocumentQuery<Log>(
                UriFactory.CreateDocumentCollectionUri(Database, Collection))
                .AsDocumentQuery();
        List<Log> messages = new List<Log>();
        while (documents.HasMoreResults)
        {
            messages.AddRange(await documents.ExecuteNextAsync<Log>());
        }

        // Create a sublist of messages containing the number of requested records.
        List<Log> messageSublist = messages.GetRange(messages.Count - numberOfRecords, numberOfRecords);

        string history = "";

        // Send the last 3 messages.
        foreach (Log logEntry in messageSublist)
        {
            history += ("Message was: " + logEntry.Message + " Reply was: " + logEntry.Reply + "  ");
        }

        return history;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

아래 코드는 여전히 `onTurn()` 함수 내에 있으며, 사용자가 문자열 "history"를 입력하면 호출됩니다.

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through the info.log array
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    }
    //...
}})

```

---

#### <a name="define-onturn"></a>OnTurn() 정의

그런 후에 표준 미들웨어 메서드 `OnTurn()`가 나머지 작업을 처리합니다. 현재 활동이 메시지인 경우에만 로깅하려고 합니다. 메시지인지 여부는 `next()` 호출 전과 후에 확인합니다.

첫 번째로 확인하는 사항은 사용자가 가장 최근 기록을 물어보게 되는 특별한 사례 메시지인지 여부입니다. 이러한 경우 데이터베이스에서 읽기를 호출하여 가장 최근 기록을 가져온 후 대화로 전송합니다. 이를 통해 현재 활동이 완전히 처리되므로, 실행을 전달하는 대신, 파이프라인을 단락합니다.

봇이 전송하는 응답을 가져오기 위해, 메시지를 수신할 때마다 해당 응답을 포착하는 처리기를 만듭니다. 이 내용은 `OnSendActivity()`에 제공하는 람다에서 확인할 수 있습니다. 이 람다는 이 컨텍스트 개체에 대해 `SendActivity()`를 통해 전송되는 모든 메시지를 수집하는 문자열을 빌드합니다.

`next()`에서 파이프라인으로 실행이 반환되면 로그 데이터를 어셈블한 후 데이터베이스에 씁니다. 

이 봇으로 몇 개의 메시지가 전송된 후에 Cosmos DB 데이터 탐색기를 살펴보면 개별 레코드의 데이터를 확인할 수 있습니다. 이러한 레코드 중 하나를 검사하면 처음 3개 항목이 해당 `JsonProperty`에 대해 지정한 문자열로 명명되는 로그 데이터의 3개 값이라는 것을 알 수 있습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task OnTurn
        (ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        string botReply = "";

        if (context.Activity.Type == ActivityTypes.Message)
        {
            if (context.Activity.Text == "history")
            {
                // Read last 3 responses from the database, and short circuit future execution.
                await context.SendActivity(await ReadFromDatabase(3));
                return;
            }

            // Create a send activity handler to grab all response activities 
            // from the activity list.
            context.OnSendActivity(async (activityContext, activityList, activityNext) =>
            {
                foreach (Activity activity in activityList)
                {
                    botReply += (activity.Text + " ");
                }
                await activityNext();
            });
        }

        // Pass execution on to the next layer in the pipeline.
        await next();

        // Save logs for each conversational exchange only.
        if (context.Activity.Type == ActivityTypes.Message)
        {
            // Build a log object to write to the database.
            var logData = new Log
            {
                Time = DateTime.Now.ToString(),
                Message = context.Activity.Text,
                Reply = botReply
            };

            // Write our log to the database.
            try
            {
                var document = await docClient.CreateDocumentAsync(UriFactory.
                    CreateDocumentCollectionUri(Database, Collection), logData);
            }
            catch (Exception ex)
            {
                // More logic for what to do on a failed write can be added here
                throw ex;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

위에 설명된 `onTurn` 함수에서 빌드합니다.

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through info.log 
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    } else if(isMessage){
        // Store the users response
        info.log.push(`Message was: ${context.activity.text}`); 

        await context.onSendActivities(async (handlerContext, activities, handlerNext) => 
        {
            // Store the bot's reply
            info.log.push(`Reply was: ${activities[0].text}`);
            
            await handlerNext(); 
        });
        await next();
    }
}})

```

---

## <a name="sample-output"></a>샘플 출력

전체 샘플에서 봇은 프롬프트 라이브러리를 사용하여 사용자의 이름 및 즐겨 찾는 번호를 묻습니다. 프롬프트 라이브러리에 대한 자세한 내용은 [사용자에게 프롬프트](~/v4sdk/bot-builder-prompts.md) 항목에서 찾을 수 있습니다.

위에 설명된 코드를 실행하는 샘플 봇과의 예제 대화는 다음과 같습니다.

![Cosmos 미들웨어 예제 대화](./media/cosmos-middleware-conversation.png)

## <a name="additional-resources"></a>추가 리소스

* [Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/)
* [Cosmos DB 에뮬레이터](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)

