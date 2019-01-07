---
title: 봇 구성 문제 해결 | Microsoft Docs
description: 배포된 봇의 구성 문제를 해결하는 방법입니다.
keywords: 문제 해결, 구성, 웹 채팅, 문제.
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/20/2018
ms.openlocfilehash: 18350c7ea6fb7390796567b1754cd88be6b9be61
ms.sourcegitcommit: 561185b9c83f3e082e8b7aba1122b1706e431540
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/26/2018
ms.locfileid: "53785416"
---
# <a name="troubleshoot-bot-configuration-issues"></a>봇 구성 문제 해결

봇의 문제를 해결하는 첫 번째 단계는 웹 채팅에서 테스트하는 것입니다. 이렇게 하면 봇과 관련된 문제인지(봇이 모든 채널에서 작동하지 않음), 특정 채널과 관련된 문제인지(봇이 일부 채널에서 작동하지만 다른 채널에서는 작동하지 않음) 확인할 수 있습니다.

## <a name="test-in-web-chat"></a>웹 채팅에서 테스트

1. [Azure Portal](http://portal.azure.com/)에서 봇 리소스를 엽니다.
1. **웹 채팅에서 테스트** 창을 엽니다.
1. 봇에 메시지를 보냅니다.

![웹 채팅에서 테스트](./media/test-in-webchat.png)

봇이 올바른 출력으로 응답할 경우 [봇이 웹 채팅에서 작동하지 않음](#bot-does-not-work-in-web-chat)으로 이동하세요. 그렇지 않을 경우 [봇이 웹 채팅에서 작동하지만 다른 채널에서 작동하지 않음](#bot-works-in-web-chat-but-not-in-other-channels)으로 이동하세요.

## <a name="bot-does-not-work-in-web-chat"></a>봇이 웹 채팅에서 작동하지 않음

봇이 작동하지 않는 데는 다양한 이유가 있을 수 있습니다. 봇 애플리케이션이 다운되어 메시지를 수신할 수 없거나 봇이 메시지를 수신하지만 응답하지 못했을 가능성이 가장 높습니다.

봇이 실행되는지 확인하려면:

1. **개요** 창을 엽니다.
1. **메시징 엔드포인트**를 복사하여 브라우저에 붙여넣습니다.

엔드포인트가 HTTP 오류 405를 반환한다면 봇에 연결할 수 있고, 봇이 메시지에 응답할 수 있다는 의미입니다. 봇이 [제한 시간을 초과](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md)했는지, [HTTP 5xx 오류와 함께 실패](bot-service-troubleshoot-500-errors.md)했는지 조사해야 합니다.

엔드포인트에서 "이 사이트에 연결할 수 없음" 또는 "이 페이지에 연결할 수 없음" 오류를 반환한다면 봇이 다운되었고 봇을 다시 배포해야 한다는 의미입니다.

## <a name="bot-works-in-web-chat-but-not-in-other-channels"></a>봇이 웹 채팅에서 작동하지만 다른 채널에서 작동하지 않음

웹 채팅에서 봇이 예상대로 작동하지 않고 다른 채널에서 실패할 경우 가능한 이유는 다음과 같습니다.

- [채널 구성 문제](#channel-configuration-issues)
- [채널 관련 동작](#channel-specific-behavior)
- [채널 중단](#channel-outage)

### <a name="channel-configuration-issues"></a>채널 구성 문제

채널 구성 매개 변수가 잘못 설정되었거나 외부에서 변경되었을 수 있습니다. 예를 들어, 봇이 특정 페이지에 대한 Facebook 채널을 구성했는데 해당 페이지가 나중에 삭제된 경우가 있습니다. 가장 간단한 해결 방법은 채널을 제거하고 채널 구성 작업을 처음부터 다시 수행하는 것입니다.

아래 링크는 Bot Framework에서 지원하는 채널을 구성하기 위한 지침을 제공합니다.

- [Cortana](bot-service-channel-connect-cortana.md)
- [DirectLine](bot-service-channel-connect-directline.md)
- [Email](bot-service-channel-connect-email.md)
- [Facebook](bot-service-channel-connect-facebook.md)
- [GroupMe](bot-service-channel-connect-groupme.md)
- [Kik](bot-service-channel-connect-kik.md)
- [Microsoft 팀](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Skype](bot-service-channel-connect-skype.md)
- [비즈니스용 Skype](bot-service-channel-connect-skypeforbusiness.md)
- [Slack](bot-service-channel-connect-slack.md)
- [Telegram](bot-service-channel-connect-telegram.md)
- [Twilio](bot-service-channel-connect-twilio.md)

### <a name="channel-specific-behavior"></a>채널 관련 동작

일부 기능의 구현 방식은 채널마다 다를 수 있습니다. 예를 들어 일부 채널은 적응형 카드를 지원하지 않습니다. 대부분의 채널은 단추를 지원하지만 채널 관련 방식으로 렌더링됩니다. 일부 메시지 유형이 채널별로 다르게 작동하는 것이 확인될 경우 [Channel Inspector](https://docs.botframework.com/channel-inspector/channels/Skype)에게 문의하세요.

다음은 개별 채널에 도움이 되는 몇 가지 추가 링크입니다.

- [Microsoft Teams 앱에 봇 추가](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Facebook: Messenger 플랫폼 소개](https://developers.facebook.com/docs/messenger-platform/introduction)
- [Cortana 기술의 디자인 원칙](https://docs.microsoft.com/cortana/skills/design-principles)
- [개발자용 Skype](https://dev.skype.com/bots)
- [Slack: 봇과의 상호 작용 구현](https://api.slack.com/bot-users)

### <a name="channel-outage"></a>채널 중단

간혹 일부 채널의 서비스가 중단되는 경우가 있을 수 있습니다. 일반적으로 이러한 중단은 오래 지속되지 않습니다. 그러나 중단이 의심될 경우 채널 웹 사이트 또는 소셜 미디어를 참조하세요.

채널이 중단되었는지 확인하는 또 다른 방법은 테스트 봇(예: 간단한 Echo Bot)을 만들고 채널을 추가하는 것입니다. 테스트 봇이 일부 채널에서 작동하지만 다른 채널에서는 작동하지 않을 경우 프로덕션 봇의 문제가 아닐 수 있습니다.
