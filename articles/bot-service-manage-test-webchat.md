---
title: 웹 채팅에서 Bot Service 테스트 | Microsoft Docs
description: Azure Portal에서 웹 채팅 컨트롤을 사용하여 Bot Service를 테스트하는 방법을 알아봅니다.
keywords: 웹 채팅에서 테스트, Azure Portal
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 0c358f4e53f3fd64cce3635f644cc0f2f612e983
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300677"
---
# <a name="test-in-web-chat"></a>웹 채팅에서 테스트
Bot Service에는 편리하게 봇을 테스트할 수 있도록 지원하는 [웹 채팅 컨트롤](bot-service-channel-connect-webchat.md)이 포함되어 있습니다. 

## <a name="test-a-bot-in-the-azure-portal-with-web-chat"></a>웹 채팅을 사용하여 Azure Portal에서 봇 테스트
[Azure Portal](https://portal.azure.com)에 로그인하고 봇 블레이드를 엽니다. 봇 관리 섹션에서 **웹 채팅에서 테스트**를 클릭합니다. Bot Service가 웹 채팅 컨트롤을 로드하고 봇에 연결합니다.

![웹 채팅 UI에서 테스트](~/media/test-in-webchat/test-in-webchat.png)

텍스트를 입력하여 봇과 채팅할 수 있습니다. 봇이 음성을 지원하는 경우 마이크 단추를 클릭하여 봇에 말할 수 있습니다. 봇이 첨부 파일을 지원하는 경우 이미지와 같은 첨부 파일을 업로드할 수 있습니다. [Bot Builder SDK](bot-builder-overview-getstarted.md)를 사용하여 봇에 기능을 추가하는 방법을 알아봅니다.

> [!NOTE]
> 몇 분 후에도 웹 채팅 컨트롤이 완전히 로드되지 않으면 페이지를 새로 고쳐 보세요.

## <a name="next-steps"></a>다음 단계
Azure에서 봇을 테스트하는 방법을 알아보았으므로, 이제 심층 테스트 및 디버깅 기능을 위해 Bot Framework Emulator를 활용하는 방법을 알아봅니다.

> [!div class="nextstepaction"]
> [Bot Framework Emulator](bot-service-debug-emulator.md)