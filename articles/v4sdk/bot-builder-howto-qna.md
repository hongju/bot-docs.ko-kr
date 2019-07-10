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
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 15581daa570b9e51ff8f7bec93d16deebcd71d45
ms.sourcegitcommit: 93508adfb79523f610a919b361fc34f5c8dd3eff
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/02/2019
ms.locfileid: "67533387"
---
# <a name="use-qna-maker-to-answer-questions"></a>QnA Maker를 사용하여 질문에 답변

[!INCLUDE[applies-to](../includes/applies-to.md)]

QnA Maker는 사용자의 데이터에 대한 대화형 질문 및 답변 레이어를 제공합니다. 이 서비스를 사용하면 사용자가 질문을 구문 분석하고 해당 질문의 의도를 해석할 필요 없이 봇에서 QnA Maker에 질문을 보내고 답변을 받을 수 있습니다. 

사용자 고유의 QnA Maker 서비스를 만들 때 기본적인 요구 사항 중 하나는 질문 및 답변을 통해 보내는 것입니다. 대부분의 경우 질문과 답변은 FAQ 또는 기타 설명서와 같은 컨텍스트에 이미 존재하지만, 질문에 대한 답변을 더 자연스러운 대화형 방법으로 사용자 지정하려는 경우도 있습니다. 

## <a name="prerequisites"></a>필수 조건

