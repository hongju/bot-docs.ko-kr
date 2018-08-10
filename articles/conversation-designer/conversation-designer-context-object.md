---
title: 컨텍스트 개체의 API 참조 | Microsoft Docs
description: 대화 디자이너 봇에서 컨텍스트 개체를 참조하는 방법을 알아봅니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 5ba70189c16815539524d3c9046da03ed0344b9b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300976"
---
# <a name="api-reference"></a>API 참조
> [!IMPORTANT]
> 대화 디자이너는 일부 고객에게 제공되지 않습니다. 대화 디자이너의 가용성에 대한 자세한 내용은 올해 하반기에 제공됩니다.

대화 디자이너는 사용자 지정 비즈니스 논리를 봇에 추가하는 기능을 제공합니다. 이러한 스크립트 함수는 **스크립트** 편집기에서 구현됩니다. 함수는 조건부 응답 템플릿, 작업의 ‘트리거’나 ‘동작’ 또는 대화 상태 같은 다양한 위치에서 연결되고 모두 함수 매개 변수로 **[IConversationContext]** 형식의 `context` 개체를 수신합니다.

다음 코드 샘플은 **context** 개체가 전달된 조건부 응답 함수의 서명을 보여 줍니다.

```javascript
/**
* @param {IConversationContext} context
*/
module.exports.NewConditionalResponse_onRun = function(context) {
    // Business logic here
    return true; // Returns a boolean
}
```

`context` 개체를 통해 사용자와 봇 간의 대화 정보에 액세스할 수 있습니다.

## <a name="script-callback-functions"></a>스크립트 콜백 함수

직접 만든 사용자 지정 스크립트 콜백 함수에는 다양한 형식을 사용할 수 있습니다. 서로 다른 이름을 제공할 수 있지만 기능적으로 다음 형식 중 하나를 사용합니다.

| 형식 | 매개 변수 | 반환 형식 | 설명 |
| ---- | ---- | ---- | ---- |
| 응답 전 함수 | context | **void** 또는 **promise** | 응답이 제공되기 전에 실행되는 함수입니다. |
| 프로세스 함수 | context | **void** 또는 **promise** | 비즈니스 논리를 수행하는 함수입니다. |
| 의사 결정 함수 | context | **string** 또는 **promise** | 비즈니스 논리에 따라 결정하는 함수입니다. 반환 문자열은 [decision](conversation-designer-dialogues.md#decision-state) 블록의 조건과 일치해야 합니다. |
| 코드 인식기 함수 | context | **boolean** 또는 **promise** | **스크립트 트리거**가 발생할 때 실행되는 사용자 지정 비즈니스 논리입니다. `true`를 반환하여 일치 항목을 나타냅니다. 그렇지 않으면 `false`를 반환하여 일치 항목을 취소합니다. |
| onRecognize 함수 | context | **boolean** 또는 **promise** | LUIS 인식기에서 일치 항목이 있는 경우에만 실행되는 함수입니다. 이 콜백 함수를 사용하여 LUIS 엔터티를 처리하고 적절한 **boolean** 값을 반환합니다. `true`를 반환하여 일치 항목을 나타냅니다. 그렇지 않으면 `false`를 반환하여 일치 항목을 취소합니다. |

## <a name="iconversationcontext-interface"></a>IConversationContext 인터페이스

`IConversationContext` 인터페이스는 사용자와 봇 간의 대화 정보를 추적합니다. 대화 디자이너를 통해 연결되는 모든 사용자 지정 함수는 `context` 개체를 매개 변수 인수로 수신합니다.

## <a name="context-properties"></a>컨텍스트 속성
`context` 개체는 다음 속성을 공개합니다.

| Name |  코드 | 설명 |
| ---- | ---- | ---- |
| `request` | `context.request` | 봇의 활동이 포함된 요청 개체를 가져옵니다.  |
| | `context.request.attachment` | 적응형 카드를 포함할 수 있는 첨부 파일 활동입니다. |
| | `context.request.text` | 클라이언트에서 들어오는 텍스트 메시지를 포함하는 텍스트 활동입니다. |
| | `context.request.speak` | 클라이언트의 음성 텍스트(있는 경우)를 포함하는 음성 활동입니다. |
| | `context.request.type` | 활동 형식을 지정합니다(기본값: “message”). |
| `responses` | `context.responses` | 현재 상태 또는 코드 숨김 실행의 끝에 다시 클라이언트로 보낼 활동의 배열을 유지 관리합니다. |
| | `context.responses.push` | 응답에 활동을 추가합니다. |
| `global` | `context.global` | 사용자가 정의한 대화형 데이터를 포함하는 JavaScript 개체입니다. 이 개체는 대화 내내 지속됩니다. |
| `local` | `context.local` | 사용자가 정의한 작업 데이터를 포함하는 JavaScript 개체입니다. 이 개체는 특정 작업의 기간 동안 지속됩니다. LUIS 의도는 항상 로컬 컨텍스트로 반환됩니다. LUIS 결과를 지속하려면 `context.global` 컨텍스트에 복사하는 것이 좋습니다. |
| | `context.local['@description']` | LUIS에서 받은 원시 엔터티를 반환합니다. |
| `sticky` | `context.sticky` | 현재 작업 이름을 나타냄 |
| `currentTemplate` | `context.currentTemplate` | 표시 및 음성 평가를 위해 호출되는 [조건부 응답 템플릿](conversation-designer-response-templates.md#conditional-response-templates)입니다. 이 개체는 다음 세 가지 속성을 포함합니다. <br/>1. **name**: 현재 템플릿의 이름입니다. <br/>2. **modalityDisplay**: 형식이 표시 평가와 연결됨을 나타내는 부울입니다. <br/>3. **modalitySpeak**: 형식이 음성 평가와 연결됨을 나타내는 부울입니다. |

## <a name="context-methods"></a>컨텍스트 메서드
`context` 개체는 다음 메서드를 공개합니다.

| Name | 반환 형식 | 코드 | 설명 |
| ---- | ---- | ---- | ---- |
| `getCurrentTurn` | **number** | `context.getCurrentTurn();` | 재프롬프트를 실행할 경우 스택 맨 위의 프레임에서 회전을 가져옵니다. |

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [봇 만들기](conversation-designer-create-bot.md)
