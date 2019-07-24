---
title: 순차적 대화 흐름 구현 | Microsoft Docs
description: Bot Framework SDK에서 대화를 사용하여 간단한 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 간단한 대화 흐름, 순차적 대화 흐름, 대화 상자, 프롬프트, 폭포, 대화 상자 집합
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c3c116eec8222ce50cd7dde672cc86f9765a3f97
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/05/2019
ms.locfileid: "67587480"
---
# <a name="implement-sequential-conversation-flow"></a>순차적 대화 흐름 구현

[!INCLUDE[applies-to](../includes/applies-to.md)]

질문을 제시하여 정보를 수집하는 것은 봇에서 사용자와 상호 작용하는 주요 방법 중 하나입니다. 대화 상자 라이브러리는 쉽게 질문하고 응답의 유효성을 검사하여 특정 데이터 형식과 일치하거나 사용자 지정 유효성 검사 규칙을 충족하는지 확인할 수 있는 ‘프롬프트’ 클래스 등의 유용한 기본 제공 기능을 제공합니다.  

대화 상자 라이브러리를 사용하여 단순 및 복합 대화 흐름을 관리할 수 있습니다. 간단한 상호 작용에서 봇은 고정된 일련의 단계를 통해 실행되고 대화가 완료됩니다. 일반적으로 대화는 봇이 사용자로부터 정보를 수집해야 할 때 유용합니다. 이 항목에서는 프롬프트를 만들고 폭포 대화에서 호출하는 방식으로 간단한 대화 흐름을 구현하는 방법을 자세히 설명합니다. 

> [!TIP]
> 대화 상자 라이브러리를 사용하지 않고 사용자 고유의 프롬프트를 작성하는 방법의 예제는 [사용자 입력을 수집하기 위해 고유한 프롬프트 만들기](bot-builder-primitive-prompts.md)를 참조하세요. 


## <a name="prerequisites"></a>필수 조건

- [봇 기본 사항][concept-basics], [managing state][concept-state] 및 [대화 상자 라이브러리][concept-dialogs]에 대한 지식
- [**C#**][cs-sample] 또는 [**JavaScript**][js-sample]로 작성된 **다중 순서 프롬프트** 샘플의 복사본

## <a name="about-this-sample"></a>이 샘플 정보

다중 순서 프롬프트 샘플에서는 폭포 대화, 몇 가지 프롬프트 및 구성 요소 대화를 사용하여 사용자에게 일련의 질문을 하는 간단한 상호 작용을 만듭니다. 코드는 대화 상자를 사용하여 다음 단계를 순환합니다.

| 단계        | 프롬프트 형식  |
|:-------------|:-------------| 
| 사용자에게 사용자의 전송 모드 요청 | 선택 항목 프롬프트 |
| 사용자에게 사용자의 이름 요청 | 텍스트 프롬프트 |
| 사용자에게 자신의 나이 제공을 원하는지 여부 질문 | 확인 프롬프트 |
| 사용자가 예로 답하면 나이 요청  | 0보다 크고 150보다 작은 나이만 허용하는 유효성 검사를 포함한 숫자 프롬프트 |
| 수집된 정보가 "올바른지" 질문 | 확인 프롬프트 재사용 |

끝으로, 사용자가 예로 답하면 수집된 정보를 표시합니다. 그렇지 않으면 사용자에게 해당 사용자의 정보가 보관되지 않을 것임을 알려줍니다.

## <a name="create-the-main-dialog"></a>기본 대화 만들기

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

대화를 사용하려면 **Microsoft.Bot.Builder.Dialogs** NuGet 패키지를 설치합니다.

봇은 `UserProfileDialog`를 통해 사용자와 상호 작용합니다. 봇의 `DialogBot` 클래스를 만들 때는 `UserProfileDialog`를 기본 대화로 설정합니다. 그러면 봇이 `Run` 도우미 메서드를 사용하여 대화에 액세스합니다.

![사용자 프로필 대화](media/user-profile-dialog.png)

**Dialogs\UserProfileDialog.cs**

먼저 `ComponentDialog` 클래스에서 파생되며 6단계가 있는 `UserProfileDialog`를 만듭니다.

`UserProfileDialog` 생성자에서 폭포 단계, 프롬프트 및 폭포 대화를 만들어 대화 세트에 추가합니다. 프롬프트는 프롬프트가 사용되는 대화 세트에 있어야 합니다.

[!code-csharp[Constructor snippet](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=22-41)]

그런 다음, 대화가 사용하는 단계를 구현합니다. 프롬프트를 사용하려면 대화의 단계에서 호출하고 다음 단계에서 `stepContext.Result`를 사용하여 프롬프트 결과를 검색합니다. 프롬프트는 내부적으로 두 가지 단계로 수행되는 대화입니다. 먼저 프롬프트에서 입력을 요청하며, 다음으로 유효한 값을 반환하거나 유효한 입력을 수신할 때까지 다시 프롬프트를 사용하여 처음부터 다시 시작합니다.



항상 폭포 단계에서 null이 아닌 `DialogTurnResult`를 반환해야 합니다. 그렇지 않으면 대화가 올바르게 작동하지 않을 수 있습니다. 다음은 폭포 대화의 `NameStepAsync` 구현입니다.

[!code-csharp[Name step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=56-61)]

`AgeStepAsync`에서 프롬프트가 구문 분석할 수 없는 형식이거나 입력이 유효성 검사 조건에 실패하여 사용자 입력의 유효성을 검사하는 데 실패하는 경우를 위해 다시 시도 프롬프트를 지정합니다. 이 경우 다시 시도 프롬프트가 제공되지 않으면 프롬프트에서 초기 프롬프트 텍스트를 사용하여 사용자에게 다시 입력하도록 요청하는 프롬프트를 표시합니다.

[!code-csharp[Age step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=74-93&highlight=10)]

**UserProfile.cs**

사용자의 교통 수단, 이름 및 나이가 `UserProfile` 클래스의 인스턴스에 저장됩니다.

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/UserProfile.cs?range=9-16)]

