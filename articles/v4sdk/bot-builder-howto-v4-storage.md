---
title: 저장소에 직접 작성 | Microsoft Docs
description: .NET용 Bot Framework SDK를 통해 직접 스토리지에 작성하는 방법을 알아봅니다.
keywords: 저장소, 읽기 및 쓰기, 메모리 저장소, eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 07a66eb468bc456fb463c9c215a2c941e4fafe0a
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215331"
---
# <a name="write-directly-to-storage"></a>저장소에 직접 작성

[!INCLUDE[applies-to](../includes/applies-to.md)]

미들웨어 또는 컨텍스트 개체를 사용하지 않고 저장소 개체를 직접 읽고 작성할 수 있습니다. 이는 봇에서 대화를 보존하는 데 사용하는 데이터 또는 봇의 대화 흐름 외부 소스에서 제공되는 데이터에 적합할 수 있습니다. 이 데이터 스토리지 모델에서는 상태 관리자를 사용하지 않고 스토리지에서 바로 데이터를 읽습니다. 이 문서의 코드 예제에서는 **메모리 스토리지**, **Cosmos DB**, **Blob Storage** 및 **Azure Blob Transcript Store**를 사용하여 데이터를 읽고 스토리지에 작성하는 방법을 보여줍니다. 

## <a name="prerequisites"></a>필수 조건
- Azure 구독이 아직 없는 경우 시작하기 전에 [체험](https://azure.microsoft.com/free/) 계정을 만듭니다.
- 문서 로컬에서 [dotnet](https://aka.ms/bot-framework-www-c-sharp-quickstart) 또는 [nodeJS](https://aka.ms/bot-framework-www-node-js-quickstart)용 봇 만들기 숙지
- [C# 템플릿](https://aka.ms/bot-vsix) 또는 [nodeJS](https://nodejs.org) 및 [yeoman](http://yeoman.io)용 Bot Framework SDK v4 템플릿

## <a name="about-this-sample"></a>이 샘플 정보
이 문서의 샘플 코드는 기본 에코 봇의 구조로 시작한 다음, 아래에 나와 있는 코드를 추가하여 봇의 기능을 확장합니다. 이러한 확장 코드는 수신된 사용자 입력을 저장할 목록을 만듭니다. 매 순서에 사용자 입력의 전체 목록이 사용자에게 출력됩니다. 그런 다음, 이러한 입력 목록이 포함된 데이터 구조가 해당 순서 종료 시 스토리지에 저장됩니다. 이 샘플 코드에 기능을 추가할 때 다양한 스토리지 유형을 살펴봅니다.

## <a name="memory-storage"></a>메모리 저장소

Bot Framework SDK를 사용하면 메모리 내 스토리지를 사용하여 사용자 입력을 저장할 수 있습니다. 메모리 저장소는 테스트 목적으로만 사용되며 프로덕션용이 아닙니다. 데이터베이스 스토리지 같은 영구 스토리지 형식은 프로덕션 봇에 가장 적합합니다. 봇을 게시하기 전에 스토리지를 Cosmos DB 또는 Blob Storage로 설정해야 합니다.

#### <a name="build-a-basic-bot"></a>기본 봇 빌드

이 항목의 나머지에서는 Echo 봇을 빌드합니다. Echo 봇 샘플 코드는 [C# EchoBot](https://aka.ms/bot-framework-www-c-sharp-quickstart) 또는 [JS EchoBot](https://aka.ms/bot-framework-www-node-js-quickstart) 빌드에 대한 빠른 시작 지침에 따라 로컬에서 빌드할 수 있습니다.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Represents a bot saves and echoes back user input.
public class EchoBot : ActivityHandler
{
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
     
   // Echo back user input.
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      // preserve user input.
      var utterance = turnContext.Activity.Text;  
      // make empty local logitems list.
      UtteranceLog logItems = null;
          
      // see if there are previous messages saved in storage.
      try
      {
         string[] utteranceList = { "UtteranceLog" };
         logItems = _myStorage.ReadAsync<UtteranceLog>(utteranceList).Result?.FirstOrDefault().Value;
      }
      catch
      {
         // Inform the user an error occured.
         await turnContext.SendActivityAsync("Sorry, something went wrong reading your stored messages!");
      }
         
      // If no stored messages were found, create and store a new entry.
      if (logItems is null)
      {
         // add the current utterance to a new object.
         logItems = new UtteranceLog();
         logItems.UtteranceList.Add(utterance);
         // set initial turn counter to 1.
         logItems.TurnNumber++;

         // Show user new user message.
         await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");

         // Create Dictionary object to hold received user messages.
         var changes = new Dictionary<string, object>();
         {
            changes.Add("UtteranceLog", logItems);
         }
         try
         {
            // Save the user message to your Storage.
            await _myStorage.WriteAsync(changes, cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
      // Else, our Storage already contained saved user messages, add new one to the list.
      else
      {
         // add new message to list of messages to display.
         logItems.UtteranceList.Add(utterance);
         // increment turn counter.
         logItems.TurnNumber++;
         
         // show user new list of saved messages.
         await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");
         
         // Create Dictionary object to hold new list of messages.
         var changes = new Dictionary<string, object>();
         {
            changes.Add("UtteranceLog", logItems);
         };
         
         try
         {
            // Save new list to your Storage.
            await _myStorage.WriteAsync(changes,cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
      ...  // OnMessageActivityAsync( )
   }
}

```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

.env 구성 파일을 사용하려면 봇에 추가 패키지가 포함되어 있어야 합니다. 아직 설치되지 않은, 경우 npm에서 dotnet 패키지를 받습니다.

```powershell
npm install --save dotenv
```

**bot.js**
```javascript
const { ActivityHandler, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Add memory storage.
var storage = new MemoryStorage();

// Process incoming requests - adds storage for messages.
class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async turnContext => { console.log('this gets called (message)'); 
        await turnContext.sendActivity(`You said '${ turnContext.activity.text }'`); 
        // Save updated utterance inputs.
        await logMessageText(storage, turnContext);
    });
        this.onConversationUpdate(async turnContext => { console.log('this gets called (conversation update)'); 
        await turnContext.sendActivity('[conversationUpdate event detected]'); });
    }
}

// This function stores new user messages. Creates new utterance log if none exists.
async function logMessageText(storage, turnContext) {
    let utterance = turnContext.activity.text;
    // debugger;
    try {
        // Read from the storage.
        let storeItems = await storage.read(["UtteranceLogJS"])
        // Check the result.
        var UtteranceLogJS = storeItems["UtteranceLogJS"];
        if (typeof (UtteranceLogJS) != 'undefined') {
            // The log exists so we can write to it.
            storeItems["UtteranceLogJS"].turnNumber++;
            storeItems["UtteranceLogJS"].UtteranceList.push(utterance);
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                turnContext.sendActivity(`Write failed of UtteranceLogJS: ${err}`);
            }
        }
        else{
            turnContext.sendActivity(`Creating and saving new utterance log`);
            var turnNumber = 1;
            storeItems["UtteranceLogJS"] = { UtteranceList: [`${utterance}`], "eTag": "*", turnNumber }
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                turnContext.sendActivity(`Write failed: ${err}`);
            }
        }
    }
    catch (err){
        turnContext.sendActivity(`Read rejected. ${err}`);
    }
}

module.exports.MyBot = MyBot;

```
---

### <a name="start-your-bot"></a>봇 시작
봇을 로컬로 실행합니다.

### <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결
- Bot Framework [Emulator](https://aka.ms/bot-framework-emulator-readme)를 설치한 다음, 에뮬레이터를 시작한 후 에뮬레이터의 봇에 연결합니다.

1. 에뮬레이터 "시작" 탭에서 **새 봇 구성 만들기** 링크를 클릭합니다. 
2. 봇을 시작할 때 표시되는 웹 페이지의 정보를 고려하여 필드를 채워 봇에 연결합니다.

### <a name="interact-with-your-bot"></a>봇과의 상호 작용
봇에게 메시지를 보냅니다. 봇이 수신된 메시지를 나열합니다.

![에뮬레이터 테스트 스토리지 봇](./media/emulator-direct-storage-test.png)

## <a name="using-cosmos-db"></a>Cosmos DB 사용
이제 메모리 저장소를 사용했으므로 Azure Cosmos DB를 사용하도록 코드를 업데이트합니다. Cosmos DB는 전 세계에 배포된 Microsoft의 멀티모델 데이터베이스입니다. Azure Cosmos DB를 사용하면 Azure의 여러 지리적 영역에서 처리량 및 저장소를 탄력적이고 독립적으로 크기 조정할 수 있습니다. 포괄적인 SLA(서비스 수준 계약)를 통해 처리량, 대기 시간, 가용성 및 일관성을 보장합니다. 

### <a name="set-up"></a>설정
봇에서 Cosmos DB를 사용하려면 코드를 살펴보기 전에 몇 가지 작업을 설정해야 합니다. Cosmos DB 데이터베이스 및 앱 생성에 대한 자세한 설명을 보려면 [Cosmos DB dotnet](https://aka.ms/Bot-framework-create-dotnet-cosmosdb) 또는 [Cosmos DB nodejs](https://aka.ms/Bot-framework-create-nodejs-cosmosdb)에 대한 설명서에 액세스하세요.

### <a name="create-your-database-account"></a>데이터베이스 계정 만들기

1. 새 브라우저 창에서 [Azure Portal](http://portal.azure.com)에 로그인합니다.

![Cosmos DB 데이터베이스 만들기](./media/create-cosmosdb-database.png)

2. **리소스 만들기 > 데이터베이스 > Azure Cosmos DB**를 클릭합니다.

![Cosmos DB 새 계정 페이지](./media/cosmosdb-new-account-page.png)

3. **새 계정 페이지**에서 **구독**, **리소스 그룹** 정보를 제공합니다. **계정 이름** 필드의 고유한 이름을 만듭니다. 이는 최종적으로 데이터 액세스 URL 이름에 포함됩니다. **API**에 **Core(SQL)** 를 선택하고, 가까운 **위치**를 지정하여 데이터 액세스 시간을 단축합니다.
4. 그런 다음, **검토 + 만들기**를 클릭합니다.
5. 유효성 검사에 성공하면 **만들기**를 클릭합니다.

계정 생성에는 몇 분 정도가 소요됩니다. 포털에서 축하합니다! Azure Cosmos DB 계정이 만들어졌습니다. 페이지가 표시될 때까지 기다립니다.

### <a name="add-a-collection"></a>컬렉션 추가

![Cosmos DB 컬렉션 추가](./media/add_database_collection.png)

1. **설정 > 새 컬렉션**을 클릭합니다. **컬렉션 추가** 영역이 맨 오른쪽에 표시되면 확인하기 위해 오른쪽으로 스크롤해야 합니다. 최근에 Cosmos DB가 업데이트되어 단일 파티션 키 _/id_를 추가해야 합니다. 이 키는 파티션 간 쿼리 오류를 방지합니다.

![Cosmos DB](./media/cosmos-db-sql-database.png)

2. "bot-storage"의 컬렉션 ID가 있는 새 데이터베이스 컬렉션, "bot-cosmos-sql-db" 아래에 나와 있는 코딩 예제에서 이러한 값을 사용합니다.

![Cosmos DB 키](./media/comos-db-keys.png)

3. 엔드포인트 URI와 키를 데이터베이스 설정의 **키** 탭 내에서 사용할 수 있습니다. 이 문서의 아래쪽에 코드를 더 추가하는 데 이러한 값이 필요합니다. 

### <a name="add-configuration-information"></a>구성 정보 추가
Cosmos DB 저장소를 추가하는 구성 데이터는 짧고 간단하며, 봇이 더 복잡해짐에 따라 이러한 동일한 메서드를 사용하여 추가 구성 설정을 추가할 수 있습니다. 이 예제에서는 위의 예제의 Cosmos DB 데이터베이스 및 컬렉션 이름을 사용합니다.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
public class EchoBot : ActivityHandler
{
   private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
   private const string CosmosDBKey = "<your-cosmos-db-account-key>";
   private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
   private const string CosmosDBCollectionName = "bot-storage";
   ...
   
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`.env` 파일에 다음 정보를 추가합니다.

**.env**
```javascript
DB_SERVICE_ENDPOINT="<your-Cosmos-db-URI>"
AUTH_KEY="<your-cosmos-db-account-key>"
DATABASE="<bot-cosmos-sql-db>"
COLLECTION="<bot-storage>"
```
---

#### <a name="installing-packages"></a>패키지 설치
Cosmos DB에 필요한 패키지가 있는지 확인

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

npm을 통해 프로젝트에서 botbuilder-azure에 참조를 추가할 수 있습니다.
>**참고** - 이 npm 패키지는 개발 머신에 설치된 Python에 의존합니다. 이전에 Python을 설치한 적이 없다면 여기에서 머신의 설치 리소스를 확인할 수 있습니다. [Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure 
```

`.env` 파일 설정에 액세스하려면 npm에서 dotnet 패키지를 받으세요(아직 설치되지 않은 경우).

```powershell
npm install --save dotenv
```
---

### <a name="implementation"></a>구현 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

다음 샘플 코드는 위에 제공된 [메모리 저장소](#memory-storage) 샘플과 동일한 봇 코드를 사용하여 실행됩니다.
아래 코드 조각에서는 로컬 메모리 스토리지를 대체하는 '_myStorage_'에 대한 Cosmos DB 스토리지의 구현을 보여줍니다. 메모리 스토리지가 주석 처리되고 Cosmos DB에 대한 참조로 바뀝니다.

**EchoBot.cs**
```csharp

using System;
...
using Microsoft.Bot.Builder.Azure;
...
public class EchoBot : ActivityHandler
{
   // Create local Memory Storage - commented out.
   // private static readonly MemoryStorage _myStorage = new MemoryStorage();

   // Replaces Memory Storage with reference to Cosmos DB.
   private static readonly CosmosDbStorage _myStorage = new CosmosDbStorage(new CosmosDbStorageOptions
   {
      AuthKey = CosmosDBKey,
      CollectionId = CosmosDBCollectionName,
      CosmosDBEndpoint = new Uri(CosmosServiceEndpoint),
      DatabaseId = CosmosDBDatabaseName,
   });
   
   ...
}

```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

다음 샘플 코드는 [메모리 저장소](#memory-storage)와 비슷하지만 약간의 변경 내용이 있습니다.

`CosmosDbStorage`에서 `botbuilder-azure`가 필요하며 `.env` 파일을 읽도록 dotenv를 구성합니다.

**bot.js**

```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
```
메모리 스토리지를 Cosmos DB에 대한 참조로 바꾸라는 주석을 추가합니다.

**bot.js**
```javascript
// initialized to access values in .env file.
var dotenv = require('dotenv');
dotenv.load();

// Create local Memory Storage - commented out.
// var storage = new MemoryStorage();

// Create access to CosmosDb Storage - this replaces local Memory Storage.
var storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT, 
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
})

```
---

## <a name="start-your-bot"></a>봇 시작
봇을 로컬로 실행합니다.

## <a name="test-your-bot-with-bot-framework-emulator"></a>Bot Framework Emulator를 사용하여 봇 테스트
이제 Bot Framework Emulator를 시작하고 봇에 연결합니다.

1. 에뮬레이터 "시작" 탭에서 **새 봇 구성 만들기** 링크를 클릭합니다. 
2. 봇을 시작할 때 표시되는 웹 페이지의 정보를 고려하여 필드를 채워 봇에 연결합니다.

## <a name="interact-with-your-bot"></a>봇과의 상호 작용
봇에 메시지를 보내면 봇이 수신한 메시지를 나열합니다.
![에뮬레이터 실행](./media/emulator-direct-storage-test.png)


### <a name="view-your-data"></a>데이터 보기
봇을 실행하고 정보를 저장한 후 **데이터 탐색기** 탭 아래의 Azure Portal에서 저장된 데이터를 볼 수 있습니다. 

![데이터 탐색기 예제](./media/data_explorer.PNG)


## <a name="using-blob-storage"></a>Blob Storage 사용 
Azure Blob Storage는 클라우드를 위한 Microsoft의 개체 스토리지 솔루션입니다. Blob Storage는 텍스트 또는 이진 데이터와 같이 구조화되지 않은 대량의 데이터를 저장하는 데 최적화되어 있습니다.

### <a name="create-your-blob-storage-account"></a>Blob Storage 계정 만들기
봇에서 Blob Storage를 사용하려면 코드를 살펴보기 전에 몇 가지 작업을 설정해야 합니다.
1. 새 브라우저 창에서 [Azure Portal](http://portal.azure.com)에 로그인합니다.

![Blob Storage 만들기](./media/create-blob-storage.png)

2. **리소스 만들기 > Storage > Storage 계정 - Blob, 파일, 테이블, 큐**를 클릭합니다.

![Blob Storage 새 계정 페이지](./media/blob-storage-new-account.png)

3. **새 계정 페이지**에서 스토리지 계정에 대한 **이름**을 입력하고, **계정 유형**에 **Blob Storage**를 선택하고, **위치**, **리소스 그룹** 및 **구독** 정보를 제공합니다.  
4. 그런 다음, **검토 + 만들기**를 클릭합니다.
5. 유효성 검사에 성공하면 **만들기**를 클릭합니다.

### <a name="create-blob-storage-container"></a>Blob Storage 컨테이너 만들기
Blob Storage 계정이 생성되면 다음과 같은 방법으로 이 계정을 엽니다. 
1. 리소스 선택
2. 이제 Storage Explorer(미리 보기)를 사용하여 "열기"

![Blob Storage 컨테이너 만들기](./media/create-blob-container.png)

3. 마우스 오른쪽 단추로 Blob 컨테이너를 클릭하고 _Blob 컨테이너 만들기_를 선택합니다.
4. 이름을 추가합니다. "your-blob-storage-container-name" 값에 이 이름을 사용하여 Blob Storage 계정에 대한 액세스를 제공합니다.

#### <a name="add-configuration-information"></a>구성 정보 추가
위에 표시된 대로 봇에 대한 Blob Storage를 구성하는 데 필요한 Blob Storage 키를 찾습니다.
1. Azure Portal에서 Blob Storage 계정을 열고 **설정 > 액세스 키**를 선택합니다.

![Blob Storage 키 찾기](./media/find-blob-storage-keys.png)

"your-blob-storage-account-string" 값에 key1 _연결 문자열_을 사용하여 Blob Storage 계정에 대한 액세스를 제공합니다.

#### <a name="installing-packages"></a>패키지 설치
이전에 Cosmos DB를 사용하도록 설치되지 않은 경우 다음 패키지를 설치합니다.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

npm을 통해 프로젝트에서 botbuilder-azure에 참조를 추가합니다.
>**참고** - 이 npm 패키지는 개발 머신에 설치된 Python에 의존합니다. 이전에 Python을 설치한 적이 없다면 여기에서 머신의 설치 리소스를 확인할 수 있습니다. [Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure 
```

`.env` 파일 설정에 액세스하려면 npm에서 dotnet 패키지를 받으세요(아직 설치되지 않은 경우).

```powershell
npm install --save dotenv
```
---

### <a name="implementation"></a>구현 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;
```
기존 Blob Storage 계정에 대한 "_myStorage_"를 가리키는 코드 줄을 업데이트합니다.

**EchoBot.cs**
```csharp
private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`.env` 파일에 다음 정보를 추가합니다.

**.env**
```javascript
BLOB_NAME="<your-blob-storage-container-name>"
BLOB_STRING="<your-blob-storage-account-string>"
```

`bot.js` 파일을 다음과 같이 업데이트합니다. `botbuilder-azure`에서 `BlobStorage` 요구

**bot.js**
```javascript
const { BlobStorage } = require("botbuilder-azure");
```

`.env` 파일을 로드하여 Cosmos DB 스토리지를 구현하는 코드를 추가하지 않은 경우 여기에서 추가하세요.

```javascript
// initialized to access values in .env file.
var dotenv = require('dotenv');
dotenv.load();
```
이제 이전의 스토리지 정의를 주석 처리하고 다음을 추가하여 "_storage_"가 기존의 Blob Storage 계정을 가리키도록 코드를 업데이트합니다.

**bot.js**
```javascript
var storage = new BlobStorage({
    containerName: process.env.BLOB_NAME,
    storageAccountOrConnectionString: process.env.BLOB_STRING
});
```
---

스토리지가 Blob Storage 계정을 가리키도록 설정되면 봇 코드는 이제 Blob Storage에서 데이터를 저장하고 검색합니다.

## <a name="start-your-bot"></a>봇 시작
봇을 로컬로 실행합니다.

## <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결
에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 "시작" 탭에서 **새 봇 구성 만들기** 링크를 클릭합니다. 
2. 봇을 시작할 때 표시되는 웹 페이지의 정보를 고려하여 필드를 채워 봇에 연결합니다.

## <a name="interact-with-your-bot"></a>봇과의 상호 작용
봇에 메시지를 보내면 봇이 수신한 메시지를 나열합니다.

![에뮬레이터 테스트 스토리지 봇](./media/emulator-direct-storage-test.png)

### <a name="view-your-data"></a>데이터 보기
봇을 실행하고 정보를 저장한 후 Azure Portal의 **Storage Explorer** 탭 아래에서 볼 수 있습니다.

## <a name="blob-transcript-storage"></a>Blob 기록 저장소
Azure Blob 기록 저장소는 기록된 기록의 형태로 사용자 대화를 쉽게 저장하고 검색할 수 있는 특수 저장소 옵션을 제공합니다. Azure Blob 음성 텍스트 스토리지는 봇의 성능을 디버깅할 때 검사할 사용자 입력을 자동으로 캡처하는 데 특히 유용합니다.

### <a name="set-up"></a>설정
Azure Blob 기록 스토리지는 위의 섹션 &quot;_Blob Storage 계정 만들기_&quot; 및 &quot;_구성 정보 추가_&quot;에 설명된 단계를 따라 만들어진 동일한 Blob Storage 계정을 사용할 수 있습니다. 이제 음성 텍스트를 저장할 컨테이너를 추가합니다.

![음성 텍스트 컨테이너 만들기](./media/create-blob-transcript-container.png)

1. Azure Blob Storage 계정을 엽니다.
1. _Storage Explorer_를 클릭합니다.
1. _Blob 컨테이너_를 마우스 오른쪽 단추로 클릭하고 _Blob 컨테이너 만들기_를 선택합니다.
1. 음성 텍스트 컨테이너의 이름을 입력한 후 _확인_을 선택합니다. (여기서는 mybottranscripts 입력)

### <a name="implementation"></a>구현 
다음 코드는 기록 스토리지 포인터 `_myTranscripts`를 새 Azure Blob 기록 스토리지 계정에 연결합니다. 새 컨테이너 이름 <your-blob-transcript-container-name>을 사용하여 이 링크를 만들려면 Blob Storage 내에 음성 텍스트 파일을 저장할 새 컨테이너를 만듭니다.

**echoBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;

public class EchoBot : ActivityHandler
{
   ...
   
   private readonly AzureBlobTranscriptStore _myTranscripts = new AzureBlobTranscriptStore("<your-blob-transcript-storage-account-string>", "<your-blob-transcript-container-name>");
   
   ...
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>Azure Blob 기록에 사용자 대화 저장
Blob 컨테이너를 기록을 저장하는 데 사용할 수 있으면 봇과 사용자의 대화를 저장하도록 시작할 수 있습니다. 이러한 대화는 사용자가 봇과 상호 작용하는 방법을 확인하기 위해 나중에 디버깅 도구로 사용될 수 있습니다. 각 에뮬레이터 _대화 다시 시작_은 새로운 음성 텍스트 대화 목록의 생성을 시작합니다. 다음 코드는 사용자 대화 입력을 저장된 음성 텍스트 파일 내에 저장합니다.
- 현재 음성 텍스트는 `LogActivityAsync`를 사용하여 저장됩니다.
- 저장된 음성 텍스트는 `ListTranscriptsAsync`를 사용하여 검색됩니다.
이 샘플 코드에서는 저장된 각 음성 텍스트의 ID가 "storedTranscripts"라는 목록에 저장됩니다. 이 목록은 나중에 저장된 Blob 음성 텍스트 수를 관리하는 데 사용됩니다.

**echoBot.cs**
```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);
    
    ...
}

```

### <a name="manage-stored-blob-transcripts"></a>저장된 Blob 음성 텍스트 관리
저장된 음성 텍스트는 디버깅 도구로 사용할 수 있지만 시간이 지나면 저장된 음성 텍스트 수가 감당할 수 없을 정도로 커질 수 있습니다. 아래에 포함된 추가 코드는 `DeleteTranscriptAsync`를 사용하여 Blob 음성 텍스트 저장소에서 검색된 마지막 3개 음성 텍스트 항목을 제외한 모든 항목을 제거합니다.

**echoBot.cs**
```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);
    
    // Manage the size of your transcript storage.
    for (int i = 0; i < pageSize; i++)
    {
       // Remove older stored transcripts, save just the last three.
       if (i < pageSize - 3)
       {
          string thisTranscriptId = storedTranscripts[i];
          try
          {
             await _myTranscripts.DeleteTranscriptAsync("emulator", thisTranscriptId);
           }
           catch (System.Exception ex)
           {
              await turnContext.SendActivityAsync("Debug Out: DeleteTranscriptAsync had a problem!");
              await turnContext.SendActivityAsync("exception: " + ex.Message);
           }
       }
    }
    ...
}

```

다음 링크는 [Azure Blob Transcript Storage](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore)와 관련된 자세한 정보를 제공합니다. 

## <a name="additional-information"></a>추가 정보

### <a name="manage-concurrency-using-etags"></a>Etag를 사용하여 동시성 관리
봇 코드 예제에서는 각 `IStoreItem`의 `eTag` 속성을 `*`로 설정합니다. 저장소 개체의 `eTag`(엔터티 태그) 멤버는 Cosmos DB 내에서 동시성을 관리하는 데 사용됩니다. `eTag`는 데이터베이스에 봇이 쓰고 있는 동일한 저장소에서 다른 봇 인스턴스가 개체를 변경할 경우 수행할 작업을 지시합니다. 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>마지막 쓰기 우선 - 덮어쓰기 허용 
별표(`*`)의 `eTag` 속성 값은 마지막 작성자가 우선임을 의미합니다. 새 데이터 저장소를 만들 때 속성의 `eTag`를 `*`로 설정하여 이전에 저장하지 않은 데이터를 쓰고 있거나, 마지막 작성자가 모든 이전에 저장한 속성을 덮어쓰기를 원한다고 표시합니다. 동시성은 봇에는 문제가 되지 않습니다. 쓰는 데이터에 대해 `eTag` 속성을 `*`로 설정하면 덮어쓰기를 활성화합니다.

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>동시성을 유지 관리 및 덮어쓰기 방지
데이터를 Cosmos DB로 저장할 때 속성에 대한 동시 액세스를 방지하려면 `eTag`에 `*` 이외의 값을 사용하여 다른 봇 인스턴스의 변경 내용 덮어쓰기를 차단합니다. 봇이 상태 데이터 저장을 시도하고 `eTag`가 저장소의 `eTag`와 동일한 값이 아니면 `etag conflict key=` 메시지와 함께 오류 응답을 받습니다. <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

기본적으로 Cosmos DB 저장소는 봇이 해당 항목에 쓸 때마다 저장소 개체의 `eTag` 속성이 같은지 확인한 다음, 모든 쓰기 후에 새 고유 값으로 업데이트합니다. 쓰기의 `eTag` 속성이 저장소의 `eTag`와 일치하지 않은 경우 다른 봇 또는 스레드가 해당 데이터를 변경한 것입니다. 

예를 들어, 봇이 저장된 메모를 편집했으나 이 봇이 다른 봇 인스턴스가 수행한 변경 내용을 덮어쓰는 것을 원치 않는다고 가정하겠습니다. 다른 봇 인스턴스가 편집을 수행한 경우 사용자가 최신 업데이트로 버전을 편집하길 바랍니다.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

먼저 `IStoreItem`를 구현하는 클래스를 만듭니다.

**EchoBot.cs**
```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

다음으로 저장소 개체를 만들어 최초 메모를 만들고 이 개체를 저장소에 추가합니다.

**EchoBot.cs**
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

**EchoBot.cs**
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


### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

데이터 저장소에 샘플 메모를 작성하는 도우미 함수를 봇의 끝에 추가합니다.
먼저 `myNoteData` 개체를 만듭니다.

**bot.js**
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

**bot.js**
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
     var list = changes["Note"].contents;
     await context.sendActivity(`Successful created a note: ${list}`);
} catch (err) {
     await context.sendActivity(`Could not create note: ${err}`);
}
```

다음 호출을 추가하여 봇의 논리 내에서 이 도우미 함수에 액세스합니다.

**bot.js**
```javascript
// Save a note with etag.
await createSampleNote(storage, turnContext);
```

생성되었으면 나중에 메모를 검색하고 업데이트하기 위해 사용자가 "메모 업데이트"를 입력할 때 액세스할 수 있는 다른 도우미 함수를 만듭니다.

**bot.js**
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
             var list = note["Note"].contents;
             await context.sendActivity(`Successfully updated note: ${list}`);
        } catch (err) {            
             console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

다음 호출을 추가하여 봇의 논리 내에서 이 도우미 함수에 액세스합니다.

**bot.js**
```javascript
// Update a note with etag.
await updateSampleNote(storage, turnContext);
```

변경 사항을 다시 작성하려고 하기 전에 다른 사용자가 저장소에서 메모를 업데이트한 경우 `eTag` 값이 더 이상 일치하지 않으며, `write`에 대한 호출이 예외를 throw합니다.

---

동시성을 유지하려면 항상 저장소에서 속성을 읽은 다음, 읽은 속성을 수정하여 `eTag`를 유지 관리합니다. 저장소에서 사용자 데이터를 읽은 경우 응답에 eTag 속성이 포함됩니다. 데이터를 변경하고 업데이트된 데이터를 저장소에 쓴 경우 요청에 앞서 읽은 것과 동일한 값을 지정하는 eTag 속성이 포함됩니다. 그러나 `eTag`가 `*`로 설정된 개체를 쓰면 이 쓰기가 다른 변경 내용을 덮어쓰게 됩니다.

## <a name="next-steps"></a>다음 단계
이제 저장소에서 직접 읽고 쓰는 방법을 알았으므로 상태 관리자를 사용하여 이를 수행하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [대화 및 사용자 속성을 사용하여 상태 저장](bot-builder-howto-v4-state.md)

