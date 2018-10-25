---
title: 봇에게 메시지 보내기 | Microsoft Docs
description: 직접 회선 API v1.1을 사용하여 봇에게 메시지를 보내는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 360ec3a6a6c9a3be16370aaf445f24a237a702e3
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998020"
---
# <a name="send-a-message-to-the-bot"></a>봇에게 메시지 보내기

> [!IMPORTANT]
> 이 문서에서는 직접 회선 API 1.1을 사용하여 봇에게 메시지를 보내는 방법을 설명합니다. 클라이언트 응용 프로그램과 봇 간에 새 연결을 만들 경우에는 [직접 회선 API 3.0](bot-framework-rest-direct-line-3-0-send-activity.md)을 대신 사용합니다.

직접 회선 1.1 프로토콜을 사용하면 클라이언트가 봇을 사용하여 메시지를 교환할 수 있습니다. 이러한 메시지는 봇이 지원하는(Bot Framework v1 또는 Bot Framework v3) 스키마로 변환됩니다. 클라이언트는 요청당 하나의 메시지를 보낼 수 있습니다. 

## <a name="send-a-message"></a>메시지 보내기

봇에게 메시지를 보내려면 [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) 개체를 만들어서 메시지를 정의한 다음, 요청 본문에 Message 개체를 지정하는 `https://directline.botframework.com/api/conversations/{conversationId}/messages`에 `POST` 요청을 발급해야 합니다.

다음 코드 조각은 메시지 보내기 요청 및 응답 예제를 제공합니다.

### <a name="request"></a>요청

```http
POST https://directline.botframework.com/api/conversations/abc123/messages
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
  "text": "hello",
  "from": "user1"
}
```

### <a name="response"></a>response

메시지가 봇에 전송되면 서비스는 봇의 상태 코드를 반영하는 HTTP 상태 코드로 응답합니다. 봇이 오류를 생성하면 메시지 보내기 요청에 대한 응답으로 HTTP 500 응답("내부 서버 오류")이 클라이언트에 반환됩니다. POST가 성공하면 서비스는 HTTP 204 상태 코드를 반환합니다. 응답 본문에 데이터가 반환되지 않습니다. 클라이언트의 메시지와 봇의 모든 메시지는 [폴링](bot-framework-rest-direct-line-1-1-receive-messages.md)을 통해 얻을 수 있습니다. 

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a name="total-time-for-the-send-message-requestresponse"></a>메시지 보내기 요청/응답의 총 시간

직접 회선 대화에 메시지를 POST하는 데 걸린 총 시간은 다음의 합계입니다.

- HTTP 요청이 클라이언트에서 Direct Line 서비스로 이동하는 데 소요되는 전송 시간
- Direct Line 내부 처리 시간(일반적으로 120ms 미만)
- Direct Line 서비스에서 봇으로 전송 시간
- 봇 내부 처리 시간
- HTTP 응답이 클라이언트까지 다시 이동하는 데 소요되는 시간

## <a name="send-attachments-to-the-bot"></a>봇에 첨부 파일 보내기

상황에 따라 클라이언트에서 이미지나 문서와 같은 첨부 파일을 봇에 보내야 할 수도 있습니다. 클라이언트는 `POST /api/conversations/{conversationId}/messages`를 사용하여 보내는 [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) 개체 내에 첨부 파일의 [URL을 지정](#send-by-url)하거나 `POST /api/conversations/{conversationId}/upload`를 사용하여 [첨부 파일을 업로드](#upload-attachments)하는 방식으로 봇에 첨부 파일을 보냅니다.

## <a id="send-by-url"></a> URL을 통해 첨부 파일 보내기

`POST /api/conversations/{conversationId}/messages`를 사용하여 [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) 개체의 일부로 하나 이상의 첨부 파일을 보내려면 메시지의 `images` 배열 및/또는 `attachments` 배열 내에 첨부 파일 URL을 지정합니다.

## <a id="upload-attachments"></a> 업로드를 통해 첨부 파일 보내기

봇에 보낼 이미지나 문서가 클라이언트의 장치에 있지만 파일에 해당하는 URL은 없는 경우도 있을 수 있습니다. 이런 경우 클라이언트는 `POST /api/conversations/{conversationId}/upload` 요청을 발급하여 업로드를 통해 봇에게 첨부 파일을 보낼 수 있습니다. 요청의 형식과 콘텐츠는 클라이언트가 [단일 첨부 파일을 보내는지](#upload-one-attachment) 또는 [여러 첨부 파일을 보내는지](#upload-multiple-attachments)에 따라 달라집니다.

### <a id="upload-one-attachment"></a> 업로드를 통해 단일 첨부 파일 보내기

업로드를 통해 단일 첨부 파일을 보내려면 이 요청을 실행합니다. 

```http
POST https://directline.botframework.com/api/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

이 요청 URI에서 **{conversationId}** 를 대화의 ID로 변경하고 **{userId}** 는 메시지를 보내는 사용자의 ID로 바꿉니다. 요청 헤더에 `Content-Type`을 설정하여 첨부 파일의 유형을 지정하고 `Content-Disposition`을 설정하여 첨부 파일의 파일 이름을 지정합니다.

다음 코드 조각은 첨부 파일(단일) 보내기 요청 및 응답 예제를 제공합니다.

#### <a name="request"></a>요청

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>response

요청이 성공하는 경우 업로드가 완료되면 메시지가 봇에 전송되고 서비스는 HTTP 204 상태 코드를 반환합니다.

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a id="upload-multiple-attachments"></a> 업로드를 통해 여러 첨부 파일 보내기

업로드를 통해 여러 첨부 파일을 보내려면 `/api/conversations/{conversationId}/upload` 엔드포인트에 여러 파트를 포함하는 요청을 `POST`합니다. 요청의 `Content-Type` 헤더를 `multipart/form-data`로 설정하고 각 파트마다 `Content-Type` 헤더 및 `Content-Disposition` 헤더를 포함하여 각 첨부 파일의 형식과 파일 이름을 지정합니다. 요청 URI에서 `userId` 매개 변수를 메시지를 보내는 사용자의 ID로 설정합니다. 

`Content-Type` 헤더 값 `application/vnd.microsoft.bot.message`를 지정하는 파트를 추가하여 요청 내에 [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) 개체를 포함할 수 있습니다. 이렇게 하면 첨부 파일이 포함된 메시지를 클라이언트가 사용자 지정할 수 있습니다. 요청에 메시지가 포함된 경우, 페이로드의 다른 파트에 의해 지정된 첨부 파일이 메시지를 보내기 전에 해당 메시지의 첨부 파일로 추가됩니다. 

다음 코드 조각은 첨부 파일(여러) 보내기 요청 및 응답 예제를 제공합니다. 이 예제의 요청은 약간의 텍스트와 단일 이미지 첨부 파일을 포함하는 메시지를 보냅니다. 요청에 파트를 더 추가하여 메시지에 여러 개의 첨부 파일을 포함시킬 수 있습니다.

#### <a name="request"></a>요청

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.bot.message
[other headers]

{
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe",
  "from": "user1"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>response

요청이 성공하는 경우 업로드가 완료되면 메시지가 봇에 전송되고 서비스는 HTTP 204 상태 코드를 반환합니다.

```http
HTTP/1.1 204 No Content
[other headers]
```

## <a name="additional-resources"></a>추가 리소스

- [주요 개념](bot-framework-rest-direct-line-1-1-concepts.md)
- [인증](bot-framework-rest-direct-line-1-1-authentication.md)
- [대화 시작](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [봇에서 메시지 받기](bot-framework-rest-direct-line-1-1-receive-messages.md)