---
title: 저장소에 직접 작성 | Microsoft Docs
description: .NET용 Bot Builder SDK를 통해 직접 저장소에 작성하는 방법을 알아봅니다.
keywords: 저장소, 읽기 및 쓰기, 메모리 저장소, eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/14/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9e63390d157c75e1079654549831cf4936c62710
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997990"
---
# <a name="write-directly-to-storage"></a>저장소에 직접 작성

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

미들웨어 또는 컨텍스트 개체를 사용하지 않고 저장소 개체를 직접 읽고 작성할 수 있습니다. 봇의 대화 흐름 외부 소스에서 제공되는 봇에서 사용하는 데이터에 적합할 수 있습니다. 예를 들어 봇을 통해 사용자는 일기 예보를 요청하고, 봇은 외부 데이터베이스에서 읽어 지정된 날짜에 대한 일기 예보를 검색한다고 가정해 봅니다. 날씨 데이터베이스의 콘텐츠는 사용자 정보 또는 대화 컨텍스트에 따라 달라지지 않으므로 상태 관리자를 사용하는 대신 저장소에서 직접 읽을 수 있습니다. 이 문서의 코드 예제에서는 **메모리 저장소**, **Cosmos DB**, **Blob Storage** 및 **Azure Blob Transcript Store**를 사용하여 데이터를 읽고 저장소에 작성하는 방법을 보여줍니다. 

