---
title: 사용자 중단 처리 | Microsoft Docs
description: 사용자 중단 및 직접 대화 흐름을 처리하는 방법을 알아봅니다.
keywords: 인터럽트, 중단, 토픽 전환, 중단하기기
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/18/2019
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bd8682966dbb2e33a536a72a4016ef23e9c1fc75
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032623"
---
# <a name="handle-user-interruptions"></a>사용자 중단 처리

[!INCLUDE[applies-to](../includes/applies-to.md)]

중단을 처리하는 작업은 강력한 봇의 중요한 측면입니다. 사용자가 항상 정의된 대화 흐름을 단계별로 따르는 것은 아닙니다. 사용자가 프로세스 중간에 질문을 하거나 프로세스를 완료하지 않고 취소하려는 경우가 있을 수 있습니다. 이 항목에서는 봇에서 사용자 중단을 처리하는 몇 가지 방법을 살펴봅니다.

## <a name="prerequisites"></a>필수 조건

- [봇 기본 사항][concept-basics], [상태 관리][concept-state], [대화 라이브러리][concept-dialogs] 및 [대화 재사용][component-dialogs] 방법에 대한 지식
- [**CSharp**][cs-sample] 또는 [**JavaScript**][js-sample]로 작성된 샘플의 복사본

## <a name="about-this-sample"></a>이 샘플 정보

이 문서에 사용된 샘플은 대화를 통해 사용자의 항공편 정보를 확인하는 항공편 예약 봇을 모델링합니다. 사용자는 봇과 대화 중에 언제든지 _help_ 또는 _cancel_ 명령을 실행하여 대화를 중단할 수 있습니다. 여기서 처리하는 대화 유형은 두 가지가 있습니다.

- **순서 수준**: 순서 수준에서 처리를 무시하되, 스택의 대화에 제공된 정보를 남겨놓습니다. 다음 순서에 중단된 부분부터 계속합니다. 
- **대화 수준**: 봇이 완전히 다시 시작될 수 있도록 처리를 완전히 취소합니다.

## <a name="define-and-implement-the-interruption-logic"></a>중단 논리 정의 및 구현

먼저 _help_ 및 _cancel_ 중단을 정의하고 구현합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

대화를 사용하려면 **Microsoft.Bot.Builder.Dialogs** NuGet 패키지를 설치합니다.

**Dialogs\CancelAndHelpDialog.cs**

먼저 사용자 중단을 처리하는 `CancelAndHelpDialog` 클래스를 구현합니다.

[!code-csharp[Class signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=11)]

`CancelAndHelpDialog` 클래스에서 `OnBeginDialogAsync` 및 `OnContinueDialogAsync` 메서드는 `InerruptAsync` 메서드를 호출하여 사용자가 정상적인 흐름을 중단했는지 여부를 확인합니다. 흐름이 중단된 경우 기본 클래스 메서드가 호출되며, 그렇지 않은 경우에는 `InterruptAsync`의 반환 값이 반환됩니다.

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=18-27)]

사용자가 "help"를 입력하면 `InterrupAsync` 메서드가 메시지를 보낸 후 `DialogTurnResult (DialogTurnStatus.Waiting)`를 호출하여 맨 위에 있는 대화가 사용자의 응답을 기다리고 있음을 나타냅니다. 이러한 방식으로 대화 흐름이 한 순서 동안만 중단되고, 다음 순서에는 중단된 부분부터 계속됩니다.

사용자가 "cancel"을 입력하면 내부 대화 컨텍스트의 `CancelAllDialogsAsync`가 호출되어 대화 스택이 삭제되며 취소된 상태로 아무런 결과 값 없이 종료됩니다. 뒷부분에 표시되는 `MainDialog`에는 사용자가 예약을 확인하지 않도록 선택한 경우와 마찬가지로 예약 대화가 종료되고 null이 반환된 것으로 나타납니다.

[!code-csharp[Interrupt](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=40-61&highlight=11-12,16-17)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

대화를 사용하려면 **botbuilder-dialogs** npm 패키지를 설치하세요.

**dialogs/cancelAndHelpDialog.js**

먼저 사용자 중단을 처리하는 `CancelAndHelpDialog` 클래스를 구현합니다.

[!code-javascript[Class signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=10)]

`CancelAndHelpDialog` 클래스에서 `onBeginDialog` 및 `onContinueDialog` 메서드는 `interrupt` 메서드를 호출하여 사용자가 정상적인 흐름을 중단했는지 여부를 확인합니다. 흐름이 중단된 경우 기본 클래스 메서드가 호출되며, 그렇지 않은 경우에는 `interrupt`의 반환 값이 반환됩니다.

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=11-25)]

사용자가 "help"를 입력하면 `interrupt` 메서드가 메시지를 보낸 후 `{ status: DialogTurnStatus.waiting }` 개체를 반환하여 맨 위에 있는 대화가 사용자의 응답을 기다리고 있음을 나타냅니다. 이러한 방식으로 대화 흐름이 한 순서 동안만 중단되고, 다음 순서에는 중단된 부분부터 계속됩니다.

사용자가 "cancel"을 입력하면 내부 대화 컨텍스트의 `cancelAllDialogs`가 호출되어 대화 스택이 삭제되며 취소된 상태로 아무런 결과 값 없이 종료됩니다. 뒷부분에 표시되는 `MainDialog`에는 사용자가 예약을 확인하지 않도록 선택한 경우와 마찬가지로 예약 대화가 종료되고 null이 반환된 것으로 나타납니다.

[!code-javascript[Interrupt](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=27-40&highlight=7-8,11-12)]

---

## <a name="check-for-interruptions-each-turn"></a>각 순서의 중단 확인

