---
title: QnA Maker를 사용하여 질문에 답변 | Microsoft Docs
description: 봇에서 QnA Maker를 사용하는 방법에 대해 알아봅니다.
keywords: 질문 및 대답, QnA, FAQ, QnA Maker
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4a221f6e94324c56f88dd1d4d6851d5cc4d38e6c
ms.sourcegitcommit: 3cc768a8e676246d774a2b62fb9c688bbd677700
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/16/2019
ms.locfileid: "54323679"
---
# <a name="use-qna-maker-to-answer-questions"></a>QnA Maker를 사용하여 질문에 답변

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

QnA Maker 서비스를 사용하여 봇에 질문 및 답변 지원 기능을 추가할 수 있습니다. 사용자 고유의 QnA Maker 서비스를 만들 때 기본적인 요구 사항 중 하나는 질문 및 답변을 통해 보내는 것입니다. 대부분의 경우 질문 및 답변은 이미 FAQ 또는 기타 설명서와 같은 콘텐츠에 있습니다. 다른 경우 더 자연스러운 대화식으로 질문에 대한 답변을 사용자 지정하려고 합니다.

이 항목에서는 기술 자료를 만들고 봇에서 이를 사용합니다.

## <a name="prerequisites"></a>필수 조건
- [QnA Maker](https://www.qnamaker.ai/) 계정
- 이 문서의 코드는 **QnA Maker** 샘플을 기반으로 합니다. [C#](https://aka.ms/cs-qna) 또는 [JS](https://aka.ms/js-qna-sample)로 작성된 샘플의 복사본이 필요합니다.
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- [봇 기본 사항](bot-builder-basics.md), [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview) 및 [.bot](bot-file-basics.md) 파일에 대한 지식이 필요합니다.

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>QnA Maker 서비스를 만들고 기술 자료 게시
1. 먼저 [QnA Maker 서비스](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)를 만들어야 합니다.
1. 다음으로, 프로젝트의 CognitiveModels 폴더에 있는 `smartLightFAQ.tsv` 파일을 사용하여 기술 자료를 만듭니다. QnA Maker [기술 자료](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)를 생성, 학습 및 게시하는 단계는 QnA Maker 설명서에 나와 있습니다. 다음 단계에 따라 `qna` KB의 이름을 지정하고, `smartLightFAQ.tsv` 파일을 사용하여 KB를 채웁니다.

## <a name="obtain-values-to-connect-to-your-connect-your-bot-to-the-knowledge-base"></a>봇을 기술 자료에 연결하는 데 필요한 값 가져오기
1. [QnA Maker](https://www.qnamaker.ai/) 사이트에서 기술 자료를 선택합니다.
1. 기술 자료를 연 후 **설정**을 선택합니다. _서비스 이름_에 표시되는 <your_kb_name>이라는 값을 기록합니다.
1. 아래로 스크롤하여 **배포 세부 정보**를 찾아 다음 값을 기록합니다.
   - POST /knowledgebases/<your_knowledge_base_id>/generateAnswer
   - 호스트: <your_hostname>/qnamaker
   - 권한 부여: EndpointKey <your_endpoint_key>

## <a name="update-the-bot-file"></a>.bot 파일 업데이트
먼저, 호스트 이름, 엔드포인트 키 및 기술 자료 ID(KbId) 등 기술 자료에 액세스하는 데 필요한 정보를 `qnamaker.bot`에 추가합니다. 이는 QnA Maker의 기술 자료 **설정**에서 저장한 값입니다.

```json
{
  "name": "qnamaker",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": ""
      "id": "25",
    
    },
    {
      "type": "qna",
      "name": "QnABot",
      "KbId": "<YOUR_KNOWLEDGE_BASE_ID>",
      "subscriptionKey": "<Your_Azure_Subscription_Key>", // Used when creating your QnA service.
      "endpointKey": "<Your_Recorded_Endpoint_Key>",
      "hostname": "<Your_Recorded_Hostname>",
      "id": "117"
    }
  ],
  "padlock": "",
   "version": "2.0"
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)
다음으로, .bot 파일에서 위의 정보를 가져오는 BotServices.cs에 있는 BotService 클래스의 새 인스턴스를 초기화합니다. 외부 서비스는 BotConfiguration 클래스를 사용하여 구성됩니다.

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.QnA:
            {
                // Create a QnA Maker that is initialized and suitable for passing
                // into the IBot-derived class (QnABot).
                var qna = (QnAMakerService)service;
                if (qna == null)
                {
                    throw new InvalidOperationException("The QnA service is not configured correctly in your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.KbId))
                {
                    throw new InvalidOperationException("The QnA KnowledgeBaseId ('kbId') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.EndpointKey))
                {
                    throw new InvalidOperationException("The QnA EndpointKey ('endpointKey') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.Hostname))
                {
                    throw new InvalidOperationException("The QnA Host ('hostname') is required to run this sample. Please update your '.bot' file.");
                }

                var qnaEndpoint = new QnAMakerEndpoint()
                {
                    KnowledgeBaseId = qna.KbId,
                    EndpointKey = qna.EndpointKey,
                    Host = qna.Hostname,
                };

                var qnaMaker = new QnAMaker(qnaEndpoint);
                qnaServices.Add(qna.Name, qnaMaker);
                break;
            }
        }
    }
    var connectedServices = new BotServices(qnaServices);
    return connectedServices;
}
```

그런 다음, QnABot.cs에서 이 QnAMaker 인스턴스를 봇에 제공합니다. 자체 기술 자료에 액세스하는 경우 아래에 표시된 _welcome_ 메시지를 변경하여 사용자에게 유용한 초기 지침을 제공하세요.

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a question to get started.";
    private readonly BotServices _services;
    public QnABot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        Console.WriteLine($"{_services}");
        if (!_services.QnAServices.ContainsKey(QnAMakerKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a QnA service named '{QnAMakerKey}'.");
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

이 샘플에서 시작 코드는 **index.js** 파일에 있으며, 봇 논리의 코드는 **bot.js** 파일에 있고, 추가 구성 정보는 **qnamaker.bot** 파일에 있습니다.

**index.js** 파일에서 구성 정보를 읽어들여 QnA Maker 서비스를 생성하고 봇을 초기화합니다.

`QNA_CONFIGURATION` 값을 구성 파일에 표시되는 기술 자료의 이름으로 업데이트합니다.

```js
// QnA Maker knowledge base name as specified in .bot file.
const QNA_CONFIGURATION = '<YOUR_KB_NAME>';

// Get endpoint and QnA Maker configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const qnaConfig = botConfig.findServiceByNameOrId(QNA_CONFIGURATION);

// Map the contents to the required format for `QnAMaker`.
const qnaEndpointSettings = {
    knowledgeBaseId: qnaConfig.kbId,
    endpointKey: qnaConfig.endpointKey,
    host: qnaConfig.hostname
};

// Create adapter...

// Create the QnAMakerBot.
let bot;
try {
    bot = new QnAMakerBot(qnaEndpointSettings, {});
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

그런 다음, HTTP 서버를 만들고 수신 요청을 대시하면 봇 논리에 대한 호출이 생성됩니다.

```js
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open qnamaker.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

## <a name="calling-qna-maker-from-your-bot"></a>봇에서 QnA Maker 호출

# <a name="ctabcs"></a>[C#](#tab/cs)

봇에서 QnAMaker의 응답을 요구하는 경우 봇 코드에서 `GetAnswersAsync()`를 호출하여 현재 컨텍스트에 맞는 응답을 가져옵니다. 자체 기술 자료에 액세스하는 경우 아래의 _no answers_ 메시지를 변경하여 사용자에게 유용한 지침을 제공하세요.

```csharp
// Check QnA Maker model
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext);
if (response != null && response.Length > 0)
{
    await turnContext.SendActivityAsync(response[0].Answer, cancellationToken: cancellationToken);
}
else
{
    var msg = @"No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs.
                To see QnA Maker in action, ask the bot questions like 'Why won't it turn on?' or 'I need help'.";
    await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
}

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**bot.js** 파일에서 QnA Maker 서비스의 `generateAnswer` 메서드에 사용자 입력을 전달하여 기술 자료에서 응답을 가져옵니다. 자체 기술 자료에 액세스하는 경우 아래의 _no answers_ 및 _welcome_ 메시지를 변경하여 사용자에게 유용한 지침을 제공하세요.

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');
const { QnAMaker, QnAMakerEndpoint, QnAMakerOptions } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from QnA Maker.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class QnAMakerBot {
    /**
     * The QnAMakerBot constructor requires one argument (`endpoint`) which is used to create an instance of `QnAMaker`.
     * @param {QnAMakerEndpoint} endpoint The basic configuration needed to call QnA Maker. In this sample the configuration is retrieved from the .bot file.
     * @param {QnAMakerOptions} config An optional parameter that contains additional settings for configuring a `QnAMaker` when calling the service.
     */
    constructor(endpoint, qnaOptions) {
        this.qnaMaker = new QnAMaker(endpoint, qnaOptions);
    }

    /**
     * Every conversation turn for our QnAMakerBot will call this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls QnA Maker in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
            const qnaResults = await this.qnaMaker.generateAnswer(turnContext.activity.text);

            // If an answer was received from QnA Maker, send the answer back to the user.
            if (qnaResults[0]) {
                await turnContext.sendActivity(qnaResults[0].answer);

            // If no answers were returned from QnA Maker, reply with help.
            } else {
                await turnContext.sendActivity('No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. To see QnA Maker in action, ask the bot questions like "Why won\'t it turn on?" or "I need help."');
            }

        // If the Activity is a ConversationUpdate, send a greeting message to the user.
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
                   turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            await turnContext.sendActivity('Welcome to the QnA Maker sample! Ask me a question and I will try to answer it.');

        // Respond to all other Activity types.
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.QnAMakerBot = QnAMakerBot;
```

---

## <a name="test-the-bot"></a>봇 테스트

샘플을 머신에서 로컬로 실행합니다. 지침이 필요한 경우 [C#](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/11.qnamaker) 또는 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/11.qnamaker/README.md) 샘플에 대한 추가 정보 파일을 참조하세요.

에뮬레이터에서 아래와 같이 봇에 메시지를 보냅니다.

![qna 샘플 테스트](~/media/emulator-v4/qna-test-bot.png)


## <a name="next-steps"></a>다음 단계

QnA Maker는 기타 Cognitive Services와 결합하여 봇을 훨씬 더 강력하게 만들 할 수 있습니다. 디스패치 도구는 봇에서 LUIS(Language Understanding)와 QnA를 결합하는 방법을 제공합니다.

> [!div class="nextstepaction"]
> [디스패치 도구를 사용하여 LUIS 및 QnA 서비스 결합](./bot-builder-tutorial-dispatch.md)
