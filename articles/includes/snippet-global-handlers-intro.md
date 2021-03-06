---
ms.openlocfilehash: b5809b6d46cdc09035efb36c3ea58c2ca9dc6c3e
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563627"
---
사용자는 일반적으로 “도움말”, “취소” 또는 “다시 시작”과 같은 키워드를 사용하여 봇의 특정 기능에 액세스하려고 합니다. 이런 상황은 대화 도중 봇이 다른 응답을 예상하는 경우에 자주 발생합니다. **전역 메시지 처리기**를 구현하면 이러한 요청을 정상적으로 처리하도록 봇을 디자인할 수 있습니다.
처리기는 사용자 입력에서 “도움말”, “취소” 또는 “다시 시작”과 같은 지정한 키워드를 검사하고 적절하게 응답합니다. 

![사용자의 대화 방법](~/media/designing-bots/capabilities/trigger-actions.png)

> [!NOTE]
> 전역 메시지 처리기에 논리를 정의하여 모든 대화 상자에서 논리에 액세스할 수 있게 합니다. 개별 대화 상자 및 프롬프트가 키워드를 안전하게 무시하도록 구성할 수 있습니다.
