---
title: 봇에 자연어 해석 추가 | Microsoft Docs
description: 자연어 이해를 위한 LUIS를 Bot Framework SDK와 함께 사용하는 방법을 알아봅니다.
keywords: Language Understanding, LUIS, 의도, 인식기, 엔터티, 미들웨어
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 4/18/19
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 58d1e83b67d0601a9783e175318a50f8c0646bb2
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033529"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>봇에 자연어 해석 추가

[!INCLUDE[applies-to](../includes/applies-to.md)]

사용자가 대화 및 컨텍스트를 통해 의미하는 바를 이해하는 기능을 구현하기가 어려울 수 있지만, 봇에 보다 자연스러운 대화 느낌을 제공할 수 있습니다. LUIS라고도 하는 Language Understanding을 사용하면 이와 같은 기능을 구현하여 봇이 사용자 메시지의 의도를 인식하고, 사용자가 보다 자연스러운 언어를 사용하고, 대화 흐름을 보다 원활하게 안내할 수 있습니다. 이 항목에서는 항공편 예약 애플리케이션에 LUIS를 추가하여 사용자 입력에 포함된 여러 의도 및 엔터티를 인식하는 방법을 안내합니다. 

## <a name="prerequisites"></a>필수 조건
- [LUIS](https://www.luis.ai) 계정
- 이 문서의 코드는 **Core Bot** 샘플을 기반으로 합니다. **[CSharp](https://aka.ms/cs-core-sample) 또는 [JavaScript](https://aka.ms/js-core-sample)** 로 작성된 샘플의 복사본이 필요합니다. 
- [봇 기본 ](bot-builder-basics.md), [자연어 처리](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis) 및 [봇 리소스 관리 처리](bot-file-basics.md)에 대한 지식이 필요합니다.

## <a name="about-this-sample"></a>이 샘플 정보

이 코어 봇 코딩 샘플에서는 공항 항공편 예약 애플리케이션의 예를 보여줍니다. 이는 LUIS 서비스를 사용하여 사용자 입력을 인식하고 상위에 인식된 LUIS 의도를 반환합니다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
사용자 입력 처리가 완료될 때마다 `DialogBot`이 `UserState` 및 `ConversationState`의 현재 상태를 저장합니다. 코딩 샘플은 필요한 모든 정보가 수집된 후 데모 항공편 예약을 만듭니다. 이 문서에서는 이 샘플의 LUIS 측면을 다룹니다. 하지만 샘플의 일반적인 흐름은 다음과 같습니다.

- 새로운 사용자가 연결되면 `OnMembersAddedAsync`가 호출되고 환영 카드가 표시됩니다. 
- 사용자 입력이 수신될 때마다 `OnMessageActivityAsync`가 호출됩니다. 

![LUIS 샘플 논리 흐름](./media/how-to-luis/luis-logic-flow.png)

`OnMessageActivityAsync` 모듈은 `Run` 대화 확장 메서드를 통해 적절한 대화를 실행합니다. 해당 주 대화는 LUIS 도우미를 호출하여 가장 점수가 높은 사용자 의도를 찾습니다. 사용자 입력의 상위 의도가 "Book_Flight"를 반환하면 도우미가 LUIS에서 반환한 사용자의 정보를 채우고 `BookingDialog`를 시작합니다. 이는 필요에 따라 사용자로부터 다음과 같은 추가 정보를 얻습니다.

- `Origin` 출발 도시입니다.
- `TravelDate` 항공편 예약 날짜입니다. 
- `Destination` 도착 도시입니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
사용자 입력 처리가 완료될 때마다 `dialogBot`이 `userState` 및 `conversationState`의 현재 상태를 저장합니다. 코딩 샘플은 필요한 모든 정보가 수집된 후 데모 항공편 예약을 만듭니다. 이 문서에서는 이 샘플의 LUIS 측면을 다룹니다. 하지만 샘플의 일반적인 흐름은 다음과 같습니다.

- 새로운 사용자가 연결되면 `onMembersAdded`가 호출되고 환영 카드가 표시됩니다. 
- 사용자 입력이 수신될 때마다 `OnMessage`가 호출됩니다. 

![LUIS 샘플 javascript 논리 흐름](./media/how-to-luis/luis-logic-flow-js.png)

`onMessage` 모듈은 `mainDialog`를 실행하고, 이는 LUIS로 사용자 입력을 보냅니다. LUIS로부터 응답이 수신되면 `mainDialog`가 LUIS에서 반환된 사용자에 대한 정보를 보존하고 `bookingDialog`를 시작합니다. `bookingDialog`는 필요에 따라 사용자로부터 다음과 같은 정보를 얻습니다.

- `destination` 도착 도시입니다.
- `origin` 출발 도시입니다.
- `travelDate` 항공편 예약 날짜입니다. 

---

대화나 상태와 같은 샘플의 다른 측면에 대한 자세한 내용은 [대화 프롬프트를 사용하여 사용자 입력 수집](bot-builder-prompts.md) 또는 [사용자 및 대화 데이터 저장](bot-builder-howto-v4-state.md)을 참조하세요. 

## <a name="create-a-luis-app-in-the-luis-portal"></a>LUIS 포털에서 LUIS 앱 만들기
LUIS 포털에 로그인하여 사용자 고유 버전의 LUIS 샘플 앱을 만듭니다. 애플리케이션은 **내 앱**에서 만들고 관리할 수 있습니다. 

1. **새 앱 가져오기**를 선택합니다. 
1. **앱 파일 선택(JSON 형식)...** 을 클릭합니다. 
1. 샘플의 `CognitiveModels` 폴더에 있는 `FlightBooking.json` 파일을 선택합니다. **옵션 이름**에서 **FlightBooking**을 입력합니다. 이 파일에는 세 가지 의도 'Book Flight', 'Cancel' 및 'None'이 포함되어 있습니다. 이러한 의도를 사용하여 사용자가 봇에 메시지를 보낼 때 의미하는 바를 이해합니다.
1. 앱을 [학습](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)합니다.
1. 앱을 *프로덕션* 환경에 [게시](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)합니다.

### <a name="why-use-entities"></a>엔터티를 사용하는 이유
LUIS 엔터티를 사용하면 표준 의도와 다른 특정 사물이나 이벤트를 지능적으로 인식할 수 있습니다. 이렇게 하면 사용자로부터 추가 정보를 수집할 수 있으므로 봇이 더 지능적으로 응답하거나 사용자에게 해당 정보를 요청하는 특정 질문을 건너뛸 수 있습니다. FlightBooking.json 파일에는 'Book Flight', 'Cancel' 및 'None'이라는 세 가지 LUIS 의도에 대한 정의와 함께 'From.Airport' 및 'To.Airport'와 같은 엔터티 세트도 포함되어 있습니다. 이러한 엔터티는 LUIS가 새로운 여행 예약 요청 시 사용자의 원래 입력 내에 포함된 추가 정보를 감지하고 반환할 수 있게 해줍니다.

## <a name="obtain-values-to-connect-to-your-luis-app"></a>LUIS 앱에 연결할 값 가져오기
LUIS 앱이 게시되면 봇에서 액세스할 수 있습니다. 봇 내에서 LUIS 앱에 액세스하려면 여러 값을 기록해야 합니다. LUIS 포털을 사용하여 해당 정보를 검색할 수 있습니다.

### <a name="retrieve-application-information-from-the-luisai-portal"></a>LUIS.ai 포털에서 애플리케이션 정보 검색
설정 파일(`appsettings.json` 또는 `.env`)은 모든 서비스 참조를 한 곳에 모을 수 있는 장소입니다. 검색한 정보는 다음 섹션에서 이 파일에 추가됩니다. 
1. [luis.ai](https://www.luis.ai)에서 게시된 LUIS 앱을 선택합니다.
1. 게시된 LUIS 앱이 열리면 **관리** 탭을 선택합니다. ![LUIS 앱 관리](./media/how-to-luis/manage-luis-app.png)
1. 왼쪽에서 **애플리케이션 정보** 탭을 선택하고 _애플리케이션 ID_에 &lt;YOUR_APP_ID&gt;로 표시되는 값을 기록합니다.
1. 왼쪽에서 **키 및 엔드포인트** 탭을 선택하고 _제작 키_에 <YOUR_AUTHORING_KEY>로 표시되는 값을 기록합니다.
1. 페이지 끝까지 아래로 스크롤하여 _지역_에 대해 표시된 값을 <YOUR_REGION>으로 기록합니다.

### <a name="update-the-settings-file"></a>설정 파일 업데이트

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

애플리케이션 ID, 제작 키 및 지역을 포함하여 LUIS 앱에 액세스하는 데 필요한 정보를 `appsettings.json` 파일에 추가합니다. 이는 이전에 게시된 LUIS 앱에서 저장한 값입니다. API 호스트 이름은 `<your region>.api.cognitive.microsoft.com` 형식이어야 합니다.

**appsetting.json** [!code-json[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/appsettings.json?range=1-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

애플리케이션 ID, 제작 키 및 지역을 포함하여 LUIS 앱에 액세스하는 데 필요한 정보를 `.env` 파일에 추가합니다. 이는 이전에 게시된 LUIS 앱에서 저장한 값입니다. API 호스트 이름은 `<your region>.api.cognitive.microsoft.com` 형식이어야 합니다.

**.env**
[!code[env](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/.env?range=1-5)]

---

## <a name="configure-your-bot-to-use-your-luis-app"></a>LUIS 앱을 사용하도록 봇 구성

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

프로젝트에 **Microsoft.Bot.Builder.AI.Luis** NuGet 패키지가 설치되어 있는지 확인합니다.

LUIS 서비스에 연결할 때 봇은 위에 추가된 정보를 appsetting.json 파일에서 끌어옵니다. `LuisHelper` 클래스에는 appsetting.json 파일에서 설정을 가져오고 `RecognizeAsync` 메서드를 호출하여 LUIS 서비스를 쿼리하는 코드가 포함되어 있습니다. 반환되는 상위 의도는 'Book_Flight'이며, 이는 예약의 To, From 및 TravelDate 정보가 포함된 엔터티를 검사합니다.

**LuisHelper.cs** [!code-csharp[luis helper](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/LuisHelper.cs?range=15-54)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

LUIS를 사용하려면 프로젝트에서 **botbuilder-ai** npm 패키지를 설치해야 합니다.

LUIS 서비스에 연결할 때 봇은 위에 추가된 정보를 `.env` 파일에서 끌어옵니다. `LuisHelper` 클래스에는 `.env` 파일에서 설정을 가져오고 `recognize()` 메서드를 호출하여 LUIS 서비스를 쿼리하는 코드가 포함되어 있습니다. 반환되는 상위 의도는 'Book_Flight'이며, 이는 예약의 To, From 및 TravelDate 정보가 포함된 엔터티를 검사합니다.

[!code-javascript[luis helper](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/luisHelper.js?range=6-65)]

---

이제 봇에 LUIS가 구성되고 연결되었습니다. 

## <a name="test-the-bot"></a>봇 테스트

최신 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)를 다운로드하고 설치합니다.

1. 샘플을 머신에서 로컬로 실행합니다. 지침이 필요한 경우 [C# 샘플](https://aka.ms/cs-core-sample) 또는 [JS 샘플](https://aka.ms/js-core-sample)에 대한 추가 정보 파일을 참조하세요.

1. 에뮬레이터에서 "travel to paris" 또는 "going from paris to berlin"과 같은 메시지를 입력합니다. FlightBooking.json 파일에서 검색된 발화를 사용하여 "Book flight" 의도를 학습시킵니다.

![LUIS 예약 입력](./media/how-to-luis/luis-user-travel-input.png)

LUIS에서 반환된 상위 의도가 "Book flight"로 확인되면 봇이 여행 예약을 만들기에 충분한 정보가 저장될 때까지 추가 질문을 묻습니다. 충분한 정보가 수집되면 봇이 사용자에게 이러한 예약 정보를 다시 반환합니다. 

![LUIS 예약 결과](./media/how-to-luis/luis-travel-result.png)

이제 코드 봇 논리가 다시 설정되고, 추가 예약을 계속 만들 수 있습니다. 

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [QnA Maker를 사용하여 질문에 답변](./bot-builder-howto-qna.md)
