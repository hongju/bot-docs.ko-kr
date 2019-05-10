---
ms.openlocfilehash: cf0e23e349ace78958f861ea2e650a5ebcb78466
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563753"
---
<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">오픈 소스 웹 채팅 컨트롤</a>은 클라이언트와 봇 사이에 `activities`를 전송하는 [Direct Line API](https://docs.botframework.com/en-us/restapi/directline3/#navtitle)를 사용하여 봇과 통신합니다. 작업의 가장 일반적인 형식은 `message`이지만 다른 형식도 있습니다. 예를 들어, 작업 형식 `typing`은 사용자가 입력 중이거나 봇이 응답을 컴파일하기 위해 작업 중임을 나타냅니다. 

작업 형식을 `event`로 설정하면 백채널 메커니즘을 사용하여 사용자에게 이를 표시하지 않으면서 클라이언트와 봇 간에 정보를 교환할 수 있습니다. 웹 채팅 컨트롤은 자동으로 `type="event"`가 있는 작업을 무시합니다.