---
ms.openlocfilehash: 91b2729aa10c8ac1985e62845126296e8141ef77
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230640"
---
> [!IMPORTANT]
> 기본 봇 턴을 처리하는 스레드는 사용을 마친 컨텍스트 개체를 폐기합니다. 턴 컨텍스트의 처리 및 폐기가 완료될 때까지 기본 스레드가 생성된 작업을 기다리도록 **작업 호출을 `await`해야 합니다.** 또는 응답(처리기 포함)에 상당한 시간이 소요되고 응답이 컨텍스트 개체에 따라 작동하는 경우 _컨텍스트가 삭제되었습니다_ 오류가 발생할 수 있습니다.