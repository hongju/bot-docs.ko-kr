---
title: 상태 데이터 관리 | Microsoft Docs
description: .NET용 Bot Builder SDK를 사용하여 상태 데이터를 저장 및 검색하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: deb8361a5cca2f37840abb1180c2de571ee08143
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999760"
---
# <a name="manage-state-data"></a>상태 데이터 관리

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>메모리 내 데이터 저장소

메모리 내 데이터 저장소는 테스트 전용입니다. 이 저장소는 휘발성이며 임시입니다. 데이터는 봇이 다시 시작될 때마다 지워집니다. 테스트 목적으로 메모리 내 저장소를 사용하려면 다음이 필요합니다. 

다음 NuGet 패키지를 설치합니다. 
- Microsoft.Bot.Builder.Azure
- Autofac.WebApi2

**Application_Start** 메서드에서 메모리 내 저장소의 새 인스턴스를 만들고 새 데이터 저장소를 등록합니다.

```cs
// Global.asax file

var store = new InMemoryDataStore();

Conversation.UpdateContainer(
           builder =>
           {
               builder.Register(c => store)
                         .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                         .AsSelf()
                         .SingleInstance();

               builder.Register(c => new CachingBotDataStore(store,
                          CachingBotDataStoreConsistencyPolicy
                          .ETagBasedConsistency))
                          .As<IBotDataStore<BotData>>()
                          .AsSelf()
                          .InstancePerLifetimeScope();


           });
GlobalConfiguration.Configure(WebApiConfig.Register);

```

이 메서드를 사용하여 고유한 사용자 지정 데이터 저장소를 설정하거나 *Azure 확장* 중 하나를 사용합니다.

## <a name="manage-custom-data-storage"></a>사용자 지정 데이터 저장소 관리

프로덕션 환경에서는 성능 및 보안상의 이유로 고유한 데이터 저장소를 구현하거나 다음과 같은 데이터 저장소 옵션 중 하나를 구현할 수 있습니다.

1. [Cosmos DB를 사용하여 상태 데이터 관리](bot-builder-dotnet-state-azure-cosmosdb.md)

2. [Table Storage를 사용하여 상태 데이터 관리](bot-builder-dotnet-state-azure-table-storage.md)

이러한 [Azure 확장](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/) 옵션 중 하나를 사용하면 .NET용 Bot Framework SDK를 통해 데이터를 설정 및 유지하는 메커니즘을 메모리 내 데이터 저장소와 동일하게 유지됩니다.

## <a name="bot-state-methods"></a>봇 상태 메서드

다음 표에는 상태 데이터를 관리하는 데 사용할 수 있는 메서드가 나와 있습니다.