## <a name="prerequisites"></a>필수 조건
- Azure 구독이 아직 없는 경우 시작하기 전에 [체험](https://azure.microsoft.com/en-us/free/) 계정을 만듭니다.
- Bot Framework [Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) 설치

## <a name="memory-storage"></a>메모리 저장소

먼저 데이터를 읽고 메모리 저장소에 작성하는 봇을 만들겠습니다. 메모리 저장소는 테스트 목적으로만 사용되며 프로덕션용이 아닙니다. 봇을 게시하기 전에 저장소를 Cosmos DB 또는 Blob Storage로 설정해야 합니다.

#### <a name="build-a-basic-bot"></a>기본 봇 빌드

이 항목의 나머지에서는 Echo 봇을 빌드합니다. [C#](../dotnet/bot-builder-dotnet-sdk-quickstart.md) 또는 [JS](../javascript/bot-builder-javascript-quickstart.md)에서 만들 수 있습니다. Bot Framework Emulator를 사용하여 봇에 연결하고, 봇과 대화하고, 봇을 테스트할 수 있습니다. 다음 샘플은 사용자의 모든 메시지를 목록에 추가합니다. 목록을 포함하는 데이터 구조가 저장소에 저장됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Create local Memory Storage.
private static readonly MemoryStorage _myStorage = new MemoryStorage();

// Create cancellation token (used by Async Write operation).
public CancellationToken cancellationToken { get; private set; }

// Class for storing a log of utterances (text of messages) as a list.
public class UtteranceLog : IStoreItem
{
     // A list of things that users have said to the bot
     public List<string> UtteranceList { get; } = new List<string>();

     // The number of conversational turns that have occurred        
     public int TurnNumber { get; set; } = 0;

     // Create concurrency control where this is used.
     public string ETag { get; set; } = "*";
}

// Every Conversation turn for our Bot calls this method.
public async Task OnTurnAsync(ITurnContext context)
{
     
     var activityType = context.Activity.Type;
     // See if activity type for this turn is a message from the user.
     if (activityType == ActivityTypes.Message)
     {
         var utterance = context.Activity.Text;
         UtteranceLog logItems = null;
          
         // see if there are previous messages saved in sstorage.
         string[] utteranceList = { "UtteranceLog" };
         logItems = _myStorage.ReadAsync<UtteranceLog>(utteranceList).Result?.FirstOrDefault().Value;

         // If no stored messages were found, create and store a new entry.
         if (logItems is null)
         {
             logItems = new UtteranceLog();
         }
         
         // add new message to list of messages to display.
         logItems.UtteranceList.Add(utterance);
         // increment turn counter.
         logItems.TurnNumber++;
         
         // show user new list of saved messages.
         await context.SendActivityAsync($"The list is now: {string.Join(", ", logItems.UtteranceList)}");
         
         // Create Dictionary object to hold new list of messages.
         {
             changes.Add("UtteranceLog", logItems);
          };
          
          // Save new list to your Storage.
          await _myStorage.WriteAsync(changes,cancellationToken);
     }
     return;
}

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const { BotFrameworkAdapter, ConversationState, BotStateSet, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Create server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add memory storage.
var storage = new MemoryStorage();

const conversationState = new ConversationState(storage);
adapter.use(conversationState);

// Listen for incoming activity .
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing.
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            await logMessageText(storage, context);

            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});

async function logMessageText(storage, context) {
    let utterance = context.activity.text;
    try {
        // Read from the storage.
        let storeItems = await storage.read(["UtteranceLog"])
        // Check the result.
        var utteranceLog = storeItems["UtteranceLog"];

        if (typeof (utteranceLog) != 'undefined') {
            // The log exists so we can write to it.
            storeItems["UtteranceLog"].UtteranceList.push(utterance);

            try {
                await storage.write(storeItems)
                context.sendActivity('Successful write to utterance log.');
            } catch (err) {
                context.sendActivity(`Write failed of UtteranceLog: ${err}`);
            }

         } else {
            context.sendActivity(`need to create new utterance log`);
            storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }

            try {
                await storage.write(storeItems)
                context.sendActivity('Successful write to log.');
            } catch (err) {
                context.sendActivity(`Write failed: ${err}`);
            }
        }
    } catch (err) {
        context.sendActivity(`Read rejected. ${err}`);
    };
}
```

---

### <a name="start-your-bot"></a>봇 시작
봇을 로컬로 실행합니다.

### <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결
에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 "시작" 탭에서 **봇 열기** 링크를 클릭합니다. 
2. 프로젝트를 만든 디렉터리에 있는 .bot 파일을 선택합니다.

### <a name="interact-with-your-bot"></a>봇과의 상호 작용
봇에 메시지를 보내면 봇이 수신한 메시지를 나열합니다.
![에뮬레이터 실행](../media/emulator-v4/emulator-running.png)

 
## <a name="using-cosmos-db"></a>Cosmos DB 사용
이제 메모리 저장소를 사용했으므로 Azure Cosmos DB를 사용하도록 코드를 업데이트합니다. Cosmos DB는 전 세계에 배포된 Microsoft의 멀티모델 데이터베이스입니다. Azure Cosmos DB를 사용하면 Azure의 여러 지리적 영역에서 처리량 및 저장소를 탄력적이고 독립적으로 크기 조정할 수 있습니다. 포괄적인 SLA(서비스 수준 계약)를 통해 처리량, 대기 시간, 가용성 및 일관성을 보장합니다. 

### <a name="set-up"></a>설정
봇에서 Cosmos DB를 사용하려면 코드를 살펴보기 전에 몇 가지 작업을 설정해야 합니다.

#### <a name="create-your-database-account"></a>데이터베이스 계정 만들기
1. 새 브라우저 창에서 [Azure Portal](http://portal.azure.com)에 로그인합니다.
2. **리소스 만들기 > 데이터베이스 > Azure Cosmos DB**를 클릭합니다.
3. **새 계정 페이지**의 **ID** 필드에 고유 이름을 제공합니다. **API**의 경우 **SQL**을 선택하고 **구독**, **위치** 및 **리소스 그룹** 정보를 제공합니다.
4. 그런 다음, **만들기**를 클릭합니다.

계정 생성에는 몇 분 정도가 소요됩니다. 포털에서 축하합니다! Azure Cosmos DB 계정이 만들어졌습니다. 페이지가 표시될 때까지 기다립니다.

##### <a name="add-a-collection"></a>컬렉션 추가
1. **설정 > 새 컬렉션**을 클릭합니다. **컬렉션 추가** 영역이 맨 오른쪽에 표시되면 확인하기 위해 오른쪽으로 스크롤해야 합니다.

![Cosmos DB 컬렉션 추가](./media/add_database_collection.png)

2. "bot-storage"의 컬렉션 ID가 있는 새 데이터베이스 컬렉션, "bot-cosmos-sql-db" 아래에 나와 있는 코딩 예제에서 이러한 값을 사용합니다.

![Cosmos DB](./media/cosmos-db-sql-database.png)

3. 엔드포인트 URI와 키를 데이터베이스 설정의 **키** 탭 내에서 사용할 수 있습니다. 이 문서의 아래쪽에 코드를 더 추가하는 데 이러한 값이 필요합니다. 

![Cosmos DB 키](./media/comos-db-keys.png)

#### <a name="add-configuration-information"></a>구성 정보 추가
Cosmos DB 저장소를 추가하는 구성 데이터는 짧고 간단하며, 봇이 더 복잡해짐에 따라 이러한 동일한 메서드를 사용하여 추가 구성 설정을 추가할 수 있습니다. 이 예제에서는 위의 예제의 Cosmos DB 데이터베이스 및 컬렉션 이름을 사용합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
private const string CosmosDBKey = "<your-cosmos-db-account-key>";
private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
private const string CosmosDBCollectionName = "bot-storage";
```


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`.env` 파일에 다음 정보를 추가합니다. 
 
