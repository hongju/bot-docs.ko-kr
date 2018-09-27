---
title: .NET용 Bot Builder SDK를 사용하여 메시지 만들기 | Microsoft Docs
description: .NET용 Bot Builder SDK 내에서 일반적으로 사용되는 메시지 속성에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c35e651f674d65728ac93a815cc7116515790f53
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707879"
---
# <a name="create-messages"></a>메시지 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

봇은 사용자에게 정보를 전달하도록 **메시지** [작업](bot-builder-dotnet-activities.md)을 보내고 마찬가지로 사용자에게 **메시지** 작업을 받습니다. 일부 메시지는 단순히 일반 텍스트로 구성될 수 있지만 기타 메시지에는 [음성화된 텍스트](bot-builder-dotnet-text-to-speech.md), [제안된 작업](bot-builder-dotnet-add-suggested-actions.md), [미디어 첨부 파일](bot-builder-dotnet-add-media-attachments.md), [다양한 카드](bot-builder-dotnet-add-rich-card-attachments.md) 및 [채널 관련 데이터](bot-builder-dotnet-channeldata.md)와 같은 다양한 콘텐츠가 포함될 수 있습니다. 

이 문서는 일반적으로 사용되는 메시지 속성의 일부를 설명합니다.

## <a name="customizing-a-message"></a>메시지 사용자 지정

메시지의 텍스트 서식에 대한 제어를 강화하려면 [Activity](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) 개체를 사용하여 사용자 지정 메시지를 만들고 사용자에게 보내기 전에 필요한 속성을 설정할 수 있습니다.

이 샘플에서는 사용자 지정 `message` 개체를 만들고 `Text`, `TextFormat` 및 `Local` 속성을 설정하는 방법을 보여줍니다.

