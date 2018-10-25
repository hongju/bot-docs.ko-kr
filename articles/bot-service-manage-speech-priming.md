---
title: 음성 초기화 구성 | Microsoft Docs
description: Azure Portal을 사용하여 봇 서비스에 대해 음성 초기화를 구성하는 방법을 알아봅니다.
keywords: 음성 초기화, 음성 인식, LUIS
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 12/13/2017
ms.openlocfilehash: 5cb47be530f9f82d83272684e6405730c72f3cb7
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997156"
---
# <a name="configure-speech-priming"></a>음성 초기화 구성

음성 초기화는 봇에서 일반적으로 사용되는 음성 및 구의 인식을 향상시킵니다. 웹 채팅 및 Cortana 채널을 사용하는 음성 지원 봇의 경우, 음성 초기화는 LUIS(Language Understanding) 앱에 지정된 예제를 사용하여 중요한 단어에 대한 음성 인식 정확도를 개선합니다.

봇이 LUIS 앱과 이미 통합되어 있을 수도 있고, 사용자가 음성 초기화를 위해 봇에 연결할 LUIS 앱을 만들도록 선택할 수 있습니다. LUIS 앱에는 사용자가 봇에 말할 것으로 예상되는 발언 예제가 포함되어 있습니다. 봇이 인식하도록 하려는 중요한 단어에는 엔터티로 레이블을 지정해야 합니다. 예를 들어, 체스 봇에서는 사용자가 "Move knight"라고 말할 때 "Move night"로 해석되지 않기를 원할 것입니다. LUIS 앱에는 "knight"가 엔터티로 레이블이 지정된 예제가 포함되어야 합니다.

> [!NOTE]
> 웹 채팅 채널에서 음성 초기화를 사용하려면 Bing Speech Service를 사용해야 합니다. Bing Speech Service 사용 방법에 대한 설명을 보려면 [웹 채팅 채널에서 음성 사용](~/bot-service-channel-connect-webchat-speech.md)을 참조하세요.

> [!IMPORTANT]
> 음성 초기화는 Cortana 채널 또는 웹 채팅 채널용으로 구성된 봇에만 적용됩니다.

> [!IMPORTANT]
> eu.luis.ai 및 au.luis.ai를 포함한 미국 이외 지역 LUIS 앱에서는 초기화가 지원되지 않습니다.

## <a name="change-the-list-of-luis-apps-your-bot-uses"></a>봇이 사용자는 LUIS 앱 목록 변경

봇에서 Bing Speech가 사용하는 LUIS 앱 목록을 변경하려면 다음을 수행합니다.

1. Bot Service 블레이드에서 **음성 초기화**를 클릭합니다. 사용할 수 있는 LUIS 앱 목록이 표시됩니다.
2. Bing Speech에서 사용할 LUIS 앱을 선택합니다.
 
    a. 목록에서 LUIS 앱을 선택하려면 확인란이 나타날 때까지 LUIS 모델 위로 마우스를 가져간 후 확인란을 선택합니다.
     
    b. 목록에 없는 LUIS 앱을 선택하려면 아래쪽으로 스크롤하고 텍스트 상자에 LUIS 응용 프로그램 ID GUID를 입력합니다.
     
3. **저장**을 클릭하여 봇에 대한 Bing Speech와 연결된 LUIS 앱 목록을 저장합니다.

![음성 초기화 패널](~/media/bot-service-manage-speech-priming/speech-priming.png)

## <a name="additional-resources"></a>추가 리소스

- 웹 채팅에서 음성을 사용하도록 설정하는 방법을 알아보려면 [웹 채팅 채널에서 음성 사용](~/bot-service-channel-connect-webchat-speech.md)을 참조하세요.
- 음성 초기화에 대해 자세히 알아보려면 [Bot Framework의 음성 지원 – 웹 채팅, 직접 회선, Cortana](https://blog.botframework.com/2017/06/26/Speech-To-Text/)합니다.
- LUIS 앱에 대한 자세한 내용은 [Language Understanding Intelligent Service](https://www.luis.ai)를 참조하세요.
