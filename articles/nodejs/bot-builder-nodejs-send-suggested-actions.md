---
title: 메시지에 제안된 동작 추가 | Microsoft Docs
description: Node.js용 Bot Framework SDK를 사용하여 메시지 내에서 제안된 동작을 전송하는 방법을 알아봅니다.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 73193b82c8f03a1a49df1927ced15684bd7af955
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "64564156"
---
# <a name="add-suggested-actions-to-messages"></a>메시지에 제안된 동작 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

## <a name="suggested-actions-example"></a>제안된 작업 예제

제안된 작업을 메시지를 추가하려면 메시지의 `suggestedActions` 속성을 사용자에게 제공될 단추를 나타내는 [작업 카드][ICardAction] 목록으로 설정합니다.

이 코드 예제에서는 세 가지 제안된 작업을 사용자에게 제공하는 메시지 전송 방법을 보여 줍니다.

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

사용자가 제안된 작업 중 하나를 탭하면 봇은 해당 작업의 `value`를 포함하는 메시지를 사용자로부터 수신합니다.

`imBack` 메서드는 사용 중인 채널의 채팅 창에 `value`를 게시합니다. 원하는 결과가 아니면 `postBack` 메서드를 사용할 수 있습니다. 이 메서드는 선택 항목을 봇에 계속 게시하지만 채팅 창에는 표시하지 않습니다. 일부 채널은 `postBack`을 지원하지 않지만 해당 경우에 이 메서드는 `imBack`과 같이 동작합니다.

## <a name="additional-resources"></a>추가 리소스

- [샘플][samples]
- [IMessage][IMessage]
- [ICardAction][ICardAction]
- [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

<!-- The inspector is no longer supported: we're redirecting to the samples for now. -->
[samples]: https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples
