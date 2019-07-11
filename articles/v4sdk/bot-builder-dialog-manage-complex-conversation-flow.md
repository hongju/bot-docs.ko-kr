---
title: 분기 및 루프를 사용하여 고급 대화 흐름 만들기 | Microsoft Docs
description: Bot Framework SDK에서 대화를 사용하여 복잡한 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 복잡한 대화 흐름, 반복, 루프, 메뉴, 다이얼로그, 프롬프트, 폭포, 다이얼로그 집합
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b7ffa16c2f0a00043b12faec1d31bbfe5bfa250f
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/05/2019
ms.locfileid: "67587469"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>분기 및 루프를 사용하여 고급 대화 흐름 만들기

[!INCLUDE[applies-to](../includes/applies-to.md)]

대화 상자 라이브러리를 사용하여 단순 및 복합 대화 흐름을 관리할 수 있습니다.
이 문서에서는 분기하고 반복되는 복잡한 대화를 관리하는 방법에 대해 설명합니다.
또한 대화의 다른 부분 간에 인수를 전달하는 방법도 보여 줍니다.

## <a name="prerequisites"></a>필수 조건

- [봇 기본 사항][concept-basics], [managing state][concept-state], [대화 상자 라이브러리][concept-dialogs] 및 [순차적인 대화 흐름을 구현][simple-dialog]하는 방법에 대한 지식.
- [**CSharp**][cs-sample]or [**JavaScript**][js-sample]로 작성된 복잡한 대화 상자 샘플의 복사본.

## <a name="about-this-sample"></a>이 샘플 정보

이 샘플은 목록에서 최대 2개의 회사를 검토하기 위해 사용자가 가입할 수 있는 봇을 나타냅니다.

`DialogAndWelcomeBot`은 다양한 작업에 대한 처리기 및 봇의 턴 처리기를 정의하는 `DialogBot`의 확장입니다. `DialogBot`은 대화 상자를 실행합니다.

- _실행_ 메서드는 `DialogBot`이 대화 상자를 시작하기 위해 사용합니다.
- `MainDialog`는 대화의 특정 시기에 호출되는 다른 두 대화 상자의 부모입니다. 해당 대화 상자에 대한 세부 정보는 이 문서 전체에서 제공됩니다.

대화 상자는 `MainDialog`, `TopLevelDialog` 및 `ReviewSelectionDialog` 구성 요소로 분할되며, 이들이 함께 다음을 수행합니다.

- 이들은 사용자의 이름과 나이를 요청한 다음, 사용자의 나이에 따라 _분기_합니다.
  - 사용자의 나이가 너무 적은 경우 사용자에게 회사를 검토하도록 요청하지 않습니다.
  - 사용자의 나이가 충분히 많은 경우 사용자의 검토 기본 설정을 수집하기 시작합니다.
    - 사용자는 이 과정을 통해 검토할 회사를 선택할 수 있습니다.
    - 사용자가 회사를 선택하는 경우 두 번째 회사를 선택할 수 있도록 이 과정이 _순환_됩니다.
- 끝으로, 사용자에게 참여에 대한 감사 인사를 보냅니다.

이 과정에서 복잡한 대화를 관리하기 위해 폭포 대화 상자와 몇 개의 프롬프트를 사용합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![복잡한 봇 흐름](./media/complex-conversation-flow.png)

대화 상자를 사용하려면 프로젝트에서 **Microsoft.Bot.Builder.Dialogs** NuGet 패키지를 설치해야 합니다.

**Startup.cs**

여기서는 봇 서비스를 `Startup`에 등록합니다. 해당 서비스는 종속성 주입을 통해 코드의 다른 부분에 사용할 수 있습니다.

- 봇 기본 서비스: 자격 증명 공급자, 어댑터 및 봇 구현
- 상태 관리 서비스: 스토리지, 사용자 상태 및 대화 상태
- 봇이 사용할 대화 상자

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Startup.cs?range=22-39)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![복잡한 봇 흐름](./media/complex-conversation-flow-js.png)

대화 상자를 사용하려면 프로젝트에서 **botbuilder-dialogs** npm 패키지를 설치해야 합니다.

**index.js**

코드의 다른 부분에 필요한 봇 서비스를 만듭니다.

- 봇 기본 서비스: 어댑터 및 봇 구현
- 상태 관리 서비스: 스토리지, 사용자 상태 및 대화 상태
- 봇이 사용할 대화 상자

[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=25-38)]
[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=43-45)]

---

> [!NOTE]
> 메모리 저장소는 테스트 목적으로만 사용되며 프로덕션용이 아닙니다.
> 프로덕션 봇에는 영구 스토리지 형식을 사용해야 합니다.

## <a name="define-a-class-in-which-to-store-the-collected-information"></a>수집된 정보를 저장할 클래스를 정의합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**UserProfile.cs**

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/UserProfile.cs?range=8-16)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**userProfile.js**

