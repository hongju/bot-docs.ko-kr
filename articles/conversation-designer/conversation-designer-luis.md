---
title: LUIS 인식기를 작업 트리거로 정의 | Microsoft Docs
description: LUIS.ai를 사용하여 언어 이해 인식기를 작업 트리거로 설정하는 방법을 알아봅니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 39fe222fb1d54346b33617c425b1fdf2d56daa0d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304171"
---
# <a name="define-a-luis-recognizer-as-task-trigger"></a>LUIS 인식기를 작업 트리거로 정의
> [!IMPORTANT]
> 대화 디자이너는 일부 고객에게 제공되지 않습니다. 대화 디자이너의 가용성에 대한 자세한 내용은 올해 하반기에 제공됩니다.

대부분의 경우 사용자는 작업을 수행하기 위한 자신의 의도를 자연어로 표현합니다. 대화 디자이너를 사용하면 <a href="https://luis.ai" target="_blank">LUIS.ai</a>에서 제공하는 봇을 위한 자연어 이해 모델을 손쉽게 설정할 수 있습니다.

이렇게 하려면 트리거 형식 **사용자가 무언가를 말하거나 입력**을 선택합니다. 그러면 **의도** 이름을 지정하는 옵션이 제공됩니다. 

기존 의도를 검색하거나 **어떤 언어 의도?** 필드에서 새로 만들 수 있습니다.

## <a name="create-a-new-intent"></a>새 의도 만들기

새 의도를 만들려면 의도에 사용할 이름을 입력하고 **새 의도 만들기**를 클릭합니다. 이 특정 작업을 트리거해야 하는 사용자가 말할 것으로 예상하는 가능한 항목에 대한 예제 발언을 입력합니다.

예를 들어 카페 봇은 사용자 근처에 있는 카페 위치를 찾는 작업을 수행할 수 있어야 합니다. 이 시나리오를 처리하려면 **사용자가 무언가를 말하거나 입력**을 선택하고 **의도 이름**을 “LocationNearMe”로 설정합니다. 그런 다음, 이 의도에 대한 예제 발언을 입력합니다. 예:  
- *find locations near me(주변 위치 찾기)*
- *find cafe locations near me(주변 카페 위치 찾기)*
- *is the Redmond store open now?(현재 Redmond 상점이 열려 있나요?)*
- *is there a store in Seattle?(시애틀에 상점이 있나요?)*
- *what cafe locations are open now?(현재 열려있는 카페 위치는?)*
- *where can I get something to eat?(먹을 것을 살 수 있는 곳은?)*
- *I would like to get something to eat(뭔가 먹고 싶다)*
- *I'm hungry(배가 고프다)*

이 특정 작업을 트리거하는 사용자의 의도를 표현하는 데 유용한, 생각해 낼 수 있는 많은 발언을 입력합니다.

## <a name="default-intents-provisioned-for-your-bot"></a>봇에 프로비전된 기본 의도

기본적으로 봇은 4가지 의도를 통해 프로비전됩니다. 
- **None(없음)**: 이는 봇에 대한 콜백(기본값) 의도입니다. 이 의도를 사용하면 봇이 아직 응답하는 방법을 알지 못하는 항목을 캡처하는 데 유용합니다.
- **Help(도움말)**: 사용자가 도움이 필요한지 판단하는 데 유용한 예제 발언으로 설정합니다. *Help(도움말), I need help(도움이 필요합니다), what can I say?(뭐라고 말해야 하나요?), I'm stuck(생각이 안 나네요)* 등입니다.
- **Greeting(인사말)**: *Hi(안녕), Hello(안녕하세요), Good morning(좋은 아침), How are you bot(안녕, 봇)* 등과 같이 인사말 의도에 일치하는 데 유용한 예제 발언으로 설정합니다.
- **Cancel(취소)**: 취소 의도에 대한 예제 발언으로 설정합니다. *Stop(중지), Cancel that(취소해), do not do it(그거 하지 마), revert(되돌아가기)* 등입니다.

## <a name="create-and-label-entities"></a>엔터티 만들기 및 레이블 지정

언어 이해는 사용자 의도를 결정하는 데 유용한 것 외에, 작업에 관련된 특정 관심사 엔터티를 결정하는 데도 유용합니다. 예를 들어 사용자가 *find cafe locations near Redmond(Redmond 근처 카페 위치 찾기)* 라고 말한 경우 엔터티 ***위치***에 대한 가능한 값으로 *Redmond*를 추출할 수 있습니다. 

작업에 대한 엔터티를 설정하려면 **발언** 문자열에서 엔터티 값에 대한 예제가 되어야 하는 발언의 부분을 선택합니다. 이를 기존 엔터티에 할당하거나 새로 만듭니다. 새 엔터티를 만들려면 엔터티의 이름을 **검색 또는 만들기** 필드에 입력하고 **새 엔터티 만들기**를 클릭합니다. 

# <a name="supported-entity-types"></a>지원되는 엔터티 형식

언어 이해는 다양한 형식의 엔터티를 만드는 기능을 제공합니다. 새로운 엔터티를 만들 때 `type`을 지정해야 합니다. 

사용 가능한 형식은 다음과 같습니다.

- **단순**: 이것은 *기본* 형식입니다.
- **목록**: 엔터티에 가능한 값의 한정된 집합이 있는 경우 이 형식을 사용합니다. 예: *Color*, *City*.
- **계층적**: 부모-자식 관계의 엔터티를 만들려면 이 형식을 사용합니다. 예: *fromCity* 및 *toCity*에는 모두 부모인 *City* 엔터티가 있습니다.
- **복합**: 의미 있는 단위를 구성하는 값의 그룹을 만들려면 이 형식을 사용합니다. 예: *City* 및 *State*는 함께 *Location* 엔터티를 구성합니다.

<!-- # pre-built entity types TBD -->

# <a name="entities-in-use"></a>사용 중인 엔터티

엔터티를 만들고 언어 이해 섹션에 추가하면 **사용 중인 엔터티** 테이블이 이 특정 작업을 사용하는 엔터티 목록으로 업데이트됩니다. 이 작업에서 사용되는 다른 엔터티를 목록에 수동으로 추가할 수 있습니다. 

사용 가능한 옵션은 다음과 같습니다.

- **코드**: 이는 사용자 지정 스크립트에서 만든 엔터티입니다. 여기서 IntelliSense와 같은 작성 기능에 유용하도록 지정할 수 있습니다.

<!-- # Use as help tip TBD  -->

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [코드 인식기](conversation-designer-code-recognizer.md)
