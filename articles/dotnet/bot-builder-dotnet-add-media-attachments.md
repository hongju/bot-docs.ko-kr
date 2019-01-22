---
title: 메시지에 미디어 첨부 파일 추가 | Microsoft Docs
description: .NET용 Bot Framework SDK를 사용하여 메시지에 미디어 첨부 파일을 추가하는 방법에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6dcfe6595f1c5961151a90783dd8ceee9c7684dd
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224338"
---
# <a name="add-media-attachments-to-messages"></a>메시지에 미디어 첨부 파일 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

사용자와 봇 간의 메시지 교환에는 미디어 첨부 파일(예: 이미지, 비디오, 오디오, 파일)이 포함될 수 있습니다. <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">작업</a> 개체의 `Attachments` 속성에는 메시지 내의 미디어 첨부 파일과 서식 있는 카드를 나타내는 <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">첨부 파일</a> 개체의 배열이 포함되어 있습니다. 

> [!NOTE]
> [메시지에 서식 있는 카드 추가](bot-builder-dotnet-add-rich-card-attachments.md).

## <a name="add-a-media-attachment"></a>미디어 첨부 파일 추가  

메시지에 미디어 첨부 파일을 추가하려면 `message` 작업에 대한 `Attachment` 개체를 만들고 `ContentType`, `ContentUrl` 및 `Name` 속성을 설정합니다. 

[!code-csharp[Add media attachment](../includes/code/dotnet-add-attachments.cs#addMediaAttachment)]

첨부 파일이 이미지, 오디오 또는 비디오인 경우 커넥터 서비스는 [채널](bot-builder-dotnet-channeldata.md)이 대화 내의 해당 첨부 파일을 렌더링할 수 있도록 채널에 첨부 파일 데이터를 전달합니다. 첨부 파일이 파일인 경우 파일 URL은 대화 내 하이퍼링크로 렌더링됩니다.

## <a name="additional-resources"></a>추가 리소스

- [채널 검사기를 사용하여 기능 미리 보기][inspector]
- [작업 개요](bot-builder-dotnet-activities.md)
- [메시지 만들기](bot-builder-dotnet-create-messages.md)
- [메시지에 서식 있는 카드 추가](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">작업 클래스</a>
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">첨부 파일 클래스</a>

[inspector]: ../bot-service-channel-inspector.md