```javascript
ACTUAL_SERVICE_ENDPOINT=<your database URI>
ACTUAL_AUTH_KEY=<your database key>
DATABASE=Tasks
COLLECTION=Items
```
---

#### <a name="installing-packages"></a>패키지 설치
Cosmos DB에 필요한 패키지가 있는지 확인

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

npm을 통해 프로젝트에서 botbuilder-azure에 참조를 추가할 수 있습니다.


```powershell
npm install --save botbuilder-azure 
```

.env 구성 파일을 사용하려면 봇에 추가 패키지가 포함되어 있어야 합니다. 먼저 npm에서 dotnet 패키지를 가져옵니다.

```powershell
npm install --save dotenv
```

---

### <a name="implementation"></a>구현 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

다음 샘플 코드는 위에 제공된 [메모리 저장소](#memory-storage) 샘플과 동일한 봇 코드를 사용하여 실행됩니다.
아래 코드 조각에서는 로컬 메모리 저장소를 대체하는 '_myStorage_'에 대한 Cosmos DB 저장소의 구현을 보여줍니다. 

```csharp
using Microsoft.Bot.Builder.Azure;

// Create access to Cosmos DB storage.
// Replaces Memory Storage with reference to Cosmos DB.
private static readonly CosmosDbStorage _myStorage = new CosmosDbStorage(new CosmosDbStorageOptions
{
   AuthKey = CosmosDBKey,
   CollectionId = CosmosDBCollectionName,
   CosmosDBEndpoint = new Uri(CosmosServiceEndpoint),
   DatabaseId = CosmosDBDatabaseName,
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

다음 샘플 코드는 [메모리 저장소](#memory-storage)와 비슷하지만 약간의 변경 내용이 있습니다.

botbuilder-azure에서 `CosmosDbStorage`가 필요하며 `.env` 파일을 읽도록 dotenv를 구성합니다.

**app.js**
```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
require('dotenv').config()
```

메모리 저장소를 Cosmos DB에 대한 참조로 바꿉니다.

```javascript
//Add CosmosDB 
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.ACTUAL_SERVICE_ENDPOINT, 
    authKey: process.env.ACTUAL_AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
})

