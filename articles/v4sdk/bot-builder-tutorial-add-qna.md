---
title: Azure Bot Service 자습서 질문에 대답하는 봇 만들기 | Microsoft Docs
description: 봇에서 QnA Maker를 사용하여 질문에 대답하는 자습서입니다.
keywords: QnA Maker, 질문과 대답, 기술 자료
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: tutorial
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: aff1440f739150181ddc2d65d9b749b4eeda5d79
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464690"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>자습서: 봇에서 QnA Maker를 사용하여 질문에 대답

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

QnA Maker 서비스 및 기술 자료를 사용하여 봇에 질문과 대답 지원을 추가할 수 있습니다. 기술 자료를 만드는 것은 질문과 대답의 기반을 제공하는 것입니다.

이 자습서에서는 다음 방법에 대해 알아봅니다.

> [!div class="checklist"]
> * QnA Maker 서비스 및 기술 자료 만들기
> * 구성 파일에 기술 자료 정보 추가
> * 기술 자료를 쿼리하도록 봇 업데이트
> * 앱 다시 게시

Azure 구독이 아직 없는 경우 시작하기 전에 [무료 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.

## <a name="prerequisites"></a>필수 조건

* [이전 자습서](bot-builder-tutorial-basic-deploy.md)에서 만든 봇. 봇에 질문과 대답 기능을 추가할 것입니다.
* [QnA Maker](https://qnamaker.ai/)에 익숙하면 도움이 됩니다. QnA Maker 포털을 사용하여 봇에 사용할 기술 자료를 만들고, 교육하고, 게시할 것입니다.
* Azure Bot Service를 사용한 [QnA 봇 만들기](https://aka.ms/azure-create-qna) 친숙도.

또한 이전 자습서의 다음 필수 구성 요소를 이미 완료했어야 합니다.

## <a name="sign-in-to-qna-maker-portal"></a>QnA Maker 포털에 로그인

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

Azure 자격 증명을 사용하여 [QnA Maker 포털](https://qnamaker.ai/)에 로그인합니다.

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>QnA Maker 서비스 및 기술 자료 만들기

[Microsoft/BotBuilder 샘플](https://github.com/Microsoft/BotBuilder-Samples) 리포지토리의 QnA Maker 샘플에서 기존 기술 자료 정의를 가져올 것입니다.

1. 샘플 리포지토리를 컴퓨터에 복제 또는 복사합니다.
1. QnA Maker 포털에서 **기술 자료를 만듭니다**.
   1. 필요한 경우 QnA 서비스를 만듭니다. (기존 QnA Maker 서비스를 사용해도 되고 이 자습서에서 사용할 새 서비스를 만들어도 됩니다.) 보다 자세한 QnA Maker 지침은 [QnA Maker 서비스 만들기](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) 및 [QnA Maker 기술 자료 만들기, 학습 및 게시](https://docs.microsoft.com/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)를 참조하세요.
   1. 기술 자료에 QnA 서비스를 연결합니다.
   1. 기술 자료의 이름을 지정합니다.
   1. 기술 자료를 채우려면 샘플 리포지토리의 **BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv** 파일을 사용합니다.
   1. **kb 만들기**를 클릭하여 기술 자료를 만듭니다.
1. 기술 자료를 **저장 및 학습**합니다.
1. 기술 자료를 **게시**합니다.

QnA Maker 앱이 게시되면 _설정_ 탭을 선택하고 '배포 세부 정보'가 나올 때까지 아래로 스크롤합니다. _Postman_ 샘플 HTTP 요청의 다음 값을 기록합니다.

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL ending in /qnamaker.
Authorization: EndpointKey <qna-maker-resource-key>
```

호스트 이름에 대한 전체 URL 문자열은 "https://< >.azure.net/qnamaker"와 같습니다.

이러한 값은 다음 단계에서 `appsettings.json` 또는 `.env` 파일 내에서 사용됩니다.

봇에서 사용할 기술 자료가 준비되었습니다.

## <a name="add-knowledge-base-information-to-your-bot"></a>봇에 기술 자료 정보 추가
Bot Framework v4.3부터 Azure는 다운로드한 봇 소스 코드의 일부로 .bot 파일을 더 이상 제공하지 않습니다. 다음 지침을 사용하여 CSharp 또는 JavaScript 봇을 기술 자료에 연결할 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

다음 값을 appseting.json 파일에 추가합니다.

```json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "ScmType": "None",
  
  "QnAKnowledgebaseId": "knowledge-base-id",
  "QnAAuthKey": "qna-maker-resource-key",
  "QnAEndpointHostName": "your-hostname" // This is a URL ending in /qnamaker
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

다음 값을 .env 파일에 추가합니다.

```
MicrosoftAppId=""
MicrosoftAppPassword=""
ScmType=None

QnAKnowledgebaseId="knowledge-base-id"
QnAAuthKey="qna-maker-resource-key"
QnAEndpointHostName="your-hostname" // This is a URL ending in /qnamaker
```

---

| 필드 | 값 |
|:----|:----|
| QnAKnowledgebaseId | QnA Maker 포털에서 자동으로 생성한 기술 자료 ID입니다. |
| QnAAuthKey | QnA Maker 포털에서 자동으로 생성한 엔드포인트 키입니다. |
| QnAEndpointHostName | QnA Maker 포털에서 생성한 호스트 URL입니다. `https://`로 시작하고 `/qnamaker`로 끝나는 완전한 URL을 사용하세요. 전체 URL 문자열은 "https://< >.azure.net/qnamaker"와 유사할 것입니다. |

이제 편집 내용을 저장합니다.

## <a name="update-your-bot-to-query-the-knowledge-base"></a>기술 자료를 쿼리하도록 봇 업데이트

기술 자료에 대한 서비스 정보를 로드하도록 초기화 코드를 업데이트합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. **Microsoft.Bot.Builder.AI.QnA** NuGet 패키지를 프로젝트에 추가합니다.

   NuGet 패키지 관리자 또는 명령줄을 통해 이 작업을 수행할 수 있습니다.

   ```cmd
   dotnet add package Microsoft.Bot.Builder.AI.QnA
   ```

   NuGet에 대한 자세한 내용은 [NuGet 문서](https://docs.microsoft.com/nuget/#pivot=start&panel=start-all)를 참조하세요.

1. **Microsoft.Extensions.Configuration** NuGet 패키지를 프로젝트에 추가합니다.

1. **Startup.cs** 파일에서 다음 네임스페이스 참조를 추가합니다.

   **Startup.cs**

   ```csharp
   using Microsoft.Bot.Builder.AI.QnA;
   using Microsoft.Extensions.Configuration;
   ```

1. 그리고 _ConfigureServices_ 메서드를 수정하여 **appsettings.json** 파일에 정의된 기술 자료에 연결하는 QnAMakerEndpoint를 만듭니다.

   **Startup.cs**

   ```csharp
   // Create QnAMaker endpoint as a singleton
   services.AddSingleton(new QnAMakerEndpoint
   {
      KnowledgeBaseId = Configuration.GetValue<string>($"QnAKnowledgebaseId"),
      EndpointKey = Configuration.GetValue<string>($"QnAAuthKey"),
      Host = Configuration.GetValue<string>($"QnAEndpointHostName")
    });

   ```

1. **EchoBot.cs** 파일에서 다음 네임스페이스 참조를 추가합니다.

   **EchoBot.cs**

   ```csharp
   using System.Linq;
   using Microsoft.Bot.Builder.AI.QnA;
   ```

1. `EchoBotQnA` 커넥터를 추가하고 봇의 생성자에서 초기화합니다.

   **EchoBot.cs**

   ```csharp
   public QnAMaker EchoBotQnA { get; private set; }
   public EchoBot(QnAMakerEndpoint endpoint)
   {
      // connects to QnA Maker endpoint for each turn
      EchoBotQnA = new QnAMaker(endpoint);
   }
   ```

1. _OnMembersAddedAsync( )_ 메서드 아래에서 다음 코드를 추가하여 _AccessQnAMaker( )_ 메서드를 만듭니다.

   **EchoBot.cs**

   ```csharp
   private async Task AccessQnAMaker(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      var results = await EchoBotQnA.GetAnswersAsync(turnContext);
      if (results.Any())
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("QnA Maker Returned: " + results.First().Answer), cancellationToken);
      }
      else
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("Sorry, could not find an answer in the Q and A system."), cancellationToken);
      }
   }
   ```

1. 이제 _OnMessageActivityAsync( )_ 내에서 새 _AccessQnAMaker( )_ 메서드를 다음과 같이 호출합니다.

   **EchoBot.cs**

   ```csharp
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      // First send the user input to your QnA Maker knowledge base
      await AccessQnAMaker(turnContext, cancellationToken);
      ...
   }
   ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 터미널 또는 명령 프롬프트를 프로젝트의 루트 디렉터리로 엽니다.
1. 프로젝트에 **botbuilder-ai** npm 패키지를 추가합니다.
   ```shell
   npm i botbuilder-ai
   ```

1. **index.js**에서 // Create Adapter(어댑터 만들기) 섹션 뒤에 다음 코드를 추가하여 QnA Maker 서비스를 생성하는 데 필요한 .env 파일 구성 정보를 읽습니다.

   **index.js**
   ```javascript
   // Map knowledge base endpoint values from .env file into the required format for `QnAMaker`.
   const configuration = {
      knowledgeBaseId: process.env.QnAKnowledgebaseId,
      endpointKey: process.env.QnAAuthKey,
      host: process.env.QnAEndpointHostName
   };

   ```

1. QnA 서비스 구성 정보를 전달하도록 봇 구문을 업데이트합니다.

   **index.js**
   ```javascript
   // Create the main dialog.
   const myBot = new MyBot(configuration, {});
   ```

1. **bot.js** 파일에서 QnAMaker에 대한 다음 require를 추가합니다.

   **bot.js**
   ```javascript
   const { QnAMaker } = require('botbuilder-ai');
   ```

1. 이제 QnAMaker 커넥터를 만드는 데 필요한 전달된 구성 매개 변수를 받도록 생성자를 수정하고, 이러한 매개 변수가 제공되지 않으면 오류를 throw합니다.

   **bot.js**
   ```javascript
      class MyBot extends ActivityHandler {
         constructor(configuration, qnaOptions) {
            super();
            if (!configuration) throw new Error('[QnaMakerBot]: Missing parameter. configuration is required');
            // now create a qnaMaker connector.
            this.qnaMaker = new QnAMaker(configuration, qnaOptions);
   ```

1. 마지막으로, 기술 자료를 쿼리하여 답변을 찾도록 `onMessage` 함수를 업데이트합니다. 각 사용자 입력을 QnA Maker 기술 자료에 전달하고, 첫 번째 QnA Maker 응답을 사용자에게 다시 반환합니다.

    **bot.js**

    ```javascript
    this.onMessage(async (context, next) => {
        // send user input to QnA Maker.
        const qnaResults = await this.qnaMaker.getAnswers(context);

        // If an answer was received from QnA Maker, send the answer back to the user.
        if (qnaResults[0]) {
            await context.sendActivity(`QnAMaker returned response: ' ${ qnaResults[0].answer}`);
        }
        else {
            // If no answers were returned from QnA Maker, reply with help.
            await context.sendActivity('No QnA Maker response was returned.'
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        }
        await next();
    });
    ```

---

### <a name="test-the-bot-locally"></a>로컬로 봇 테스트

이제 봇이 일부 질문에 답할 수 있을 것입니다. 봇을 로컬로 실행하고 에뮬레이터에서 봇을 엽니다.

![qna 샘플 테스트](./media/qna-test-bot.png)

## <a name="republish-your-bot"></a>앱 다시 게시

이제 봇을 Azure에 다시 게시할 수 있습니다.

> [!IMPORTANT]
> 프로젝트 파일의 zip을 만들기 전에 올바른 폴더 _안_ 에 있는지 확인합니다. 
> - C# 봇의 경우 .csproj 파일이 있는 폴더입니다. 
> - JS 봇의 경우 app.js 또는 index.js 파일이 있는 폴더입니다. 
>
> 해당 폴더에 있는 동안 모든 파일을 선택하고 압축한 다음, 여전히 해당 폴더에 있는 동안 명령을 실행합니다.
>
> 루트 폴더 위치가 올바르지 않을 경우 **봇이 Azure Portal에서 실행되지 못하게 됩니다**.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
```cmd
az webapp deployment source config-zip --resource-group "resource-group-name" --name "bot-name-in-azure" --src "c:\bot\mybot.zip"
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish-js.md)]

---

### <a name="test-the-published-bot"></a>게시된 봇 테스트

봇을 게시한 후 Azure가 봇을 업데이트하고 시작할 때까지 1~2분 정도 기다립니다.

에뮬레이터를 사용하여 봇의 프로덕션 엔드포인트를 테스트하거나, Azure Portal을 사용하여 웹 채팅에서 봇을 테스트합니다.
어떤 방법을 사용하든, 로컬로 테스트할 때와 동일한 동작이 발생합니다.

## <a name="clean-up-resources"></a>리소스 정리

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

이 애플리케이션을 계속 사용할 계획이 없으면 다음 단계에 따라 관련 리소스를 삭제합니다.

1. Azure Portal에서 봇의 리소스 그룹을 엽니다.
1. **리소스 그룹 삭제**를 클릭하여 그룹 및 그룹에 포함된 모든 리소스를 삭제합니다.
1. 확인 창에 리소스 그룹 이름을 입력하고 **삭제**를 클릭합니다.

## <a name="next-steps"></a>다음 단계

봇에 기능을 추가하는 방법에 대한 자세한 내용은 **문자 메시지 보내기 및 받기**와 다른 개발 방법 섹션 문서를 참조하세요.
> [!div class="nextstepaction"]
> [텍스트 메시지 보내기 및 받기](bot-builder-howto-send-messages.md)
