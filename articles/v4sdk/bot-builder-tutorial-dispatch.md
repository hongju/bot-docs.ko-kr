---
title: 다중 LUIS 및 QnA 모델 사용 | Microsoft Docs
description: 봇에서 LUIS 및 QnA Maker를 사용하는 방법에 대해 알아봅니다.
keywords: LUIS, QnA, 디스패치 도구, 여러 서비스, 의도 라우팅
author: diberry
ms.author: diberry
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c72f978e927f05f430ec94cf747016f4ebc15c5d
ms.sourcegitcommit: 0e6c49964b96c1ac8485ba7afe0daae04b671138
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/01/2019
ms.locfileid: "67492006"
---
# <a name="use-multiple-luis-and-qna-models"></a>다중 LUIS 및 QnA 모델 사용

[!INCLUDE[applies-to](../includes/applies-to.md)]

봇이 여러 LUIS 모델과 QnA Maker 기술 자료를 사용하는 경우 디스패치 도구를 사용하여 사용자 입력과 가장 일치하는 LUIS 모델 또는 QnA Maker 기술 자료를 확인할 수 있습니다. 디스패치 도구는 단일 LUIS 앱을 만들어 사용자 입력을 올바른 모델에 라우팅하는 방식으로 이 작업을 수행합니다. CLI 명령을 비롯한 디스패치에 대한 자세한 내용은 [README][dispatch-readme]를 참조하세요.

