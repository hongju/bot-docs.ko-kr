---
title: 입력에 단추 사용 | Microsoft Docs
description: JavaScript용 Bot Builder SDK를 사용하여 메시지 내에서 제안된 동작을 전송하는 방법을 알아봅니다.
keywords: 제안된 동작, 단추, 추가 입력
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 10b9fa9664e8c18cdc5dcd2fcf3ae400296a4abb
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/28/2018
ms.locfileid: "52451985"
---
# <a name="use-button-for-input"></a>입력에 단추 사용

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

사용자가 입력하기 위해 탭할 수 있는 단추를 봇이 표시하도록 할 수 있습니다. 단추는 사용자가 키보드를 사용하여 응답을 입력하는 대신 질문에 대답하거나 간단히 단추를 탭하여 선택할 수 있도록 하여 사용자 환경을 개선합니다. 제안된 동작 창 내에 표시되는 단추는 서식 있는 카드 내에 표시되는 단추와 달리(탭한 후에도 사용자에게 표시되고 액세스 가능함) 사용자가 선택한 후에 사라집니다. 따라서 사용자가 대화 내에서 유효하지 않은 단추를 탭하지 않게 되며, 봇 개발이 간소화됩니다(해당 시나리오를 고려할 필요가 없으므로). 

## <a name="suggest-action-using-button"></a>단추를 사용하는 작업 제안

*제안된 작업*을 사용하면 봇이 단추를 표시할 수 있습니다. 단일-턴(single-turn) 대화에서 사용자에게 표시될 제안된 동작(다른 이름: “빠른 회신”) 목록을 만들 수 있습니다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

여기서 사용하는 소스 코드는 [GitHub](https://aka.ms/SuggestedActionsCSharp)에서 액세스할 수 있습니다.

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

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
