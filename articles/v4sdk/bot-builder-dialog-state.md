---
title: Dialog 상태 | Microsoft Docs
description: Bot Builder SDK 내에서 상태가 작동하는 방법을 설명합니다.
keywords: 상태, 봇 상태, 대화 상태, 사용자 상태
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4da715650eb1c3e7b284aaa47aa5ce4ac7280f4c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998760"
---
# <a name="dialog-state"></a>Dialog 상태

Dialogs는 다중 순서 대화 논리를 구현하기 위한 방법이며, 지속적인 상태를 사용하는 SDK 기능의 한 예와 같은 것입니다. 

Dialog 기반 봇은 일반적으로 DialogSet 컬렉션을 봇 구현의 멤버 변수로 보유합니다. DialogSet은 Accessor(접근자)라는 개체에 대한 핸들을 사용하여 만들어집니다. 

Accessor는 SDK 상태 모델의 핵심 개념입니다. Accessor는 IStatePropertyAccessor 인터페이스를 구현합니다. 즉 Get, Set 및 Delete 메서드에 대한 구현을 제공해야 합니다. Accessor를 만들려면 여기에 속성 이름을 제공해야 합니다. 

Accessor를 사용하는 코드는 해당 이름의 속성을 참조한다고 인식하는 Get, Set 및 Delete 메서드를 호출할 수 있습니다. 상위 수준의 구성 요소에 일부 상태 지속성이 필요한 경우 Accessor를 사용해야 합니다. 이와 같이 다양한 시나리오에서 상태 저장소의 구현을 가져올 수 있지만 여전히 동일한 상위 수준의 코드를 사용합니다. 예를 들어 Dialogs는 Accessor를 사용하며, 이는 지속된 상태에 액세스하는 유일한 방법입니다.

![Dialog 상태](media/bot-builder-dialog-state.png)

봇의 OnTurn이 호출되면 봇에서 DialogSet의 CreateContext를 호출하여 Dialog 하위 시스템을 초기화합니다. DialogContext를 만들려면 상태가 필요하므로 DialogSet에서 해당 Accessor를 사용하여 적절한 대화 상태 JSON을 가져옵니다(Get). SDK는 BotState 클래스 모양의 Accessor 구현을 제공합니다. 상태 구현을 활용하려는 애플리케이션은 BotState의 하위 클래스가 될 수 있으므로 Accessor의 구현을 상속할 수 있습니다. SDK에는 다음 두 개의 BotState 하위 클래스가 포함되어있습니다.

- UserState
- ConversationState

UserState 및 ConversationState는 기본 저장소에 키를 사용합니다. 키는 실제 저장소로 전달됩니다. 논리적으로 이 키는 Accessor에서 이름이 지정된 속성에 대한 네임스페이스로 작동합니다. BotState 구현은 내부적으로 TurnContext의 인바운드 Activity를 사용하여 사용하는 저장소 키를 동적으로 생성합니다.

- UserState는 채널 ID와 원본 ID를 사용하여 키를 만듭니다. 예: _{Activity.ChannelId}/conversations/{Activity.From.Id}#DialogState_
- ConversationState는 채널 ID 및 대화 ID를 사용하여 키를 만듭니다. 예: _{Activity.ChannelId}/users/{Activity.Conversation.Id}#YourPropertyName_

애플리케이션은 Accessor를 제공해야 하며, 동적으로 만들어진 적절한 저장소 키와 속성 이름에 대한 바인딩이 모두 백그라운드에서 수행됩니다. BotState의 Accessor 구현에는 다음과 같은 몇 가지 최적화가 포함됩니다. 

- 첫 번째 최적화는 캐시되고 지연되는 로드입니다. Accessor에 대한 첫 번째 Get이 호출될 때까지 실제 저장소 구현의 Load(로드)가 지연되고, 해당 Load의 결과가 캐시에 보관됩니다. 이 캐시에 대한 참조는 해당 BotState에서 제공하는 키를 사용하여 TurnContext에 추가됩니다. 따라서 UserState에 해당하는 캐시는 "UserState"라는 필드에 보관되고, ConversationState에 해당하는 캐시는 "ConversationState"라는 필드에 보관됩니다. 다양한 Accessor 개체를 호출하면 TurnContext가 전달되므로 적절한 캐시로 이동하여 이 캐시를 가져올 수 있습니다.

