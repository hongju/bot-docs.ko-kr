---
title: Bot Connector 서비스와 Bot State 서비스의 주요 개념 | Microsoft Docs
description: Bot Framework의 Bot Connector 서비스와 Bot State 서비스의 주요 개념을 이해합니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a3d6bd957b835a0b8d86e47595ce28506c32d636
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224558"
---
# <a name="key-concepts"></a>주요 개념

Bot Connector 서비스와 Bot State 서비스를 사용하여 Skype, Email, Slack 등과 같은 여러 채널에서 사용자와 통신할 수 있습니다. 이 문서에서는 Bot Connector 서비스와 Bot State 서비스의 주요 개념을 소개합니다.

> [!IMPORTANT]
> Bot Framework 상태 서비스 API는 프로덕션 환경에는 권장되지 않으며 이후 릴리스에서 사용되지 않을 수 있습니다. 테스트 용도로 메모리 내 저장소를 사용하거나 프로덕션 봇용으로 **Azure 확장**을 사용하도록 봇 코드를 업데이트하는 것이 좋습니다. 자세한 내용은 [.NET](~/dotnet/bot-builder-dotnet-state.md) 또는 [Node](~/nodejs/bot-builder-nodejs-state.md) 구현에 대한 **상태 데이터 관리** 항목을 참조하세요.

## <a name="bot-connector-service"></a>Bot Connector 서비스

Bot Connector 서비스를 사용하면 봇이 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 포털</a>에 구성된 채널로 메시지를 교환할 수 있습니다. HTTPS를 통해 업계 표준 REST 및 JSON을 사용하며 JWT 전달자 토큰을 사용하여 인증할 수 있습니다. Bot Connector 서비스를 사용하는 방법에 대한 자세한 내용은 이 섹션의 [인증](bot-framework-rest-connector-authentication.md) 및 나머지 문서를 참조하세요.

### <a name="activity"></a>작업

Bot Connector 서비스는 [Activity][Activity](활동) 개체를 전달하여 봇과 채널(사용자) 사이에 정보를 교환합니다. 가장 일반적인 형식의 활동은 **메시지**이지만 다양한 형식의 정보를 봇이나 채널에 전달하는 데 사용할 수 있는 다른 활동 형식이 있습니다. Bot Connector 서비스의 활동에 대한 자세한 내용은 [활동 개요](bot-framework-rest-connector-activities.md)를 참조하세요.

## <a name="bot-state-service"></a>Bot State 서비스

Bot State 서비스를 사용하면 사용자, 대화 또는 특정 대화 컨텍스트 내의 특정 사용자와 연결된 상태 데이터를 봇이 저장하고 검색할 수 있습니다. HTTPS를 통해 업계 표준 REST 및 JSON을 사용하며 JWT 전달자 토큰을 사용하여 인증할 수 있습니다. Bot State 서비스를 사용하여 저장한 모든 데이터는 Azure에 저장되고 암호화된 상태로 유지됩니다.

Bot State 서비스는 Bot Connector 서비스와 함께 사용할 때만 유용합니다. 즉, Bot Connector 서비스를 사용하여 봇이 수행하는 대화와 관련된 상태 데이터를 저장하는 데에만 사용할 수 있습니다. Bot State 서비스를 사용하는 방법에 대한 자세한 내용은 [인증](bot-framework-rest-connector-authentication.md) 및 [상태 데이터 관리](bot-framework-rest-state.md)를 참조하세요.

## <a name="authentication"></a>인증

Bot Connector 서비스와 Bot State 서비스 모두 JWT 전달자 토큰으로 인증이 가능합니다. 봇이 Bot Framework에 보내는 아웃바운드 요청을 인증하는 방법, 봇이 Bot Framework에서 받은 인바운드 요청을 인증하는 방법 등에 대한 자세한 내용은 [인증](bot-framework-rest-connector-authentication.md)을 참조하세요. 

## <a name="client-libraries"></a>클라이언트 라이브러리

Bot Framework는 C # 또는 Node.js로 봇을 만드는 데 사용할 수 있는 클라이언트 라이브러리를 제공합니다. 

- C#을 사용하여 봇을 빌드하려면 [C#용 Bot Framework SDK](../dotnet/bot-builder-dotnet-overview.md)를 참조하세요. 
- Node.js를 사용하여 봇을 빌드하려면 [Node.js용 Bot Framework SDK](../nodejs/index.md)를 참조하세요. 

Bot Connector 서비스와 Bot State 서비스 모델링 외에도 각 Bot Framework SDK는 대화 논리를 캡슐화하는 대화를 빌드하는 강력한 시스템, 예/아니요, 문자열, 숫자 및 열거형과 같은 간단한 항목에 대한 기본 프롬프트, <a href="https://www.luis.ai/" target="_blank">LUIS</a> 등과 같은 강력한 AI 프레임워크에 대한 내장된 지원을 제공합니다. 

> [!NOTE]
> C# SDK나 Node.js SDK를 사용하는 대신 <a href="https://raw.githubusercontent.com/Microsoft/BotBuilder/master/CSharp/Library/Microsoft.Bot.Connector.Shared/Swagger/ConnectorAPI.json" target="_blank">Bot Connector Swagger 파일</a>과 <a href="https://raw.githubusercontent.com/Microsoft/BotBuilder/master/CSharp/Library/Microsoft.Bot.Connector.Shared/Swagger/StateAPI.json" target="_blank">Bot State Swagger 파일</a>을 사용하여 원하는 언어로 자신만의 클라이언트 라이브러리를 생성할 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

Bot Connector 서비스 및 Bot State 서비스를 사용하여 봇을 빌드하는 방법에 대해 자세히 알아보려면 이 섹션 전체에서 [승인](bot-framework-rest-connector-authentication.md)으로 시작하는 문서를 참조하세요. 문제가 발생하거나 Bot Connector 서비스나 Bot State 서비스에 대한 제안이 있는 경우 [지원](../bot-service-resources-links-help.md)에서 사용 가능한 리소스 목록을 참조하세요. 

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object