const conversationState = new ConversationState(storage);
adapter.use(conversationState);
```

---

## <a name="start-your-bot"></a>봇 시작
봇을 로컬로 실행합니다.

## <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결
에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 "시작" 탭에서 **봇 열기** 링크를 클릭합니다. 
2. 프로젝트를 만든 디렉터리에 있는 .bot 파일을 선택합니다.

## <a name="interact-with-your-bot"></a>봇과의 상호 작용
봇에 메시지를 보내면 봇이 수신한 메시지를 나열합니다.
![에뮬레이터 실행](../media/emulator-v4/emulator-running.png)


### <a name="view-your-data"></a>데이터 보기
봇을 실행하고 정보를 저장한 후 **데이터 탐색기** 탭 아래의 Azure Portal에서 볼 수 있습니다. 

![데이터 탐색기 예제](./media/data_explorer.PNG)

### <a name="manage-concurrency-using-etags"></a>Etag를 사용하여 동시성 관리
봇 코드 예제에서는 각 `IStoreItem`의 `eTag` 속성을 `*`로 설정합니다. 저장소 개체의 `eTag`(엔터티 태그) 멤버는 Cosmos DB 내에서 동시성을 관리하는 데 사용됩니다. `eTag`는 데이터베이스에 봇이 쓰고 있는 동일한 저장소에서 다른 봇 인스턴스가 개체를 변경할 경우 수행할 작업을 지시합니다. 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>마지막 쓰기 우선 - 덮어쓰기 허용 
별표(`*`)의 `eTag` 속성 값은 마지막 작성자가 우선임을 의미합니다. 새 데이터 저장소를 만들 때 속성의 `eTag`를 `*`로 설정하여 이전에 저장하지 않은 데이터를 쓰고 있거나, 마지막 작성자가 모든 이전에 저장한 속성을 덮어쓰기를 원한다고 표시합니다. 동시성은 봇에는 문제가 되지 않습니다. 쓰는 데이터에 대해 `eTag` 속성을 `*`로 설정하면 덮어쓰기를 활성화합니다.

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>동시성을 유지 관리 및 덮어쓰기 방지
데이터를 Cosmos DB로 저장할 때 속성에 대한 동시 액세스를 방지하려면 `eTag`에 `*` 이외의 값을 사용하여 다른 봇 인스턴스의 변경 내용 덮어쓰기를 차단합니다. 봇이 상태 데이터 저장을 시도하고 `eTag`가 저장소의 `eTag`와 동일한 값이 아니면 `etag conflict key=` 메시지와 함께 오류 응답을 받습니다. <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

기본적으로 Cosmos DB 저장소는 봇이 해당 항목에 쓸 때마다 저장소 개체의 `eTag` 속성이 같은지 확인한 다음, 모든 쓰기 후에 새 고유 값으로 업데이트합니다. 쓰기의 `eTag` 속성이 저장소의 `eTag`와 일치하지 않은 경우 다른 봇 또는 스레드가 해당 데이터를 변경한 것입니다. 

예를 들어, 봇이 저장된 메모를 편집했으나 이 봇이 다른 봇 인스턴스가 수행한 변경 내용을 덮어쓰는 것을 원치 않는다고 가정하겠습니다. 다른 봇 인스턴스가 편집을 수행한 경우 사용자가 최신 업데이트로 버전을 편집하길 바랍니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

먼저 `IStoreItem`를 구현하는 클래스를 만듭니다.

```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

다음으로 저장소 개체를 만들어 최초 메모를 만들고 이 개체를 저장소에 추가합니다.

```csharp
// create a note for the first time, with a non-null, non-* ETag.
var note = new Note { Name = "Shopping List", Contents = "eggs", ETag = "x" };

var changes = Dictionary<string, object>();
{
    changes.Add("Note", note);
};
await NoteStore.WriteAsync(changes, cancellationToken);
```

그런 다음, 나중에 메모를 액세스 및 업데이트하여 저장소에서 읽은 `eTag`를 유지합니다. 

```csharp
var note = NoteStore.ReadAsync<Note>("Note").Result?.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new Dictionary<string, object>();
    {
         changes.Add("Note1", note);
    };
    await NoteStore.WriteAsync(changes, cancellationToken);
}
```

변경 내용을 기록하기 전에 저장소에서 메모가 업데이트된 경우 `Write` 호출에서 예외가 throw됩니다.


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

데이터 저장소에 샘플 메모를 작성하는 도우미 함수를 봇의 끝에 추가합니다.
먼저 `myNoteData` 개체를 만듭니다.

```javascript
// Helper function for writing a sample note to a data store
async function createSampleNote(storage, context) {
    var myNoteData = {
        name: "Shopping List",
        contents: "eggs",
        // If any Note file is already stored, the eTag field
        // must be set to "*" in order to allow writing without first reading the stored eTag
        // otherwise you'll likely get an exception indicating an eTag conflict. 
        eTag: "*"
    }
}
```

