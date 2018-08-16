---
title: Do 동작으로 회신 정의 | Microsoft Docs
description: Do 동작으로 회신을 정의하는 방법을 알아봅니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 344a771a55cf729587863a70398ef944863ac738
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303219"
---
# <a name="define-a-reply-as-a-do-action"></a>Do 동작으로 회신 정의

회신은 표시된 텍스트, 음성 텍스트 또는 리치(rich) 카드의 조합일 수 있습니다. 이 동작을 통해 봇은 사용자에게 회신을 보내고 작업을 완료합니다. 회신은 단일-턴(single-turn) 대화이므로 이 작업은 사용자의 후속 메시지가 필요하지 않습니다. 회신을 *회신* 동작으로 설정하려면 **DO** 동작 드롭다운에서 **Give a reply**(회신하기)를 선택합니다. 그런 다음, **Bot's response to user**(사용자에 대한 봇의 응답) 필드에 간단한 응답을 입력합니다.

선택적으로, 간단한 응답에 [adaptive card](conversation-designer-adaptive-cards.md)(적응형 카드)를 포함할 수 있습니다. 사용자에게 응답을 보내기 전에 실행할 사용자 지정 스크립트 함수를 선택적으로 제공할 수도 있습니다. 이러한 옵션은 작업을 만들거나 편집하는 동안 사용할 수 있습니다. 

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [스크립트 작업](conversation-designer-script-function.md)