**Dialogs\UserProfileDialog.cs**

마지막 단계에서는 이전의 폭포 단계에서 호출된 대화가 반환한 `stepContext.Result`를 확인합니다. 반환 값이 true이면 사용자 프로필 접근자를 사용하여 사용자 프로필을 가져오고 업데이트합니다. 사용자 프로필을 가져오기 위해 `GetAsync` 메서드를 호출한 후 `userProfile.Transport`, `userProfile.Name` 및 `userProfile.Age` 속성 값을 설정합니다. 마지막으로, 대화를 종료하는 `EndDialogAsync`를 호출하기 전에 사용자의 정보를 요약합니다. 대화를 종료하면 대화 스택이 사라지고 대화 부모에 선택적 결과가 반환됩니다. 부모는 대화 또는 방금 전에 종료된 대화를 시작한 메서드입니다.

[!code-csharp[SummaryStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=108-134&highlight=5-10,25-26)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

대화를 사용하려면 프로젝트에서 **botbuilder-dialogs** npm 패키지를 설치해야 합니다.

봇은 `UserProfileDialog`를 통해 사용자와 상호 작용합니다. 봇의 `DialogBot`을 만들 때는 `UserProfileDialog`를 기본 대화로 설정합니다. 그러면 봇이 `run` 도우미 메서드를 사용하여 대화에 액세스합니다.

![사용자 프로필 대화](media/user-profile-dialog-js.png)

**dialogs\userProfileDialog.js**

먼저 `ComponentDialog` 클래스에서 파생되며 6단계가 있는 `UserProfileDialog`를 만듭니다.

`UserProfileDialog` 생성자에서 폭포 단계, 프롬프트 및 폭포 대화를 만들어 대화 세트에 추가합니다. 프롬프트는 프롬프트가 사용되는 대화 세트에 있어야 합니다.

[!code-javascript[Constructor snippet](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=25-47)]

그런 다음, 대화가 사용하는 단계를 구현합니다. 프롬프트를 사용하려면 대화의 단계에서 호출하고 단계 컨텍스트의 다음 단계에서 `step.result`를 사용하여 프롬프트 결과를 검색합니다. 프롬프트는 내부적으로 두 가지 단계로 수행되는 대화입니다. 먼저 프롬프트에서 입력을 요청하며, 다음으로 유효한 값을 반환하거나 유효한 입력을 수신할 때까지 다시 프롬프트를 사용하여 처음부터 다시 시작합니다.

항상 폭포 단계에서 null이 아닌 `DialogTurnResult`를 반환해야 합니다. 그렇지 않으면 대화가 올바르게 작동하지 않을 수 있습니다. 다음은 폭포 대화의 `nameStep` 구현입니다.

[!code-javascript[name step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=75-78)]

`ageStep`에서 프롬프트가 구문 분석할 수 없는 형식이거나 입력이 위의 생성자에 지정된 유효성 검사 조건에 실패하여 사용자 입력의 유효성을 검사하는 데 실패하는 경우를 위해 다시 시도 프롬프트를 지정합니다. 이 경우 다시 시도 프롬프트가 제공되지 않으면 프롬프트에서 초기 프롬프트 텍스트를 사용하여 사용자에게 다시 입력하도록 요청하는 프롬프트를 표시합니다.

[!code-javascript[age step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=90-101&highlight=5)]

**userProfile.js**

사용자의 교통 수단, 이름 및 나이가 `UserProfile` 클래스의 인스턴스에 저장됩니다.

[!code-javascript[user profile](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/userProfile.js?range=4-10)]

**dialogs\userProfileDialog.js**

마지막 단계에서는 이전의 폭포 단계에서 호출된 대화가 반환한 `step.result`를 확인합니다. 반환 값이 true이면 사용자 프로필 접근자를 사용하여 사용자 프로필을 가져오고 업데이트합니다. 사용자 프로필을 가져오기 위해 `get` 메서드를 호출한 후 `userProfile.transport`, `userProfile.name` 및 `userProfile.age` 속성 값을 설정합니다. 마지막으로, 대화를 종료하는 `endDialog`를 호출하기 전에 사용자의 정보를 요약합니다. 대화를 종료하면 대화 스택이 사라지고 대화 부모에 선택적 결과가 반환됩니다. 부모는 대화 또는 방금 전에 종료된 대화를 시작한 메서드입니다.

[!code-javascript[summary step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=115-136&highlight=4-8,20-21)]

**폭포 대화를 실행하는 확장 메서드 만들기**

대화 컨텍스트를 만들고 액세스하는 데 사용할 `run` 도우미 메서드를 `userProfileDialog` 내에 정의했습니다. 여기서 `accessor`는 대화 상태 속성의 상태 속성 접근자이며, `this`는 사용자 프로필 구성 요소 대화입니다. 구성 요소 대화는 내부 대화 세트를 정의하므로 메시지 처리기 코드에서 볼 수 있는 외부 대화 세트를 만들어야 하며, 대화 컨텍스트를 만들 때 해당 세트를 사용합니다.

대화 컨텍스트는 `createContext` 메서드를 호출하여 생성되며, 봇의 순서 처리기 내에서 대화 세트와 상호 작용하는 데 사용됩니다. 대화 컨텍스트는 현재 순서 컨텍스트, 부모 대화 및 대화 상태를 포함하여 대화 내에서 정보를 유지하는 메서드를 제공합니다.

대화 컨텍스트를 사용하면 해당 문자열 ID를 통해 대화를 시작하거나 현재 대화(예: 여러 단계가 있는 폭포 대화)를 계속할 수 있습니다. 대화 컨텍스트는 봇의 모든 대화 및 폭포 단계를 통해 전달됩니다.

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=55-64)]

