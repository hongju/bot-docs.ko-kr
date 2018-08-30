---
title: 자습서 - 사용자 상태 데이터 저장 | Microsoft Docs
description: Bot Builder SDK에서 사용자 상태 데이터를 저장하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 86f70fd66f1bc2261339cbe0590061913b51ddbc
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904441"
---
# <a name="save-user-state-data"></a>사용자 상태 데이터 저장

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇에서 사용자에게 입력을 요청하는 경우, 일부 정보를 어떤 형태의 저장소에 유지하려는 경우가 있습니다. Bot Builder SDK를 사용하면 메모리 내 저장소, 파일 저장소, *CosmosDB* 또는 *SQL*과 같은 데이터베이스 저장소를 사용하여 사용자 입력을 저장할 수 있습니다. 

이 자습서에서는 미들웨어 계층에 저장소 개체를 정의하고 저장소 개체에 사용자를 저장하여 지속될 수 있도록 만드는 방법을 보여줍니다.

## <a name="prequisite"></a>필수 구성 요소 

이 자습서는 [waterfall을 통해 대화 흐름 관리](bot-builder-tutorial-waterfall.md) 자습서를 기반으로 합니다.

## <a name="add-storage-to-middleware-layer"></a>미들웨어 계층에 저장소 추가


## <a name="save-user-input-to-storage"></a>저장소에 사용자 입력 저장

테이블 예약 대화를 관리하려면, **waterfall** dialog를 4단계로 정의해야 합니다. 이 대화에는 `DatetimePrompt` 및 `NumberPrompt` 외에 `TextPrompt`로 사용합니다.

`reserveTable` dialog는 다음과 같은 모양입니다.

```javascript
// Reserve a table:
// Help the user to reserve a table

var reservationInfo = {
    dateTime: '',
    partySize: '',
    reserveName: ''
}

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");
        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);

```

이것을 봇 논리에 연결할 준비가 되었습니다.

## <a name="start-the-dialog"></a>dialog 시작

봇의 `processActivity()` 메서드를 다음과 같이 수정합니다.

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // State will store all of your information 
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greeting');
            }
            if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
            else{
                // Continue executing the "current" dialog, if any.
                await dc.continue();
            }
        }
    });
});
```

실행하면 사용자가 `reserve table` 문자열이 포함된 메시지를 보낼 때마다 봇은 `reserveTable` 대화를 시작합니다.

## <a name="next-steps"></a>다음 단계

??? 

> [!div class="nextstepaction"]
> [사용자 상태 데이터 저장](bot-builder-tutorial-save-data.md)
