---
title: 봇의 첫 번째 사용자 상호 작용 설계 | Microsoft Docs
description: 뛰어난 첫 번째 사용자 환경을 만드는 방법과 성공을 위한 봇을 설계하는 방법에 대해 알아봅니다.
keywords: 첫 인상, 시작, 언어 대 메뉴
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f59a7acbdb7d580aebeef6ffe81d8b15505aed90
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999810"
---
# <a name="design-a-bots-first-user-interaction"></a>봇의 첫 번째 사용자 상호 작용 설계

## <a name="first-impressions-matter"></a>중요한 첫 인상

사용자와 봇 간의 첫 번째 상호 작용은 사용자 경험에 중요합니다. 봇을 설계할 때 첫 번째 메시지는 "안녕" 이라는 의미보다 더 많은 것을 담고 있습니다. 앱을 빌드할 때는 중요한 [탐색](bot-service-design-navigation.md) 단서를 제공하는 첫 번째 화면을 설계하세요. 사용자는 메뉴의 위치와 작동 방식, 도움을 요청할 위치, 개인 정보 보호 정책 등을 직관적으로 이해해야 합니다. 봇을 설계할 때 봇과 사용자의 첫 번째 상호 작용은 동일한 유형의 정보를 제공해야 합니다. 

## <a name="language-versus-menus"></a>언어 대 메뉴 

다음 두 가지 설계를 고려하세요.

### <a name="design-1"></a>설계 1

![봇](~/media/bot-service-design-first-interaction/hello1.png)


### <a name="design-2"></a>설계 2

![봇](~/media/bot-service-design-first-interaction/hello2.png)

“어떻게 도와 드릴까요?”와 같은 개방형 질문으로 봇을 시작하는 것은 일반적으로 권장되지 않습니다. 봇의 기능이 100가지라면 사용자는 대부분을 추측할 수 없을 것입니다. 봇의 기능을 알려주지 않았는데 사용자가 어떻게 알 수 있을까요?

메뉴는 그 문제에 대한 간단한 해결책을 제공합니다. 먼저, 설계사용 가능한 옵션을 나열하여 사용자에게 기능을 전달합니다. 두 번째로 메뉴는 사용자가 너무 많이 입력하지 않아도 되고 클릭하기만 하면 됩니다. 마지막으로, 메뉴를 사용하면 봇이 사용자로부터 받을 수 있는 입력의 범위가 좁아져 자연어 모델이 크게 간소화됩니다. 

> [!TIP]
> 메뉴는 뛰어난 사용자 환경을 위한 봇을 설계할 때 유용한 도구입니다. 따라서 메뉴가 "충분한 만족"을 주지 못한다고 해제하지 마세요. 자유 형식의 입력을 계속 지원하면서 메뉴를 사용하도록 봇을 설계할 수 있습니다. 사용자가 메뉴 옵션을 선택하는 대신 초기 메뉴에 응답하면 봇은 사용자의 텍스트 입력을 구문 분석하려고 시도할 수 있습니다. 

또는 봇에 특정 기능이 있는 경우 사용자를 유도하는 더 구체적인 질문을 할 수 있습니다. 예를 들어, 봇이 샌드위치 주문을 담당하는 경우 첫 번째 상호 작용은 "안녕하세요! 샌드위치를 주문 받겠습니다. 어떤 빵을 드시겠습니까? 일반빵, 통밀빵, 호밀빵이 있습니다." 이렇게 하면 사용자가 응답하는 방법을 알 수 있으며, 대화를 통해 탐색 단서가 제공됩니다.

## <a name="other-considerations"></a>기타 고려 사항

잘 설계된 봇은 직관적이고 쉽게 탐색할 수 있는 첫 번째 상호 작용을 제공할 뿐만 아니라, 사용자가 개인 정보 보호 정책 및 사용 약관에 대한 정보를 볼 수 있게 해줍니다. 

> [!TIP]
> 봇이 사용자로부터 개인 데이터를 수집하는 경우, 이를 전달하고 데이터로 수행할 작업을 설명하는 것이 중요합니다.

## <a name="next-steps"></a>다음 단계

이제 사용자와 봇 간의 첫 번째 상호 작용을 설계하는 데 필요한 몇 가지 기본 원칙을 익혔으므로 [대화 흐름을 설계](~/bot-service-design-conversation-flow.md)하는 방법에 대해 자세히 알아봅니다.
