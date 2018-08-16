---
title: 응답 템플릿 | Microsoft Docs
description: 대화 디자이너 봇을 위한 응답 템플릿을 설정하는 방법을 알아봅니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: b09b7fa5f5672c121711deb2edcdc9718a79983f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304186"
---
# <a name="response-template-for-conversation-designer-bots"></a>대화 디자이너 봇을 위한 응답 템플릿
> [!IMPORTANT]
> 대화 디자이너는 일부 고객에게 제공되지 않습니다. 대화 디자이너의 가용성에 대한 자세한 내용은 올해 하반기에 제공됩니다.

봇은 언어 생성을 통해 변수 응답 메시지를 사용하여 풍부하고 자연스러운 방식으로 사용자와 통신할 수 있습니다. 이러한 메시지는 대화 디자이너의 응답 템플릿을 통해 관리됩니다.

응답 템플릿은 응답에서 봇 응답의 일관성 있는 어조 및 언어를 다시 사용하고 유지하도록 활성화합니다. 

## <a name="create-response-templates"></a>응답 템플릿 만들기

대화 디자이너에서 사용자에 대한 응답을 보내야 하는 상황에 다시 사용할 수 있는 응답 템플릿을 만들 수 있습니다. 

간단한 응답 템플릿은 가능한 말하기의 일회성 컬렉션을 정의하며 발언을 표시합니다. 그런 다음, 봇의 응답, 프롬프트 상태 또는 적응 카드에서 이러한 템플릿을 사용하여 완벽하게 확인된 문자열을 다시 구성합니다.

간단한 응답 템플릿을 추가하려면 다음을 수행합니다.
1. 왼쪽 패널에서 **추가**를 클릭합니다. 상황에 맞는 메뉴가 표시됩니다.
2. **간단한 응답**을 클릭합니다. 주 콘텐츠 패널에 편집 창이 나타납니다.
3. **간단한 응답 이름** 필드에 이 간단한 응답에 사용할 이름을 입력합니다.
4. **사용자에 대한 봇의 응답** 필드에 한 번에 하나의 문구를 입력합니다.
5. 간단한 응답 설정을 완료했으면 오른쪽 위 모서리에서 **저장**을 클릭합니다. 

예를 들어 다음 항목을 통해 승인 구문 템플릿(“AckPhrase”라고 함)을 만들 수 있습니다.

- 확인
- Sure(물론입니다.)
- You bet(물론)
- I'd be happy to help(성심껏 도와드리겠습니다.)
- Glad to help(도움이 되어 기쁩니다.)

이 템플릿을 사용하면 봇의 응답이 지루하고 단조로운 대신 더 자연스럽게 들릴 수 있도록 대화 런타임이 이러한 문자열 중 하나를 임의로 확인합니다.

## <a name="conditional-response-templates"></a>조건부 응답 템플릿

조건부 응답 템플릿은 조건을 사용하여 여러 응답을 지정합니다. 사용자가 작성하는 콜백 함수는 사용자가 지정한 조건에 따라 사용할 응답 문자열을 결정하는 언어 생성 엔진에 유용합니다. 

예를 들어 하루의 시각 템플릿은 하루의 실제 시각에 따라 “good morning” 또는 “good evening”을 확인해야 합니다. 

조건부 응답 템플릿을 추가하려면 다음을 수행합니다.
1. 왼쪽 패널에서 **추가**를 클릭합니다. 상황에 맞는 메뉴가 표시됩니다.
2. **조건부 응답**을 클릭합니다. 주 콘텐츠 패널에 편집 창이 나타납니다.
3. **조건부 응답 이름** 필드에서 이 템플릿에 사용할 이름을 입력합니다.
4. **조건부 응답** 필드에서 한 번에 하나의 구를 입력합니다. 예를 들어 “timeOfDayTemplate”의 경우 템플릿에서 “Good morning” 및 “Good evening”을 두 가지 가능한 응답으로 입력합니다.
5. “Good morning” 응답의 경우 해당 **조건 이름**을 “morning”으로 지정합니다. 마찬가지로 “Good evening” 응답의 경우 해당 **조건 이름**을 “evening”으로 지정합니다. 사용자가 작성하는 사용자 지정 스크립트는 이러한 조건 중 하나를 반환하게 됩니다.
6. **실행 시 실행할 코드** 필드에서 콜백 함수 이름을 입력합니다(예: `fnResolveTimeOfDayTemplate`). 그런 다음, **코드 보기**를 클릭하여 *스크립트** 편집기를 로드합니다. 여기에서 이 콜백 함수에 대한 구현을 정의할 수 있습니다.
7. 오른쪽 위 모서리에서 **저장**을 클릭하여 변경 사항을 저장하고 이 템플릿을 만듭니다.

