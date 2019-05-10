---
ms.openlocfilehash: 8e5677fe59dd9edad6ac1da9d029e5f7c08bf179
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563966"
---
## <a name="next-steps"></a>다음 단계
봇을 클라우드에 배포하고 Bot Framework Emulator로 봇을 테스트하여 배포가 성공했는지 확인한 후, 봇 게시 프로세스의 다음 단계는 Bot Framework에 봇을 등록했는지 여부에 따라 달라집니다.

### <a name="if-you-have-already-registered-your-bot-with-the-bot-framework"></a>Bot Framework에 봇을 이미 등록한 경우:

1. <a href="https://dev.botframework.com" target="_blank">Bot Framework 포털</a>로 돌아가서 봇에 대한 **메시지 엔드포인트**를 지정하도록 [봇의 설정 데이터](~/bot-service-manage-settings.md)를 업데이트합니다.

2. [하나 이상의 채널에서 실행되도록 봇을 구성합니다](~/bot-service-manage-channels.md).

### <a name="if-you-have-not-yet-registered-your-bot-with-the-bot-framework"></a>Bot Framework에 봇을 아직 등록하지 않은 경우:

1. [Bot Framework에 봇을 등록합니다](~/bot-service-quickstart-registration.md).

2. 등록 과정에서 봇에 대해 생성된 **appID** 및 **암호** 값을 지정하여 배포된 애플리케이션의 구성 설정에서 Microsoft 앱 ID 및 Microsoft 앱 암호 값을 업데이트합니다. 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

3. [하나 이상의 채널에서 실행되도록 봇을 구성합니다](~/bot-service-manage-channels.md).