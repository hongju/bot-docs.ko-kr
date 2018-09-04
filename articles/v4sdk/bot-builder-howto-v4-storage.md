---
title: 저장소에 직접 작성 | Microsoft Docs
description: .NET용 Bot Builder SDK V4를 통해 직접 저장소에 기록하는 방법을 살펴봅니다.
keywords: 저장소, 읽기 및 쓰기, 메모리 저장소, eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/2/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 76f8976aefe3d4fefcffc46e691dbd0b35e41ec7
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756437"
---
# <a name="write-directly-to-storage"></a>저장소에 직접 작성

<!--
 Note for V4: You can write directly to storage without using the state manager. Therefore, this topic isn't called "managing state". State is in a separate topic.
 ## Manage state data by writing directly to storage
-->
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

컨텍스트 개체나 미들웨어를 사용하지 않고 저장소 개체에서 직접 읽고 써서 저장소에 직접 쓸 수 있습니다.

이 코드 예제는 저장소에 데이터를 쓰고 읽는 방법을 보여 줍니다. 이 예제에서는 저장소가 파일이지만 코드를 간편하게 변경하여 Azure Table Storage의 메모리 저장소를 대신 사용하게 저장소 개체를 초기화할 수 있습니다. 

# <a name="ctabcsharpechorproperty"></a>[C#](#tab/csharpechorproperty)
개체를 정의하고 `IStorage.Write` 및 `IStorage.Read`를 사용하여 상태를 저장 및 검색합니다. 

다음 샘플은 사용자의 모든 메시지를 목록에 추가합니다. 목록을 포함하는 데이터 구조는 `FileStorage` 생성에 제공한 디렉터리 내 파일에 저장됩니다.

BotBuilder V4 SDK에서 EchoBot Visual Studio 템플릿을 시작합니다.
`EchoBot.cs` 파일을 편집합니다. 명령문을 사용하여 다음을 추가하고 `EchoBot` 클래스 정의를 바꿉니다.

```csharp
using System.Collections.Generic;
using System.Linq;
```

```csharp
// In the constructor initialize file storage
public class EchoBot : IBot
{
    private readonly FileStorage _myStorage;

    public EchoBot()
    {
        _myStorage = new FileStorage(System.IO.Path.GetTempPath());
    }

    // Add a class for storing a log of utterances (text of messages) as a list
    public class UtteranceLog : IStoreItem
    {
        // A list of things that users have said to the bot
        public List<string> UtteranceList { get; private set; } = new List<string>();

        // The number of conversational turns that have occurred        
        public int TurnNumber { get; set; } = 0;

        public string eTag { get; set; } = "*";
    }

    // Replace the OnTurn in EchoBot.cs with the following:
    public async Task OnTurn(ITurnContext context)
    {
        var activityType = context.Activity.Type;

        await context.SendActivity($"Activity type: {context.Activity.Type}.");

        if (activityType == ActivityTypes.Message)
        {
            // *********** begin (create or add to log of messages)
            var utterance = context.Activity.Text;
            bool restartList = false;

            if (utterance.Equals("restart"))
            {
                restartList = true;
            }

            // Attempt to read the existing property bag
            UtteranceLog logItems = null;
            try
            {
                logItems = _myStorage.Read<UtteranceLog>("UtteranceLog").Result?.FirstOrDefault().Value;
            }
            catch (System.Exception ex)
            {
                await context.SendActivity(ex.Message);
            }

            // If the property bag wasn't found, create a new one
            if (logItems is null)
            {
                try
                {
                    // add the current utterance to a new object.
                    logItems = new UtteranceLog();
                    logItems.UtteranceList.Add(utterance);

                    await context.SendActivity($"The list is now: {string.Join(", ",logItems.UtteranceList)}");

                    var changes = new KeyValuePair<string, object>[]
                    {
                        new KeyValuePair<string, object>("UtteranceLog", logItems)
                    };
                    await _myStorage.Write(changes);
                }
                catch (System.Exception ex)
                {
                    await context.SendActivity(ex.Message);
                }
            }
            // logItems.ContainsKey("UtteranceLog") == true, we were able to read a log from storage
            else
            {
                // Modify its property
                if (restartList)
                {
                    logItems.UtteranceList.Clear();
                }
                else
                {
                    logItems.UtteranceList.Add(utterance);
                    logItems.TurnNumber++;
                }

                await context.SendActivity($"The list is now: {string.Join(", ", logItems.UtteranceList)}");

                var changes = new KeyValuePair<string, object>[]
                {
                        new KeyValuePair<string, object>("UtteranceLog", logItems)
                };
                await _myStorage.Write(changes);
            }
        }

        return;
    }
}
```
# <a name="javascripttabjsechoproperty"></a>[JavaScript](#tab/jsechoproperty)

