---
title: 사용자에게 환영 메시지 보내기 | Microsoft Docs
description: 봇이 환영하는 사용자 환경을 제공하도록 개발하는 방법을 알아봅니다.
keywords: 개요, 개발, 사용자 환경, 환영, 개인 설정 환경, C#, JS, 환영 메시지, 봇, 인사, 인사말
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 24009289f65d214663693aa9221f9e076ba93bc2
ms.sourcegitcommit: d29d3d7ccef401aa1e84e19e623db33b5ff13e63
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2019
ms.locfileid: "67160625"
---
# <a name="send-welcome-message-to-users"></a>사용자에게 환영 메시지 보내기

[!INCLUDE[applies-to](../includes/applies-to.md)]

봇을 만들 때 가장 중요한 목표는 사용자를 의미 있는 대화에 참여시키는 것입니다. 이 목표를 달성하는 가장 좋은 방법 중 하나는 사용자가 처음 연결하는 순간부터 봇의 주요 목적과 기능, 즉 봇이 만들어진 이유를 이해하는 것입니다. 이 문서에서는 봇의 사용자를 환영하는 데 유용한 코드 예제를 제공합니다.

## <a name="prerequisites"></a>필수 조건
- [봇 기본 사항](bot-builder-basics.md)을 이해합니다. 
- [C# 샘플](https://aka.ms/welcome-user-mvc) 또는 [JS 샘플](https://aka.ms/bot-welcome-sample-js)의 **사용자 환영 샘플** 복사본입니다. 샘플의 코드는 환영 메시지를 보내는 방법을 설명하는 데 사용됩니다.

## <a name="about-this-sample-code"></a>이 샘플 코드 정보
이 샘플 코드는 새로운 사용자가 봇에 처음 연결되었을 때 이를 감지하고 환영 메시지를 표시하는 방법을 보여줍니다. 다음 다이어그램은 이 봇의 논리적 흐름을 보여줍니다. 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
봇에서 발생하는 두 가지 주요 이벤트는 다음과 같습니다.
- `OnMembersAddedAsync` - 봇에 새 사용자가 연결될 때마다 호출됩니다.
- `OnMessageActivityAsync` - 새 사용자 입력이 수신될 때마다 호출됩니다.

![사용자 환영 논리 흐름](media/welcome-user-flow.png)

새 사용자가 연결될 때마다 봇이 사용자에게 `WelcomeMessage`, `InfoMessage`, `PatternMessage`를 표시합니다. 새 사용자 입력이 수신되면 봇이 WelcomeUserState의 `DidBotWelcomeUser`가 _true_ 로 설정되어 있는지 확인합니다. 그렇지 않을 경우 초기 사용자 환영 메시지가 사용자에게 반환됩니다.

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
봇에서 발생하는 두 가지 주요 이벤트는 다음과 같습니다.
- `onMembersAdded` - 봇에 새 사용자가 연결될 때마다 호출됩니다.
- `onMessage` - 새 사용자 입력이 수신될 때마다 호출됩니다.

![사용자 환영 논리 흐름](media/welcome-user-flow-js.png)

새 사용자가 연결될 때마다 봇이 사용자에게 `welcomeMessage`, `infoMessage`, `patternMessage`를 표시합니다. 새 사용자 입력이 수신되면 봇이 `welcomedUserProperty`의 `didBotWelcomeUser`가 _true_ 로 설정되어 있는지 확인합니다. 그렇지 않을 경우 초기 사용자 환영 메시지가 사용자에게 반환됩니다.

---

 DidBotWelcomeUser가 _true_ 일 경우 사용자 입력이 평가됩니다. 이 봇은 사용자 입력의 내용에 따라 다음 중 하나를 수행합니다.
- 사용자로부터 수신한 인사말을 그대로 출력합니다.
- 봇에 대한 추가 정보를 제공하는 영웅 카드를 표시합니다.
- 이 봇의 올바른 입력을 설명하는 `WelcomeMessage`를 다시 보냅니다.

## <a name="create-user-object"></a>사용자 개체 만들기
### <a name="ctabcsharp"></a>[C#](#tab/csharp)
사용자 상태 개체는 시작 시 만들어지며 종속성이 봇 생성자에 주입됩니다.

**Startup.cs**  
[!code-csharp[ConfigureServices](~/../botBuilder-samples/samples/csharp_dotnetcore/03.welcome-user/Startup.cs?range=29-33)]

**WelcomeUserBot.cs**  
[!code-csharp[Class](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=41-47)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
시작 시 메모리 스토리지 및 사용자 상태가 모두 index.js에 정의됩니다.

**Index.js**  
[!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/Index.js?range=8-10,33-41)]

---

## <a name="create-property-accessors"></a>속성 접근자 만들기
### <a name="ctabcsharp"></a>[C#](#tab/csharp)
이제 OnMessageActivityAsync 메서드 내에서 WelcomeUserState에 대한 핸들을 제공하는 속성 접근자를 만듭니다.
그런 다음, GetAsync 메서드를 호출하여 올바르게 범위가 지정된 키를 가져옵니다. 그리고 `SaveChangesAsync` 메서드를 사용한 각 사용자 입력 반복 후 사용자 상태 데이터를 저장합니다.

**WelcomeUserBot.cs**  
[!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-71, 102-105)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
이제 UserState 내에 지속되는 WelcomedUserProperty에 대한 핸들을 제공하는 속성 접근자를 만듭니다.

**WelcomeBot.js**  
[!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=7-10,16-22)]

