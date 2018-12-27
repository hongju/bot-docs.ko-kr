---
title: 봇에서 활동 수신 | Microsoft Docs
description: 직접 회선 API v3.0을 사용하여 봇에서 활동을 수신하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: c2d4b9a8e2b8ffc1656df44e04ee1bde912e36ea
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998160"
---
# <a name="receive-activities-from-the-bot"></a>봇에서 활동 수신

직접 회선 3.0 프로토콜을 사용하면 클라이언트는 `WebSocket` 스트림을 통해 활동을 수신하거나 `HTTP GET` 요청을 실행하여 활동을 검색할 수 있습니다. 

## <a name="websocket-vs-http-get"></a>WebSocket 대 HTTP GET

스트리밍 WebSocket은 메시지를 클라이언트에 효율적으로 푸시하지만, GET 인터페이스를 사용하면 클라이언트가 메시지를 명시적으로 요청할 수 있습니다. WebSocket 메커니즘은 효율성으로 인해 종종 선호되지만, GET 메커니즘은 WebSocket을 사용할 수 없는 클라이언트 및 대화 기록을 검색하려는 클라이언트에 유용할 수 있습니다. 

모든 [활동 형식](bot-framework-rest-connector-activities.md)이 WebSocket 및 HTTP GET을 둘 다 사용하여 제공되는 것은 아닙니다. 다음 표에서는 직접 회선 프로토콜을 사용하는 클라이언트에 대한 다양한 활동 형식의 가용성을 설명합니다.

| 활동 유형 | 가용성 | 
|----|----|
| Message | HTTP GET 및 WebSocket |
| 입력 | WebSocket만 |
| conversationUpdate | 클라이언트를 통해 전송/수신되지 않음 |
| contactRelationUpdate | 직접 회선에서 지원되지 않음 |
| endOfConversation | HTTP GET 및 WebSocket |
| 모든 기타 활동 형식 | HTTP GET 및 WebSocket |

## <a id="connect-via-websocket"></a> WebSocket 스트림을 통해 활동 수신

클라이언트가 봇과 대화를 열기 위해 [대화 시작](bot-framework-rest-direct-line-3-0-start-conversation.md) 요청을 보내면 서비스 응답에는 클라이언트가 이후에 WebSocket을 통해 연결하는 데 사용할 수 있는 `streamUrl` 속성이 포함됩니다. 스트림 URL은 권한이 미리 부여되어 있으므로 WebSocket을 통해 연결하기 위한 클라이언트의 요청에 `Authorization` 헤더가 필요하지 않습니다.

다음 예제에서는 `streamUrl`을 사용하여 WebSocket을 통해 연결하는 요청을 보여 줍니다.

```http
-- connect to wss://directline.botframework.com --
GET /v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
Upgrade: websocket
Connection: upgrade
[other headers]
```

서비스에서 상태 코드 HTTP 101(“프로토콜 전환”)로 응답합니다.

```http
HTTP/1.1 101 Switching Protocols
[other headers]
```

### <a name="receive-messages"></a>메시지 받기

WebSocket을 통해 연결한 후 클라이언트는 직접 회선 서비스에서 다음 형식의 메시지를 받을 수 있습니다.

- 하나 이상의 활동 및 워터마크(아래 설명됨)를 포함하는 [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object)이 있는 메시지.
- 연결이 계속 유효한지 확인하기 위해 직접 회선 서비스에서 사용하는 빈 메시지.
- 나중에 정의되는 추가 형식. 이러한 형식은 JSON 루트의 속성으로 식별됩니다.

`ActivitySet`에는 봇 및 대화의 모든 사용자가 보내는 메시지가 포함됩니다. 다음 예제에서는 단일 메시지가 포함된 `ActivitySet`을 보여 줍니다.

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }
    ],
    "watermark": "0000a-42"
}
```

### <a name="process-messages"></a>메시지 처리

클라이언트는 각 [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object)으로 받는 `watermark` 값을 추적해야 합니다. 이렇게 해야 워터마크를 사용하여 연결이 끊어져서 [대화에 다시 연결](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)해야 할 경우 손실되는 메시지가 없도록 보장할 수 있습니다. 클라이언트가 `ActivitySet`을 받을 때 여기에 포함된 `watermark` 속성이 `null`이거나 없는 경우에는 값을 무시하고 클라이언트가 받은 이전 워터마크를 덮어쓰지 않아야 합니다.

클라이언트는 직접 회선 서비스에서 받는 빈 메시지를 무시해야 합니다.

클라이언트는 직접 회선 서비스에 빈 메시지를 보내 연결을 확인할 수 있습니다. 직접 회선 서비스는 클라이언트에서 받은 빈 메시지를 무시합니다.

직접 회선 서비스는 특정 조건에서 WebSocket 연결을 강제로 끊을 수 있습니다. 클라이언트가 `endOfConversation` 활동을 받지 않으면 [새 WebSocket 스트림 URL을 생성](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)하여 대화에 다시 연결하는 데 사용할 수 있습니다. 

WebSocket 스트림에는 라이브 업데이트와 최근 메시지(WebSocket을 통한 연결 호출이 실행된 이후)가 포함되지만 가장 최근 `POST` 전에 `/v3/directline/conversations/{id}`에 전송된 메시지는 포함되지 않습니다. 대화 이전에 전송된 메시지를 검색하려면 아래 설명대로 `HTTP GET`을 사용합니다.

## <a id="http-get"></a> HTTP GET을 사용하여 활동 검색

WebSocket을 사용할 수 없는 클라이언트나 대화 기록을 가져오려는 클라이언트는 `HTTP GET`을 사용하여 활동을 검색할 수 있습니다.

특정 대화에 대한 메시지를 검색하려면 `/v3/directline/conversations/{conversationId}/activities` 엔드포인트에 `GET` 요청을 실행하고, 선택적으로 `watermark` 매개 변수를 지정하여 클라이언트가 표시하는 가장 최근 메시지를 나타냅니다. 

다음 코드 조각은 대화 활동 가져오기 요청 및 응답의 예제를 제공합니다. 대화 활동 가져오기 응답에는 `watermark`가 [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object)의 속성으로 포함됩니다. 클라이언트는 활동이 반환되지 않을 때까지 `watermark` 값을 전달하여 사용 가능한 활동을 페이징해야 합니다.

### <a name="request"></a>요청

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123/activities?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>response

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }, 
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0001",
            "from": {
                "id": "bot1"
            },
            "text": "Nice to see you, user1!"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>타이밍 고려 사항

대부분 클라이언트는 전체 메시지 기록을 보존하려고 합니다. 직접 회선은 잠재적인 타이밍 격차가 있는 다중 파트 프로토콜이지만, 프로토콜 및 서비스는 신뢰할 수 있는 클라이언트를 빌드할 수 있도록 디자인되어 있습니다.

- WebSocket 스트림 및 대화 활동 가져오기 응답으로 전송되는 `watermark` 속성은 신뢰할 수 있습니다. 클라이언트는 워터마크 축자를 재생하는 경우 메시지를 놓치지 않습니다.

- 클라이언트가 대화를 시작하고 WebSocket 스트림에 연결하면 POST 이후, 소켓이 열리기 전에 전송되는 모든 활동은 새 활동 전에 재생됩니다.

- 클라이언트가 WebSocket 스트림에 연결된 동안 대화 활동 가져오기 요청을 실행하면(기록을 새로 고치기 위해) 두 개의 채널에서 활동이 중복될 수 있습니다. 클라이언트는 중복 활동이 발생할 경우 이를 거부할 수 있도록 모든 알려진 활동 ID를 추적해야 합니다.

`HTTP GET`을 사용하여 폴링하는 클라이언트는 용도와 일치하는 폴링 간격을 선택해야 합니다.

- 서비스 간 애플리케이션은 종종 5초 또는 10초의 폴링 간격을 사용합니다.

- 클라이언트 쪽 애플리케이션은 종종 1초의 폴링 간격을 사용하고, 클라이언트가 보내는(봇의 응답을 빠르게 검색하기 위해) 모든 메시지 이후 300ms까지 추가 요청을 실행합니다. 이 300ms 지연은 봇의 속도 및 전송 시간에 따라 조정해야 합니다.

## <a name="additional-resources"></a>추가 리소스

- [주요 개념](bot-framework-rest-direct-line-3-0-concepts.md)
- [인증](bot-framework-rest-direct-line-3-0-authentication.md)
- [대화 시작](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [대화에 다시 연결](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [봇에 활동 보내기](bot-framework-rest-direct-line-3-0-send-activity.md)
- [대화 종료](bot-framework-rest-direct-line-3-0-end-conversation.md)