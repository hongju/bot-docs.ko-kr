---
title: 메시지에 서식 있는 카드 첨부 파일 추가 | Microsoft Docs
description: .NET용 Bot Framework SDK를 사용하여 메시지에 서식 있는 카드를 추가하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5a6fc63005797a1c645de7506a8f15df2dcd0557
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317678"
---
# <a name="add-rich-card-attachments-to-messages"></a>메시지에 서식 있는 카드 첨부 파일 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

사용자와 봇 간의 메시지 교환에는 목록 또는 회전식으로 렌더링되는 하나 이상의 서식 있는 카드가 포함될 수 있습니다. <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">작업</a> 개체의 `Attachments` 속성에는 메시지 내의 서식 있는 카드 및 미디어 첨부 파일을 나타내는 <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">첨부 파일</a> 개체의 배열이 포함되어 있습니다. 

> [!NOTE]
> 미디어 첨부 파일을 메시지에 추가하는 방법에 대한 자세한 내용은 [메시지에 미디어 첨부 파일 추가](bot-builder-dotnet-add-media-attachments.md)를 참조하세요.

## <a name="types-of-rich-cards"></a>서식 있는 카드 형식

Bot Framework는 현재 8가지 형식의 서식 있는 카드를 지원합니다. 

| 카드 형식 | 설명 |
|----|----|
| <a href="/adaptive-cards/get-started/bots">적응형 카드</a> | 텍스트, 음성, 이미지, 단추 및 입력 필드의 조합을 포함할 수 있는 사용자 지정 가능한 카드입니다. [채널별 지원](/adaptive-cards/get-started/bots#channel-status)을 참조하세요.  |
| [애니메이션 카드][animationCard] | 애니메이션 GIF 또는 짧은 비디오를 재생할 수 있는 카드입니다. |
| [오디오 카드][audioCard] | 오디오 파일을 재생할 수 있는 카드입니다. |
| [영웅 카드][heroCard] | 일반적으로 하나의 큰 이미지, 하나 이상의 단추 및 텍스트를 포함하는 카드입니다. |
| [썸네일 카드][thumbnailCard] | 일반적으로 하나의 썸네일 이미지, 하나 이상의 단추 및 텍스트를 포함하는 카드입니다. |
| [수신 확인 카드][receiptCard] | 봇이 사용자에게 수신 확인을 제공할 수 있는 카드입니다. 일반적으로 수신 확인, 세금 및 합계 정보에 포함할 항목 목록과 기타 텍스트를 포함합니다. |
| [로그인 카드][signinCard] | 봇이 사용자 로그인을 요청하는 데 사용되는 카드입니다. 일반적으로 사용자가 로그인 프로세스를 시작하기 위해 클릭할 수 있는 하나 이상의 단추와 텍스트를 포함합니다. |
| [비디오 카드][videoCard] | 비디오를 재생할 수 있는 카드입니다. |

> [!TIP]
> 여러 서식 있는 카드를 목록 형식으로 표시하려면 작업의 `AttachmentLayout` 속성을 “list”로 설정합니다. 여러 서식 있는 카드를 회전식 형식으로 표시하려면 작업의 `AttachmentLayout` 속성을 “carousel”로 설정합니다. 채널이 회전식 형식을 지원하지 않으면 `AttachmentLayout` 속성을 “carousel”로 지정된 경우에도 서식 있는 카드가 목록 형식으로 표시됩니다.

## <a name="process-events-within-rich-cards"></a>서식 있는 카드 내에서 이벤트 처리

서식 있는 카드 내에서 이벤트를 처리하려면 `CardAction` 개체를 정의하여 사용자가 단추를 클릭하거나 카드의 섹션을 탭할 때 수행되어야 하는 작업을 지정합니다. 각 `CardAction` 개체는 다음 속성을 포함합니다.

| 자산 | type | 설명 | 
|----|----|----|
| type | string | 작업 형식(아래 표에 지정된 값 중 하나) |
| 제목 | string | 단추 제목 |
| 이미지 | string | 단추의 이미지 URL |
| 값 | string | 지정된 형식의 작업을 수행하는 데 필요한 값 |

> [!NOTE]
> 적응형 카드 내의 단추를 만들 때는 `CardAction` 개체를 사용하는 대신, <a href="http://adaptivecards.io" target="_blank">적응형 카드</a>에서 정의한 스키마를 사용합니다. 적응형 카드에 단추를 추가하는 방법을 보여 주는 예제는 [메시지에 적응형 카드 추가](#adaptive-card)를 참조하세요.

이 표에는 `CardAction.Type`의 유효한 값이 나열되어 있으며 각 유형별로 `CardAction.Value`의 올바른 콘텐츠가 설명되어 있습니다.

| CardAction.Type | CardAction.Value | 
|----|----|
| openUrl | 기본 제공 브라우저에서 열리는 URL |
| imBack | 단추를 클릭하거나 카드를 탭한 사용자에서 봇으로 전송할 메시지의 텍스트. 이 메시지(사용자에서 봇으로)는 대화를 호스트하고 있는 클라이언트 애플리케이션을 통해 모든 대화 참가자에게 표시됩니다. |
| postBack | 단추를 클릭하거나 카드를 탭한 사용자에서 봇으로 전송할 메시지의 텍스트. 일부 클라이언트 애플리케이션은 이 텍스트를 메시지 피드로 표시할 수 있습니다. 메시지 피드에서는 텍스트가 모든 대화 참가자에게 표시됩니다. |
| 다음을 호출합니다. | 이 형식의 전화 통화 대상: **tel:123123123123** |
| playAudio | 재생되는 오디오의 URL |
| playVideo | 재생되는 비디오의 URL |
| showImage | 표시되는 이미지의 URL |
| downloadFile | 다운로드되는 파일의 URL |
| signin | 시작되는 OAuth 흐름의 URL |

## <a name="add-a-hero-card-to-a-message"></a>메시지에 영웅 카드 추가

영웅 카드는 일반적으로 하나의 큰 이미지, 하나 이상의 단추 및 텍스트를 포함합니다. 

이 코드 예제에서는 회전식 형식으로 렌더링된 세 개의 영웅 카드를 포함하는 회신 메시지를 만드는 방법을 보여 줍니다. 

[!code-csharp[Add HeroCard attachment](../includes/code/dotnet-add-attachments.cs#addHeroCardAttachment)]

## <a name="add-a-thumbnail-card-to-a-message"></a>메시지에 썸네일 카드 추가

썸네일 카드는 일반적으로 하나의 썸네일 이미지, 하나 이상의 단추 및 텍스트를 포함합니다. 

이 코드 예제에서는 목록 형식으로 렌더링된 두 개의 썸네일 카드를 포함하는 회신 메시지를 만드는 방법을 보여 줍니다. 

[!code-csharp[Add ThumbnailCard attachment](../includes/code/dotnet-add-attachments.cs#addThumbnailCardAttachment)]

## <a name="add-a-receipt-card-to-a-message"></a>메시지에 수신 확인 카드 추가

수신 확인 카드는 봇이 사용자에게 수신 확인을 제공하는 데 사용됩니다. 일반적으로 수신 확인, 세금 및 합계 정보에 포함할 항목 목록과 기타 텍스트를 포함합니다. 

이 코드 예제에서는 수신 확인 카드를 포함하는 회신 메시지를 만드는 방법을 보여 줍니다. 

[!code-csharp[Add ReceiptCard attachment](../includes/code/dotnet-add-attachments.cs#addReceiptCardAttachment)]

## <a name="add-a-sign-in-card-to-a-message"></a>메시지에 로그인 카드 추가

로그인 카드는 봇이 사용자 로그인을 요청하는 데 사용됩니다. 일반적으로 사용자가 로그인 프로세스를 시작하기 위해 클릭할 수 있는 하나 이상의 단추와 텍스트를 포함합니다. 

이 코드 예제에서는 로그인 카드를 포함하는 회신 메시지를 만드는 방법을 보여 줍니다.

[!code-csharp[Add SignInCard attachment](../includes/code/dotnet-add-attachments.cs#addSignInCardAttachment)]

## <a id="adaptive-card"></a> 메시지에 적응형 카드 추가

적응형 카드는 텍스트, 음성, 이미지, 단추 및 입력 필드의 모든 조합을 포함할 수 있습니다. 적응형 카드는 <a href="http://adaptivecards.io" target="_blank">적응형 카드</a>에 지정된 JSON 형식을 사용하여 생성되며, 카드 콘텐츠 및 형식에 대한 모든 권한을 제공합니다. 

.NET을 사용하여 적응형 카드를 만들려면 `AdaptiveCards` NuGet 패키지를 설치합니다. 그런 다음, <a href="http://adaptivecards.io" target="_blank">적응형 카드</a> 내의 정보를 이용하여 적응형 카드 스키마를 이해하고, 적응형 카드 요소를 살펴보고, 다양한 구성 및 복잡성이 포함된 카드를 만드는 데 사용할 수 있는 JSON 샘플을 확인합니다. 또한, 대화형 시각화 도우미를 사용하여 적응형 카드 페이로드를 설계하고 카드 출력을 미리 볼 수 있습니다.

이 코드 예제에서는 일정 미리 알림을 위한 적응형 카드를 포함하는 메시지를 만드는 방법을 보여 줍니다. 

[!code-csharp[Add Adaptive Card attachment](../includes/code/dotnet-add-attachments.cs#addAdaptiveCardAttachment)]

결과 카드에는 세 개의 텍스트 블록, 입력 필드(선택 목록) 및 세 개의 단추가 포함됩니다.

![적응형 카드 일정 미리 알림](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>추가 리소스

- [채널 검사기를 사용하여 기능 미리 보기][inspector]
- <a href="http://adaptivecards.io" target="_blank">적응형 카드</a>
- [작업 개요](bot-builder-dotnet-activities.md)
- [메시지 만들기](bot-builder-dotnet-create-messages.md)
- [메시지에 미디어 첨부 파일 추가](bot-builder-dotnet-add-media-attachments.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">작업 클래스</a>
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">첨부 파일 클래스</a>

[animationCard]: /dotnet/api/microsoft.bot.connector.animationcard

[audioCard]: /dotnet/api/microsoft.bot.connector.audiocard 

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard 

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 

[videoCard]: /dotnet/api/microsoft.bot.connector.videocard

[inspector]: ../bot-service-channel-inspector.md
