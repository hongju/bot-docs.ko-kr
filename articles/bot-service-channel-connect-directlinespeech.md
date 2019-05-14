---
title: Direct Line Speech에 봇 연결(미리 보기)
titleSuffix: Bot Service
description: 안정성이 높으면서 대기 시간은 짧은 음성 입력 및 음성 출력 상호 작용을 위해 Direct Line Speech 채널에 기존 Bot Framework 봇을 연결하는 데 필요한 단계와 개요입니다.
services: bot-service
author: trrwilson
manager: nitinme
ms.service: bot-service
ms.subservice: bot-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: travisw
ms.custom: ''
ms.openlocfilehash: 8e0d2939078e1e27162c7056373e95790a03eb88
ms.sourcegitcommit: 5042e31bc6b2762d7a6636e98c8f496b90ea33c1
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/07/2019
ms.locfileid: "65240437"
---
# <a name="connect-a-bot-to-direct-line-speech-preview"></a>Direct Line Speech에 봇 연결(미리 보기)

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

Direct Line Speech 채널을 통해 클라이언트 애플리케이션과 통신하도록 봇을 구성할 수 있습니다.

봇을 빌드한 후 Direct Line Speech를 사용하여 온보딩하면 [Speech SDK](https://aka.ms/speech/sdk)를 사용하여 클라이언트 애플리케이션과 안정성이 높으면서 대기 시간은 짧은 연결을 구현할 수 있습니다. 이러한 연결은 음성 입력, 음성 출력 대화 환경에 최적화되었습니다. Direct Line Speech 및 클라이언트 애플리케이션 구축 방법에 대한 자세한 내용을 보려면 [사용자 지정 음성 우선 가상 도우미](https://aka.ms/bots/speech/va) 페이지를 방문하세요.  

## <a name="sign-up-for-direct-line-speech-preview"></a>Direct Line Speech 미리 보기에 가입

Direct Line Speech는 현재 미리 보기 상태이며 [Azure Portal](https://portal.azure.com)에서 간단한 가입 절차를 거쳐야 사용 가능합니다. 아래에서 자세한 내용을 참조하세요. 승인되면 채널에 액세스할 수 있습니다.

## <a name="add-the-direct-line-speech-channel"></a>Direct Line Speech 채널 추가

1. Direct Line Speech 채널을 추가하려면 먼저 [Azure Portal](https://portal.azure.com)에서 봇을 열고, 구성 블레이드에서 **채널**을 클릭합니다.

    ![연결할 채널을 선택하는 위치 강조 표시](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-selectchannel.png "채널 선택")

1. 채널 선택 페이지에서 `Direct Line Speech`를 찾아서 클릭하여 채널을 선택합니다.

    ![direct line speech 채널 선택](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-connectspeechchannel.png "Direct Line Speech 연결")

1. 아직 액세스가 승인되지 않았다면 액세스 요청 페이지가 표시됩니다. 요청된 정보를 입력하고 '요청'을 클릭합니다. 확인 페이지가 표시됩니다. 요청이 승인 보류 중 상태일 때는 이 페이지를 넘어갈 수 없습니다.   

1. 액세스가 승인되면 Direct Line Speech 구성 페이지가 표시됩니다. 사용 약관을 검토한 후 `Save`를 클릭하여 채널 선택을 확인합니다.

    ![Direct Line Speech 채널 구현 저장](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-savechannel.png "채널 구성 저장")

## <a name="enable-the-bot-framework-protocol-streaming-extensions"></a>Bot Framework 프로토콜 스트리밍 확장 사용

Direct Line Speech 채널이 봇에 연결되었으므로 이제 대기 시간이 짧은 최적의 상호 작용을 위해 Bot Framework 프로토콜 스트리밍 확장 지원을 사용하도록 설정해야 합니다.

1. [Azure Portal](https://portal.azure.com)에서 봇의 블레이드를 엽니다(아직 열지 않은 경우). 

1. **채널** 바로 아래에 있는 **봇 관리** 카테고리에서 **설정**을 클릭합니다. **스트리밍 엔드포인트 사용** 확인란을 클릭합니다.

    ![스트리밍 프로토콜 사용](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablestreamingsupport.png "스트리밍 확장 지원 사용")

1. 페이지 위쪽에서 **저장**을 클릭합니다.

1. 동일한 블레이드의 **App Service 설정** 카테고리에서 **구성**을 클릭합니다.

    ![app service 설정으로 이동](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-configureappservice.png "app service 구성")

1. `General settings`를 클릭한 후 `Web socket` 지원을 사용하는 옵션을 선택합니다.

    ![app service에 websocket 사용](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablewebsockets.png "websocket 사용")

1. 구성 페이지의 맨 위에 있는 `Save`를 클릭합니다.

1. 이제 봇에서 Bot Framework 프로토콜 스트리밍 확장을 사용할 수 있습니다. 이제 봇 코드를 업데이트하고 기존 봇 프로젝트에 [스트리밍 확장 지원을 통합](https://aka.ms/botframework/addstreamingprotocolsupport)할 준비가 되었습니다.

## <a name="manage-secret-keys"></a>비밀 키 관리

클라이언트 애플리케이션이 Direct Line Speech 채널을 통해 봇에 연결할 때는 채널 비밀이 필요합니다. 채널 선택을 저장했으면 Azure Portal의 **Direct Line Speech 구성** 페이지에서 이러한 비밀 키를 검색할 수 있습니다.

![Direct Line Speech용 비밀 키 가져오기](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-getspeechsecretkeys.png "Direct Line Speech용 비밀 키 가져오기")

## <a name="adding-protocol-support-to-your-bot"></a>봇에 프로토콜 지원 추가

Direct Line Speech 채널을 연결하고 Bot Framework 프로토콜 스트리밍 확장을 구현한 후에는 봇에 코드를 추가하기만 하면 최적화된 통신이 지원됩니다. [봇에 스트리밍 확장 지원 추가](https://aka.ms/botframework/addstreamingprotocolsupport)의 지침을 따라 Direct Line Speech와의 완전한 호환성을 보장하세요.

## <a name="known-issues"></a>알려진 문제

이 서비스는 미리 보기 상태이며 봇 개발 및 전반적인 성능에 영향을 줄 수 있는 변경 사항이 생길 수 있습니다. 다음은 알려진 문제의 목록입니다. 

1. 이 서비스는 현재 [Azure 지역](https://azure.microsoft.com/en-us/global-infrastructure/regions/) 미국 서부 2에 배포되어 있습니다. 모든 고객이 봇과 대기 시간이 짧은 음성 상호 작용의 이점을 누릴 수 있도록 다른 지역에도 곧 배포될 예정입니다.

1. 제어 필드(예: [serviceUrl](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#service-url))에 대한 사소한 변경 사항이 있을 예정입니다.

1. 환영 메시지를 생성할 때 일반적으로 사용되며, 대화의 시작과 끝을 알리는 [conversationUpdate](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation-update-activity) 및 [endOfCoversation](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#end-of-conversation-activity) 작업이 다른 채널과의 일관성을 위해 업데이트될 예정입니다.

1. [SigninCard](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-add-rich-cards?view=azure-bot-service-4.0)는 채널에서 아직 지원되지 않습니다. 
