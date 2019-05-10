---
title: 웹 사이트에 봇 포함 | Microsoft Docs
description: 웹 사이트 내에 포함되는 봇을 디자인하는 방법을 알아봅니다.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f9fa2bee156752f1545d201768040b6106558e01
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563754"
---
# <a name="embed-a-bot-in-a-website"></a>웹 사이트에 봇 포함

봇은은 일반적으로 웹 사이트 외부에 있지만 웹 사이트 내에 포함될 수도 있습니다. 예를 들어, 웹 사이트 내에 [지식 봇](~/bot-service-design-pattern-knowledge-base.md)을 포함하여 복잡한 웹 사이트 구조 내에서 찾기 어려울 수 있는 정보를 빠르게 찾도록 할 수 있습니다. 또는 지원 센터 웹 사이트에 봇을 포함하여 들어오는 사용자 요청에 대한 첫 번째 응답자 역할을 하도록 할 수 있습니다. 봇은 간단한 문제는 독립적으로 해결하고 좀 더 복잡한 문제는 상담원에게 [전달](~/bot-service-design-pattern-handoff-human.md)할 수 있습니다. 

이 문서에서는 웹 사이트와 봇을 통합하고 *백채널* 메커니즘을 사용하여 웹 페이지 및 봇 간에 개인 통신을 용이하게 하는 프로세스를 알아봅니다. 

Microsoft는 웹 사이트에서 봇을 통합하는 두 가지 방법인 Skype 웹 컨트롤과 오픈 소스 웹 컨트롤을 제공합니다.

## <a name="skype-web-control"></a>Skype 웹 컨트롤

[Skype 웹 컨트롤](https://aka.ms/bot-skype-web-control)은 기본적으로 웹 지원 컨트롤의 Skype 클라이언트입니다. 기본 제공 Skype 인증을 통해 개발자가 사용자 지정 코드를 작성하지 않아도 봇이 사용자를 인증하고 인식할 수 있습니다. Skype는 해당 웹 클라이언트에서 사용되는 Microsoft 계정을 자동으로 인식합니다. 

Skype 웹 컨트롤은 단순히 Skype용 프런트 엔드의 역할을 하므로, 사용자의 Skype 클라이언트는 웹 컨트롤이 이용하는 모든 대화의 전체 컨텍스트에 자동으로 액세스합니다. 웹 브라우저를 닫은 후에도 사용자는 Skype 클라 이언트를 사용하여 봇과 계속 상호 작용할 수 있습니다. 

## <a name="open-source-web-control"></a>오픈 소스 웹 컨트롤

<a href="https://aka.ms/BotFramework-WebChat" target="_blank">오픈 소스 웹 채팅 컨트롤</a>은 ReactJS를 기준으로 하며 [직접 회선 API][directLineAPI]를 사용하여 Bot Framework와 통신합니다. 웹 채팅 컨트롤은 웹 채팅을 구현하기 위한 빈 캔버스를 제공하고 사용자가 해당 동작 및 제공되는 사용자 환경을 완전히 제어할 수 있도록 합니다. 

*백채널* 메커니즘을 통해 컨트롤을 호스트하는 웹 페이지는 사용자가 전혀 볼 수 없는 방식으로 봇과 직접 통신할 수 있습니다. 이 기능을 사용하면 다음과 같은 다양한 시나리오가 지원될 수 있습니다. 

- 웹 페이지에서 봇에 관련 데이터를 전송할 수 있습니다(예: GPS 위치).
- 웹 페이지에서 봇에게 사용자 작업에 대해 조언을 할 수 있습니다(예: "사용자가 방금 드롭다운에서 옵션을 선택했음).
- 웹 페이지에서 로그인한 사용자에 대한 인증 토큰을 봇에게 전송할 수 있습니다.
- 봇에서 웹 페이지로 관련 데이터를 전송할 수 있습니다(예: 사용자 포트폴리오의 현재 값).
- 봇에서 "명령"을 웹 페이지로 전송할 수 있습니다(예: 배경색 변경).

## <a name="using-the-backchannel-mechanism"></a>백채널 메커니즘 사용

[!INCLUDE [Introduction to backchannel mechanism](~/includes/snippet-backchannel.md)]

## <a name="sample-code"></a>샘플 코드

<a href="https://aka.ms/BotFramework-WebChat" target="_blank">오픈 소스 웹 채팅 컨트롤</a>은 GitHub를 통해 사용할 수 있습니다. Node.js용 오픈 소스 웹 채팅 컨트롤 및 Bot Framework SDK를 사용하여 백채널 메커니즘을 구현하는 방법에 대한 자세한 내용은 [백채널 메커니즘 사용](~/nodejs/bot-builder-nodejs-backchannel.md)을 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- [직접 회선 API][directLineAPI]
- [오픈 소스 웹 채팅 컨트롤](https://github.com/Microsoft/BotFramework-WebChat)
- [백채널 메커니즘 사용](~/nodejs/bot-builder-nodejs-backchannel.md)

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
