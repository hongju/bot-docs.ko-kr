---
title: 동일한 .NET Framework 프로젝트 내에서 기존 봇 마이그레이션 | Microsoft Docs
description: 동일한 프로젝트를 사용하여 기존 .NET v3 봇을 가져와 .NET v4 SDK로 마이그레이션합니다.
keywords: 봇 마이그레이션, Formflow, 대화, v3 봇
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 519515a2174a7028af7bc170ca8a7c40f7d48c52
ms.sourcegitcommit: b053c0ca7f2e9e60679f7e82e583c57ae83fcb50
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/19/2019
ms.locfileid: "68336728"
---
# <a name="migrate-a-net-v3-bot-to-a-net-framework-v4-bot"></a>.NET v3 봇을 .NET Framework v4 봇으로 마이그레이션

이 문서에서는 _프로젝트 형식을 변환하지 않고_ [v3 ContosoHelpdeskChatBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3)을 v4 봇으로 변환합니다. 이것은 .NET Framework 프로젝트로 그대로 유지됩니다.
이 변환은 다음 단계로 구분됩니다.

1. NuGet 패키지 업데이트 및 설치
1. Global.asax.cs 파일 업데이트
1. MessagesController 클래스 업데이트
1. 대화 변환

이 변환의 결과가 [.NET Framework v4 ContosoHelpdeskChatBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework)입니다.
새 프로젝트에서 .NET Core v4 봇으로 마이그레이션하려면 [.NET v3 봇을 .NET Core v4 봇으로 마이그레이션](conversion-core.md)을 참조하세요.

Bot Framework SDK v4는 SDK v3과 동일한 기본 REST API를 기반으로 합니다. 그러나 SDK v4는 이전 버전의 SDK를 리팩터링하여 개발자가 자신의 봇을 더 유연하게 제어할 수 있도록 합니다. SDK의 주요 변경 내용은 다음과 같습니다.

- 상태가 상태 관리 개체 및 속성 접근자를 통해 관리됩니다.
- 턴 처리기 설정 및 활동 전달이 변경되었습니다.
- 점수 지정 가능 개체가 더 이상 존재하지 않습니다. 대화에 제어를 전달하기 전에 턴 처리기에서 "global" 명령을 확인할 수 있습니다.
- 이전 버전과 매우 다른 새 Dialogs 라이브러리입니다. 구성 요소, 폭포 대화 및 v4용 Formflow 대화의 커뮤니티 구현을 사용하여 이전 대화를 새 대화 시스템으로 변환해야 합니다.

특정 변경에 대한 자세한 내용은 [v3 및 v4 .NET SDK 간의 차이점](migration-about.md)을 참조하세요.

## <a name="update-and-install-nuget-packages"></a>NuGet 패키지 업데이트 및 설치

1. **Microsoft.Bot.Builder.Azure** 및 **Microsoft.Bot.Builder.Integration.AspNet.WebApi**를 안정적인 최신 버전으로 업데이트합니다.

    그러면 **Microsoft.Bot.Builder** 및 **Microsoft.Bot.Connector** 패키지도 종속성이므로 업데이트됩니다.

1. **Microsoft.Bot.Builder.History** 패키지를 삭제합니다. 이 패키지는 v4 SDK의 일부가 아닙니다.
1. **Autofac.WebApi2**를 추가합니다.

    이 패키지는 ASP.NET에 종속성 주입을 지원하기 위해 사용합니다.

1. **Bot.Builder.Community.Dialogs.Formflow**를 추가합니다.

    이 패키지는 v3 Formflow 정의 파일에서 v4 대화를 빌드하기 위한 커뮤니티 라이브러리입니다. **Microsoft.Bot.Builder.Dialogs**도 종속성 중 하나이므로 설치됩니다.

