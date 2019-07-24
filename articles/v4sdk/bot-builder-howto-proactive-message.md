---
title: 사용자에게 자동 관리 알림 보내기 | Microsoft Docs
description: 알림 메시지를 보내는 방법 이해
keywords: 자동 관리 메시지, 알림 메시지, 봇 알림
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9389f97cbba2e8766bf29b2502d36e9ec03077cf
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215379"
---
# <a name="send-proactive-notifications-to-users"></a>사용자에게 자동 관리 알림 보내기

[!INCLUDE[applies-to](../includes/applies-to.md)]

일반적으로 봇이 사용자에게 직접 보내는 각각의 메시지는 사용자의 이전 입력과 관련이 있습니다.
경우에 따라 봇은 현재 대화 주제 또는 사용자가 보낸 마지막 메시지와 직접 관련이 없는 메시지를 사용자에게 보내야 할 수도 있습니다. 이러한 종류의 메시지를 _자동 관리 메시지_ 라고 합니다.

자동 관리 메시지는 다양한 시나리오에서 유용할 수 있습니다. 예를 들어, 사용자가 이전에 제품 가격을 모니터링할 것을 봇에게 요청한 경우 봇은 제품 가격이 20%까지 떨어질 경우 사용자에게 경고할 수 있습니다. 또는 봇이 사용자 질문에 대한 응답을 컴파일할 시간이 필요한 경우 지연 사실을 알리고, 당분간 대화를 계속 진행할 수 있습니다. 봇이 질문에 대한 응답 컴파일을 완료하면 해당 정보를 사용자와 공유합니다.

봇에서 자동 관리 메시지를 구현하는 경우 짧은 시간 내에 여러 개의 자동 관리 메시지를 보내지 마십시오. 일부 채널은 봇이 사용자에게 메시지를 보낼 수 있는 빈도를 제한하며, 해당 제한을 위반할 경우 봇을 사용하지 않도록 설정합니다.

임시 자동 관리 메시지는 가장 간단한 형식의 자동 관리 메시지입니다. 봇은 메시지가 트리거될 때마다 사용자가 현재 봇과 다른 주제의 대화에 참여하고 있으며 대화를 변경하지 않으려고 하는지 여부에 관계없이 메시지를 간단히 대화에 삽입합니다.

알림을 더 원활하게 처리하려면 대화 상태에 플래그를 설정하거나 알림을 큐에 추가하는 것처럼 알림을 대화 흐름에 통합하는 다른 방법을 사용하는 것이 좋습니다.

## <a name="prerequisites"></a>필수 조건

- [봇 기본 사항](bot-builder-basics.md)을 이해합니다.
- **[CSharp](https://aka.ms/proactive-sample-cs) 또는 [JavaScript](https://aka.ms/proactive-sample-js)** 로 작성된 자동 관리 메시지 샘플의 복사본 이 샘플은 이 문서에서 자동 관리 메시지를 설명하는 데 사용됩니다.

## <a name="about-the-proactive-sample"></a>자동 관리 샘플 정보

샘플에는 봇 및 다음 그림과 같이 봇에 자동 관리 메시지를 보내는 데 사용하는 추가 컨트롤러가 있습니다.

![자동 관리 봇](media/proactive-sample-bot.png)

## <a name="retrieve-and-store-conversation-reference"></a>대화 참조 검색 및 저장

에뮬레이터가 봇에 연결할 때 봇은 두 가지 대화 업데이트 작업을 받습니다. 봇의 대화 업데이트 작업 처리기에서는 아래와 같이 참조를 검색하고 사전에 저장합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\ProactiveBot.cs**

[!code-csharp[OnConversationUpdateActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Bots/ProactiveBot.cs?range=26-37&highlight=3-4,9)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/proactiveBot.js**

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=13-17&highlight=2)]

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=41-44&highlight=2-3)]

---

참고: 실제 시나리오에서는 메모리의 개체를 사용하는 대신 대화 참조를 데이터베이스에 유지할 것입니다.

대화 참조에는 작업이 존재하는 대화를 설명하는 _대화_ 속성이 있습니다. 대화에는 대화에 참여하는 사용자를 나열하는 _사용자_ 속성 및 채널에서 현재 작업에 대한 응답을 보낼 수 있는 URL을 표시하는 _서비스 URL_ 속성이 있습니다. 사용자에게 자동 관리 메시지를 보내려면 유효한 대화 참조가 필요합니다.

## <a name="send-proactive-message"></a>자동 관리 메시지 보내기

두 번째 컨트롤러인 _알림_ 컨트롤러는 봇에 자동 관리 메시지를 보내는 기능을 합니다. 다음 단계를 사용하여 자동 관리 메시지를 생성합니다.

1. 자동 관리 메시지를 보내는 대화에 대한 참조를 검색합니다.
1. 어댑터의 _대화 계속_ 메서드를 호출하여 대화 참조 및 사용할 턴 처리기 대리자를 제공합니다. 대화 계속 메서드는 참조한 대화에 대한 턴 컨텍스트를 생성한 다음, 지정된 턴 처리기 대리자를 호출합니다.
1. 대리자에서는 턴 컨텍스트를 사용하여 자동 관리 메시지를 보냅니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Controllers\NotifyController.cs**

봇의 알림 페이지를 요청받을 때마다 알림 컨트롤러는 사전에서 대화 참조를 검색합니다.
그런 다음, 컨트롤러는 `ContinueConversationAsync` 및 `BotCallback` 메서드를 사용하여 자동 관리 메시지를 보냅니다.

[!code-csharp[Notify logic](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Controllers/NotifyController.cs?range=17-59&highlight=28,40-43)]

어댑터가 자동 관리 메시지를 보내려면 봇에 대한 앱 ID가 필요합니다. 프로덕션 환경에서는 봇의 앱 ID를 사용할 수 있습니다. 로컬 테스트 환경에서는 임의의 GUID를 사용할 수 있습니다. 봇에 현재 앱 ID가 할당되지 않은 경우 알림 컨트롤러가 호출에 사용할 자리 표시자 ID를 자체 생성합니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

서버의 `/api/notify` 페이지를 요청받을 때마다 서버는 사전에서 대화 참조를 검색합니다.
그런 다음, 서버는 `continueConversation` 메서드를 사용하여 자동 관리 메시지를 보냅니다.
`continueConversation`에 대한 매개 변수는 이 턴에 대한 봇의 턴 처리기 역할을 하는 함수입니다.

[!code-javascript[Notify logic](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/index.js?range=56-62&highlight=4-5)]

---

## <a name="test-your-bot"></a>봇 테스트

1. 아직 설치하지 않은 경우 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)를 설치합니다.
1. 샘플을 머신에서 로컬로 실행합니다.
1. 에뮬레이터를 시작하고 봇에 연결합니다.
1. 봇의 api/notify 페이지를 로드합니다. 이렇게 하면 에뮬레이터에서 자동 관리 메시지가 생성됩니다.

## <a name="additional-information"></a>추가 정보

이 문서에 사용한 샘플 외에도 [GitHub](https://github.com/Microsoft/BotBuilder-Samples/)에서는 C# 및 JS로 작성된 추가 샘플을 사용할 수 있습니다.

### <a name="avoiding-401-unauthorized-errors"></a>401 "권한 없음" 오류 방지 

기본적으로 BotBuilder SDK는 들어오는 요청이 BotAuthentication에서 인증된 경우 신뢰할 수 있는 호스트 이름의 목록에 `serviceUrl`을 추가합니다. 이 정보는 메모리 내 캐시에서 유지 관리됩니다. 봇이 다시 시작된 경우 자동 관리 메시지를 기다리는 사용자는 봇이 다시 시작된 후 사용자가 봇에 메시지를 다시 보내지 않는 한 해당 메시지를 검색할 수 없습니다. 

이 문제를 방지하려면 다음을 사용하여 신뢰할 수 있는 호스트 이름의 목록에 `serviceUrl`을 수동으로 추가해야 합니다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp 
MicrosoftAppCredentials.TrustServiceUrl(serviceUrl); 
``` 

자동 관리 메시지의 경우 `serviceUrl`은 자동 관리 메시지의 수신자가 사용 중인 채널의 URL이며 `Activity.ServiceUrl`에서 확인할 수 있습니다. 

위의 코드를 자동 관리 메시지를 보내는 코드 바로 앞에 추가하려고 합니다. 이 샘플에는 해당 코드가 `ProactiveBot.cs`의 `CreateCallback()` 끝 부근에 있지만 `appId` 및 `appPassword`가 없으면 에뮬레이터에서 작동하지 않으므로 주석으로 처리되었습니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```js
MicrosoftAppCredentials.trustServiceUrl(serviceUrl);
```

자동 관리 메시지의 경우 `serviceUrl`은 자동 관리 메시지의 수신자가 사용 중인 채널의 URL이며 `activity.serviceUrl`에서 확인할 수 있습니다.

위의 코드를 자동 관리 메시지를 보내는 코드 바로 앞에 추가하려고 합니다. 이 샘플에는 해당 코드가 `bot.js`의 `completeJob()` 끝 부근에 있지만 `appId` 및 `appPassword`가 없으면 에뮬레이터에서 작동하지 않으므로 주석으로 처리되었습니다.

---

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [순차적 대화 흐름 구현](bot-builder-dialog-manage-conversation-flow.md)