[!code-javascript[UserProfile class](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/userProfile.js?range=4-12)]

---

## <a name="create-the-dialogs-to-use"></a>사용할 대화 상자를 만듭니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

여기서는 두 개의 주 단계를 포함하고 대화 상자 및 프롬프트를 지시하는 구성 요소 대화 상자 `MainDialog`를 정의했습니다. 초기 단계에서는 아래에 설명하는 `TopLevelDialog`를 호출합니다.

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/MainDialog.cs?range=31-50&highlight=3)]

**Dialogs\TopLevelDialog.cs**

초기의 최상위 대화에는 다음 네 가지 단계가 있습니다.

1. 사용자의 이름을 요청합니다.
1. 사용자의 나이를 요청합니다.
1. 사용자의 나이에 따라 분기합니다.
1. 마지막으로 사용자의 참여에 대한 감사 인사를 보내고, 수집된 정보를 반환합니다.

첫 번째 단계에서는 대화 상자가 매번 빈 프로필로 시작하도록 사용자의 프로필을 지웁니다. 마지막 단계는 종료할 때 정보를 반환하므로 `AcknowledgementStepAsync`는 정보를 사용자 상태에 저장하고 해당 정보를 마지막 단계에 사용하기 위해 주 대화 상자에 반환하는 작업으로 종료합니다.

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=39-96&highlight=3-4,47-49,56-57)]

**Dialogs\ReviewSelectionDialog.cs**

검토-선택 대화 상자는 최상위 대화 상자의 `StartSelectionStepAsync`에서 시작하며, 다음 두 단계로 이루어져 있습니다.

1. 사용자에게 검토할 회사를 선택하도록 요청하거나 `done`을 선택하여 완료합니다.
1. 이 대화를 반복하거나 적절하게 종료합니다.

이 디자인에서는 최상위 대화 상자가 항상 스택의 검토-선택 대화 상자 앞에 배치되며, 검토-선택 대화 상자는 최상위 대화 상자의 자식으로 간주할 수 있습니다.

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=42-106)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

여기서는 두 개의 주 단계를 포함하고 대화 상자 및 프롬프트를 지시하는 구성 요소 대화 상자 `MainDialog`를 정의했습니다. 초기 단계에서는 아래에 설명하는 `TopLevelDialog`를 호출합니다.

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=43-55&highlight=2)]

**dialogs/topLevelDialog.js**

초기의 최상위 대화에는 다음 네 가지 단계가 있습니다.

1. 사용자의 이름을 요청합니다.
1. 사용자의 나이를 요청합니다.
1. 사용자의 나이에 따라 분기합니다.
1. 마지막으로 사용자의 참여에 대한 감사 인사를 보내고, 수집된 정보를 반환합니다.

첫 번째 단계에서는 대화 상자가 매번 빈 프로필로 시작하도록 사용자의 프로필을 지웁니다. 마지막 단계는 종료할 때 정보를 반환하므로 `acknowledgementStep`는 정보를 사용자 상태에 저장하고 해당 정보를 마지막 단계에 사용하기 위해 주 대화 상자에 반환하는 작업으로 종료합니다.

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=32-76&highlight=2-3,37-39,43-44)]

**dialogs/reviewSelectionDialog.js**

검토-선택 대화 상자는 최상위 대화 상자의 `startSelectionStep`에서 시작하며, 다음 두 단계로 이루어져 있습니다.

1. 사용자에게 검토할 회사를 선택하도록 요청하거나 `done`을 선택하여 완료합니다.
1. 이 대화를 반복하거나 적절하게 종료합니다.

이 디자인에서는 최상위 대화 상자가 항상 스택의 검토-선택 대화 상자 앞에 배치되며, 검토-선택 대화 상자는 최상위 대화 상자의 자식으로 간주할 수 있습니다.

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=33-78)]

---

## <a name="implement-the-code-to-manage-the-dialog"></a>대화 상자를 관리하는 코드를 구현 합니다.

봇의 턴 처리기는 이러한 대화에서 정의된 하나의 대화 흐름을 반복합니다.
사용자로부터 메시지를 받은 경우,

1. 활성 대화가 있으면 계속 진행합니다.
   - 활성 대화가 없으면 사용자 프로필을 지우고 최상위 대화를 시작합니다.
   - 활성 대화가 완료되면 반환된 정보를 수집 및 저장하고 상태 메시지를 표시합니다.
   - 그렇지 않으면 활성 대화가 아직 진행 중이므로 이 시점에서는 다른 작업을 수행할 필요가 없습니다.
1. 대화 상태에 대한 모든 업데이트가 유지되도록 대화 상태를 저장합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- **DialogExtensions.cs**

In this sample, we've defined a `Run` helper method that we will use to create and access the dialog context.
Since component dialog defines an inner dialog set, we have to create an outer dialog set that's visible to the message handler code, and use that to create a dialog context.

- `dialog` is the main component dialog for the bot.
- `turnContext` is the current turn context for the bot.

[!code-csharp[Run method](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/DialogExtensions.cs?range=13-24)]

-->

**Bots\DialogBot.cs**

메시지 처리기는 `RunAsync` 메서드를 호출하여 대화 상자를 관리하며, 여기서는 턴 중에 발생했을 수 있는 대화 및 사용자 상태의 변경 내용을 저장하도록 순서 처리기를 재정의했습니다. 기본 `OnTurnAsync`는 `OnMessageActivityAsync` 메서드를 호출하여 해당 턴의 끝에서 저장 호출이 발생하는지 확인합니다.

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

**Bots\DialogAndWelcome.cs**

`DialogAndWelcomeBot`은 위의 `DialogBot`을 사용자가 대화에 참가할 때 환영 메시지를 제공하도록 확장하며, `Startup.cs`에서 이 메서드를 호출합니다.

[!code-csharp[On members added](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogAndWelcome.cs?range=21-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

이 샘플에서는 대화 상자 컨텍스트를 만들고 액세스하는 데 사용할 `run` 메서드를 정의했습니다.
구성 요소 대화 상자는 내부 대화 상자 세트를 정의하므로 메시지 처리기 코드에서 볼 수 있는 외부 대화 상자 세트를 만들어야 하며, 대화 상자 컨텍스트를 만들 때 해당 세트를 사용합니다.

- `turnContext`는 봇에 대한 현재 턴 컨텍스트입니다.
- `accessor`는 대화 상태를 관리하기 위해 만든 접근자입니다.

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=32-41)]

**bots/dialogBot.js**

메시지 처리기는 `run` 도우미 메서드를 호출하여 대화 상자를 관리하며, 여기서는 턴 중에 발생했을 수 있는 대화 및 사용자 상태의 변경 내용을 저장하는 턴 처리기를 구현합니다. `next`를 호출하면 기본 구현이 `onDialog` 메서드를 호출하여 해당 턴의 끝에서 저장 호출이 발생하는지 확인하게 합니다.

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=30-47)]

**bots/dialogAndWelcomeBot.js**

`DialogAndWelcomeBot`은 위의 `DialogBot`을 사용자가 대화에 참가할 때 환영 메시지를 제공하도록 확장하며, `Startup.cs`에서 이 메서드를 호출합니다.

[!code-javascript[On members added](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogAndWelcomeBot.js?range=10-21)]

---

## <a name="branch-and-loop"></a>분기 및 루프

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\TopLevelDialog.cs**

다음은 _최상위_ 대화 상자의 단계에서 나온 샘플 분기 논리입니다.

[!code-csharp[branching logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=68-80)]

**Dialogs\ReviewSelectionDialog.cs**

다음은 _검토-선택_ 대화 상자의 단계에서 나온 샘플 루프 논리입니다.

[!code-csharp[looping logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=96-105)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/topLevelDialog.js**

다음은 _최상위_ 대화 상자의 단계에서 나온 샘플 분기 논리입니다.

[!code-javascript[branching logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=56-64)]

**dialogs/reviewSelectionDialog.js**

다음은 _검토-선택_ 대화 상자의 단계에서 나온 샘플 루프 논리입니다.

[!code-javascript[looping logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=71-77)]

---

## <a name="to-test-the-bot"></a>봇 테스트

1. 아직 설치하지 않은 경우 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)를 설치합니다.
1. 샘플을 머신에서 로컬로 실행합니다.
1. 에뮬레이터를 시작하고 봇에 연결한 다음, 아래와 같은 메시지를 보냅니다.

![복잡한 대화 샘플 테스트](~/media/emulator-v4/test-complex-dialog.png)

## <a name="additional-resources"></a>추가 리소스

대화를 구현하는 방법에 대한 소개는 [순차적 대화 흐름 구현][simple-dialog]을 참조하세요. 여기에서는 단일 폭포 대화와 몇 가지 프롬프트를 사용하여 사용자에게 일련의 질문을 하는 간단한 상호 작용을 만듭니다.

Dialogs 라이브러리에는 프롬프트에 대한 기본 유효성 검사가 포함되어 있습니다. 사용자 지정 유효성 검사를 추가할 수도 있습니다. 자세한 내용은 [대화 프롬프트를 사용하여 사용자 입력 수집][dialog-prompts]을 참조하세요.

대화 코드를 단순화하고 여러 봇을 다시 사용하기 위해 대화 세트의 일부를 별도의 클래스로 정의할 수 있습니다.
자세한 내용은 [대화 재사용][component-dialogs]을 참조하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [대화 상자 재사용](bot-builder-compositcontrol.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-complex-dialog-sample
[js-sample]: https://aka.ms/js-complex-dialog-sample
