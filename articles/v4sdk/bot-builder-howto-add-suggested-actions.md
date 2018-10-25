---
title: 메시지에 제안된 동작 추가 | Microsoft Docs
description: JavaScript용 Bot Builder SDK를 사용하여 메시지 내에서 제안된 동작을 전송하는 방법을 알아봅니다.
keywords: 제안된 동작, 단추, 추가 입력
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 88186d3c6c925220fba099a5983c86b305f2dcae
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997103"
---
# <a name="add-suggested-actions-to-messages"></a>메시지에 제안된 동작 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

[!include[Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)] 

## <a name="send-suggested-actions"></a>제안된 동작 보내기

단일-턴(single-turn) 대화에서 사용자에게 표시될 제안된 동작(다른 이름: “빠른 회신”) 목록을 만들 수 있습니다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

여기서 사용하는 소스 코드는 [GitHub](https://aka.ms/SuggestedActionsCSharp)에서 액세스할 수 있습니다.

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

```csharp
var reply = turnContext.Activity.CreateReply("What is your favorite color?");

reply.SuggestedActions = new SuggestedActions()
{
    Actions = new List<CardAction>()
    {
        new CardAction() { Title = "Red", Type = ActionTypes.ImBack, Value = "Red" },
        new CardAction() { Title = "Yellow", Type = ActionTypes.ImBack, Value = "Yellow" },
        new CardAction() { Title = "Blue", Type = ActionTypes.ImBack, Value = "Blue" },
    },

};
await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
여기서 사용하는 소스 코드는 [GitHub](https://aka.ms/SuggestActionsJS)에서 액세스할 수 있습니다.

```javascript
const { ActivityTypes, MessageFactory, TurnContext } = require('botbuilder');

async sendSuggestedActions(turnContext) {
    var reply = MessageFactory.suggestedActions(['Red', 'Yellow', 'Blue'], 'What is the best color?');
    await turnContext.sendActivity(reply);
}
```

---

## <a name="additional-resources"></a>추가 리소스

여기서 보여 주는 전체 소스 코드는 [[C#](https://aka.ms/SuggestedActionsCSharp) | [JS](https://aka.ms/SuggestActionsJS)] GitHub에서 액세스할 수 있습니다.
