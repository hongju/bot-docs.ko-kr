---
title: 입력에 단추 사용 | Microsoft Docs
description: JavaScript용 Bot Framework SDK를 사용하여 메시지 내에서 제안된 동작을 전송하는 방법을 알아봅니다.
keywords: 제안된 동작, 단추, 추가 입력
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3dac86bbcd98d48c636521b44d107f1e6341a3f7
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033383"
---
# <a name="use-button-for-input"></a>입력에 단추 사용

[!INCLUDE[applies-to](../includes/applies-to.md)]

사용자가 입력하기 위해 탭할 수 있는 단추를 봇이 표시하도록 할 수 있습니다. 단추는 사용자가 키보드를 사용하여 응답을 입력하는 대신 질문에 대답하거나 간단히 단추를 탭하여 선택할 수 있도록 하여 사용자 환경을 개선합니다. 제안된 동작 창 내에 표시되는 단추는 서식 있는 카드 내에 표시되는 단추와 달리(탭한 후에도 사용자에게 표시되고 액세스 가능함) 사용자가 선택한 후에 사라집니다. 따라서 사용자가 대화 내에서 유효하지 않은 단추를 탭하지 않게 되며, 봇 개발이 간소화됩니다(해당 시나리오를 고려할 필요가 없으므로). 

## <a name="suggest-action-using-button"></a>단추를 사용하는 작업 제안

*제안된 작업*을 사용하면 봇이 단추를 표시할 수 있습니다. 단일-턴(single-turn) 대화에서 사용자에게 표시될 제안된 동작(다른 이름: “빠른 회신”) 목록을 만들 수 있습니다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

여기에 표시된 소스 코드는 [제안 작업 샘플](https://aka.ms/SuggestedActionsCSharp)을 기반으로 합니다.

[!code-csharp[suggested actions](~/../botbuilder-samples/samples/csharp_dotnetcore/08.suggested-actions/Bots/SuggestedActionsBot.cs?range=87-100)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

여기에 표시된 소스 코드는 [제안된 작업 샘플](https://aka.ms/SuggestActionsJS)을 기반으로 합니다.

[!code-javascript[suggested actions](~/../botbuilder-samples/samples/javascript_nodejs/08.suggested-actions/bots/suggestedActionsBot.js?range=61-64)]

---

## <a name="additional-resources"></a>추가 리소스

[CSharp 샘플](https://aka.ms/SuggestedActionsCSharp) 또는 [JavaScript 샘플](https://aka.ms/SuggestActionsJS)에 표시된 전체 소스 코드에 액세스할 수 있습니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [사용자 및 대화 데이터 저장](./bot-builder-howto-v4-state.md)
