---
title: API 참조 | Microsoft Docs
description: Bot Connector 서비스 및 Bot State 서비스의 헤더, 작업, 개체 및 오류에 대해 알아봅니다.
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/25/2018
ms.openlocfilehash: 41aceaa20613d9b6b7ac95a7837b4ae197d1dd4a
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464789"
---
# <a name="api-reference"></a>API 참조

> [!NOTE]
> REST API는 SDK와 동일하지 않습니다. REST API는 표준 REST 통신을 허용하기 위해 제공되지만, SDK는 Bot Framework와 상호 작용하는 기본 방법입니다. 

Bot Framework 내에서 Bot Connector 서비스는 봇에서 Bot Framework 포털에 구성된 채널의 사용자와 메시지를 교환할 수 있게 하며, Bot State 서비스를 사용하면 봇에서 Bot Connector 서비스를 통해 수행하는 대화와 관련된 상태 데이터를 저장하고 검색할 수 있습니다. 두 서비스는 모두 HTTPS를 통해 업계 표준 REST와 JSON을 사용합니다.

> [!IMPORTANT]
> Bot Framework State Service API는 프로덕션 환경에 권장되지 않으며, 이후 릴리스에서 사용되지 않을 수 있습니다. 테스트 용도로 메모리 내 저장소를 사용하거나 프로덕션 봇용으로 **Azure 확장**을 사용하도록 봇 코드를 업데이트하는 것이 좋습니다. 자세한 내용은 [.NET](~/dotnet/bot-builder-dotnet-state.md) 또는 [Node](~/nodejs/bot-builder-nodejs-state.md) 구현에 대한 **상태 데이터 관리** 항목을 참조하세요.

## <a name="base-uri"></a>기본 URI

사용자가 봇에 메시지를 보내면 들어오는 요청에는 봇이 응답을 보내야 하는 엔드포인트를 지정하는 `serviceUrl` 속성이 있는 [Activity](#activity-object) 개체가 포함됩니다. Bot Connector 서비스 또는 Bot State 서비스에 액세스하려면 `serviceUrl` 값을 API 요청에 대한 기본 URI로 사용합니다. 

예를 들어 사용자가 봇에 메시지를 보내면 봇에서 다음 활동을 받는다고 가정합니다.

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

사용자의 메시지 내에서 `serviceUrl` 속성은 봇에서 응답을 `https://smba.trafficmanager.net/apis` 엔드포인트에 보내야 함을 나타냅니다. 이 엔드포인트는 이 대화의 컨텍스트에서 봇이 발급하는 모든 후속 요청에 대한 기본 URI가 됩니다. 봇이 사용자에게 사전 메시지를 보내야 하는 경우 `serviceUrl` 값을 저장해야 합니다.

다음 예제에서는 봇이 사용자의 메시지에 응답하기 위해 발급하는 요청을 보여 줍니다. 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have several times available on Saturday!",
    "replyToId": "bf3cc9a2f5de..."
}
```

## <a name="headers"></a>헤더

### <a name="request-headers"></a>헤더 요청

발급하는 모든 API 요청에는 표준 HTTP 요청 헤더 외에도 봇을 인증하기 위한 액세스 토큰을 지정하는 `Authorization` 헤더가 포함되어야 합니다. 다음 형식을 사용하여 `Authorization` 헤더를 지정합니다.

```http
Authorization: Bearer ACCESS_TOKEN
```

봇에 대한 액세스 토큰을 얻는 방법에 대한 자세한 내용은 [Bot Connector 서비스에서 봇 요청 인증](bot-framework-rest-connector-authentication.md#bot-to-connector)을 참조하세요.

### <a name="response-headers"></a>응답 헤더

모든 응답에는 표준 HTTP 응답 헤더 외에도 `X-Correlating-OperationId` 헤더가 포함됩니다. 이 헤더의 값은 요청에 대한 세부 정보가 포함된 Bot Framework 로그 항목에 해당하는 ID입니다. 오류 응답을 받을 때마다 이 헤더의 값을 캡처해야 합니다. 문제를 독립적으로 해결할 수 없는 경우 해당 문제를 보고할 때 이 값을 지원 팀에 제공하는 정보에 포함시키세요.

## <a name="http-status-codes"></a>HTTP 상태 코드

각 응답과 함께 반환되는 <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">HTTP 상태 코드</a>는 해당 요청의 결과를 나타냅니다. 

| HTTP 상태 코드 | 의미 |
|----|----|
| 200 | 요청이 성공했습니다. |
| 201 | 요청이 성공했습니다. |
| 202 | 요청이 처리를 위해 수락되었습니다. |
| 204 | 요청이 성공했지만 콘텐츠가 반환되지 않았습니다. |
| 400 | 요청의 형식이 잘못되었거나 요청이 잘못되었습니다. |
| 401 | 봇에는 요청할 수 있는 권한이 없습니다. |
| 403 | 봇에서 요청된 작업을 수행할 수 없습니다. |
| 404 | 요청된 리소스를 찾을 수 없습니다. |
| 500 | 내부 서버 오류가 발생했습니다. |
| 503 | 서비스를 사용할 수 없습니다. |

### <a name="errors"></a>오류

4xx 범위 또는 5xx 범위의 HTTP 상태 코드를 지정하는 모든 응답에서 오류 관련 정보를 제공하는 응답 본문에는 [ErrorResponse](#errorresponse-object) 개체가 포함됩니다. 4xx 범위의 오류 응답을 받으면 **ErrorResponse** 개체를 검사하여 오류의 원인을 파악하고 요청을 다시 제출하기 전에 문제를 해결합니다.

## <a name="conversation-operations"></a>대화 작업 
대화를 만들고, 메시지(활동)를 보내고, 대화 콘텐츠를 관리하는 데 사용하는 작업은 다음과 같습니다.

| 작업(Operation) | 설명 |
|----|----|
| [대화 만들기](#create-conversation) | 새 대화를 만듭니다. |
| [대화에 보내기](#send-to-conversation) | 지정한 대화의 끝에 활동(메시지)을 보냅니다. |
| [활동에 회신](#reply-to-activity) | 지정한 대화에 활동(메시지)을 지정한 활동에 대한 회신으로 보냅니다. |
| [대화 가져오기](#get-conversations) | 봇이 참여한 대화 목록을 가져옵니다. |
| [대화 멤버 가져오기](#get-conversation-members) | 지정한 대화의 멤버를 가져옵니다. |
| [대화 페이징 멤버 가져오기](#get-conversation-paged-members) | 지정된 대화 멤버를 한 번에 한 페이지씩 가져옵니다. |
| [활동 멤버 가져오기](#get-activity-members) | 지정한 대화 내에서 지정한 활동의 멤버를 가져옵니다. |
| [활동 업데이트](#update-activity) | 기존 활동을 업데이트합니다. |
| [활동 삭제](#delete-activity) | 기존 활동을 삭제합니다. |
| [대화 멤버 삭제](#delete-conversation-member) | 대화에서 멤버를 제거합니다. |
| [대화 기록 보내기](#send-conversation-history) | 과거 대화 활동 기록을 업로드합니다. |
| [채널에 첨부 파일 업로드](#upload-attachment-to-channel) | 첨부 파일을 채널의 Blob Storage에 직접 업로드합니다. |

### <a name="create-conversation"></a>대화 만들기
새 대화를 만듭니다. 
```http 
POST /v3/conversations
```

| | |
|----|----|
| **요청 본문** | [Conversation](#conversation-object) 개체 |
| **반환** | [ConversationResourceResponse](#conversationresourceresponse-object) 개체 | 

### <a name="send-to-conversation"></a>대화에 보내기
지정한 대화에 활동(메시지)을 보냅니다. 활동은 채널의 타임스탬프 또는 의미 체계에 따라 대화의 끝에 추가됩니다. 대화 내의 특정 메시지에 회신하려면 [활동에 회신](#reply-to-activity)을 대신 사용합니다.
```http
POST /v3/conversations/{conversationId}/activities
```

| | |
|----|----|
| **요청 본문** | [Activity](#activity-object) 개체 |
| **반환** | [Identification](#identification-object) 개체 | 

### <a name="reply-to-activity"></a>활동에 회신
지정한 대화에 활동(메시지)을 지정한 활동에 대한 회신으로 보냅니다. 채널에서 지원하는 경우 활동이 다른 활동에 대한 응답으로 추가됩니다. 채널에서 중첩된 응답을 지원하지 않는 경우 이 작업은 [대화에 보내기](#send-to-conversation)처럼 작동합니다.
```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **요청 본문** | [Activity](#activity-object) 개체 |
| **반환** | [Identification](#identification-object) 개체 | 

### <a name="get-conversations"></a>대화 가져오기
봇이 참여한 대화 목록을 가져옵니다.
```http
GET /v3/conversations?continuationToken={continuationToken}
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [ConversationsResult](#conversationsresult-object) 개체 | 

### <a name="get-conversation-members"></a>대화 멤버 가져오기
지정한 대화의 멤버를 가져옵니다.
```http
GET /v3/conversations/{conversationId}/members
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [ChannelAccount](#channelaccount-object) 개체의 배열 | 

### <a name="get-conversation-paged-members"></a>대화 페이징 멤버 가져오기
지정된 대화 멤버를 한 번에 한 페이지씩 가져옵니다.
```http
GET /v3/conversations/{conversationId}/pagedmembers?pageSize={pageSize}&continuationToken={continuationToken}
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | 더 많은 값을 가져오는 데 사용할 수 있는 [ChannelAccount](#channelaccount-object) 개체와 연속 토큰의 배열 |

### <a name="get-activity-members"></a>활동 멤버 가져오기
지정한 대화 내에서 지정한 활동의 멤버를 가져옵니다.
```http
GET /v3/conversations/{conversationId}/activities/{activityId}/members
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [ChannelAccount](#channelaccount-object) 개체의 배열 | 

### <a name="update-activity"></a>활동 업데이트
일부 채널에서는 봇 대화의 새 상태를 반영하도록 기존 활동을 편집할 수 있습니다. 예를 들어 사용자가 단추 중 하나를 클릭하면 대화의 메시지에서 단추를 제거할 수 있습니다. 성공하면 이 작업은 지정한 대화 내에서 지정한 활동을 업데이트합니다. 
```http
PUT /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **요청 본문** | [Activity](#activity-object) 개체 |
| **반환** | [Identification](#identification-object) 개체 | 

### <a name="delete-activity"></a>활동 삭제
일부 채널에서는 기존 활동을 삭제할 수 있습니다. 성공하면 이 작업은 지정한 대화에서 지정한 활동을 제거합니다.
```http
DELETE /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | 작업 결과를 나타내는 HTTP 상태 코드입니다. 응답 본문에는 아무 것도 지정되지 않습니다. | 

### <a name="delete-conversation-member"></a>대화 멤버 삭제
대화에서 멤버를 제거합니다. 해당 멤버가 대화의 마지막 멤버인 경우 대화도 삭제됩니다.
```http
DELETE /v3/conversations/{conversationId}/members/{memberId}
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | 작업 결과를 나타내는 HTTP 상태 코드입니다. 응답 본문에는 아무 것도 지정되지 않습니다. | 

### <a name="send-conversation-history"></a>대화 기록 보내기
클라이언트가 렌더링할 수 있도록 과거 대화 활동 기록을 업로드합니다.
```http
POST /v3/conversations/{conversationId}/activities/history
```

| | |
|----|----|
| **요청 본문** | [Transcript](#transcript-object) 개체 |
| **반환** | [ResourceResponse](#resourceresponse-object) 개체. | 

### <a name="upload-attachment-to-channel"></a>채널에 첨부 파일 업로드
지정한 대화에 대한 첨부 파일을 채널의 Blob Storage에 직접 업로드합니다. 이렇게 하면 호환되는 저장소에 데이터를 저장할 수 있습니다.
```http 
POST /v3/conversations/{conversationId}/attachments
```

| | |
|----|----|
| **요청 본문** | [AttachmentUpload](#attachmentupload-object) 개체 |
| **반환** | [ResourceResponse](#resourceresponse-object) 개체. **id** 속성은 [첨부 파일 정보 가져오기](#get-attachment-info) 작업과 [첨부 파일 가져오기](#get-attachment) 작업에서 사용할 수 있는 첨부 파일 ID를 지정합니다. | 

## <a name="attachment-operations"></a>첨부 파일 작업 
첨부 파일에 대한 정보와 파일 자체의 이진 데이터를 검색하는 데 사용하는 작업은 다음과 같습니다.

| 작업(Operation) | 설명 |
|----|----|
| [첨부 파일 정보 가져오기](#get-attachment-info) | 파일 이름, 파일 형식 및 사용 가능한 보기(예: 원시 또는 썸네일 이미지)를 포함하여 지정한 첨부 파일에 대한 정보를 가져옵니다. |
| [첨부 파일 가져오기](#get-attachment) | 지정한 첨부 파일에 대한 지정한 보기를 이진 콘텐츠로 가져옵니다. | 

### <a name="get-attachment-info"></a>첨부 파일 정보 가져오기 
파일 이름, 형식 및 사용 가능한 보기(예: 원시 또는 썸네일 이미지)를 포함하여 지정한 첨부 파일에 대한 정보를 가져옵니다.
```http
GET /v3/attachments/{attachmentId}
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [AttachmentInfo](#attachmentinfo-object) 개체 | 

### <a name="get-attachment"></a>첨부 파일 가져오기
지정한 첨부 파일에 대한 지정한 보기를 이진 콘텐츠로 가져옵니다.
```http
GET /v3/attachments/{attachmentId}/views/{viewId}
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | 지정한 첨부 파일에 대한 지정한 보기를 나타내는 이진 콘텐츠 | 

## <a name="state-operations"></a>상태 작업
상태 데이터를 저장하고 검색하는 데 사용하는 작업은 다음과 같습니다.

| 작업(Operation) | 설명 |
|----|----|
| [사용자 데이터 설정](#set-user-data) | 채널의 특정 사용자에 대한 상태 데이터를 저장합니다. | 
| [대화 데이터 설정](#set-conversation-data) | 채널의 특정 대화에 대한 상태 데이터를 저장합니다. | 
| [프라이빗 대화 데이터 설정](#set-private-conversation-data) | 채널의 특정 대화 컨텍스트 내에 포함된 특정 사용자에 대한 상태 데이터를 저장합니다. | 
| [사용자 데이터 가져오기](#get-user-data) | 채널의 모든 대화에서 이전에 저장된 특정 사용자에 대한 상태 데이터를 검색합니다. | 
| [대화 데이터 가져오기](#get-conversation-data) | 이전에 저장된 채널의 특정 대화에 대한 상태 데이터를 검색합니다. | 
| [프라이빗 대화 데이터 가져오기](#get-private-conversation-data) | 이전에 저장된 채널의 특정 대화 컨텍스트 내에 있는 특정 사용자에 대한 상태 데이터를 검색합니다. | 
| [사용자에 대한 상태 삭제](#delete-state-for-user) | [사용자 데이터 설정](#set-user-data) 작업 또는 [프라이빗 대화 데이터 설정](#set-private-conversation-data) 작업을 사용하여 이전에 저장된 사용자에 대한 상태 데이터를 삭제합니다. | 

### <a name="set-user-data"></a>사용자 데이터 설정
지정한 채널의 지정한 사용자에 대한 상태 데이터를 저장합니다.
```http
POST /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **요청 본문** | [BotData](#botdata-object) 개체 |
| **반환** | [BotData](#botdata-object) 개체 | 

### <a name="set-conversation-data"></a>대화 데이터 설정
지정한 채널의 지정한 대화에 대한 상태 데이터를 저장합니다.
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

| | |
|----|----|
| **요청 본문** | [BotData](#botdata-object) 개체 |
| **반환** | [BotData](#botdata-object) 개체 | 

### <a name="set-private-conversation-data"></a>프라이빗 대화 데이터 설정
지정한 채널의 지정한 대화 컨텍스트 내에 있는 지정한 사용자에 대한 상태 데이터를 저장합니다.
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **요청 본문** | [BotData](#botdata-object) 개체 |
| **반환** | [BotData](#botdata-object) 개체 | 

### <a name="get-user-data"></a>사용자 데이터 가져오기
지정한 채널의 모든 대화에서 이전에 저장된 특정 사용자에 대한 상태 데이터를 검색합니다.
```http
GET /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [BotData](#botdata-object) 개체 | 

### <a name="get-conversation-data"></a>대화 데이터 가져오기
지정한 채널에서 이전에 저장된 특정 사용자에 대한 상태 데이터를 검색합니다.
```http
GET /v3/botstate/{channelId}/conversations/{conversationId} 
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [BotData](#botdata-object) 개체 | 

### <a name="get-private-conversation-data"></a>프라이빗 대화 데이터 가져오기
이전에 저장된 지정한 채널의 지정한 대화 컨텍스트 내에 있는 지정한 사용자에 대한 상태 데이터를 검색합니다.
```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | [BotData](#botdata-object) 개체 | 

### <a name="delete-state-for-user"></a>사용자에 대한 상태 삭제
[사용자 데이터 설정](#set-user-data) 작업 또는 [프라이빗 대화 데이터 설정](#set-private-conversation-data) 작업을 사용하여 이전에 저장된 지정한 채널의 지정한 사용자에 대한 상태 데이터를 삭제합니다.
```http
DELETE /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **요청 본문** | 해당 없음 |
| **반환** | 문자열 배열(ID) | 

## <a id="objects"></a> 스키마

스키마는 봇에서 사용자와 통신하는 데 사용할 수 있는 개체와 해당 속성을 정의합니다. 

| Object | 설명 |
| ---- | ---- |
| [Activity 개체](#activity-object) | 봇과 사용자 간에 교환되는 메시지를 정의합니다. |
| [AnimationCard 개체](#animationcard-object) | 애니메이션 GIF 또는 짧은 비디오를 재생할 수 있는 카드를 정의합니다. |
| [Attachment 개체](#attachment-object) | 메시지에 포함할 추가 정보를 정의합니다. 첨부 파일은 미디어 파일(예: 오디오, 비디오, 이미지, 파일) 또는 서식 있는 카드일 수 있습니다. |
| [AttachmentData 개체](#attachmentdata-object) | 첨부 파일 데이터를 설명합니다. |
| [AttachmentInfo 개체](#attachmentinfo-object) | 첨부 파일을 설명합니다. |
| [AttachmentView 개체](#attachmentview-object) | 첨부 파일 보기를 정의합니다. |
| [AttachmentUpload 개체](#attachmentupload-object) | 업로드할 첨부 파일을 정의합니다. |
| [AudioCard 개체](#audiocard-object) | 오디오 파일을 재생할 수 있는 카드를 정의합니다. |
| [BotData 개체](#botdata-object) | Bot State 서비스를 사용하여 저장된 사용자, 대화 또는 특정 대화의 컨텍스트에 있는 사용자에 대한 상태 데이터를 정의합니다. |
| [CardAction 개체](#cardaction-object) | 수행할 작업을 정의합니다. |
| [CardImage 개체](#cardimage-object) | 카드에 표시할 이미지를 정의합니다. |
| [ChannelAccount 개체](#channelaccount-object) | 채널의 봇 또는 사용자 계정을 정의합니다. |
| [Conversation 개체](#conversation-object) | 대화 내에 포함된 봇 및 사용자를 포함한 대화를 정의합니다. |
| [ConversationAccount 개체](#conversationaccount-object) | 채널의 대화를 정의합니다. |
| [ConversationMembers 개체](#conversationmembers-object) | 대화의 멤버를 정의합니다. |
| [ConversationParameters 개체](#conversationparameters-object) | 새 대화를 만들기 위한 매개 변수를 정의합니다. |
| [ConversationReference 개체](#conversationreference-object) | 대화의 특정 지점을 정의합니다. |
| [ConversationResourceResponse 개체](#conversationresourceresponse-object) | [대화 만들기](#create-conversation)에 대한 응답을 정의합니다. |
| [ConversationsResult 개체](#conversationsresult-object) | [대화 가져오기](#get-conversations) 호출의 결과를 정의합니다. |
| [Entity 개체](#entity-object) | 엔터티 개체를 정의합니다. |
| [Error 개체](#error-object) | 오류를 정의합니다. |
| [ErrorResponse 개체](#errorresponse-object) | HTTP API 응답을 정의합니다. |
| [Fact 개체](#fact-object) | 팩트가 포함된 키-값 쌍을 정의합니다. |
| [GeoCoordinates 개체](#geocoordinates-object) | World Geodetic System(WSG84) 좌표를 사용하여 지리적 위치를 정의합니다. |
| [HeroCard 개체](#herocard-object) | 큰 이미지, 제목, 텍스트 및 작업 단추가 있는 카드를 정의합니다. |
| [Identification 개체](#identification-object) | 리소스를 식별합니다. |
| [MediaEventValue 개체](#mediaeventvalue-object) | 미디어 이벤트에 대한 보조 매개 변수입니다. |
| [MediaUrl 개체](#mediaurl-object) | 미디어 파일의 원본에 대한 URL을 정의합니다. |
| [Mention 개체](#mention-object) | 대화에서 언급된 사용자 또는 봇을 정의합니다. |
| [MessageReaction 개체](#messagereaction-object) | 메시지에 대한 반응을 정의합니다. |
| [Place 개체](#place-object) | 대화에서 언급된 위치를 정의합니다. |
| [ReceiptCard 개체](#receiptcard-object) | 구매 영수증이 포함된 카드를 정의합니다. |
| [ReceiptItem 개체](#receiptitem-object) | 영수증 내의 품목을 정의합니다. |
| [ResourceResponse 개체](#resourceresponse-object) | 리소스를 정의합니다. |
| [SemanticAction 개체](#semanticaction-object) | 프로그래밍 방식 작업에 대한 참조를 정의합니다. |
| [SignInCard 개체](#signincard-object) | 사용자가 서비스에 로그인할 수 있도록 하는 카드를 정의합니다. |
| [SuggestedActions 개체](#suggestedactions-object) | 사용자가 선택할 수 있는 옵션을 정의합니다. |
| [ThumbnailCard 개체](#thumbnailcard-object) | 썸네일 이미지, 제목, 텍스트 및 작업 단추가 있는 카드를 정의합니다. |
| [ThumbnailUrl 개체](#thumbnailurl-object) | 이미지 원본에 대한 URL을 정의합니다. |
| [Transcript 개체](#transcript-object) | [대화 기록 보내기](#send-conversation-history)를 사용하여 업로드할 활동 컬렉션입니다. |
| [VideoCard 개체](#videocard-object) | 비디오를 재생할 수 있는 카드를 정의합니다. |

### <a name="activity-object"></a>Activity 개체
봇과 사용자 간에 교환되는 메시지를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **action** | 문자열 | 적용할 작업 또는 적용된 작업입니다. **type** 속성을 사용하여 작업의 컨텍스트를 결정합니다. 예를 들어 **type**이 **contactRelationUpdate**일 때 사용자가 자신의 연락처 목록에 봇을 추가한 경우 **action** 속성의 값은 **add**이고, 해당 연락처 목록에서 봇을 제거한 경우 **remove**입니다. |
| **attachments** | [Attachment](#attachment-object)[] | 메시지에 포함할 추가 정보를 정의하는 **Attachment** 개체의 배열입니다. 각 첨부 파일은 미디어 파일(예: 오디오, 비디오, 이미지, 파일) 또는 서식 있는 카드일 수 있습니다. |
| **attachmentLayout** | 문자열 | 메시지에 포함된 **attachments** 서식 있는 카드의 레이아웃입니다. **carousel**, **list** 값 중 하나입니다. 서식 있는 카드 첨부 파일에 대한 자세한 내용은 [메시지에 서식 있는 카드 첨부 파일 추가](bot-framework-rest-connector-add-rich-cards.md)를 참조하세요. |
| **channelData** | object | 채널 관련 콘텐츠가 포함된 개체입니다. 일부 채널에서는 첨부 파일 스키마를 사용하여 표현할 수 없는 추가 정보가 필요한 기능을 제공합니다. 이러한 경우 채널의 설명서에서 정의한 대로 이 속성을 채널 관련 콘텐츠로 설정합니다. 자세한 내용은 [채널 관련 기능 구현](bot-framework-rest-connector-channeldata.md)을 참조하세요. |
| **channelId** | 문자열 | 채널을 고유하게 식별하는 ID입니다. 채널별로 설정합니다. | 
| **대화** | [ConversationAccount](#conversationaccount-object) | 활동이 속한 대화를 정의하는 **ConversationAccount** 개체입니다. |
| **code** | 문자열 | 대화가 종료된 이유를 나타내는 코드입니다. |
| **entities** | object[] | 메시지에 언급된 엔터티를 나타내는 개체의 배열입니다. 이 배열의 개체는 <a href="http://schema.org/" target="_blank">Schema.org</a> 개체일 수 있습니다. 예를 들어 배열에는 대화에서 언급된 사람을 식별하는 [Mention](#mention-object) 개체와 대화에서 언급된 위치를 식별하는 [Place](#place-object) 개체가 포함될 수 있습니다. |
| **from** | [ChannelAccount](#channelaccount-object) | 메시지의 보낸 사람을 지정하는 **ChannelAccount** 개체입니다. |
| **historyDisclosed** | 부울 | 기록이 공개되는지 여부를 나타내는 플래그입니다. 기본값은 **false**입니다. |
| **id** | 문자열 | 채널의 활동을 고유하게 식별하는 ID입니다. | 
| **inputHint** | 문자열 | 메시지가 클라이언트에 전달된 후에 봇에서 사용자 입력을 수락, 요구 또는 무시하는지 여부를 나타내는 값입니다. **acceptingInput**, **expectingInput**, **ignoringInput** 값 중 하나입니다. |
| **locale** | 문자열 | 메시지 내에 텍스트를 `<language>-<country>` 형식으로 표시하는 데 사용해야 하는 언어의 로캘입니다. 채널은 이 속성을 사용하여 사용자의 언어를 나타내므로 봇에서 표시 문자열을 해당 언어로 지정할 수 있습니다. 기본값은 **en-US**입니다. |
| **localTimestamp** | 문자열 | 메시지를 현지 표준 시간대로 보낸 날짜와 시간으로 <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 형식으로 표시됩니다. |
| **membersAdded** | [ChannelAccount](#channelaccount-object)[] | 대화에 조인한 사용자의 목록을 나타내는 **ChannelAccount** 개체의 배열입니다. **type** 활동이 "conversationUpdate"이고 사용자가 대화에 조인한 경우에만 표시됩니다. | 
| **membersRemoved** | [ChannelAccount](#channelaccount-object)[] | 대화에서 나간 사용자의 목록을 나타내는 **ChannelAccount** 개체의 배열입니다. **type** 활동이 "conversationUpdate"이고 사용자가 대화에서 나간 경우에만 표시됩니다. | 
| **name** | 문자열 | 호출할 작업의 이름이거나 이벤트의 이름입니다. |
| **recipient** | [ChannelAccount](#channelaccount-object) | 메시지를 받는 사람을 지정하는 **ChannelAccount** 개체입니다. |
| **relatesTo** | [ConversationReference](#conversationreference-object) | 대화의 특정 지점을 정의하는 **ConversationReference** 개체입니다. |
| **replyToId** | 문자열 | 이 메시지에서 회신하는 메시지의 ID입니다. 사용자가 보낸 메시지에 회신하려면 이 속성을 사용자의 메시지 ID로 설정합니다. 일부 채널에서는 스레드된 회신을 지원하지 않습니다. 이러한 경우 채널에서 이 속성을 무시하고 시간 정렬된 의미 체계(타임스탬프)를 사용하여 메시지에 대화를 추가합니다. | 
| **serviceUrl** | 문자열 | 채널의 서비스 엔드포인트를 지정하는 URL입니다. 채널별로 설정합니다. | 
| **speak** | 문자열 | 음성 지원 채널에서 봇으로 발화될 텍스트입니다. 음성, 속도, 음량, 발음 및 음조와 같이 봇 발화의 다양한 특성을 제어하려면 이 속성을 <a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">SSML(Speech Synthesis Markup Language)</a> 형식으로 지정합니다. |
| **suggestedActions** | [SuggestedActions](#suggestedactions-object) | 사용자가 선택할 수 있는 옵션을 정의하는 **SuggestedActions** 개체입니다. |
| **summary** | 문자열 | 메시지에 포함된 정보에 대한 요약입니다. 예를 들어 이메일 채널에서 보내는 메시지의 경우 이 속성은 이메일 메시지의 처음 50개 문자를 지정할 수 있습니다. |
| **text** | 문자열 | 사용자가 봇으로 보내거나 봇에서 사용자로 보내는 메시지의 텍스트입니다. 이 속성의 내용에 적용되는 제한 사항은 채널의 설명서를 참조하세요. |
| **textFormat** | 문자열 | 메시지의 **text** 형식입니다. **markdown**, **plain**, **xml** 값 중 하나입니다. 텍스트 형식에 대한 자세한 내용은 [메시지 만들기](bot-framework-rest-connector-create-messages.md)를 참조하세요. |
| **timestamp** | 문자열 | 메시지를 UTC 표준 시간대로 보낸 날짜와 시간으로 <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 형식으로 표시됩니다. |
| **topicName** | 문자열 | 활동이 속한 대화의 주제입니다. |
| **type** | 문자열 | 활동의 유형입니다. **contactRelationUpdate**, **conversationUpdate**, **deleteUserData**, **message**, **typing**, **event** 및 **endOfConversation** 값 중 하나입니다. 활동 유형에 대한 자세한 내용은 [활동 개요](bot-framework-rest-connector-activities.md)를 참조하세요. |
| **값** | object | 무한 값입니다. |
| **semanticAction** |[SemanticAction](#semanticaction-object) | 프로그래밍 방식 작업의 참조를 나타내는 **SemanticAction** 개체입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="animationcard-object"></a>AnimationCard 개체
애니메이션 GIF 또는 짧은 비디오를 재생할 수 있는 카드를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **autoloop** | 부울 | 마지막 애니메이션 GIF가 끝날 때 해당 애니메이션 GIF 목록을 재생할지 여부를 나타내는 플래그입니다. 애니메이션을 자동으로 다시 재생하려면 이 속성을 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **true**입니다. |
| **autostart** | 부울 | 카드가 표시될 때 애니메이션을 자동으로 재생할지 여부를 나타내는 플래그입니다. 애니메이션을 자동으로 재생하려면 이 속성을 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **true**입니다. |
| **buttons** | [CardAction](#cardaction-object)[] | 사용자가 하나 이상의 작업을 수행할 수 있도록 하는 **CardAction** 개체의 배열입니다. 채널에서 지정할 수 있는 단추의 수를 결정합니다. |
| **duration** | 문자열 | [ISO 8601 기간 형식](https://www.iso.org/iso-8601-date-and-time-format.html)의 미디어 콘텐츠 길이입니다. |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | 카드에 표시할 이미지를 지정하는 **ThumbnailUrl** 개체입니다. |
| **media** | [MediaUrl](#mediaurl-object)[] | 재생할 애니메이션 GIF의 목록을 지정하는 **MediaUrl** 개체의 배열입니다. |
| **shareable** | 부울 | 애니메이션을 다른 사용자와 공유할 수 있는지 여부를 나타내는 플래그입니다. 애니메이션을 공유할 수 있으면 이 속성을 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **true**입니다. |
| **subtitle** | 문자열 | 카드의 제목 아래에 표시할 자막입니다. |
| **text** | 문자열 | 카드의 제목 또는 자막 아래에 표시할 설명 또는 프롬프트입니다. |
| **title** | 문자열 | 카드의 제목입니다. |
| **값** | object | 이 카드에 대한 보조 매개 변수입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="attachment-object"></a>Attachment 개체
메시지에 포함할 추가 정보를 정의합니다. 첨부 파일은 미디어 파일(예: 오디오, 비디오, 이미지, 파일) 또는 서식 있는 카드일 수 있습니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **contentType** | 문자열 | 첨부 파일에 있는 콘텐츠의 미디어 유형입니다. 미디어 파일의 경우 이 속성을 **image/png**, **audio/wav** 및 **video/mp4**와 같이 알려진 미디어 유형으로 설정합니다. 서식 있는 카드의 경우 이 속성을 공급업체 관련 유형 중 하나로 설정합니다.<ul><li>**application/vnd.microsoft.card.adaptive**: 텍스트, 음성, 이미지, 단추 및 입력 필드의 조합을 포함할 수 있는 서식 있는 카드입니다. **content** 속성을 <a href="http://adaptivecards.io/documentation/#create-cardschema" target="_blank">AdaptiveCard</a> 개체로 설정합니다.</li><li>**application/vnd.microsoft.card.animation**: 애니메이션을 재생하는 서식 있는 카드입니다. **content** 속성을 [AnimationCard](#animationcard-object) 개체로 설정합니다.</li><li>**application/vnd.microsoft.card.audio**: 오디오 파일을 재생하는 서식 있는 카드입니다. **content** 속성을 [AudioCard](#audiocard-object) 개체로 설정합니다.</li><li>**application/vnd.microsoft.card.video**: 비디오를 재생하는 서식 있는 카드입니다. **content** 속성을 [VideoCard](#videocard-object) 개체로 설정합니다.</li><li>**application/vnd.microsoft.card.hero**: 영웅 카드입니다. **content** 속성을 [HeroCard](#herocard-object) 개체로 설정합니다.</li><li>**application/vnd.microsoft.card.thumbnail**: 썸네일 카드입니다. **content** 속성을 [ThumbnailCard](#thumbnailcard-object) 개체로 설정합니다.</li><li>**application/vnd.microsoft.com.card.receipt**: 수신 카드입니다. **content** 속성을 [ReceiptCard](#receiptcard-object) 개체로 설정합니다.</li><li>**application/vnd.microsoft.com.card.signin**: 사용자 로그인 카드입니다. **content** 속성을 [SignInCard](#signincard-object) 개체로 설정합니다.</li></ul> |
| **contentUrl** | 문자열 | 첨부 파일의 콘텐츠에 대한 URL입니다. 예를 들어 첨부 파일이 이미지인 경우 **contentUrl**을 이미지의 위치를 나타내는 URL로 설정합니다. 지원되는 프로토콜은 HTTP, HTTPS, 파일 및 데이터입니다. |
| **content** | object | 첨부 파일의 콘텐츠입니다. 첨부 파일이 서식 있는 카드인 경우 이 속성을 서식 있는 카드 개체로 설정합니다. 이 속성과 **contentUrl** 속성은 상호 배타적입니다. |
| **name** | 문자열 | 첨부 파일의 이름입니다. |
| **thumbnailUrl** | 문자열 | 더 작은 다른 형태의 **content** 또는 **contentUrl**을 지원하는 경우 채널에서 사용할 수 있는 썸네일 이미지에 대한 URL입니다. 예를 들어 **contentType**을 **application/word**로 설정하고 **contentUrl**을 Word 문서의 위치로 설정하면 문서를 나타내는 썸네일 이미지를 포함시킬 수 있습니다. 채널에서 문서 대신 썸네일 이미지를 표시할 수 있습니다. 사용자가 이미지를 클릭하면 채널에서 해당 문서를 엽니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="attachmentdata-object"></a>AttachmentData 개체 
첨부 파일 데이터를 설명합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **name** | 문자열 | 첨부 파일의 이름입니다. |
| **originalBase64** | 문자열 | 첨부 파일의 콘텐츠입니다. |
| **thumbnailBase64** | 문자열 | 첨부 파일의 썸네일 콘텐츠입니다. |
| **type** | 문자열 | 첨부 파일의 콘텐츠 형식입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="attachmentinfo-object"></a>AttachmentInfo 개체
첨부 파일을 설명합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **name** | 문자열 | 첨부 파일의 이름입니다. |
| **type** | 문자열 | 첨부 파일의 ContentType입니다. |
| **뷰** | [AttachmentView](#attachmentview-object)[] | 첨부 파일에 사용할 수 있는 보기를 나타내는 **AttachmentView** 개체의 배열입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="attachmentview-object"></a>AttachmentView 개체
첨부 파일 보기를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **viewId** | 문자열 | 보기 ID입니다. |
| **크기** | number | 파일의 크기입니다. |

<a href="#objects">스키마 표로 이동</a>

<!-- TODO - can't find in swagger file -->
### <a name="attachmentupload-object"></a>AttachmentUpload 개체
업로드할 첨부 파일을 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **type** | 문자열 | 첨부 파일의 ContentType입니다. | 
| **name** | 문자열 | 첨부 파일의 이름입니다. | 
| **originalBase64** | 문자열 | 파일의 원래 버전에 대한 콘텐츠를 나타내는 이진 데이터입니다. |
| **thumbnailBase64** | 문자열 | 파일의 썸네일 버전에 대한 콘텐츠를 나타내는 이진 데이터입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="audiocard-object"></a>AudioCard 개체
오디오 파일을 재생할 수 있는 카드를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **aspect** | 문자열 | **image** 속성에 지정한 썸네일의 가로 세로 비율입니다. 유효한 값은 **16:9** 및 **9:16**입니다. |
| **autoloop** | 부울 | 마지막 오디오 파일이 끝날 때 해당 오디오 파일 목록을 재생할지 여부를 나타내는 플래그입니다. 오디오 파일을 자동으로 재생하려면 이 속성을 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **true**입니다. |
| **autostart** | 부울 | 카드가 표시될 때 오디오를 자동으로 재생할지 여부를 나타내는 플래그입니다. 오디오를 자동으로 재생하려면 이 속성을 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **true**입니다. |
| **buttons** | [CardAction](#cardaction-object)[] | 사용자가 하나 이상의 작업을 수행할 수 있도록 하는 **CardAction** 개체의 배열입니다. 채널에서 지정할 수 있는 단추의 수를 결정합니다. |
| **duration** | 문자열 | [ISO 8601 기간 형식](https://www.iso.org/iso-8601-date-and-time-format.html)의 미디어 콘텐츠 길이입니다. |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | 카드에 표시할 이미지를 지정하는 **ThumbnailUrl** 개체입니다. |
| **media** | [MediaUrl](#mediaurl-object)[] | 재생할 오디오 파일의 목록을 지정하는 **MediaUrl** 개체의 배열입니다. |
| **shareable** | 부울 | 오디오 파일을 다른 사용자와 공유할 수 있는지 여부를 나타내는 플래그입니다. 오디오를 공유할 수 있으면 이 속성을 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **true**입니다. |
| **subtitle** | 문자열 | 카드의 제목 아래에 표시할 자막입니다. |
| **text** | 문자열 | 카드의 제목 또는 자막 아래에 표시할 설명 또는 프롬프트입니다. |
| **title** | 문자열 | 카드의 제목입니다. |
| **값** | object | 이 카드에 대한 보조 매개 변수입니다. |

<a href="#objects">스키마 표로 이동</a>

<!-- TODO - can't find in swagger file -->
### <a name="botdata-object"></a>BotData 개체
Bot State 서비스를 사용하여 저장된 사용자, 대화 또는 특정 대화의 컨텍스트에 있는 사용자에 대한 상태 데이터를 정의합니다.<br/><br/>

| 자산 | Type | 설명 |
|----|----|----|
| **데이터** | object | 요청에서 Bot State 서비스를 사용하여 저장할 속성과 값을 지정하는 JSON 개체입니다. 응답에서 Bot State 서비스를 사용하여 저장한 속성과 값을 지정하는 JSON 개체입니다. | 
| **eTag** | 문자열 | Bot State 서비스를 사용하여 저장하는 데이터에 대한 데이터 동시성을 제어하는 데 사용할 수 있는 엔터티 태그 값입니다. 자세한 내용은 [상태 데이터 관리](bot-framework-rest-state.md)를 참조하세요. | 

<a href="#objects">스키마 표로 이동</a>

### <a name="cardaction-object"></a>CardAction 개체
수행할 작업을 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **image** | 문자열 | 표시할 이미지에 대한 URL입니다. | 
| **text** | 문자열 | 작업에 대한 텍스트입니다. |
| **title** | 문자열 | 단추에 대한 텍스트입니다. 단추의 작업에만 적용됩니다. |
단추를 선택합니다. 단추의 작업에만 적용됩니다. |
| **type** | 문자열 | 수행할 작업의 유형입니다. 유효한 값 목록은 [메시지에 서식 있는 카드 첨부 파일 추가](bot-framework-rest-connector-add-rich-cards.md)를 참조하세요. |
| **값** | object | 작업에 대한 보조 매개 변수입니다. 이 속성의 값은 **type** 작업에 따라 다릅니다. 자세한 내용은 [메시지에 서식 있는 카드 첨부 파일 추가](bot-framework-rest-connector-add-rich-cards.md)를 참조하세요. |

<a href="#objects">스키마 표로 이동</a>

### <a name="cardimage-object"></a>CardImage 개체
카드에 표시할 이미지를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **alt** | 문자열 | 이미지에 대한 설명입니다. 접근성을 지원하는 설명이 포함되어야 합니다. |
| **tap** | [CardAction](#cardaction-object) | 사용자가 이미지를 탭하거나 클릭하면 수행할 작업을 지정하는 **CardAction** 개체입니다. |
| **url** | 문자열 | 이미지 원본 또는 이미지의 base64 이진 파일에 대한 URL입니다(예: `data:image/png;base64,iVBORw0KGgo...`). |

<a href="#objects">스키마 표로 이동</a>

### <a name="channelaccount-object"></a>ChannelAccount 개체
채널의 봇 또는 사용자 계정을 정의합니다.<br/><br/>

| 자산 | Type | 설명 |
|----|----|----|
| **id** | 문자열 | 채널의 봇 또는 사용자를 고유하게 식별하는 ID입니다. |
| **name** | 문자열 | 봇 또는 사용자에 대한 이름입니다. |

<a href="#objects">스키마 표로 이동</a>

<!--TODO can't find-->
### <a name="conversation-object"></a>Conversation 개체
대화 내에 포함된 봇 및 사용자를 포함한 대화를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **bot** | [ChannelAccount](#channelaccount-object) | 봇을 식별하는 **ChannelAccount** 개체입니다. |
| **isGroup** | 부울 | 그룹 대화인지 여부를 나타내는 플래그입니다. 그룹 대화이면 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **false**입니다. 그룹 대화를 시작하려면 채널에서 그룹 대화를 지원해야 합니다. |
| **members** | [ChannelAccount](#channelaccount-object)[] | 대화 멤버를 식별하는 **ChannelAccount** 개체의 배열입니다. **isGroup**이 **true**로 설정되지 않으면 이 목록에는 단일 사용자가 포함되어야 합니다. 이 목록에는 다른 봇이 포함될 수 있습니다. |
| **topicName** | 문자열 | 대화의 제목입니다. |
| **activity** | [활동](#activity-object) | [대화 만들기](#create-conversation) 요청에서 새 대화에 게시할 첫 번째 메시지를 정의하는 **Activity** 개체입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="conversationaccount-object"></a>ConversationAccount 개체
채널의 대화를 정의합니다.<br/><br/>

| 자산 | Type | 설명 |
|----|----|----|
| **id** | 문자열 | 대화를 식별하는 ID입니다. ID는 채널별로 고유합니다. 채널에서 대화를 시작하는 경우 이 ID를 설정합니다. 그러지 않으면 봇에서 대화를 시작할 때 응답으로 반환되는 ID로 이 속성을 설정합니다(‘대화 시작’ 참조). |
| **isGroup** | 부울 | 활동이 생성된 시점에 세 명 이상의 참가자가 대화에 포함되어 있는지 여부를 나타내는 플래그입니다. 그룹 대화이면 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **false**입니다. |
| **name** | 문자열 | 대화를 식별하는 데 사용할 수 있는 표시 이름입니다. |
| **conversationType** | 문자열 | 대화 유형을 구분하는 채널의 대화 유형을 나타냅니다(예: 그룹, 개인). |

<a href="#objects">스키마 표로 이동</a>

### <a name="conversationmembers-object"></a>ConversationMembers 개체
대화의 멤버를 정의합니다.<br/><br/>

| 자산 | Type | 설명 |
|----|----|----|
| **id** | 문자열 | 대화 ID입니다. |
| **members** | array | [ChannelAccount](#channelaccount-object) 개체 배열입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="conversationparameters-object"></a>ConversationParameters 개체
새 대화를 만들기 위한 매개 변수를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **isGroup** | 부울 | 그룹 대화인지 여부를 나타냅니다. |
| **bot** | [ChannelAccount](#channelaccount-object) | 대화에 있는 봇의 주소입니다. |
| **members** | array | 대화에 추가할 멤버의 목록입니다. |
| **topicName** | 문자열 | 대화의 주제 제목입니다. 이 속성은 채널에서 지원하는 경우에만 사용됩니다. |
| **activity** | [활동](#activity-object) | (선택 사항) 새 대화를 만들 때 이 활동을 대화의 초기 메시지로 사용합니다. |
| **channelData** | object | 대화를 만들기 위한 채널 특정 페이로드입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="conversationreference-object"></a>ConversationReference 개체
대화의 특정 지점을 정의합니다.<br/><br/>

| 자산 | Type | 설명 |
|----|----|----|
| **activityId** | 문자열 | 이 개체에서 참조하는 활동을 고유하게 식별하는 ID입니다. | 
| **bot** | [ChannelAccount](#channelaccount-object) | 이 개체에서 참조하는 대화에서 봇을 식별하는 **ChannelAccount** 개체입니다. |
| **channelId** | 문자열 | 이 개체에서 참조하는 대화에서 채널을 고유하게 식별하는 ID입니다. | 
| **대화** | [ConversationAccount](#conversationaccount-object) | 이 개체에서 참조하는 대화를 정의하는 **ConversationAccount** 개체입니다. |
| **serviceUrl** | 문자열 | 이 개체에서 참조하는 대화에서 채널의 서비스 엔드포인트를 지정하는 URL입니다. | 
| **user** | [ChannelAccount](#channelaccount-object) | 이 개체에서 참조하는 대화에서 사용자를 식별하는 **ChannelAccount** 개체입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="conversationresourceresponse-object"></a>ConversationResourceResponse 개체
[대화 만들기](#create-conversation)에 대한 응답을 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **activityId** | 문자열 | 활동의 ID입니다. |
| **id** | 문자열 | 리소스의 ID입니다. |
| **serviceUrl** | 문자열 | 서비스 엔드포인트입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="conversationsresult-object"></a>ConversationsResult 개체
[대화 가져오기](#get-conversations)의 결과를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **continuationToken** | 문자열 | [대화 가져오기](#get-conversations) 후속 호출에서 사용할 수 있는 연속 토큰입니다. |
| **대화** | array | [ConversationMembers](#conversationmembers-object) 개체 배열입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="error-object"></a>Error 개체
오류를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **code** | 문자열 | 오류 코드 |
| **message** | 문자열 | 오류에 대한 설명입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="entity-object"></a>Entity 개체
엔터티 개체를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **type** | 문자열 | 엔터티 형식입니다. 일반적으로 schema.org의 형식을 포함합니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="errorresponse-object"></a>ErrorResponse 개체
HTTP API 응답을 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **error** | [오류](#error-object) | 오류에 대한 정보를 포함하는 **Error** 개체입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="fact-object"></a>Fact 개체
팩트가 포함된 키-값 쌍을 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **key** | 문자열 | 팩트의 이름입니다. 예: **체크 인**. 키는 팩트 값을 표시할 때 레이블로 사용됩니다. |
| **값** | 문자열 | 팩트의 값입니다. 예: **2016년 10월 10일** |

<a href="#objects">스키마 표로 이동</a>

### <a name="geocoordinates-object"></a>GeoCoordinates 개체
World Geodetic System(WSG84) 좌표를 사용하여 지리적 위치를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **elevation** | number | 위치의 고도입니다. |
| **name** | 문자열 | 위치의 이름입니다. |
| **latitude** | number | 위치의 위도입니다. |
| **longitude** | number | 위치의 경도입니다. |
| **type** | 문자열 | 이 개체의 형식입니다. 항상 **GeoCoordinates**로 설정합니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="herocard-object"></a>HeroCard 개체
큰 이미지, 제목, 텍스트 및 작업 단추가 있는 카드를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 사용자가 하나 이상의 작업을 수행할 수 있도록 하는 **CardAction** 개체의 배열입니다. 채널에서 지정할 수 있는 단추의 수를 결정합니다. |
| **images** | [CardImage](#cardimage-object)[] | 카드에 표시할 이미지를 지정하는 **CardImage** 개체의 배열입니다. 영웅 카드에는 하나의 이미지만 포함됩니다. |
| **subtitle** | 문자열 | 카드의 제목 아래에 표시할 자막입니다. |
| **tap** | [CardAction](#cardaction-object) | 사용자가 카드를 탭하거나 클릭하면 수행할 작업을 지정하는 **CardAction** 개체입니다. 단추 중 하나와 동일한 작업이거나 다른 작업일 수 있습니다. |
| **text** | 문자열 | 카드의 제목 또는 자막 아래에 표시할 설명 또는 프롬프트입니다. |
| **title** | 문자열 | 카드의 제목입니다. |

<a href="#objects">스키마 표로 이동</a>

<!--TODO can't find-->
### <a name="identification-object"></a>Identification 개체
리소스를 식별합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **id** | 문자열 | 리소스를 고유하게 식별하는 ID입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="mediaeventvalue-object"></a>MediaEventValue 개체 
미디어 이벤트에 대한 보조 매개 변수입니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **cardValue** | object | 이 이벤트를 발생시킨 미디어 카드의 **Value** 필드에 지정한 콜백 매개 변수입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="mediaurl-object"></a>MediaUrl 개체
미디어 파일의 원본에 대한 URL을 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **profile** | 문자열 | 미디어의 콘텐츠를 설명하는 힌트입니다. |
| **url** | 문자열 | 미디어 파일의 원본에 대한 URL입니다. |

<a href="#objects">스키마 표로 이동</a>

<!--TODO can't find-->
### <a name="mention-object"></a>Mention 개체
대화에서 언급된 사용자 또는 봇을 정의합니다.<br/><br/> 


|          자산          |                   Type                   |                                                                                                                                                                                                                           설명                                                                                                                                                                                                                            |
|----------------------------|------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <strong>mentioned</strong> | [ChannelAccount](#channelaccount-object) | 언급된 사용자 또는 봇을 지정하는 <strong>ChannelAccount</strong> 개체입니다. Slack과 같은 일부 채널에서는 대화별로 이름을 할당하므로 메시지의 <strong>recipient</strong> 속성에서 언급된 봇의 이름이 봇을 [등록](../bot-service-quickstart-registration.md)했을 때 지정한 핸들과 다를 수도 있습니다. 그러나 둘 다에 대한 계정 ID는 동일합니다. |
|   <strong>text</strong>    |                  문자열                  |                                                                                                                         대화에서 언급된 사용자 또는 봇입니다. 예를 들어 메시지가 "@ColorBot에서 나에게 새로운 색을 지정합니다."인 경우 이 속성은 <strong>@ColorBot</strong>으로 설정됩니다. 일부 채널에서는 이 속성을 설정하지 않습니다.                                                                                                                          |
|   <strong>type</strong>    |                  문자열                  |                                                                                                                                                                                                   이 개체의 형식입니다. 항상 <strong>Mention</strong>으로 설정합니다.                                                                                                                                                                                                    |

<a href="#objects">스키마 표로 이동</a>

### <a name="messagereaction-object"></a>MessageReaction 개체
메시지에 대한 반응을 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **type** | 문자열 | 반응의 형식입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="place-object"></a>Place 개체
대화에서 언급된 위치를 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **address** | object |  위치의 주소입니다. 이 속성은 `string`이거나 `PostalAddress` 형식의 복합 개체일 수 있습니다. |
| **geo** | [GeoCoordinates](#geocoordinates-object) | 위치의 지리적 좌표를 지정하는 **GeoCoordinates** 개체입니다. |
| **hasMap** | object | 위치에 매핑됩니다. 이 속성은 `string`(URL) 또는 `Map` 형식의 복합 개체일 수 있습니다. |
| **name** | 문자열 | 위치의 이름입니다. |
| **type** | 문자열 | 이 개체의 형식입니다. 항상 **Place**로 설정합니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="receiptcard-object"></a>ReceiptCard 개체
구매 영수증이 포함된 카드를 정의합니다.<br/><br/>

| 자산 | Type | 설명 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 사용자가 하나 이상의 작업을 수행할 수 있도록 하는 **CardAction** 개체의 배열입니다. 채널에서 지정할 수 있는 단추의 수를 결정합니다. |
| **facts** | [Fact](#fact-object)[] | 구매에 대한 정보를 지정하는 **Fact** 개체의 배열입니다. 예를 들어 호텔 투숙 영수증에 대한 팩트 목록에는 체크 인 날짜와 체크 아웃 날짜가 포함될 수 있습니다. 채널에서 지정할 수 있는 팩트의 수를 결정합니다. |
| **items** | [ReceiptItem](#receiptitem-object)[] | 구입한 항목을 지정하는 **ReceiptItem** 개체의 배열입니다. |
| **tap** | [CardAction](#cardaction-object) | 사용자가 카드를 탭하거나 클릭하면 수행할 작업을 지정하는 **CardAction** 개체입니다. 단추 중 하나와 동일한 작업이거나 다른 작업일 수 있습니다. |
| **tax** | 문자열 | 구매에 적용되는 세금의 금액을 지정하는 통화 형식 문자열입니다. |
| **title** | 문자열 | 영수증 위쪽에 표시되는 제목입니다. |
| **total** | 문자열 | 적용되는 모든 세금을 포함하여 총 구매 가격을 지정하는 통화 형식 문자열입니다. |
| **vat** | 문자열 | 구매 가격에 적용되는 VAT(부가가치세)의 금액을 지정하는 통화 형식 문자열입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="receiptitem-object"></a>ReceiptItem 개체
영수증 내의 품목을 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **image** | [CardImage](#cardimage-object) | 품목 옆에 표시할 썸네일 이미지를 지정하는 **CardImage** 개체입니다.  |
| **price** | 문자열 | 구입한 모든 단위의 총 가격을 지정하는 통화 형식 문자열입니다. |
| **quantity** | 문자열 | 구입한 단위 수를 지정하는 숫자 문자열입니다. |
| **subtitle** | 문자열 | 품목의 제목 아래에 표시할 자막입니다. |
| **tap** | [CardAction](#cardaction-object) | 사용자가 품목을 탭하거나 클릭하면 수행할 작업을 지정하는 **CardAction** 개체입니다. |
| **text** | 문자열 | 품목에 대한 설명입니다. |
| **title** | 문자열 | 품목에 대한 제목입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="resourceresponse-object"></a>ResourceResponse 개체
리소스 ID를 포함하는 응답을 정의합니다.<br/><br/>

|      자산       |  Type  |                설명                |
|---------------------|--------|-------------------------------------------|
| <strong>id</strong> | 문자열 | 리소스를 고유하게 식별하는 ID입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="semanticaction-object"></a>SemanticAction 개체
프로그래밍 방식 작업에 대한 참조를 정의합니다.<br/><br/>

| 자산 | Type | 설명 |
|----|----|----|
| **id** | 문자열 | 이 작업 ID |
| **entities** | [엔터티](#entity-object) | 이 작업과 연결된 엔터티 |

<a href="#objects">스키마 표로 이동</a>

### <a name="signincard-object"></a>SignInCard 개체
사용자가 서비스에 로그인할 수 있도록 하는 카드를 정의합니다.<br/><br/>

| 자산 | Type | 설명 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 사용자가 서비스에 로그인할 수 있도록 하는 **CardAction** 개체의 배열입니다. 채널에서 지정할 수 있는 단추의 수를 결정합니다. |
| **text** | 문자열 | 로그인 카드에 포함할 설명 또는 프롬프트입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="suggestedactions-object"></a>SuggestedActions 개체
사용자가 선택할 수 있는 옵션을 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **actions** | [CardAction](#cardaction-object)[] | 제안된 작업을 정의하는 **CardAction** 개체의 배열입니다. |
| **to** | string[] | 제안된 작업이 표시되어야 하는 받는 사람의 ID를 포함하는 문자열의 배열입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="thumbnailcard-object"></a>ThumbnailCard 개체
썸네일 이미지, 제목, 텍스트 및 작업 단추가 있는 카드를 정의합니다.<br/><br/>

| 자산 | Type | 설명 |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 사용자가 하나 이상의 작업을 수행할 수 있도록 하는 **CardAction** 개체의 배열입니다. 채널에서 지정할 수 있는 단추의 수를 결정합니다. |
| **images** | [CardImage](#cardimage-object)[] | 카드에 표시할 썸네일 이미지를 지정하는 **CardImage** 개체의 배열입니다. 채널에서 지정할 수 있는 썸네일 이미지의 수를 결정합니다. |
| **subtitle** | 문자열 | 카드의 제목 아래에 표시할 자막입니다. |
| **tap** | [CardAction](#cardaction-object) | 사용자가 카드를 탭하거나 클릭하면 수행할 작업을 지정하는 **CardAction** 개체입니다. 단추 중 하나와 동일한 작업이거나 다른 작업일 수 있습니다. |
| **text** | 문자열 | 카드의 제목 또는 자막 아래에 표시할 설명 또는 프롬프트입니다. |
| **title** | 문자열 | 카드의 제목입니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="thumbnailurl-object"></a>ThumbnailUrl 개체
이미지 원본에 대한 URL을 정의합니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **alt** | 문자열 | 이미지에 대한 설명입니다. 접근성을 지원하는 설명이 포함되어야 합니다. |
| **url** | 문자열 | 이미지 원본 또는 이미지의 base64 이진 파일에 대한 URL입니다(예: `data:image/png;base64,iVBORw0KGgo...`). |

<a href="#objects">스키마 표로 이동</a>

### <a name="transcript-object"></a>Transcript 개체
[대화 기록 보내기](#send-conversation-history)를 사용하여 업로드할 활동 컬렉션입니다.<br/><br/> 

| 자산 | Type | 설명 |
|----|----|----|
| **activities** | array | [Activity](#activity-object) 개체 배열입니다. 각 개체에 고유 ID와 타임스탬프가 있어야 합니다. |

<a href="#objects">스키마 표로 이동</a>

### <a name="videocard-object"></a>VideoCard 개체
비디오를 재생할 수 있는 카드를 정의합니다.<br/><br/>

| 자산 | Type | 설명 |
|----|----|----|
| **aspect** | 문자열 | 비디오의 가로 세로 비율입니다(예: 16:9, 4:3).|
| **autoloop** | 부울 | 마지막 비디오 파일이 끝날 때 해당 비디오 파일 목록을 재생할지 여부를 나타내는 플래그입니다. 비디오를 자동으로 다시 재생하려면 이 속성을 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **true**입니다. |
| **autostart** | 부울 | 카드가 표시될 때 비디오를 자동으로 재생할지 여부를 나타내는 플래그입니다. 비디오를 자동으로 재생하려면 이 속성을 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **true**입니다. |
| **buttons** | [CardAction](#cardaction-object)[] | 사용자가 하나 이상의 작업을 수행할 수 있도록 하는 **CardAction** 개체의 배열입니다. 채널에서 지정할 수 있는 단추의 수를 결정합니다. |
| **duration** | 문자열 | [ISO 8601 기간 형식](https://www.iso.org/iso-8601-date-and-time-format.html)의 미디어 콘텐츠 길이입니다. |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | 카드에 표시할 이미지를 지정하는 **ThumbnailUrl** 개체입니다. |
| **media** | [MediaUrl](#mediaurl-object)[] | 재생할 비디오의 목록을 지정하는 **MediaUrl** 개체의 배열입니다. |
| **shareable** | 부울 | 비디오를 다른 사용자와 공유할 수 있는지 여부를 나타내는 플래그입니다. 비디오를 공유할 수 있으면 이 속성을 **true**로 설정하고, 그렇지 않으면 **false**로 설정합니다. 기본값은 **true**입니다. |
| **subtitle** | 문자열 | 카드의 제목 아래에 표시할 자막입니다. |
| **text** | 문자열 | 카드의 제목 또는 자막 아래에 표시할 설명 또는 프롬프트입니다. |
| **title** | 문자열 | 카드의 제목입니다. |
| **값** | object | 이 카드에 대한 보조 매개 변수입니다.|

<a href="#objects">스키마 표로 이동</a>
