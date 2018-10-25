---
title: Bot Framework User-Agent 요청 | Microsoft Docs
description: 봇 프레임워크에서 웹 서버로의 요청 이해
author: JohnD-ms
ms.author: v-jodemp
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: befd46cc13fa10a2d1a0f8cf2beed4cb041ab3bd
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998543"
---
# <a name="bot-framework-user-agent-requests"></a>Bot Framework User-Agent 요청

이 메시지를 읽는 경우 Microsoft Bot Framework 서비스에서 요청을 받았을 것입니다. 이 가이드는 이러한 요청의 특성을 이해하도록 돕고 원하는 경우 중지하는 단계를 제공합니다.

서비스에서 요청을 받은 경우 아래 문자열과 유사하게 User-Agent 헤더가 서식이 지정되었을 수 있습니다.

```User-Agent: BF-DirectLine/3.0 (Microsoft-BotFramework/3.0 +https://botframework.com/ua)```

이 문자열의 가장 중요한 부분은 **Microsoft BotFramework** 식별자입니다. 이는 Microsoft Bot Framework에서 사용되는 도구 및 서비스의 컬렉션으로 독립 소프트웨어 개발자가 고유한 봇을 만들고 운영할 수 있도록 합니다.

봇 개발자인 경우 이러한 요청이 서비스에 전달되는 이유를 이미 알고 있을 것입니다. 그렇지 않은 경우 자세히 알아보려면 읽기를 계속합니다.

## <a name="why-is-microsoft-contacting-my-service"></a>Microsoft에서 내 서비스에 연결하는 이유는?

Bot Framework는 Skype 및 Facebook Messenger와 같은 채팅 서비스의 사용자를 봇에 연결합니다. 이는 인터넷 액세스 가능한 엔드포인트에서 실행되는 REST API를 사용하는 웹 서버입니다. 봇에 대한 HTTP 호출(웹후크 호출이라고도 함)은 Bot Framework 개발자 포털로 등록된 봇 개발자에 의해 지정된 URL로만 전송됩니다.

Bot Framework 서비스에서 웹 서비스로 원하지 않는 요청을 받는 경우 개발자가 해당 봇에 대한 웹후크 콜백으로 URL을 실수로 또는 의도적으로 입력했기 때문일 수 있습니다.

## <a name="to-stop-these-requests"></a>이러한 요청을 중지하려면

원치 않는 요청이 웹 서비스에 도달하는 것을 중지하는 데 도움이 필요한 경우 [bf-reports@microsoft.com](mailto://bf-reports@microsoft.com)에 문의하세요.