## <a name="prerequisites"></a>필수 조건
- [봇 기본 사항](bot-builder-basics.md), [LUIS][howto-luis], and [QnA Maker][howto-qna]에 대한 지식 
- [Dispatch 도구](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
- [C# 샘플][cs-sample] 또는 [JS Sample][js-sample] 코드 리포지토리의 **NLP with Dispatch** 복사본
- LUIS 앱을 게시할 사용할 [luis.ai](https://www.luis.ai/) 계정
- QnA 지식 자료를 게시할 [QnA Maker](https://www.qnamaker.ai/) 계정

## <a name="about-this-sample"></a>이 샘플 정보

이 샘플은 미리 정의된 LUIS 및 QnA Maker 앱 세트를 기준으로 합니다.

## <a name="ctabcs"></a>[C#](#tab/cs)

![코드 샘플 논리 흐름](./media/tutorial-dispatch/dispatch-logic-flow.png)

사용자 입력이 수신될 때마다 `OnMessageActivityAsync`가 호출됩니다. 이 모듈은 가장 점수가 높은 사용자 의도를 찾고 해당 결과를 `DispatchToTopIntentAsync`에 전달합니다. 그런 다음, DispatchToTopIntentAsync가 적절한 앱 처리기를 호출합니다.

- `ProcessSampleQnAAsync` - bot FAQ 질문용
- `ProcessWeatherAsync` - 날씨 쿼리용
- `ProcessHomeAutomationAsync` - 주택 조명 명령용

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![코드 샘플 논리 흐름](./media/tutorial-dispatch/dispatch-logic-flow-js.png)

사용자 입력이 수신될 때마다 `onMessage`가 호출됩니다. 이 모듈은 가장 점수가 높은 사용자 의도를 찾고 해당 결과를 `dispatchToTopIntentAsync`에 전달합니다. 그런 다음, dispatchToTopIntentAsync가 적절한 앱 처리기를 호출합니다.

- `processSampleQnA` - bot FAQ 질문용
- `processWeather` - 날씨 쿼리용
- `processHomeAutomation` - 주택 조명 명령용

---

처리기가 LUIS 또는 QnA Maker 서비스를 호출하고, 생성된 결과를 사용자에게 다시 반환합니다.

## <a name="create-luis-apps-and-qna-knowledge-base"></a>LUIS 앱 및 QnA 기술 자료 만들기
디스패치 모델을 만들려면 LUIS 앱 및 QnA 기술 자료를 만들고 게시해야 합니다. 이 문서에서는 `\CognitiveModels` 폴더의 _NLP With Dispatch_ 샘플에 포함된 다음 모델을 게시합니다. 

| 이름 | 설명 |
|------|------|
| HomeAutomation | 연결된 엔터티 데이터를 사용하여 홈 자동화 의도를 인식하는 LUIS 앱입니다.|
| Weather | 위치 데이터를 사용하여 날씨 관련 의도를 인식하는 LUIS 앱입니다.|
| QnAMaker  | 봇과 관련된 간단한 질문에 대한 답변을 제공하는 QnA Maker 기술 자료입니다. |

### <a name="create-luis-apps"></a>LUIS 앱 만들기
1. [LUIS 웹 포털](https://www.luis.ai/)에 로그인합니다. ‘내 앱’ 섹션에서 ‘새 앱 가져오기’ 탭을 선택합니다.   다음과 같은 대화 상자가 표시됩니다.

    ![LUIS json 파일 가져오기](./media/tutorial-dispatch/import-new-luis-app.png)

2. _앱 파일 선택_ 단추를 선택하고 샘플 코드의 CognitiveModel 폴더로 이동한 후 'HomeAutomation.json' 파일을 선택합니다. 이름 필드(옵션)는 비워 두세요. 

3. _완료_ 를 선택합니다.

4. LUIS에 Home Automation 앱이 열리면 _학습_ 단추를 선택합니다. 그러면 'home-automation.json' 파일을 사용하여 방금 가져온 발화 세트를 앱이 학습합니다.

5. 학습이 완료되면 _게시_ 단추를 선택합니다. 다음과 같은 대화 상자가 표시됩니다.

    ![LUIS 앱 게시](./media/tutorial-dispatch/publish-luis-app.png)

6. '프로덕션' 환경을 선택한 후 _게시_ 단추를 선택합니다.

7. 새 LUIS 앱이 게시되면 _관리_ 탭을 선택합니다. '애플리케이션 정보' 페이지에서 `Application ID` 값에 "_app-id-for-app_"를, `Display name` 값에 "_name-of-app_"를 기재합니다. '키 및 엔드포인트' 페이지에서 `Authoring Key` 값에 "_your-luis-authoring-key_"를, `Region`에 "_your-region_"을 기재합니다. 이러한 값은 나중에 'appsetting.json' 파일 내에서 사용됩니다.

8. 완료되면 'Weather.json' 파일에 이러한 단계를 반복하여 LUIS 날씨 앱 및 LUIS 디스패치 앱을 모두 _학습_ 시키고 _게시_ 합니다.

### <a name="create-qna-maker-knowledge-base"></a>QnA Maker 기술 자료 만들기

QnA Maker 기술 자료를 설정하는 첫 번째 단계는 Azure에서 QnA Maker 서비스를 설정하는 것입니다. 이렇게 하려면 [여기](https://aka.ms/create-qna-maker)에 나와 있는 단계별 지침을 따르세요.

Azure에서 QnA Maker 서비스가 만들어지면 QnA Maker 서비스에 제공된 Cognitive Services _키 1_ 을 기록해야 합니다. 이 키는 QnA Maker 앱을 디스패치 애플리케이션에 추가할 때 \<azure-qna-service-key1>로 사용됩니다. 다음 단계에서는 이 키를 제공합니다.
    
![Cognitive Service 선택](./media/tutorial-dispatch/select-qna-cognitive-service.png)

1. Azure Portal에서 QnA Maker Cognitive Service를 선택합니다.

    ![Cognitive Service 키 선택](./media/tutorial-dispatch/select-cognitive-service-keys.png)

1. 왼쪽 메뉴의 _리소스 관리_ 섹션 아래에 있는 [키] 아이콘을 선택합니다.

    ![Cognitive Service 키 1 선택](./media/tutorial-dispatch/select-cognitive-service-key1.png)

1. _키 1_ 값을 클립보드에 복사하고 로컬로 저장합니다. 이 키는 QnA Maker 앱을 디스패치 애플리케이션에 추가할 때 (-k) 키 값인 \<azure-qna-service-key1>로 사용됩니다.

1. 이제 [QnAMaker 웹 포털](https://qnamaker.ai)에 로그인합니다. 

1. 2단계에서 다음을 선택합니다.

    * Azure AD 계정.
    * Azure 구독 이름.
    * QnA Maker 서비스에 대해 만든 이름. (처음에 Azure QnA 서비스가 이 풀다운 목록에 표시되지 않을 경우 페이지를 새로 고쳐 보세요.)

    ![QnA 2단계 만들기](./media/tutorial-dispatch/create-qna-step-2.png) 
     

1. 3단계에서 QnA Maker 기술 자료의 이름을 지정합니다. 이 예제에서는 ‘sample-qna’라는 이름을 사용합니다.

    ![QnA 3단계 만들기](./media/tutorial-dispatch/create-qna-step-3.png)

1. 4단계에서 ‘+ 파일 추가’ 옵션을 선택하고 샘플 코드의 CognitiveModel 폴더로 이동한 다음, ‘QnAMaker.tsv’ 파일을 선택합니다.  기술 자료에 ‘잡담’ 퍼스낼리티를 추가하는 선택 항목도 있지만, 예제에는 이 옵션이 포함되어 있지 않습니다. 

    ![QnA 4단계 만들기](./media/tutorial-dispatch/create-qna-step-4.png)

1. 5단계에서 ‘기술 자료 만들기’를 선택합니다. 

1. 업로드된 파일에서 기술 자료가 생성되면 ‘저장 후 학습’을 선택하고, 작업이 완료되면 ‘게시’ 탭을 선택하고 앱을 게시합니다.  

1. QnA Maker 앱이 게시되면 _설정_ 탭을 선택하고 '배포 세부 정보'가 나올 때까지 아래로 스크롤합니다. _Postman_ 샘플 HTTP 요청의 다음 값을 기록합니다.

    ```text
    POST /knowledge bases/<knowledge-base-id>/generateAnswer
    Host: <your-hostname>  // NOTE - this is a URL.
    Authorization: EndpointKey <qna-maker-resource-key>
    ```
    
    호스트 이름에 대한 전체 URL 문자열은 "https://< >.azure.net/qnamaker"와 같습니다. 이러한 값은 나중에 `appsettings.json` 또는 `.env` 파일 내에서 사용됩니다.

## <a name="dispatch-app-needs-read-access-to-existing-apps"></a>디스패치 앱에 기존 앱에 대한 읽기 권한이 필요함

디스패치 도구는 LUIS 및 QnA Maker 앱에 디스패치되는 새로운 부모 LUIS 앱을 만들기 위해 기존 LUIS 및 QnA Maker 앱을 읽는 작성 권한이 필요합니다. 이 액세스 권한은 앱 ID 및 작성 키와 함께 제공됩니다. 두 개의 LUIS 앱과 QnA Maker 앱의 ID와 키가 각각 필요합니다.

|앱|정보 위치|
|--|--|
|LUIS|앱 ID - [LUIS 포털](https://www.luis.ai)에서 확인할 수 있습니다. 각 앱에 대해 관리 -> 애플리케이션 정보를 선택합니다.<br>작성 키 - LUIS 포털에서 확인할 수 있습니다. 오른쪽 상단 모서리에서 고유한 사용자, 설정을 차례로 선택합니다.|
|QnA Maker| 앱 ID - 앱을 게시한 후 [QnA Maker 포털](https://http://qnamaker.ai)의 설정 페이지에서 확인할 수 있습니다. 이 ID는 기술 자료 이후 POST 명령의 첫 번째 부분에 있는 ID입니다. 앱 ID를 확인할 수 있는 위치의 예는 `POST /knowledgebases/{APP-ID}/generateAnswer`입니다.<br>작성 키 - Azure Portal의 **키** 아래에서 QnA Maker 리소스에 대해 제공됩니다. 키 중 하나만 있으면 됩니다.|

## <a name="create-the-dispatch-model"></a>디스패치 모델 만들기

디스패치 도구용 CLI 인터페이스는 올바른 LUIS 또는 QnA Maker 앱에 디스패치하기 위한 모델을 만듭니다.

1. 명령 프롬프트 또는 터미널 창을 열고 **CognitiveModels** 디렉터리로 이동합니다.
1. 최신 버전의 npm 및 디스패치 도구가 설치되어 있는지 확인합니다.

    ```cmd
    npm i -g npm
    npm i -g botdispatch
    ```

1. `dispatch init`를 사용하여 디스패치 모델의 `.dispatch` 파일을 초기화합니다. 인식할 수 있는 파일 이름을 사용하여 파일을 만듭니다.

    ```cmd
    dispatch init -n <filename-to-create> --luisAuthoringKey "<your-luis-authoring-key>" --luisAuthoringRegion <your-region>
    ```

1. `dispatch add`를 사용하여 LUIS 앱 및 QnA Maker 기술 자료를 `.dispatch` 파일에 추가합니다.

    ```cmd
    dispatch add -t luis -i "<app-id-for-weather-app>" -n "<name-of-weather-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_Weather
    dispatch add -t luis -i "<app-id-for-home-automation-app>" -n "<name-of-home-automation-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_HomeAutomation
    dispatch add -t qna -i "<knowledge-base-id>" -n "<knowledge-base-name>" -k "<azure-qna-service-key1>" --intentName q_sample-qna
    ```

1. `dispatch create`를 사용하여 `.dispatch` 파일에서 디스패치 모델을 생성합니다.

    ```cmd
    dispatch create
    ```

1. 방금 만든 디스패치 LUIS 앱을 게시합니다.

## <a name="use-the-dispatch-luis-app"></a>디스패치 LUIS 앱 사용

생성된 LUIS 앱은 각 자식 앱과 기술 자료의 의도를 정의할 뿐 아니라, 발화에 적합한 항목이 없는 경우를 나타내는 ‘없음’ 의도를 정의합니다. 

- `l_HomeAutomation`
- `l_Weather`
- `None`
- `q_sample-qna`

해당 서비스를 올바른 이름으로 게시해야 봇을 제대로 실행할 수 있습니다. 봇이 이러한 서비스에 액세스하려면 게시된 서비스에 대한 정보를 알아야 합니다.

봇에는 단일 QnA Maker 기술 자료와 세 개의 LUIS 앱(디스패치, 날씨, 홈 자동화)에 대한 쿼리 예측 엔드포인트가 필요합니다. 다음 표를 사용하여 엔드포인트 키를 찾습니다.

|앱|쿼리 엔드포인트 키 위치|
|--|--|
|LUIS|LUIS 포털에서 각 LUIS 앱에 대해 관리 섹션의 **키와 엔드포인트 설정**을 선택하여 각 앱과 연결된 키를 찾습니다. 이 자습서를 따르는 경우 엔드포인트 키는 `<your-luis-authoring-key>`과 동일한 키입니다. 작성 키는 1,000개 엔드포인트 적중을 허용한 후 만료됩니다.|
|QnA Maker|QnA Maker 포털에서 기술 자료에 대해, 관리 설정의 **권한** 헤더 Postman 설정에 `EndpointKey ` 텍스트 없이 표시된 키 값을 사용합니다.|

이 값은 C#의 경우 **appsettings.json**, JavaScript의 경우 **.env** 파일에서 사용됩니다.

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="installing-packages"></a>패키지 설치

이 앱을 처음으로 실행하기 전에 몇 가지 nuget 패키지가 설치되어 있는지 확인하세요.

**Microsoft.Bot.Builder**

**Microsoft.Bot.Builder.AI.Luis**

**Microsoft.Bot.Builder.AI.QnA**

### <a name="manually-update-your-appsettingsjson-file"></a>appsettings.json 파일을 수동으로 업데이트

모든 서비스 앱이 생성되면 각 앱에 대한 정보를 'appsettings.json' 파일에 추가해야 합니다. 초기 [C# 샘플][cs-sample] 코드에는 빈 appsettings.json 파일이 포함되어 있습니다.

**appsettings.json**  
[!code-json[AppSettings](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/AppSettings.json?range=8-17)]

다음 지침에 따라 아래에 표시된 각 엔터티에 앞서 기록한 값을 추가합니다.

**appsettings.json**
```json
"MicrosoftAppId": "",
"MicrosoftAppPassword": "",
  
"QnAKnowledgebaseId": "<knowledge-base-id>",
"QnAEndpointKey": "<qna-maker-resource-key>",
"QnAEndpointHostName": "<your-hostname>",

"LuisAppId": "<app-id-for-dispatch-app>",
"LuisAPIKey": "<your-luis-endpoint-key>",
"LuisAPIHostName": "<your-dispatch-app-region>",
```
모든 변경을 완료했으면 이 파일을 저장합니다.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="installing-packages"></a>패키지 설치

이 앱을 처음으로 실행하기 전에 몇 가지 npm 패키지를 설치해야 합니다.

```powershell
npm install --save botbuilder
npm install --save botbuilder-ai
```
.env 구성 파일을 사용하려면 봇에 추가 패키지가 포함되어 있어야 합니다.

```powershell
npm install --save dotenv
```

### <a name="manually-update-your-env-file"></a>수동으로 .env 파일 업데이트

모든 서비스 앱이 생성되면 각 앱에 대한 정보를 '.env' 파일에 추가해야 합니다. 초기 [JavaScript 샘플][js-sample] 코드에는 빈 .env 파일이 포함되어 있습니다. 

**.env**  
[!code-file[EmptyEnv](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/.env?range=1-10)]

아래에 나와 있는 것처럼 서비스 연결 값을 추가합니다.

**.env**
```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAEndpointKey="<qna-maker-resource-key>"
QnAEndpointHostName="<your-hostname>"

LuisAppId=<app-id-for-dispatch-app>
LuisAPIKey=<your-luis-endpoint-key>
LuisAPIHostName=<your-dispatch-app-region>

```
모든 변경을 완료했으면 이 파일을 저장합니다.

---

### <a name="connect-to-the-services-from-your-bot"></a>봇에서 서비스에 연결

봇은 디스패치, LUIS 및 QnA Maker 서비스에 연결하기 위해 설정 파일(`appsettings.json` 또는 `.env` 파일)에서 정보를 끌어옵니다.

## <a name="ctabcs"></a>[C#](#tab/cs)

**BotServices.cs**에서 구성 파일 _appsettings.json_ 에 포함된 정보는 디스패치 봇을 `Dispatch` 및 `SampleQnA` 서비스에 연결하는 데 사용됩니다. 생성자는 개발자가 지정한 값을 사용하여 이러한 서비스에 연결합니다.

**BotServices.cs**  
[!code-csharp[ReadConfigurationInfo](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/BotServices.cs?range=14-30)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**dispatchBot.js**에서 구성 파일 _.env_ 에 포함된 정보는 디스패치 봇을 _LuisRecognizer(디스패치)_ 및 _QnAMaker_ 서비스에 연결하는 데 사용됩니다. 생성자는 개발자가 지정한 값을 사용하여 이러한 서비스에 연결합니다.

**dispatchBot.js**  
[!code-javascript[ReadConfigurationInfo](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=18-31)]

---

### <a name="call-the-services-from-your-bot"></a>봇에서 서비스 호출

봇 논리는 사용자 입력이 있을 때마다 결합된 디스패치 모델을 기준으로 사용자 입력을 검사하고, 상위에 반환된 의도를 찾고, 이러한 정보를 사용하여 입력에 적절한 서비스를 호출합니다.

## <a name="ctabcs"></a>[C#](#tab/cs)

**DispatchBot.cs** 파일에서 `OnMessageActivityAsync` 메서드가 호출될 때마다 디스패치 모델을 기준으로 수신 사용자 메시지를 검사합니다. 그런 다음, 디스패치 모델의 `topIntent` 및 `recognizerResult`를 올바른 메서드에 전달하여 서비스를 호출하고 결과를 반환합니다.

**DispatchBot.cs**  
[!code-csharp[OnMessageActivity](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=26-36)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**dispatchBot.js** `onMessage` 메서드에서 디스패치 모델을 기준으로 사용자 입력 메시지를 검사하고, _topIntent_ 를 확인한 후, _dispatchToTopIntentAsync_ 를 호출하여 이를 전달합니다.

**dispatchBot.js**  

[!code-javascript[OnMessageActivity](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=37-50)]

---

### <a name="work-with-the-recognition-results"></a>인식 결과 작업

## <a name="ctabcs"></a>[C#](#tab/cs)

모델에서 생성하는 결과에 발언을 가장 적절하게 처리할 수 있는 서비스가 나타납니다. 이 봇의 코드는 해당 서비스에 요청을 라우팅한 후 호출된 서비스에의 응답을 요약합니다. 이 코드는 디스패치에서 반환된 _의도_ 에 따라 올바른 LUIS 모델 또는 QnA 서비스로 라우팅합니다.

**DispatchBot.cs**  
[!code-csharp[DispatchToTop](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=51-69)]

`ProcessHomeAutomationAsync` 또는 `ProcessWeatherAsync` 메서드가 호출되면 _luisResult.ConnectedServiceResult_ 내에 디스패치 모델의 결과가 전달됩니다. 그러면 지정된 메서드가 디스패치 모델의 상위 의도와 검색된 모든 의도 및 엔터티의 순위 목록을 보여주는 사용자 피드백을 제공합니다.

`q_sample-qna` 메서드가 호출되면 turnContext 내에 포함된 사용자 입력을 사용하여 기술 자료의 답변을 생성하고 해당 결과를 사용자에게 표시합니다.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

모델에서 생성하는 결과에 발언을 가장 적절하게 처리할 수 있는 서비스가 나타납니다. 이 샘플의 코드는 인식된 _topIntent_ 를 사용하여 요청을 해당 서비스로 라우팅하는 방법을 보여줍니다.

**DispatchBot.cs**  
[!code-javascript[DispatchToTop](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=67-83)]

`processHomeAutomation` 또는 `processWeather` 메서드가 호출되면 _recognizerResult.luisResult_ 내에 디스패치 모델의 결과가 전달됩니다. 그러면 지정된 메서드가 디스패치 모델의 상위 의도와 검색된 모든 의도 및 엔터티의 순위 목록을 보여주는 사용자 피드백을 제공합니다.

`q_sample-qna` 메서드가 호출되면 turnContext 내에 포함된 사용자 입력을 사용하여 기술 자료의 답변을 생성하고 해당 결과를 사용자에게 표시합니다.

---

> [!NOTE]
> 프로덕션 애플리케이션일 경우 선택된 LUIS 메서드가 지정된 서비스에 연결하고, 사용자 입력을 전달하고, 반환된 LUIS 의도 및 엔터티 데이터를 처리합니다.

## <a name="test-your-bot"></a>봇 테스트

1. 개발 환경을 사용하여 샘플 코드를 시작합니다. 앱에서 열린 브라우저 창의 주소 표시줄에 표시된 _localhost_ 주소(“https://localhost:<Port_Number>”)를 적어 둡니다. 
1. Bot Framework Emulator를 열고 `Create a new bot configuration`을 선택합니다. `.bot` 파일을 사용하면 봇 에뮬레이터의 ‘검사기’를 통해 LUIS 및 QnA Maker에서 반환된 JSON을 확인할 수 있습니다. 
1. **새 봇 구성** 대화 상자에서 봇 이름과 엔드포인트 URL(예: `http://localhost:3978/api/messages`)을 입력합니다. 봇 샘플 코드 프로젝트의 루트에 파일을 저장합니다.
1. 봇 파일을 열고 LUIS 및 QnA Maker 앱 섹션을 추가합니다. [이 예제 파일](https://github.com/microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json)을 설정 템플릿으로 사용합니다. 변경 내용을 저장합니다.
1. **내 봇** 목록에서 봇 이름을 선택하여 실행 중인 봇에 액세스합니다. 참고로, 봇용 서비스에서 다루는 몇 가지 질문과 명령은 다음과 같습니다.

    - QnA Maker
      - `hi`, `good morning`
      - `what are you`, `what do you do`
    - LUIS(가정 자동화)
      - `turn on bedroom light`
      - `turn off bedroom light`
      - `make some coffee`
    - LUIS(날씨)
      - `whats the weather in redmond washington`
      - `what's the forecast for london`
      - `show me the forecast for nebraska`

## <a name="dispatch-for-user-utterance-to-qna-maker"></a>QnA Maker에 대한 사용자 발화의 디스패치

1. 봇 에뮬레이터에서 `hi` 텍스트를 입력하고 발화를 제출합니다. 봇은 이 쿼리를 디스패치 LUIS 앱에 제출하고, 추가 처리를 위해 이 발화를 가져와야 하는 자식 앱을 나타내는 응답을 받습니다. 

1. 로그에서 `LUIS Trace` 줄을 선택하면 봇 에뮬레이터에서 LUIS 응답을 확인할 수 있습니다. 디스패치 LUIS 앱의 LUIS 결과가 검사기에 표시됩니다. 

    ```json
    {
      "luisResponse": {
        "entities": [],
        "intents": [
          {
            "intent": "q_sample-qna",
            "score": 0.9489713
          },
          {
            "intent": "l_HomeAutomation",
            "score": 0.0612499453
          },
          {
            "intent": "None",
            "score": 0.008567564
          },
          {
            "intent": "l_Weather",
            "score": 0.0025761195
          }
        ],
        "query": "Hi",
        "topScoringIntent": {
          "intent": "q_sample-qna",
          "score": 0.9489713
        }
      }
    }
    ```
    
    `hi` 발화가 디스패치 LUIS 앱의 **q_sample-qna** 의도에 속하고, `topScoringIntent`로 선택되었기 때문에 봇이 두 번째 요청을 수행하며, 이번에는 동일한 발화를 사용하여 QnA Maker 앱에 요청합니다. 

1. 봇 에뮬레이터 로그에서 `QnAMaker Trace` 줄을 선택합니다. QnA Maker 결과가 검사기에 표시됩니다. 

```json
{
    "questions": [
        "hi",
        "greetings",
        "good morning",
        "good evening"
    ],
    "answer": "Hello!",
    "score": 1,
    "id": 96,
    "source": "QnAMaker.tsv",
    "metadata": [],
    "context": {
        "isContextOnly": false,
        "prompts": []
    }
}
```

## <a name="resolving-incorrect-top-intent-from-dispatch"></a>디스패치에서 잘못된 상위 의도 해결

봇이 실행되면 디스패치된 앱 간에 유사하거나 겹치는 발화를 제거하여 봇의 성능을 개선할 수 있습니다. 예를 들어 `Home Automation` LUIS 앱에서 “TurnOnLights” 의도에 매핑되는 “내 전등 켜기” 등의 요청과 QnA maker에 전달될 수 있도록 "None" 의도에 매핑하는 "내 전등이 켜지지 않는 이유는?"과 같은 요청을 가정해 봅시다. 이 두 가지 발화는 너무 비슷해, 디스패치 LUIS 앱에서 올바른 자식 앱이 LUIS 앱인지, QnA Maker 앱인지 확인할 수 없습니다.

디스패치를 사용하여 LUIS 앱과 QnA Maker 앱을 결합하는 경우, 다음 중 ‘하나’를 수행해야 합니다. 

* 자식 `Home Automation` LUIS 앱에서 “없음” 의도를 제거하고, 해당 의도의 발화를 디스패처 앱의 “없음” 의도에 추가합니다.
* 디스패치 LUIS 앱의 “None” 의도와 일치하는 메시지를 QnA Maker 서비스에 전달하도록 봇에 논리를 추가합니다. 디스패치 LUIS 앱의 점수와 QnA Maker 앱의 점수를 비교합니다. 최고 점수를 사용합니다. 그러면 디스패치 주기에서 QnA Maker가 사실상 제거됩니다. 

위의 두 동작 중 하나로 봇이 '답변을 찾을 수 없습니다.'라는 메시지로 사용자에게 응답하는 횟수를 줄일 수 있습니다.

### <a name="to-update-or-create-a-new-luis-model"></a>LUIS 모델 업데이트 또는 새로 만들기

이 샘플은 미리 구성된 LUIS 모델을 기반으로 합니다. 이 모델을 업데이트하거나 새 LUIS 모델을 만드는 데 도움이 되는 추가 정보는 [여기](https://aka.ms/create-luis-model#updating-your-cognitive-models)서 찾을 수 있습니다.

### <a name="to-delete-resources"></a>리소스 삭제

이 샘플에서는 아래에 나열된 단계를 사용하여 삭제할 수 있는 다양한 애플리케이션과 리소스를 만들지만 *다른 애플리케이션 또는 서비스*에서 사용하는 리소스는 삭제하면 안 됩니다.

LUIS 리소스를 삭제하려면:

1. [luis.ai](https://www.luis.ai) 포털에 로그인합니다.
1. _내 앱_ 페이지로 이동합니다.
1. 이 샘플로 만든 앱을 선택합니다.
   - `Home Automation`
   - `Weather`
   - `NLP-With-Dispatch-BotDispatch`
1. _삭제_ 를 클릭하고 _확인_ 을 클릭하여 확인합니다.

QnA Maker 리소스를 삭제하려면:

1. [qnamaker.ai](https://www.qnamaker.ai/) 포털에 로그인합니다.
1. _내 기술 자료_ 페이지로 이동합니다.
1. `Sample QnA` 기술 자료의 삭제 단추를 클릭하고 _삭제_를 클릭하여 확인합니다.

### <a name="best-practice"></a>모범 사례

이 샘플에서 사용된 서비스를 향상시키려면 [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-best-practices) 및 [QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/concepts/best-practices)에 대한 모범 사례를 참조하세요.


[howto-luis]: bot-builder-howto-v4-luis.md
[howto-qna]: bot-builder-howto-qna.md

[cs-sample]: https://aka.ms/dispatch-sample-cs
[js-sample]: https://aka.ms/dispatch-sample-js

[dispatch-readme]: https://aka.ms/botbuilder-tools-dispatch
