---
title: LUIS를 통해 의도 및 엔터티 인식 | Microsoft Docs
description: .NET용 Bot Builder SDK에서 LUIS 대화를 사용하여 봇이 자연어를 이해하도록 설정하는 방법을 알아봅니다.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f95335149fa2c896d905834832089ffbfa960bf2
ms.sourcegitcommit: d4afc924b0e1907c4d6f7a6fc5ac1fe521aeef7e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47447394"
---
# <a name="recognize-intents-and-entities-with-luis"></a>LUIS를 통해 의도 및 엔터티 인식 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

이 문서에서는 Note(메모)를 작성하는 봇 예제를 사용하여, Language Understanding([LUIS][LUIS])이 봇이 자연어 입력에 적절하게 응답하는 데 어떻게 도움이 될 수 있는지를 보여줍니다. 봇은 **의도**를 식별하여 사용자가 원하는 바를 감지합니다. 의도는 음성이나 텍스트 입력 또는 **발언**을 통해 결정됩니다. 의도는 봇이 취하는 동작에 발언을 매핑합니다. 예를 들어 Note(메모)를 작성하는 봇은 `Notes.Create` 의도를 인식하여 메모를 생성하는 기능을 호출합니다. 봇은 발언에서 중요한 단어인 **엔터티**를 추출해야 할 수 도 있습니다. 메모 작성 봇의 예에서 `Notes.Title` 엔터티는 각 메모의 제목을 식별합니다.

## <a name="create-a-language-understanding-bot-with-bot-service"></a>Bot Service를 사용하여 Language Understanding 봇 만들기

1. [Azure Portal](https://portal.azure.com)의 메뉴 블레이드에서 **새 리소스 만들기**를 선택하고 **모두 표시**를 클릭합니다.

    ![새 리소스 만들기](../media/bot-builder-dotnet-use-luis/bot-service-creation.png)

2. 검색 상자에서 **웹앱 봇**을 검색합니다. 

    ![새 리소스 만들기](../media/bot-builder-dotnet-use-luis/bot-service-selection.png)

3. **Bot Service** 블레이드에서 필요한 정보를 제공하고 **만들기**를 클릭합니다. 이렇게 하면 Bot Service 및 LUIS 앱이 만들어지고 Azure에 배포됩니다. 
   * **앱 이름**을 봇 이름으로 설정합니다. 이 이름은 봇이 클라우드에 배포될 때 하위 도메인으로 사용됩니다(예: mynotesbot.azurewebsites.net). 이 이름은 봇과 연결된 LUIS 앱의 이름으로도 사용됩니다. 나중에 봇과 관련된 LUIS 앱을 찾을 때 사용할 수 있도록 복사합니다.
   * 구독, [리소스 그룹](/azure/azure-resource-manager/resource-group-overview), App Service 계획 및 [위치](https://azure.microsoft.com/en-us/regions/)를 선택합니다.
   * **봇 템플릿** 필드에 **Language Understanding(C#)** 템플릿을 선택합니다.

     ![Bot Service 블레이드](../media/bot-builder-dotnet-use-luis/bot-service-setting-callout-template.png)

   * 상자를 선택하여 서비스 약관을 확인합니다.

4. Bot Service가 배포되었는지 확인합니다.
    * 알림(Azure Portal의 위쪽 가장자리에 있는 벨 아이콘)을 클릭합니다. 알림이 **배포가 시작됨**에서 **배포 성공**으로 변경됩니다.
    * 알림이 **배포 성공**으로 변경된 후 해당 알림에서 **리소스로 이동**을 클릭합니다.

## <a name="try-the-bot"></a>봇 사용해 보기

**알림**을 선택하여 봇이 배포되었는지 확인합니다. 알림이 **배포 진행 중...** 에서 **배포 성공**으로 변경됩니다. **리소스로 이동** 단추를 클릭하여 봇의 리소스 블레이드를 엽니다.

봇이 등록되면 **웹 채팅에서 테스트**를 클릭하여 [웹 채팅] 창을 엽니다. 웹 채팅에 “hello”를 입력합니다.

  ![웹 채팅에서 봇 테스트](../media/bot-builder-dotnet-use-luis/bot-service-web-chat.png)

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

     ![LUIS 앱에 표시된 의도](../media/bot-builder-dotnet-use-luis/luis-intent-list.png)

3. 오른쪽 위에 있는 **학습** 단추를 클릭하여 앱을 학습시킵니다.
4. 위쪽 탐색 모음에 있는 **게시**를 클릭하여 **게시** 페이지를 엽니다. **Publish to production slot**(프로덕션 슬롯 게시) 단추를 클릭합니다. 게시에 성공하면 **앱 게시** 페이지의 **엔드포인트** 열에 표시된 URL을 리소스 이름이 Starter_Key로 시작하는 행에 복사합니다. URL을 나중에 봇의 코드에 사용할 수 있도록 저장합니다. URL은 아래 예와 유사한 형식입니다. `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`

## <a name="modify-the-bot-code"></a>봇 코드 수정

**빌드**를 클릭한 다음, **온라인 코드 편집기 열기**를 클릭합니다.
    ![온라인 코드 편집기 열기](../media/bot-builder-dotnet-use-luis/bot-service-build.png)

코드 편집기에서 `BasicLuisDialog.cs`를 엽니다. LUIS 앱에서 의도를 처리하는 다음과 같은 코드가 포함되어 있습니다.
```cs
using System;
using System.Configuration;
using System.Threading.Tasks;

using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

namespace Microsoft.Bot.Sample.LuisBot
{
    // For more information about this template visit http://aka.ms/azurebots-csharp-luis
    [Serializable]
    public class BasicLuisDialog : LuisDialog<object>
    {
        public BasicLuisDialog() : base(new LuisService(new LuisModelAttribute(
            ConfigurationManager.AppSettings["LuisAppId"], 
            ConfigurationManager.AppSettings["LuisAPIKey"], 
            domain: ConfigurationManager.AppSettings["LuisAPIHostName"])))
        {
        }

        [LuisIntent("None")]
        public async Task NoneIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        // Go to https://luis.ai and create a new intent, then train/publish your luis app.
        // Finally replace "Greeting" with the name of your newly created intent in the following handler
        [LuisIntent("Greeting")]
        public async Task GreetingIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Cancel")]
        public async Task CancelIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Help")]
        public async Task HelpIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        private async Task ShowLuisResult(IDialogContext context, LuisResult result) 
        {
            await context.PostAsync($"You have reached {result.Intents[0].Intent}. You said: {result.Query}");
            context.Wait(MessageReceived);
        }
    }
}
```
### <a name="create-a-class-for-storing-notes"></a>Note(메모) 저장용 클래스 만들기

BasicLuisDialog.cs에 다음 `using` 문을 추가합니다.

```cs
using System.Collections.Generic;
```

`BasicLuisDialog` 클래스 내에서 생성자 정의 뒤에 다음 코드를 추가합니다.

```cs
        // Store notes in a dictionary that uses the title as a key
        private readonly Dictionary<string, Note> noteByTitle = new Dictionary<string, Note>();
        
        [Serializable]
        public sealed class Note : IEquatable<Note>
        {

            public string Title { get; set; }
            public string Text { get; set; }

            public override string ToString()
            {
                return $"[{this.Title} : {this.Text}]";
            }

            public bool Equals(Note other)
            {
                return other != null
                    && this.Text == other.Text
                    && this.Title == other.Title;
            }

            public override bool Equals(object other)
            {
                return Equals(other as Note);
            }

            public override int GetHashCode()
            {
                return this.Title.GetHashCode();
            }
        }

        // CONSTANTS        
        // Name of note title entity
        public const string Entity_Note_Title = "Note.Title";
        // Default note title
        public const string DefaultNoteTitle = "default";
```

### <a name="handle-the-notecreate-intent"></a>Note.Create 의도 처리
Note.Create 의도를 처리하려면 다음 코드를 `BasicLuisDialog` 클래스에 추가합니다.

```cs
        private Note noteToCreate;
        private string currentTitle;
        [LuisIntent("Note.Create")]
        public Task NoteCreateIntent(IDialogContext context, LuisResult result)
        {
            EntityRecommendation title;
            if (!result.TryFindEntity(Entity_Note_Title, out title))
            {
                // Prompt the user for a note title
                PromptDialog.Text(context, After_TitlePrompt, "What is the title of the note you want to create?");
            }
            else
            {
                var note = new Note() { Title = title.Entity };
                noteToCreate = this.noteByTitle[note.Title] = note;

                // Prompt the user for what they want to say in the note           
                PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");
            }
            
            return Task.CompletedTask;
        }
        
        
        private async Task After_TitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            EntityRecommendation title;
            // Set the title (used for creation, deletion, and reading)
            currentTitle = await result;
            if (currentTitle != null)
            {
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = currentTitle };
            }
            else
            {
                // Use the default note title
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = DefaultNoteTitle };
            }

            // Create a new note object 
            var note = new Note() { Title = title.Entity };
            // Add the new note to the list of notes and also save it in order to add text to it later
            noteToCreate = this.noteByTitle[note.Title] = note;

            // Prompt the user for what they want to say in the note           
            PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");

        }

        private async Task After_TextPrompt(IDialogContext context, IAwaitable<string> result)
        {
            // Set the text of the note
            noteToCreate.Text = await result;
            
            await context.PostAsync($"Created note **{this.noteToCreate.Title}** that says \"{this.noteToCreate.Text}\".");
            
            context.Wait(MessageReceived);
        }
```

### <a name="handle-the-notereadaloud-intent"></a>Note.ReadAloud 의도 처리
봇은 `Note.ReadAloud` 의도를 사용하여 하나의 Note(메모) 또는 모든 Note(노트 제목이 감지되지 않는 경우)의 의도를 표시합니다.

다음 코드를 `BasicLuisDialog` 클래스에 붙여넣습니다.
```cs
        [LuisIntent("Note.ReadAloud")]
        public async Task NoteReadAloudIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                await context.PostAsync($"**{note.Title}**: {note.Text}.");
            }
            else
            {
                // Print out all the notes if no specific note name was detected
                string NoteList = "Here's the list of all notes: \n\n";
                foreach (KeyValuePair<string, Note> entry in noteByTitle)
                {
                    Note noteInList = entry.Value;
                    NoteList += $"**{noteInList.Title}**: {noteInList.Text}.\n\n";
                }
                await context.PostAsync(NoteList);
            }

            context.Wait(MessageReceived);
        }
        
        public bool TryFindNote(string noteTitle, out Note note)
        {
            bool foundNote = this.noteByTitle.TryGetValue(noteTitle, out note); // TryGetValue returns false if no match is found.
            return foundNote;
        }
        
        public bool TryFindNote(LuisResult result, out Note note)
        {
            note = null;

            string titleToFind;

            EntityRecommendation title;
            if (result.TryFindEntity(Entity_Note_Title, out title))
            {
                titleToFind = title.Entity;
            }
            else
            {
                titleToFind = DefaultNoteTitle;
            }

            return this.noteByTitle.TryGetValue(titleToFind, out note); // TryGetValue returns false if no match is found.
        }
```

### <a name="handle-the-notedelete-intent"></a>Note.Delete 의도 처리
다음 코드를 `BasicLuisDialog` 클래스에 붙여넣습니다.

```cs
        [LuisIntent("Note.Delete")]
        public async Task NoteDeleteIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {                             
                // Prompt the user for a note title
                PromptDialog.Text(context, After_DeleteTitlePrompt, "What is the title of the note you want to delete?");                         
            }           
        }

        private async Task After_DeleteTitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            Note note;
            string titleToDelete = await result;
            bool foundNote = this.noteByTitle.TryGetValue(titleToDelete, out note);

            if (foundNote)
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {
                await context.PostAsync($"Did not find note named {titleToDelete}.");
            }

            context.Wait(MessageReceived);
        }
```

## <a name="build-the-bot"></a>봇 빌드
코드 편집기에서 **build.cmd**를 마우스 오른쪽 단추로 클릭하고 **Run from Console**(콘솔에서 실행)을 선택합니다.

   ![build.cmd 실행](../media/bot-builder-dotnet-use-luis/bot-service-run-console.png)

## <a name="test-the-bot"></a>봇 테스트

Azure Portal에서 **Web Chat에서 테스트**를 클릭하여 봇을 테스트합니다. "Create a note"(메모 생성), "read my notes"(메모 읽기) 및 "delete notes"(메모 삭제)와 같은 메시지를 입력해봅니다.
   ![Web Chat에서 메모 봇 테스트](../media/bot-builder-dotnet-use-luis/bot-service-test-notebot.png)

> [!TIP]
> 봇이 항상 올바른 의도나 엔터티를 인식하지는 않는다는 것을 알게 되면 더 많은 예제 발화를 제공하여 학습시키는 방식으로 LUIS 앱 성능을 개선합니다. 봇의 코드를 수정하지 않고 LUIS 앱을 다시 학습시킬 수 있습니다. [예제 발화 추가](/azure/cognitive-services/LUIS/add-example-utterances) 및 [LUIS 앱 학습 및 테스트](/azure/cognitive-services/LUIS/train-test)를 참조하세요.

> [!TIP]
> 봇 코드에서 문제가 발생하면 다음을 확인합니다.
> * [봇이 빌드](./bot-builder-dotnet-luis-dialogs.md#build-the-bot)되었습니다.
> * 봇 코드에 LUIS 앱의 모든 의도에 대한 처리기가 정의되어 있습니다.

## <a name="next-steps"></a>다음 단계

봇을 사용해보면 LUIS 의도에 의해 작업이 호출되는 방식을 볼 수 있습니다. 하지만 이 간단한 예제로는 현재 활성인 대화를 중단시킬 수 없습니다. "도움"또는 "취소"와 같은 중단을 허용하고 처리하는 것이 사용자가 실제로 수행하는 일을 설명하는 유연한 디자인입니다. 대화에서 중단을 처리할 수 있도록 채점이 가능한 대화를 사용하는 방법에 대해 자세히 알아봅니다.

> [!div class="nextstepaction"]
> [Scorables(채점 가능)를 사용한 글로벌 메시지 처리기](bot-builder-dotnet-scorable-dialogs.md)

## <a name="additional-resources"></a>추가 리소스

- [다이얼로그](bot-builder-dotnet-dialogs.md)
- [Dialogs(대화)를 사용하여 대화 흐름 관리](bot-builder-dotnet-manage-conversation-flow.md)
- <a href="https://www.luis.ai" target="_blank">LUIS</a>
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Builder SDK 참조</a>

[LUIS]: https://www.luis.ai/
[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample
[NotesSampleJSON]: https://github.com/Microsoft/BotFramework-Samples/blob/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample/Notes.json
