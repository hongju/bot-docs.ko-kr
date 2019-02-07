---
title: Bot Framework 직접 회선 API 3.0의 주요 개념 | Microsoft Docs
description: Bot Framework 직접 회선 API 3.0의 주요 개념을 살펴봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/06/2019
ms.openlocfilehash: 94ef9cc221e67f4f3762eb7a1a006a915e3c5307
ms.sourcegitcommit: fd60ad0ff51b92fa6495b016e136eaf333413512
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/06/2019
ms.locfileid: "55764101"
---
# <a name="key-concepts-in-direct-line-api-30"></a>직접 회선 API 3.0의 주요 개념

직접 회선 API를 통해 봇과 고유한 클라이언트 애플리케이션 간에 통신을 사용할 수 있습니다. 이 문서에서는 직접 회선 API 3.0의 주요 개념을 소개하고 관련 개발자 리소스에 대한 정보를 제공합니다.

## <a name="authentication"></a>인증

직접 회선 API 3.0 요청을 인증하려면 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 포털</a>의 직접 회선 채널 구성 페이지에서 가져오는 **비밀**을 사용하거나 런타임에 가져오는 **토큰**을 사용합니다. 자세한 내용은 [인증](bot-framework-rest-direct-line-3-0-authentication.md)을 참조하세요.

## <a name="starting-a-conversation"></a>대화 시작

직접 회선 대화는 클라이언트에서 명시적으로 열리고 봇과 클라이언트가 참여하고 유효한 자격 증명이 있는 경우에만 실행될 수 있습니다. 자세한 내용은 [대화 시작](bot-framework-rest-direct-line-3-0-start-conversation.md)을 참조하세요.

## <a name="sending-messages"></a>메시지 보내기

직접 회선 API 3.0을 사용하면 클라이언트가 `HTTP POST` 요청을 실행하여 봇에 메시지를 보낼 수 있습니다. 클라이언트는 요청당 하나의 메시지를 보낼 수 있습니다. 자세한 내용은 [봇에 작업 보내기](bot-framework-rest-direct-line-3-0-send-activity.md)를 참조하세요.

## <a name="receiving-messages"></a>메시지 수신

직접 회선 API 3.0을 사용하면 클라이언트가 `WebSocket` 스트림을 통하거나 `HTTP GET` 요청을 발행하여 봇으로부터 메시지를 수신할 수 있습니다. 이러한 기술 중 하나를 사용하면 클라이언트는 `ActivitySet`의 일환으로 봇으로부터 한 번에 여러 메시지를 받을 수 있습니다. 자세한 내용은 [봇에서 작업 수신](bot-framework-rest-direct-line-3-0-receive-activities.md)을 참조하세요.

## <a name="developer-resources"></a>개발자 리소스

### <a name="client-libraries"></a>클라이언트 라이브러리

Bot Framework는 C# 및 Node.js를 통해 직접 회선 API 3.0에 쉽게 액세스할 수 있는 클라이언트 라이브러리를 제공합니다. 

- Visual Studio 프로젝트 내에서 .NET 클라이언트 라이브러리를 사용하려면 `Microsoft.Bot.Connector.DirectLine` <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine" target="_blank">NuGet 패키지</a>를 설치합니다. 

- Node.js 클라이언트 라이브러리를 사용하려면 <a href="https://www.npmjs.com/package/botframework-directlinejs" target="_blank">NPM</a>을 사용(또는 소스 <a href="https://github.com/Microsoft/BotFramework-DirectLineJS" target="_blank">다운로드</a>)하여 `botframework-directlinejs` 라이브러리를 설치합니다.

::: moniker range="azure-bot-service-3.0"

### <a name="sample-code"></a>샘플 코드

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples" target="_blank">BotBuilder 샘플</a> GitHub 리포지토리에는 C# 및 Node.js를 사용하여 직접 회선 API 3.0을 사용하는 방법을 보여 주는 여러 샘플이 포함되어 있습니다.

| 샘플 | 언어 | 설명 |
|----|----|----|
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLine" target="_blank">직접 회선 봇 샘플</a> | C# | 직접 회선 API를 사용하여 서로 통신하는 샘플 봇과 사용자 지정 클라이언트. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLineWebSockets" target="_blank">직접 회선 봇 샘플(클라이언트 Websockets 사용)</a> | C# | 직접 회선 API 및 WebSockets를 사용하여 서로 통신하는 샘플 봇과 사용자 지정 클라이언트. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLine" target="_blank">직접 회선 봇 샘플</a> | JavaScript | 직접 회선 API를 사용하여 서로 통신하는 샘플 봇과 사용자 지정 클라이언트. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLineWebSockets" target="_blank">직접 회선 봇 샘플(클라이언트 Websockets 사용)</a> | JavaScript | 직접 회선 API 및 WebSockets를 사용하여 서로 통신하는 샘플 봇과 사용자 지정 클라이언트. |

::: moniker-end

### <a name="web-chat-control"></a>웹 채팅 컨트롤 

Bot Framework는 클라이언트 애플리케이션에 직접 회선 기반 봇을 포함할 수 있는 컨트롤을 제공합니다. 자세한 내용은 <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework 웹 채팅 컨트롤</a>을 참조하세요.
