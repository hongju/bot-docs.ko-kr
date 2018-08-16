---
title: 메시지에 제안된 동작 추가 | Microsoft Docs
description: JavaScript용 Bot Builder SDK를 사용하여 메시지 내에서 제안된 동작을 전송하는 방법을 알아봅니다.
keywords: 제안된 동작, 단추, 추가 입력
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 802d245112baa6d5fb10f4e992d9f6722b70a224
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303834"
---
# <a name="add-suggested-actions-to-messages"></a>메시지에 제안된 동작 추가

[!include[Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)] 

## <a name="send-suggested-actions"></a>제안된 동작 보내기

단일-턴(single-turn) 대화에서 사용자에게 표시될 제안된 동작(다른 이름: “빠른 회신”) 목록을 만들 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and add suggested actions.
var activity = MessageFactory.SuggestedActions(
    new CardAction[]
    {
        new CardAction(title: "red", type: ActionTypes.ImBack, value: "red"),
        new CardAction( title: "green", type: ActionTypes.ImBack, value: "green"),
        new CardAction(title: "blue", type: ActionTypes.ImBack, value: "blue")
    }, text: "Choose a color");

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Require MessageFactory from botbuilder.
const {MessageFactory} = require('botbuilder');

//  Initialize the message object.
const basicMessage = MessageFactory.suggestedActions(['red', 'green', 'blue'], 'Choose a color');

await context.sendActivity(basicMessage);
```

---

## <a name="additional-resources"></a>추가 리소스

기능을 미리 보려면 [채널 검사기](../bot-service-channel-inspector.md)를 사용해보세요.