- 두 번째 최적화는 상태가 변경되었는지 여부를 결정하는 논리가 BotState 클래스에 포함되어 있다는 것입니다. 변경된 경우에만 기본 저장소에서 실제 Save(저장) 작업이 실행됩니다.

- BotState 구현 내의 세 번째 최적화는 BotStateSet이라는 클래스에 있습니다. BotStateSet은 해당 SaveChanges 메서드의 호출을 컬렉션의 모든 멤버에 병렬로 위임하는 BotState 개체의 컬렉션입니다.

BotState의 SaveChanges 호출이 IStatePropertyAccessor 인터페이스의 일부가 아닌 것은 중요합니다. 그 이유는 SaveChanges가 모델의 핵심 측면이 아니라 BotState 구현의 특별한 최적화이기 때문입니다. 특히 Dialogs와 같은 코드는 SaveChanges는 커녕 BotState에 대해서도 전혀 인식하지 못합니다. 실제로 Dialogs 코드는 Accessor에만 결합됩니다. 실행이 완료되면 Dialog 시스템 외부에서 SaveChanges 메서드를 호출해야 합니다. 예를 들어 다이어그램에서 볼 수 있듯이 이 메서드는 봇의 OnTurn 메서드 내에서 호출할 수 있습니다.

BotState 구현은 몇 가지 특정 의미 체계를 제공한다는 점에 유의해야 합니다. 특히 마지막 쓰기가 이전에 쓴 상태를 덮어쓰는 "마지막으로 성공한 쓰기" 동작을 지원합니다. 이는 많은 애플리케이션에서 제대로 작동할 수 있지만, 특히 진행 중인 일정 수준의 동시성이 있을 수 있는 규모 확장 시나리오에서 의미가 있습니다. 이로 인해 문제가 되는 경우 대답은 사용자 고유의 Accessor를 구현하여 Dialog에 전달하는 것입니다. 다양한 대체 방법이 있습니다. 예를 들어 솔루션에서 Azure Storage와 같은 클라우드 저장소 서비스에서 널리 사용되는 eTag 조건을 활용할 수 있습니다. 이 경우 솔루션은 아마도 다른 중요한 부분을 구현하고 있을 것입니다. 예를 들어 아웃바운드 활동을 버퍼링하고, Save 작업이 성공한 후에만 활동을 보낼 수 있습니다. 주목해야 할 중요 사항으로 이 동작은 BotState 구현의 동작이 아니라 애플리케이션이 제공하고 Accessor 수준에서 플러그 인하는 동작이라는 것입니다.

BotState 구현 자체에는 플러그형 저장소 모델이 있습니다. 이는 간단한 Load/Save 패턴을 따르고 SDK는 두 가지 대체 구현을 프로덕션에 제공합니다. 하나는 Azure Storage용이고, 다른 하나는 CosmosDB용이며, 테스트 용도의 메모리 내 구현이 있습니다. 그러나 여기서 주목해야 할 중요 사항으로 "마지막으로 성공한 쓰기"의 의미 체계가 BotState 구현을 통해 결정된다는 것입니다.

## <a name="handling-state-in-middleware"></a>미들웨어의 상태 처리
위의 다이어그램에서는 봇의 OnTurn이 완료될 때 발생하는 SaveAllChanges에 대한 호출을 보여 줍니다. 다음은 호출에 초점을 맞춘 다이어그램입니다.

![상태 미들웨어 문제](media/bot-builder-dialog-state-problem.png)

이 방법의 문제는 봇의 OnTurn 메서드가 반환된 후에 발생하는 일부 사용자 지정 미들웨어에서 수행된 모든 상태 업데이트가 영구적 저장소에 저장되지 않는다는 것입니다. 해결 방법은 AutoSaveChangesMiddleware를 미들웨어 스택의 끝에 추가하여 SaveAllChanges에 대한 호출을 사용자 지정 미들웨어가 완료된 후로 이동시키는 것입니다. 실행은 아래와 같습니다.

![상태 미들웨어 해결 방법](media/bot-builder-dialog-state-solution.png)

## <a name="additional-resources"></a>추가 리소스
자세한 내용은 GitHub의 [[C#](https://github.com/Microsoft/BotBuilder-dotnet) | [JavaScript](https://github.com/Microsoft/BotBuilder-js)] Bot Builder SDK를 참조하세요.
