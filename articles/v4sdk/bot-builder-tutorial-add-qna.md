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
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b1c34531ee60b2ce9037f42e4f5a7093501cf83a
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360954"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>자습서: 봇에서 QnA Maker를 사용하여 질문에 대답

QnA Maker 서비스 및 기술 자료를 사용하여 봇에 질문과 대답 지원을 추가할 수 있습니다. 기술 자료를 만드는 것은 질문과 대답의 기반을 제공하는 것입니다.

이 자습서에서는 다음 방법에 대해 알아봅니다.

> [!div class="checklist"]
> * QnA Maker 서비스 및 기술 자료 만들기
> * .bot 파일에 기술 자료 정보 추가
> * 기술 자료를 쿼리하도록 봇 업데이트
> * 봇 다시 게시

Azure 구독이 아직 없는 경우 시작하기 전에 [무료 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 을 만듭니다.

## <a name="prerequisites"></a>필수 조건

* [이전 자습서](bot-builder-tutorial-basic-deploy.md)에서 만든 봇. 봇에 질문과 대답 기능을 추가할 것입니다.
* QnA Maker에 익숙하면 도움이 됩니다. QnA Maker 포털을 사용하여 봇에 사용할 기술 자료를 만들고, 교육하고, 게시할 것입니다.

여러분은 다음과 같은 이전 자습서의 필수 구성 요소를 이미 갖추고 있어야 합니다.

[!INCLUDE [deployment prerequisites snippet](~/includes/deploy/snippet-prerequisite.md)]

## <a name="sign-in-to-qna-maker-portal"></a>QnA Maker 포털에 로그인

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

Azure 자격 증명을 사용하여 [QnA Maker 포털](https://qnamaker.ai/)에 로그인합니다.

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>QnA Maker 서비스 및 기술 자료 만들기

[Microsoft/BotBuilder 샘플](https://github.com/Microsoft/BotBuilder-Samples) 리포지토리의 QnA Maker 샘플에서 기존 기술 자료 정의를 가져올 것입니다.

1. 샘플 리포지토리를 컴퓨터에 복제 또는 복사합니다.
1. QnA Maker 포털에서 **기술 자료를 만듭니다**.
   1. 필요한 경우 QnA 서비스를 만듭니다. (기존 QnA Maker 서비스를 사용해도 되고 이 자습서에서 사용할 새 서비스를 만들어도 됩니다.) 보다 자세한 QnA Maker 지침은 [QnA Maker 서비스 만들기](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) 및 [QnA Maker 기술 자료 만들기, 학습 및 게시](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)를 참조하세요.
   1. 기술 자료에 QnA 서비스를 연결합니다.
   1. 기술 자료의 이름을 지정합니다.
   1. 기술 자료를 채우려면 샘플 리포지토리의 **BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv** 파일을 사용합니다.
   1. **kb 만들기**를 클릭하여 기술 자료를 만듭니다.
1. 기술 자료를 **저장 및 학습**합니다.
1. 기술 자료를 **게시**합니다.

   봇에서 사용할 기술 자료가 준비되었습니다. 기술 자료 ID, 엔드포인트 키 및 호스트 이름을 기록해 둡니다. 다음 단계에서 필요합니다.

## <a name="add-knowledge-base-information-to-your-bot-file"></a>.bot 파일에 기술 자료 정보 추가

기술 자료에 액세스하는 데 필요한 정보를 .bot 파일에 추가합니다.

1. 편집기에서 .bot 파일을 엽니다.
1. `qna` 요소를 `services` 배열에 추가합니다.

    ```json
    {
        "type": "qna",
        "name": "<your-knowledge-base-name>",
        "kbId": "<your-knowledge-base-id>",
        "hostname": "<your-qna-service-hostname>",
        "endpointKey": "<your-knowledge-base-endpoint-key>",
        "subscriptionKey": "<your-azure-subscription-key>",
        "id": "<a-unique-id>"
    }
    ```

    | 필드 | 값 |
    |:----|:----|
    | 형식 | `qna`이어야 합니다. 이 서비스 항목이 QnA 기술 자료를 설명한다는 것을 나타냅니다. |
    | 이름 | 기술 자료에 할당할 이름입니다. |
    | kbId | QnA Maker 포털에서 자동으로 생성한 기술 자료 ID입니다. |
    | hostname | QnA Maker 포털에서 생성한 호스트 URL입니다. `https://`로 시작하고 `/qnamaker`로 끝나는 완전한 URL을 사용하세요. |
    | endpointKey | QnA Maker 포털에서 자동으로 생성한 엔드포인트 키입니다. |
    | subscriptionKey | Azure에서 QnA Maker 서비스를 만들 때 사용한 구독 ID입니다. |
    | id | "201"처럼 .bot 파일에 나열된 다른 서비스 중 하나에 아직 사용되지 않은 고유 ID입니다. |

1. 편집 내용을 저장합니다.

## <a name="update-your-bot-to-query-the-knowledge-base"></a>기술 자료를 쿼리하도록 봇 업데이트

기술 자료에 대한 서비스 정보를 로드하도록 초기화 코드를 업데이트합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. **Microsoft.Bot.Builder.AI.QnA** NuGet 패키지를 프로젝트에 추가합니다.
1. **IBot**을 구현하는 클래스 이름을 `QnaBot`으로 바꿉니다.
1. 봇의 접근자를 포함하는 클래스 이름을 `QnaBotAccessors`로 바꿉니다.
1. **Startup.cs** 파일에서 다음 네임스페이스 참조를 추가합니다.
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Bot.Builder.AI.QnA;
    using Microsoft.Bot.Builder.Integration;
    ```
1. 그리고 **.bot** 파일에 정의된 기술 자료를 초기화하고 등록하도록 **ConfigureServices** 메서드를 수정합니다. 처음 몇 줄이 `services.AddBot<QnaBot>(options =>` 호출의 본문에서 그 앞으로 이동되었습니다.
    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        var secretKey = Configuration.GetSection("botFileSecret")?.Value;
        var botFilePath = Configuration.GetSection("botFilePath")?.Value;

        // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
        var botConfig = BotConfiguration.Load(botFilePath ?? @".\jfEchoBot.bot", secretKey);
        services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

        // Initialize the QnA knowledge bases for the bot.
        services.AddSingleton(sp => {
            var qnaServices = new List<QnAMaker>();
            foreach (var qnaService in botConfig.Services.OfType<QnAMakerService>())
            {
                qnaServices.Add(new QnAMaker(qnaService));
            }
            return qnaServices;
        });

        services.AddBot<QnaBot>(options =>
        {
            // Retrieve current endpoint.
            // ...
        });

        // Create and register state accessors.
        // ...
    }
    ```
1. **QnaBot.cs** 파일에서 다음 네임스페이스 참조를 추가합니다.
    ```csharp
    using System.Collections.Generic;
    using Microsoft.Bot.Builder.AI.QnA;
    ```
1. `_qnaServices` 속성을 추가하고 봇의 생성자에서 초기화합니다.
    ```csharp
    private readonly List<QnAMaker> _qnaServices;

    /// ...
    public QnaBot(QnaBotAccessors accessors, List<QnAMaker> qnaServices, ILoggerFactory loggerFactory)
    {
        // ...
        _qnaServices = qnaServices;
    }
    ```
1. 사용자의 입력에 대해 기술 자료를 쿼리하고 등록하도록 순서 처리기를 수정합니다. 봇에서 QnAMaker의 응답을 요구하는 경우 봇 코드에서 `GetAnswersAsync`를 호출하여 현재 컨텍스트에 맞는 응답을 가져옵니다. 자체 기술 자료에 액세스하는 경우 아래의 _no answers_ 메시지를 변경하여 사용자에게 유용한 지침을 제공하세요.
    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            foreach(var qnaService in _qnaServices)
            {
                var response = await qnaService.GetAnswersAsync(turnContext);
                if (response != null && response.Length > 0)
                {
                    await turnContext.SendActivityAsync(
                        response[0].Answer,
                        cancellationToken: cancellationToken);
                    return;
                }
            }

            var msg = "No QnA Maker answers were found. This example uses a QnA Maker knowledge base that " +
                "focuses on smart light bulbs. Ask the bot questions like 'Why won't it turn on?' or 'I need help'.";

            await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
        }
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 터미널 또는 명령 프롬프트를 프로젝트의 루트 디렉터리로 엽니다.
1. 프로젝트에 **botbuilder-ai** npm 패키지를 추가합니다.
    ```shell
    npm i botbuilder-ai
    ```
1. **index.js** 파일에서 이 require 문을 추가합니다.
    ```javascript
    const { QnAMaker } = require('botbuilder-ai');
    ```
1. 구성 정보를 읽고 QnA Maker 서비스를 생성합니다.
    ```javascript
    // Read bot configuration from .bot file.
    // ...

    // Initialize the QnA knowledge bases for the bot.
    // Assume each QnA entry in the .bot file is well defined.
    const qnaServices = [];
    botConfig.services.forEach(s => {
        if (s.type == 'qna') {
            const endpoint = {
                knowledgeBaseId: s.kbId,
                endpointKey: s.endpointKey,
                host: s.hostname
            };
            const options = {};
            qnaServices.push(new QnAMaker(endpoint, options));
        }
    });

    // Get bot endpoint configuration by service name
    // ...
    ```
1. QnA 서비스를 전달하도록 봇 구문을 업데이트 합니다.
    ```javascript
    // Create the bot.
    const myBot = new MyBot(qnaServices);
    ```
1. **bot.js** 파일에서 생성자를 추가합니다.
    ```javascript
    constructor(qnaServices) {
        this.qnaServices = qnaServices;
    }
    ```
1. 그리고 기술 자료를 쿼리하여 답변을 찾도록 순서 처리기를 업데이트합니다.
    ```javascript
    async onTurn(turnContext) {
        if (turnContext.activity.type === ActivityTypes.Message) {
            for (let i = 0; i < this.qnaServices.length; i++) {
                // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
                const qnaResults = await this.qnaServices[i].getAnswers(turnContext);

                // If an answer was received from QnA Maker, send the answer back to the user and exit.
                if (qnaResults[0]) {
                    await turnContext.sendActivity(qnaResults[0].answer);
                    return;
                }
            }
            // If no answers were returned from QnA Maker, reply with help.
            await turnContext.sendActivity('No QnA Maker answers were found. '
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        } else {
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }
    ```

---

### <a name="test-the-bot-locally"></a>로컬로 봇 테스트

이제 봇이 일부 질문에 답할 수 있을 것입니다. 봇을 로컬로 실행하고 에뮬레이터에서 봇을 엽니다.

![qna 샘플 테스트](~/media/emulator-v4/qna-test-bot.png)

## <a name="re-publish-your-bot"></a>봇 다시 게시

이제 봇을 다시 게시할 수 있습니다.

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

### <a name="test-the-published-bot"></a>게시된 봇 테스트

봇을 게시한 후 Azure가 봇을 업데이트하고 시작할 때까지 1~2분 정도 기다립니다.

1. 에뮬레이터를 사용하여 봇의 프로덕션 엔드포인트를 테스트하거나, Azure Portal을 사용하여 웹 채팅에서 봇을 테스트합니다.

   어떤 방법을 사용하든, 로컬로 테스트할 때와 동일한 동작이 발생합니다.

## <a name="clean-up-resources"></a>리소스 정리

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

이 애플리케이션을 계속 사용할 계획이 없으면 다음 단계에 따라 관련 리소스를 삭제합니다.

1. Azure Portal에서 봇의 리소스 그룹을 엽니다.
1. **리소스 그룹 삭제**를 클릭하여 그룹 및 그룹에 포함된 모든 리소스를 삭제합니다.
1. 확인 창에 리소스 그룹 이름을 입력하고 **삭제**를 클릭합니다.

## <a name="next-steps"></a>다음 단계

봇에 기능을 추가하는 방법에 대한 자세한 내용은 방법 개발 섹션의 문서를 참조하세요.
> [!div class="nextstepaction"]
> [다음 단계 단추](bot-builder-howto-send-messages.md)
