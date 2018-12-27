---
title: 하나 이상의 채널에서 실행되도록 봇 구성 | Microsoft Docs
description: Bot Framework Portal을 사용하여 하나 이상의 채널에서 실행되도록 봇을 구성하는 방법에 대해 알아봅니다.
keywords: 봇 채널, 구성, cortana, facebook messenger, kik, slack, skype, azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/22/2018
ms.openlocfilehash: f5ca01b592266d005c8f000e2351ddcf812acb6d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997387"
---
# <a name="connect-a-bot-to-channels"></a>채널에 봇 연결

채널은 봇과 통신 앱 사이의 연결입니다. 봇을 사용할 수 있기를 원하는 채널에 연결하도록 구성합니다. Azure Portal을 통해 구성된 Bot Framework Service는 봇을 이러한 채널에 연결하고, 봇과 사용자 간 통신을 용이하게 합니다. [Cortana](bot-service-channel-connect-cortana.md), [Facebook Messenger](bot-service-channel-connect-facebook.md), [Kik](bot-service-channel-connect-kik.md), [Slack](bot-service-channel-connect-slack.md) 및 기타 여러 서비스와 같은 많은 유명한 서비스에 연결할 수 있습니다. [Skype](https://dev.skype.com/bots) 및 Web Chat은 사용자를 위해 미리 구성됩니다. Bot Connector Service를 통해 제공되는 표준 채널 외에도 직접 회선을 채널로 사용하여 고유한 클라이언트 응용 프로그램에 봇을 연결할 수도 있습니다.

Bot Framework Service를 사용하면 봇이 채널에 전송하는 메시지를 정규화하여 채널 독립적인 방법으로 봇을 개발할 수 있습니다. 이는 봇 작성기 스키마에서 채널의 스키마로 변환하는 것을 포함합니다. 단, 채널이 봇 작성기 스키마 중 일부를 지원하지 않는 경우 서비스는 메시지를 채널이 지원하는 형식으로 변환하려고 시도합니다. 예를 들어, 봇이 작업 단추가 있는 카드가 포함된 메시지를 SMS 채널에 보내면 커넥터는 카드를 이미지로 전송하고 메시지의 텍스트에 링크로 작업을 포함시킬 수 있습니다.



대부분의 채널의 경우 채널에서 봇을 실행하려면 채널 구성 정보를 제공해야 합니다. 대부분의 채널의 경우 봇이 채널에 계정이 있어야 하며, Facebook Messenger 등의 경우에는 봇이 채널에도 등록된 애플리케이션이 있어야 합니다.

봇이 채널에 연결되도록 구성하려면 다음 단계를 수행합니다.

1. <a href="https://portal.azure.com" target="_blank">Azure Portal</a>에 로그인합니다.
1. 구성하려는 봇을 선택합니다.
3. Bot Service 블레이드의 **봇 관리**에서 **채널**을 클릭합니다.
4. 봇에 추가하려는 채널의 아이콘을 클릭합니다.

![채널에 연결](./media/channels/connect-to-channels.png)

채널을 구성한 후 해당 채널에서 사용자는 봇을 사용하여 시작할 수 있습니다.

## <a name="publish-a-bot"></a>봇 게시

게시 프로세스는 각 채널마다 다릅니다.

[!INCLUDE [publishing](./includes/snippet-publish-to-channel.md)]

## <a name="additional-resources"></a>추가 리소스
SDK에는 봇을 빌드하는 데 사용할 수 있는 샘플이 포함되어 있습니다. [GitHub](https://github.com/Microsoft/BotBuilder-samples) 리포지토리를 방문하여 샘플 목록을 참조하세요.
