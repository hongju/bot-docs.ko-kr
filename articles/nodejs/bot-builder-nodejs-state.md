---
title: 상태 데이터 관리 | Microsoft Docs
description: Node.js용 Bot Framework SDK를 사용하여 상태 데이터를 저장 및 검색하는 방법을 알아봅니다.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6d653e47d2e906c6306134804c7731b374d830ba
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563975"
---
# <a name="manage-state-data"></a>상태 데이터 관리

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>메모리 내 데이터 저장소

메모리 내 데이터 저장소는 테스트 전용입니다. 이 저장소는 휘발성이며 임시입니다. 데이터는 봇이 다시 시작될 때마다 지워집니다. 테스트 목적으로 메모리 내 저장소를 사용하려면 두 가지를 수행해야 합니다. 먼저 메모리 내 저장소의 새 인스턴스를 만듭니다.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
```

그런 다음, 이것을 **UniversalBot**을 만들 때 봇으로 설정합니다.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..])
                    .set('storage', inMemoryStorage); // Register in-memory storage 
```

이 메서드를 사용하여 고유한 사용자 지정 데이터 저장소를 설정하거나 *Azure Extensions* 중 하나를 사용합니다.

## <a name="manage-custom-data-storage"></a>사용자 지정 데이터 저장소 관리

프로덕션 환경에서는 성능 및 보안상의 이유로 고유한 데이터 저장소를 구현하거나 다음과 같은 데이터 저장소 옵션 중 하나를 구현할 수 있습니다.

1. [Cosmos DB를 사용하여 상태 데이터 관리](bot-builder-nodejs-state-azure-cosmosdb.md)

2. [Table Storage를 사용하여 상태 데이터 관리](bot-builder-nodejs-state-azure-table-storage.md)

