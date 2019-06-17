---
title: 추가 채널 | Microsoft Docs
description: 봇에 대한 추가 채널을 구성하는 방법을 알아봅니다.
keywords: 봇 채널, 행아웃, Twilio, 피드백, Azure Portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/08/2019
ms.openlocfilehash: d6cf068f3cd4d57ff9cde2be6d41e084cee6524d
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693743"
---
# <a name="additional-channels"></a>추가 채널

이 문서 내에 나열된 채널 외에, Botkit을 통해 [제공된 플랫폼](https://botkit.ai/docs/v4/platforms/)을 통해 어댑터로 사용하거나 [커뮤니티 리포지토리](https://github.com/BotBuilderCommunity/)를 통해 액세스할 수 있는 몇 가지 채널이 더 있습니다. 제공되는 추가 채널은 다음과 같습니다.

- [Webex 팀](https://botkit.ai/docs/v4/platforms/webex.html)
- [Websocket 및 Webhook](https://botkit.ai/docs/v4/platforms/web.html)
- [Google Hangout 및 Google Assistant](https://github.com/BotBuilderCommunity/)(커뮤니티를 통해 사용 가능)
- [Amazon Alexa](https://github.com/BotBuilderCommunity/)(커뮤니티를 통해 사용 가능)

## <a name="currently-available-adapters"></a>현재 사용 가능한 어댑터

사용 가능한 어댑터의 전체 목록은 [여기에서 찾을 수 있습니다](https://botkit.ai/docs/v4/platforms/). 일부 채널은 어댑터로도 사용할 수 있으며 채널과 어댑터를 사용하는 것은 사용자의 판단입니다.

### <a name="when-to-use-an-adapter"></a>어댑터를 사용하는 경우

1. 원하는 채널을 서비스가 지원하지 않습니다.
2. 배포에 대한 보안 및 규정 준수 요구 사항에 따라 외부 서비스에 의존할 수 없습니다.
3. 특정 채널에 필요한 수준의 기능이 지원되지 않을 수 있습니다.

### <a name="when-to-use-a-channel"></a>채널을 사용하는 경우

1. 채널 간 호환성이 필요합니다. 사용 가능한 채널 중 둘 이상에서 봇이 작동해야 합니다.
2. 기본 지원: 타사의 업데이트가 있을 때마다, 각 채널이 유지 관리되고 패치가 적용되며 원활하게 서비스가 제공됩니다.
3. 신속하게 성장하는 Microsoft 팀처럼, 추가 독점 Microsoft 채널에 액세스할 수 있습니다.
4. GUI 인터페이스를 사용하여 봇에 추가 채널을 사용하려는 경우입니다.