---

## <a name="run-the-dialog"></a>대화 실행

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\DialogBot.cs**

`OnMessageActivityAsync` 처리기는 `RunAsync` 메서드를 사용하여 대화를 시작하거나 계속합니다. `OnTurnAsync`에서는 봇의 상태 관리 개체를 사용하여 스토리지에 대한 모든 상태 변경 사항을 유지합니다. (`ActivityHandler.OnTurnAsync` 메서드는 `OnMessageActivityAsync`와 같은 다양한 작업 처리기 메서드를 호출합니다. 메시지 처리기가 완료된 후 순서 자체가 완료되기 전에 상태를 이러한 방식으로 저장하고 있습니다.)

[!code-csharp[overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`onMessage` 처리기는 도우미 메서드를 사용하여 대화를 시작하거나 계속합니다. `onDialog`에서는 봇의 상태 관리 개체를 사용하여 스토리지에 대한 모든 상태 변경 사항을 유지합니다. (`onDialog` 메서드는 `onMessage`와 같은 정의된 다른 처리기가 실행된 후 마지막으로 호출됩니다. 메시지 처리기가 완료된 후 순서 자체가 완료되기 전에 상태를 이러한 방식으로 저장하고 있습니다.)

[!code-javascript[overrides](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=30-44)]

---

## <a name="register-services-for-the-bot"></a>봇용 서비스 등록

이 봇은 다음과 같은 _서비스_ 를 사용합니다.

- 봇 기본 서비스: 자격 증명 공급자, 어댑터 및 봇 구현
- 상태 관리 서비스: 스토리지, 사용자 상태 및 대화 상태
- 봇이 사용할 대화

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**

여기서는 봇 서비스를 `Startup`에 등록합니다. 해당 서비스는 종속성 주입을 통해 코드의 다른 부분에 사용할 수 있습니다.

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Startup.cs?range=17-41)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

여기서는 봇 서비스를 `index.js`에 등록합니다. 

[!code-javascript[overrides](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/index.js?range=18-49)]

---

> [!NOTE]
> 메모리 저장소는 테스트 목적으로만 사용되며 프로덕션용이 아닙니다.
> 프로덕션 봇에는 영구 스토리지 형식을 사용해야 합니다.

## <a name="to-test-the-bot"></a>봇 테스트

1. 아직 설치하지 않은 경우 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)를 설치합니다.
1. 샘플을 머신에서 로컬로 실행합니다.
1. 에뮬레이터를 시작하고 봇에 연결한 다음, 아래와 같은 메시지를 보냅니다.

