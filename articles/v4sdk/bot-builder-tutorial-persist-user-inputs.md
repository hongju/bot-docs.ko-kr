---
title: 사용자 데이터 유지 | Microsoft Docs
description: Bot Builder SDK의 저장소에 사용자 상태 데이터를 저장하는 방법을 알아봅니다.
keywords: 사용자 데이터 유지, 저장소, 대화 데이터
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 28377a1e611464012df28d3edf78d1cf01351345
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928371"
---
# <a name="persist-user-data"></a>사용자 데이터 유지

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇에서 사용자에게 입력을 요청하는 경우 일부 형태의 저장소에 일부 정보를 유지하려는 경우가 있습니다. Bot Builder SDK를 통해 *메모리 내 저장소*, *파일 저장소* 또는 *CosmosDB* 또는 *SQL*과 같은 데이터베이스 저장소를 사용하여 사용자 입력을 저장할 수 있습니다. 여기서 로컬 저장소 유형은 주로 테스트 및 프로토타입에 사용되고 뒤의 저장소 유형은 프로덕션 봇에 가장 적합합니다.

이 자습서에서는 저장소 개체를 정의하고 사용자 입력이 유지될 수 있도록 저장소 개체에 저장하는 방법을 보여줍니다. 

> [!NOTE]
> 사용하도록 선택한 저장소 유형에 관계없이 데이터 연결 및 유지를 위한 프로세스는 동일합니다. 이 자습서에서는 데이터를 유지할 저장소로 `FileStorage`를 사용합니다.
> 상태 및 기타 저장소 유형에 대한 자세한 내용은 [대화 및 사용자 상태 관리](bot-builder-howto-v4-state.md)를 참조하세요.

## <a name="prequisite"></a>필수 구성 요소 

이 자습서는 [테이블 예약](bot-builder-tutorial-waterfall.md) 자습서를 기본으로 합니다. 해당 자습서에서는 사용자에게 식당에서 테이블 예약에 대한 세 가지 정보를 요청하는 봇을 빌드합니다. 그러나 사용자의 입력은 유지되지 않았습니다. 이 자습서는 해당 봇에 데이터 저장소 지속성을 추가합니다.

## <a name="add-storage-to-middleware-layer"></a>미들웨어 계층에 저장소 추가

Bot Builder V4 SDK는 상태 관리자 미들웨어를 통해 상태 및 저장소를 처리합니다. 미들웨어는 기본 저장소의 형식과 관계없이 간단한 키-값 저장소를 사용하여 속성에 액세스할 수 있는 추상화 계층을 제공합니다. 상태 관리자는 기본 저장소 유형이 메모리 내, 파일 저장소 또는 Azure 테이블 저장소인지 여부에 따라 저장소에 데이터 작성 및 동시성 관리를 담당합니다. 이 자습서에서는 `FileStorage`를 사용하여 사용자 입력을 유지합니다.

`FileStorage` 공급자는 `bot-builder` 패키지와 함께 제공됩니다. 함께 시작한 샘플은 `MemoryStorage` 공급자를 사용합니다. 해당 저장소 유형은 봇이 다시 시작되면 삭제되는 봇의 휘발성 *메모리 내*를 사용합니다. 반대로 `FileStorage` 라이브러리는 데이터베이스와 유사하게 동작합니다. 즉, 이 라이브러리는 로컬 컴퓨터의 파일에 저장소 정보를 작성합니다. 나중에 검사할 수 있도록 이 저장소 파일을 넣을 위치를 지정할 수 있습니다.

`FileStorage`를 사용하려면 `conversationState` 개체가 정의된 봇에서 명령문을 찾고 `new botbuilder.FileStorage("c:/temp")`를 만들도록 업데이트합니다. 또한 이 저장소 파일이 작성되어야 하는 위치를 정의할 수 있습니다. 이렇게 하면 쉽게 찾아서 유지될 콘텐츠를 검사할 수 있습니다.

# <a name="ctabcstab"></a>[C#](#tab/cstab)
```cs
var storage = new FileStorage("c:/temp");

// These two classes are simply Dictionaries to store state
options.Middleware.Add(new ConversationState<MyBot.convoState>(storage));
options.Middleware.Add(new UserState<MyBot.userState>(storage));
```

Bot Builder SDK는 선택할 수 있는 다양한 범위로 세 가지 상태 개체를 제공합니다.

| 상태 | 범위 | 설명 |
| ---- | ---- | ---- |
| `dc.ActiveDialog.State` | 대화 상자 | 폭포 대화 상자의 단계에 사용할 수 있는 상태입니다. |
| `ConversationState` | 대화 | 현재 대화에 사용할 수 있는 상태입니다. |
| `UserState` | 사용자 | 여러 대화에 사용할 수 있는 상태입니다. |

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

**app.js**
```javascript
// Storage
const storage = new FileStorage("c:/temp");
const convoState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(convoState, userState));
```

`BotStateSet`는 `ConversationState` 및 `UserState` 모두를 동시에 관리할 수 있습니다. 사용자 데이터를 저장할 때 선택할 수 있습니다. Bot Builder SDK는 선택할 수 있는 다양한 범위로 세 가지 상태 개체를 제공합니다.

