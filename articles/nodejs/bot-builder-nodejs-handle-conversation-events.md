---
title: 사용자 및 대화 이벤트 처리 | Microsoft Docs
description: Node.js용 Bot Builder SDK를 사용하여 사용자의 대화 참여와 같은 이벤트를 처리하는 방법을 알아봅니다.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 23cb3189de10a67a524114d0c7d9f1f9d2b0441d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301939"
---
# <a name="handle-user-and-conversation-events"></a>사용자 및 대화 이벤트 처리

이 문서에서는 봇이 사용자의 대화 참여, 연락처 목록에 봇 추가 또는 봇이 대화에서 젝저되고 있을 때 **Goodbye**라고 말하는 것과 같은 이벤트를 처리하는 방법을 보여 줍니다.


## <a name="greet-a-user-on-conversation-join"></a>대화 참여 시 사용자에게 인사말 제공
Bot Framework는 멤버가 대화에 참여하거나 대화에서 나갈 때마다 봇에 알리기 위한 [conversationUpdate][conversationUpdate] 이벤트를 제공합니다. 대화 멤버는 사용자이거나 봇일 수 있습니다.

다음 코드 조각은 봇이 대화의 새 멤버에게 인사말을 하거나 대화에서 나갈 때 **Goodbye**라고 말하도록 합니다.

[!INCLUDE [conversationUpdate sample Node.js](../includes/snippet-code-node-conversationupdate-1.md)]

## <a name="acknowledge-add-to-contacts-list"></a>연락처 목록에 대한 추가 승인

[contactRelationUpdate][contactRelationUpdate] 이벤트는 사용자가 여러분을 연락처 목록에 추가했을 때 봇에게 알립니다.

[!INCLUDE [contactRelationUpdate sample Node.js](../includes/snippet-code-node-contactrelationupdate-1.md)]

## <a name="add-a-first-run-dialog"></a>처음 실행 다이얼로그 추가

**conversationUpdate** 및 **contactRelationUpdate** 이벤트가 모든 채널에서 지원되는 것은 아니므로, 대화에 참여하는 사용자에게 인사말을 하는 범용 방법은 처음 실행 다이얼로그를 추가하는 것입니다.

다음 예제에서는 이전에 본 적이 없는 사용자를 만날 때마다 다이얼로그를 트리거하는 함수를 추가했습니다. 작업에 대해 [onFindAction][onFindAction] 처리기를 제공하여 작업이 트리거되는 방식을 사용자 지정할 수 있습니다. 

[!INCLUDE [first-run sample Node.js](../includes/snippet-code-node-first-run-dialog-1.md)]

[onSelectAction][onSelectAction] 처리기를 제공하여 트리거된 후에 작업이 수행하는 동작을 사용자 지정할 수도 있습니다. 트리거 작업에 대해 [onInterrupted][onInterrupted] 처리기를 제공하여 중단이 발생하기 전에 가로챌 수 있습니다. 자세한 내용은 [사용자 작업 처리](bot-builder-nodejs-dialog-actions.md)를 참조하세요.

## <a name="additional-resources"></a>추가 리소스

* [conversationUpdate][conversationUpdate]
* [contactRelationUpdate][contactRelationUpdate]

[conversationUpdate]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iconversationupdate.html
[contactRelationUpdate]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icontactrelationupdate.html

[onFindAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onfindaction
[onSelectAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onselectaction
[onInterrupted]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#oninterrupted

[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
