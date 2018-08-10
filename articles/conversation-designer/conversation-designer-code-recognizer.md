---
title: 코드 인식기를 작업 트리거로 정의 | Microsoft Docs
description: 사용자 지정 코드 인식기를 작업 트리거로 사용하는 방법을 알아봅니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 6d44cd299e46971b2218bb78d16e454d5df3c1d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300981"
---
# <a name="define-code-recognizer-as-task-trigger"></a>코드 인식기를 작업 트리거로 정의
> [!IMPORTANT]
> 대화 디자이너는 일부 고객에게 제공되지 않습니다. 대화 디자이너의 가용성에 대한 자세한 내용은 올해 하반기에 제공됩니다.

코드 인식기를 사용하면 작업을 수행하는 데 도움이 되는 사용자 지정 스크립트를 작성할 수 있습니다. 기타 서비스에 대한 Regex 기반 식 평가 또는 호출을 사용하여 사용자의 의도를 확인할 수 있습니다. 사용자 지정 스크립트는 작업을 트리거하도록 대화 런타임에 지시합니다. 

## <a name="create-a-script-function"></a>스크립트 함수 만들기
스크립트를 인식기로 사용하려면 작업 편집기에서 “스크립트 함수”를 인식기로 선택하고, 함수 이름을 지정하고, **Create/view function**(함수 만들기/보기)을 클릭하여 스크립트 편집을 시작합니다. 또는 스크립트 이름을 지정하지 않고 **Create/view function**(함수 만들기/보기)을 클릭하면 기본 이름을 사용하여 빈 함수가 만들어집니다. 

## <a name="script-trigger-function-parameter"></a>스크립트 트리거 함수 매개 변수

지정한 스크립트 콜백 함수는 항상 [`context`](conversation-designer-context-object.md) 개체를 사용하여 호출됩니다.

`context` 개체는 `taskEntities` 및 `contextEntities`를 둘 다 포함합니다. 작업 엔터티는 이 작업을 위해 정의되거나 생성된 엔터티이고, 컨텍스트 엔터티는 사용자와의 대화에서 엔터티를 보존할 수 있는 속성 모음입니다.

## <a name="return-value"></a>반환 값

**스크립트 트리거** 함수는 부울을 반환해야 합니다.

## <a name="sample-regex-based-recognizer"></a>샘플 regex 기반 인식기
다음 코드 샘플은 regex를 사용하여 요청을 처리하며 대화 런타임이 실행할 스크립트 트리거를 확인할 수 있도록 부울을 반환합니다.

```javascript
module.exports.fnFindPhoneTrigger = function(context) {
    if (context.request.text.includes("call") || context.request.text.includes("ring")) {
        return true;
    } else {
        return false;
    }
} 
```

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [회신 작업](conversation-designer-reply.md)
