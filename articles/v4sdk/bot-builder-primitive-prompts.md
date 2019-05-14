---
title: 사용자 입력을 수집하도록 고유한 메시지 만들기 | Microsoft Docs
description: Bot Framework SDK에서 기본 프롬프트를 사용하여 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 대화 흐름, 프롬프트, 대화 상태, 사용자 상태, 사용자 지정 프롬프트
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/25/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 74c3af688a8f35b4583aa7b195348a6b205292a2
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033170"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>사용자 입력을 수집하도록 고유한 메시지 만들기

[!INCLUDE[applies-to](../includes/applies-to.md)]

봇과 사용자 간의 대화에는 종종 사용자에게 정보를 요청하고(프롬프트), 사용자의 응답을 구문 분석한 다음, 해당 정보에 대한 작업을 수행하는 것이 포함됩니다. 봇은 대화의 컨텍스트를 추적하여 해당 동작을 관리하고 이전의 질문에 대한 답변을 기억할 수 있어야 합니다. 봇의 *상태*는 들어오는 메시지에 적절하게 응답하기 위해 추적하는 정보입니다. 

> [!TIP]
> 대화 상자 라이브러리는 사용자가 사용할 수 있는 더 많은 기능을 제공하는 기본 제공 프롬프트를 제공합니다. 이러한 프롬프트의 예는 [순차적 대화 흐름 구현](bot-builder-dialog-manage-conversation-flow.md) 문서에서 확인할 수 있습니다.

## <a name="prerequisites"></a>필수 조건

- 이 문서의 코드는 사용자 입력 프롬프트 샘플을 기반으로 합니다. **[C# 샘플](https://aka.ms/cs-primitive-prompt-sample) 또는 [Javascript 샘플](https://aka.ms/js-primitive-prompt-sample)** 의 복사본이 필요합니다.
- [관리 상태](bot-builder-concept-state.md) 및 [사용자 및 대화 데이터를 저장](bot-builder-howto-v4-state.md)하는 방법에 대한 지식이 필요합니다.

## <a name="about-the-sample-code"></a>샘플 코드 정보

샘플 봇은 사용자에게 일련의 질문을 하고, 몇 가지 답변의 유효성을 검사하며, 입력을 저장합니다. 다음 다이어그램은 봇, 사용자 프로필 및 대화 흐름 클래스 간의 관계를 보여줍니다. 

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
![custom-prompts](media/CustomPromptBotSample-Overview.png)

- 봇에서 수집할 사용자 정보에 대한 `UserProfile` 클래스입니다.
- 사용자 정보를 수집하는 동안 이 대화 상태를 제어하는 `ConversationFlow` 클래스입니다.
- 대화 중인 위치를 추적하는 `ConversationFlow.Question` 내부 열거형입니다.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
![custom-prompts](media/CustomPromptBotSample-JS-Overview.png)

- 봇에서 수집할 사용자 정보에 대한 `userProfile` 클래스입니다.
- 사용자 정보를 수집하는 동안 이 대화 상태를 제어하는 `conversationFlow` 클래스입니다.
- 대화 중인 위치를 추적하는 `conversationFlow.question` 내부 열거형입니다.

---

사용자 상태에서는 이름, 나이, 선택한 날짜를 추적하고 대화 상태에서는 사용자에게 질문한 내용을 추적합니다.
이 봇은 배포하지 않으므로 _메모리 스토리지_를 사용하도록 사용자 및 대화 상태를 모두 구성합니다. 

봇의 메시지 턴 처리기, 사용자 및 대화 상태 속성을 사용하여 대화 흐름과 입력 수집을 관리합니다. 이 봇에서는 메시지 턴 처리기의 각 반복 중에 받은 상태 속성 정보를 기록할 것입니다.

## <a name="create-conversation-and-user-objects"></a>대화 및 사용자 개체 만들기

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

사용자 및 대화 상태 개체는 시작 시 만들어지며 종속성이 봇 생성자에 주입됩니다. 

**Startup.cs** [!code-csharp[Startup](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Startup.cs?range=27-34)]

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=21-28)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**에서 상태 속성과 봇을 만든 다음, `run` 봇 메서드를 `processActivity` 안에서 호출합니다.

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/index.js?range=32-35)]

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/index.js?range=55-58)]

---

## <a name="create-property-accessors"></a>속성 접근자 만들기

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

