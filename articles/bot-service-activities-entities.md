---
title: 엔터티 및 작업 형식 | Microsoft Docs
description: 엔터티 및 작업 형식.
keywords: 멘션 엔터티, 작업 형식, 엔터티 사용
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/01/2018
ms.openlocfilehash: d2f4692580843f530641827707d250fa726830e3
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000400"
---
# <a name="entities-and-activity-types"></a>엔터티 및 작업 형식

엔터티는 작업의 일부이며 작업 또는 대화에 대한 추가 정보를 제공합니다.

[!include[Entity boilerplate](includes/snippet-entity-boilerplate.md)]

## <a name="entities"></a>엔터티

메시지의 *엔터티* 속성은 채널과 봇 간의 일반적인 상황별 메타데이터를 교환할 수 있는 개방형 <a href="http://schema.org/" target="_blank">schema.org</a> 개체의 배열입니다.

### <a name="mention-entities"></a>멘션 엔터티

다양한 채널은 봇 또는 사용자가 대화 컨텍스트 내에서 누군가를 "멘션"하기 위한 기능을 지원합니다.
메시지에서 사용자를 멘션하려면 메시지의 엔터티 속성을 *mention* 개체로 채웁니다.
멘션 개체는 다음 속성을 포함합니다.

| 자산 | 설명 |
|----|----|
| type | 엔터티의 형식("멘션") |
| 멘션됨 | 멘션된 사용자를 나타내는 채널 계정 개체 | 
| 텍스트 | 멘션 자체를 나타내는 *activity.text* 속성 내의 텍스트(비어 있거나 Null일 수 있음) |

