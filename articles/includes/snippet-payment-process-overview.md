---
ms.openlocfilehash: 86faca976bc95a5e91e17749096cd148483edc61
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230704"
---
## <a name="payment-process-overview"></a>결제 프로세스 개요

결제 프로세스는 세 부분으로 구성됩니다.

1. 봇에서 결제 요청을 보냅니다.

2. 사용자는 Microsoft 계정에 로그인하여 결제, 배송 및 연락처 정보를 제공합니다. 봇에서 특정 작업을 수행해야 하는 경우를 나타내도록 봇에 콜백이 전송됩니다(배송 주소 업데이트, 배송 옵션 업데이트, 결제 완료).

3. 봇에서 배송 주소 업데이트, 배송 옵션 업데이트 및 결제 완료를 포함하여 수신하는 콜백을 처리합니다. 

봇은 이 프로세스의 1단계 및 3단계만을 구현해야 합니다. 2단계는 봇의 컨텍스트 외부에서 이루어집니다. 
