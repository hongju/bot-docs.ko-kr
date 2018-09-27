---
title: Bot Connector API를 사용하여 메시지 만들기 | Microsoft Docs
description: Bot Connector API 내에서 일반적으로 사용되는 메시지 속성에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 62088ecdf2f8402ec5456eea758f5db994de0cf9
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707319"
---
# <a name="create-messages"></a>메시지 만들기

봇은 사용자에게 정보를 전달하도록 **메시지** 형식의 [Activity][Activity] 개체를 보내고 마찬가지로 사용자에게서 **메시지** 작업을 받습니다. 일부 메시지는 단순히 일반 텍스트로 구성될 수 있지만 기타 메시지에는 [음성화된 텍스트](bot-framework-rest-connector-text-to-speech.md), [제안된 작업](bot-framework-rest-connector-add-suggested-actions.md), [미디어 첨부 파일](bot-framework-rest-connector-add-media-attachments.md), [다양한 카드](bot-framework-rest-connector-add-rich-cards.md) 및 [채널 관련 데이터](bot-framework-rest-connector-channeldata.md)와 같은 다양한 콘텐츠가 포함될 수 있습니다. 이 문서는 일반적으로 사용되는 메시지 속성의 일부를 설명합니다.

## <a name="message-text-and-formatting"></a>메시지 텍스트 및 서식 지정

메시지 텍스트는 **일반**, **Markdown** 또는 **xml**을 사용하여 서식 지정될 수 있습니다. `textFormat` 속성에 대한 기본 형식은 **Markdown**이며 Markdown 서식 지정 표준을 사용하여 텍스트를 해석합니다. 텍스트 형식 지원의 수준은 채널에 따라 달라집니다. 대상으로 하는 채널에서 사용하려는 기능이 지원되는지 확인하려면 [채널 검사기][ChannelInspector]를 사용하여 기능을 미리 봅니다. 

[Activity][Activity] 개체의 `textFormat` 속성은 텍스트의 서식을 지정하는 데 사용할 수 있습니다. 예를 들어 일반 텍스트만 포함하는 기본 메시지를 만들려면 [Activity][Activity] 개체의 `textFormat` 속성을 **일반**으로 설정하고, `text` 속성을 메시지의 콘텐츠로 설정하고 `locale` 속성을 발신자의 로캘로 설정합니다. 

## <a name="attachments"></a>첨부 파일

[Activity][Activity] 개체의 `attachments` 속성은 간단한 미디어 첨부 파일(이미지, 오디오, 비디오, 파일) 및 다양한 카드를 보내는 데 사용할 수 있습니다. 자세한 내용은 [메시지에 미디어 첨부 파일 추가](bot-framework-rest-connector-add-media-attachments.md) 및 [메시지에 다양한 카드 추가](bot-framework-rest-connector-add-rich-cards.md)를 참조하세요.

## <a name="entities"></a>엔터티

[Activity][Activity] 개체의 `entities` 속성은 채널과 봇 간의 일반적인 상황별 메타데이터를 교환할 수 있는 개방형 <a href="http://schema.org/" target="_blank">schema.org</a> 개체의 배열입니다.

### <a name="mention-entities"></a>멘션 엔터티

다양한 채널은 봇 또는 사용자가 대화 컨텍스트 내에서 누군가를 "멘션"하기 위한 기능을 지원합니다. 메시지에서 사용자를 멘션하려면 메시지의 `entities` 속성을 [Mention][Mention] 개체로 채웁니다. 

### <a name="place-entities"></a>장소 엔터티

메시지 내에서 <a href="https://schema.org/Place" target="_blank">위치 관련 정보</a>를 전달하려면 메시지의 `entities` 속성을 [Place][Place] 개체로 채웁니다. 

## <a name="channel-data"></a>채널 데이터

[Activity][Activity] 개체의 `channelData` 속성은 채널 관련 기능을 구현하는 데 사용할 수 있습니다. 자세한 내용은 [채널 관련 기능 구현](bot-framework-rest-connector-channeldata.md)을 참조하세요.

## <a name="text-to-speech"></a>텍스트 음성 변환

[Activity][Activity] 개체의 `speak` 속성은 음성 지원 채널의 봇에서 말할 텍스트를 지정하는 데 사용할 수 있고 [Activity][Activity] 개체의 `inputHint` 속성은 클라이언트의 마이크의 상태에 영향을 주는 데 사용할 수 있습니다. 자세한 내용은 [메시지에 음성 추가](bot-framework-rest-connector-text-to-speech.md) 및 [메시지에 입력 힌트 추가](bot-framework-rest-connector-add-input-hints.md)를 참조하세요.

## <a name="suggested-actions"></a>제안된 작업

[Activity][Activity] 개체의 `suggestedActions` 속성은 사용자가 입력을 제공하기 위해 누를 수 있는 단추를 표시하는 데 사용할 수 있습니다. 다양한 카드 내에 나타나는 단추와 달리(누른 후에도 사용자에게 표시되고 액세스 가능하도록 남아 있음) 제안된 작업 창 내에 나타나는 단추는 사용자가 선택한 후에 사라집니다. 자세한 내용은 [메시지에 제안된 작업 추가](bot-framework-rest-connector-add-suggested-actions.md)를 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- [채널 검사기를 사용하여 기능 미리 보기][ChannelInspector]
- [작업 개요](bot-framework-rest-connector-activities.md)
- [메시지 보내기 및 받기](bot-framework-rest-connector-send-and-receive-messages.md)
- [메시지에 미디어 첨부 파일 추가](bot-framework-rest-connector-add-media-attachments.md)
- [메시지에 다양한 카드 추가](bot-framework-rest-connector-add-rich-cards.md)
- [메시지에 음성 추가](bot-framework-rest-connector-text-to-speech.md)
- [메시지에 입력 힌트 추가](bot-framework-rest-connector-add-input-hints.md)
- [메시지에 제안된 작업 추가](bot-framework-rest-connector-add-suggested-actions.md)
- [채널 관련 기능 구현](bot-framework-rest-connector-channeldata.md)

[Mention]: bot-framework-rest-connector-api-reference.md#mention-object
[Place]: bot-framework-rest-connector-api-reference.md#place-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ChannelInspector]: ../bot-service-channel-inspector.md
[textFormating]: ../bot-service-channel-inspector.md#text-formatting