이 코드 예제는 엔터티 컬렉션에 멘션 엔터티를 추가하는 방법을 보여줍니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set Mention](includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> 사용자 의도를 확인하려고 시도할 때 봇은 멘션된 메시지의 해당 부분을 무시하려고 할 수 있습니다. `GetMentions` 메서드를 호출하고 응답에서 반환된 `Mention` 개체를 평가합니다.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const mention = {
    type: "Mention",
    text: "@johndoe",
    mentioned: {
        name: "John Doe",
        id: "UV341235"
    }
}

entity = [mention];
```

---

### <a name="place-objects"></a>장소 개체

<a href="https://schema.org/Place" target="_blank">위치 관련 정보</a>는 메시지의 엔터티 속성을 *장소* 개체 또는 *GeoCoordinates* 개체 중 하나로 채워 메시지 내에서 전달될 수 있습니다.

장소 개체는 다음 속성을 포함합니다.

| 자산 | 설명 |
|----|----|
| type | 엔터티의 형식("Place") |
| 주소 | 설명 또는 우편 주소 개체(이후) |
| 지역 | GeoCoordinates |
| HasMap | 맵 또는 맵 개체에 대한 URL(이후) |
| 이름 | 장소의 이름 |

geoCoordinates 개체는 다음 속성을 포함합니다.

| 자산 | 설명 |
|----|----|
| type | 엔터티의 형식("GeoCoordinates") |
| 이름 | 장소의 이름 |
| 경도 | 위치의 경도(<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| 경도 | 위치의 위도(<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| 상승 | 위치의 상승(<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |

이 코드 예제는 엔터티 컬렉션에 장소 엔터티를 추가하는 방법을 보여줍니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set GeoCoordinates](includes/code/dotnet-create-messages.cs#setGeoCoord)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const place = {
    elavation: 100,
    type: "GeoCoordinates",
    name : "myPlace",
    latitude: 123,
    longitude: 234
};

entity = [place];

```

---

### <a name="consume-entities"></a>엔터티 사용

# <a name="ctabcs"></a>[C#](#tab/cs)

엔터티를 사용하려면 `dynamic` 키워드 또는 강력한 형식의 클래스 중 하나를 사용합니다.

이 코드 예제는 `dynamic` 키워드를 사용하여 메시지의 `Entities` 속성 내에서 엔터티를 처리하는 방법을 보여줍니다.

[!code-csharp[examine entity using dynamic keyword](includes/code/dotnet-create-messages.cs#examineEntity1)]

이 코드 예제는 강력한 형식의 클래스를 사용하여 메시지의 `Entities` 속성 내에서 엔터티를 처리하는 방법을 보여줍니다.

[!code-csharp[examine entity using typed class](includes/code/dotnet-create-messages.cs#examineEntity2)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

이 코드 예제는 메시지의 `entity` 속성 내에서 엔터티를 처리하는 방법을 보여줍니다.

```javascript
if (entity[0].type === "GeoCoordinates" && entity[0].latitude > 34) {
    // do something
}
```

---

## <a name="activity-types"></a>작업 유형

이 코드 예제에서는 **메시지** 형식의 작업을 처리하는 방법을 보여줍니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
if (context.Activity.Type == ActivityTypes.Message){
    // do something
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```js
if(context.activity.type === 'message'){
    // do something
}
```

---

작업은 몇 가지 다른 형식의 가장 일반적인 **메시지**일 수 있습니다. 몇 가지 작업 유형이 있습니다.

| Activity.Type | 인터페이스 | 설명 |
|-----|-----|-----|
| [message](#message) | IMessageActivity(C#) <br> 작업(JS) | 봇 및 사용자 간의 통신을 나타냅니다. |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity(C#) <br> 작업(JS) | 봇이 사용자의 연락처 목록에서 추가되거나 제거됐음을 나타냅니다. |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity(C#) <br> 작업(JS) | 봇이 대화에 추가됐으며, 다른 구성원이 대화에서 추가되거나 제거됐으며, 또는 대화 메타데이터가 변경되었음을 나타냅니다. |
| [deleteUserData](#deleteuserdata) | 해당 없음 | 사용자가 봇이 저장했을 수도 있는 모든 사용자 데이터를 봇에게 삭제하도록 요청했음을 나타냅니다. |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity(C#) <br> 작업(JS) | 대화의 끝을 나타냅니다. |
| [event](#event) | IEventActivity(C#) <br> 작업(JS) | 사용자에게 표시되지 않는 봇에게 전송한 통신을 나타냅니다. |
| [installationUpdate](#installationupdate) | IInstallationUpdateActivity(C#) <br> 작업(JS) | 채널의 조직 단위(예: 고객 테넌트 또는 “팀”) 내에서 봇의 설치 또는 제거를 나타냅니다. |
| [호출](#invoke) | IInvokeActivity(C#) <br> 작업(JS) | 특정 작업을 수행하도록 요청하는 봇에 전송된 통신을 나타냅니다. 이 활동 유형은 Microsoft Bot Framework에 의해 내부 용도로 예약됐습니다. |
| [messageReaction](#messagereaction) | IMessageReactionActivity(C#) <br> 작업(JS) | 사용자가 기존 활동에 대응했음을 나타냅니다. 예를 들어 사용자는 메시지에서 "좋아요" 단추를 클릭합니다. |
| [입력](#typing) | ITypingActivity(C#) <br> 작업(JS) | 대화의 상대편인 사용자 또는 봇이 응답을 컴파일하는 중임을 나타냅니다. |

## <a name="message"></a>Message

<!-- Only the last link is different. -->
::: moniker range="azure-bot-service-3.0"
봇은 사용자에게 정보를 전달하기 위해 메시지 작업을 보내고 사용자에게서 메시지 작업을 받습니다.
일부 메시지는 단순히 일반 텍스트로 구성될 수 있지만 기타 메시지에는 [음성화된 텍스트](v4sdk/bot-builder-howto-send-messages.md#send-a-spoken-message), [제안된 작업](v4sdk/bot-builder-howto-add-suggested-actions.md), [미디어 첨부 파일](v4sdk/bot-builder-howto-add-media-attachments.md), [다양한 카드](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card) 및 [채널 관련 데이터](~/dotnet/bot-builder-dotnet-channeldata.md)와 같은 다양한 콘텐츠가 포함될 수 있습니다.
::: moniker-end
::: moniker range="azure-bot-service-4.0"
봇은 사용자에게 정보를 전달하기 위해 메시지 작업을 보내고 사용자에게서 메시지 작업을 받습니다.
일부 메시지는 단순히 일반 텍스트로 구성될 수 있지만 기타 메시지에는 [음성화된 텍스트](v4sdk/bot-builder-howto-send-messages.md#send-a-spoken-message), [제안된 작업](v4sdk/bot-builder-howto-add-suggested-actions.md), [미디어 첨부 파일](v4sdk/bot-builder-howto-add-media-attachments.md), [다양한 카드](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card) 및 [채널 관련 데이터](~/v4sdk/bot-builder-channeldata.md)와 같은 다양한 콘텐츠가 포함될 수 있습니다.
::: moniker-end

## <a name="contactrelationupdate"></a>contactRelationUpdate

봇은 사용자의 연락처 목록에 추가되거나 제거될 때마다 연락처 관계 업데이트 작업을 받습니다. 작업의 동작 속성의 값(추가 | 제거)은 봇이 사용자의 연락처 목록에 추가됐는지 또는 제거됐는지 여부를 나타냅니다.

## <a name="conversationupdate"></a>conversationUpdate

봇은 대화에 추가되고, 다른 멤버가 대화에 추가되거나 제거되고 또는 대화 메타데이터가 변경될 때마다 대화 업데이트 작업을 받습니다.

대화에 멤버를 추가한 경우 활동의 멤버 추가 속성에는 새 멤버를 식별하기 위한 채널 계정 개체의 배열이 포함됩니다.

봇을 대화에 추가했는지 여부(예: 새 멤버 중 하나)를 확인하려면 작업에 대한 수신자 ID 값(예: 봇의 ID)이 멤버 추가 배열에서 계정에 대한 ID 속성과 일치하는지 여부를 평가합니다.

대화에서 멤버를 제거한 경우 멤버 제거 속성에는 제거된 멤버를 식별하기 위한 채널 계정 개체의 배열이 포함됩니다.

> [!TIP]
> 봇이 사용자가 대화에 조인했음을 나타내는 대화 업데이트 작업을 받는 경우 해당 사용자에게 환영 메시지를 보내 응답하도록 선택할 수 있습니다.

## <a name="deleteuserdata"></a>deleteUserData

사용자가 이전에 봇이 사용자를 위해 보관했던 모든 데이터의 삭제를 요청할 경우 봇은 삭제 사용자 데이터 작업을 받습니다. 봇이 이 유형의 작업을 받는 경우 요청한 사용자에 대해 이전에 저장한 개인 식별 정보(PII)를 삭제해야 합니다.

## <a name="endofconversation"></a>endOfConversation

봇은 사용자가 대화를 종료했음을 나타내기 위해 대화 작업의 종료를 받습니다. 봇은 사용자에게 대화가 끝나감을 나타내기 위해 대화 작업의 종료를 보낼 수 있습니다.

## <a name="event"></a>event

사용자에게 표시되지 않는 정보를 봇에게 전달하려는 외부 프로세스 또는 서비스에서 봇은 이벤트 활동을 받을 수 있습니다. 일반적으로 이벤트 작업의 발송자는 어떤 식으로든 봇이 수신을 승인하기를 예상하지 않습니다.

## <a name="installationupdate"></a>installationUpdate

설치 업데이트 작업은 채널의 조직 단위(예: 고객 테넌트 또는 “팀”) 내에서 봇의 설치 또는 제거를 나타냅니다. 설치 업데이트 작업은 일반적으로 채널의 추가나 제거를 나타내지 않습니다. 테넌트, 팀 또는 채널 내의 다른 조직 단위에서 봇을 제거하거나 추가할 경우 채널에서 설치 작업을 보낼 수 있습니다. 채널에 봇을 설치하거나 제거하는 경우 채널은 설치 작업을 보내지 말아야 합니다.

## <a name="invoke"></a>호출

봇은 특정 작업을 수행하기 위한 요청을 나타내는 호출 작업을 받을 수 있습니다.
호출 작업의 발송자는 일반적으로 봇이 HTTP 응답을 통해 수신을 승인할 것을 예상합니다.
이 작업 유형은 Microsoft Bot Framework에 의해 내부 용도로 예약됐습니다.

## <a name="messagereaction"></a>messageReaction

사용자가 기존 작업에 반응하는 경우 일부 채널은 봇에게 메시지 반응 작업을 보냅니다. 예를 들어 사용자가 메시지에 "좋아요" 단추를 클릭합니다. replyToId 속성은 사용자가 반응한 활동을 나타냅니다.

메시지 반응 작업은 채널이 정의한 많은 메시지 반응 유형과 부합할 수 있습니다. 예를 들어 채널이 보낼 수 있는 반응 유형으로 "좋아요" 또는 "PlusOne"이 있습니다.

## <a name="typing"></a>입력

봇은 사용자가 응답을 입력하는 것을 나타내는 입력 작업을 받습니다.
봇은 요청을 수행하거나 응답을 컴파일하기 위해 작동하고 있음을 사용자에게 표시하는 입력 작업을 보낼 수 있습니다.

::: moniker range="azure-bot-service-3.0"
## <a name="additional-resources"></a>추가 리소스

- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">활동 클래스</a>
::: moniker-end