다음 샘플은 사용자의 모든 메시지를 목록에 추가합니다. 목록을 포함하는 데이터 구조는 `FileStorage` 생성에 제공한 디렉터리 내 파일에 저장됩니다.

다음을 `app.js`에 붙여넣습니다.
``` javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState, BotStateSet } = require('botbuilder');
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

// Add storage.
var storage = new FileStorage("C:/temp");
const conversationState = new ConversationState(storage);
adapter.use(new BotStateSet(conversationState));

// Listen for incoming activities.
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing.
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            let utterance = context.activity.text;
            let storeItems = await storage.read(["UtteranceLog"])
            try {
                // check result
                var utteranceLog = storeItems["UtteranceLog"];
                
                if (typeof (utteranceLog) != 'undefined') {
                    // log exists so we can write to it
                    storeItems["UtteranceLog"].UtteranceList.push(utterance);
                    
                    await storage.write(storeItems)
                    try {
                        context.sendActivity('Successful write.');
                    } catch (err) {
                        context.sendActivity(`Srite failed of UtteranceLog: ${err}`);
                    }

                } else {
                    context.sendActivity(`need to create new utterance log`);
                    storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }
                    await storage.write(storeItems)
                    try {
                        context.sendActivity('Successful write.');
                    } catch (err) {
                        context.sendActivity(`Write failed: ${err}`);
                    }
                }
            } catch (err) {
                context.sendActivity(`Read rejected. ${err}`);
            };

            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="manage-concurrency-using-etags"></a>Etag를 사용하여 동시성 관리

이전 예제에서 `eTag` 속성을 `*`로 설정합니다. 저장소의 `eTag`(엔터티 태그) 멤버는 동시성 관리를 위한 것입니다. `eTag`는 봇이 쓰고 있는 저장소에서 다른 봇 인스턴스가 개체를 변경할 경우 수행할 작업을 표시합니다. 

<!-- define optimistic concurrency -->

### <a name="last-write-wins---allow-overwrites"></a>마지막 쓰기 우선 - 덮어쓰기 허용 

ETag를 `*`로 설정하면 다른 봇 인스턴스가 이전에 쓴 데이터를 덮어쓸 수 있습니다. 별표(`*`)의 `eTag` 속성 값은 마지막 작성자가 우선임을 의미합니다. 새 데이터 저장소를 만들 때 속성의 `eTag`를 `*`로 설정하여 이전에 저장하지 않은 데이터를 쓰고 있거나, 마지막 작성자가 모든 이전에 저장한 속성을 덮어쓰기를 원한다고 표시합니다. 동시성은 봇에는 문제가 되지 않습니다. 쓰는 데이터에 대해 `eTag` 속성을 `*`로 설정하면 덮어쓰기를 허용합니다.

다음 코드 예제에 이러한 내용이 나와 있습니다.

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)
앞서 정의한 `UtteranceLog` 클래스를 사용합니다.
```csharp
// Add the current utterance to a new object and save it to storage.
logItems = new UtteranceLog();
logItems.UtteranceList.Add(utterance);
logItems.eTag = "*";

var changes = new KeyValuePair<string, object>[]
{
    new KeyValuePair<string, object>("UtteranceLog", logItems)
};
await _myStorage.Write(changes);

```
# <a name="javascripttabjstagoverwrite"></a>[JavaScript](#tab/jstagoverwrite)
로그에 새 발화를 추가하고 덮어쓰기를 허용합니다.

```javascript
storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }
await storage.write(storeItems)
```
---

### <a name="maintain-concurrency-and-prevent-overwrites"></a>동시성을 유지 관리 및 덮어쓰기 방지
속성에 대한 동시 액세스를 방지하려면 `eTag`에 `*` 이외의 값을 사용하여 다른 봇 인스턴스의 변경 내용 덮어쓰기를 차단합니다. 봇이 상태 데이터 저장을 시도하고 `eTag`가 저장소의 것과 같지 않으면 `etag conflict key=` 메시지와 함께 오류 응답을 받습니다.  <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

기본적으로 저장소는 봇이 해당 항목에 쓸 때마다 저장소 개체의 `eTag` 속성이 같은지 확인한 다음, 모든 쓰기 후에 새 고유 값으로 업데이트합니다. 쓰기의 `eTag` 속성이 저장소의 `eTag`와 일치하지 않은 경우 다른 봇 또는 스레드가 해당 데이터를 변경한 것입니다. 

예를 들어, 봇이 저장된 메모를 편집했으나 이 봇이 다른 봇 인스턴스가 수행한 변경 내용을 덮어쓰는 것을 원치 않는다고 가정하겠습니다. 다른 봇 인스턴스가 편집을 수행한 경우 사용자가 최신 업데이트로 버전을 편집하길 바랍니다.

# <a name="ctabcsetag"></a>[C#](#tab/csetag)
먼저 `IStoreItem`를 구현하는 클래스를 만듭니다.
```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string eTag { get; set; }
}
```
다음으로 저장소 개체를 만들어 최초 메모를 만들고 이 개체를 저장소에 추가합니다.
```csharp
// create a note for the first time, with a non-null, non-* eTag.
var note = new Note { Name = "Shopping List", Contents = "eggs", eTag = "x" };

var changes = new KeyValuePair<string, object>[]
{
    new KeyValuePair<string, object>("Note", note)
};
await NoteStore.Write(changes);
```
그런 다음, 나중에 메모를 액세스 및 업데이트하여 저장소에서 읽은 `eTag`를 유지합니다. 
```csharp
var note = NoteStore.Read<Note>("Note").Result.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new KeyValuePair<string, object>[]
    {
        new KeyValuePair<string, object>("Note1", note)
    };
    await NoteStore.Write(changes);
}
```
변경 내용을 기록하기 전에 저장소에서 메모가 업데이트된 경우 `Write` 호출에서 예외가 throw됩니다.

# <a name="javascripttabjsetag"></a>[JavaScript](#tab/jsetag)

먼저 `myNote` 개체를 만듭니다.
```javascript
var myNote = {
    name: "Shopping List",
    contents: "eggs",
    eTag: "*"
}
```
다음으로 `changes` 개체를 초기화하고 *메모*를 추가한 다음, 저장소에 기록합니다.

```javascript
// Write a note
var changes = {};
changes["Note"] = myNote;
await storage.write(changes);
```
그런 다음, 나중에 메모를 액세스 및 업데이트하여 저장소에서 읽은 `eTag`를 유지합니다. 
```javascript
// Read in a note
var note = await storage.read(["Note"]);
try {
    // Someone has updated the note. Need to update our local instance to match
    if(myNote.eTag != note.eTag){
        myNote = note;
    }

    // Add any updates to the note and write back out
    myNote.contents += ", bread";   // Add to the note
    changes["Note"] = note;
    await storage.write(changes); // Write the changes back to storage
    try {
        context.sendActivity('Successful write.');
    } catch (err) {
        context.sendActivity(`Write failed: ${err}`);
    }
}
catch (err) {
    context.sendActivity(`Unable to read the Note: ${err}`);
}
```
변경 내용을 기록하기 전에 저장소에서 메모가 업데이트된 경우 `Write` 호출에서 예외가 throw됩니다.

---

동시성을 유지하려면 항상 저장소에서 속성을 읽은 다음, 읽은 속성을 수정하여 `eTag`를 유지 관리합니다. 저장소에서 사용자 데이터를 읽은 경우 응답에 eTag 속성이 포함됩니다. 데이터를 변경하고 업데이트된 데이터를 저장소에 쓴 경우 요청에 앞서 읽은 것과 동일한 값을 지정하는 eTag 속성이 포함됩니다. 그러나 `eTag`가 `*`로 설정된 개체를 쓰면 이 쓰기가 다른 변경 내용을 덮어쓰게 됩니다.

<!-- If the ETag specified in your `Storage.Write()` request matches the current value in the store, the server will save the data and specify a new eTag value in the body of the response, that indicates that the data has been updated. If the ETag specified in your Storage.Write() request does not match the current value in the store, the bot responds with an error indicating an eTag conflict, to indicate that the user's data in the store has changed since you last saved or retrieved it. -->

<!-- TODO: new snippet -->

<!-- jf: I think this section can be cut entirely.

## Save a conversation reference using storage

You can use IStorage to save BotBuilder SDK objects as well as user-defined data. This code snippet uses `Storage.Write()` to save a ConversationReference object for use in sending a proactive message later.

# [C#](#tab/csharpwriteconvref)
```csharp
            var reference = context.ConversationReference;
            var userId = reference.User.Id;

            // save the ConversationReference to a global variable of this class
            conversationReference = reference;

            StoreItems storeItems = new StoreItems();
            StoreItem conversationReferenceToStore = new StoreItem();
            // set the eTag to "*" to indicate you're overwriting previous data
            conversationReferenceToStore.eTag = "*";
            conversationReferenceToStore.Add("ref", reference);
            storeItems[$"ConversationReference/{userId}"] = conversationReferenceToStore;

            await storage.Write(storeItems).ConfigureAwait(false);

            SubscribeUser(userId);

            await context.SendActivity("Thank You! We will message you shortly.");
        }

public void SubscribeUser(string userId)
{
    CreateContextForUserAsync(userId, async (ITurnContext context) =>
    {
        await context.SendActivity("You've been notified.");
        await Task.Delay(2000);
    });
                
}
```
# [JavaScript](#tab/jswriteconvref)
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
                const reference = context.activity;
                const userId = reference.id;
                const changes = {};
                changes['reference/' + userId] = reference;
                await storage.write(changes)
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe'");
            }
    
        }
    });
});
```
---

To read the saved object from storage, call `Storage.Read()`.

# [C#](#tab/csharpread)
```csharp


        private async void CreateContextForUserAsync(String userId,Func<ITurnContext, Task> onReady)
        {
            var referenceKey = $"ConversationReference/{userId}";
            
            ConversationReference localRef = null;
            StoreItems conversationReferenceFromStorage = await storage.Read(referenceKey);
            foreach (var item in conversationReferenceFromStorage)
            {
                StoreItem value = item.Value as StoreItem;
                localRef = value["ref"];
            }
            
            await bot.CreateContext(this.conversationReference, onReady);

        }
```
# [JavaScript](#tab/jsread)
```javascript
async function subscribeUser(userId) {
    setTimeout(() => {
        createContextForUser(userId, (context) => {
            context.sendActivity("You have been notified");
        })
    }, 2000);
}

async function createContextForUser(userId, callback) {
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]
    await callback(adapter.createContext(reference))
          
}
```
---

-->

## <a name="next-steps"></a>다음 단계

이제 저장소에서 직접 읽고 쓰는 방법을 알았으므로 상태 관리자를 사용하여 이를 수행하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [대화 및 사용자 속성을 사용하여 상태 저장](bot-builder-howto-v4-state.md)

## <a name="additional-resources"></a>추가 리소스

- [개념: .Bot Builder SDK의 저장소](bot-builder-storage-concept.md)