| 상태 | 범위 | 설명 |
| ---- | ---- | ---- |
| `dc.activeDialog.state` | 대화 상자 | 폭포 대화 상자의 단계에 사용할 수 있는 상태입니다. |
| `ConversationState` | 대화 | 현재 대화에 사용할 수 있는 상태입니다. |
| `UserState` | 사용자 | 여러 대화에 사용할 수 있는 상태입니다. |

---
 

## <a name="persist-state"></a>상태 유지

봇의 경우 저장하려는 항목 및 유지해야 하는 시간에 따라 세 개의 상태 위치 중 하나에 작성할 수 있습니다.  

# <a name="ctabcstab"></a>[C#](#tab/cstab)

*상태 관리자 미들웨어*는 각 순서의 끝에서 상태 작성을 처리합니다. 따라서 봇에서 수행해야 하는 작업은 원하는 상태 개체에 데이터를 할당하는 것입니다. 이 예제에서는 `dc.ActiveDialog.State`를 사용하여 예약에 대한 사용자 입력을 추적합니다. 이러한 방식으로 사용자 입력을 글로벌 변수에 저장하는 대신 대화 상자에 대한 임시 저장소 개체 범위에 저장할 수 있습니다. 이 개체는 대화 상자가 활성화되는 한 존재하며, 더 길게 유지하려는 경우 다른 상태 개체 중 하나에 전송해야 합니다. 이 경우 예약 `msg`를 폭포의 마지막 단계에서 대화 상태에 할당합니다.

```cs
dialogs.Add("reserveTable", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt for the guest's name.
        await dc.Context.SendActivity("Welcome to the reservation service.");

        dc.ActiveDialog.State = new Dictionary<string, object>();

        await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
    },
    async(dc, args, next) =>
    {
        var dateTimeResult = ((DateTimeResult)args).Resolution.First();

        dc.ActiveDialog.State["date"] = Convert.ToDateTime(dateTimeResult.Value);
        
        // Ask for next info
        await dc.Prompt("partySizePrompt", "How many people are in your party?");

    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["partySize"] = (int)args["Value"];

        // Ask for next info
        await dc.Prompt("textPrompt", "Who's name will this be under?");
    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["name"] = args["Text"];
        string msg = "Reservation confirmed. Reservation details - " +
        $"\nDate/Time: {dc.ActiveDialog.State["date"].ToString()} " +
        $"\nParty size: {dc.ActiveDialog.State["partySize"].ToString()} " +
        $"\nReservation name: {dc.ActiveDialog.State["name"]}";

        var convo = ConversationState<convoState>.Get(dc.Context);

        // In production, you may want to store something more helpful
        convo[$"{dc.ActiveDialog.State["name"]} reservation"] = msg;

        await dc.Context.SendActivity(msg);
        await dc.End();
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

*상태 관리자 미들웨어*는 각 순서의 끝에서 파일에 상태 작성을 처리합니다. 따라서 봇에서 수행해야 하는 작업은 원하는 상태 개체에 데이터를 할당하는 것입니다. 이 예제에서는 `dc.activeDialog.state`를 사용하여 `reservervationInfo` 개체에서 사용자 입력을 추적합니다. 이러한 방식으로 사용자 입력을 글로벌 변수에 저장하는 대신 대화 상자에 대한 임시 저장소 개체 범위에 저장할 수 있습니다. 이 개체는 대화 상자가 활성화되는 한 존재하므로 유지하려는 경우 다른 상태 개체 중 하나에 전송해야 합니다. 이 경우 `reservationInfo`를 폭포의 마지막 단계에서 `convo` 상태에 할당합니다.

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        dc.activeDialog.state.reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.reserveName = result;
        
        // Persist data
        var convo = conversationState.get(dc.context);; // conversationState.get(dc.context);
        convo.reservationInfo = dc.activeDialog.state.reservationInfo;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${dc.activeDialog.state.reservationInfo.dateTime} 
            <br/>Party size: ${dc.activeDialog.state.reservationInfo.partySize} 
            <br/>Reservation name: ${dc.activeDialog.state.reservationInfo.reserveName}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

이제 봇 논리에 연결할 준비가 되었습니다.

## <a name="start-the-dialog"></a>대화 상자 시작

여기에서 변경해야 하는 코드 변경 내용이 없습니다. 단순히 봇을 실행하고 메시지를 전송하여 `reserveTable` 대화를 시작합니다.

## <a name="check-file-storage-content"></a>파일 저장소 콘텐츠 확인

봇을 실행하고 `reserveTable` 대화를 완료한 후 지정한 위치에서 파일에 저장된 정보를 찾습니다(예: "C:/temp"). 파일 이름 앞에 "conversation!" 또는 "user!"가 추가됩니다. 최신 항목을 더 쉽게 찾을 수 있도록 날짜별로 파일을 정렬하도록 도울 수 있습니다.

## <a name="next-steps"></a>다음 단계

이제 사용자 입력을 저장하는 방법을 배웠으므로 프롬프트 라이브러리를 사용하여 사용자에게 요청할 수 있는 입력의 유형을 살펴봅시다.

> [!div class="nextstepaction"]
> [사용자에게 입력할 프롬프트 창 표시](~/v4sdk/bot-builder-prompts.md)