중단 처리 클래스의 작동 방식을 살펴보았으므로 본론으로 돌아가서 봇이 사용자로부터 새 메시지를 수신하면 어떤 일이 일어나는지 살펴보도록 하겠습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

새 메시지 작업이 도착하면 봇이 `MainDialog`를 실행합니다. `MainDialog`가 사용자에게 어떤 도움이 필요한지 묻는 메시지를 표시합니다. 그런 다음, 아래 표시된 것처럼 `BeginDialogAsync`를 호출하여 `MainDialog.ActStepAsync` 메서드에서 `BookingDialog`를 시작합니다.

[!code-csharp[ActStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=54-68&highlight=13-14)]

다음으로, `MainDialog` 클래스의 `FinalStepAsync` 메서드에서 예약 대화가 종료되고, 예약이 완료되거나 취소된 것으로 간주됩니다.

[!code-csharp[FinalStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=70-91)]

`BookingDialog`의 코드는 중단 처리와 직접적인 관련이 없으므로 여기에 표시하지 않았습니다. 이는 사용자에게 예약 세부 정보를 묻는 데 사용됩니다. 이 코드는 **Dialogs\BookingDialogs.cs**에서 확인할 수 있습니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

새 메시지 작업이 도착하면 봇이 `MainDialog`를 실행합니다. `MainDialog`가 사용자에게 어떤 도움이 필요한지 묻는 메시지를 표시합니다. 그런 다음, 아래 표시된 것처럼 `beginDialog`를 호출하여 `MainDialog.actStep` 메서드에서 `bookingDialog`를 시작합니다.

[!code-javascript[Act step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=71-88&highlight=16-17)]

다음으로, `MainDialog` 클래스의 `finalStep` 메서드에서 예약 대화가 종료되고, 예약이 완료되거나 취소된 것으로 간주됩니다.

[!code-javascript[Final step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=93-110)]

`BookingDialog`의 코드는 중단 처리와 직접적인 관련이 없으므로 여기에 표시하지 않았습니다. 이는 사용자에게 예약 세부 정보를 묻는 데 사용됩니다. 이 코드는 **dialogs/bookingDialogs.js**에서 확인할 수 있습니다.

---

## <a name="handle-unexpected-errors"></a>예기치 않은 오류 처리

다음으로, 발생 가능한 처리되지 않은 예외를 살펴보겠습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**AdapterWithErrorHandler.cs**

이 샘플에서 어댑터의 `OnTurnError` 처리기는 봇의 순서 논리가 throw한 예외를 수신합니다. throw된 예외가 있으면 봇이 비정상 상태로 인해 오류 루프에 갇히지 않도록 처리기가 현재 대화의 대화 상태를 삭제합니다.

[!code-csharp[AdapterWithErrorHandler](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/AdapterWithErrorHandler.cs?range=12-41)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

이 샘플에서 어댑터의 `onTurnError` 처리기는 봇의 순서 논리가 throw한 예외를 수신합니다. throw된 예외가 있으면 봇이 비정상 상태로 인해 오류 루프에 갇히지 않도록 처리기가 현재 대화의 대화 상태를 삭제합니다.

[!code-javascript[AdapterWithErrorHandler](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=28-38)]

---

## <a name="register-services"></a>서비스 등록

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**

마지막으로, `Startup.cs`에서 봇이 일시적으로 생성되고, 매 순서에 봇의 새 인스턴스가 생성됩니다.

[!code-csharp[Add transient bot](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Startup.cs?range=46-47)]

참고로, 위의 봇을 만드는 호출에서 사용되는 클래스 정의는 다음과 같습니다.

[!code-csharp[MainDialog signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=15)]
[!code-csharp[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogAndWelcomeBot.cs?range=16)]
[!code-csharp[DialogBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogBot.cs?range=18)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

마지막으로, `index.js`에서 봇이 생성됩니다.

[!code-javascript[Create bot](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=55-56)]

참고로, 위의 봇을 만드는 호출에서 사용되는 클래스 정의는 다음과 같습니다.

[!code-javascript[MainDialog signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=12)]
[!code-javascript[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogAndWelcomeBot.js?range=8)]
[!code-javascript[DialogBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogBot.js?range=6)]

---

## <a name="to-test-the-bot"></a>봇 테스트

1. 아직 설치하지 않은 경우 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)를 설치합니다.
1. 샘플을 머신에서 로컬로 실행합니다.
1. 에뮬레이터를 시작하고 봇에 연결한 다음, 아래와 같은 메시지를 보냅니다.

<!--![test dialog prompt sample](~/media/emulator-v4/test-dialog-prompt.png)-->

## <a name="additional-information"></a>추가 정보

- [인증 샘플](https://aka.ms/logout)은 여기에 표시된 것과 비슷한 패턴을 사용하여 중단을 처리하는 로그아웃 처리 방법을 보여줍니다.

- 아무 작업도 수행하지 않고 사용자가 무엇을 해야 할지 고민하는 상황을 만들지 말고, 기본 응답을 전송해야 합니다. 기본 응답은 사용자가 다시 정상적인 프로세스를 진행할 수 있도록 봇이 이해하는 명령을 사용자에게 알려주어야 합니다.

- 순서 진행 중에 순서 컨텍스트의 _responded_ 속성은 봇이 이번 순서에 사용자에게 메시지를 보냈는지 여부를 나타냅니다. 봇은 순서가 종료되기 전에 사용자에게 메시지를 보내야 합니다(사용자 입력에 대한 간단한 수신 확인이라도 보내야 함).

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[using-luis]: bot-builder-howto-v4-luis.md
[using-qna]: bot-builder-howto-qna.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-core-sample
[js-sample]: https://aka.ms/js-core-sample
