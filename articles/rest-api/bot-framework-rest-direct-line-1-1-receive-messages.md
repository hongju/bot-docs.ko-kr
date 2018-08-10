---
title: 봇에서 메시지 받기 | Microsoft Docs
description: 직접 회선 API v1.1을 사용하여 봇에서 메시지를 받는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: d9f1821767e5bd26c9a8bfdf3927f257077f0e79
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303331"
---
# <a name="receive-messages-from-the-bot"></a>봇에서 메시지 받기

> [!IMPORTANT]
> 이 문서에서는 직접 회선 API v1.1을 사용하여 봇에서 메시지를 받는 방법을 설명합니다. 클라이언트 응용 프로그램과 봇 간에 새 연결을 만들 경우에는 [직접 회선 API 3.0](bot-framework-rest-direct-line-3-0-receive-activities.md)을 대신 사용합니다.

직접 회선 1.1 프로토콜을 사용할 경우 메시지를 받으려면 클라이언트에서 `HTTP GET` 인터페이스를 폴링해야 합니다. 

## <a name="retrieve-messages-with-http-get"></a>HTTP GET을 사용하여 메시지 검색

특정 대화에 대한 메시지를 검색하려면 `api/conversations/{conversationId}/messages` 엔드포인트에 `GET` 요청을 실행하고, 선택적으로 `watermark` 매개 변수를 지정하여 클라이언트가 표시하는 가장 최근 메시지를 나타냅니다. 메시지가 포함되지 않은 경우에도 업데이트된 `watermark` 값이 JSON 응답으로 반환됩니다.

다음 코드 조각은 메시지 가져오기 요청 및 응답의 예제를 제공합니다. 메시지 가져오기 응답에는 `watermark`가 [MessageSet](bot-framework-rest-direct-line-1-1-api-reference.md#messageset-object)의 속성으로 포함됩니다. 클라이언트는 메시지가 반환되지 않을 때까지 `watermark` 값을 전달하여 사용 가능한 메시지를 페이징해야 합니다. 

### <a name="request"></a>요청

```http
GET https://directline.botframework.com/api/conversations/abc123/messages?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>response

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "messages": [
        {
            "conversation": "abc123",
            "id": "abc123|0000",
            "text": "hello",
            "from": "user1"
        }, 
        {
            "conversation": "abc123",
            "id": "abc123|0001",
            "text": "Nice to see you, user1!",
            "from": "bot1"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>타이밍 고려 사항

직접 회선은 잠재적인 타이밍 격차가 있는 다중 파트 프로토콜이지만, 프로토콜 및 서비스는 신뢰할 수 있는 클라이언트를 빌드할 수 있도록 디자인되어 있습니다. 메시지 가져오기 응답으로 전송되는 `watermark` 속성은 신뢰할 수 있습니다. 클라이언트는 워터마크 축자를 재생하는 경우 메시지를 놓치지 않습니다.

클라이언트는 용도와 일치하는 폴링 간격을 선택해야 합니다.

- 서비스 간 응용 프로그램은 종종 5초 또는 10초의 폴링 간격을 사용합니다.

- 클라이언트 쪽 응용 프로그램은 종종 1초의 폴링 간격을 사용하고, 클라이언트가 보내는(봇의 응답을 빠르게 검색하기 위해) 모든 메시지 이후 300ms까지 추가 요청을 실행합니다. 이 300ms 지연은 봇의 속도 및 전송 시간에 따라 조정해야 합니다.

## <a name="additional-resources"></a>추가 리소스

- [주요 개념](bot-framework-rest-direct-line-1-1-concepts.md)
- [인증](bot-framework-rest-direct-line-1-1-authentication.md)
- [대화 시작](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [봇에 메시지 보내기](bot-framework-rest-direct-line-1-1-send-message.md)