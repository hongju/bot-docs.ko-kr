---
title: 메시지에 입력 힌트 추가 | Microsoft Docs
description: .NET용 Bot Builder SDK를 사용하여 메시지에 입력 힌트를 추가하는 방법에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a88461e8a03ac941671cc78080e38efecc26aa30
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998400"
---
# <a name="add-input-hints-to-messages"></a>메시지에 입력 힌트 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-add-input-hints.md)

메시지에 대한 입력 힌트를 지정하여 메시지를 클라이언트에 전달한 후 봇이 사용자 입력을 허용, 필요 또는 무시하는지 여부를 나타낼 수 있습니다. 많은 채널에서 이 기능을 사용하여 클라이언트가 사용자 입력 컨트롤 상태를 적절히 설정할 수 있습니다. 예를 들어 메시지의 입력 힌트가 봇이 사용자 입력을 무시하고 있는 것으로 표시하면 클라이언트는 마이크를 닫고 사용자가 입력을 제공하지 못하도록 입력 상자를 비활성화할 수 있습니다.

## <a name="accepting-input"></a>입력 허용

봇이 입력에 대해 수동적으로 준비하지만 사용자의 응답을 기다리지 않고 있음을 나타내려면 메시지의 입력 힌트를 `InputHints.AcceptingInput`으로 설정합니다. 많은 채널에서 이렇게 하면 클라이언트의 입력 상자가 활성화되고 마이크는 닫히지만 사용자는 계속 액세스할 수 있습니다. 예를 들어 사용자가 마이크 단추를 누르고 있으면 Cortana는 마이크를 열고 사용자의 입력을 허용합니다. 다음 예제 코드는 봇이 사용자 입력을 수락한다는 것을 나타내는 메시지를 만듭니다.

```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.AcceptingInput;
await connector.Conversations.ReplyToActivityAsync(reply);
```

## <a name="expecting-input"></a>입력 필요

봇이 사용자의 응답을 기다리고 있다고 나타내려면 메시지의 입력 힌트를 `InputHints.ExpectingInput`으로 설정합니다. 많은 채널에서 이렇게 하면 클라이언트의 입력 상자가 활성화되고 마이크가 열립니다. 다음 예제 코드는 봇이 사용자 입력을 기다리고 있음을 나타내는 메시지를 만듭니다.

```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.ExpectingInput;
await connector.Conversations.ReplyToActivityAsync(reply);
```

## <a name="ignoring-input"></a>입력 무시

봇이 사용자의 입력을 받을 준비가 되어 있지 않다고 나타내려면 메시지의 입력 힌트를 `InputHints.IgnorningInput`로 설정합니다. 많은 채널에서 이렇게 하면 클라이언트의 입력 상자가 비활성화되고 마이크가 닫힙니다. 다음 예제 코드는 봇이 사용자 입력을 무시하고 있는 것을 나타내는 메시지를 만듭니다.

```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.IgnoringInput;
await connector.Conversations.ReplyToActivityAsync(reply);
```

## <a name="default-values-for-input-hint"></a>입력 힌트에 대한 기본값

메시지에 대한 입력 힌트를 설정하지 않은 경우 Bot Builder SDK가 이 논리를 사용하여 자동으로 입력 힌트를 설정해줍니다.

- 봇이 프롬프트를 보내면, 메시지에 대한 입력 힌트가 봇에 **입력이 필요**하다는 것을 지정합니다.</li>
- 봇이 단일 메시지를 보내는 경우 메시지에 대한 입력 힌트는 봇이 **입력을 허용**하고 있음을 지정합니다.</li>
- 봇이 일련의 연속 메시지를 보내는 경우 그 중 최종 메시지를 제외한 모든 메시지에 대한 입력 힌트는 봇이 **입력을 무시**하고 있음을 지정하고 최종 메시지에 대한 입력 힌트는 **입력을 허용**하고 있음을 지정합니다.

## <a name="additional-resources"></a>추가 리소스

- [메시지 만들기](bot-builder-dotnet-create-messages.md)
- [메시지에 음성 추가](bot-builder-dotnet-text-to-speech.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">활동 클래스</a>
- <a href="/dotnet/api/microsoft.bot.connector.inputhints" target="_blank">InputHints 클래스</a>
