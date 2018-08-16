---
title: 봇 테스트 | Microsoft Docs
description: 대화 디자이너 봇을 테스트 및 디버깅합니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: b08bd96bc7c413ff7cb6db4899c8c0d3ad613ab8
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304218"
---
# <a name="test-your-conversation-designer-bot"></a>대화 디자이너 봇 테스트
> [!IMPORTANT]
> 대화 디자이너는 일부 고객에게 제공되지 않습니다. 대화 디자이너의 가용성에 대한 자세한 내용은 올해 하반기에 제공됩니다.

대화 디자이너는 여러 다른 출력 창을 통해 유용한 디버깅 도구를 제공합니다(화면 맨 아래에 있음). 창의 오른쪽 위 모서리에 있는 **테스트** 단추를 클릭하여 테스트 및 디버깅을 시작할 수 있습니다. 

## <a name="validation-errors"></a>유효성 검사 오류
유효성 검사 오류가 발생하는 몇 가지 조건이 있습니다. 예:  
- 트리거에 응답을 누락한 경우 
- 흐름에 대한 정보를 부분적으로 완료한 경우
- 조건부 응답 템플릿, 의사 결정 및 프로세스 상태에 대한 누락된 트리거 및 코드 숨김
- 템플릿 참조에서 재귀 또는 무한 루프가 검색된 경우 
- 사용자 대화 상자에 흐름에 연결되어 있지 않은 분리된 상태가 있는 경우
- 작업을 추가했지만 트리거 또는 작업이 누락된 경우 


## <a name="javascript-output"></a>JavaScript 출력
추가 기능을 제공하는 응답 실행에 스크립트를 연결할 수 있습니다. 스크립트에 오류가 있으면 여기에 표시됩니다. `console.log()`, `error.log()` 또는 `debug.log()`를 봇의 비즈니스 논리에 추가할 수 있습니다. 그러면 출력 창에 메시지가 나열됩니다. 예: 

``` javascript
module.exports.Respond_beforeResponse = function(context) {
    console.log(JSON.stringify(context));
}
```

## <a name="runtime-output"></a>Runtime 출력
런타임에 의해 생성된 오류 또는 예외가 여기에 표시됩니다. 예를 들어 봇이 응답에 실패하는 경우 예외 또는 오류를 일으킨 원인을 조사하려면 출력을 확인합니다. 출력 창에 너무 많은 메시지가 있을 경우 **모두 지우기**를 클릭한 다음, 오류 세부 정보를 확인하도록 봇에 메시지를 보내 봇을 다시 테스트합니다. 

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [봇 가져오기 및 내보내기](conversation-designer-export-import-bot.md)
