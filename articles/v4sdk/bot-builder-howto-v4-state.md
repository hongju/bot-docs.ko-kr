---
title: 사용자 및 대화 데이터 저장 | Microsoft Docs
description: Bot Framework SDK를 사용하여 상태 데이터를 저장 및 검색하는 방법을 알아봅니다.
keywords: 대화 상태, 사용자 상태, 대화, 상태 저장, 봇 상태 관리
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3e844a254f1aee709d7dd0b0866ee4ae14737f65
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693719"
---
# <a name="save-user-and-conversation-data"></a>사용자 및 대화 데이터 저장

[!INCLUDE[applies-to](../includes/applies-to.md)]

봇은 기본적으로 상태 비저장입니다. 일단 봇이 배포되면 동일한 프로세스 또는 동일한 머신에서 한 턴에서 다음 턴으로 실행되지 않을 수 있습니다. 그러나 봇은 대화의 컨텍스트를 추적하여 해당 동작을 관리하고 이전 질문에 대한 답변을 기억해야 할 수도 있습니다. Bot Framework SDK의 상태 및 스토리지 기능을 사용하면 상태를 봇에 추가할 수 있습니다. 봇은 상태 관리 및 저장소 개체를 사용하여 상태를 관리하고 유지합니다. 상태 관리자는 기본 스토리지의 유형에 관계없이 속성 접근자를 사용하여 상태 속성에 액세스할 수 있는 추상화 레이어를 제공합니다.

## <a name="prerequisites"></a>필수 조건
- [봇 기본 사항](bot-builder-basics.md) 및 봇에서 [상태를 관리하는 방법](bot-builder-concept-state.md)에 대한 지식이 필요합니다.
- 이 문서의 코드는 **상태 관리 봇 샘플**을 기반으로 합니다. [CSharp](https://aka.ms/statebot-sample-cs) 또는 [JavaScript](https://aka.ms/statebot-sample-js)로 작성된 샘플의 복사본이 필요합니다.

## <a name="about-this-sample"></a>이 샘플 정보
사용자 입력을 수신할 때 이 샘플은 저장된 대화 상태를 확인하여 이 사용자에게 해당 사용자의 이름을 제공하라는 프롬프트를 표시했는지 확인합니다. 프롬프트를 표시하지 않은 경우 사용자의 이름을 요청하며 해당 입력은 사용자 상태 내에 저장됩니다. 프롬프트를 표시한 경우 사용자 상태 내에 저장된 이름을 사용하여 사용자와 대화하며 사용자의 입력 데이터를 수신된 시간 및 입력 채널 ID와 함께 사용자에게 다시 반환합니다. 시간 및 채널 ID 값은 사용자 대화 데이터에서 검색된 후 대화 상태에 저장됩니다. 다음 다이어그램은 봇, 사용자 프로필 및 대화 데이터 클래스 간의 관계를 보여줍니다.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
![상태 봇 샘플](media/StateBotSample-Overview.png)

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
![상태 봇 샘플](media/StateBotSample-JS-Overview.png)

---

## <a name="define-classes"></a>클래스 정의

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

상태 관리를 설정하는 첫 번째 단계는 사용자 및 대화 상태에서 관리하려는 모든 정보를 포함하는 클래스를 정의하는 것입니다. 이 샘플에서는 다음 클래스를 정의했습니다.

- **UserProfile.cs**에서는 봇이 수집할 사용자 정보에 대한 `UserProfile` 클래스를 정의합니다. 
- **ConversationData.cs**에서는 사용자 정보를 수집하는 동안 대화 상태를 제어하기 위한 `ConversationData` 클래스를 정의합니다.

다음 코드 예제에서는 UserProfile 클래스에 대한 정의를 만드는 과정을 보여줍니다.

**UserProfile.cs**  
[!code-csharp[UserProfile](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/UserProfile.cs?range=7-11)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

첫 번째 단계에서는 `UserState` 및 `ConversationState`에 대한 정의를 포함하는 botbuilder 서비스를 요구합니다.

**index.js**  
[!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=7-9)]

---

## <a name="create-conversation-and-user-state-objects"></a>대화 및 사용자 상태 개체 만들기

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

다음으로, `UserState` 및 `ConversationState` 개체를 만드는 데 사용한 `MemoryStorage`를 등록합니다. 사용자 및 대화 상태 개체는 `Startup`에 만들어지며 종속성은 봇 생성자에 주입됩니다. 등록된 봇에 대한 다른 서비스는 자격 증명 공급자, 어댑터 및 봇 구현입니다.

**Startup.cs**  
[!code-csharp[ConfigureServices](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Startup.cs?range=16-38)]

**StateManagementBot.cs**  
[!code-csharp[StateManagement](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=15-22)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

다음으로, `UserState` 및 `ConversationState` 개체를 만드는 데 사용한 `MemoryStorage`를 등록합니다.

**index.js**  
[!code-javascript[DefineMemoryStore](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=32-38)]

---

## <a name="add-state-property-accessors"></a>상태 속성 접근자 추가

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

이제 `BotState` 개체에 대한 핸들을 제공하는 `CreateProperty` 메서드를 사용하여 속성 접근자를 만듭니다. 각 상태 속성 접근자를 사용하면 연결된 상태 속성의 값을 가져오거나 설정할 수 있습니다. 상태 속성을 사용하기 전에 각 접근자를 사용하여 스토리지에서 속성을 로드하고 상태 캐시에서 이를 가져옵니다. 상태 속성과 연결된 올바른 범위의 키를 가져오기 위해 `GetAsync` 메서드를 호출합니다.

**StateManagementBot.cs**  
[!code-csharp[StateAccessors](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=38-46)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

이제 `UserState` 및 `ConversationState`에 대한 속성 접근자를 만듭니다. 각 상태 속성 접근자를 사용하면 연결된 상태 속성의 값을 가져오거나 설정할 수 있습니다. 각 접근자를 사용하여 상태에서 연결된 속성을 로드하고 캐시에서 해당 현재 상태를 검색합니다.

**StateManagementBot.js**.  
[!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=6-19)]

---

## <a name="access-state-from-your-bot"></a>봇에서 상태 액세스
앞의 섹션에서는 상태 속성 접근자를 봇에 추가하는 초기화 시간 단계에 대해 다루고 있습니다. 이제 런타임에 이러한 접근자를 사용하여 상태 정보를 읽고 쓸 수 있습니다. 아래 샘플 코드는 다음 논리 흐름을 사용합니다.
- userProfile.Name이 비어 있고 conversationData.PromptedUserForName이 _True_이면 제공된 사용자 이름을 검색하고 이를 사용자 상태 내에 저장합니다.
- userProfile.Name이 비어 있고 conversationData.PromptedUserForName이 _False_이면 사용자의 이름을 요청합니다.
- userProfile.Name이 이전에 저장된 경우 사용자 입력에서 메시지 시간 및 채널 ID를 검색하고 모든 데이터를 사용자에게 다시 보여주고 검색된 데이터를 대화 상태 내에 저장합니다.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**StateManagementBot.cs**  
[!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=38-85)]

턴 처리기를 종료하기 전에 상태 관리 개체의 _SaveChangesAsync()_ 메서드를 사용하여 모든 상태 변경 내용을 스토리지에 다시 씁니다.

**StateManagementBot.cs**  
[!code-csharp[OnTurnAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=24-31)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**StateManagementBot.js**  
[!code-javascript[OnMessage](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=21-54)]

각 대화 상자 턴을 종료하기 전에 상태 관리 개체의 _saveChanges()_ 메서드를 호출하여 상태를 스토리지에 다시 써서 모든 변경 내용을 유지합니다.

**StateManagementBot.js**  
[!code-javascript[OnDialog](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=60-67)]

---

## <a name="test-the-bot"></a>봇 테스트

최신 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)를 다운로드하고 설치합니다.

1. 샘플을 머신에서 로컬로 실행합니다. 지침이 필요한 경우 [C# 샘플](https://aka.ms/statebot-sample-cs) 또는 [JS 샘플](https://aka.ms/statebot-sample-js)에 대한 추가 정보 파일을 참조하세요.
1. 아래와 같이 에뮬레이터를 사용하여 봇을 테스트합니다.

![상태 봇 샘플 테스트](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>추가 리소스

**개인 정보:** 사용자의 개인 데이터를 저장하려면 [GDPR(일반 데이터 보호 규정)](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr)을 준수해야 합니다.

**상태 관리:** 상태 관리 호출은 모두 비동기적이며, 기본적으로 최종 작성자 인정(last-writer-wins)입니다. 실제로 봇의 상태를 최대한 가깝게 가져오고, 설정하고, 저장해야 합니다.

**중요 비즈니스 데이터:** 봇 상태를 사용하여 기본 설정, 사용자 이름 또는 요청한 마지막 항목을 저장하지만, 중요 비즈니스 데이터를 저장하는 데는 사용하지 않습니다. 중요한 데이터의 경우 [사용자 고유의 스토리지 구성 요소를 만들거나](bot-builder-custom-storage.md) [스토리지](bot-builder-howto-v4-storage.md)에 직접 씁니다.

**Recognizer-Text:** 샘플에서는 Microsoft/Recognizers-Text 라이브러리를 사용하여 사용자 입력을 구문 분석하고 유효성을 검사합니다. 자세한 내용은 [개요](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview) 페이지를 참조하세요.

## <a name="next-steps"></a>다음 단계

이제 스토리지에서 봇 데이터를 읽고 쓰는 데 도움이 되는 상태를 구성하는 방법을 알아보았으므로, 사용자에게 일련의 질문을 하고 답변을 확인하고 입력을 저장하는 방법을 알아보겠습니다.

> [!div class="nextstepaction"]
> [사용자 입력을 수집하도록 고유한 메시지 만들기](bot-builder-primitive-prompts.md).
