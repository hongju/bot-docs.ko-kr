---
title: 상태 데이터 관리 | Microsoft Docs
description: Bot State 서비스를 사용하여 상태 데이터를 저장 및 검색하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: b0d5ca6893d70a73bc005a949ef6cc2518d3862f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000030"
---
# <a name="manage-state-data"></a>상태 데이터 관리

Bot State 서비스를 사용하면 사용자, 대화 또는 특정 대화 컨텍스트 내의 특정 사용자와 연결된 상태 데이터를 봇이 저장하고 검색할 수 있습니다. 채널에 있는 각 사용자, 채널에 있는 각 대화 및 채널에 있는 대화 컨텍스트 내의 각 사용자에 대한 최대 32KB의 데이터를 저장할 수 있습니다. 상태 데이터는 이전 대화를 중단한 위치를 확인하거나 단순히 돌아온 사용자에게 이름으로 인사말을 제공하는 등의 다양한 용도로 사용할 수 있습니다. 사용자의 기본 설정을 저장하면 이 정보를 사용하여 다음에 채팅할 때 대화를 사용자 지정할 수 있습니다. 예를 들어, 관심 있는 주제에 대한 뉴스 기사를 사용자에게 알리거나 약속이 가능해질 때 사용자에게 알릴 수 있습니다. 

> [!IMPORTANT]
> Bot Framework 상태 서비스 API는 프로덕션 환경에는 권장되지 않으며 이후 릴리스에서 사용되지 않을 수 있습니다. 테스트 용도로 메모리 내 저장소를 사용하거나 프로덕션 봇용으로 **Azure 확장**을 사용하도록 봇 코드를 업데이트하는 것이 좋습니다. 자세한 내용은 [.NET](~/dotnet/bot-builder-dotnet-state.md) 또는 [Node](~/nodejs/bot-builder-nodejs-state.md) 구현에 대한 **상태 데이터 관리** 항목을 참조하세요.

## <a id="concurrency"></a> 데이터 동시성

Bot State 서비스를 사용하여 저장된 데이터의 동시성을 제어하기 위해 프레임워크는 `POST` 요청에 엔터티 태그(ETag)를 사용합니다. 프레임워크는 ETag에 대한 표준 헤더를 사용하지 않습니다. 대신에 요청 및 응답 본문에서 `eTag` 속성을 사용하여 ETag를 지정합니다. 

예를 들어, `GET` 요청을 실행하여 저장소에서 사용자 데이터를 검색할 경우 응답에는 `eTag` 속성이 포함됩니다. 데이터를 변경하고 `POST` 요청을 실행하여 업데이트된 데이터를 저장소에 저장할 경우, 요청에는 `GET` 응답에서 이전에 받은 동일한 값을 지정하는 `eTag` 속성이 포함될 수 있습니다. `POST` 요청에 지정된 ETag가 저장소의 현재 값과 일치할 경우 서버는 사용자 데이터를 저장하고, **HTTP 200 성공**으로 응답하고, 응답 본문에 새 `eTag` 값을 지정합니다. `POST` 요청에 지정된 ETag가 저장소의 현재 값과 일치하지 않을 경우, 서버는 **HTTP 412 사전 조건 실패**로 응답하여 사용자 데이터를 마지막으로 저장하거나 검색한 후 저장소의 사용자 데이터가 변경되었음을 나타냅니다.

> [!TIP]
> `GET` 응답에는 항상 `eTag` 속성이 포함되지만 `GET` 요청에 `eTag` 속성을 지정할 필요는 없습니다. 별표(`*`)의 `eTag` 속성 값은 채널, 사용자 및 대화의 지정된 조합에 해당하는 데이터를 이전에 저장한 적이 없음을 나타냅니다.
>
> `POST` 요청에 `eTag` 속성을 지정하는 것은 선택 사항입니다. 
> 동시성이 봇에 중요하지 않으면 `POST` 요청에 `eTag` 속성을 포함하지 마세요. 

## <a id="save-user-data"></a> 사용자 데이터 저장

특정 채널 사용자의 상태 데이터를 저장하려면 다음 요청을 실행합니다.

```http
POST /v3/botstate/{channelId}/users/{userId}
```

이 요청 URI에서 **{channelId}** 를 채널 ID로 바꾸고 **{userId}** 를 채널의 사용자 ID로 바꿉니다. 봇이 이전에 사용자로부터 받은 모든 메시지 내의 `channelId` 및 `from` 속성에 이러한 ID가 포함됩니다. 메시지에서 사용자 데이터를 추출할 필요 없이 나중에 사용자 데이터에 액세스할 수 있도록 안전한 위치에 이러한 값을 캐시할 수 있습니다.

요청 본문을 [BotData][BotData] 개체로 설정합니다. 여기서 `data` 속성은 저장하려는 데이터를 지정합니다. [동시성 컨트롤](#concurrency)에 엔터티 태그를 사용할 경우 `eTag` 속성을, 사용자 데이터를 마지막으로 저장하거나 검색할 때 응답으로 받은 ETag로 설정합니다(어떤 것이든 가장 최근 항목). 동시성에 엔터티 태그를 사용하지 않을 경우에는 요청에 `eTag` 속성을 포함하지 마세요.

> [!NOTE]
> 이 `POST` 요청은 ETag가 서버의 ETag와 일치하거나 요청에 ETag가 지정되지 않은 경우에만 저장소에서 사용자 데이터를 덮어씁니다.

### <a name="request"></a>요청 

다음 예제에서는 특정 채널의 사용자에 대한 데이터를 저장하는 요청을 보여 줍니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

```http
POST https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "a1b2c3d4"
}
```

### <a name="response"></a>response 

응답에는 [BotData][BotData] 개체가 새 `eTag` 값과 함께 포함됩니다.

## <a name="get-user-data"></a>사용자 데이터 가져오기

특정 채널의 사용자에 대한 이전에 저장된 상태 데이터를 가져오려면 이 요청을 실행합니다.

```http
GET /v3/botstate/{channelId}/users/{userId}
```

이 요청 URI에서 **{channelId}** 를 채널 ID로 바꾸고 **{userId}** 를 채널의 사용자 ID로 바꿉니다. 봇이 이전에 사용자로부터 받은 모든 메시지 내의 `channelId` 및 `from` 속성에 이러한 ID가 포함됩니다. 메시지에서 사용자 데이터를 추출할 필요 없이 나중에 사용자 데이터에 액세스할 수 있도록 안전한 위치에 이러한 값을 캐시할 수 있습니다.

### <a name="request"></a>요청

다음 예제에서는 사용자에 대해 이전에 저장된 데이터를 가져오는 요청을 보여 줍니다. 이 예제 요청에서 `https://smba.trafficmanager.net/apis`는 기본 URI를 나타냅니다. 봇이 실행하는 요청의 기본 URI는 다를 수 있습니다. 기본 URI를 설정하는 방법에 대한 자세한 내용은 [API 참조](bot-framework-rest-connector-api-reference.md#base-uri)를 참조하세요.

```http
GET https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

### <a name="response"></a>response

응답에는 [BotData][BotData] 개체가 포함됩니다. 여기서 `data` 속성에는 사용자에 대한 이전에 저장된 데이터가 포함되고 `eTag` 속성에는 후속 요청 내에서 사용자 데이터를 저장하는 데 사용할 수 있는 ETag가 포함됩니다. 사용자 데이터를 이전에 저장하지 않은 경우에는 `data` 속성이 `null`이고 `eTag` 속성에 별표(`*`)가 포함됩니다.

이 예제는 `GET` 요청에 대한 응답을 보여 줍니다.

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "xyz12345"
}
```

## <a name="save-conversation-data"></a>대화 데이터 저장

특정 채널의 대화에 대한 상태 데이터를 저장하려면 다음 요청을 실행합니다.

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

이 요청 URI에서 **{channelId}** 를 채널 ID로 바꾸고 **{conversationId}** 를 대화 ID로 바꿉니다. 봇이 이전에 대화 컨텍스트에서 보내거나 받은 모든 메시지 내의 `channelId` 및 `conversation` 속성에 이러한 ID가 포함됩니다. 메시지에서 대화 데이터를 추출할 필요 없이 나중에 대화 데이터에 액세스할 수 있도록 안전한 위치에 이러한 값을 캐시할 수 있습니다.

[사용자 데이터 저장](#save-user-data)에 설명된 대로 요청 본문을 [BotData][BotData] 개체로 설정합니다.

> [!IMPORTANT]
> [사용자 데이터 삭제](#delete-user-data) 작업으로는 **대화 데이터 저장** 작업을 사용하여 저장된 데이터가 삭제되지 않으므로 사용자의 PII(개인 식별 정보)를 저장하는 데는 이 작업을 사용하면 안 됩니다.

### <a name="response"></a>response

응답에는 [BotData][BotData] 개체가 새 `eTag` 값과 함께 포함됩니다.

## <a name="get-conversation-data"></a>대화 데이터 가져오기

특정 채널의 대화에 대한 이전에 저장된 상태 데이터를 가져오려면 이 요청을 실행합니다.

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}
```

이 요청 URI에서 **{channelId}** 를 채널 ID로 바꾸고 **{conversationId}** 를 대화 ID로 바꿉니다. 봇이 이전에 대화 컨텍스트에서 보내거나 받은 모든 메시지 내의 `channelId` 및 `conversation` 속성에 이러한 ID가 포함됩니다. 메시지에서 대화 데이터를 추출할 필요 없이 나중에 대화 데이터에 액세스할 수 있도록 안전한 위치에 이러한 값을 캐시할 수 있습니다.

### <a name="response"></a>response

응답에는 [BotData][BotData] 개체가 새 `eTag` 값과 함께 포함됩니다.

## <a name="save-private-conversation-data"></a>프라이빗 대화 데이터 저장

특정 대화의 컨텍스트 내에서 사용자의 상태 데이터를 저장하려면 다음 요청을 실행합니다.

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

이 요청 URI에서 **{channelId}** 를 채널 ID로 바꾸고, **{conversationId}** 를 대화 ID로 바꾸고, **{userId}** 를 해당 채널의 사용자 ID로 바꿉니다. 봇이 이전에 대화 컨텍스트에서 사용자로부터 받은 모든 메시지 내의 `channelId`, `conversation` 및 `from` 속성에 이러한 ID가 포함됩니다. 메시지에서 대화 데이터를 추출할 필요 없이 나중에 대화 데이터에 액세스할 수 있도록 안전한 위치에 이러한 값을 캐시할 수 있습니다.

[사용자 데이터 저장](#save-user-data)에 설명된 대로 요청 본문을 [BotData][BotData] 개체로 설정합니다.

### <a name="response"></a>response

응답에는 [BotData][BotData] 개체가 새 `eTag` 값과 함께 포함됩니다.

## <a name="get-private-conversation-data"></a>프라이빗 대화 데이터 가져오기

특정 채널의 컨텍스트 내에서 사용자에 대한 이전에 저장된 상태 데이터를 가져오려면 다음 요청을 실행합니다. 

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

이 요청 URI에서 **{channelId}** 를 채널 ID로 바꾸고, **{conversationId}** 를 대화 ID로 바꾸고, **{userId}** 를 해당 채널의 사용자 ID로 바꿉니다. 봇이 이전에 대화 컨텍스트에서 사용자로부터 받은 모든 메시지 내의 `channelId`, `conversation` 및 `from` 속성에 이러한 ID가 포함됩니다. 메시지에서 대화 데이터를 추출할 필요 없이 나중에 대화 데이터에 액세스할 수 있도록 안전한 위치에 이러한 값을 캐시할 수 있습니다.

### <a name="response"></a>response

응답에는 [BotData][BotData] 개체가 새 `eTag` 값과 함께 포함됩니다.

## <a name="delete-user-data"></a>사용자 데이터 삭제

특정 채널 사용자의 상태 데이터를 삭제하려면 다음 요청을 실행합니다.

```http
DELETE /v3/botstate/{channelId}/users/{userId}
```
이 요청 URI에서 **{channelId}** 를 채널 ID로 바꾸고 **{userId}** 를 채널의 사용자 ID로 바꿉니다. 봇이 이전에 사용자로부터 받은 모든 메시지 내의 `channelId` 및 `from` 속성에 이러한 ID가 포함됩니다. 메시지에서 사용자 데이터를 추출할 필요 없이 나중에 사용자 데이터에 액세스할 수 있도록 안전한 위치에 이러한 값을 캐시할 수 있습니다.

> [!IMPORTANT]
> 이 작업은 [사용자 데이터 저장](#save-user-data) 작업 또는 [프라이빗 대화 데이터 저장](#save-private-conversation-data) 작업을 사용하여 이전에 저장된 사용자에 대한 데이터를 삭제합니다. [대화 데이터 저장](#save-conversation-data) 작업을 사용하여 이전에 저장된 데이터는 삭제되지 않습니다. 따라서 사용자의 PII(개인 식별 정보)를 저장하는 데는 **대화 데이터 저장** 작업을 사용하면 안 됩니다.

봇은 [deleteUserData](bot-framework-rest-connector-activities.md#deleteuserdata) 형식의 [활동][Activity]이나 사용자의 연락처 목록에서 봇이 제거되었음을 나타내는 [contactRelationUpdate](bot-framework-rest-connector-activities.md#contactrelationupdate) 형식의 활동을 수신할 경우 **사용자 데이터 삭제** 작업을 실행해야 합니다.

## <a name="additional-resources"></a>추가 리소스

- [주요 개념](bot-framework-rest-connector-concepts.md)
- [작업 개요](bot-framework-rest-connector-activities.md)

[BotData]: bot-framework-rest-connector-api-reference.md#botdata-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
