---
title: Node.js용 Bot Builder SDK | Microsoft Docs
description: 강력하고 사용하기 쉬운 봇 빌드 프레임워크인 Node.js용 Bot Builder SDK를 살펴봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 41c276186f08e9997497e5649a6566954f0c8079
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304570"
---
# <a name="bot-builder-sdk-for-nodejs"></a>Node.js용 Bot Builder SDK

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Node.js용 Bot Builder SDK는 Node.js 개발자가 봇을 작성할 수 있는 친숙한 방법을 제공하는 강력하고 사용하기 쉬운 프레임워크입니다.
이를 사용하여 간단한 프롬프트에서 자유 형식 대화까지 다양한 대화형 사용자 인터페이스를 빌드할 수 있습니다.

봇에 대한 대화형 논리는 웹 서비스로 호스트됩니다. Bot Builder SDK는 웹 서버를 빌드하기 위한 인기 있는 프레임워크인 <a href="http://restify.com">restify</a>를 사용하여 봇의 웹 서버를 만듭니다. SDK는 또한 <a href="http://expressjs.com/">Express</a>와 호환 가능하며 일부 조정으로 다른 웹앱 프레임워크를 사용할 수 있습니다. 

SDK를 사용하여 다음 SDK 기능을 활용할 수 있습니다. 

- 대화형 논리를 캡슐화하는 대화 상자를 빌드하기 위한 강력한 시스템
- 예/아니요, 문자열, 숫자 및 열거형과 같은 간단한 것에 대한 기본 제공 프롬프트뿐만 아니라 이미지 및 첨부 파일을 포함하는 메시지와 단추를 포함하는 다양한 카드에 대한 지원
- <a href="http://luis.ai" target="_blank">LUIS</a>와 같은 강력한 AI 프레임워크에 대한 기본 제공 지원
- 필요에 따라 도움말, 탐색, 설명 및 확인을 제공하여 대화를 통해 사용자를 안내하는 기본 제공 인식기 및 이벤트 처리기

## <a name="get-started"></a>시작하기

봇을 처음 작성하는 경우 프로젝트를 설정하고, SDK를 설치하고, 첫 번째 봇을 실행하는 데 도움을 주는 단계별 지침을 사용하여 [Node.js로 첫 번째 봇을 만듭니다](bot-builder-nodejs-quickstart.md). 

Node.js용 Bot Builder SDK를 처음 접하는 경우 Bot Builder SDK의 주요 구성 요소를 이해할 수 있도록 돕는 주요 개념을 시작할 수 있습니다. [주요 개념](bot-builder-nodejs-concepts.md)을 참조하세요.

봇에서 상위 사용자 시나리오를 해결하는지 확인하려면 지침에 대해 [디자인 원칙](../bot-service-design-principles.md) 및 [패턴 살펴보기](../bot-service-design-pattern-task-automation.md)를 검토하세요.

## <a name="get-samples"></a>샘플 받기

[Node.js용 Bot Builder SDK 샘플](bot-builder-nodejs-samples.md)은 Node.js용 Bot Builder SDK의 기능을 활용하는 방법을 보여주는 작업에 초점을 맞춘 봇을 설명합니다. 샘플을 사용하여 다양한 기능이 포함된 유용한 봇 빌드를 빠르게 시작할 수 있습니다.

## <a name="next-steps"></a>다음 단계
> [!div class="nextstepaction"]
> [주요 개념](bot-builder-nodejs-concepts.md)

## <a name="additional-resources"></a>추가 리소스

다음 작업에 초점을 맞춘 방법 가이드는 Node.js용 Bot Builder SDK의 다양한 기능을 보여줍니다.

* [메시지에 응답](bot-builder-nodejs-use-default-message-handler.md)
* [사용자 작업 처리](bot-builder-nodejs-dialog-actions.md)
* [사용자 의도 인식](bot-builder-nodejs-recognize-intent-messages.md)
* [다양한 카드 보내기](bot-builder-nodejs-send-rich-cards.md)
* [첨부 파일 보내기](bot-builder-nodejs-send-receive-attachments.md)
* [사용자 데이터 저장](bot-builder-nodejs-save-user-data.md)


문제가 발생하거나 Node.js용 Bot Builder SDK에 대한 제안이 있는 경우 사용 가능한 리소스의 목록은 [지원](../bot-service-resources-links-help.md)을 참조하세요. 


[DesignGuide]: ../bot-service-design-principles.md 
[DesignPatterns]: ../bot-service-design-pattern-task-automation.md 
[HowTo]: bot-builder-nodejs-use-default-message-handler.md 
