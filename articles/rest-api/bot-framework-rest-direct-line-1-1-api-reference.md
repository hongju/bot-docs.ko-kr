---
title: API 참조 - 직접 회선 API 1.1 | Microsoft Docs
description: Direct Line API 1.1의 헤더, HTTP 상태 코드, 스키마, 작업 및 개체에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 3607957cd5cb8738e8268ece6eba4417250bc596
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997963"
---
# <a name="api-reference---direct-line-api-11"></a>API 참조 - 직접 회선 API 1.1

> [!IMPORTANT]
> 이 문서에는 Direct Line API 1.1에 대한 참조 정보가 있습니다. 클라이언트 응용 프로그램과 봇 간에 새 연결을 만들 경우에는 [직접 회선 API 3.0](bot-framework-rest-direct-line-3-0-api-reference.md)을 대신 사용합니다.

직업 회선 API 1.1을 사용하여 클라이언트 응용 프로그램이 봇과 통신하도록 할 수 있습니다. 직접 회선 API 1.1은 HTTPS를 통해 업계 표준 REST와 JSON을 사용합니다.

## <a name="base-uri"></a>기본 URI

직접 회선 API 1.1에 액세스하려면 모든 API 요청에 이 기본 URI를 사용합니다.

`https://directline.botframework.com`

## <a name="headers"></a>헤더

표준 HTTP 요청 헤더 외에 직접 회선 API 요청에는 요청을 실행 중인 클라이언트를 인증할 비밀 또는 토큰을 지정하는 `Authorization` 헤더를 포함해야 합니다. “전달자” 체계나 “BotConnector” 체계를 사용하여 `Authorization` 헤더를 지정할 수 있습니다. 

**전달자 체계**:
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**BotConnector 체계**:
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

클라이언트가 직접 회선 API 요청을 인증하는 데 사용할 수 있는 비밀 또는 토큰을 가져오는 방법에 대한 자세한 내용은 [인증](bot-framework-rest-direct-line-1-1-authentication.md)을 참조하세요.

## <a name="http-status-codes"></a>HTTP 상태 코드

각 응답과 함께 반환되는 <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">HTTP 상태 코드</a>는 해당 요청의 결과를 나타냅니다. 

| HTTP 상태 코드 | 의미 |
|----|----|
| 200 | 요청이 성공했습니다. |
| 204 | 요청이 성공했지만 콘텐츠가 반환되지 않았습니다. |
| 400 | 요청의 형식이 잘못되었거나 요청이 잘못되었습니다. |
| 401 | 클라이언트에는 요청할 수 있는 권한이 없습니다. `Authorization` 헤더가 누락되거나 형식이 잘못되어 이 상태 코드가 종종 발생합니다. |
| 403 | 클라이언트는 요청한 작업을 수행하는 데 허용되지 않습니다. `Authorization` 헤더에 잘못된 토큰 또는 비밀이 지정되어 이 상태 코드가 종종 발생합니다. |
| 404 | 요청된 리소스를 찾을 수 없습니다. 일반적으로 이 상태 코드는 잘못된 요청 URI를 나타냅니다. |
| 500 | 직접 회선 서비스 내에서 내부 서버 오류가 발생했습니다. |
| 502 | 봇 내에서 오류가 발생하여 봇을 사용할 수 없거나 오류가 반환되었습니다.  **이는 일반적인 오류 코드입니다.** |

## <a name="token-operations"></a>토큰 작업 
이러한 작업을 사용하여 클라이언트가 단일 대화에 액세스하는 데 사용할 수 있는 토큰을 만들거나 새로 고칩니다.

| 작업(Operation) | 설명 |
|----|----|
| [토큰 생성](#generate-token) | 새 대화에 대한 토큰을 생성합니다. | 
| [토큰 새로 고침](#refresh-token) | 토큰을 새로 고칩니다. | 

### <a name="generate-token"></a>토큰 생성
하나의 대화에 유효한 토큰을 생성합니다. 
```http 
POST /api/tokens/conversation
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | 토큰을 나타내는 문자열입니다. | 

### <a name="refresh-token"></a>토큰 새로 고침
토큰을 새로 고칩니다. 
```http 
GET /api/tokens/{conversationId}/renew
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | 새 토큰을 나타내는 문자열입니다. | 

## <a name="conversation-operations"></a>대화 작업 
이러한 작업을 사용하여 봇과 대화를 열고 클라이언트와 봇 간에 메시지를 교환합니다.

| 작업(Operation) | 설명 |
|----|----|
| [대화 시작](#start-conversation) | 봇과 새 대화를 엽니다. | 
| [메시지 가져오기](#get-messages) | 봇에서 메시지를 검색합니다. |
| [메시지 보내기](#send-a-message) | 봇에 메시지를 보냅니다. | 
| [파일 업로드 및 보내기](#upload-send-files) | 파일을 첨부 파일로 업로드하고 보냅니다. |

### <a name="start-conversation"></a>대화 시작
봇과 새 대화를 엽니다. 
```http 
POST /api/conversations
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [Conversation](#conversation-object) 개체 | 

### <a name="get-messages"></a>메시지 가져오기
지정된 대화에 대한 메시지를 봇에서 검색합니다. 요청 URI에서 `watermark` 매개 변수를 설정하여 클라이언트에 표시된 가장 최근 메시지를 나타냅니다. 

```http
GET /api/conversations/{conversationId}/messages?watermark={watermark_value}
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [MessageSet](#messageset-object) 개체. 응답에는 `watermark`가 `MessageSet` 개체의 속성으로 포함됩니다. 클라이언트는 메시지가 반환되지 않을 때까지 `watermark` 값을 전달하여 사용 가능한 메시지를 페이징해야 합니다. | 

### <a name="send-a-message"></a>메시지 보내기
봇에 메시지를 보냅니다. 
```http 
POST /api/conversations/{conversationId}/messages
```

| | |
|----|----|
| **요청 본문** | [Message](#message-object) 개체 |
| **반환** | 응답 본문에 데이터가 반환되지 않습니다. 메시지가 성공적으로 전송되면 서비스가 HTTP 204 상태 코드로 응답합니다. 클라이언트는 [Get Messages](#get-messages) 작업을 사용하여 전송된 메시지를 봇이 클라이언트에 보낸 모든 메시지와 함께 가져올 수 있습니다. |

### <a id="upload-send-files"></a> 파일 업로드 및 보내기
파일을 첨부 파일로 업로드하고 보냅니다. 요청 URI에서 `userId` 매개 변수를 설정하여 첨부 파일을 보내는 사용자의 ID를 지정합니다.
```http 
POST /api/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **요청 본문** | 단일 첨부 파일의 경우 요청 본문을 파일 콘텐츠로 채웁니다. 여러 첨부 파일의 경우 첨부 파일별로 하나의 파트를 포함하고, 필요한 경우 지정된 첨부 파일의 컨테이너로 제공되어야 하는 [Message](#message-object) 개체에 대한 하나의 파트를 포함하는 다중 파트 요청 본문을 만듭니다. 자세한 내용은 [봇에 메시지 보내기](bot-framework-rest-direct-line-1-1-send-message.md)를 참조하세요. |
| **반환** | 응답 본문에 데이터가 반환되지 않습니다. 메시지가 성공적으로 전송되면 서비스가 HTTP 204 상태 코드로 응답합니다. 클라이언트는 [Get Messages](#get-messages) 작업을 사용하여 전송된 메시지를 봇이 클라이언트에 보낸 모든 메시지와 함께 가져올 수 있습니다. | 

> [!NOTE]
> 업로드된 파일은 24시간 이후 삭제됩니다.

## <a name="schema"></a>스키마

직접 회선 1.1 스키마는 다음 개체를 포함하는 Bot Framework v1 스키마의 간소화된 복사본입니다.

### <a name="message-object"></a>Message 개체

클라이언트가 봇에 보내거나 봇에서 받는 메시지를 정의합니다.

| 자산 | type | 설명 |
|----|----|----|
| **id** | string | 메시지를 고유하게 식별하는 ID입니다(직접 회선에서 할당됨). | 
| **conversationId** | string | 대화를 식별하는 ID입니다.  | 
| **created** | string | <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 형식으로 표시된, 메시지가 생성된 날짜 및 시간입니다. | 
| **from** | string | 메시지를 보낸 사람인 사용자를 식별하는 ID입니다. 메시지를 만들 때 클라이언트는 이 속성을 안정적인 사용자 ID로 설정해야 합니다. 아무것도 제공되지 않은 경우 직접 회선은 사용자 ID를 할당하지만 일반적으로 이로 인해 예기치 않은 동작이 발생합니다. | 
| **text** | string | 사용자가 봇으로 보내거나 봇에서 사용자로 보내는 메시지의 텍스트입니다. | 
| **channelData** | object | 채널 관련 콘텐츠가 포함된 개체입니다. 일부 채널에서는 첨부 파일 스키마를 사용하여 표현할 수 없는 추가 정보가 필요한 기능을 제공합니다. 이러한 경우에는 채널 설명서에 정의된 대로 이 속성을 채널 관련 콘텐츠로 설정합니다. 이 데이터는 클라이언트와 봇 간에 수정되지 않은 상태로 전송됩니다. 이 속성은 복합 개체로 설정하거나 비워 두어야 합니다. 문자열, 숫자 또는 기타 단순 형식으로 설정하지 마세요. | 
| **images** | string[] | 메시지에 포함된 이미지의 URL을 포함하는 문자열 배열입니다. 경우에 따라 이 배열의 문자열은 상대 URL일 수 있습니다. 이 배열의 문자열이 “http” 또는 “https”로 시작되지 않으면 `https://directline.botframework.com`을 문자열에 추가하여 완전한 URL을 구성합니다. | 
| **attachments** | [Attachment](#attachment-object)[] | 메시지에 포함된 이미지가 아닌 첨부 파일을 나타내는 **Attachment** 개체의 배열입니다. 배열의 각 개체에는 `url` 속성과 `contentType` 속성이 포함됩니다. 클라이언트가 봇에서 받는 메시지에서 `url` 속성에 상대 URL이 지정되는 경우가 있을 수 있습니다. “http” 또는 “https”로 시작되지 않는 `url` 속성 값의 경우, `https://directline.botframework.com`을 문자열에 추가하여 완전한 URL을 구성합니다. | 

다음 예제에서는 모든 가능한 속성이 포함된 Message 개체를 보여 줍니다. 대부분의 경우 메시지를 만들 때 클라이언트는 `from` 속성과 하나 이상의 콘텐츠 속성(예: `text`, `images`, `attachments` 또는 `channelData`)을 제공하면 됩니다.

```json
{
    "id": "CuvLPID4kDb|000000000000000004",
    "conversationId": "CuvLPID4kDb",
    "created": "2016-10-28T21:19:51.0357965Z",
    "from": "examplebot",
    "text": "Hello!",
    "channelData": {
        "examplefield": "abc123"
    },
    "images": [
        "/attachments/CuvLPID4kDb/0.jpg?..."
    ],
    "attachments": [
        {
            "url": "https://example.com/example.docx",
            "contentType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        }, 
        {
            "url": "https://example.com/example.doc",
            "contentType": "application/msword"
        }
    ]
}
```

### <a name="messageset-object"></a>MessageSet 개체 
메시지 집합을 정의합니다.<br/><br/>

| 자산 | type | 설명 |
|----|----|----|
| **messages** | [Message](#message-object)[] | **Message** 개체의 배열입니다. |
| **watermark** | string | 집합 내 메시지의 최대 워터마크입니다. 클라이언트는 `watermark` 값을 사용하여 [봇에서 메시지를 검색](bot-framework-rest-direct-line-1-1-receive-messages.md)할 때 확인한 가장 최근 메시지를 나타냅니다. |

### <a name="attachment-object"></a>Attachment 개체
이미지가 아닌 첨부 파일을 정의합니다.<br/><br/> 

| 자산 | type | 설명 |
|----|----|----|
| **contentType** | string | 첨부 파일에 있는 콘텐츠의 미디어 유형입니다. |
| **url** | string | 첨부 파일의 콘텐츠에 대한 URL입니다. |

### <a name="conversation-object"></a>Conversation 개체
직접 회선 대화를 정의합니다.<br/><br/>

| 자산 | type | 설명 |
|----|----|----|
| **conversationId** | string | 지정된 토큰이 유효한 대화를 고유하게 식별하는 ID입니다. |
| **토큰** | string | 지정된 대화에 유효한 토큰입니다. |
| **expires_in** | number | 토큰이 만료되기 전 시간(초)입니다. |

### <a name="error-object"></a>Error 개체
오류를 정의합니다.<br/><br/> 

| 자산 | type | 설명 |
|----|----|----|
| **code** | string | 오류 코드 다음 값 중 하나: **MissingProperty**, **MalformedData**, **NotFound**, **ServiceError**, **Internal**, **InvalidRange**, **NotSupported**, **NotAllowed**, **BadCertificate**. |
| **message** | string | 오류에 대한 설명입니다. |
| **statusCode** | number | 상태 코드입니다. |

### <a name="errormessage-object"></a>ErrorMessage 개체
표준화된 메시지 오류 페이로드입니다.<br/><br/> 


|        자산        |          type          |                                 설명                                 |
|------------------------|------------------------|-----------------------------------------------------------------------------|
| <strong>error</strong> | [오류](#error-object) | 오류에 대한 정보를 포함하는 <strong>Error</strong> 개체입니다. |

