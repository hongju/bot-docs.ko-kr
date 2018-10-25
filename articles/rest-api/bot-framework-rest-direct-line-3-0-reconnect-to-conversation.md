---
title: 대화에 다시 연결 | Microsoft Docs
description: 직접 회선 API v3.0을 사용하여 대화에 다시 연결하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 812ad04232629e889b0ab5472af07ff986ca787b
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000310"
---
# <a name="reconnect-to-a-conversation"></a>대화에 다시 연결

클라이언트는 [WebSocket 인터페이스](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)를 사용하여 메시지를 수신하는 경우, 연결이 끊어지면 다시 연결해야 할 수 있습니다. 이 시나리오에서 클라이언트는 대화에 다시 연결하는 데 사용할 수 있는 새 WebSocket 스트림 URL을 생성해야 합니다.

## <a name="generate-a-new-websocket-stream-url"></a>새 WebSocket 스트림 URL 생성

기존 대화에 다시 연결하는 데 사용할 수 있는 새 WebSocket 스트림 URL을 생성하려면 다음 요청을 실행합니다. 

```http
GET https://directline.botframework.com/v3/directline/conversations/{conversationId}?watermark={watermark_value}
Authorization: Bearer SECRET_OR_TOKEN
```

이 요청 URI에서 **{conversationId}** 를 대화 ID로 바꾸고 **{watermark_value}** 를 워터마크 값으로 바꿉니다(`watermark` 매개 변수가 제공될 경우). `watermark` 매개 변수는 선택 사항입니다. 요청 URI에 `watermark` 매개 변수가 지정된 경우 워터마크부터 대화가 재생되므로 메시지가 손실되지 않도록 보장됩니다. 요청 URI에서 `watermark` 매개 변수가 생략되면 연결 요청 이후에 수신된 메시지만 재생됩니다.

다음 코드 조각은 다시 연결 요청 및 응답의 예제를 제공합니다.

### <a name="request"></a>요청

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123?watermark=0000a-42
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>response

요청이 성공적이면 응답에는 대화의 ID, 토큰 및 새 WebSocket 스트림 URL가 포함됩니다.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?watermark=000a-4&amp;t=RCurR_XV9ZA.cwA..."
}
```

## <a name="reconnect-to-the-conversation"></a>대화에 다시 연결

클라이언트는 새 WebSocket 스트림 URL을 사용하여 60초 이내에 [대화에 다시 연결](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)해야 합니다. 이 시간 동안 연결을 설정할 수 없는 경우, 클라이언트는 다른 다시 연결 요청을 실행하여 새 스트림 URL을 생성해야 합니다.

## <a name="additional-resources"></a>추가 리소스

- [주요 개념](bot-framework-rest-direct-line-3-0-concepts.md)
- [인증](bot-framework-rest-direct-line-3-0-authentication.md)
- [WebSocket 스트림을 통해 활동 수신](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)