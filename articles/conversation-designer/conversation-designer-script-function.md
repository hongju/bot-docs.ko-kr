---
title: Do 작업으로 스크립트 함수 정의 | Microsoft Docs
description: Do 작업으로 스크립트 작업 함수를 설정하는 방법에 대해 알아봅니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 674781c2cb5a8700941feb59b2ce7ab902979d4f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304090"
---
# <a name="define-a-script-function-as-a-do-action"></a>Do 작업으로 스크립트 함수 정의

[스크립트 작업 함수](conversation-designer-context-object.md#script-callback-functions)는 작업을 완료하는 데 유용하도록 정의한 사용자 지정 스크립트를 실행합니다. 예를 들어 사용자가 “Set my thermostat to 74 degrees.(내 자동 온도 조절기를 74도로 설정해.)”라고 말한 경우 실제로 자동 온도 조절기를 74도로 설정하는 서비스를 호출합니다. 

사용자 지정 스크립트를 작업으로 사용하려면 작업 편집기에서 **스크립트 작업**을 **DO** 작업 형식으로 선택합니다. 그런 다음, 작업을 구현하는 함수의 이름을 입력합니다. **편집**을 클릭하여 **스크립트** 편집기를 불러오고 이 함수에 대한 구현을 작성합니다. 

## <a name="script-action-function-parameter"></a>스크립트 작업 함수 매개 변수

지정한 작업 콜백 함수는 항상 [`context`](conversation-designer-context-object.md) 개체를 사용하여 호출됩니다.

`context` 개체는 `taskEntities` 및 `contextEntities`를 모두 포함합니다. 작업 엔터티는 이 작업을 위해 정의되거나 생성된 엔터티이고, 컨텍스트 엔터티는 사용자와의 대화에서 엔터티를 보존할 수 있는 속성 모음입니다.

## <a name="return-value"></a>반환 값
**스크립트 작업** 함수는 부울을 반환해야 합니다.

## <a name="sample-script-action-function"></a>샘플 스크립트 작업 함수
다음 샘플 함수는 일치하는 트리거를 반환하기 전에 응답을 가져오도록 HTTP 호출을 만듭니다.

```javascript
module.exports.NewTask_do_onRun = function(context) {
    var options =  {
        host: 'HOST',
        path: 'PATH',
        method: 'post',
        headers: {
            "HEADER1" : "VALUE"
        }, 
        body: {
            "BODY": VALUE
        }
    };

    return http.request(options).then(function(response) {
      // parse response payload
      // Send a message to user
      context.responses.push({text: "Done", type: "message"});
    });
} 
```

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [대화 상자 작업](conversation-designer-dialogues.md)
