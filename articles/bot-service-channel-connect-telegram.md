---
title: Telegram용 봇 만들기 | Microsoft Docs
description: Telegram에 대한 봇 연결을 구성하는 방법을 알아봅니다.
keywords: 봇 구성성, Telegram, 봇 채널, Telegram 봇, 액세스 토큰
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: efe38392117fb871b2b98e3f1d8d798bfaef0c41
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563763"
---
# <a name="connect-a-bot-to-telegram"></a>Telegram에 봇 연결

Telegram 메시징 앱을 사용하여 사용자와 통신하도록 봇을 구성할 수 있습니다.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="visit-the-bot-father-to-create-a-new-telegram-bot"></a>Bot Father를 방문하여 새 Telegram 봇 만들기

Bot Father를 사용하여 <a href="https://telegram.me/botfather" target="_blank">새 Telegram 봇을 만듭니다</a>.

![Bot Father 방문](~/media/channels/tg-StepVisitBotFather.png)

## <a name="create-a-new-telegram-bot"></a>새 Telegram 봇 만들기
새 Telegram 봇을 만들려면 명령 `/newbot`을 전송합니다.

![새 봇 만들기](~/media/channels/tg-StepNewBot.png)

### <a name="specify-a-friendly-name"></a>친숙한 이름 지정

Telegram 봇에 친숙한 이름을 지정합니다.

![봇에 친숙한 이름 제공](~/media/channels/tg-StepNameBot.png)

### <a name="specify-a-username"></a>사용자 이름 지정

Telegram 봇에 고유한 이름을 제공합니다.

![봇에 고유한 이름 제공](~/media/channels/tg-StepUsername.png)

### <a name="copy-the-access-token"></a>액세스 토큰 복사

Telegram 봇의 액세스 토큰을 복사합니다.

![액세스 토큰 복사](~/media/channels/tg-StepBotCreated.png)

## <a name="enter-the-telegram-bots-access-token"></a>Telegram 봇의 액세스 토큰 입력

이전에 복사한 토큰을 **액세스 토큰** 필드에 붙여넣고 **제출**을 클릭합니다.

## <a name="enable-the-bot"></a>봇 사용
**Enable this bot on Telegram**(Telegram에서 이 봇 사용)을 선택합니다. 그런 다음, **I'm done configuring Telegram**(Telegram 구성 완료)을 클릭합니다.

이러한 단계를 완료한 경우, 봇은 Telegram에서 사용자와 통신하도록 성공적으로 구성됩니다.