`createSampleNote` 도우미 함수 내에서 `changes` 개체를 초기화하고 *메모*를 추가한 다음, 저장소에 기록합니다.

```javascript
// Write the note data to the "Note" key
var changes = {};
changes["Note"] = myNoteData;
// Creates a file named Note, if it doesn't already exist.
// specifying eTag= "*" will overwrite any existing contents.
// The act of writing to the file automatically updates the eTag property
// The first time you write to Note, the eTag is changed from *, and file contents will become:
//    {"name":"Shopping List","contents":"eggs","eTag":"1"}
try {
    await storage.write(changes);
    await context.sendActivity('Successful created a note.');
} catch (err) {
    await context.sendActivity(`Could not create note: ${err}`);
}
```

그런 다음, 나중에 메모에 액세스하고 업데이트하기 위해 사용자가 "메모 업데이트"를 입력할 때 액세스할 수 있는 다른 도우미 함수를 만듭니다.

```javascript
async function updateSampleNote(storage, context) {
    try {
        // Read in a note
        var note = await storage.read(["Note"]);
        console.log(`note.eTag=${note["Note"].eTag}\n note=${JSON.stringify(note)}`);
        // update the note that we just read
        note["Note"].contents += ", bread";
        console.log(`Updated note=${JSON.stringify(note)}`);

        try {
            await storage.write(note); // Write the changes back to storage
            await context.sendActivity('Successfully updated to note.');
        } catch (err) {            console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

변경 내용을 기록하기 전에 저장소에서 메모가 업데이트된 경우 `write` 호출에서 예외가 throw됩니다.

---

동시성을 유지하려면 항상 저장소에서 속성을 읽은 다음, 읽은 속성을 수정하여 `eTag`를 유지 관리합니다. 저장소에서 사용자 데이터를 읽은 경우 응답에 eTag 속성이 포함됩니다. 데이터를 변경하고 업데이트된 데이터를 저장소에 쓴 경우 요청에 앞서 읽은 것과 동일한 값을 지정하는 eTag 속성이 포함됩니다. 그러나 `eTag`가 `*`로 설정된 개체를 쓰면 이 쓰기가 다른 변경 내용을 덮어쓰게 됩니다.

## <a name="using-blob-storage"></a>Blob 저장소 사용 
Azure Blob 저장소는 클라우드를 위한 Microsoft의 개체 저장소 솔루션입니다. Blob 저장소는 텍스트 또는 이진 데이터와 같이 구조화되지 않은 대량의 데이터를 저장하는 데 최적화되어 있습니다.

### <a name="create-your-blob-storage-account"></a>Blob 저장소 계정 만들기
봇에서 Blob 저장소를 사용하려면 코드를 살펴보기 전에 몇 가지 작업을 설정해야 합니다.
1. 새 브라우저 창에서 [Azure Portal](http://portal.azure.com)에 로그인합니다.
2. **리소스 만들기 > Storage > Storage 계정 - Blob, 파일, 테이블, 큐**를 클릭합니다.
3. **새 계정 페이지**에서 저장소 계정에 대한 **이름**을 입력하고, **계정 유형**에 **Blob 저장소**를 선택하고, **위치**, **리소스 그룹** 및 **구독** 정보를 제공합니다.  
4. 그런 다음, **만들기**를 클릭합니다.

![Blob 저장소 만들기](./media/create-blob-storage.png)

#### <a name="add-configuration-information"></a>구성 정보 추가

위에 표시된 대로 봇에 대한 Blob Storage를 구성하는 데 필요한 Blob Storage 키를 찾습니다.
1. Azure Portal에서 Blob Storage 계정을 열고 **설정 > 액세스 키**를 선택합니다.

![Blob Storage 키 찾기](./media/find-blob-storage-keys.png)

이제 이러한 두 개의 키를 사용하여 Blob Storage 계정에 대한 액세스로 코드를 제공합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder.Azure;
```
기존 Blob Storage 계정에 대한 "_myStorage_"를 가리키는 코드 줄을 업데이트합니다.

