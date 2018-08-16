---
title: Do 작업으로 다이얼로그 정의 | Microsoft Docs
description: 다이얼로그를 Do 작업으로 설정하는 방법을 알아봅니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: d59df20821a7f63eb9ee5dea365597b1af839f75
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301570"
---
# <a name="define-a-dialogue-as-a-do-action"></a>Do 작업으로 다이얼로그 정의
> [!IMPORTANT]
> 대화 디자이너는 일부 고객에게 제공되지 않습니다. 대화 디자이너의 가용성에 대한 자세한 내용은 올해 하반기에 제공됩니다.

다이얼로그는 특정 태스크에 대한 대화형 모델을 포함합니다. 예를 들어, 사용자가 테이블을 예약하도록 도와주는 카페 봇은 "Book Table"이라는 작업을 정의할 수 있습니다. **Do** 작업은 봇과 사용자 간의 대화형 흐름을 모델링하는 다이얼로그에 바인딩될 수 있습니다. 

다이얼로그는 봇이 사용자와 주고받는 대화에 참여하여 태스크 완료에 도움을 줄 때 특히 유용합니다.  사용자는 단일 발언으로 태스크 완료에 필요한 모든 값을 제공하기 어렵습니다. 

테이블을 예약하려면 위치, 파티 규모, 날짜 및 시간 기본 설정이 필요합니다. 테이블을 예약하는 봇은 사용자가 말할 수 있는 다양한 구를 이해하고 처리해야 합니다. 

- *테이블 예약*: 이 경우에는는 필요한 어떤 엔터티도 캡처되지 않았습니다. 이제 봇은 사용자와의 대화에 참여해야 합니다.
- *이번 주 토요일 오후 7시로 테이블 예약*: 이 경우 날짜 및 시간 기본 설정이 지정되었으나 봇은 의도한 위치 및 파티 규모를 계속 수집해야 합니다.

대화를 하는 동안 일부 엔터티가 사용자 입력에 따라 변경될 수 있습니다. 예를 들어, 사용자가 "Book a table in Redmond for this Sunday at 7 pm"이라고 말하지만 레드몬드점은 일요일 6시에 문들 닫는다면 봇의 다이얼로그는 이러한 잘못된 요청을 처리해야 합니다. 

대화 디자이너는 대화 흐름을 시각화하는 끌어서 놓기 다이얼로그 디자이너를 제공합니다. 이 다이얼로그 디자이너는 대화 흐름을 모델링하는 데 사용할 수 있는 7가지 *다이얼로그 상태*를 제공합니다.

## <a name="dialogue-states"></a>다이얼로그 상태

다이얼로그는 대화형 상태로 구성됩니다. 다이얼로그 자체는 대화 런타임에 대화형 흐름을 실행하는 방법에 대한 구조를 제공하는 구조화된 직접 흐름으로 모델링됩니다.

다이얼로그에는 사용 가능한 여러 기본 제공 상태가 제공됩니다. 지원되는 기본 제공 상태는 다음과 같습니다.