`OnMessageActivityAsync` 메서드 내부의 `BotState` 안에 핸들을 제공하는 속성 접근자부터 만듭니다. 그런 다음, `GetAsync` 메서드를 호출하여 올바르게 범위가 지정된 키를 가져옵니다.

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-37)]

마지막으로 `SaveChangesAsync` 메서드를 사용하여 데이터를 저장합니다.

[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=42-43)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

생성자에서 상태 속성 접근자를 만들고, 대화를 위한 상태 관리 개체(위에서 만듦)를 설정합니다.

**bots/customPromptBot.js** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=23-29)]

그런 다음, 기본 메시지 처리기 이후에 실행할 두 번째 처리기 `onDialog`(다음 섹션에서 설명)를 정의합니다. 이 두 번째 처리기는 턴마다 상태를 저장했는지 확인합니다.

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=41-48)]

---

## <a name="the-bots-message-turn-handler"></a>봇의 메시지 턴 처리기

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

메시지 활동을 처리하기 위해, _SaveChangesAsync()_ 를 사용하여 상태를 저장하기 전에 도우미 메서드 _FillOutUserProfileAsync()_ 를 사용합니다. 다음은 완료된 코드입니다.

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-44)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

메시지 활동을 처리하기 위해, 대화와 사용자 데이터를 설정한 다음, 도우미 메서드 `fillOutUserProfile()`을 사용합니다. 턴 처리기에 대한 전체 코드는 다음과 같습니다.

**bots/customPromptBot.js** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=31-39)]
---

## <a name="filling-out-the-user-profile"></a>사용자 프로필 작성

먼저 정보를 수집하여 시작하겠습니다. 각각은 비슷한 인터페이스를 제공합니다.

- 반환 값은 입력이 이 질문에 대한 유효한 답변인지 여부를 나타냅니다.
- 유효성 검사를 통과하면 구문 분석되고 정규화된 값이 생성되어 저장됩니다.
- 유효성 검사가 실패하면 봇에서 해당 정보를 다시 요청할 수 있는 메시지를 생성합니다.

 다음 섹션에서는 사용자 입력을 구문 분석하고 유효성을 검사하는 도우미 메서드를 정의합니다.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=46-103)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.js** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=52-116)]

---

## <a name="parse-and-validate-input"></a>입력 구문 분석 및 유효성 검사

다음 조건을 사용하여 입력의 유효성을 검사합니다.

- **name**(이름)은 비어 있지 않은 문자열이어야 합니다. 공백을 잘라서 정규화합니다.
- **age**(나이)는 18~120여야 합니다. 정수를 반환하여 정규화합니다.
- **date**(날짜)는 앞으로 한 시간 이내의 날짜 또는 시간이어야 합니다.
  구문 분석된 입력의 날짜 부분만 반환하여 정규화합니다.

> [!NOTE]
> age 및 date 입력의 경우 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) 라이브러리를 사용하여 초기 구문 분석을 수행합니다.
> 샘플 코드를 제공하지만 텍스트 인식기 라이브러리의 작동 방식을 설명하지 않으며, 이는 입력을 구문 분석하는 한 가지 방법일 뿐입니다.
> 이러한 라이브러리에 대한 자세한 내용은 리포지토리의 **README**를 참조하세요.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

다음 유효성 검사 메서드를 봇에 추가합니다.

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=105-203)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.cs** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=118-189)]

---

## <a name="test-the-bot-locally"></a>로컬로 봇 테스트
봇을 로컬로 테스트하기 위해 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)를 다운로드하여 설치합니다.

1. 샘플을 머신에서 로컬로 실행합니다. 지침이 필요한 경우 [C# 샘플](https://aka.ms/cs-primitive-prompt-sample) 또는 [JS 샘플](https://aka.ms/js-primitive-prompt-sample)에 대한 추가 정보 파일을 참조하세요.
1. 아래와 같이 에뮬레이터를 사용하여 테스트합니다.

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>추가 리소스

[대화 라이브러리](bot-builder-concept-dialog.md)는 대화 관리의 다양한 측면을 자동화하는 클래스를 제공합니다. 

## <a name="next-step"></a>다음 단계

> [!div class="nextstepaction"]
> [순차적 대화 흐름 구현](bot-builder-dialog-manage-conversation-flow.md)
