---
title: .NET용 Bot Builder SDK의 주요 개념 | Microsoft Docs
description: .Net용 Bot Builder SDK에서 사용할 수 있는 대화 봇을 빌드 및 배포하기 위한 주요 개념과 도구를 살펴봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0a8c64cced0cb6c10daeedee4edfd777ff6d8e78
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300901"
---
# <a name="key-concepts-in-the-bot-builder-sdk-for-net"></a>.NET용 Bot Builder SDK의 주요 개념
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.js](../nodejs/bot-builder-nodejs-concepts.md)

이 문서에서는 .NET용 Bot Builder SDK의 주요 개념을 소개합니다.

## <a name="connector"></a>커넥터

[Bot Framework Connector](bot-builder-dotnet-connector.md)는 봇이 Skype, 메일, Slack과 같은 여러 채널을 통해 통신할 수 있는 단일 REST API를 제공합니다. 봇에서 채널로, 그리고 채널에서 봇으로 메시지를 릴레이하면 봇과 사용자 간에 쉽게 통신할 수 있습니다. 

.NET용 Bot Builder SDK에서는 [Connector][connectorLibrary] 라이브러리를 통해 Connector에 액세스할 수 있습니다. 

## <a name="activity"></a>활동

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

.NET용 Bot Builder SDK의 활동에 대한 자세한 내용은 [활동 개요](bot-builder-dotnet-activities.md)를 참조하세요.

## <a name="dialog"></a>대화 상자

.NET용 Bot Builder SDK를 사용하는 봇을 만들 경우 [대화 상자](bot-builder-dotnet-dialogs.md)를 사용하여 대화를 모델링하고 [대화 흐름](../bot-service-design-conversation-flow.md#dialog-stack)을 관리할 수 있습니다. 한 대화 상자를 다른 대화 상자로 구성하여 재사용을 최대화할 수 있고 대화 상자 컨텍스트로 특정 시점에 대화에서 활성 상태인 [대화 상자 스택](../bot-service-design-conversation-flow.md)을 유지 관리할 수 있습니다. 대화 상자를 구성하는 대화는 컴퓨터 간에 이식 가능하므로 봇 구현을 확장할 수 있습니다. 

.NET용 Bot Builder SDK에서 [Builder][builderLibrary] 라이브러리를 사용하여 대화 상자를 관리할 수 있습니다.

## <a name="formflow"></a>FormFlow

.NET용 Bot Builder SDK에서 [FormFlow](bot-builder-dotnet-formflow.md)를 사용하여 사용자의 정보를 수집하는 봇 빌드 과정을 간소화할 수 있습니다. 예를 들어, 샌드위치 주문을 받는 봇은 사용자로부터 빵 종류, 토핑 선택, 크기 등의 여러 가지 정보를 수집해야 합니다. 기본 지침을 제공하면 FormFlow가 자동으로 이와 같이 단계별 대화를 관리하는 데 필요한 대화 상자를 생성할 수 있습니다.

## <a name="state"></a>시스템 상태

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

.NET용 Bot Builder SDK를 사용하여 상태를 관리하는 방법에 대한 자세한 내용은 [상태 데이터 관리](bot-builder-dotnet-state.md)를 참조하세요.

## <a name="naming-conventions"></a>명명 규칙

.NET용 Bot Builder SDK 라이브러리는 강력한 형식의 파스칼식 대/소문자 명명 규칙을 사용합니다. 그러나 네트워크를 통해 앞뒤로 전송되는 JSON 메시지는 카멜식 대/소문자 명명 규칙을 사용합니다. 예를 들어, C# 속성 **ReplyToId**는 네트워크를 통해 전송되는 JSON 메시지에서 **replyToId**로 직렬화됩니다.

## <a name="security"></a>보안

봇의 엔드포인트는 Bot Framework Connector 서비스를 통해서만 호출할 수 있도록 해야 합니다. 이 항목에 대한 자세한 내용은 [봇 보호](bot-builder-dotnet-security.md)를 참조하세요.

## <a name="next-steps"></a>다음 단계

이제 모든 봇에 대한 개념을 알고 있습니다. 템플릿을 사용하면 [Visual Studio를 사용하여 빠르게 봇을 빌드](bot-builder-dotnet-quickstart.md)할 수 있습니다. 다음으로, 대화 상자부터 각 주요 개념을 자세히 살펴봅니다.

> [!div class="nextstepaction"]
> [.NET용 Bot Builder SDK의 대화 상자](bot-builder-dotnet-dialogs.md)

[connectorLibrary]: /dotnet/api/microsoft.bot.connector

[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs
