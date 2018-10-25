---
title: LUIS를 통해 의도 및 엔터티 인식 | Microsoft Docs
description: Node.js용 Bot Builder SDK를 사용하여 대화를 트리거하여 봇과 LUIS를 통합하여 사용자의 의도를 감지하고 적합하게 응대합니다.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/28/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5df1352241485bf95a46fa981b9b16c3cb7e3925
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998700"
---
# <a name="recognize-intents-and-entities-with-luis"></a>LUIS를 통해 의도 및 엔터티 인식 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

이 문서에서는 Note(메모)를 작성하는 봇 예제를 사용하여, Language Understanding([LUIS][LUIS])이 봇이 자연어 입력에 적절하게 응답하는 데 어떻게 도움이 될 수 있는지를 보여줍니다. 봇은 **의도**를 식별하여 사용자가 원하는 바를 감지합니다. 의도는 음성이나 텍스트 입력 또는 **발언**을 통해 결정됩니다. 의도는 대화 상자 호출처럼 봇이 취하는 동작에 발언을 매핑합니다. 봇은 발언에서 중요한 단어인 **엔터티**를 추출해야 할 수도 있습니다. 경우에 따라 의도 충족을 위해 엔터티가 필요합니다. 메모 작성 봇의 예에서 `Notes.Title` 엔터티는 각 메모의 제목을 식별합니다.

## <a name="create-a-language-understanding-bot-with-bot-service"></a>Bot Service를 사용하여 Language Understanding 봇 만들기

1. [Azure Portal](https://portal.azure.com)의 메뉴 블레이드에서 **새 리소스 만들기**를 선택하고 **모두 표시**를 클릭합니다.

    ![새 리소스 만들기](../media/bot-builder-nodejs-use-luis/bot-service-creation.png)

2. 검색 상자에서 **웹앱 봇**을 검색합니다. 

    ![새 리소스 만들기](../media/bot-builder-nodejs-use-luis/bot-service-selection.png)

3. **Bot Service** 블레이드에서 필요한 정보를 제공하고 **만들기**를 클릭합니다. 이렇게 하면 Bot Service 및 LUIS 앱이 만들어지고 Azure에 배포됩니다. 
   * **앱 이름**을 봇 이름으로 설정합니다. 이 이름은 봇이 클라우드에 배포될 때 하위 도메인으로 사용됩니다(예: mynotesbot.azurewebsites.net). 이 이름은 봇과 연결된 LUIS 앱의 이름으로도 사용됩니다. 나중에 봇과 관련된 LUIS 앱을 찾을 때 사용할 수 있도록 복사합니다.
   * 구독, [리소스 그룹](/azure/azure-resource-manager/resource-group-overview), App Service 계획 및 [위치](https://azure.microsoft.com/en-us/regions/)를 선택합니다.
   * **봇 템플릿** 필드에 **Language Understanding(Node.js)** 템플릿을 선택합니다.

     ![Bot Service 블레이드](../media/bot-builder-nodejs-use-luis/bot-service-setting-callout-template.png)

   * 상자를 선택하여 서비스 약관을 확인합니다.

4. Bot Service가 배포되었는지 확인합니다.
    * 알림(Azure Portal의 위쪽 가장자리에 있는 벨 아이콘)을 클릭합니다. 알림이 **배포가 시작됨**에서 **배포 성공**으로 변경됩니다.
    * 알림이 **배포 성공**으로 변경된 후 해당 알림에서 **리소스로 이동**을 클릭합니다.

## <a name="try-the-bot"></a>봇 사용해 보기

**알림**을 선택하여 봇이 배포되었는지 확인합니다. 알림이 **배포 진행 중...** 에서 **배포 성공**으로 변경됩니다. **리소스로 이동** 단추를 클릭하여 봇의 리소스 블레이드를 엽니다.

봇이 등록되면 **웹 채팅에서 테스트**를 클릭하여 [웹 채팅] 창을 엽니다. 웹 채팅에 “hello”를 입력합니다.

  ![웹 채팅에서 봇 테스트](../media/bot-builder-nodejs-use-luis/bot-service-web-chat.png)

봇이 다음과 같이 응답합니다. “You have reached Greeting. You said: hello”. 이는 봇이 메시지를 수신했고 봇이 만든 기본 LUIS 앱에 메시지를 전달했음을 확인합니다. 이 기본 LUIS 앱이 Greeting 의도를 검색했습니다.

## <a name="modify-the-luis-app"></a>LUIS 앱 수정

Azure에 로그인하는 데 사용하는 계정과 동일한 계정을 사용하여 [https://www.luis.ai](https://www.luis.ai)에 로그인합니다. **내 앱**을 클릭합니다. 앱 목록에서 봇 서비스를 만들 때 **Bot Service** 블레이드의 **앱 이름**에 지정된 이름으로 시작하는 앱을 찾습니다. 

LUIS 앱은 취소, 인사말, 도움말 및 없음이라는 4가지 의도로 시작합니다. <!-- picture -->

다음 단계에서는 Note.Create, Note.ReadAloud 및 Note.Delete 의도를 추가합니다. 

1. 페이지의 왼쪽 아래에서 **Prebuit Domains**(미리 빌드된 도메인)을 클릭합니다. **메모** 도메인을 찾아서 **도메인 추가**를 클릭합니다.

2. 이 자습서에서는 미리 빌드된 **메모** 도메인에 포함된 의도 중 일부만 사용합니다. **의도** 페이지에서 다음 의도 이름 각각을 클릭한 다음, **의도 삭제** 단추를 클릭합니다.
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   LUIS 앱에 남아있어야 하는 유일한 의도는 다음과 같습니다. 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * 없음
   * 도움말
   * Greeting
   * 취소

     ![LUIS 앱에 표시된 의도](../media/bot-builder-nodejs-use-luis/luis-intent-list.png)


3.  오른쪽 위에 있는 **학습** 단추를 클릭하여 앱을 학습시킵니다.
4.  위쪽 탐색 모음에 있는 **게시**를 클릭하여 **게시** 페이지를 엽니다. **프로덕션 슬롯에 게시** 단추를 클릭합니다. 게시에 성공하면 **앱 게시** 페이지의 **엔드포인트** 열에 표시된 URL에서 Resource Name Starter_Key로 시작되는 행에 LUIS 앱이 배포됩니다. URL은 `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=` 예제와 유사한 형식입니다.  이 URL의 앱 ID와 구독 키는 ** App Service 설정 > ApplicationSettings > 앱 설정 **의 LuisAppId 및 LuisAPIKey와 동일합니다.


## <a name="modify-the-bot-code"></a>봇 코드 수정

**빌드**를 클릭한 다음, **온라인 코드 편집기 열기**를 클릭합니다.

   ![온라인 코드 편집기 열기](../media/bot-builder-nodejs-use-luis/bot-service-build.png)

코드 편집기에서 `app.js`를 엽니다. 다음 코드를 포함합니다.

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send('You reached the default message handler. You said \'%s\'.', session.message.text);
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// Add a dialog for each intent that the LUIS app recognizes.
// See https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-recognize-intent-luis 
bot.dialog('GreetingDialog',
    (session) => {
        session.send('You reached the Greeting intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Greeting'
})

bot.dialog('HelpDialog',
    (session) => {
        session.send('You reached the Help intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Help'
})

bot.dialog('CancelDialog',
    (session) => {
        session.send('You reached the Cancel intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Cancel'
})

```


> [!TIP] 
> 이 문서에서 설명하는 샘플 코드는 [Notes 봇 샘플][NotesSample]에도 있습니다.



## <a name="edit-the-default-message-handler"></a>기본 메시지 처리기 편집
봇에는 기본 메시지 처리기가 있습니다. 다음과 일치하도록 편집합니다. 
```javascript
// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});
```

## <a name="handle-the-notecreate-intent"></a>Note.Create 의도 처리

다음 코드를 복사하여 app.js의 마지막에 붙여넣습니다.

```javascript
// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});
```

발언의 모든 엔터티는 `args` 매개 변수를 사용하여 대화에 전달됩니다. [waterfall][waterfall]의 첫 단계는 [EntityRecognizer.findEntity][EntityRecognizer_findEntity]를 호출하여 LUIS 응답의 모든 `Note.Title` 엔터티에서 메모 제목을 가져오는 것입니다. LUIS 앱이 `Note.Title` 엔터티를 검색하지 않은 경우 봇은 사용자에게 메모 이름 입력을 요청하는 프롬프트를 표시합니다. waterfall의 두 번째 단계는 메모에 포함할 텍스트 입력을 요청하는 프롬프트입니다. 봇이 메모의 텍스트를 갖게 된 후 세 번째 단계는 제목을 키로 사용하여 [session.userData][session_userData]로 `notes` 개체에 메모를 저장하는 것입니다. `session.UserData`에 대한 자세한 내용은 [상태 데이터 관리](./bot-builder-nodejs-state.md)를 참조하세요. 



## <a name="handle-the-notedelete-intent"></a>Note.Delete 의도 처리
`Note.Create` 의도처럼 봇은 제목에 대해 `args` 매개 변수를 검사합니다. 제목이 없으면 봇이 사용자에게 입력하라는 프롬프트를 표시합니다. 제목은 `session.userData.notes`에서 삭제할 메모를 조회하는 데 사용됩니다. 



다음 코드를 복사하여 app.js의 마지막에 붙여넣습니다.
```javascript
// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});
```

`Note.Delete`를 처리하는 코드는 `noteCount` 함수를 사용하여 `notes` 개체에 메모가 포함되었는지 여부를 판단합니다. 

`noteCount` 도우미 함수를 `app.js`의 끝에 붙여넣습니다.

[!code-js[Add a helper function that returns the number of notes (JavaScript)](../includes/code/node-basicNote.js#CountNotesHelper)]

## <a name="handle-the-notereadaloud-intent"></a>Note.ReadAloud 의도 처리

다음 코드를 복사하여 `app.js`에서 `Note.Delete`에 대한 처리기 다음에 붙여넣습니다.

```javascript
// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});

```

`session.userData.notes` 개체가 세 번째 인수로 `builder.Prompts.choice`에 전달되므로 프롬프트가 사용자에게 메모 목록을 표시합니다.

이제 새 의도에 대한 처리기를 추가했으므로 `app.js`의 전체 코드에 다음이 포함됩니다.

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});

// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});


// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});


// Helper function to count the number of notes stored in session.userData.notes
function noteCount(notes) {

    var i = 0;
    for (var name in notes) {
        i++;
    }
    return i;
}
```

## <a name="test-the-bot"></a>봇 테스트

Azure Portal에서 **Web Chat에서 테스트**를 클릭하여 봇을 테스트합니다. "메모 만들기", "내 메모 읽기", "메모 삭제" 같은 메시지를 입력하여 추가한 의도를 호출합니다.
   ![Web Chat에서 메모 봇 테스트](../media/bot-builder-nodejs-use-luis/bot-service-test-notebot.png)

> [!TIP]
> 봇이 항상 올바른 의도나 엔터티를 인식하지는 않는다는 것을 알게 되면 더 많은 예제 발화를 제공하여 학습시키는 방식으로 LUIS 앱 성능을 개선합니다. 봇의 코드를 수정하지 않고 LUIS 앱을 다시 학습시킬 수 있습니다. [예제 발화 추가](/azure/cognitive-services/LUIS/add-example-utterances) 및 [LUIS 앱 학습 및 테스트](/azure/cognitive-services/LUIS/train-test)를 참조하세요.


## <a name="next-steps"></a>다음 단계

봇 테스트를 통해 인식기가 현재 활성 상태인 대화 상자의 중단을 트리거할 수 있음을 확인했습니다. 중단을 허용하고 처리하는 것이 사용자가 실제로 수행하는 일을 설명하는 유연한 디자인입니다. 인식된 의도에 연결할 수 있는 다양한 작업에 자세히 알아보세요.

> [!div class="nextstepaction"]
> [사용자 작업 처리](bot-builder-nodejs-dialog-actions.md)


[LUIS]: https://www.luis.ai/

[intentDialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html

[intentDialog_matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html#matches 

[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/Node/basics-naturalLanguage

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[confirmPrompt]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#confirmprompt

[waterfall]: bot-builder-nodejs-dialog-manage-conversation-flow.md#manage-conversation-flow-with-a-waterfall

[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata

[EntityRecognizer_findEntity]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#findentity

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[Dialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html

[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISConcepts]: https://docs.botframework.com/en-us/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