*fnResolveTimeOfDayTemplate* 콜백 함수에 대한 샘플 코드입니다. 이 함수는 응답에서 지정한 “morning” 또는 “evening” **조건 이름**에 일치하는 문자열을 반환하게 됩니다.

```javascript
module.exports.fnTimeOfDayTemplate = function(context) {
    var currentTime = new Date().getHours();
    if(currentTime >= 12) {
        return "evening!";
    } else {
        return "morning!";
    }
}
```

## <a name="send-a-response-to-user"></a>사용자에게 응답 보내기

응답 템플릿을 사용하여 템플릿 이름을 대괄호로 묶습니다. 예를 들어 피드백 상태에서 “AckPhrase”의 간단한 응답 템플릿을 사용하려면 **사용자에 대한 봇의 응답** 구를 `[AckPhrase], I will get that done for you right away`로 입력합니다.

응답에서 대괄호를 사용하려는 경우 “\"를 이스케이프 문자로 사용하여 대화 런타임이 대괄호에 대한 확인을 건너뛰도록 합니다.

정의된 엔터티가 있는 경우 마찬가지로 사용자에 대한 응답에 사용할 수 있습니다. 엔터티 이름을 중괄호로 묶어 언어 생성에서 엔터티를 참조할 수 있습니다. 예를 들어 봇에 `location` 엔터티가 있는 경우 `[AckPhrase], I can help you find a table at {location}.`과 같이 응답에서 참조할 수 있습니다.

## <a name="nesting-response-templates"></a>응답 템플릿 중첩

응답 템플릿은 “중첩”될 수 있습니다. 하나의 응답 템플릿은 다른 응답 템플릿에 대한 참조를 가질 수 있습니다. 예를 들어 `AckPhrase` 간단한 응답 템플릿 및 `timeOfDayTemplate` 조건부 응답 템플릿을 사용하여 최종 응답을 확인하는 데 두 템플릿을 모두 사용하는 새 `timeBasedGreeting` 응답 템플릿을 구성할 수 있습니다. 

> [!TIP]
> 템플릿 중첩은 강력한 기능이지만, 중첩된 템플릿이 무한 루프를 유발하지 않도록 확인해야 합니다. 즉, `AckPhrase`가 `timeOfDayTemplate` 및 `timeOfDayTemplate`을 호출하고 다시 `AckPhrase`로 호출하는 상황을 방지해야 합니다.

예를 들어 새 **간단한 응답** 이름 `timeBasedGreeting`을 만들고, 이 템플릿에 대해 가능한 **사용자에 대한 봇의 응답**으로 `[timeOfDayTemplate] [AckPhrase], ... `와 같은 텍스트를 입력합니다.

## <a name="define-user-utterance-as-help-tips"></a>도움말 팁으로 사용자 발언 정의

사용자 **발언**을 정의하는 동안 발언을 **도움말 팁**으로 설정할 수 있습니다. 발언을 **도움말 팁**으로 설정하려면 발언의 오른쪽에 있는 `...`을 클릭하고 **도움말 팁으로 사용**을 선택합니다. 

도움말 팁은 **사용자에 대한 봇의 응답** 필드를 정의하는 기본 제공 언어 생성 템플릿 `[builtin-tasktips]`에서 사용됩니다. 예를 들어 `Sorry I did not understand that. Here are some things you can try - [builtin.tasktips]`와 같은 응답을 작성할 수 있습니다.

작업에 정의된 **도움말 팁**이 없는 경우 `[builtin-tasktips]`는 빈 문자열로 확인됩니다. 작업에 정의된 **도움말 팁**이 여러 개인 경우 `[builtin-tasktips]`는 응답할 때마다 하나를 임의로 선택합니다.

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [적응 카드](conversation-designer-adaptive-cards.md)