> [!TIP]
> **Bot.Builder.Community.Dialogs.Formflow**는 .NET Standard 2.0 라이브러리이므로 프로젝트가 .NET Framework 4.6을 대상으로 하는 경우 4.6.1 이상으로 업데이트해야 합니다.
> 자세한 내용은 [.NET 구현 지원](https://docs.microsoft.com/en-us/dotnet/standard/net-standard#net-implementation-support)을 참조하세요.

이 시점에서 빌드하면 컴파일러 오류가 발생하지만, 무시할 수 있습니다. 변환이 완료되면 작업 코드가 준비됩니다.

## <a name="update-your-globalasaxcs-file"></a>Global.asax.cs 파일 업데이트

스캐폴딩 중 일부가 변경되었으며 v4에서 [상태 관리](../bot-builder-concept-state.md) 인프라의 일부를 직접 설정해야 합니다. 예를 들어 v4에서는 봇 어댑터를 사용하여 인증을 처리하고, 활동을 봇 코드로 전달하며, 상태 속성을 미리 선언합니다.

이제 v4에서 대화 지원에 필요한 `DialogState`에 대한 상태 속성을 만듭니다. 종속성 주입을 사용하여 컨트롤러와 봇 코드에 필요한 정보를 가져옵니다.

**Global.asax.cs**에서,

1. `using` 문을 업데이트합니다.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=4-13)]

1. `Application_Start` 메서드에서 다음 줄을 제거합니다.  
    [!code-csharp[Removed lines](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3/ContosoHelpdeskChatBot/Global.asax.cs?range=23-24)]

    그리고 다음 줄을 삽입합니다.  
    [!code-csharp[Reference BotConfig.Register](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=22)]

1. 더 이상 참조되지 않는 `RegisterBotModules` 메서드를 제거합니다.

1. `BotConfig.UpdateConversationContainer` 메서드를 이 `BotConfig.Register` 메서드로 바꿉니다. 이 메서드는 종속성 주입을 지원하는 데 필요한 개체를 등록합니다. 이 봇은 _user_ 또는 _private conversation_ 상태도 사용하지 않으므로, 대화 상태 관리 개체만 생성합니다.  
    [!code-csharp[Define BotConfig.Register](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Global.asax.cs?range=31-61)]

## <a name="update-your-messagescontroller-class"></a>MessagesController 클래스 업데이트

여기서는 봇이 v4에서 턴을 시작하므로 로트를 변경해야 합니다. 봇의 턴 처리기 자체를 제외한 대부분은 상용구로 생각할 수 있습니다. **Controllers\MessagesController.cs** 파일에서,

1. `using` 문을 업데이트합니다.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=4-8)]

1. 클래스에서 `[BotAuthentication]` 특성을 제거합니다. v4에서 봇의 어댑터는 인증을 처리합니다.

1. 이러한 필드와 생성자를 추가하여 초기화합니다. ASP.NET 및 Autofac은 종속성 주입을 사용하여 매개 변수 값을 가져옵니다. 이 작업을 지원하기 위해 **Global.asax.cs**에서 어댑터와 봇 개체를 등록했습니다.  
    [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=14-21)]

1. `Post` 메서드의 본문을 바꿉니다. 여기서는 어댑터를 사용하여 봇의 메시지 루프(턴 처리기)를 사용합니다.  
    [!code-csharp[Post method](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Controllers/MessagesController.cs?range=23-31)]

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>CancelScorable 및 GlobalMessageHandlersBotModule 클래스 삭제

v4에는 점수 지정 가능 개체가 없고 `cancel` 메시지에 반응하도록 순서 처리기를 업데이트했으므로 `CancelScorable`(**Dialogs\CancelScorable.cs**) 및 `GlobalMessageHandlersBotModule` 클래스를 삭제할 수 있습니다.

## <a name="create-your-bot-class"></a>봇 클래스 만들기

v4에서 턴 처리기 또는 메시지 루프 논리는 기본적으로 봇 파일에 있습니다. 일반 형식의 작업에 대한 처리기를 정의하는 `ActivityHandler`에서 파생하려고 합니다.

1. **Bots\DialogBots.cs** 파일을 만듭니다.

1. `using` 문을 업데이트합니다.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=4-8)]

1. `ActivityHandler`에서 `DialogBot`을 파생하고 대화 상자에 대한 일반 매개 변수를 추가합니다.  
    [!code-csharp[Class definition](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=19)]

1. 이러한 필드와 생성자를 추가하여 초기화합니다. 다시, ASP.NET 및 Autofac은 종속성 주입을 사용하여 매개 변수 값을 가져옵니다.  
    [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=21-28)]

1. `OnMessageActivityAsync`를 재정의하여 기본 대화 상자를 호출합니다. 잠시 후에 `Run` 확장 메서드를 정의하겠습니다.  
    [!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=38-47)]

1. `OnTurnAsync`를 재정의하여 턴이 끝나면 대화 상태를 저장합니다. v4에서 지속성 계층으로 상태를 기록하려면 이 작업을 명시적으로 수행해야 합니다. `ActivityHandler.OnTurnAsync` 메서드는 수신한 작업 형식에 따라 특정 작업 처리기 메서드를 호출하므로, 기본 메서드로 호출 후 상태를 저장합니다.  
    [!code-csharp[OnTurnAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=30-36)]

### <a name="create-the-run-extension-method"></a>Run 확장명 메서드 만들기

봇에서 기본 구성 요소를 실행하는 데 필요한 코드를 통합하기 위해 확장 메서드를 만들려고 합니다.

**DialogExtensions.cs** 파일을 만들고 `Run` 확장 메서드를 구현합니다.  
[!code-csharp[The extension](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/DialogExtensions.cs?range=4-41)]

## <a name="convert-your-dialogs"></a>대화 변환

원래 대화를 많이 변경하여 v4 SDK로 마이그레이션합니다. 지금은 컴파일러 오류에 대해 걱정하지 마세요. 변환이 완료되면 이러한 문제가 해결됩니다.
필요 이상으로 원래 코드를 수정하지 않기 위해 마이그레이션을 완료한 후에도 컴파일러 경고가 남아 있습니다.

모든 대화는 v3의 `IDialog<object>` 인터페이스를 구현하는 대신 `ComponentDialog`에서 파생됩니다.

이 봇에는 변환해야 하는 다음 네 개의 대화가 있습니다.

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | 옵션을 표시하고 다른 대화를 시작합니다. |
| [InstallAppDialog](#update-the-install-app-dialog) | 머신에 앱을 설치하라는 요청을 처리합니다. |
| [LocalAdminDialog](#update-the-local-admin-dialog) | 로컬 머신 관리자 권한에 대한 요청을 처리합니다. |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | 암호 재설정에 대한 요청을 처리합니다. |

이러한 대화는 입력을 수집하지만, 머신에서는 이러한 작업을 수행하지 않습니다.

### <a name="make-solution-wide-dialog-changes"></a>솔루션 수준 대화 변경

1. 전체 솔루션에서 나타나는 모든 `IDialog<object>` 항목을 `ComponentDialog`로 바꿉니다.
1. 전체 솔루션에서 나타나는 모든 `IDialogContext` 항목을 `DialogContext`로 바꿉니다.
1. 각 대화 클래스에서 `[Serializable]` 특성을 제거합니다.

대화 내의 제어 흐름과 메시징은 더 이상 동일한 방식으로 처리되지 않으므로 각 대화를 변환할 때 이를 수정해야 합니다.

| 작업(Operation) | v3 코드 | v4 코드 |
| :--- | :--- | :--- |
| 대화의 시작 처리 | `IDialog.StartAsync` 구현 | 폭포 대화의 첫 번째 단계로 설정 또는 `Dialog.BeginDialogAsync` 구현 |
| 대화의 연속 처리 | `IDialogContext.Wait` 호출 | 폭포 대화에 추가 단계 추가 또는 `Dialog.ContinueDialogAsync` 구현 |
| 사용자에게 메시지 보내기 | `IDialogContext.PostAsync` 호출 | `ITurnContext.SendActivityAsync` 호출 |
| 자식 대화 시작 | `IDialogContext.Call` 호출 | `DialogContext.BeginDialogAsync` 호출 |
| 현재 대화에 대한 완료 신호 | `IDialogContext.Done` 호출 | `DialogContext.EndDialogAsync` 호출 |
| 사용자의 입력 가져오기 | `IAwaitable<IMessageActivity>` 매개 변수 사용 | 폭포 내에서 프롬프트 사용 또는 `ITurnContext.Activity` 사용 |

v4 코드에 대한 참고 사항:

- 대화 상자 코드 내에서 `DialogContext.Context` 속성을 사용하여 현재 턴 컨텍스트를 가져옵니다.
- 폭포 단계에는 `DialogContext`에서 파생된 `WaterfallStepContext` 매개 변수가 있습니다.
- 모든 구체적인 대화 및 프롬프트 클래스는 `Dialog` 추상 클래스에서 파생됩니다.
- 구성 요소 대화를 만들 때 ID를 할당합니다. 대화 집합의 각 대화에는 해당 집합 내에서 고유한 ID가 할당되어야 합니다.

### <a name="update-the-root-dialog"></a>루트 대화 업데이트

이 봇에서 루트 대화는 사용자에게 옵션 집합에서 선택하도록 요청하는 메시지를 표시한 다음, 해당 선택 항목에 따라 자식 대화를 시작합니다. 그런 다음, 대화의 수명 동안 반복됩니다.

- 기본 흐름을 v4 SDK의 새로운 개념인 폭포 대화로 설정할 수 있습니다. 고정된 일단의 단계를 순서대로 실행합니다. 자세한 내용은 [순차적 대화 흐름 구현](~/v4sdk/bot-builder-dialog-manage-conversation-flow.md)을 참조하세요.
- 프롬프트는 이제 프롬프트 클래스를 통해 처리됩니다. 이 클래스는 입력을 요청하고, 최소한의 일부 처리 및 유효성 검사를 수행하며, 값을 반환하는 짧은 자식 대화입니다. 자세한 내용은 [대화 프롬프트를 사용하여 사용자 입력 수집](~/v4sdk/bot-builder-prompts.md)을 참조하세요.

**Dialogs/RootDialog.cs** 파일에서,

1. `using` 문을 업데이트합니다.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=4-10)]

1. `HelpdeskOptions` 옵션을 문자열 목록에서 선택 목록으로 변환해야 합니다. 여기서는 선택 번호(목록에서), 선택 값 또는 선택 항목의 동의어를 유효한 입력으로 수락하는 선택 프롬프트가 사용됩니다.  
    [!code-csharp[HelpDeskOptions](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=28-33)]

1. 생성자를 추가합니다. 이 코드는 다음을 수행합니다.
   - 만들어지는 ID가 대화의 각 인스턴스에 할당됩니다. 대화 ID는 대화가 추가되는 대화 집합의 일부입니다. 앞서 설명한 것처럼 `MessageController` 클래스에서 대화 상자 개체로 봇이 초기화되었습니다. 각 `ComponentDialog`에는 고유한 대화 ID 집합이 있는 자체의 내부 대화 집합이 있습니다.
   - 선택 프롬프트를 포함한 다른 대화를 자식 대화로 추가합니다. 여기서는 각 대화 ID에 대한 클래스 이름만 사용하고 있습니다.
   - 3단계 폭포 대화를 정의합니다. 이 대화는 잠시 후에 구현하겠습니다.
     - 대화는 먼저 사용자에게 수행할 작업을 선택하도록 요청하는 메시지를 표시합니다.
     - 그런 다음, 해당 선택 항목과 연결된 자식 대화를 시작합니다.
     - 그리고, 마지막으로 다시 시작합니다.
   - 폭포의 각 단계는 대리자이며, 가능한 한 원래 대화의 기존 코드를 사용하여 폭포의 다음 단계를 구현합니다.
   - 구성 요소 대화 상자를 시작하면 해당 _초기 대화 상자_가 시작됩니다. 기본적으로 이 대화는 구성 요소 대화에 추가된 첫 번째 자식 대화입니다. 이번에는 `InitialDialogId` 속성을 명시적으로 설정합니다. 즉, 기본 폭포 대화 상자가 집합에 추가되는 첫 번째 항목이 될 필요가 없습니다. 예를 들어 프롬프트를 먼저 추가하려면 런타임 문제를 일으키지 않고 해당 프롬프트를 추가할 수 있습니다.  
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=35-49)]

1. `StartAsync` 메서드는 삭제할 수 있습니다. 구성 요소 대화가 시작되면 해당 _초기_ 대화가 자동으로 시작됩니다. 이 경우 이 대화는 생성자에서 정의한 폭포 대화입니다. 또한 첫 번째 단계에서 자동으로 시작됩니다.

1. `MessageReceivedAsync` 및 `ShowOptions` 메서드를 삭제하고 폭포의 첫 번째 단계로 바꾸겠습니다. 이 두 가지 메서드는 사용자에게 인사하고 사용 가능한 옵션 중 하나를 선택하도록 요청했습니다.
   - 여기서는 선택 목록을 볼 수 있으며, 인사말 및 오류 메시지가 선택 프롬프트에 대한 호출에서 옵션으로 제공됩니다.
   - 선택 프롬프트가 완료되면 폭포 대화에서 다음 단계로 계속 진행되므로 대화에서 호출할 다음 메서드를 지정할 필요가 없습니다.
   - 선택 프롬프트는 유효한 입력을 받거나 전체 대화 스택이 취소될 때까지 반복됩니다.  
    [!code-csharp[PromptForOptionsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=51-65)]

1. `OnOptionSelected`를 폭포의 두 번째 단계로 바꿀 수 있습니다. 여전히 사용자의 입력에 따라 자식 대화를 시작합니다.
   - 선택 프롬프트에서 `FoundChoice` 값을 반환합니다. 이 값은 단계 컨텍스트의 `Result` 속성에 표시됩니다. 대화 스택에서는 모든 반환 값을 개체로 처리합니다. 반환 값이 대화 중 하나에서 가져온 것이면 개체의 값 형식을 알 수 있습니다. 각 프롬프트 유형에서 반환하는 목록은 [프롬프트 유형](../bot-builder-concept-dialog.md#prompt-types)을 참조하세요.
   - 선택 프롬프트에서 예외를 throw하지 않으므로 try-catch 블록을 제거할 수 있습니다.
   - 이 메서드는 항상 적절한 값을 반환하도록 fall을 반복적으로 추가해야 합니다. 이 코드는 결코 적중되지 않아야 하지만, 해당 코드가 적중되면 대화가 "정상적으로 실패"할 수 있습니다.  
    [!code-csharp[ShowChildDialogAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=67-102)]

1. 마지막으로, 이전 `ResumeAfterOptionDialog` 메서드를 폭포의 마지막 단계로 바꿉니다.
    - 원래 대화에서 수행한 대로 대화를 종료하고 티켓 번호를 반환하는 대신, 스택에서 원래 인스턴스를 자체의 새 인스턴스로 바꿔 폭포를 다시 시작합니다. 원래 앱에서는 항상 반환 값(티켓 번호)을 무시하고 루트 대화를 다시 시작하므로 이 작업을 수행할 수 있습니다.  
    [!code-csharp[ResumeAfterAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=104-138)]

### <a name="update-the-install-app-dialog"></a>설치 앱 대화 업데이트

설치 앱 대화에서는 4단계 폭포 대화로 설정된 몇 가지 논리적 작업을 수행합니다. 기존 코드를 폭포 단계로 팩터링하는 방법은 각 대화에 대한 논리적 연습입니다. 각 단계마다 코드의 원본이 되는 원래 메서드가 표시됩니다.

1. 사용자에게 검색 문자열을 요청합니다.
1. 데이터베이스에서 잠재적인 일치 항목을 쿼리합니다.
   - 하나의 적중 항목이 있는 경우 이 항목을 선택하고 계속합니다.
   - 여러 개의 적중 항목이 있는 경우 사용자에게 이러한 항목 중 하나를 선택하도록 요청합니다.
   - 적중 항목이 없는 경우 대화가 종료됩니다.
1. 머신에 앱을 설치하도록 사용자에게 요청합니다.
1. 정보를 데이터베이스에 기록하고 확인 메시지를 보냅니다.

**Dialogs/InstallAppDialog.cs** 파일에서,

1. `using` 문을 업데이트합니다.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=4-11)]

1. 수집된 정보를 추적하기 위해 사용할 키에 대해 상수를 정의합니다.  
    [!code-csharp[Key ID](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=17-18)]

1. 생성자를 추가하고 구성 요소의 대화 집합을 초기화합니다.  
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=20-33)]

1. `StartAsync`를 폭포의 첫 번째 단계로 바꿀 수 있습니다.
    - 상태를 직접 관리해야 하므로 대화 상태에서 설치 앱 개체를 추적할 수 있습니다.
    - 사용자에게 입력을 요청하는 메시지는 프롬프트에 대한 호출에서 옵션이 됩니다.  
    [!code-csharp[GetSearchTermAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=35-50)]

1. `appNameAsync` 및 `multipleAppsAsync`를 폭포의 두 번째 단계로 바꿀 수 있습니다.
    - 이제 사용자의 마지막 메시지를 살펴보는 대신 프롬프트 결과를 얻을 수 있습니다.
    - 데이터베이스 쿼리 및 if 문은 `appNameAsync`와 동일하게 구성되었습니다. v4 대화에서 작동하도록 if 문에 있는 각 블록의 코드가 업데이트되었습니다.
        - 하나의 적중 항목이 있으면 대화 상태를 업데이트하고 다음 단계로 계속 진행합니다.
        - 여러 개의 적중 항목이 있으면 선택 프롬프트를 사용하여 옵션 목록에서 선택하도록 사용자에게 요청합니다. 즉, `multipleAppsAsync`를 삭제할 수 있습니다.
        - 적중 항목이 없으면 이 대화를 종료하고 루트 대화에 null을 반환합니다.  
    [!code-csharp[ResolveAppNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=52-91)]

1. 또한 `appNameAsync`는 쿼리를 확인한 후 사용자에게 머신 이름을 요청했습니다. 폭포의 다음 단계에서 논리의 해당 부분을 캡처합니다.
    - 마찬가지로 v4에서 상태를 직접 관리해야 합니다. 여기서 유일하게 까다로운 점은 이전 단계에서 서로 다른 두 개의 논리 분기를 통해 이 단계에 도달할 수 있다는 것입니다.
    - 이번에는 다른 옵션을 제공하기만 하면 이전과 동일한 텍스트 프롬프트를 사용하여 사용자에게 머신 이름을 요청합니다.  
    [!code-csharp[GetMachineNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=93-114)]

1. `machineNameAsync`의 논리는 폭포의 최종 단계에 래핑되었습니다.
    - 텍스트 프롬프트 결과에서 머신 이름을 검색하고 대화 상태를 업데이트합니다.
    - 지원 코드가 다른 프로젝트에 있으므로 데이터베이스에 대한 업데이트 호출을 제거합니다.
    - 그런 다음, 사용자에게 성공 메시지를 보내고 대화를 종료합니다.  
    [!code-csharp[SubmitRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=116-135)]

1. 데이터베이스 호출을 시뮬레이트하기 위해 데이터베이스 대신 정적 목록을 쿼리하도록 `getAppsAsync`를 모의로 만듭니다.  
    [!code-csharp[GetAppsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=137-200)]

### <a name="update-the-local-admin-dialog"></a>로컬 관리자 대화 업데이트

v3에서 이 대화는 사용자에게 인사하고, Formflow 대화를 시작한 다음, 결과를 데이터베이스에 저장합니다. 여기서는 2단계 폭포로 쉽게 변환됩니다.

1. `using` 문을 업데이트합니다. 이 대화에는 v3 Formflow 대화가 포함되어 있습니다. v4에서는 Formflow 커뮤니티 라이브러리를 사용할 수 있습니다.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=4-8)]

1. 대화 상태에서 결과를 사용할 수 있으므로 `LocalAdmin`에 대한 인스턴스 속성을 제거할 수 있습니다.

1. 생성자를 추가하고 구성 요소의 대화 집합을 초기화합니다. Formflow 대화도 동일한 방식으로 만들어집니다. 생성자에 있는 구성 요소의 대화 집합에 추가하기만 하면 됩니다.  
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=14-23)]

1. `StartAsync`를 폭포의 첫 번째 단계로 바꿀 수 있습니다. Formflow는 생성자에 이미 만들었고, 나머지 두 명령문은 이를 해석합니다. `FormBuilder`는 모델의 형식 이름을 이 모델에 대한 `LocalAdminPrompt`인 생성된 대화 상자의 ID로 할당합니다.  
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=25-35)]

1. `ResumeAfterLocalAdminFormDialog`를 폭포의 두 번째 단계로 바꿀 수 있습니다. 인스턴스 속성 대신 단계 컨텍스트에서 반환 값을 가져와야 합니다.  
    [!code-csharp[SaveResultAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=37-50)]

1. `BuildLocalAdminForm`은 Formflow를 통해 인스턴스 속성을 업데이트하지 않는다는 점을 제외하고 대체로 동일하게 유지됩니다.  
    [!code-csharp[BuildLocalAdminForm](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=52-76)]

### <a name="update-the-reset-password-dialog"></a>재설정 암호 대화 업데이트

v3에서 이 대화는 사용자에게 인사하고, 전달 코드를 사용하여 사용자에게 권한을 부여하고, 실패하거나 Formflow 대화를 시작한 다음, 암호를 다시 설정합니다. 이 경우에도 여전히 폭포로 잘 변환됩니다.

1. `using` 문을 업데이트합니다. 이 대화에는 v3 Formflow 대화가 포함되어 있습니다. v4에서는 Formflow 커뮤니티 라이브러리를 사용할 수 있습니다.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=4-9)]

1. 생성자를 추가하고 구성 요소의 대화 집합을 초기화합니다. Formflow 대화도 동일한 방식으로 만들어집니다. 생성자에 있는 구성 요소의 대화 집합에 추가하기만 하면 됩니다.  
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=15-25)]

1. `StartAsync`를 폭포의 첫 번째 단계로 바꿀 수 있습니다. Formflow는 이미 생성자에 만들었습니다. 그렇지 않으면 동일한 논리를 유지하면서 v3 호출을 이와 동등한 v4 호출로 변환할 수 있습니다.  
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=27-45)]

1. `sendPassCode`는 주로 연습으로 남아 있습니다. 원래 코드는 주석 처리되고, 메서드는 true를 반환합니다. 또한 원래 봇에서 사용되지 않았으므로 이메일 주소를 다시 제거할 수 있습니다.  
    [!code-csharp[SendPassCode](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=47-81)]

1. `BuildResetPasswordForm`에는 변경 내용이 없습니다.

1. `ResumeAfterResetPasswordFormDialog`를 폭포의 두 번째 단계로 바꿀 수 있습니다. 단계 컨텍스트에서 반환 값을 가져오겠습니다. 원래 대화에서 아무 작업도 수행하지 않은 이메일 주소를 제거하고, 데이터베이스를 쿼리하는 대신 더미 결과를 제공했습니다. 동일한 논리를 유지하면서 v3 호출을 이와 동등한 v4 호출로 변환할 수 있습니다.  
    [!code-csharp[ProcessRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=90-113)]

### <a name="update-models-as-necessary"></a>필요에 따라 모델 업데이트

Formflow 라이브러리를 참조하는 일부 모델에서 `using` 문을 업데이트해야 합니다.

1. `LocalAdminPrompt`에서 다음으로 변경합니다.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Models/LocalAdminPrompt.cs?range=4)]

1. `ResetPasswordPrompt`에서 다음으로 변경합니다.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetFramework/ContosoHelpdeskChatBot/Models/ResetPasswordPrompt.cs?range=4-5)]

## <a name="update-webconfig"></a>Web.config 업데이트

**MicrosoftAppId** 및 **MicrosoftAppPassword**에 대한 구성 키를 주석 처리합니다. 이렇게 하면 에뮬레이터에 이러한 값을 제공하지 않고도 봇을 로컬로 디버그할 수 있습니다.

## <a name="run-and-test-your-bot-in-the-emulator"></a>에뮬레이터에서 봇 실행 및 테스트

이 시점에서 봇을 IIS에서 로컬로 실행하고 에뮬레이터를 사용하여 연결할 수 있어야 합니다.

1. IIS에서 봇을 실행합니다.
1. 에뮬레이터를 시작하고, 봇의 엔드포인트(예: **http://localhost:3978/api/messages** )에 연결합니다.
    - 처음으로 봇을 실행하는 경우라면 **파일 > 새 봇**을 클릭하고 화면에 표시된 지시를 따릅니다. 그렇지 않은 경우 **파일 > 봇 열기**를 클릭하여 기존 봇을 엽니다.
    - 구성에서 포트 설정을 다시 확인합니다. 예를 들어 봇이 브라우저에서 `http://localhost:3979/`로 열렸으면 에뮬레이터에서 봇의 엔드포인트를 `http://localhost:3979/api/messages`로 설정합니다.
1. 네 개의 대화가 모두 작동해야 하고, 폭포 단계에서 중단점을 설정하여 대화 컨텍스트 및 대화 상태가 이러한 지점에 있는지 확인할 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

v4 개념 항목:

- [봇 작동 방식](../bot-builder-basics.md)
- [상태 관리](../bot-builder-concept-state.md)
- [다이얼로그 라이브러리](../bot-builder-concept-dialog.md)

v4 방법 항목:

- [문자 메시지 보내기 및 받기](../bot-builder-howto-send-messages.md)
- [사용자 및 대화 데이터 저장](../bot-builder-howto-v4-state.md)
- [순차적 대화 흐름 구현](../bot-builder-dialog-manage-conversation-flow.md)
- [에뮬레이터를 사용하여 디버그](../../bot-service-debug-emulator.md)
- [봇에 원격 분석 추가](../bot-builder-telemetry.md)
