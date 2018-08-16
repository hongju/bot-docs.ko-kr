---
title: 채널 및 Bot Connector Service | Microsoft Docs
description: Bot Builder SDK 핵심 개념입니다.
keywords: 작업, 대화
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/28/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: abb2b18d45fb722d7de98c5cd02bd88350aad803
ms.sourcegitcommit: 9a38d76afb0e82fdccc1f36f9b1a65042671e538
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/04/2018
ms.locfileid: "39515093"
---
# <a name="channels-and-the-bot-connector-service"></a>채널 및 Bot Connector Service

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

채널은 Facebook, Skype, 이메일, Slack 등 사용자가 봇에 연결되어 있는 엔드포인트입니다. [Azure Portal](https://portal.azure.com)을 통해 구성된 Bot Connector Service는 봇을 이러한 채널에 연결하고, 봇과 사용자 간 통신을 용이하게 합니다. 

또한 Bot Connector Service를 통해 제공되는 표준 채널 이외에도 [직접 회선](bot-builder-howto-direct-line.md)을 채널로 사용하여 자체 클라이언트 응용 프로그램에 봇을 연결할 수 있습니다.

Bot Connector Service를 사용하면 봇이 채널에 전송하는 메시지를 정규화하여 채널 불가지론적 방법으로 봇을 개발할 수 있습니다. 이는 봇 작성기 스키마에서 채널의 스키마로 변환하는 것을 포함합니다. 단, 채널이 봇 작성기 스키마 중 일부를 지원하지 않는 경우 서비스는 메시지를 채널이 지원하는 형식으로 변환하려고 시도합니다. 예를 들어, 봇이 작업 단추가 있는 카드가 포함된 메시지를 SMS 채널에 보내면 커넥터는 카드를 이미지로 전송하고 메시지의 텍스트에 링크로 작업을 포함시킬 수 있습니다.

## <a name="activities-and-conversations"></a>작업 및 대화


Bot Connector Service는 JSON을 사용하여 봇과 사용자 간에 정보를 교환하고, Bot Builder SDK는 언어별 작업 개체에서 이 정보를 래핑합니다. [봇과의 상호 작용](bot-builder-basics.md#interaction-with-your-bot)을 논의할 때 가장 일반적인 작업 형식은 메시지이지만 다른 작업 형식도 똑같이 중요하다는 내용으로 [작업 형식](../bot-service-activities-entities.md)을 설명했습니다. 여기에는 대화 업데이트, 연락처 관계 업데이트, 사용자 데이터 삭제, 대화 종료, 입력, 메시지 대응 및 사용자가 거의 볼 가능성이 없는 다른 봇 특정 작업이 포함됩니다. 각 작업 형식에 대한 세부 사항 및 발생하는 경우는 작업 참조 콘텐츠에서 확인할 수 있습니다.

모든 턴 및 연결된 작업은 하나 이상의 봇과 특정 사용자 또는 사용자 그룹 간의 상호 작용을 나타내는 논리적 **대화**에 속합니다. 대화는 채널에 따라 다르며 해당 채널에 고유한 ID를 가집니다.

## <a name="next-steps"></a>다음 단계

이제 봇의 몇 가지 주요 개념에 익숙해졌으므로 봇이 사용할 수 있는 대화의 다른 양식에 대해 살펴보겠습니다.

> [!div class="nextstepaction"]
> [대화 양식](bot-builder-conversations.md)