```csharp


private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
```javascript
const mystorage = new BlobStorage({
   <youy_containerName>,
   <your_storageAccountOrConnectionString>,
   <your_storageAccessKey>
})
```
---

"_myStorage_"가 Blob Storage 계정을 가리키도록 설정되면 봇 코드는 이제 Blob Storage에서 데이터를 저장하고 검색합니다.

## <a name="start-your-bot"></a>봇 시작
봇을 로컬로 실행합니다.

## <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결
에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 "시작" 탭에서 **봇 열기** 링크를 클릭합니다. 
2. 프로젝트를 만든 디렉터리에 있는 .bot 파일을 선택합니다.

## <a name="interact-with-your-bot"></a>봇과의 상호 작용
봇에 메시지를 보내면 봇이 수신한 메시지를 나열합니다.
![에뮬레이터 실행](../media/emulator-v4/emulator-running.png)

### <a name="view-your-data"></a>데이터 보기
봇을 실행하고 정보를 저장한 후 Azure Portal의 **Storage 탐색기** 탭 아래에서 볼 수 있습니다.

## <a name="blob-transcript-storage"></a>Blob 기록 저장소
Azure Blob 기록 저장소는 기록된 기록의 형태로 사용자 대화를 쉽게 저장하고 검색할 수 있는 특수 저장소 옵션을 제공합니다. Azure Blob 기록 저장소는 봇의 성능을 디버깅할 때 검사할 사용자 입력을 자동으로 캡처하는 데 특히 유용합니다.

### <a name="set-up"></a>설정
Azure Blob 기록 저장소는 위의 섹션 "_Blob 저장소 계정 만들기_" 및 "_구성 정보 추가_"에 설명된 단계를 따라 만들어진 동일한 Blob 저장소 계정을 사용할 수 있습니다. 이 문서에서는 새 Blob 컨테이너, "_mybottranscripts_"를 추가했습니다. 

### <a name="implementation"></a>구현 
다음 코드는 기록 저장소 포인터 "_transcriptStore_"를 새 Azure Blob 기록 저장소 계정에 연결합니다. 여기에 표시된 사용자 대화를 저장하는 소스 코드는 [대화 기록](https://aka.ms/bot-history-sample-code) 샘플에 기반을 두고 있습니다. 

```csharp
// In Startup.cs
using Microsoft.Bot.Builder.Azure;

// Enable the conversation transcript middleware.
blobStore = new AzureBlobTranscriptStore(blobStorageConfig.ConnectionString, storageContainer);
var transcriptMiddleware = new TranscriptLoggerMiddleware(blobStore);
options.Middleware.Add(transcriptMiddleware);

// In ConversationHistoryBot.cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;

private readonly AzureBlobTranscriptStore _transcriptStore;

