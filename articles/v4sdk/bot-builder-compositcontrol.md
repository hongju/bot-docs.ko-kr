---
title: 다이얼로그 다시 사용 | Microsoft Docs
description: Bot Framework SDK에서 구성 요소 대화 상자를 사용하여 봇 논리를 모듈화하는 방법을 알아봅니다.
keywords: 복합 컨트롤, 모듈식 봇 논리
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77f1c154af5821b1e476546f307a01be27f568c0
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/05/2019
ms.locfileid: "67587497"
---
# <a name="reuse-dialogs"></a>대화 상자 재사용

[!INCLUDE[applies-to](../includes/applies-to.md)]

구성 요소 대화 상자를 사용하면 큰 대화 상자 세트를 관리가 용이한 더 작은 구성 요소로 분할하여 특정 시나리오를 처리하는 독립적인 대화 상자를 만들 수 있습니다. 이러한 각 구성 요소에는 자체의 고유한 대화 상자 세트가 있으며, 해당 세트의 외부에 있는 대화 상자 세트와의 이름 충돌을 방지합니다.

## <a name="prerequisites"></a>필수 조건

- [봇 기본 사항][concept-basics], the [dialogs library][concept-dialogs] 및 [대화 관리][간단한 흐름]에 대한 방법의 지식.
- [**C#**][cs-sample] 또는 [**JavaScript**][js-sample]로 작성된 다중 턴 프롬프트 샘플의 복사본.

## <a name="about-the-sample"></a>샘플 정보

다중 턴 프롬프트 샘플에서는 폭포 대화 상자, 몇 가지 프롬프트 및 구성 요소 대화 상자를 사용하여 사용자에게 일련의 질문을 하는 간단한 상호 작용을 만듭니다. 코드는 대화 상자를 사용하여 다음 단계를 순환합니다.

| 단계        | 프롬프트 형식  |
|:-------------|:-------------|
| 사용자에게 사용자의 전송 모드 요청 | 선택 항목 프롬프트 |
| 사용자에게 사용자의 이름 요청 | 텍스트 프롬프트 |
| 사용자에게 자신의 나이 제공을 원하는지 여부 질문 | 확인 프롬프트 |
| 사용자가 예로 답하면 나이 요청  | 0보다 크고 150보다 작은 나이만 허용하는 유효성 검사를 포함한 숫자 프롬프트 |
| 수집된 정보가 "올바른지" 질문 | 확인 프롬프트 재사용 |

끝으로, 사용자가 예로 답하면 수집된 정보를 표시합니다. 그렇지 않으면 사용자에게 해당 사용자의 정보가 보관되지 않을 것임을 알려줍니다.

## <a name="implement-the-component-dialog"></a>구성 요소 대화 상자 구현

다중 턴 프롬프트 샘플에서는 _폭포 대화 상자_, 몇 가지 _프롬프트_ 및 _구성 요소 대화 상자_ 를 사용하여 사용자에게 일련의 질문을 하는 간단한 상호 작용을 만듭니다.

구성 요소 대화 상자는 하나 이상의 대화 상자를 캡슐화합니다. 구성 요소 대화 상자에는 내부 대화 상자 세트가 있으며, 이 내부 대화 상자 세트에 추가하는 대화 상자 및 프롬프트에는 구성 요소 대화 상자 내에서만 볼 수 있는 자체의 고유한 ID가 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

대화 상자를 사용하려면 **Microsoft.Bot.Builder.Dialogs** NuGet 패키지를 설치합니다.

**Dialogs\UserProfileDialog.cs**

여기서 `UserProfileDialog` 클래스는 `ComponentDialog` 클래스에서 파생됩니다.

[!code-csharp[Class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=13)]

생성자 내에서 `AddDialog` 메서드는 구성 요소 대화 상자에 프롬프트를 표시합니다. 이 메서드를 사용하여 추가하는 첫 번째 항목은 초기 대화 상자로 설정되지만 `InitialDialogId` 속성을 명시적으로 설정하여 이를 변경할 수 있습니다. 구성 요소 대화 상자를 시작하면 해당 _초기 대화 상자_ 가 시작됩니다.

[!code-csharp[Constructor](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=17-42)]

이는 폭포 대화 상자의 첫 번째 단계에 대한 구현입니다.

[!code-csharp[First step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=44-54)]

폭포 대화 상자의 구현에 대한 자세한 내용은 [순차적 대화 흐름을 구현하는 방법](bot-builder-dialog-manage-complex-conversation-flow.md)을 참조하세요.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

대화 상자를 사용하려면 프로젝트에서 **botbuilder-dialogs** npm 패키지를 설치해야 합니다.

**dialogs/userProfileDialog.js**

여기서 `UserProfileDialog` 클래스는 `ComponentDialog`를 확장합니다.

[!code-javascript[Class](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=24)]

생성자 내에서 `AddDialog` 메서드는 구성 요소 대화 상자에 프롬프트를 표시합니다. 이 메서드를 사용하여 추가하는 첫 번째 항목은 초기 대화 상자로 설정되지만 `InitialDialogId` 속성을 명시적으로 설정하여 이를 변경할 수 있습니다. 구성 요소 대화 상자를 시작하면 해당 _초기 대화 상자_가 시작됩니다.

[!code-javascript[Constructor](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=25-47)]

이는 폭포 대화 상자의 첫 번째 단계에 대한 구현입니다.

[!code-javascript[First step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=66-73)]

폭포 대화 상자의 구현에 대한 자세한 내용은 [순차적 대화 흐름을 구현하는 방법](bot-builder-dialog-manage-complex-conversation-flow.md)을 참조하세요.

---

런타임에 구성 요소 대화 상자는 자체의 고유한 대화 상자 스택을 유지 관리합니다. 구성 요소 대화 상자가 시작될 때:

- 인스턴스가 생성되어 외부 대화 상자 스택에 추가됩니다.
- 자체의 상태에 추가하는 내부 대화 상자 스택을 만듭니다.
- 자체의 초기 대화 상자를 시작하고 이를 내부 대화 상자 스택에 추가합니다.

부모 컨텍스트에서는 구성 요소가 활성 대화 상자인 것처럼 보입니다. 구성 요소 내에서는 초기 대화 상자가 활성 대화 상자인 것처럼 보입니다.

### <a name="implement-the-rest-of-the-dialog-and-add-it-to-the-bot"></a>대화 상자의 나머지 부분을 구현하고 이를 봇에 추가합니다.

구성 요소 대화 상자를 추가하는 외부 대화 상자 세트에서 구성 요소 대화 상자에는 해당 대화 상자를 만들 때 사용한 ID가 지정됩니다. 외부 세트의 경우 구성 요소는 프롬프트와 매우 유사하게 단일 대화 상자처럼 보입니다.

구성 요소 대화 상자를 사용하려면 해당 대화 상자의 인스턴스를 봇의 대화 상자 세트에 추가합니다. 이는 외부 대화 상자 세트입니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\DialoBot.cs**

샘플에서 이 작업은 봇의 `OnMessageActivityAsync` 메서드에서 호출된 `RunAsync` 메서드를 사용하여 수행됩니다.

[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=42-48)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/userProfileDialog.js**

샘플에서는 `run` 메서드를 사용자 프로필 대화 상자에 추가했습니다.

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=55-64)]

**bots/dialogBot.js**

`run` 메서드는 봇의 `onMessage` 메서드에서 호출됩니다.

[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=30-37)]

---

## <a name="to-test-the-bot"></a>봇 테스트

1. 아직 설치하지 않은 경우 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)를 설치합니다.
1. 샘플을 머신에서 로컬로 실행합니다.
1. 에뮬레이터를 시작하고 봇에 연결한 다음, 아래와 같은 메시지를 보냅니다.

![다중 턴 프롬프트 대화 상자의 샘플 실행](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>추가 정보

### <a name="how-cancellation-works-for-component-dialogs"></a>구성 요소 대화 상자에 대한 취소가 작동하는 방법

구성 요소 대화 상자의 컨텍스트에서 _모든 대화 상자 취소_ 를 호출하면 구성 요소 대화 상자에서 자체의 스택에 있는 모든 대화 상자를 취소한 다음, 종료하고 제어권을 외부 스택의 다음 대화 상자에 되돌려줍니다.

외부 컨텍스트에서 _모든 대화 상자 취소_ 를 호출하면 구성 요소가 외부 컨텍스트의 나머지 대화 상자와 함께 취소됩니다.

봇에서 중첩된 구성 요소 대화 상자를 관리할 때 이 점을 염두에 두어야 합니다.

## <a name="next-steps"></a>다음 단계

대화의 정상적인 흐름을 중단할 수 있는 "도움말" 또는 "취소"와 같은 추가 입력에 반응하도록 봇을 향상시킬 수 있습니다.

> [!div class="nextstepaction"]
> [사용자 작업 중단 처리](bot-builder-howto-handle-user-interrupt.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample
