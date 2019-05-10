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
ms.openlocfilehash: 9fa9a23f4d14667aeb97d304498b415f2c8041d1
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033063"
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
| Type | 엔터티의 형식("멘션") |
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
| Type | 엔터티의 형식("Place") |
| 주소 | 설명 또는 우편 주소 개체(이후) |
| 지역 | GeoCoordinates |
| HasMap | 맵 또는 맵 개체에 대한 URL(이후) |
| Name | 장소의 이름 |

geoCoordinates 개체는 다음 속성을 포함합니다.

| 자산 | 설명 |
|----|----|
| Type | 엔터티의 형식("GeoCoordinates") |
| Name | 장소의 이름 |
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

몇 가지 작업 형식이 있습니다. 작업은 다른 형식의 가장 일반적인 **메시지**일 수 있습니다. 설명 및 자세한 내용은 [활동 스키마 페이지](https://aka.ms/botSpecs-activitySchema)에서 확인할 수 있습니다.

::: moniker range="azure-bot-service-3.0"

## <a name="additional-resources"></a>추가 리소스

- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">활동 클래스</a>
::: moniker-end
