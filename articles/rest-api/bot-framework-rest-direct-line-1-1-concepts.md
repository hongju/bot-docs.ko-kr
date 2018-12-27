---
title: Bot Framework 직접 회선 API 1.1의 주요 개념 | Microsoft Docs
description: Bot Framework 직접 회선 API 1.1의 주요 개념을 살펴봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a049d77d506fa3fa678a079de52aa424264847c9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997050"
---
# <a name="key-concepts-in-direct-line-api-11"></a>직접 회선 API 1.1의 주요 개념

직접 회선 API를 통해 봇과 고유한 클라이언트 애플리케이션 간에 통신을 사용할 수 있습니다. 

> [!IMPORTANT]
> 이 문서에서는 직접 회선 API 1.1의 주요 개념을 소개하고 관련 개발자 리소스에 대한 정보를 제공합니다. 클라이언트 애플리케이션과 봇 간에 새 연결을 만들 경우에는 [직접 회선 API 3.0](bot-framework-rest-direct-line-3-0-concepts.md)을 대신 사용합니다.

## <a name="authentication"></a>인증

직접 회선 API 1.1 요청을 인증하려면 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 포털</a>의 직접 회선 채널 구성 페이지에서 가져오는 **비밀**을 사용하거나 런타임에 가져오는 **토큰**을 사용합니다.  자세한 내용은 [인증](bot-framework-rest-direct-line-1-1-authentication.md)을 참조하세요.

## <a name="starting-a-conversation"></a>대화 시작

직접 회선 대화는 클라이언트에서 명시적으로 열리고 봇과 클라이언트가 참여하고 유효한 자격 증명이 있는 경우에만 실행될 수 있습니다. 자세한 내용은 [대화 시작](bot-framework-rest-direct-line-1-1-start-conversation.md)을 참조하세요.

## <a name="sending-messages"></a>메시지 보내기

직접 회선 API 1.1을 사용하면 클라이언트가 `HTTP POST` 요청을 실행하여 봇에 메시지를 보낼 수 있습니다. 클라이언트는 요청당 하나의 메시지를 보낼 수 있습니다. 자세한 내용은 [봇에 메시지 보내기](bot-framework-rest-direct-line-1-1-send-message.md)를 참조하세요.

## <a name="receiving-messages"></a>메시지 수신

직접 회선 API 1.1을 사용하면 클라이언트가 `HTTP GET` 요청으로 폴링하여 메시지를 받을 수 있습니다. 각 요청에 대한 응답으로 클라이언트는 여러 봇의 메시지를 `MessageSet`의 일부로 받을 수 있습니다. 자세한 내용은 [봇에서 메시지 받기](bot-framework-rest-direct-line-1-1-receive-messages.md)를 참조하세요.

## <a name="developer-resources"></a>개발자 리소스

### <a name="client-library"></a>클라이언트 라이브러리

Bot Framework는 C#을 통해 직접 회선 API 1.1에 쉽게 액세스할 수 있는 클라이언트 라이브러리를 제공합니다. Visual Studio 프로젝트 내에서 클라이언트 라이브러리를 사용하려면 `Microsoft.Bot.Connector.DirectLine` <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine/1.1.1" target="_blank">v1.x NuGet 패키지</a>를 설치합니다. 

C# 클라이언트 라이브러리를 사용하는 대신, <a href="https://docs.botframework.com/en-us/restapi/directline/swagger.json" target="_blank">직접 회선 API 1.1 Swagger 파일</a>을 사용하여 선택한 언어로 고유한 클라이언트 라이브러리를 생성할 수 있습니다.

### <a name="web-chat-control"></a>웹 채팅 컨트롤 

Bot Framework는 클라이언트 애플리케이션에 직접 회선 기반 봇을 포함할 수 있는 컨트롤을 제공합니다. 자세한 내용은 <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework 웹 채팅 컨트롤</a>을 참조하세요.
