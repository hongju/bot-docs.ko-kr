---
title: 작업 보내기 및 받기 | Microsoft Docs
description: .NET용 Bot Framework SDK를 통해 커넥터 서비스를 사용하여 다양한 채널에서 사용자와 정보를 교환하는 방법을 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0407ec0d90c58e10aa14616e2aa9205bb8840d55
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225228"
---
# <a name="send-and-receive-activities"></a>작업 보내기 및 받기

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Framework Connector는 봇이 Skype, 이메일, Slack 등과 같은 여러 채널에서 통신할 수 있게 해주는 단일 REST API를 제공합니다. 봇에서 채널로, 그리고 채널에서 봇으로 메시지를 릴레이하면 봇과 사용자 간에 쉽게 통신할 수 있습니다. 

이 문서에서는 .NET용 Bot Framework SDK를 통해 커넥터를 사용하여 채널에서 봇과 사용자 간에 정보를 교환하는 방법을 설명합니다. 

> [!NOTE]
> 이 문서에서 설명된 기술만을 사용하여 봇을 구성할 수 있지만, Bot Framework SDK는 대화 흐름과 상태를 관리하는 프로세스를 간소화하고 언어 이해와 같은 인지 서비스를 통합하는 작업을 단순화할 수 있는 [대화 상자](bot-builder-dotnet-dialogs.md) 및 [FormFlow](bot-builder-dotnet-formflow.md)와 같은 추가 기능을 제공합니다.

## <a name="create-a-connector-client"></a>커넥터 클라이언트 만들기

[ConnectorClient][ConnectorClient] 클래스에는 봇이 채널에서 사용자와 통신하는 데 사용하는 메서드가 포함되어 있습니다. 봇이 커넥터에서 <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">작업</a> 개체를 받으면 해당 작업에 지정된 `ServiceUrl`을 사용하여 나중에 응답을 생성하는 데 사용할 커넥터 클라이언트를 만들어야 합니다. 

[!code-csharp[Create connector client](../includes/code/dotnet-send-and-receive.cs#createConnectorClient)]

> [!TIP]
> 채널의 엔드포인트가 안정적이지 않을 수 있으므로 봇은 가능한 경우(캐시된 엔드포인트에 의존하는 대신) 커넥터가 `Activity` 개체에 지정하는 엔드포인트로 직접 통신해야 합니다. 
>
> 봇이 대화를 시작해야 하는 경우 지정된 채널에 캐시된 엔드포인트를 사용할 수 있지만(해당 시나리오에 들어오는 `Activity` 개체가 없으므로) 캐시된 엔드포인트를 자주 새로 고쳐야 합니다. 

## <a id="create-reply"></a> 회신 만들기

커넥터는 [작업](bot-builder-dotnet-activities.md) 개체를 사용하여 봇과 채널(사용자) 간에 정보를 주고 받습니다. 모든 작업에는 메시지를 만든 사람(`From` 속성), 메시지 컨텍스트 및 메시지 받는 사람(`Recipient` 속성)에 대한 정보와 함께 메시지를 적절한 대상에게 라우팅하는 데 사용되는 정보가 포함되어 있습니다.

봇이 커넥터로부터 작업을 수신하면 들어오는 작업의 `Recipient` 속성이 해당 대화에서 봇의 ID를 지정합니다. 일부 채널(예: Slack)이 대화에 추가될 때 봇에게 새 ID를 할당하기 때문에, 봇은 들어오는 작업의 `Recipient` 속성 값을 응답의 `From` 속성 값으로 항상 사용해야 합니다.

나가는 `Activity` 개체를 처음부터 직접 만들고 초기화할 수 있지만 Bot Framework SDK를 사용하면 회신을 더 쉽게 만들 수 있습니다. 들어오는 작업의 `CreateReply` 메서드를 사용할 경우 응답에 대한 메시지 텍스트를 지정하기만 하면 `Recipient`, `From` 및 `Conversation` 속성이 자동으로 채워지는 나가는 작업이 생성됩니다.

[!code-csharp[Create reply](../includes/code/dotnet-send-and-receive.cs#createReply)]

## <a name="send-a-reply"></a>회신 보내기

회신을 만든 후 커넥터 클라이언트의 `ReplyToActivity` 메서드를 호출하여 회신을 보낼 수 있습니다. 커넥터는 적절한 채널 의미 체계를 사용하여 회신을 제공합니다. 

[!code-csharp[Send reply](../includes/code/dotnet-send-and-receive.cs#sendReply)]

> [!TIP]
> 봇이 사용자의 메시지에 회신하는 경우 항상 `ReplyToActivity` 메서드를 사용합니다.

## <a name="send-a-non-reply-message"></a>(비회신) 메시지 보내기 

봇이 대화의 일부인 경우 `SendToConversation` 메서드를 호출하여 사용자가 보낸 모든 메시지에 대한 직접적인 회신이 아닌 메시지를 보낼 수 있습니다. 

[!code-csharp[Send non-reply message](../includes/code/dotnet-send-and-receive.cs#sendNonReplyMessage)]

`CreateReply` 메시지를 사용하여 새 메시지를 초기화할 수 있습니다(메시지의 `Recipient`, `From` 및 `Conversation` 속성이 자동으로 설정됨). 또는 `CreateMessageActivity` 메서드를 사용하여 새 메시지를 만들고 모든 속성 값을 직접 설정할 수 있습니다.

> [!NOTE]
> Bot Framework에는 봇이 보낼 수 있는 메시지 수에 대한 제한이 없습니다. 그러나 대부분의 채널은 제한 한도를 적용하여 봇이 짧은 기간에 다수의 메시지를 보낼 수 없도록 제한합니다. 또한 봇이 여러 메시지를 빠르게 연속적으로 보내는 경우 채널이 메시지를 올바른 순서로 렌더링하지 못할 수도 있습니다.

## <a name="start-a-conversation"></a>대화 시작

봇이 한 명 이상의 사용자와 대화를 시작해야 하는 경우도 있을 수 있습니다. `CreateDirectConversation` 메서드(단일 사용자와 개인 대화인 경우) 또는 `CreateConversation` 메서드(여러 사용자와 그룹 대화인 경우)를 호출하여 `ConversationAccount` 개체를 검색해서 대화를 시작할 수 있습니다. 그런 다음, 메시지를 만들고 `SendToConversation` 메서드를 호출하여 메시지를 보냅니다. `CreateDirectConversation` 메서드 또는 `CreateConversation` 메서드를 사용하려면 먼저 대상 채널의 서비스 URL을 사용하여 [커넥터 클라이언트를 생성](#create-a-connector-client)해야 합니다(이전 메시지에서 계속 유지한 경우 캐시에서 검색할 수 있음). 

> [!NOTE]
> 일부 채널은 그룹 대화를 지원하지 않습니다. 채널이 그룹 대화를 지원하는지 여부를 확인하려면 채널의 설명서를 참조하세요.

이 코드 예제에서는 `CreateDirectConversation` 메서드를 사용하여 단일 사용자와의 개인적인 대화를 만듭니다.

[!code-csharp[Start private conversation](../includes/code/dotnet-send-and-receive.cs#startPrivateConversation)]

이 코드 예제에서는 `CreateConversation` 메서드를 사용하여 여러 사용자와의 그룹 대화를 만듭니다.

[!code-csharp[Start group conversation](../includes/code/dotnet-send-and-receive.cs#startGroupConversation)]

## <a name="additional-resources"></a>추가 리소스

- [작업 개요](bot-builder-dotnet-activities.md)
- [메시지 만들기](bot-builder-dotnet-create-messages.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Framework SDK 참조</a>
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">활동 클래스</a>
- <a href="/dotnet/api/microsoft.bot.connector.connectorclient" target="_blank">ConnectorClient 클래스</a>

[ConnectorClient]: /dotnet/api/microsoft.bot.connector.connectorclient