- [**시작**](#start-state): 대화형 흐름에 대한 시작 상태를 나타냅니다. 모든 다이얼로그에는 하나 이상의 시작 상태가 정의되어야 합니다.
- [**반환**](#return-state): 특정 대화형 흐름의 완료를 나타냅니다. 대화형 흐름이 구성 가능할 경우, 반환은 대화 런타임에 이 다이얼로그의 가능한 호출자에게 실행을 반환하도록 지시합니다.
- [**의사 결정**](#decision-state): 대화형 흐름에서 분기 지점을 나타냅니다.
- [**프로세스**](#process-state): 봇이 비즈니스 논리를 실행하는 상태를 나타냅니다.
- [**프롬프트**](#prompt-state): 사용자에게 입력을 요구할 수 있는 상태를 나타냅니다. 
- [**피드백**](#feedback-state): 사용자에게 피드백 또는 확인을 제공할 수 있는 상태를 나타냅니다. 예를 들어, 예약이 수행되었음을 확인하는 다이얼로그가 있습니다.
- [**모듈**](#module-state): 다른 다이얼로그에 대한 호출을 나타냅니다. 기본적으로 다이얼로그 흐름이 구성 가능하므로 이 상태는 이 작업에 정의된 대로 공유 다이얼로그 또는 일부 다른 다이얼로그를 호출할 수 있습니다.

각 대화 상태는 다이얼로그 디자이너의 다이얼로그 커넥터를 사용하여 다른 상태로 연결됩니다.

모든 다이얼로그 상태는 사용자 지정 스크립트에 대한 콜백 함수 이름을 포함하여 해당 상태에 대한 속성을 지정하는 데 사용되는 연결된 상태 편집기가 있습니다. **상태 편집기**는 다이얼로그 디자이너의 기본 뷰포트 하단에 크기 조정 창으로 표시됩니다. 이 편집기를 호출하려면 다이얼로그 디자이너에서 상태를 두 번 클릭합니다. 그러면 **상태 편집기**에 해당 상태에 대한 속성이 표시됩니다.
<!-- TODO: insert screenshot of the wrench in horizontal menu -->

다음 하위 섹션에서는 이러한 각 기본 제공 대화 상태에 대한 자세한 세부 정보를 제공합니다.

### <a name="start-state"></a>시작 상태

시작 상태는 대화의 시작 지점을 나타냅니다. 필요한 값은 상태 **이름**입니다. 이름은 기본적으로 "Start"이고 편집하여 상태 이름을 바꿀 수 있습니다.

### <a name="return-state"></a>반환 상태

반환 태는 대화형 흐름의 특정 분기의 완료를 나타냅니다. 다이얼로그가 구성 가능하므로 이 상태는 대화 런타임에 호출자 다이얼로그로 실행을 반환하도록 지시합니다. 필요한 값은 상태 **이름**입니다. 이름은 기본적으로 "반환"이고 편집하여 상태 이름을 바꿀 수 있습니다.

### <a name="decision-state"></a>의사 결정 상태

의사 결정 상태는 대화형 흐름에서 분기를 나타냅니다. 따를 분기를 평가하는 사용자 지정 스크립트를 작성할 수 있습니다. 사용자 입력 및 비즈니스 논리에 따라, 이 스크립트는 가능한 전환 값 중 하나를 반환합니다. 각 전환 값은 대화 런타임이 다이얼로그의 다른 분기를 실행하도록 요구합니다.

의사 결정 상태에 대한 필수 속성:
- **이름**: 의사 결정 상태의 고유한 이름입니다.
- **실행 시 실행할 코드**: 진행할 대화 분기를 결정하는 비즈니스 논리를 구현하는 콜백 함수의 이름입니다. 

#### <a name="example-code-for-decision-state"></a>의사 결정 상태에 대한 예제 코드

다음 샘플 콜백 함수는 실행할 분기를를 대화 런타임에 알려주는 의사 결정을 반환합니다.

```javascript
module.exports.fnDecisionState = function(context) {
    var a = context.taskEntities['a'];
    if (a[0].value === '0') {
        return 'yes';
    }
    else if (a[0].value === '1') {
        return 'no';
    }
}
```

### <a name="process-state"></a>프로세스 상태

프로세스 상태는 봇이 대화를 계속 진행하거나 최종 태스크 완료 작업을 수행하려고 하는 다이얼로그의 지점을 나타냅니다. 

프로세스 상태에 대한 필수 속성:
- **이름**: 프로세스 상태의 고유한 이름입니다.
- **실행 시 실행할 코드**: 비즈니스 논리를 구현하는 콜백 함수의 이름입니다.

#### <a name="example-code-for-process-state"></a>프로세스 상태에 대한 예제 코드

다음 샘플 콜백 함수는 날씨를 가져오고 사용자에게 날씨 정보를 반환합니다.

```javascript
module.exports.fnGetWeather = function(context) {
    var options =  {
        host: 'mock',
        path: '/get?a=b',
        method: 'get'
    };
    return http.request(options).then(function(response) {
        context.contextEntities['x'].value= response.statusCode;
        var jsonBody = JSON.parse(response.body);
          // understand response
        if (response.statusCode != "200") {
            // error
        }
    });
}
```

### <a name="prompt-state"></a>프롬프트 상태

프롬프트 상태는 사용자에게 특정 정보를 묻습니다. 프롬프트 상태는 하위 다이얼로그 시스템을 포함하므로 당연히 복잡한 상태입니다. 

프롬프트 상태 내에서, 사용자에게 제공할 실제 응답을 정의하고 필요에 따라 적응형 카드를 포함할 수 있습니다. 그런 후 사용자의 응답을 구문 분석하고 이해하기 위한 트리거를 지정할 수 있습니다. 이 트리거는 LUIS이거나 정규식을 사용하는 사용자 지정 코드 인식기일 수 있습니다.  

사용자가 잘못된 입력을 제공하는 경우 봇은 동일한 정보를 사용자에게 다시 확인할 수 있습니다. 이 동작을 프롬프트 상태 편집기에서 정의할 수도 있습니다. 

#### <a name="prompting-the-user"></a>사용자에게 확인

프롬프트 응답을 사용하여 **사용자에게 입력을 확인**할 때 사용할 메시지를 지정할 수 있습니다. 예를 들어, 날짜 및 시간을 수집하려는 경우 가능한 응답은 "When would you like to come in?" 또는 "When would you like to dine with us?"일 수 있습니다.

#### <a name="prompt-listening-for-user-input"></a>사용자 입력 수신 프롬프트

사용자가 응답하도록 요구된 후에, 대화 런타임은 사용자 입력을 자동으로 듣고 말한 내용을 이해하려고 합니다. LUIS 또는 사용자 지정 코드 인식기를 기준으로 하는 트리거가 사용자 입력 및 의도를 이해하도록 구성합니다. 이것은 태스크 트리거와 비슷합니다.

#### <a name="re-prompting-the-user"></a>사용자에게 다시 확인

다시 프롬프트 섹션을 사용하여 각 시도에 대한 응답을 지정합니다. 다시 확인 섹션의 각 행은 특정 차례에 사용되는 다시 프롬프트 문자열에 해당합니다. 첫 번째 응답은 첫 번째 다시 프롬프트에서, 두 번째 응답은 두 번째 다시 프롬프트에서 사용되며 이와 같은 방식으로 진행됩니다. 예: 

*I'm sorry, I did not understand, when did you want to come in?*
*My bad, I'm having a hard time understanding you. Let's try this again - when did you want to come in?*

#### <a name="prompt-callback-functions"></a>프롬프트 콜백 함수

프롬프트 상태에서 두 개의 서로 다른 콜백 함수를 지정할 수 있습니다. 

1. **모든 프롬프트 및 다시 프롬프트 전에**: 모든 프롬프트 또는 다시 프롬프트 전에 이 함수를 실행합니다. 이 콜백 함수는 true는 이 프롬프트 또는 다시 프롬프트를 실행함을 의미하고, false는 이 프롬프트 또는 다시 프롬프트를 실행하지 않음을 의미하는 부울 반환 값을 예상합니다. `getTurnIndex()`를 사용하여 프롬프트 실행에 대한 현재 차례 인덱스를 가져올 수 있습니다.
2. **응답 시**: 프롬프트가 생성되었으나 사용자에게 전송되기 전에(다시 프롬프트 응답 포함) 이 함수를 실행합니다. 이를 통해 스크립트는 사용자에게 전송되는 메시지를 수정할 수 있습니다.

#### <a name="sample-code"></a>샘플 코드

이 코드 조각에서는 **실행 전** 콜백 예제를 보여 줍니다.

```javascript
module.exports.fnBeforeExecuting = function(context) {
    if(context.responses[0].text === "C") {
        return false;
    }
    return true;
}
```

이 코드 조각에서는 **프롬프트 시** 콜백 예제를 보여 줍니다.

```javascript
module.exports.fnOnPrompting = function(context) {
    // include a hint card
    var activity = context.responses.slice(-1).pop();
    activity.attachments.push({
        "contentType": "application/vnd.microsoft.card.hero",
        "content": {
            "buttons": [
                {
                    "type": "imBack",
                    "title": "1",
                    "value": "1"
                },
                {
                    "type": "imBack",
                    "title": "2",
                    "value": "2"
                }
            ]
        }
    });
}
```

이 코드 조각에서는 **다시 프롬프트 전** 콜백 예제를 보여 줍니다.

```javascript
module.exports.fnBeforeReprompting = function(context) {
    if(context.responses[0].text === "C") {
        return false;
    }
    return true;
}
```

### <a name="feedback-state"></a>피드백 상태

이 상태를 사용하여 사용자에게 다시 응답을 제공합니다. 이 상태의 일반적인 사용 사례에는 태스트 완료 시도 이후의 최종 결과를 포함하거나, 실패 조건에서 사용자에게 응답을 다시 제공하는 경우입니다. 

각 피드백 상태는 고유한 이름, 몇 가지 가능한 응답 값이 필요하며, 경우에 따라 적응형 카드 정의를 포함할 수 있습니다. [적응형 카드 정의](conversation-designer-adaptive-cards.md)에 대해 자세히 알아보세요.

각 피드백 상태는 필요한 경우 사용자에게 전송되기 전에 활성 페이로드를 수정하는 사용자 지정 스크립트를 작성할 수 있는 **응답 시** 콜백 함수도 허용합니다. 


```javascript
module.exports.fnOnResponding = function(context) {
    // include a hint card
    var activity = context.responses.slice(-1).pop();
    activity.attachments.push({
        "contentType": "application/vnd.microsoft.card.hero",
        "content": {
            "buttons": [
                {
                    "type": "imBack",
                    "title": "1",
                    "value": "1"
                },
                {
                    "type": "imBack",
                    "title": "2",
                    "value": "2"
                }
            ]
        }
    });

}
```

### <a name="module-state"></a>모듈 상태

모듈 상태는 하위 다이얼로그를 실행하기 위한 참조를 추가하는 데 사용됩니다. 이 상태를 사용하여 다이얼로그를 함께 연결할 수 있습니다. 

각 모듈 상태에는 고유한 이름과 실행할 특정 다이얼로그에 대한 포인터가 필요합니다. 실행할 다이얼로그에 사용할 수 있는 옵션은 **공유 다이얼로그** 또는 특정 **태스크** 아래에 표시되어야 합니다.

## <a name="multiple-dialogues-under-a-task"></a>하나의 태스크에 여러 다이얼로그

하나의 태스크에 여러 다이얼로그가 있을 수 있습니다. 태스크에 다이얼로그를 추가하려면 태스크를 추가하고 왼쪽 트리 패널에서 **추가** 단추를 클릭한 후 **다이얼로그 추가**를 클릭합니다. 그러면 선택한 태스크 아래에 새 다이얼로그가 추가됩니다. 

다이얼로그가 구성 가능하므로 루트 다이얼로그 흐름을 해당 태스크 아래에 다른 다이얼로그를 호출하는 태스크에 바인딩할 수 있습니다. 이렇게 하면 하위 태스크를 캡슐화하고 재사용을 설정할 수 있습니다. 또한 *모듈* 상태를 사용하여 대화형 흐름에서 이러한 다이얼로그를 연결할 수도 있습니다.

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [응답 템플릿](conversation-designer-response-templates.md)
