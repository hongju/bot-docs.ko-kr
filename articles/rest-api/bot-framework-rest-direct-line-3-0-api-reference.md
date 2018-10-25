---
title: API 참조 - 직접 회선 API 3.0 | Microsoft Docs
description: Direct Line API 3.0의 헤더, HTTP 상태 코드, 스키마, 작업 및 개체에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: d69f1f658520790ff429ecd25a190319e321164d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998110"
---
# <a name="api-reference---direct-line-api-30"></a>API 참조 - 직접 회선 API 3.0

직업 회선 API 3.0을 사용하여 클라이언트 응용 프로그램이 봇과 통신하도록 할 수 있습니다. 직접 회선 API 3.0은 HTTPS를 통해 업계 표준 REST와 JSON을 사용합니다.

## <a name="base-uri"></a>기본 URI

직접 회선 API 3.0에 액세스하려면 모든 API 요청에 이 기본 URI를 사용합니다.

`https://directline.botframework.com`

## <a name="headers"></a>헤더

표준 HTTP 요청 헤더 외에 직접 회선 API 요청에는 요청을 실행 중인 클라이언트를 인증할 비밀 또는 토큰을 지정하는 `Authorization` 헤더를 포함해야 합니다. 다음 형식을 사용하여 `Authorization` 헤더를 지정합니다.

```http
Authorization: Bearer SECRET_OR_TOKEN
```

클라이언트가 직접 회선 API 요청을 인증하는 데 사용할 수 있는 비밀 또는 토큰을 가져오는 방법에 대한 자세한 내용은 [인증](bot-framework-rest-direct-line-3-0-authentication.md)을 참조하세요.

## <a name="http-status-codes"></a>HTTP 상태 코드

각 응답과 함께 반환되는 <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">HTTP 상태 코드</a>는 해당 요청의 결과를 나타냅니다. 