/// <param name="transcriptStore">Injected via ASP.NET dependency injection.</param>
public ConversationHistoryBot(AzureBlobTranscriptStore transcriptStore)
{
    _transcriptStore = transcriptStore ?? throw new ArgumentNullException(nameof(transcriptStore));
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>Azure Blob 기록에 사용자 대화 저장
Blob 컨테이너를 기록을 저장하는 데 사용할 수 있으면 봇과 사용자의 대화를 저장하도록 시작할 수 있습니다. 이러한 대화는 사용자가 봇과 상호 작용하는 방법을 확인하기 위해 나중에 디버깅 도구로 사용될 수 있습니다. 다음 코드는 activity.text가 입력 메시지 _!history_를 검색할 때 사용자 대화 입력을 보존합니다.


```csharp
/// <summary>
/// Every Conversation turn for our EchoBot will call this method. 
/// </summary>
/// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
/// for processing this conversation turn. </param>        
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    var activity = turnContext.Activity;
    if (activity.Type == ActivityTypes.Message)
    {
        if (activity.Text == "!history")
        {
           // Download the activities from the Transcript (blob store) when a request to upload history arrives.
           var connectorClient = turnContext.TurnState.Get<ConnectorClient>(typeof(IConnectorClient).FullName);
           // Get all the message type activities from the Transcript.
           string continuationToken = null;
           var count = 0;
           do
           {
               var pagedTranscript = await _transcriptStore.GetTranscriptActivitiesAsync(activity.ChannelId, activity.Conversation.Id);
               var activities = pagedTranscript.Items
                  .Where(a => a.Type == ActivityTypes.Message)
                  .Select(ia => (Activity)ia)
                  .ToList();
               
               var transcript = new Transcript(activities);

               await connectorClient.Conversations.SendConversationHistoryAsync(activity.Conversation.Id, transcript, cancellationToken: cancellationToken);

               continuationToken = pagedTranscript.ContinuationToken;
           }
           while (continuationToken != null);
```

### <a name="find-all-stored-transcripts-for-your-channel"></a>채널에 대해 저장된 모든 기록 찾기
저장한 데이터를 보려면 다음 코드를 사용하여 프로그래밍 방식으로 저장한 모든 기록에 대한 "_ConversationIDs_"를 찾을 수 있습니다. 봇 에뮬레이터를 사용하여 코드를 테스트할 때 "_Start Over_"를 선택하면 새"_ConversationID_"로 새 기록이 시작됩니다.

```csharp
List<string> storedTranscripts = new List<string>();
PagedResult<Transcript> pagedResult = null;
var pageSize = 0;
do
{
    pagedResult = await _transcriptStore.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
    
    // transcript item contains ChannelId, Created, Id.
    // save the converasationIds (Id) found by "ListTranscriptsAsync" to a local list.
    foreach (var item in pagedResult.Items)
    {
         // Make sure we store an unescaped conversationId string.
         var strConversationId = item.Id;
         storedTranscripts.Add(Uri.UnescapeDataString(strConversationId));
    }
} while (pagedResult.ContinuationToken != null);
```

### <a name="retrieve-user-conversations-from-azure-blob-transcript-storage"></a>Azure Blob 기록 저장소에서 사용자 대화 검색
봇 상호 작용 기록이 Azure Blob 기록 저장소에 저장되면 AzureBlobTranscriptStorage 메서드, "_GetTranscriptActivities_"를 사용하여 테스트 또는 디버깅을 위해 프로그래밍 방식으로 검색할 수 있습니다. 다음 코드 조각은 저장된 각 기록에서 이전 24시간 내에 수신되고 저장된 모든 사용자 입력 기록을 검색합니다.


```csharp
var numTranscripts = storedTranscripts.Count();
for (int i = 0; i < numTranscripts; i++)
{
    PagedResult<IActivity> pagedActivities = null;
    do
    {
        string thisConversationId = storedTranscripts[i];
        // Find all inputs in the last 24 hours.
        DateTime yesterday = DateTime.Now.AddDays(-1);
        // Retrieve iActivities for this transcript.
        pagedActivities = await _myTranscripts.GetTranscriptActivitiesAsync("emulator", thisConversationId, pagedActivities?.ContinuationToken, yesterday);
        foreach (var item in pagedActivities.Items)
        {
            // View as message and find value for key "text" :
            var thisMessage = item.AsMessageActivity();
            var userInput = thisMessage.Text;
         }
    } while (pagedActivities.ContinuationToken != null);
}
```

### <a name="remove-stored-transcripts-from-azure-blob-transcript-storage"></a>Azure Blob 기록 저장소에서 저장된 기록 삭제
봇을 테스트 또는 디버그하기 위해 사용자 입력 데이터 사용을 마치면 저장된 기록은 Azure Blob 기록 저장소에서 프로그래밍 방식으로 제거될 수 있습니다. 다음 코드 조각은 봇 기록 저장소에서 저장된 모든 기록을 제거합니다.


```csharp
for (int i = 0; i < numTranscripts; i++)
{
   // Remove all stored transcripts except the last one found.
   if (i > 0)
   {
       string thisConversationId = storedTranscripts[i];    
       await _transcriptStore.DeleteTranscriptAsync("emulator", thisConversationId);
    }
}
```


다음 링크는 [Azure Blob Transcript Storage](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore)와 관련된 자세한 정보를 제공합니다.  

## <a name="next-steps"></a>다음 단계
이제 저장소에서 직접 읽고 쓰는 방법을 알았으므로 상태 관리자를 사용하여 이를 수행하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [대화 및 사용자 속성을 사용하여 상태 저장](bot-builder-howto-v4-state.md)