[!code-csharp[Set message properties](../includes/code/dotnet-create-messages.cs#setBasicProperties)]

메시지의 `TextFormat` 속성은 텍스트의 서식을 지정하는 데 사용할 수 있습니다. `TextFormat` 속성은 **일반**, **Markdown** 또는 **xml**로 설정할 수 있습니다. `TextFormat`의 기본값은 **Markdown**입니다. 

## <a name="attachments"></a>첨부 파일

메시지 작업의 `Attachments` 속성은 간단한 미디어 첨부 파일(이미지, 오디오, 비디오, 파일) 및 다양한 카드를 보내고 받는 데 사용할 수 있습니다. 자세한 내용은 [메시지에 미디어 첨부 파일 추가](bot-builder-dotnet-add-media-attachments.md) 및 [메시지에 다양한 카드 추가](bot-builder-dotnet-add-rich-card-attachments.md)를 참조하세요.

## <a name="entities"></a>엔터티

메시지의 `Entities` 속성은 채널과 봇 간의 일반적인 상황별 메타데이터를 교환할 수 있는 개방형 <a href="http://schema.org/" target="_blank">schema.org</a> 개체의 배열입니다.

### <a name="mention-entities"></a>멘션 엔터티

다양한 채널은 봇 또는 사용자가 대화 컨텍스트 내에서 누군가를 "멘션"하기 위한 기능을 지원합니다. 메시지에서 사용자를 멘션하려면 메시지의 `Entities` 속성을 `Mention` 개체로 채웁니다. `Mention` 개체는 다음 속성을 포함합니다. 

| 자산 | 설명 | 
|----|----|
| type | 엔터티의 형식("멘션") | 
| 멘션됨 | 멘션된 사용자를 나타내는 `ChannelAccount` 개체 | 
| 텍스트 | 멘션 자체를 나타내는 `Activity.Text` 속성 내의 텍스트(비어 있거나 Null일 수 있음) |

이 코드 예제는 `Entities` 컬렉션에 `Mention` 엔터티를 추가하는 방법을 보여줍니다.

[!code-csharp[set Mention](../includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> 사용자 의도를 확인하려고 시도할 때 봇은 멘션된 메시지의 해당 부분을 무시하려고 합니다. `GetMentions` 메서드를 호출하고 응답에서 반환된 `Mention` 개체를 평가합니다.

### <a name="place-objects"></a>장소 개체

<a href="https://schema.org/Place" target="_blank">위치 관련 정보</a>는 메시지의 `Entities` 속성을 `Place` 개체 또는 `GeoCoordinates` 개체 중 하나로 채워 메시지 내에서 전달될 수 있습니다. 

`Place` 개체는 다음 속성을 포함합니다.

| 자산 | 설명 | 
|----|----|
| type | 엔터티의 형식("Place") |
| 주소 | 설명 또는 `PostalAddress` 개체(이후) | 
| 지역 | GeoCoordinates | 
| HasMap | 맵 또는 `Map` 개체에 대한 URL(이후) |
| 이름 | 장소의 이름 |

`GeoCoordinates` 개체는 다음 속성을 포함합니다.

| 자산 | 설명 | 
|----|----|
| type | 엔터티의 형식("GeoCoordinates") |
| 이름 | 장소의 이름 |
| 경도 | 위치의 경도(<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| 위도 | 위치의 위도(<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| 상승 | 위치의 상승(<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 

이 코드 예제는 `Entities` 컬렉션에 `Place` 엔터티를 추가하는 방법을 보여줍니다.

[!code-csharp[set GeoCoordinates](../includes/code/dotnet-create-messages.cs#setGeoCoord)]

### <a name="consume-entities"></a>엔터티 사용

엔터티를 사용하려면 `dynamic` 키워드 또는 강력한 형식의 클래스 중 하나를 사용합니다.

이 코드 예제는 `dynamic` 키워드를 사용하여 메시지의 `Entities` 속성 내에서 엔터티를 처리하는 방법을 보여줍니다.

[!code-csharp[examine entity using dynamic keyword](../includes/code/dotnet-create-messages.cs#examineEntity1)]

이 코드 예제는 강력한 형식의 클래스를 사용하여 메시지의 `Entities` 속성 내에서 엔터티를 처리하는 방법을 보여줍니다.

[!code-csharp[examine entity using typed class](../includes/code/dotnet-create-messages.cs#examineEntity2)]

## <a name="channel-data"></a>채널 데이터

메시지 작업의 `ChannelData` 속성은 채널 관련 기능을 구현하는 데 사용할 수 있습니다. 자세한 내용은 [채널 관련 기능 구현](bot-builder-dotnet-channeldata.md)을 참조하세요.

## <a name="text-to-speech"></a>텍스트 음성 변환

메시지 작업의 `Speak` 속성은 음성 지원 채널의 봇에서 말할 텍스트를 지정하는 데 사용할 수 있습니다. 메시지 작업의 `InputHint` 속성은 클라이언트의 마이크와 입력 상자의 상태를 제어하는데 사용할 수 있습니다(있는 경우). 자세한 내용은 [메시지에 음성 추가](bot-builder-dotnet-text-to-speech.md)를 참조하세요.

## <a name="suggested-actions"></a>제안된 작업

사용자 작업의 `SuggestedActions` 속성은 사용자가 입력을 제공하기 위해 누를 수 있는 단추를 표시하는 데 사용할 수 있습니다. 다양한 카드 내에 나타나는 단추와 달리(누른 후에도 사용자에게 표시되고 액세스 가능하도록 남아 있음) 제안된 작업 창 내에 나타나는 단추는 사용자가 선택한 후에 사라집니다. 자세한 내용은 [메시지에 제안된 작업 추가](bot-builder-dotnet-add-suggested-actions.md)를 참조하세요.

## <a name="next-steps"></a>다음 단계

봇 및 사용자는 서로 메시지를 보낼 수 있습니다. 메시지가 더 복잡한 경우 봇은 메시지에서 다양한 카드를 사용자에게 보낼 수 있습니다. 다양한 카드는 대부분의 봇에서 일반적으로 필요한 다양한 프레젠테이션 및 상호 작용 시나리오를 다룹니다.

> [!div class="nextstepaction"]
> [메시지에서 다양한 카드 보내기](bot-builder-dotnet-add-rich-card-attachments.md)

## <a name="additional-resources"></a>추가 리소스

- [작업 개요](bot-builder-dotnet-activities.md)
- [작업 보내기 및 받기](bot-builder-dotnet-connector.md)
- [메시지에 미디어 첨부 파일 추가](bot-builder-dotnet-add-media-attachments.md)
- [메시지에 다양한 카드 추가](bot-builder-dotnet-add-rich-card-attachments.md)
- [메시지에 음성 추가](bot-builder-dotnet-text-to-speech.md)
- [메시지에 제안된 작업 추가](bot-builder-dotnet-add-suggested-actions.md)
- [채널 관련 기능 구현](bot-builder-dotnet-channeldata.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">작업 클래스</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity 인터페이스</a>