---

## <a name="detect-and-greet-newly-connected-users"></a>새로 연결된 사용자 감지 및 인사말 표시

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
**WelcomeUserBot**에서 `OnMembersAddedAsync()`를 통해 활동 업데이트를 확인하여 새 사용자가 대화에 추가되었는지 본 다음, 일련의 초기 환영 메시지 3개(`WelcomeMessage`, `InfoMessage`, `PatternMessage`)를 전송합니다. 이 상호 작용의 전체 코드는 다음과 같습니다.

**WelcomeUserBot.cs**  
[!code-csharp[WelcomeMessages](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=20-40, 55-66)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
이 JavaScript 코드는 사용자가 추가되면 초기 환영 메시지를 보냅니다. 이는 대화 활동을 확인하고 대화에 새 멤버가 추가되었는지 확인하는 방식으로 수행됩니다.

**WelcomeBot.js**  
[!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=65-87)]

---

## <a name="welcome-new-user-and-discard-initial-input"></a>새 사용자를 환영하고 초기 입력 무시

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
사용자 입력에 실제로 유용한 정보가 포함될 수 있는 시점을 고려하는 것도 중요하며, 이 또한 채널별로 다를 수 있습니다. 사용자가 모든 가능한 채널에 대해 우수한 환경을 갖출 수 있도록 _didBotWelcomeUser_ 상태 플래그를 확인하여 "false"인 경우 이 초기 사용자 입력을 처리하지 않습니다. 대신 사용자에게 초기 환영 메시지를 제공합니다. 그런 다음, _welcomedUserProperty_ 부울이 "true"로 설정되고, UserState에 저장되며, 이제 코드에서 모든 추가 메시지 작업의 이 사용자 입력을 처리합니다.

**WelcomeUserBot.cs**  
[!code-csharp[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-82)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
사용자 입력에 실제로 유용한 정보가 포함될 수 있는 시점을 고려하는 것도 중요하며, 이 또한 채널별로 다를 수 있습니다. 사용자가 모든 가능한 채널에서 높은 만족도를 느낄 수 있도록 didBotWelcomedUser 속성을 확인하여 이 속성이 없는 경우 "false"로 설정하고 이 초기 사용자 입력을 처리하지 않습니다. 대신 사용자에게 초기 환영 메시지를 제공합니다. 그런 다음, _didBotWelcomeUser_ 부울이 "true"로 설정되고, 코드에서 모든 추가 메시지 작업의 사용자 입력을 처리합니다.

**WelcomeBot.js**  
[!code-javascript[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=24-38,57-59,63)]

---

## <a name="process-additional-input"></a>추가 입력 처리

새 사용자에게 환영 메시지가 표시된 후에는 각 메시지 순서의 사용자 입력 정보가 평가되고, 봇이 해당 사용자 입력의 컨텍스트에 따라 응답을 제공합니다. 다음 코드는 해당 응답을 생성하는 데 사용되는 결정 논리를 보여줍니다. 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
'intro' 또는 'help'를 입력하면 `SendIntroCardAsync` 함수가 호출되고 사용자에게 정보 제공용 영웅 카드가 제공됩니다. 해당 코드는 이 문서의 다음 섹션에 나옵니다.

**WelcomeUserBot.cs**  
[!code-csharp[SwitchOnUtterance](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=85-100)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
'intro' 또는 'help'를 입력하면 CardFactory가 사용되고 사용자에게 시작 적응형 카드가 제공됩니다. 해당 코드는 이 문서의 다음 섹션에 나옵니다.

**WelcomeBot.js**  
[!code-javascript[SwitchOnUtterance](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=40-56)]

---

## <a name="using-hero-card-greeting"></a>영웅 카드 인사말 사용

앞서 말했듯이, 일부 사용자 입력은 요청에 대한 응답으로 _영웅 카드_ 를 생성합니다. [시작 카드 보내기](./bot-builder-howto-add-media-attachments.md)에서 영웅 카드 인사말에 대해 자세히 알아볼 수 있습니다. 이 봇의 영웅 카드 응답을 만드는 데 필요한 코드는 다음과 같습니다.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**WelcomeUserBot.cs**  
[!code-csharp[SendHeroCardGreeting](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=107-125)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**WelcomeBot.js**  
[!code-javascript[SendIntroCard](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=91-116)]

---

## <a name="test-the-bot"></a>봇 테스트

최신 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)를 다운로드하고 설치합니다.

1. 샘플을 머신에서 로컬로 실행합니다. 지침이 필요한 경우 [C# 샘플](https://aka.ms/welcome-user-mvc) 또는 [JS 샘플](https://aka.ms/bot-welcome-sample-js)에 대한 추가 정보 파일을 참조하세요.
1. 아래와 같이 에뮬레이터를 사용하여 봇을 테스트합니다.

![환영 봇 샘플 테스트](media/welcome-user-emulator-1.png)

영웅 카드 인사말을 테스트합니다.

![환영 봇 카드 테스트](media/welcome-user-emulator-2.png)

## <a name="additional-resources"></a>추가 리소스

[메시지에 미디어 추가](./bot-builder-howto-add-media-attachments.md)에서 다양한 미디어 응답에 대해 자세히 알아봅니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [사용자 입력 수집](bot-builder-prompts.md)