| 방법 | 범위 | Objective |                                                
|----|----|----|
| `GetUserData` | 사용자 | 특정 채널의 사용자에 대해 이전에 저장된 상태 데이터를 가져옵니다. |
| `GetConversationData` | 대화 | 특정 채널의 대화에 대해 이전에 저장된 상태 데이터를 가져옵니다. |
| `GetPrivateConversationData` | 사용자 및 대화 | 특정 채널의 대화 내 사용자에 대해 이전에 저장된 상태 데이터를 가져옵니다. |
| `SetUserData` | 사용자 | 지정된 채널에 사용자에 대한 상태 데이터를 저장합니다. |
| `SetConversationData` | 대화 | 지정된 채널에 대화에 대한 상태 데이터를 저장합니다. <br/><br/>**참고**: `DeleteStateForUser` 메서드는 `SetConversationData` 메서드를 사용하여 저장된 데이터를 삭제하지 않으므로 사용자의 PII(개인 식별 정보)를 저장할 때는 이 메서드를 사용하지 않아야 합니다. |
| `SetPrivateConversationData` | 사용자 및 대화 | 지정된 채널에 대화 내 사용에 대한 상태 데이터를 저장합니다. |
| `DeleteStateForUser` | 사용자 | `SetUserData` 메서드 또는 `SetPrivateConversationData` 메서드를 사용하여 이전에 저장된 사용자에 대한 상태 데이터를 삭제합니다. <br/><br/>**참고**: 봇은 [deleteUserData](bot-builder-dotnet-activities.md#deleteuserdata) 형식의 활동 또는 이나 사용자의 연락처 목록에서 봇이 제거되었음을 나타내는 [contactRelationUpdate](bot-builder-dotnet-activities.md#contactrelationupdate) 형식의 활동을 수신할 경우 이 메서드를 호출해야 합니다. |

봇에서 "**Set...Data**" 메서드 중 하나를 사용하여 상태 데이터를 저장하는 경우 같은 컨텍스트에서 봇이 수신하는 이후 메시지에는 해당 데이터가 포함되며, 봇은 해당 "**Get...Data**" 메서드를 사용하여 이 데이터에 액세스할 수 있습니다.

## <a name="useful-properties-for-managing-state-data"></a>상태 데이터를 관리하는 데 유용한 속성

각 [Activity][Activity] 개체에는 상태 데이터를 관리하는 데 사용하는 속성이 포함되어 있습니다.

| 자산 | 설명 | 사용 사례 |
|----|----|----|
| `From` | 채널에서 사용자를 고유하게 식별 | 사용자와 연결된 상태 데이터 저장 및 검색 |
| `Conversation` | 대화를 고유하게 식별 | 대화와 연결된 상태 데이터 저장 및 검색 |
| `From` 및 `Conversation` | 사용자 및 대화를 고유하게 식별 | 특정 대화의 컨텍스트 내에서 특정 사용자와 연결된 상태 데이터 저장 및 검색 |

> [!NOTE]
> Bot Framework 상태 데이터 저장소를 사용하지 않고, 자체 데이터베이스에 상태 데이터를 저장하도록 선택하는 경우에도 이러한 속성 값을 키로 사용할 수 있습니다.

## <a id="state-client"></a> 상태 클라이언트 만들기

`StateClient` 개체를 사용하여 .NET용 Bot Builder SDK에서 상태 데이터를 관리할 수 있습니다. 상태 데이터를 관리하려는 동일한 컨텍스트에 속하는 메시지에 대해 액세스 권한이 있는 경우 `Activity` 개체에 대해 `GetStateClient` 메서드를 호출하여 상태 클라이언트를 만들 수 있습니다.

[!code-csharp[Get State client](../includes/code/dotnet-state.cs#getStateClient1)]

상태 데이터를 관리하려는 동일한 컨텍스트에 속하는 메시지에 대해 액세스 권한이 없는 경우 `StateClient` 클래스의 새 인스턴스를 만들어 간단히 상태 클라이언트를 만들 수 있습니다. 이 예제에서 `microsoftAppId` 및 `microsoftAppPassword`는 [봇 만들기](../bot-service-quickstart.md) 프로세스 동안 봇에 필요한 Bot Framework 인증 자격 증명입니다.

> [!NOTE]
> 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

[!code-csharp[Get State client](../includes/code/dotnet-state.cs#getStateClient2)]

> [!NOTE]
> 기본 상태 클라이언트는 중앙 서비스에 저장됩니다. 일부 채널의 경우, 채널 자체 내에 호스트되는 상태 API를 사용하려고 할 수 있으므로 상태 데이터는 채널이 제공하는 호환 저장소에 저장될 수 있습니다.

## <a name="get-state-data"></a>상태 데이터 가져오기

각 "**Get...Data**" 메서드는 지정된 사용자 및/또는 대화에 대한 상태 데이터를 포함하는 `BotData` 개체를 반환합니다. `BotData` 개체에서 특정 속성 값을 가져오려면 `GetProperty` 메서드를 호출합니다. 

다음 코드 예제에서는 사용자 데이터에서 형식이 지정된 속성을 가져오는 방법을 보여 줍니다. 

[!code-csharp[Get state property](../includes/code/dotnet-state.cs#getProperty1)]

다음 코드 예제에서는 사용자 데이터 내의 복합 형식에서 속성을 가져오는 방법을 보여 줍니다.

[!code-csharp[Get state property](../includes/code/dotnet-state.cs#getProperty2)]

"**Get...Data**" 메서드 호출에 대해 지정된 사용자 및/또는 대화의 상태 데이터가 없는 경우 반환된 `BotData` 개체에 다음 속성 값이 포함됩니다. 
- `BotData.Data` = null
- `BotData.ETag` = "*"

## <a name="save-state-data"></a>상태 데이터 저장

상태 데이터를 저장하려면 먼저 해당 "**Get...Data**" 메서드를 호출하여 `BotData` 개체를 가져온 후 업데이트하려는 각 속성에 대해 `SetProperty` 메서드를 호출하여 업데이트하고, 해당 "**Set...Data**" 메서드를 호출하여 저장합니다. 

> [!NOTE]
> 채널에 있는 각 사용자, 채널에 있는 각 대화 및 채널에 있는 대화 컨텍스트 내의 각 사용자에 대한 최대 32KB의 데이터를 저장할 수 있습니다. 

다음 코드 예제에서는 사용자 데이터에서 형식이 지정된 속성을 저장하는 방법을 보여 줍니다.

[!code-csharp[Set state property](../includes/code/dotnet-state.cs#setProperty1)]

다음 코드 예제에서는 사용자 데이터 내의 복합 형식에서 속성을 저장하는 방법을 보여 줍니다. 

[!code-csharp[Set state property](../includes/code/dotnet-state.cs#setProperty2)]

## <a name="handle-concurrency-issues"></a>동시성 문제 처리

봇은 봇의 다른 인스턴스가 데이터를 변경한 경우 상태 데이터를 저장하려고 할 때 HTTP 상태 코드 **412 전제 조건 실패**의 오류 응답을 수신할 수 있습니다. 다음 코드 예제에 나와 있는 것처럼 이 시나리오를 고려하도록 봇을 디자인할 수 있습니다.

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>추가 리소스

- [Bot Framework 문제 해결 가이드](../bot-service-troubleshoot-general-problems.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Builder SDK 참조</a>

[Activity]: https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
