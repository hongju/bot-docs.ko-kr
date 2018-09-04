---
title: QnA Maker를 사용하여 질문에 답변 | Microsoft Docs
description: 봇에서 QnA Maker를 사용하는 방법에 대해 알아봅니다.
keywords: 질문 및 답변, QnA, FAQ, 미들웨어
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7dd973e2b5a151e754925e6f19c6e4f82507f745
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906177"
---
# <a name="use-qna-maker-to-answer-questions"></a>QnA Maker를 사용하여 질문에 답변


봇을 지원하는 간단한 질문 및 답변을 추가하려면 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home) 서비스를 사용할 수 있습니다.


사용자 고유의 QnA Maker 서비스를 작성하는 기본적인 요구 사항 중 하나는 질문 및 답변을 통해 보내는 것입니다. 대부분의 경우 질문 및 답변은 이미 FAQ 또는 기타 설명서와 같은 콘텐츠에 있습니다. 다른 경우 더 자연스러운 대화식으로 질문에 대한 답변을 사용자 지정하려고 합니다. 

## <a name="create-a-qna-maker-service"></a>QnA Maker 서비스 만들기
먼저 계정을 만들고 [QnA Maker](https://qnamaker.ai/)에 로그인합니다. 그런 다음, **기술 자료 만들기**로 이동합니다. **QnA 서비스 만들기**를 클릭하고 Azure QnA 서비스를 만들기 위한 지침을 따릅니다.

![QnA 이미지 1](media/QnA_1.png)

[QnA Maker 만들기](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker)로 리디렉션됩니다. 양식을 완료하고 **만들기**를 클릭합니다.

![QnA 이미지 2](media/QnA_2.png)
 
Azure Portal에서 QnA 서비스를 만든 후 무시할 수도 있는 리소스 관리 제목 아래에 키가 제공됩니다. Azure QnA 서비스를 연결하려면 2단계로 진행합니다. 방금 만든 Azure 서비스를 선택하려면 페이지를 새로 고치고 기술 자료의 이름을 입력합니다.

![QnA 이미지 3](media/QnA_3.png)

**KB 만들기**를 클릭합니다. 사용자 고유의 질문 및 답변을 입력하거나 다음 예제를 복사합니다. 

![QnA 이미지 4](media/QnA_4.png)

또는 **KB 채우기**를 선택하고 파일 또는 URL을 제공할 수 있습니다. 간단한 QnA Maker 서비스를 생성하기 위한 샘플 원본 파일은 [여기](https://aka.ms/qna-tsv)에 있습니다.

새 QnA 쌍을 추가하거나 KB를 채운 뒤 **저장 후 학습**을 클릭합니다. 완료되면 **게시** 탭에서 **게시**를 클릭합니다.

QnA 서비스를 봇에 연결하려면 기술 자료 ID 및 QnA Maker 구독 키가 들어 있는 HTTP 요청 문자열이 필요합니다. 게시 결과에서 HTTP 요청 예제를 복사합니다. 

![QnA 이미지 5](media/QnA_5.png)

## <a name="installing-packages"></a>패키지 설치

코딩을 가져오기 전에 QnA Maker에 필요한 패키지가 있는지 확인합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

다음과 같은 NuGet 패키지의 시험판 버전 v4에 [참조를 추가](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)합니다.

* `Microsoft.Bot.Builder.Ai.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

이러한 서비스 중 하나를 botbuilder-ai 패키지를 사용하여 봇에 추가할 수 있습니다. npm을 통해 프로젝트에 이 패키지를 추가할 수 있습니다.

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


## <a name="using-qna-maker"></a>QnA Maker 사용

QnA Maker는 먼저 미들웨어로 추가됩니다. 그런 다음, 봇 논리 내에서 결과를 사용할 수 있습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

`Startup.cs` 파일에서 `ConfigureServices` 메서드를 업데이트하여 `QnAMakerMiddleware` 개체를 추가합니다. 간단히 봇의 미들웨어 스택에 추가하여 사용자에게서 받은 모든 메시지에 대한 기술 자료를 확인하기 위해 봇을 구성할 수 있습니다.


```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai.Qna;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;

public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var endpoint = new QnAMakerEndpoint
        {
           KnowledgeBaseId = "YOUR-KB-ID",
           // Get the Host from the HTTP request example at https://www.qnamaker.ai
           // For GA services: https://<Service-Name>.azurewebsites.net/qnamaker
           // For Preview services: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0           
           Host = "YOUR-HTTP-REQUEST-HOST",
           EndpointKey = "YOUR-QNA-MAKER-SUBSCRIPTION-KEY"
        };
        options.Middleware.Add(new QnAMakerMiddleware(endpoint));
    });
}
```



QnA Maker 미들웨어가 사용자의 질문에 응답을 보내지 않은 경우 `OnTurn`이 대체(fallback) 메시지를 보내도록 EchoBot.cs 파일에서 코드를 편집합니다.

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

namespace Bot_Builder_Echo_Bot_QnA
{
    public class EchoBot : IBot
    {    
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {             
                if (!context.Responded)
                {
                    // QnA didn't send the user an answer
                    await context.SendActivity("Sorry, I couldn't find a good match in the KB.");

                }
            }
        }
    }
}
```


