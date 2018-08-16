---
title: Bot Builder REST API | Microsoft Docs
description: 봇 및 봇에 연결되는 클라이언트를 빌드하는 사용할 수 있는 Bot Framework REST API를 시작합니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: d6d83edb390933cb8895b26efeb9775cafdb1acb
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302147"
---
# <a name="bot-framework-rest-apis"></a>Bot Framework REST API
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Bot Framework REST API에서는 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 포털</a>에 구성된 채널과 메시지를 교환하는 봇을 빌드하고, 상태 데이터를 저장 및 검색하고, 자체 클라이언트 응용 프로그램을 봇에 연결할 수 있습니다. 모든 Bot Framework 서비스는 HTTPS를 통해 업계 표준 REST와 JSON을 사용합니다.

## <a name="build-a-bot"></a>봇 빌드

Bot Connector 서비스를 사용하여 Bot Framework 포털에 구성된 채널과 메시지를 교환함으로써 어떤 프로그래밍 언어로도 봇을 만들 수 있습니다. 

> [!TIP]
> Bot Framework는 C # 또는 Node.js로 봇을 만드는 데 사용할 수 있는 클라이언트 라이브러리를 제공합니다. C#을 사용하여 봇을 작성하려면 [C#용 Bot Builder SDK](../dotnet/bot-builder-dotnet-overview.md)를 참조하세요. Node.js를 사용하여 봇을 작성하려면 [Node.js용 Bot Builder SDK](../nodejs/index.md)를 참조하세요. 

Bot Connector 서비스를 사용하여 봇을 빌드하는 방법에 대한 자세한 내용은 [핵심 개념](bot-framework-rest-connector-concepts.md)을 참조하세요.

## <a name="build-a-client"></a>클라이언트 빌드

직접 회선 API를 사용하여 클라이언트 응용 프로그램이 봇과 통신하도록 할 수 있습니다. 직접 회선 API는 봇이 해당 프로토콜 버전을 변경하더라도, 표준 암호/토큰 패턴을 사용하고 안정적인 스키마를 제공하는 인증 메커니즘을 구현합니다. 직접 회선 API를 사용하여 클라이언트와 봇 간의 통신을 가능하게 하는 방법에 대한 자세한 내용은 [핵심 개념](bot-framework-rest-direct-line-3-0-concepts.md)을 참조하세요. 