![다중 턴 프롬프트 대화 상자의 샘플 실행](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>추가 정보

### <a name="about-dialog-and-bot-state"></a>대화 상자 및 봇 상태 정보

이 봇에서 두 개의 상태 속성 접근자를 정의했습니다.

- 대화 상자 상태 속성에 대해 대화 상태 내에서 만든 접근자입니다. 대화 상자 상태는 대화 상자 집합의 대화 상자 내에서 사용자를 추적합니다. 또한 시작 대화 상자를 호출하거나 대화 상자 메서드를 계속하는 경우 대화 상자 컨텍스트로 업데이트합니다.
- 사용자 프로필 속성에 대한 사용자 상태 내에서 만든 접근자입니다. 봇은 이 접근자를 사용하여 사용자에 대해 아는 정보를 추적합니다. 본사는 대화 코드에서 명시적으로 이 상태를 관리합니다.

상태 속성 접근자의 _get_ 및 _set_ 메서드는 상태 관리 개체의 캐시에서 속성의 값을 가져오고 설정합니다. 처음으로 상태 속성의 값이 순서대로 요청되는 경우 캐시가 채워지지만 명시적으로 유지되어야 합니다. 이러한 상태 속성의 변경 내용을 모두 유지하기 위해 해당하는 상태 관리 개체의 _save changes_ 메서드를 호출합니다.

이 샘플은 대화 상자 내에서 사용자 프로필 상태를 업데이트합니다. 이 방법은 간단한 봇에 사용할 수 있지만 봇을 통해 대화 상자를 다시 사용하려는 경우 작동하지 않습니다.

대화 상자 단계와 봇 상태를 분리하는 다양한 옵션이 있습니다. 예를 들어, 대화 상자가 완전한 정보를 수집하면 다음을 수행할 수 있습니다.

- end dialog 메서드를 사용하여 수집된 데이터를 부모 컨텍스트에 다시 반환 값으로 제공합니다. 이것은 봇의 순서 처리기 또는 대화 상자 스택의 초기 활성 대화 상자일 수 있습니다. 프롬프트 클래스를 디자인하는 방법입니다.
- 적절한 서비스에 요청을 생성합니다. 봇이 대규모 서비스에 대한 프론트 엔드의 역할을 하는 경우 제대로 작동할 수 있습니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [봇에 자연어 해석 추가](bot-builder-howto-v4-luis.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample
