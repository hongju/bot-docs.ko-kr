---
title: 대화 상자 개요 | Microsoft Docs
description: .NET용 Bot Builder SDK 내의 대화 상자를 사용하여 대화를 모델링하고 대화 흐름을 관리하는 방법에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 943b206e4991c52f22928d2113977249ff9d9e04
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997580"
---
# <a name="dialogs-in-the-bot-builder-sdk-for-net"></a>.NET용 Bot Builder SDK의 대화 상자

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-dialog-overview.md)

.NET용 Bot Builder SDK를 사용하는 봇을 만들 경우 대화 상자를 사용하여 대화를 모델링하고 [대화 흐름](../bot-service-design-conversation-flow.md)을 관리할 수 있습니다. 각 대화 상자는 `IDialog`를 구현하는 C# 클래스에서 자체 상태를 캡슐화하는 추상화입니다. 대화 상자를 다른 대화 상자로 구성하여 재사용을 최대화할 수 있고 대화 상자 컨텍스트는 특정 시점에 대화에서 활성 상태인 [대화 상자 스택](../bot-service-design-conversation-flow.md#dialog-stack)을 유지 관리할 수 있습니다. 

대화 상자를 구성하는 대화는 컴퓨터 간에 이동 가능하므로 봇 구현을 확장할 수 있습니다. .NET용 Bot Builder SDK의 대화 상자를 사용하면 대화 상태(대화 상자 스택 및 스택에 있는 각 대화 상자의 상태)가 선택된 [상태 데이터](bot-builder-dotnet-state.md) 저장소에 자동으로 저장됩니다. 따라서 웹 서버 메모리에 세션 상태를 저장할 필요가 없는 웹 애플리케이션처럼 봇의 서비스 코드를 상태 비저장으로 유지할 수 있습니다. 

## <a name="echo-bot-example"></a>Echo 봇 예제

[빠른 시작](bot-builder-dotnet-quickstart.md) 자습서에서 생성된 봇을 변경하여 대화 상자를 통해 사용자와 메시지를 교환하는 방법을 설명하는 이 Echo 봇 예제를 살펴봅니다.

> [!TIP]
> 이 예제를 따라 진행하려면 [빠른 시작](bot-builder-dotnet-quickstart.md) 자습서의 지침에 따라 봇을 만든 다음, 아래 설명된 대로 해당 **MessagesController.cs** 파일을 업데이트합니다.

### <a name="messagescontrollercs"></a>MessagesController.cs 

.NET용 Bot Builder SDK에서 [Builder][builderLibrary] 라이브러리를 사용하여 대화 상자를 구현할 수 있습니다. 관련 클래스에 액세스하려면 `Dialogs` 네임스페이스를 가져옵니다.

[!code-csharp[Using statement](../includes/code/dotnet-dialogs.cs#usingStatement)]

다음으로, 이 `EchoDialog` 클래스를 **MessagesController.cs**에 추가하여 대화를 나타냅니다. 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot1)]

그런 다음, `Conversation.SendAsync` 메서드를 호출하여 `EchoDialog` 클래스를 `Post` 메서드에 연결합니다.

[!code-csharp[Post method](../includes/code/dotnet-dialogs.cs#echobot2)]

### <a name="implementation-details"></a>구현 세부 정보 

Bot Builder가 C# 기능을 사용하여 비동기 통신을 처리하므로, `Post` 메서드가 `async`로 표시됩니다. 전달된 메시지에 응답을 보내는 작업을 나타내는 `Task` 개체를 반환합니다. 예외가 있는 경우 메서드에서 반환되는 `Task`에 예외 정보가 포함됩니다. 

`Conversation.SendAsync` 메서드는 .NET용 Bot Builder SDK를 사용하여 대화 상자를 구현하는 키입니다. <a href="https://en.wikipedia.org/wiki/Dependency_inversion_principle" target="_blank">종속성 반전 원칙</a>을 따르고 다음 단계를 수행합니다.

1. 필수 구성 요소 인스턴스화  
2. `IBotDataStore`에서 대화 상태(대화 상자 스택 및 스택에 있는 각 대화 상자의 상태) 역직렬화
3. 봇이 일시 중단되고 메시지를 기다리는 대화 프로세스 다시 시작
4. 응답 보내기
5. 업데이트된 대화 상태를 직렬화하여 `IBotDataStore`에 다시 저장

대화가 처음 시작될 때 대화 상자에 상태가 포함되어 있지 않으므로, `Conversation.SendAsync`는 `EchoDialog`를 구성하고 `StartAsync` 메서드를 호출합니다. `StartAsync` 메서드는 연속 대리자와 함께 `IDialogContext.Wait`를 호출하여 새 메시지가 수신될 때 호출해야 하는 메서드를 지정합니다(`MessageReceivedAsync`). 

`MessageReceivedAsync` 메서드는 메시지를 기다리고, 응답을 게시하며, 다음 메시지를 기다립니다. `IDialogContext.Wait`가 호출될 때마다 봇은 일시 중단된 상태로 전환되어 메시지를 수신하는 모든 컴퓨터에서 다시 시작될 수 있습니다. 

위의 코드 샘플을 사용하여 만든 봇은 사용자의 메시지 앞에 'You said:’라는 텍스트를 붙여 간단히 회신하는 방식으로 사용자가 보내는 각 메시지에 응답합니다. 봇은 대화 상자를 사용하여 생성되기 때문에 명시적으로 상태를 관리하지 않고도 더 복잡한 대화를 지원하도록 발전시킬 수 있습니다.

## <a name="echo-bot-with-state-example"></a>상태를 저장하는 Echo 봇 예제

다음 예제는 위의 예제에 대화 상자 상태를 추적하는 기능을 추가한 것입니다. 아래 코드 샘플에 표시된 것처럼, `EchoDialog` 클래스가 업데이트되면 봇은 사용자의 메시지 앞에 번호(`count`) 다음에 'You said:'라는 텍스트를 붙여서 회신하는 방식으로 사용자가 보내는 각 메시지에 응답합니다. 사용자가 카운트를 다시 설정하도록 선택할 때까지 봇은 매 회신 시 `count`를 계속 증분합니다.

### <a name="messagescontrollercs"></a>MessagesController.cs 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot3)]

### <a name="implementation-details"></a>구현 세부 정보

첫 번째 예제와 같이 새 메시지가 수신되면 `MessageReceivedAsync` 메서드가 호출됩니다. 그러나 이 경우 `MessageReceivedAsync` 메서드는 응답하기 전에 사용자의 메시지를 평가합니다. 사용자의 메시지가 "다시 설정"되면 기본 제공 `PromptDialog.Confirm` 프롬프트는 사용자에게 카운트 재설정을 확인하도록 요청하는 하위 대화 상자를 생성합니다. 하위 대화 상자에는 부모 대화 상자의 상태를 방해하지 않는 고유한 개인 상태가 있습니다. 사용자가 프롬프트에 응답하면 하위 대화 상자의 결과가 `AfterResetAsync` 메서드로 전달됩니다. 이 메서드는 카운트가 재설정되었는지 여부를 나타내는 메시지를 사용자에게 보낸 후 다음 메시지에서 `MessageReceivedAsync`로 다시 이어지는 `IDialogContext.Wait`를 호출합니다.

## <a name="dialog-context"></a>대화 상자 컨텍스트

각 대화 상자 메서드에 전달되는 `IDialogContext` 인터페이스는 대화 상자가 상태를 저장하고 채널과 통신하는 데 필요한 서비스에 액세스할 수 있게 해줍니다. `IDialogContext` 인터페이스는 [Internals.IBotData][iBotData], [Internals.IBotToUser][iBotToUser] 및 [ Internals.IDialogStack][iDialogStack] 등의 세 인터페이스로 구성됩니다. 

### <a name="internalsibotdata"></a>Internals.IBotData

`Internals.IBotData` 인터페이스는 커넥터에서 유지 관리되는 사용자별, 대화별 및 개인 대화 상태 데이터에 대한 액세스를 제공합니다. 사용자별 상태 데이터는 특정 대화와 관련이 없는 사용자 데이터를 저장하는 데 유용하고, 대화별 데이터는 대화에 대한 일반 데이터를 저장하는 데 유용하며, 개인 대화 데이터는 특정 대화와 관련된 사용자 데이터를 저장하는 데 유용합니다. 

### <a name="internalsibottouser"></a>Internals.IBotToUser

`Internals.IBotToUser`는 봇에서 사용자에게 메시지를 보내는 데 필요한 메서드를 제공합니다. 메시지는 웹 API 메서드 호출에 대한 응답을 사용하여 인라인으로 전송되거나 [커넥터 클라이언트](bot-builder-dotnet-connector.md#create-a-connector-client)를 사용하여 직접 전송될 수 있습니다. 대화 상자 컨텍스트를 통해 메시지를 전송하고 수신하면 `Internals.IBotData` 상태가 커넥터를 통해 전달됩니다.

### <a name="internalsidialogstack"></a>Internals.IDialogStack

`Internals.IDialogStack`은 [대화 상자 스택](../bot-service-design-conversation-flow.md#dialog-stack)을 관리하는 데 필요한 메서드를 제공합니다. 대부분의 경우 대화 상자 스택이 자동으로 관리됩니다. 그러나 스택을 명시적으로 관리하려는 경우가 있을 수 있습니다. 예를 들어, 자식 대화 상자를 호출하여 대화 상자 스택의 맨 위에 추가하거나, 현재 대화 상자를 완료 상태로 표시하여 대화 상자 스택에서 제거하고 스택의 이전 대화 상자로 결과를 반환하거나, 사용자로부터 메시지가 도착할 때까지 현재 대화 상자를 일시 중단하거나, 대화 상자 스택을 모두 다시 설정할 수도 있습니다.

## <a name="serialization"></a>직렬화

대화 상자 스택과 모든 활성 대화 상자의 상태가 사용자별, 대화별 [IBotDataBag][iBotDataBag]으로 직렬화됩니다. 직렬화된 BLOB은 봇이 [커넥터](bot-builder-dotnet-concepts.md#connector)와 주고받는 메시지에서 유지됩니다. 직렬화하려면 `Dialog` 클래스에 `[Serializable]` 특성이 포함되어야 합니다. [Builder][builderLibrary] 라이브러리의 모든 `IDialog` 구현은 직렬화 가능한 것으로 표시됩니다. 

[체인 메서드](#dialog-chains)는 LINQ 쿼리 구문에서 사용할 수 있는 대화 상자에 흐름 인터페이스를 제공합니다. LINQ 쿼리 구문의 컴파일된 형식은 종종 무명 메서드를 사용합니다. 이러한 무명 메서드가 로컬 변수 환경을 참조하지 않는 경우 이러한 무명 메서드는 상태가 없고 일반적으로 직렬화됩니다. 그러나 무명 메서드가 환경의 모든 로컬 변수를 캡처하는 경우 결과 closure 개체(컴파일러에서 생성됨)는 직렬화 가능한 것으로 표시되지 않습니다. 이 경우 Bot Builder는 문제를 식별하기 위해 `ClosureCaptureException`을 throw합니다.

리플렉션을 사용하여 직렬화 가능한 것으로 표시되지 않는 클래스를 직렬화하기 위해 Builder 라이브러리에는 [Autofac][autofac]에 등록하는 데 사용할 수 있는 리플렉션 기반 직렬화 서로게이트가 포함됩니다.

[!code-csharp[Serialization](../includes/code/dotnet-dialogs.cs#serialization)]

## <a id="dialog-chains"></a> 대화 상자 체인

`IDialogStack.Call<R>` 및 `IDialogStack.Done<R>`을 사용하여 활성 대화 상자 스택을 명시적으로 관리할 수 있지만, 이러한 흐름 [체인][chain] 메서드를 사용하여 활성 대화 상자 스택을 암시적으로 관리할 수도 있습니다.


|           방법            |  type   |                                 메모                                  |
|-----------------------------|---------|------------------------------------------------------------------------|
|     Chain.Select<T, R>      |  LINQ   |           LINQ 쿼리 구문에서 "select" 및 "let"을 지원합니다.            |
|  Chain.SelectMany<T, C, R>  |  LINQ   |            LINQ 쿼리 구문에서 연속적인 "from"을 지원합니다.            |
|       Chain.Where<T>        |  LINQ   |                 LINQ 쿼리 구문에서 "where"를 지원합니다.                 |
|        Chain.From<T>        | Chains  |                대화 상자의 새 인스턴스를 인스턴스화합니다.                |
|       Chain.Return<T>       | Chains  |                체인에 상수 값을 반환합니다.                |
|         Chain.Do<T>         | Chains  |               체인 내 부작용을 허용합니다.                |
|  Chain.ContinueWith<T, R>   | Chains  |                      대화 상자의 간단한 연결입니다.                       |
|       Chain.Unwrap<T>       | Chains  |                  대화 상자에 중첩된 대화 상자를 래핑 해제합니다.                   |
| Chain.DefaultIfException<T> | Chains  | 이전 결과에서 예외를 무효화하고 기본값(T)을 반환합니다. |
|        Chain.Loop<T>        | Branch  |                   대화 상자의 전체 체인을 반복합니다.                   |
|        Chain.Fold<T>        | Branch  |   대화 상자의 열거형 결과를 접어서 단일 결과를 만듭니다.   |
|     Chain.Switch<T, R>      | Branch  |            여러 대화 상자 체인으로의 분기를 지원합니다.            |
|     Chain.PostToUser<T>     | Message |                      사용자에게 메시지를 게시합니다.                      |
|     Chain.WaitToBot<T>      | Message |                    봇에 대한 메시지를 기다립니다.                     |
|    Chain.PostToChain<T>     | Message |              사용자의 메시지로 체인을 시작합니다.              |

### <a name="examples"></a>예 

LINQ 쿼리 구문은 `Chain.Select<T, R>` 메서드를 사용합니다.

[!code-csharp[Chain.Select](../includes/code/dotnet-dialogs.cs#chain1)]

또는 `Chain.SelectMany<T, C, R>` 메서드를 사용합니다.

[!code-csharp[Chain.SelectMany](../includes/code/dotnet-dialogs.cs#chain2)]

`Chain.PostToUser<T>` 및 `Chain.WaitToBot<T>` 메서드는 봇의 메시지를 사용자에게 게시하거나 그 반대로 게시합니다.

[!code-csharp[Chain.PostToUser](../includes/code/dotnet-dialogs.cs#chain3)]

`Chain.Switch<T, R>` 메서드는 대화 상자 흐름을 분기합니다.

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain4)]

`Chain.Switch<T, R>`가 중첩된 `IDialog<IDialog<T>>`를 반환하면 내부 `IDialog<T>`를 `Chain.Unwrap<T>`로 래핑 해제할 수 있습니다. 이렇게 하면 길이가 다른 연결 대화 상자의 다른 경로로 대화를 분기할 수 있습니다. 이 예제에서는 암시적 스택 관리를 사용하여 흐름 체인 스타일로 작성된 대화 상자를 분기하는 방법에 대한 보다 완전한 예제를 보여 줍니다.

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain5)]

## <a name="next-steps"></a>다음 단계

대화 상자는 봇과 사용자 간의 대화 흐름을 관리합니다. 대화 상자는 사용자와 상호 작용하는 방법을 정의합니다. 봇은 스택으로 구성된 많은 대화 상자를 사용하여 사용자와의 대화를 안내할 수 있습니다. 다음 섹션에서는 스택의 대화 상자를 만들고 해제할 때 대화 상자 스택이 어떻게 확장되고 축소되는지 볼 수 있습니다.

> [!div class="nextstepaction"]
> [대화 상자를 사용하여 대화 흐름 관리](bot-builder-dotnet-manage-conversation-flow.md)


[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs

[iBotData]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibotdata

[iBotToUser]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibottouser

[iDialogStack]: /dotnet/api/microsoft.bot.builder.dialogs.internals.idialogstack

[iBotDataBag]: /dotnet/api/microsoft.bot.builder.dialogs.ibotdatabag

[autofac]: /dotnet/api/microsoft.bot.builder.autofac.base

[chain]: /dotnet/api/microsoft.bot.builder.dialogs.chain
