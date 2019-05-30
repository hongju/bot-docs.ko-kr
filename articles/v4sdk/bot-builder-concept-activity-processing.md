---
title: 작업 처리 | Microsoft Docs
description: 봇 SDK의 작업 처리를 이해합니다.
keywords: 봇 어댑터, 사용자 지정 미들웨어, 단락, 대체, 이벤트 처리기
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 69ce362f35054c5ee42035d8bffefb17c6fa7f71
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215502"
---
# <a name="activity-processing"></a>작업 처리

[!INCLUDE[applies-to](../includes/applies-to.md)]

봇과 사용자는 작업을 통해 상호 작용하고 정보를 교환합니다. 봇 애플리케이션이 받은 각 작업은 봇 어댑터로 전달되고, 봇 어댑터는 작업 정보를 봇 논리로 전달하고 궁극적으로 사용자에게 응답을 보냅니다. 작업 받기 및 이후의 봇을 통한 처리를 순서라고 하며, 봇의 전체 주기 하나를 나타냅니다. 순서는 모든 실행이 수행되고, 작업이 완전히 처리되고 봇의 모든 계층이 완료된 경우 종료합니다.

작업, 특히 봇 턴 동안 [봇에서 보낸](#generating-responses) 작업은 비동기적으로 처리됩니다. 봇 빌드에서 필수적인 부분입니다. 작동 원리를 다시 살펴보려면 선택한 언어에 따라 [.NET용 비동기](https://docs.microsoft.com/en-us/dotnet/csharp/async) 또는 [JavaScript용 비동기](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)를 확인하세요.

## <a name="the-bot-adapter"></a>봇 어댑터

봇 어댑터는 인증 프로세스를 캡슐화하고 Bot Connector Service와 작업을 주고받습니다. 봇이 작업을 받으면 어댑터는 해당 작업에 대한 모든 항목을 래핑하고, 턴에 대한 [컨텍스트 개체](#turn-context)를 만들어서 봇의 애플리케이션 논리에 전달하고, 봇이 생성한 응답을 다시 사용자의 채널로 보냅니다.

## <a name="authentication"></a>인증

어댑터는 작업의 정보 및 REST 요청의 `Authentication` 헤더를 사용하여 애플리케이션에서 수신하는 각 들어오는 작업을 인증합니다. 어댑터는 커넥터 개체 및 애플리케이션의 자격 증명을 사용하여 사용자에 대한 아웃바운드 작업을 인증합니다.

Bot Connector Service 인증에는 JWT(JSON 웹 토큰) `Bearer` 토큰 및 **Microsoft 앱 ID**, 그리고 봇 서비스를 만들 때 또는 봇을 등록할 때 Azure가 자동으로 만든 **Microsoft 앱 암호**가 사용됩니다. 어댑터가 트래픽을 인증하려면 초기화 시 애플리케이션에서 이러한 자격 증명이 필요합니다.

> [!NOTE]
> 예를 들어 Bot Framework 에뮬레이터를 사용하여 봇을 로컬로 실행하거나 테스트하는 경우 봇으로 들어가고 나오는 트래픽을 인증하도록 어댑터를 구성할 필요가 없습니다.

## <a name="turn-context"></a>턴 컨텍스트

어댑터는 작업을 받으면 **턴 컨텍스트** 개체를 생성합니다. 이 개체는 들어오는 작업, 보낸 사람 및 받는 사람, 채널, 대화, 작업을 처리하는 데 필요한 기타 데이터에 대한 정보를 제공합니다. 그러면 어댑터는 이 컨텍스트 개체를 봇에 전달합니다. 컨텍스트 개체는 턴 시간 동안 유지되며, 다음에 대한 정보를 제공합니다.

* 대화 - 대화를 식별하고 대화에 참여한 봇과 사용자에 대한 정보를 포함합니다.
* 작업 - 대화의 요청과 회신은 모두 작업 형식입니다. 이 컨텍스트는 라우팅 정보, 채널 정보, 대화, 보낸 사람 및 받는 사람에 대한 정보를 포함하여 들어오는 작업에 대한 정보를 제공합니다.
* 사용자 지정 정보 - 미들웨어를 구현하여 또는 봇 논리 내에서 봇을 확장하는 경우 각 턴에서 추가 정보를 제공할 수 있습니다.

또한 컨텍스트 개체는 사용자에게 응답을 보내고, 어댑터에 대한 참조를 가져오는 데 사용될 수 있습니다.<!-- to create a new conversation or continue an existing one-->.

> [!NOTE]
> 애플리케이션 및 어댑터는 요청을 비동기적으로 처리하지만, 비즈니스 논리가 요청-응답 기반일 필요는 없습니다.

## <a name="middleware"></a>미들웨어

어댑터에 미들웨어를 추가할 수 있습니다. 미들웨어 및 봇 논리는 컨텍스트 개체를 사용하여 작업에 대한 정보를 검색하고 그에 따라 작동합니다. 미들웨어 및 봇은 턴, 대화 또는 기타 범위에 대한 추적 상태와 같은 정보를 업데이트하거나 컨텍스트 개체에 추가할 수도 있습니다. 미들웨어에 대한 자세한 내용은 [미들웨어 문서](~/v4sdk/bot-builder-concept-middleware.md)를 참조하세요.

## <a name="generating-responses"></a>응답 생성

컨텍스트 개체는 코드가 작업에 응답하도록 허용하는 작업 응답 메서드를 제공합니다.

* _send activity_ 및 _send activities_ 메서드는 하나 이상의 작업을 대화에 보냅니다.
* 채널에서 지원되는 경우 _update activity_ 메서드는 대화 내에서 작업을 업데이트합니다.
* 채널에서 지원되는 경우 _delete activity_ 메서드는 대화에서 작업을 제거합니다.

각 응답 메서드는 비동기 프로세스에서 실행됩니다. 작업 응답 메서드는 호출되면 처리기를 호출하기 전까지 연결된 [이벤트 처리기](#response-event-handlers) 목록을 복제합니다. 즉, 이 시점까지 추가된 모든 처리기를 포함하지만, 프로세스가 시작된 후에 추가된 항목을 전혀 포함하지 않습니다.

이는 응답의 순서가 보장되지 않는다는 뜻이며, 한 작업이 다른 작업보다 복잡한 경우에 더욱 그렇습니다. 봇이 들어오는 작업에 대한 여러 응답을 생성할 수 있는 경우 사용자가 받는 순서대로 의미가 통해야 합니다.

> [!IMPORTANT]
> 기본 봇 턴을 처리하는 스레드는 사용을 마친 컨텍스트 개체를 폐기합니다. 응답(처리기 포함)이 상당한 시간을 소요하고 컨텍스트 개체에 따라 작동하는 경우 `Context was disposed` 오류가 발생할 수 있습니다. 턴 컨텍스트의 처리 및 폐기가 완료될 때까지 기본 스레드가 생성된 작업을 기다리도록 **작업 호출을 `await`해야 합니다.**

## <a name="response-event-handlers"></a>응답 이벤트 처리기

봇 및 미들웨어 논리 외에도, 컨텍스트 개체에 응답 처리기(이벤트 처리기 또는 작업 이벤트 처리기라고도 함)를 추가할 수 있습니다. 이러한 처리기는 현재 컨텍스트 개체에서 관련 [응답](#generating-responses)이 발생하면 실제 응답을 실행하기 전에 호출됩니다. 이러한 처리기는 실제 이벤트 전에 또는 후에 현재 응답의 나머지 부분에서 해당 형식의 모든 작업에 대해 무엇을 할 것인지 알고 있는 경우에 유용 합니다.

> [!WARNING]
> 각 응답 이벤트 처리기 내부에서 작업 응답 메서드를 호출하지 않도록 주의해야 합니다. 예를 들어 _on send activity_ 처리기 내에서 send activity 메서드를 호출하면 안 됩니다. 호출하면 무한 루프가 생성될 수 있습니다.

새 작업마다 실행할 새 스레드가 생깁니다. 작업을 처리하는 스레드가 생성되면 해당 작업에 대한 처리기 목록이 해당하는 새 스레드에 복사됩니다. 해당 시점 이후 추가된 처리기는 해당 활동 이벤트에 대해 실행되지 않습니다.

컨텍스트 개체에 등록된 처리기는 어댑터에서 [미들웨어 파이프라인](~/v4sdk/bot-builder-concept-middleware.md#the-bot-middleware-pipeline)을 관리하는 방법과 매우 비슷하게 처리됩니다. 즉, 처리기는 추가된 순서대로 호출되며, _다음_ 대리자를 호출하면 등록된 그 다음 이벤트 처리기에 컨트롤이 전달됩니다. 처리기가 다음 대리자를 호출하지 않으면 후속 이벤트 처리기는 호출되지 않으며([단락](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting) 이벤트), 어댑터는 채널에 응답을 보내지 않습니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [미들웨어](~/v4sdk/bot-builder-concept-middleware.md)
