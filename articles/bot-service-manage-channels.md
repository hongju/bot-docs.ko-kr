---
title: 하나 이상의 채널에서 실행되도록 봇 구성 | Microsoft Docs
description: Bot Framework Portal을 사용하여 하나 이상의 채널에서 실행되도록 봇을 구성하는 방법에 대해 알아봅니다.
keywords: 봇 채널, 구성, cortana, facebook messenger, kik, slack, skype, azure portal
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: cb682bf77f801c98d00deffa0fc63249962248cd
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352922"
---
# <a name="connect-a-bot-to-channels"></a>채널에 봇 연결

채널은 통신 앱과 Bot Framework 사이의 연결입니다. 봇을 사용할 수 있기를 원하는 채널에 연결하도록 구성합니다. 예를 들어 Skype 채널에 연결된 봇은 연락처 목록에 추가될 수 있으며 이를 통해 Skype에서 사용자들이 상호 작용할 수 있습니다. 

채널에는 [Cortana](bot-service-channel-connect-cortana.md), [Facebook Messenger](bot-service-channel-connect-facebook.md), [Kik](bot-service-channel-connect-kik.md), [Slack](bot-service-channel-connect-slack.md) 및 기타 여러 서비스와 같은 많은 유명한 서비스가 포함됩니다. [Skype](https://dev.skype.com/bots) 및 Web Chat은 사용자를 위해 미리 구성됩니다. 

[Azure Portal](https://portal.azure.com)에서 채널 연결을 쉽고 빠르게 할 수 있습니다.

## <a name="get-started"></a>시작하기

대부분의 채널의 경우 채널에서 봇을 실행하려면 채널 구성 정보를 제공해야 합니다. 대부분의 채널의 경우 봇이 채널에 계정이 있어야 하며, Facebook Messenger 등의 경우에는 봇이 채널에도 등록된 응용 프로그램이 있어야 합니다.

봇이 채널에 연결되도록 구성하려면 다음 단계를 수행합니다.

1. <a href="https://portal.azure.com" target="_blank">Azure Portal</a>에 로그인합니다.
1. 구성하려는 봇을 선택합니다.
3. Bot Service 블레이드의 **봇 관리**에서 **채널**을 클릭합니다.
4. 봇에 추가하려는 채널의 아이콘을 클릭합니다.

![채널에 연결](~/media/channels/connect-to-channels.png)

채널을 구성한 후 해당 채널에서 사용자는 봇을 사용하여 시작할 수 있습니다.

## <a name="publish-a-bot"></a>봇 게시

게시 프로세스는 각 채널마다 다릅니다.

[!INCLUDE [publishing](~/includes/snippet-publish-to-channel.md)]

