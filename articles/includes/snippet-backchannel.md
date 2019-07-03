---
ms.openlocfilehash: b320aadc876074a76fe209ad55a81cb70b1ddcac
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405546"
---
<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">오픈 소스 웹 채팅 컨트롤</a>은 클라이언트와 봇 사이에 `activities`를 전송하는 [Direct Line API](https://docs.botframework.com/restapi/directline3/#navtitle)를 사용하여 봇과 통신합니다. 작업의 가장 일반적인 형식은 `message`이지만 다른 형식도 있습니다. 예를 들어, 작업 형식 `typing`은 사용자가 입력 중이거나 봇이 응답을 컴파일하기 위해 작업 중임을 나타냅니다. 

작업 형식을 `event`로 설정하면 백채널 메커니즘을 사용하여 사용자에게 이를 표시하지 않으면서 클라이언트와 봇 간에 정보를 교환할 수 있습니다. 웹 채팅 컨트롤은 자동으로 `type="event"`가 있는 작업을 무시합니다.