- 이 문서의 코드는 QnA Maker 샘플을 기반으로 합니다. **[CSharp](https://aka.ms/cs-qna) 또는 [JavaScript](https://aka.ms/js-qna-sample)** 로 작성된 샘플의 복사본이 필요합니다.
- [QnA Maker](https://www.qnamaker.ai/) 계정
- [봇 기본 사항](bot-builder-basics.md), [QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/overview/overview) 및 [봇 리소스 관리](bot-file-basics.md)에 대한 지식

## <a name="about-this-sample"></a>이 샘플 정보

봇에서 QnA Maker를 이용하려면 먼저 [QnA Maker](https://www.qnamaker.ai/)를 기반으로 지식을 만들어야 합니다(다음 섹션의 주제). 그러면 봇에서 지식을 사용자의 쿼리에 보낼 수 있으며, 응답하는 질문에 대한 최선의 답변을 제공합니다.

## <a name="ctabcs"></a>[C#](#tab/cs)
![QnABot 논리 흐름](./media/qnabot-logic-flow.png)

`OnMessageActivityAsync`는 수신된 각 사용자 입력에 대해 호출됩니다. 호출되면 샘플 코드의 `appsetting.json` 파일 내에 저장된 `_configuration` 정보에 액세스하여 미리 구성된 QnA Maker 기술 자료에 연결할 값을 찾습니다. 

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)
![QnABot JS 논리 흐름](./media/qnabot-js-logic-flow.png)

`OnMessage`는 수신된 각 사용자 입력에 대해 호출됩니다. 호출되면 샘플 코드의 `.env` 파일에서 제공된 값을 사용하여 미리 구성된 `qnamaker` 커넥터에 액세스합니다.  QnA Maker 메서드 `getAnswers`는 봇을 외부 QnA Maker 기술 자료에 연결합니다.

---
사용자의 입력은 기술 자료에 보내지며 반환된 최선의 답변이 사용자에게 다시 표시됩니다.

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>QnA Maker 서비스를 만들고 기술 자료 게시
먼저 QnA Maker 서비스를 만들어야 합니다. Azure에서 서비스를 만들려면 QnA Maker [설명서](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)에 나열된 단계를 따릅니다.

다음으로, 샘플 프로젝트의 CognitiveModels 폴더에 있는 `smartLightFAQ.tsv` 파일을 사용하여 기술 자료를 만듭니다. QnA Maker [기술 자료](https://docs.microsoft.com/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)를 생성, 학습 및 게시하는 단계는 QnA Maker 설명서에 나와 있습니다. 다음 단계에 따라 `qna` KB의 이름을 지정하고, `smartLightFAQ.tsv` 파일을 사용하여 KB를 채웁니다.

> 참고. 이 문서는 사용자가 개발한 QnA Maker 기술 자료에 액세스하는 데 사용할 수도 있습니다.

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>봇을 기술 자료에 연결하는 데 필요한 값 가져오기
1. [QnA Maker](https://www.qnamaker.ai/) 사이트에서 기술 자료를 선택합니다.
1. 기술 자료를 연 후 **설정**을 선택합니다. _서비스 이름_에 표시되는 값을 기록합니다. 이 값은 QnA Maker 포털 인터페이스를 사용하는 경우 관심 있는 기술 자료를 찾는 데 유용합니다. 기술 자료에 봇 앱을 연결하는 데 사용되지 않습니다. 
1. 아래로 스크롤하여 **배포 세부 정보**를 찾아 Postman HTTP 요청 샘플에서 다음 값을 적어 둡니다.
   - POST /knowledgebases/\<knowledge-base-id>/generateAnswer
   - Host: \<your-hostname> // Full URL ending with /qnamaker
   - 권한 부여: EndpointKey \<your-endpoint-key>
   
호스트 이름에 대한 전체 URL 문자열은 "https://< >.azure.net/qnamaker"와 같습니다. 다음 세 값은 Azure QnA 서비스를 통해 앱을 QnA Maker 기술 자료에 연결하는 데 필요한 정보를 제공합니다.  

## <a name="update-the-settings-file"></a>설정 파일 업데이트

먼저, 호스트 이름, 엔드포인트 키 및 기술 자료 ID(kbId) 등 기술 자료에 액세스하는 데 필요한 정보를 설정 파일에 추가합니다. 이 정보는 QnA Maker의 기술 자료의 **설정** 탭에서 저장한 값입니다. 

이 정보를 프로덕션용으로 배포하지 않는 경우 앱 ID 및 암호 필드를 비워둘 수 있습니다.

> [!NOTE]
> QnA Maker 기술 자료에 대한 액세스를 기존의 봇 애플리케이션에 추가하는 경우 QnA 항목에 대한 참고용 제목을 추가해야 합니다. 이 섹션의 "이름" 값은 해당 앱 내에서 이 정보에 액세스하는 데 필요한 키를 제공합니다.

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="update-your-appsettingsjson-file"></a>appsettings.json 파일 업데이트

```json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  
  "QnAKnowledgebaseId": "<knowledge-base-id>",
  "QnAAuthKey": "<your-endpoint-key>",
  "QnAEndpointHostName": "<your-hostname>"
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="update-your-env-file"></a>.env 파일 업데이트

```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<your-endpoint-key>"
QnAEndpointHostName="<your-hostname>"
```

---

## <a name="set-up-the-qna-maker-instance"></a>QnA Maker 인스턴스 설정

먼저 QnA Maker 기술 자료에 액세스하기 위한 개체를 만듭니다.

## <a name="ctabcs"></a>[C#](#tab/cs)

프로젝트에 **Microsoft.Bot.Builder.AI.QnA** NuGet 패키지가 설치되어 있는지 확인합니다.

**QnABot.cs**의 `OnMessageActivityAsync` 메서드에서 QnAMaker 인스턴스를 만듭니다. 또한 위에서 `appsettings.json`에 저장한 연결 정보의 이름을 `QnABot` 클래스에서 끌어올 수도 있습니다. 설정 파일의 기술 자료 연결 정보에 대해 다른 이름을 선택한 경우 여기서 선택한 이름을 반영하여 이름을 업데이트해야 합니다.

**Bots/QnABot.cs**  
[!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=32-37)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

프로젝트에 대한 npm 패키지 **botbuilder-ai**가 설치되어 있는지 확인합니다.

이 샘플의 경우 봇 논리의 코드는 **QnABot.js** 파일 내에 있습니다.

**QnABot.js** 파일에서는 .env 파일에 의해 제공된 연결 정보를 사용하여 QnA Maker 서비스에 대한 연결: _this.qnaMaker_를 설정합니다.

**QnAMaker.js** [!code-javascript[QnAMaker](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=19-23)]


---

## <a name="calling-qna-maker-from-your-bot"></a>봇에서 QnA Maker 호출

## <a name="ctabcs"></a>[C#](#tab/cs)

봇에서 QnAMaker의 응답을 요구하는 경우 봇 코드에서 `GetAnswersAsync()`를 호출하여 현재 컨텍스트에 맞는 응답을 가져옵니다. 자신의 고유한 기술 자료에 액세스하는 경우 아래의 _답변을 찾을 수 없음_ 메시지를 변경하여 사용자에게 유용한 지침을 제공하세요.

**QnABot.cs**  
[!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=43-52)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**QnABot.js** 파일에서는 사용자의 입력을 QnA Maker 서비스의 `getAnswers` 메서드에 전달하여 기술 자료에서 답변을 가져옵니다. QnA Maker가 응답을 반환하는 경우 해당 응답이 사용자에게 표시됩니다. 그렇지 않으면 사용자에게 'QnA Maker 답변을 찾을 수 없음' 메시지가 표시됩니다. 

**QnABot.js** [!code-javascript[OnMessage](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=43-59)]

---

## <a name="test-the-bot"></a>봇 테스트

샘플을 머신에서 로컬로 실행합니다. 아직 설치하지 않은 경우 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)를 설치합니다. 추가 지침은 [C# 샘플](https://aka.ms/cs-qna) 또는 [Javascript 샘플](https://aka.ms/js-qna-sample)에 대한 추가 정보 파일을 참조하세요.

에뮬레이터를 시작하고 봇에 연결한 다음, 아래와 같은 메시지를 보냅니다.

![qna 샘플 테스트](../media/emulator-v4/qna-test-bot.png)

## <a name="next-steps"></a>다음 단계

QnA Maker는 기타 Cognitive Services와 결합하여 봇을 훨씬 더 강력하게 만들 할 수 있습니다. 디스패치 도구는 봇에서 LUIS(Language Understanding)와 QnA를 결합하는 방법을 제공합니다.

> [!div class="nextstepaction"]
> [디스패치 도구를 사용하여 LUIS 및 QnA 서비스 결합](./bot-builder-tutorial-dispatch.md)
