---
title: 봇에 활동 보내기 | Microsoft Docs
description: Direct Line API v3.0을 사용하여 봇에 활동을 보내는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 290a2733b96a458eb3529b0b0854703631e05f22
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000040"
---
# <a name="send-an-activity-to-the-bot"></a>봇에 활동 보내기

Direct Line 3.0 프로토콜을 사용하여 클라이언트와 봇은 몇 가지 [활동](bot-framework-rest-connector-activities.md) 유형을 교환할 수 있습니다. 여기에는 **메시지** 활동, **입력** 활동 및 봇이 지원하는 사용자 지정 활동이 포함됩니다. 클라이언트는 요청당 하나의 활동을 보낼 수 있습니다. 

## <a name="send-an-activity"></a>활동 보내기

봇에게 활동을 보내려면 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 개체를 만들어서 활동을 정의한 다음, 요청 본문에 Activity 개체를 지정하는 `https://directline.botframework.com/v3/directline/conversations/{conversationId}/activities`에 `POST` 요청을 발급해야 합니다.

다음 코드 조각은 활동 보내기 요청 및 응답 예제를 제공합니다.

### <a name="request"></a>요청

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: application/json
[other headers]
```

```json
{
    "type": "message",
    "from": {
        "id": "user1"
    },
    "text": "hello"
}
```

### <a name="response"></a>response

활동이 봇에 전송되면 서비스는 봇의 상태 코드를 반영하는 HTTP 상태 코드로 응답합니다. 봇이 오류를 생성하면 활동 보내기 요청에 대한 응답으로 HTTP 502 응답("잘못된 게이트웨이")이 클라이언트에 반환됩니다. POST가 성공하면 응답은 봇에 보낸 활동의 ID를 지정하는 JSON 페이로드를 포함하게 됩니다.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0001"
}
```

### <a name="total-time-for-the-send-activity-requestresponse"></a>활동 보내기 요청/응답의 총 시간

Direct Line 대화에 메시지를 POST하는 데 걸린 총 시간은 다음의 합계입니다.

- HTTP 요청이 클라이언트에서 Direct Line 서비스로 이동하는 데 소요되는 전송 시간
- Direct Line 내부 처리 시간(일반적으로 120ms 미만)
- Direct Line 서비스에서 봇으로 전송 시간
- 봇 내부 처리 시간
- HTTP 응답이 클라이언트까지 다시 이동하는 데 소요되는 시간

## <a name="send-attachments-to-the-bot"></a>봇에 첨부 파일 보내기

상황에 따라 클라이언트에서 이미지나 문서와 같은 첨부 파일을 봇에 보내야 할 수도 있습니다. 클라이언트는 `POST /v3/directline/conversations/{conversationId}/activities`를 사용하여 보내는 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 개체 내에 첨부 파일의 [URL을 지정](#send-by-url)하거나 `POST /v3/directline/conversations/{conversationId}/upload`를 사용하여 [첨부 파일을 업로드](#upload-attachments)하는 방식으로 봇에 첨부 파일을 보냅니다.

## <a id="send-by-url"></a> URL을 통해 첨부 파일 보내기

`POST /v3/directline/conversations/{conversationId}/activities`를 사용하여 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 개체의 일부로 하나 이상의 첨부 파일을 보내려면 간단히 Activity 개체 안에 [Attachment](bot-framework-rest-connector-api-reference.md#attachment-object) 개체를 하나 이상 포함하고 각 Attachment 개체의 `contentUrl` 속성을 첨부 파일의 HTTP, HTTPS 또는 `data` URI를 지정하도록 설정합니다.

## <a id="upload-attachments"></a> 업로드를 통해 첨부 파일 보내기

봇에 보낼 이미지나 문서가 클라이언트의 장치에 있지만 파일에 해당하는 URL은 없는 경우도 있을 수 있습니다. 이런 경우 클라이언트는 `POST /v3/directline/conversations/{conversationId}/upload` 요청을 발급하여 업로드를 통해 봇에게 첨부 파일을 보낼 수 있습니다. 요청의 형식과 콘텐츠는 클라이언트가 [단일 첨부 파일을 보내는지](#upload-one-attachment) 또는 [여러 첨부 파일을 보내는지](#upload-multiple-attachments)에 따라 달라집니다.

### <a id="upload-one-attachment"></a> 업로드를 통해 단일 첨부 파일 보내기

업로드를 통해 단일 첨부 파일을 보내려면 이 요청을 실행합니다. 

```http
POST https://directline.botframework.com/v3/directline/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

이 요청 URI에서 **{conversationId}** 를 대화의 ID로 변경하고 **{userId}** 는 메시지를 보내는 사용자의 ID로 바꿉니다. `userId` 매개 변수는 필수 사항입니다. 요청 헤더에 `Content-Type`을 설정하여 첨부 파일의 유형을 지정하고 `Content-Disposition`을 설정하여 첨부 파일의 파일 이름을 지정합니다.

다음 코드 조각은 첨부 파일(단일) 보내기 요청 및 응답 예제를 제공합니다.

#### <a name="request"></a>요청

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>response

요청이 성공하면 업로드 완료 시 **메시지** 활동이 봇에 보내지며 클라이언트가 받은 응답에는 보낸 활동의 ID가 포함됩니다.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0003"
}
```

### <a id="upload-multiple-attachments"></a> 업로드를 통해 여러 첨부 파일 보내기

업로드를 통해 여러 첨부 파일을 보내려면 `/v3/directline/conversations/{conversationId}/upload` 엔드포인트에 여러 파트를 포함하는 요청을 `POST`합니다. 요청의 `Content-Type` 헤더를 `multipart/form-data`로 설정하고 각 파트마다 `Content-Type` 헤더 및 `Content-Disposition` 헤더를 포함하여 각 첨부 파일의 형식과 파일 이름을 지정합니다. 요청 URI에서 `userId` 매개 변수를 메시지를 보내는 사용자의 ID로 설정합니다. 

`Content-Type` 헤더 값 `application/vnd.microsoft.activity`를 지정하는 파트를 추가하여 요청 내에 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 개체를 포함할 수 있습니다. 요청에 활동이 포함된 경우, 페이로드의 다른 파트에 의해 지정된 첨부 파일이 활동을 보내기 전에 해당 메시지의 첨부 파일로 추가됩니다. 요청이 활동을 포함하지 않는 경우 빈 활동이 만들어지고 지정된 첨부 파일을 보내는 컨테이너 역할을 수행합니다.

다음 코드 조각은 첨부 파일(여러) 보내기 요청 및 응답 예제를 제공합니다. 이 예제의 요청은 약간의 텍스트와 단일 이미지 첨부 파일을 포함하는 메시지를 보냅니다. 요청에 파트를 더 추가하여 메시지에 여러 개의 첨부 파일을 포함시킬 수 있습니다.

#### <a name="request"></a>요청

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.activity
[other headers]

{
  "type": "message",
  "from": {
    "id": "user1"
  },
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>response

요청이 성공하면 업로드 완료 시 메시지 활동이 봇에 보내지며 클라이언트가 받은 응답에는 보낸 활동의 ID가 포함됩니다.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0004"
}
```

## <a name="additional-resources"></a>추가 리소스

- [주요 개념](bot-framework-rest-direct-line-3-0-concepts.md)
- [인증](bot-framework-rest-direct-line-3-0-authentication.md)
- [대화 시작](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [대화에 다시 연결](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [봇에서 작업 수신](bot-framework-rest-direct-line-3-0-receive-activities.md)
- [대화 종료](bot-framework-rest-direct-line-3-0-end-conversation.md)