샘플 봇은 [QnA Maker 샘플](https://aka.ms/qna-cs-bot-sample)을 참조합니다.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

먼저 [QnAMaker](https://github.com/Microsoft/botbuilder-js/tree/master/doc/botbuilder-ai/classes/botbuilder_ai.qnamaker.md) 클래스에서 필요합니다/가져옵니다.

```js
const { QnAMaker } = require('botbuilder-ai');
```

QnA 서비스에 대한 HTTP 요청에 따른 문자열을 사용하여 초기화함으로써 `QnAMaker`를 만듭니다. 설정 > 배포 세부 정보 아래의 [QnA Maker 포털](https://qnamaker.ai)에서 서비스에 대한 예제 요청을 복사할 수 있습니다.


문자열의 형식은 QnA Maker 서비스가 GA 또는 QnA Maker의 미리 보기 버전을 사용하는지 여부에 따라 다릅니다. HTTP 요청 예제를 복사하고 `QnAMaker`를 초기화할 때 사용할 기술 자료 ID, 구독 키 및 호스트를 가져옵니다.

<!--
**Preview**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    "Host: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Ocp-Apim-Subscription-Key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```

**GA**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    // Replace <Service-Name> to match the Azure URL where your service is hosted
    "Host: https://<Service-Name>.azurewebsites.net/qnamaker\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Authorization: EndpointKey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```
-->
**미리 보기**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://westus.api.cognitive.microsoft.com/qnamaker/v2.0'
    },
    {
        // set this to true to send answers from QnA Maker
        answerBeforeNext: true
    }
);
```
**GA**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://<Service-Name>.azurewebsites.net/qnamaker'
    },
    {
        answerBeforeNext: true
    }
);
```
간단히 봇의 미들웨어 스택에 추가하여 QNA Maker를 자동으로 호출하도록 봇을 구성할 수 있습니다.

```js
// Add QnA Maker as middleware
adapter.use(qna);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        // If `!context.responded`, that means an answer wasn't found for the user's utterance.
        // In this case, we send the user a fallback message.
        if (context.activity.type === 'message' && !context.responded) {
            await context.sendActivity('No QnA Maker answers were found.');
        } else if (context.activity.type !== 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

`QnAMaker`를 초기화할 경우 `answerBeforeNext`를 `true`로 초기화하는 것은 QnA Maker 미들웨어가 `processActivity`에서 봇의 기본 논리에 도달하기 전에 답변을 찾을 경우 자동으로 응답한다는 의미입니다. 대신 `answerBeforeNext`를 `false`로 설정하는 경우 `processActivity`에서 모든 봇의 기본 논리가 실행된 후 봇이 사용자에게 응답하지 않은 경우에만 봇은 QnA Maker를 호출하게 됩니다. 

## <a name="calling-qna-maker-without-using-middleware"></a>미들웨어를 사용하지 않고 QnA Maker 호출

이전 예제의 `adapter.use(qna);` 문은 QnA가 미들웨어로 실행되므로 봇이 수신하는 모든 메시지에 응답한다는 의미입니다. QnA Maker를 호출하는 방법 및 시기를 더 많이 제어하려면 미들웨어의 일부로 설치하는 대신 봇의 논리 내에서 `qna.answer()`를 직접 호출할 수 있습니다.

`adapter.use(qna);` 문을 제거하고 다음 코드를 사용하여 QnA Maker를 직접 호출합니다.

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            var handled = await qna.answer(context)
                if (!handled) {
                    await context.sendActivity(`I'm sorry. I didn't understand.`);
                }
        }
    });
});
```

QnA Maker를 사용자 지정하는 다른 방법은 `qna.generateAnswer()`를 사용하는 것입니다. 이 메서드를 사용하면 QnA Maker에서 대답에 대한 자세한 내용을 가져올 수 있습니다.


```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Get all the answers QnA Maker finds
            var results = await qna.generateAnswer(context.activity.text);
                if (results && results.length > 0) {
                    await context.sendActivity(results[0].answer);
                } else {
                    await context.sendActivity(`I don't know.`);
                }
    
        }
    });
});
```
---

QnA Maker 서비스에서 응답을 확인하려면 봇에게 질문합니다.

![QnA 이미지 6](media/QnA_6.png)



## <a name="next-steps"></a>다음 단계

QnA Maker는 기타 Cognitive Services와 결합하여 봇을 훨씬 더 강력하게 만들 할 수 있습니다. 디스패치 도구는 봇에서 LUIS(Language Understanding)와 QnA를 결합하는 방법을 제공합니다.

> [!div class="nextstepaction"]
> [디스패치 도구를 사용하여 LUIS 앱 및 QnA 서비스 결합](./bot-builder-tutorial-dispatch.md)
