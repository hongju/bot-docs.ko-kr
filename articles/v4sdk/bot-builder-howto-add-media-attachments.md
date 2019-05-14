---
title: 메시지에 미디어 추가 | Microsoft Docs
description: Bot Framework SDK를 사용하여 메시지에 미디어를 추가하는 방법에 대해 알아봅니다.
keywords: 미디어, 메시지, 이미지, 오디오, 비디오, 파일, MessageFactory, 서식 있는 카드, 메시지, 적응형 카드, 영웅 카드, 제안된 작업
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7093f13d1958c741b497a50535eb70a255dfcbe8
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032583"
---
# <a name="add-media-to-messages"></a>메시지에 미디어 추가

[!INCLUDE[applies-to](../includes/applies-to.md)]

사용자와 봇 간에 교환되는 메시지에는 이미지, 비디오, 오디오 및 파일과 같은 미디어 첨부 파일이 포함될 수 있습니다. Bot Framework SDK는 사용자에게 다양한 메시지를 보내는 작업을 지원합니다. 채널(Facebook, Skype, Slack 등)에서 지원하는 다양한 메시지의 유형을 결정하려면 채널 설명서에서 제한 사항에 대한 정보를 참조하세요.

사용 가능한 카드의 예는 [사용자 환경 디자인](../bot-service-design-user-experience.md)을 참조하세요.

## <a name="send-attachments"></a>첨부 파일 보내기

이미지 또는 비디오와 같은 사용자 콘텐츠를 보내려면 첨부 파일 또는 첨부 파일 목록을 메시지에 추가하면 됩니다.

사용 가능한 카드의 예는 [사용자 환경 디자인](../bot-service-design-user-experience.md)을 참조하세요.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` 개체의 `Attachments` 속성에는 메시지에 첨부된 미디어 첨부 파일과 서식 있는 카드를 나타내는 `Attachment` 개체의 배열이 포함되어 있습니다. 메시지에 미디어 첨부 파일을 추가하려면 `reply` 작업(`CreateReply()` 작업을 통해 만들어짐)에 대한 `Attachment` 개체를 만들고 `ContentType`, `ContentUrl` 및 `Name` 속성을 설정합니다.

여기서 보여 주는 소스 코드는 [첨부 파일 처리](https://aka.ms/bot-attachments-sample-code) 샘플을 기반으로 합니다.

회신 메시지를 만들려면 텍스트를 정의한 다음, 첨부 파일을 설정합니다. 첨부 파일을 회신에 할당하는 작업은 첨부 파일 유형마다 같지만 여러 첨부 파일은 다음 코드 조각에서 볼 수 있듯이 서로 다르게 설정 및 정의됩니다. 아래 코드는 인라인 첨부 파일에 대한 회신을 설정합니다.

**Bots/AttachmentsBot.cs** [!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=108-109)]

다음으로, 첨부 파일의 유형을 살펴봅니다. 먼저 인라인 첨부 파일입니다.

**Bots/AttachmentsBot.cs** [!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=165-176)]

다음으로 업로드된 첨부 파일입니다.

**Bots/AttachmentsBot.cs** [!code-csharp[uploaded attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=179-215)]

마지막으로, 인터넷 첨부 파일입니다.

**Bots/AttachmentsBot.cs** [!code-csharp[online attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=218-227)]


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

여기서 보여 주는 소스 코드는 [JS 첨부 파일 처리](https://aka.ms/bot-attachments-sample-code-js) 샘플을 기반으로 합니다.

첨부 파일을 사용하려면 다음 라이브러리를 봇에 포함합니다.

**bots/attachmentsBot.js** [!code-javascript[attachments libraries](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=4)]

회신 메시지를 만들려면 텍스트를 정의한 다음, 첨부 파일을 설정합니다. 첨부 파일을 회신에 할당하는 작업은 첨부 파일 유형마다 같지만 여러 첨부 파일은 다음 코드 조각에서 볼 수 있듯이 서로 다르게 설정 및 정의됩니다. 아래 코드는 인라인 첨부 파일에 대한 회신을 설정합니다.

**bots/attachmentsBot.js** [!code-javascript[attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=119,128-129)]

사용자에게 이미지 또는 비디오와 같은 콘텐츠의 한 조각을 보내려면 몇 가지 다른 방법으로 미디어를 보내면 됩니다. 먼저 인라인 첨부 파일입니다.

**bots/attachmentsBot.js** [!code-javascript[inline attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=170-179)]

다음으로 업로드된 첨부 파일입니다.

**bots/attachmentsBot.js** [!code-javascript[uploaded attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=197-215)]

마지막으로, 다음 URL에 포함된 인터넷 첨부 파일은 다음과 같습니다.

**bots/attachmentsBot.js** [!code-javascript[internet attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=184-191)]

---

첨부 파일이 이미지, 오디오 또는 비디오인 경우 커넥터 서비스는 [채널](bot-builder-channeldata.md)이 대화 내의 해당 첨부 파일을 렌더링할 수 있도록 채널에 첨부 파일 데이터를 전달합니다. 첨부 파일이 파일인 경우 파일 URL은 대화 내 하이퍼링크로 렌더링됩니다.

## <a name="send-a-hero-card"></a>영웅 카드 보내기

단순 이미지 또는 비디오 첨부 파일 외에도 **영웅 카드**를 첨부할 수 있습니다. 이 카드를 사용하면 이미지 및 단추를 하나의 개체로 결합하여 사용자에게 보낼 수 있습니다. Markdown은 대부분의 텍스트 필드에 지원되지만 채널별로 지원이 다를 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

영웅 카드와 단추가 있는 메시지를 작성하려면 `HeroCard`를 메시지에 첨부할 수 있습니다. 

여기서 보여 주는 소스 코드는 [첨부 파일 처리](https://aka.ms/bot-attachments-sample-code) 샘플을 기반으로 합니다.

**Bots/AttachmentsBot.cs** [!code-csharp[Hero card](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=39-62)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

영웅 카드와 단추가 있는 메시지를 작성하려면 `HeroCard`를 메시지에 첨부할 수 있습니다. 

여기서 보여 주는 소스 코드는 [JS 첨부 파일 처리](https://aka.ms/bot-attachments-sample-code-js) 샘플을 기반으로 합니다.

**bots/attachmentsBot.js** [!code-javascript[hero card](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=148-164)]

---

## <a name="process-events-within-rich-cards"></a>서식 있는 카드 내에서 이벤트 처리

서식 있는 카드 내에서 이벤트를 처리하려면 _카드 작업_ 개체를 사용하여 사용자가 단추를 클릭하거나 카드의 섹션을 탭할 때 수행되어야 하는 작업을 지정합니다. 각 카드 동작에는 _type_ 및 _value_가 있습니다.

올바르게 작동하려면 카드의 클릭 가능한 각 항목에 작업 유형을 할당합니다. 이 표에는 사용 가능한 동작 유형 및 관련 값 속성에 필요한 항목과 관련 설명이 나와 있습니다.

| Type | 설명 | 값 |
| :---- | :---- | :---- |
| openUrl | 기본 제공 브라우저에서 URL이 열립니다. | 열려는 URL입니다. |
| imBack | 봇에 메시지를 보내고 채팅에 표시 가능한 응답을 게시합니다. | 보낼 메시지의 텍스트입니다. |
| postBack | 봇에 메시지를 보내고 채팅에 표시 가능한 응답을 게시하지 않을 수 있습니다. | 보낼 메시지의 텍스트입니다. |
| 다음을 호출합니다. | 전화 통화를 시작합니다. | 전화 통화 대상입니다(`tel:123123123123` 형식). |
| playAudio | 오디오를 재생합니다. | 재생할 오디오의 URL입니다. |
| playVideo | 비디오를 재생합니다. | 재생할 비디오의 URL입니다. |
| showImage | 이미지를 표시합니다. | 표시할 이미지의 URL입니다. |
| downloadFile | 파일을 다운로드합니다. | 다운로드할 파일의 URL입니다. |
| signin | OAuth 로그인 프로세스를 시작합니다. | 시작할 OAuth 흐름의 URL입니다. |

## <a name="hero-card-using-various-event-types"></a>다양한 이벤트 유형을 사용하는 영웅 카드

다음 코드에서는 다양한 서식이 있는 카드 이벤트를 사용하는 예제를 보여 줍니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

사용 가능한 모든 카드의 예는 [C# 카드 샘플](https://aka.ms/bot-cards-sample-code)을 참조하세요.

**Cards.cs** [!code-csharp[hero cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=27-40)]

**Cards.cs** [!code-csharp[cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=91-100)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

사용 가능한 모든 카드의 예는 [JS 카드 샘플](https://aka.ms/bot-cards-js-sample-code)을 참조하세요.

**dialogs/mainDialog.js** [!code-javascript[hero cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=213-225)]

**dialogs/mainDialog.js** [!code-javascript[sign in cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=266-272)]

---

## <a name="send-an-adaptive-card"></a>적응형 카드 보내기
적응형 카드 및 MessageFactory는 사용자와 통신하기 위해 텍스트, 이미지, 비디오, 오디오 및 파일이 포함된 서식 있는 메시지를 보내는 데 사용됩니다. 하지만 이들 사이에는 약간의 차이점이 있습니다. 

첫째, 일부 채널만 적응형 카드를 지원하며, 이를 지원하는 채널은 부분적으로 적응형 카드를 지원할 수 있습니다. 예를 들어, Facebook에서 적응형 카드를 보내면 텍스트와 이미지가 제대로 작동하는 동안 단추가 작동하지 않습니다. MessageFactory는 Bot Framework SDK 내의 도우미 클래스로, 생성 단계를 자동화할 수 있으며, 대부분의 채널에서 지원됩니다. 

둘째, 적응형 카드는 카드 양식의 메시지를 전달하며, 채널은 카드의 레이아웃을 결정합니다. MessageFactory가 제공하는 메시지 양식은 채널에 따라 다르며, 적응형 카드가 첨부 파일의 일부가 아니라면 카드 양식이 아니어도 됩니다. 

적응형 카드 채널 지원에 대한 최신 정보는 <a href="http://adaptivecards.io/designer/">적응형 카드 디자이너</a>를 참조하세요.

> [!NOTE]
> 해당 채널이 적응형 카드를 지원하는지 여부를 확인하려면 봇이 사용할 채널을 통해 이 기능을 테스트해야 합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

적응형 카드를 사용하려면 `AdaptiveCards` NuGet 패키지를 추가해야 합니다.

여기서 보여 주는 소스 코드는 [카드 사용](https://aka.ms/bot-cards-sample-code) 샘플을 기반으로 합니다.

**Cards.cs** [!code-csharp[adaptive cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=13-25)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

적응형 카드를 사용하려면 `adaptivecards` npm 패키지를 추가해야 합니다.

여기서 보여 주는 소스 코드는 [카드 사용 JS](https://aka.ms/bot-cards-js-sample-code) 샘플을 기반으로 합니다. 

여기서 적응형 카드는 자체 파일에 저장되며 봇에 포함됩니다.

**resources/adaptiveCard.json** [!code-json[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/resources/adaptiveCard.json)]

그런 다음, CardFactory를 통해 만들어집니다.

**dialogs/mainDialog.js** [!code-javascript[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=177-179)]

---

## <a name="send-a-carousel-of-cards"></a>회전식 카드 보내기

메시지는 첨부 파일을 나란히 배치하고 사용자가 스크롤할 수 있도록 하는 회전식 레이아웃에 여러 첨부 파일을 포함할 수도 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

여기서 보여 주는 소스 코드는 [카드 샘플](https://aka.ms/bot-cards-sample-code)을 기반으로 합니다.

먼저 회신을 만들고 첨부 파일을 목록으로 정의합니다.

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=61-66)]

그런 다음, 첨부 파일을 추가합니다. 여기서는 한 번에 하나를 추가하지만 목록을 조작하여 원하는 만큼 카드를 추가할 수 있습니다.

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=105-113)]

첨부 파일이 추가되면 다른 것과 마찬가지로 회신을 보낼 수 있습니다.

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=117-118)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

여기서 보여 주는 소스 코드는 [JS 카드 샘플](https://aka.ms/bot-cards-js-sample-code)을 기반으로 합니다.

회전식 카드를 보내려면 첨부 파일을 `Carousel`로 정의된 레이아웃 유형 및 배열로 회신합니다.

**dialogs/mainDialog.js** [!code-javascript[carousel of cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=104-116)]

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>추가 리소스

사용 가능한 카드의 예는 [사용자 환경 디자인](../bot-service-design-user-experience.md)을 참조하세요.

스키마에 대한 자세한 내용은 Bot Framework 작업 스키마의 [Bot Framework 카드 스키마](https://aka.ms/botSpecs-cardSchema) 및 [메시지 작업 섹션](https://aka.ms/botSpecs-activitySchema#message-activity)을 참조하세요.

| 샘플 코드 | C# | JS |
| :------ | :----- | :---|
| 카드 | [C# 샘플](https://aka.ms/bot-cards-sample-code) | [JS 샘플](https://aka.ms/bot-cards-js-sample-code) |
| 첨부 파일 | [C# 샘플](https://aka.ms/bot-attachments-sample-code) | [JS 샘플](https://aka.ms/bot-attachments-sample-code-js) |
| 제안된 작업 | [C# 샘플](https://aka.ms/SuggestedActionsCSharp) | [JS 샘플](https://aka.ms/SuggestedActionsJS) |

추가 샘플은 [GitHub](https://aka.ms/bot-samples-readme)의 Bot Builder 샘플 리포지토리를 참조하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [사용자 작업을 안내하는 단추 추가](./bot-builder-howto-add-suggested-actions.md)
