---
title: 이벤트 처리기 사용 | Microsoft Docs
description: 이벤트 처리기를 사용하는 방법을 살펴봅니다.
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/14/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 36bd2638c599de1662a37dd85790b6126184d51b
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905692"
---
# <a name="using-event-handlers"></a>이벤트 처리기 사용

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

이벤트 처리기는 [회전](bot-builder-basics.md#defining-a-turn) 내에서 미래 활동 이벤트에 추가할 수 있는 함수입니다. 해당 활동은 `SendActivity`, `UpdateActivity` 및 `DeleteActivity`이며 각 활동에는 자체 처리기가 포함됩니다. 이러한 처리기는 현재 컨텍스트 개체에 대한 해당 형식의 모든 미래 활동에서 어떤 것을 수행해야 할 경우 유용합니다.

각 활동 이벤트에 여러 이벤트 처리기를 추가할 수 있고, 해당 처리기는 추가된 **후** 모든 이벤트를 처리하므로 코드에서 처리기를 추가하는 위치가 중요합니다.

> [!NOTE]
> 이 문서의 ‘이벤트 처리기’는 ‘이벤트’ 및 ‘처리기’의 기존 언어 관련 정의를 따르지 않습니다. 대신에 ‘이벤트’ 및 ‘처리기’의 더 개념적인 프로그래밍 아이디어를 나타냅니다.

## <a name="using-the-next-delegate"></a>*next* 대리자 사용

*next* 대리자를 호출하면 각 처리기는 처리기가 추가된 순서로 다음 처리기에 실행을 전달합니다. 마지막 *next* 대리자는 실제로 활동을 전송, 업데이트 또는 삭제하기 위한 어댑터에 대한 호출입니다.

다른 의도가 있는 것이 아니면 처리기 내에서 *next()* 를 호출하는 것이 중요합니다. 이렇게 하지 않으면 활동이 [단락](bot-builder-create-middleware.md#short-circuit-routing)됩니다. 활동 단락과 미들웨어의 차이점은 단락은 활동을 완전히 취소하는 반면, 미들웨어는 실행할 수 있는 코드를 변경하지만 활동 실행이 계속된다는 것입니다.

보내기 활동에 성공하면 *next* 대리자는 보낸 메시지에 할당된 ID를 반환합니다.

## <a name="expected-return-value"></a>예상 반환 값

이벤트 처리기는 반환 값을 next 대리자의 결과로 예상합니다. 필요한 경우, 해당 결과는 반환되기 전에 보고 저장할 수도 있고, 아래 표시된 대로 직접 반환될 수도 있습니다.

`SendActivity` 처리기의 반환 값은 일치하는 next 대리자의 반환 값으로 체인을 통해 다시 전달되는 반환 값입니다.

# <a name="ctabcseventhandler"></a>[C#](#tab/cseventhandler)

세 가지 형식의 이벤트 처리기인 `OnSendActivities()`, `OnUpdateActivity()` 및 `OnDeleteActivity()`가 제공됩니다. 여기서는 봇 코드 내에 `OnSendActivities()` 처리기를 추가하지만 처리기는 미들웨어 코드에도 추가될 수 있습니다.

이 예제에서 확인할 수 있듯이 처리기는 종종 람다 식으로 추가됩니다. 여기서는 **도움말**이 작성될 때 사용자의 입력 활동을 수신 대기합니다.

```cs
public Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        
        context.OnSendActivities(async (handlerContext, activities, handlerNext) =>
        {
            if (handlerContext.Activity.Text == "help")
            {
                Console.WriteLine("help!");
                // Do whatever logging you want to do for this help message
            }

            return await handlerNext();
        });
        // ...
    }
}
```

# <a name="javascripttabjseventhandler"></a>[JavaScript](#tab/jseventhandler)

세 가지 형식의 이벤트 처리기인 `onSendActivities()`, `onUpdateActivity()` 및 `onDeleteActivity()`가 제공됩니다. 여기서는 봇 코드 내에 `onSendActivities()` 처리기를 추가하지만 처리기는 미들웨어 코드에도 추가될 수 있습니다.

이 예제에서는 **도움말**이 작성될 때 사용자의 입력 활동을 수신 대기합니다.

```js
adapter.processActivity(req, res, async (context) => {

    if (context.activity.type === 'message') {

        context.onSendActivities(async (handlerContext, activities, handlerNext) => { 
            
            if(handlerContext.activity.text === 'help'){
                console.log('help!')
                // Do whatever logging you want to do for this help message
            }
            // Add handler logic here
        
            await handlerNext(); 
        });
        await context.sendActivity(`you said ${context.activity.text}`);
    }
});
```

---

‘보내기 활동’ 및 ‘업데이트 또는 삭제 활동’ 이벤트를 구별해야 합니다. 이러한 이벤트에서는 먼저 전체적으로 새 활동 이벤트가 생성되고 나중에 이전 활동에서 작동합니다. 또한, 일부 채널은 ‘업데이트’ 또는 ‘삭제’ 활동을 지원하지 않습니다. 이러한 이벤트 호출 및 관련 처리기 주위에 적절한 예외 처리를 추가하여 해당 가능성을 고려하는 것이 좋습니다.

