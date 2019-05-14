---
title: 다중 LUIS 및 QnA 모델 사용 | Microsoft Docs
description: 봇에서 LUIS 및 QnA Maker를 사용하는 방법에 대해 알아봅니다.
keywords: LUIS, QnA, 디스패치 도구, 여러 서비스, 의도 라우팅
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b3e488615f318529935d35dbebbed2dd3b734f62
ms.sourcegitcommit: 3e3c9986b95532197e187b9cc562e6a1452cbd95
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/06/2019
ms.locfileid: "65039739"
---
# <a name="use-multiple-luis-and-qna-models"></a>다중 LUIS 및 QnA 모델 사용

[!INCLUDE[applies-to](../includes/applies-to.md)]

봇이 여러 LUIS 모델 및 QnA Maker 기술 자료(KBs)를 사용하는 경우 디스패치 도구를 사용하여 사용자 입력과 가장 일치하는 LUIS 모델 또는 QnA Maker KB를 확인할 수 있습니다. 디스패치 도구는 단일 LUIS 앱을 만들어 사용자 입력을 적절한 모델에 라우팅하는 방식으로 이 작업을 수행합니다. CLI 명령을 비롯한 디스패치에 대한 자세한 내용은 [README][dispatch-readme]를 참조하세요.

## <a name="prerequisites"></a>필수 조건
- [봇 기본 사항](bot-builder-basics.md), [LUIS][howto-luis] 및 [QnA Maker][howto-qna]에 대한 지식 
- [Dispatch 도구](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
- [C# 샘플][cs-sample] 또는 [JS 샘플][js-sample] 코드 리포지토리의 **NLP with Dispatch** 복사본
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

## <a name="create-luis-apps-and-qna-kb"></a>LUIS 앱 및 QnA KB 만들기
디스패치 모델을 만들려면 LUIS 앱 및 QnA KB를 만들고 게시해야 합니다. 이 문서에서는 `\CognitiveModels` 폴더의 _NLP With Dispatch_ 샘플에 포함된 다음 모델을 게시합니다. 

| Name | 설명 |
|------|------|
| HomeAutomation | 연결된 엔터티 데이터를 사용하여 홈 자동화 의도를 인식하는 LUIS 앱입니다.|
| Weather | 위치 데이터를 사용하여 날씨 관련 의도를 인식하는 LUIS 앱입니다.|
| QnAMaker  | 봇과 관련된 간단한 질문에 대한 답변을 제공하는 QnA Maker KB입니다. |

### <a name="create-luis-apps"></a>LUIS 앱 만들기
1. [LUIS 웹 포털](https://www.luis.ai/)에 로그인합니다. _내 앱_ 섹션에서 _새 앱 가져오기_ 탭을 선택합니다. 다음과 같은 대화 상자가 표시됩니다.

![LUIS json 파일 가져오기](./media/tutorial-dispatch/import-new-luis-app.png)

2. _앱 파일 선택_ 단추를 선택하고 샘플 코드의 CognitiveModel 폴더로 이동한 후 'HomeAutomation.json' 파일을 선택합니다. 이름 필드(옵션)는 비워 두세요. 

3. _완료_를 선택합니다.

4. LUIS에 Home Automation 앱이 열리면 _학습_ 단추를 선택합니다. 그러면 'home-automation.json' 파일을 사용하여 방금 가져온 발화 세트를 앱이 학습합니다.

5. 학습이 완료되면 _게시_ 단추를 선택합니다. 다음과 같은 대화 상자가 표시됩니다.

![LUIS 앱 게시](./media/tutorial-dispatch/publish-luis-app.png)

6. '프로덕션' 환경을 선택한 후 _게시_ 단추를 선택합니다.

7. 새 LUIS 앱이 게시되면 _관리_ 탭을 선택합니다. '애플리케이션 정보' 페이지에서 `Application ID` 값에 "_app-id-for-app_"를, `Display name` 값에 "_name-of-app_"를 기재합니다. '키 및 엔드포인트' 페이지에서 `Authoring Key` 값에 "_your-luis-authoring-key_"를, `Region`에 "_your-region_"을 기재합니다. 이러한 값은 나중에 'appsetting.json' 파일 내에서 사용됩니다.

8. 완료되면 'Weather.json' 파일에 이러한 단계를 반복하여 LUIS 날씨 앱 및 LUIS 디스패치 앱을 모두 _학습_시키고 _게시_합니다.

### <a name="create-qna-maker-kb"></a>QnA Maker KB 만들기

QnA Maker KB를 설정하는 첫 번째 단계는 Azure에서 QnA Maker 서비스를 설정하는 것입니다. 이렇게 하려면 [여기](https://aka.ms/create-qna-maker)에 나와 있는 단계별 지침을 따르세요. 이제 [QnAMaker 웹 포털](https://qnamaker.ai)에 로그인합니다. 아래의 2단계로 이동합니다.

![QnA 2단계 만들기](./media/tutorial-dispatch/create-qna-step-2.png)

다음을 선택합니다.
1. Azure AD 계정.
1. Azure 구독 이름.
1. QnA Maker 서비스에 대해 만든 이름. (처음에 Azure QnA 서비스가 이 풀다운 목록에 표시되지 않을 경우 페이지를 새로 고쳐 보세요.) 

3단계로 이동합니다.

![QnA 3단계 만들기](./media/tutorial-dispatch/create-qna-step-3.png)

QnA Maker 기술 자료의 이름을 지정합니다. 이 예에서는 'sample-qna'라는 이름을 사용합니다.

4단계로 이동합니다.

![QnA 4단계 만들기](./media/tutorial-dispatch/create-qna-step-4.png)

_+ 파일 추가_ 옵션을 선택하고 샘플 코드의 CognitiveModel 폴더로 이동한 후 'QnAMaker.tsv' 파일을 선택합니다.

기술 자료에 _잡담_ 퍼스낼리티를 추가하는 추가 옵션이 있지만 이 예에서는 이 옵션을 다루지 않습니다.

5단계로 이동합니다.

_KB 만들기_를 선택합니다.

업로드된 파일에서 기술 자료가 생성되면 _저장 및 학습_을 선택하고, 완료되면 _게시_ 탭을 선택하고 앱을 게시합니다.

QnA Maker 앱이 게시되면 _설정_ 탭을 선택하고 '배포 세부 정보'가 나올 때까지 아래로 스크롤합니다. _Postman_ 샘플 HTTP 요청의 다음 값을 기록합니다.

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL.
Authorization: EndpointKey <your-endpoint-key>
```

호스트 이름에 대한 전체 URL 문자열은 "https://< >.azure.net/qnamaker"와 같습니다.

이러한 값은 나중에 `appsettings.json` 또는 `.env` 파일 내에서 사용됩니다.

LUIS 앱 및 QnA Maker 기술 자료 이름과 ID를 메모해 두세요. 또한 LUIS 제작 키 및 Cognitive Services 구독 키도 메모해 두세요. 이 프로세스를 완료하려면 이 모든 정보가 필요합니다.

## <a name="create-the-dispatch-model"></a>디스패치 모델 만들기

디스패치 도구용 CLI 인터페이스는 적절한 서비스로 디스패치하기 위한 모델을 만듭니다.

1. 명령 프롬프트 또는 터미널 창을 열고 **CognitiveModels** 디렉터리로 이동합니다.
1. 최신 버전의 npm 및 디스패치 도구가 설치되어 있는지 확인합니다.

    ```cmd
    npm i -g npm
    npm i -g botdispatch
    ```

1. `dispatch init`를 사용하여 디스패치 모델용 .dispatch 파일을 초기화합니다. 인식할 파일 이름을 사용하여 이를 만듭니다.

    ```cmd
    dispatch init -n <filename-to-create> --luisAuthoringKey "<your-luis-authoring-key>" --luisAuthoringRegion <your-region>
    ```

1. `dispatch add`를 사용하여 LUIS 앱 및 QnA Maker 기술 자료를 .dispatch 파일에 추가합니다.

    ```cmd
    dispatch add -t luis -i "<app-id-for-weather-app>" -n "<name-of-weather-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_Weather
    dispatch add -t luis -i "<app-id-for-home-automation-app>" -n "<name-of-home-automation-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_HomeAutomation
    dispatch add -t qna -i "<knowledge-base-id>" -n "<knowledge-base-name>" -k "<your-cognitive-services-subscription-id>" --intentName q_sample-qna
    ```

1. `dispatch create`를 사용하여 .dispatch 파일에서 디스패치 모델을 생성합니다.

    ```cmd
    dispatch create
    ```

1. 생성된 디스패치 모델 JSON 파일을 사용하여 디스패치 LUIS 앱을 게시합니다.

## <a name="use-the-dispatch-model"></a>디스패치 모델 사용

생성된 모델은 각 앱 및 기술 자료의 의도는 물론 발화에 적합한 항목이 없는 경우에 대한 _none_ 의도도 정의합니다.

- `l_HomeAutomation`
- `l_Weather`
- `None`
- `q_sample-qna`

이러한 서비스를 올바른 이름으로 게시해야 봇이 제대로 실행할 수 있습니다.

봇이 이러한 서비스에 액세스하려면 게시된 서비스에 대한 정보를 알아야 합니다.

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="installing-packages"></a>패키지 설치

이 앱을 처음으로 실행하기 전에 몇 가지 nuget 패키지가 설치되어 있는지 확인하세요.

**Microsoft.Bot.Builder**

**Microsoft.Bot.Builder.AI.Luis**

**Microsoft.Bot.Builder.AI.QnA**

### <a name="manually-update-your-appsettingsjson-file"></a>appsettings.json 파일을 수동으로 업데이트

모든 서비스 앱이 생성되면 각 앱에 대한 정보를 'appsettings.json' 파일에 추가해야 합니다. 초기 [C# 샘플][cs-sample] 코드에는 빈 appsettings.json 파일이 포함되어 있습니다.

**appsettings.json** [!code-json[AppSettings](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/AppSettings.json?range=8-17)]

다음 지침에 따라 아래에 표시된 각 엔터티에 앞서 기록한 값을 추가합니다.

**appsettings.json**
```json
"MicrosoftAppId": "",
"MicrosoftAppPassword": "",
  
"QnAKnowledgebaseId": "<knowledge-base-id>",
"QnAAuthKey": "<your-endpoint-key>",
"QnAEndpointHostName": "<your-hostname>",

"LuisAppId": "<app-id-for-dispatch-app>",
"LuisAPIKey": "<your-luis-authoring-key>",
"LuisAPIHostName": "<your-dispatch-app-region>",
```
모든 변경 사항을 적용했으면 이 파일을 저장하세요.

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

**.env** [!code-file[EmptyEnv](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/.env?range=1-10)]

아래에 나와 있는 것처럼 서비스 연결 값을 추가합니다.

**.env**
```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<your-endpoint-key>"
QnAEndpointHostName="<your-hostname>"

LuisAppId=<app-id-for-dispatch-app>
LuisAPIKey=<your-luis-authoring-key>
LuisAPIHostName=<your-dispatch-app-region>

```
모든 변경 사항을 적용했으면 이 파일을 저장하세요.

---

### <a name="connect-to-the-services-from-your-bot"></a>봇에서 서비스에 연결

봇은 Dispatch, LUIS 및 QnA Maker 서비스에 연결하기 위해 개발자가 이전에 지정한 설정에서 정보를 끌어옵니다.

## <a name="ctabcs"></a>[C#](#tab/cs)

**BotServices.cs**에서 구성 파일 _appsettings.json_에 포함된 정보는 디스패치 봇을 `Dispatch` 및 `SampleQnA` 서비스에 연결하는 데 사용됩니다. 생성자는 개발자가 지정한 값을 사용하여 이러한 서비스에 연결합니다.

**BotServices.cs** [!code-csharp[ReadConfigurationInfo](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/BotServices.cs?range=14-30)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**dispatchBot.js**에서 구성 파일 _.env_에 포함된 정보는 디스패치 봇을 _LuisRecognizer(디스패치)_ 및 _QnAMaker_ 서비스에 연결하는 데 사용됩니다. 생성자는 개발자가 지정한 값을 사용하여 이러한 서비스에 연결합니다.

**dispatchBot.js** [!code-javascript[ReadConfigurationInfo](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=18-31)]

---

### <a name="call-the-services-from-your-bot"></a>봇에서 서비스 호출

봇 논리는 사용자 입력이 있을 때마다 결합된 디스패치 모델을 기준으로 사용자 입력을 검사하고, 상위에 반환된 의도를 찾고, 이러한 정보를 사용하여 입력에 적절한 서비스를 호출합니다.

## <a name="ctabcs"></a>[C#](#tab/cs)

**DispatchBot.cs** 파일에서 `OnMessageActivityAsync` 메서드가 호출될 때마다 디스패치 모델을 기준으로 수신 사용자 메시지를 검사합니다. 그런 다음, 디스패치 모델의 `topIntent` 및 `recognizerResult`를 올바른 메서드에 전달하여 서비스를 호출하고 결과를 반환합니다.

**DispatchBot.cs** [!code-csharp[OnMessageActivity](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=26-36)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**dispatchBot.js** `onMessage` 메서드에서 디스패치 모델을 기준으로 사용자 입력 메시지를 검사하고, _topIntent_를 확인한 후, _dispatchToTopIntentAsync_를 호출하여 이를 전달합니다.

**dispatchBot.js**

[!code-javascript[OnMessageActivity](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=37-50)]

---

### <a name="work-with-the-recognition-results"></a>인식 결과 작업

## <a name="ctabcs"></a>[C#](#tab/cs)

모델에서 생성하는 결과에 발언을 가장 적절하게 처리할 수 있는 서비스가 나타납니다. 이 봇의 코드는 해당 서비스에 요청을 라우팅한 후 호출된 서비스에의 응답을 요약합니다. 이 코드는 디스패치에서 반환된 _의도_에 따라 올바른 LUIS 모델 또는 QnA 서비스로 라우팅합니다.

**DispatchBot.cs** [!code-csharp[DispatchToTop](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=51-69)]

`ProcessHomeAutomationAsync` 또는 `ProcessWeatherAsync` 메서드가 호출되면 _luisResult.ConnectedServiceResult_ 내에 디스패치 모델의 결과가 전달됩니다. 그러면 지정된 메서드가 디스패치 모델의 상위 의도와 검색된 모든 의도 및 엔터티의 순위 목록을 보여주는 사용자 피드백을 제공합니다.

`q_sample-qna` 메서드가 호출되면 turnContext 내에 포함된 사용자 입력을 사용하여 기술 자료의 답변을 생성하고 해당 결과를 사용자에게 표시합니다.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

모델에서 생성하는 결과에 발언을 가장 적절하게 처리할 수 있는 서비스가 나타납니다. 이 샘플의 코드는 인식된 _topIntent_를 사용하여 요청을 해당 서비스로 라우팅하는 방법을 보여줍니다.

**DispatchBot.cs** [!code-javascript[DispatchToTop](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=67-83)]

`processHomeAutomation` 또는 `processWeather` 메서드가 호출되면 _recognizerResult.luisResult_ 내에 디스패치 모델의 결과가 전달됩니다. 그러면 지정된 메서드가 디스패치 모델의 상위 의도와 검색된 모든 의도 및 엔터티의 순위 목록을 보여주는 사용자 피드백을 제공합니다.

`q_sample-qna` 메서드가 호출되면 turnContext 내에 포함된 사용자 입력을 사용하여 기술 자료의 답변을 생성하고 해당 결과를 사용자에게 표시합니다.

---

> [!NOTE]
> 프로덕션 애플리케이션일 경우 선택된 LUIS 메서드가 지정된 서비스에 연결하고, 사용자 입력을 전달하고, 반환된 LUIS 의도 및 엔터티 데이터를 처리합니다.

## <a name="test-your-bot"></a>봇 테스트

개발 환경을 사용하여 샘플 코드를 시작합니다. 앱이 연 브라우저 창의 주소 표시줄에 표시된 로컬 호스트 주소("https://localhost:<Port_Number>")를 메모합니다. Bot Framework Emulator를 연 후 `create new bot configuration` 아래에 요약된 파란색 테스트를 선택합니다.

![새 구성 만들기](./media/tutorial-dispatch/emulator-create-new-configuration.png)

메모해 둔 로컬 호스트 주소를 입력하고 끝 부분에 '/api/messages'를 추가합니다("https://localhost:<Port_Number>/api/messages").

![에뮬레이터 연결](./media/tutorial-dispatch/emulator-create-and-connect.png)

이제 `Save and connect` 단추를 클릭하여 실행 중인 봇에 액세스합니다. 참고로, 봇용 서비스에서 다루는 몇 가지 질문과 명령은 다음과 같습니다.

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

## <a name="additional-information"></a>추가 정보

봇이 실행되면 유사하거나 겹치는 발화를 제거하여 봇의 성능을 개선할 수 있습니다. 예를 들어 `Home Automation` LUIS 앱에서 QnA maker에 전달될 수 있도록 "TurnOnLights" 의도에 매핑하는 "내 전등 켜기"와 같은 요청이지만 "None" 의도에 매핑하는 "내 전등이 켜지지 않는 이유는?"과 같은 요청을 가정해 봅시다. 디스패치를 사용하여 LUIS 앱 및 QnA Maker 서비스를 결합할 때 다음 중 하나를 수행해야 합니다.

- 원래 `Home Automation` LUIS 앱에서 "None" 의도를 제거하고 해당 의도의 발언을 디스패처 앱의 "None" 의도에 추가합니다.
- 원래 LUIS 앱에서 "None" 의도를 제거하지 않는 경우 "None" 의도를 QnA maker 서비스에 일치시키는 메시지를 전달하도록 봇에 논리를 추가해야 합니다.

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
1. _삭제_를 클릭하고 _확인_을 클릭하여 확인합니다.

QnA Maker 리소스를 삭제하려면:

1. [qnamaker.ai](https://www.qnamaker.ai/) 포털에 로그인합니다.
1. _내 기술 자료_ 페이지로 이동합니다.
1. `Sample QnA` 기술 자료의 삭제 단추를 클릭하고 _삭제_를 클릭하여 확인합니다.

### <a name="best-practice"></a>모범 사례

이 샘플에서 사용된 서비스를 향상시키려면 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) 및 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices)에 대한 모범 사례를 참조하세요.


[howto-luis]: bot-builder-howto-v4-luis.md
[howto-qna]: bot-builder-howto-qna.md

[cs-sample]: https://aka.ms/dispatch-sample-cs
[js-sample]: https://aka.ms/dispatch-sample-js

[dispatch-readme]: https://aka.ms/botbuilder-tools-dispatch