이러한 [Azure 확장](https://www.npmjs.com/package/botbuilder-azure) 옵션 중 하나를 사용하면 Node.js용 Bot Framework SDK를 통해 데이터를 설정 및 유지하는 메커니즘을 메모리 내 데이터 저장소와 동일하게 유지됩니다.

## <a name="storage-containers"></a>저장소 컨테이너 

Node.js용 Bot Framework SDK에서는 `session` 개체가 상태 데이터 저장을 위해 다음 속성을 노출합니다.

| 자산 | 범위 | 설명 |
| ---- | ---- | ---- |
| [`userData`][userDataURL] | 사용자 | 지정된 채널에서 사용자에 대해 저장되는 데이터를 포함합니다. 이 데이터는 여러 대화에서 유지됩니다. |
| [`privateConversationData`][privateConversationDataURL] | 대화 | 특정 채널의 특정 대화의 컨텍스트에서 사용자에 대해 저장된 데이터를 포함합니다. 이 데이터는 현재 사용자에게 개인적인 것이며 현재 대화에 대해서만 유지됩니다. 대화가 종료되거나 `endConversation`이 명시적으로 호출되면 속성이 지워집니다. |
| [`conversationData`][conversationDataURL] | 대화 | 특정 채널의 특정 대화의 컨텍스트에서 저장된 데이터를 포함합니다. 이 데이터는 대화에 참여하는 모든 사용자와 공유하며 현재 대화에 대해서만 유지됩니다. 대화가 종료되거나 `endConversation`이 명시적으로 호출되면 속성이 지워집니다. |
| [`dialogData`][dialogDataURL] | 대화 상자 | 현재 대화 상자에 대해 저장된 데이터만 포함합니다. 각 대화 상자는 이 속성의 자체 사본이 있습니다. 대화 스택에서 대화 상자가 제거되면 속성이 지워집니다. |

이러한 4가지 속성은 데이터 저장에 사용할 수 있는 4가지 데이터 저장소 컨테이너에 해당합니다. 데이터를 저장하는 데 사용할 속성은 저장하는 데이터에 적합한 범위, 저장하는 데이터의 특성, 원하는 데이터 지속 기간에 따라 결정됩니다. 예를 들어, 여러 대화에서 사용 가능해야 하는 사용자 데이터를 저장하려면 `userData` 속성을을 사용합니다. 일시적으로 대화 상자 범위 안에 로컬 변수 값을 저장하려면 `dialogData` 속성을 사용합니다. 여러 대화 상자에서 액세스할 수 있어야 하는 데이터를 임시 저장해야 할 경우 `conversationData` 속성을 사용합니다.

## <a name="data-persistence"></a>데이터 지속성

기본적으로 `userData`, `privateConversationData` 및 `conversationData` 속성을 사용하여 저장된 데이터는 대화 종료 후에도 유지되게 설정됩니다. 데이터를 `userData` 컨테이너에 유지하지 않으려면 `persistUserData` 플래그를 **false**로 설정합니다. 데이터를 `conversationData` 컨테이너에 유지하지 않으려면 `persistConversationData` 플래그를 **false**로 설정합니다. 

```javascript
// Do not persist userData
bot.set(`persistUserData`, false);

// Do not persist conversationData
bot.set(`persistConversationData`, false);
```

> [!NOTE]
> `privateConversationData` 컨테이너에 대해서는 데이터 지속성을 사용하지 않게 설정할 수 없으며 항상 유지됩니다.

## <a name="set-data"></a>데이터 설정

저장소 컨테이너에 직접 저장하여 간단한 JavaScript 개체를 저장할 수 있습니다. `Date` 같은 복잡한 개체는 `string`으로 변환하는 것이 좋습니다. 상태 데이터는 직렬화되어 JSON으로 저장되기 때문입니다. 다음 코드 샘플은 기본 데이터, 배열, 개체 맵 및 복합 `Date` 개체를 저장하는 방법을 보여 줍니다. 

**기본 데이터 저장**

```javascript
session.userData.userName = "Kumar Sarma";
session.userData.userAge = 37;
session.userData.hasChildren = true;
```

**배열 저장**

```javascript
session.userData.profile = ["Kumar Sarma", "37", "true"];
```

**개체 맵 저장**

```javascript
session.userData.about = {
    "Profile": {
        "Name": "Kumar Sarma",
        "Age": 37,
        "hasChildren": true
    },
    "Job": {
        "Company": "Contoso",
        "StartDate": "June 8th, 2010",
        "Title": "Developer"
    }
}
```
**날짜 및 시간 저장** 

복잡한 JavaScript 개체의 경우 저장소 컨테이너에 저장하기 전에 문자열로 변환합니다. 

```javascript 
var startDate = builder.EntityRecognizer.resolveTime([results.response]); 

// Date as string: "2017-08-23T05:00:00.000Z" 
session.userdata.start = startDate.toISOString(); 
``` 

### <a name="saving-data"></a>데이터 저장

각 저장소 컨테이너에서 만들어진 데이터는 컨테이너를 저장할 때까지 메모리에 유지됩니다. Node.js용 Bot Framework SDK는 보낼 메시지가 있을 때 저장될 수 있게 `ChatConnector` 서비스에 일괄 처리로 데이터를 보냅니다. 메시지를 보내지 않고 저장소 컨테이너에 있는 데이터를 저장하려면 수동으로 [`save`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#save) 메서드를 호출할 수 있습니다. `save` 메서드를 호출하지 않을 경우 저장소 컨테이너에 있는 데이터가 일괄 처리의 일부로 유지됩니다.

```javascript
session.userData.favoriteColor = "Red";
session.userData.about.job.Title = "Senior Developer"; 
session.save();
```

## <a name="get-data"></a>데이터 가져오기

특정 저장소 컨테이너에 저장된 데이터에 액세스하려면 간단히 해당 속성을 참조합니다. 다음 코드 샘플은 이전에 기본 데이터로 저장된 데이터, 배열, 개체 맵 및 복합 Date 개체에 액세스하는 방법을 보여 줍니다.

**기본 데이터 액세스**

```javascript
var userName = session.userData.userName;
var userAge = session.userData.userAge;
var hasChildren = session.userData.hasChildren;
```

**배열 액세스**

```javascript
var userProfile = session.userData.userProfile;

session.send("User Profile:");
for(int i = 0; i < userProfile.length, i++){
    session.send(userProfile[i]);
}
```

**개체 맵 액세스**

```javascript
var about = session.userData.about;

session.send("User %s works at %s.", about.Profile.Name, about.Job.Company);
```

**Date 개체 액세스** 

문자열로 날짜 데이터를 검색한 다음, JavaScript의 Date 개체로 변환합니다. 

```javascript 
// startDate as a JavaScript Date object. 
var startDate = new Date(session.userdata.start); 
``` 

## <a name="delete-data"></a>데이터 삭제

기본적으로 `dialogData` 컨테이너에 저장된 데이터는 대화 상자가 대화 스택에서 제거되면 지워집니다. 마찬가지로 `conversationData` 컨테이너 및 `privateConversationData` 컨테이너에 저장된 데이터는 `endConversation` 메서드가 호출되면 지워집니다. 그러나 `userData` 컨테이너에 저장된 데이터를 삭제하려면 명시적으로 지워야 합니다.

저장소 컨테이너에 저장된 데이터를 명시적으로 지우려면 간단히 다음 코드 샘플에서처럼 컨테이너를 재설정합니다. 

```javascript
// Clears data stored in container.
session.userData = {}; 
session.privateConversationData = {};
session.conversationData = {};
session.dialogData = {};
```

데이터 컨테이너를 `null`로 설정하거나 `session` 개체에서 제거하면 안 됩니다. 그러면 다음에 컨테이너 액세스를 시도할 때 오류가 발생합니다. 또한 이전에 지속된 해당하는 모든 데이터를 지우기 위해 메모리에서 수동으로 컨테이너를 지운 후 `session.save();`를 수동으로 호출하려 할 수 있습니다.

## <a name="next-steps"></a>다음 단계

사용자 상태 데이터 관리 방법을 이해했으므로 이 상태 데이터를 사용하여 대화 흐름을 보다 잘 관리하는 방법을 알아보겠습니다.

> [!div class="nextstepaction"]
> [대화 흐름 관리](bot-builder-nodejs-dialog-manage-conversation-flow.md)

## <a name="additional-resources"></a>추가 리소스
- [사용자에게 입력 프롬프트](bot-builder-nodejs-dialog-prompt.md)

[userDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
[conversationDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#conversationdata
[privateConversationDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#privateconversationdata
[dialogDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#dialogdata

[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