| HTTP 상태 코드 | 의미 |
|----|----|
| 200 | 요청이 성공했습니다. |
| 201 | 요청이 성공했습니다. |
| 202 | 요청이 처리를 위해 수락되었습니다. |
| 204 | 요청이 성공했지만 콘텐츠가 반환되지 않았습니다. |
| 400 | 요청의 형식이 잘못되었거나 요청이 잘못되었습니다. |
| 401 | 클라이언트에는 요청할 수 있는 권한이 없습니다. `Authorization` 헤더가 누락되거나 형식이 잘못되어 이 상태 코드가 종종 발생합니다. |
| 403 | 클라이언트는 요청한 작업을 수행하는 데 허용되지 않습니다. 요청이 이전에는 유효했으나 만료된 토큰을 지정하는 경우 [ErrorResponse](bot-framework-rest-connector-api-reference.md#errorresponse-object) 개체 내에서 반환된 [Error](bot-framework-rest-connector-api-reference.md#error-object)의 `code` 속성이 `TokenExpired`으로 설정됩니다. |
| 404 | 요청된 리소스를 찾을 수 없습니다. 일반적으로 이 상태 코드는 잘못된 요청 URI를 나타냅니다. |
| 500 | 직접 회선 서비스 내에서 내부 서버 오류가 발생했습니다. |
| 502 | 봇이 사용 불가능하거나 오류를 반환했습니다. **이는 일반적인 오류 코드입니다.** |

> [!NOTE]
> HTTP 상태 코드 101은 WebSocket 클라이언트에서 처리될 가능성이 높지만 WebSocket 연결 경로에서 사용됩니다.

### <a name="errors"></a>오류

4xx 범위 또는 5xx 범위의 HTTP 상태 코드를 지정하는 모든 응답에서 오류 관련 정보를 제공하는 응답 본문에는 [ErrorResponse](bot-framework-rest-connector-api-reference.md#errorresponse-object) 개체가 포함됩니다. 4xx 범위의 오류 응답을 받으면 **ErrorResponse** 개체를 검사하여 오류의 원인을 파악하고 요청을 다시 제출하기 전에 문제를 해결합니다.

> [!NOTE]
> **ErrorResponse** 개체 내의 `code` 속성에 지정된 HTTP 상태 코드 및 값은 안정적입니다. **ErrorResponse** 개체 내의 `message` 속성에 지정된 값은 시간에 따라 변경될 수 있습니다.

다음 코드 조각은 예제 요청 및 결과 오류 응답을 보여 줍니다.

#### <a name="request"></a>요청

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
[detail omitted]
```

#### <a name="response"></a>response
```http
HTTP/1.1 502 Bad Gateway
[other headers]
```
```json
{
    "error": {
        "code": "BotRejectedActivity",
        "message": "Failed to send activity: bot returned an error"
    }
}
```

## <a name="token-operations"></a>토큰 작업 
이러한 작업을 사용하여 클라이언트가 단일 대화에 액세스하는 데 사용할 수 있는 토큰을 만들거나 새로 고칩니다.

| 작업(Operation) | 설명 |
|----|----|
| [토큰 생성](#generate-token) | 새 대화에 대한 토큰을 생성합니다. | 
| [토큰 새로 고침](#refresh-token) | 토큰을 새로 고칩니다. | 

### <a name="generate-token"></a>토큰 생성
하나의 대화에 유효한 토큰을 생성합니다. 
```http 
POST /v3/directline/tokens/generate
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [Conversation](#conversation-object) 개체 | 

### <a name="refresh-token"></a>토큰 새로 고침
토큰을 새로 고칩니다. 
```http 
POST /v3/directline/tokens/refresh
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [Conversation](#conversation-object) 개체 | 

## <a name="conversation-operations"></a>대화 작업 
이러한 작업을 사용하여 봇과 대화를 열고 클라이언트와 봇 간에 활동을 교환합니다.

| 작업(Operation) | 설명 |
|----|----|
| [대화 시작](#start-conversation) | 봇과 새 대화를 엽니다. | 
| [대화 정보 가져오기](#get-conversation-information) | 기존 대화에 대한 정보를 가져옵니다. 이 작업은 클라이언트가 대화에 [다시 연결](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)하는 데 사용할 수 있는 새로운 WebSocket 스트림 URL을 생성합니다. |
| [활동 가져오기](#get-activities) | 봇에서 활동을 검색합니다. |
| [활동 보내기](#send-an-activity) | 봇에 활동을 보냅니다. | 
| [파일 업로드 및 보내기](#upload-send-files) | 파일을 첨부 파일로 업로드하고 보냅니다. |

### <a name="start-conversation"></a>대화 시작
봇과 새 대화를 엽니다. 
```http 
POST /v3/directline/conversations
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [Conversation](#conversation-object) 개체 | 

### <a name="get-conversation-information"></a>대화 정보 가져오기
기존 대화에 대한 정보를 가져오고 클라이언트가 대화에 [다시 연결](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)하는 데 사용할 수 있는 새로운 WebSocket 스트림 URL을 생성합니다. 필요에 따라 요청 URI에서 `watermark` 매개 변수를 제공하여 클라이언트에 표시된 가장 최근 메시지를 나타냅니다.
```http 
GET /v3/directline/conversations/{conversationId}?watermark={watermark_value}
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [Conversation](#conversation-object) 개체 | 

### <a name="get-activities"></a>활동 가져오기
지정된 대화에 대한 활동을 봇에서 검색합니다. 필요에 따라 요청 URI에서 `watermark` 매개 변수를 제공하여 클라이언트에 표시된 가장 최근 메시지를 나타냅니다. 

```http 
GET /v3/directline/conversations/{conversationId}/activities?watermark={watermark_value}
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [ActivitySet](#activityset-object) 개체입니다. 응답에는 `watermark`가 `ActivitySet` 개체의 속성으로 포함됩니다. 클라이언트는 활동이 반환되지 않을 때까지 `watermark` 값을 전달하여 사용 가능한 활동을 페이징해야 합니다. | 

### <a name="send-an-activity"></a>활동 보내기
봇에 활동을 보냅니다. 
```http 
POST /v3/directline/conversations/{conversationId}/activities
```

| | |
|----|----|
| **요청 본문** | [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 개체 |
| **반환** | 봇에 전송된 활동의 ID를 지정하는 `id` 속성이 포함된 [ResourceResponse](bot-framework-rest-connector-api-reference.md#resourceresponse-object)입니다. | 

### <a id="upload-send-files"></a> 파일 업로드 및 보내기
파일을 첨부 파일로 업로드하고 보냅니다. 요청 URI에서 `userId` 매개 변수를 설정하여 첨부 파일을 보내는 사용자의 ID를 지정합니다.
```http 
POST /v3/directline/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **요청 본문** | 단일 첨부 파일의 경우 요청 본문을 파일 콘텐츠로 채웁니다. 여러 첨부 파일의 경우 첨부 파일별로 하나의 파트를 포함하고, 필요한 경우 지정된 첨부 파일의 컨테이너로 제공되어야 하는 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 개체에 대한 하나의 파트를 포함하는 다중 파트 요청 본문을 만듭니다. 자세한 내용은 [봇에 작업 보내기](bot-framework-rest-direct-line-3-0-send-activity.md)를 참조하세요. |
| **반환** | 봇에 전송된 활동의 ID를 지정하는 `id` 속성이 포함된 [ResourceResponse](bot-framework-rest-connector-api-reference.md#resourceresponse-object)입니다. | 

> [!NOTE]
> 업로드된 파일은 24시간 이후 삭제됩니다.

## <a name="schema"></a>스키마

직접 회선 3.0 스키마에는 `ActivitySet` 개체 및 `Conversation` 개체 뿐만 아니라 [Bot Framework v3 스키마](bot-framework-rest-connector-api-reference.md#objects)로 정의된 모든 개체가 포함되어 있습니다.

### <a name="activityset-object"></a>ActivitySet 개체 
활동 집합을 정의합니다.<br/><br/>

| 자산 | type | 설명 |
|----|----|----|
| **activities** | [Activity](bot-framework-rest-connector-api-reference.md#activity-object)[] | **Activity** 개체의 배열입니다. |
| **watermark** | string | 집합 내에서 활동의 최대 워터마크입니다. 클라이언트는 `watermark` 값을 사용하여 [봇에서 활동을 검색](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get)하거나 [새 WebSocket 스트림 URL을 생성](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)할 때 확인한 가장 최근 메시지를 나타낼 수 있습니다. |

### <a name="conversation-object"></a>대화 개체
직접 회선 대화를 정의합니다.<br/><br/>

| 자산 | type | 설명 |
|----|----|----|
| **conversationId** | string | 지정된 토큰이 유효한 대화를 고유하게 식별하는 ID입니다. |
| **expires_in** | number | 토큰이 만료되기 전 시간(초)입니다. |
| **streamUrl** | string | 대화의 메시지 스트림에 대한 URL입니다. |
| **토큰** | string | 지정된 대화에 유효한 토큰입니다. |

### <a name="activities"></a>활동

클라이언트가 직접 회선을 통해 봇에서 수신하는 각 [활동](bot-framework-rest-connector-api-reference.md#activity-object)에 대해 다음을 수행됩니다.

- 첨부 파일 카드가 유지됩니다.
- 업로드된 첨부 파일의 URL은 전용 연결을 사용하여 숨겨져 있습니다.
- `channelData` 속성은 수정 없이 그대로 보존됩니다. 

클라이언트는 봇에서 [ActivitySet](#activityset-object)의 일부로 여러 활동을 [수신](bot-framework-rest-direct-line-3-0-receive-activities.md)할 수 있습니다. 

클라이언트가 직접 회선을 통해 봇에 [활동](bot-framework-rest-connector-api-reference.md#activity-object)을 전송하는 경우 다음이 적용됩니다.

- `type` 속성은 전송하는 활동 형식(일반적으로 **메시지**)을 지정합니다.
- `from` 속성은 클라이언트에서 선택한 사용자 ID로 채워져야 합니다.
- 첨부 파일에는 기존 리소스의 URL 또는 직접 회선 첨부 파일 끝점을 통해 업로드된 URL이 포함될 수 있습니다.
- `channelData` 속성은 수정 없이 그대로 보존됩니다.

클라이언트는 요청당 하나의 활동을 [전송](bot-framework-rest-direct-line-3-0-send-activity.md)할 수 있습니다. 
