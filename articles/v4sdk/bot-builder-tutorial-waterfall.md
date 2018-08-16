---
title: 사용자 관련 질문하기 | Microsoft Docs
description: 폭포 모델을 사용하여 Bot Builder SDK에서 여러 입력을 사용자에게 요청하는 방법을 알아봅니다.
keywords: 폭포, 대화 상자, 질문하기, 프롬프트
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 62ff445e4aabf2afd41cc4bf1f15badb3f47e945
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304138"
---
# <a name="ask-the-user-questions"></a>사용자에게 질문하기

핵심은, 봇이 사용자와의 대화를 중심으로 빌드되었다는 것입니다. 대화는 [다양한 형식](bot-builder-conversations.md)을 취할 수 있습니다. 짧을 수도 있고, 더 복잡할 수도 있고, 질문하거나 질문에 답하는 형식일 수도 있습니다. 대화의 모양새는 몇 가지 요인에 따라 다르지만 모두 대화라는 범주에 포함됩니다.

이 자습서에서는 다중 턴 봇을 통해 간단한 질문에서 대화를 빌드하는 법을 안내합니다. 예제는 테이블 예약에 관한 것이지만, 주문을 하고 FAQ에 대답하고 예약을 하는 등 다중 턴 대화를 통해 다양한 항목을 수행하는 봇을 상상해 볼 수 있습니다.

대화형 채팅 봇은 특정 입력에 대해 사용자에게 응답하거나 사용자에게 질문할 수 있습니다. 이 자습서에서는 `Dialogs`의 일부분인 `Prompts` 라이브러리를 사용하여 사용자에게 질문하는 방법을 보여 줍니다. [대화 상자](../bot-service-design-conversation-flow.md)는 대화 구조를 정의하는 컨테이너로 생각할 수 있으며, 대화 상자 내 프롬프트는 자체 [방법 문서](bot-builder-prompts.md)에서 더 자세히 다루어집니다.

## <a name="prerequisite"></a>필수 요소

이 자습서의 코드는 [시작](~/bot-service-quickstart.md) 환경을 통해 만든 **기본 봇**에서 빌드됩니다.

## <a name="get-the-package"></a>패키지 가져오기

# <a name="ctabcstab"></a>[C#](#tab/cstab)

NuGet 패킷 관리자에서 **Microsoft.Bot.Builder.Dialogs** 패키지를 설치합니다.

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)
봇의 프로젝트 폴더로 이동하고 NPM에서 `botbuilder-dialogs` 패키지를 설치합니다.

```cmd
npm install --save botbuilder-dialogs@preview
```

---

## <a name="import-package-to-bot"></a>봇에 패키지 가져오기

# <a name="ctabcstab"></a>[C#](#tab/cstab)

봇 코드에서 대화 상자 및 프롬프트 모두에 참조를 추가합니다.

```cs
// ...
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
// ...
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

**app.js** 파일을 열고 봇 코드에 `botbuilder-dialogs` 라이브러리를 포함시킵니다.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

이렇게 하면 사용자 질문을 하는 데 사용하는 `DialogSet` 및 `Prompts` 라이브러리에 액세스할 수 있습니다. `DialogSet`은 대화 상자의 컬렉션일 뿐이며 여기서는 **폭포** 패턴으로 구조화합니다. 이는 단순히 하나의 대화가 다른 대화를 따르는 것을 의미합니다.

## <a name="instantiate-a-dialogs-object"></a>대화 상자 개체 인스턴스화

`dialogs` 개체를 인스턴스화합니다. 질문 및 답변 프로세스를 관리하는 데 이 대화 상자 개체를 사용하게 됩니다.

# <a name="ctabcstab"></a>[C#](#tab/cstab)
봇 클래스에서 멤버 변수를 선언하고 봇에 대한 생성자에서 이를 초기화합니다. 
```cs
public class MyBot : IBot
{
    private readonly DialogSet dialogs;
    public MyBot()
    {
        dialogs = new DialogSet();
    }
    // The rest of the class definition is omitted here
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```
---

## <a name="define-a-waterfall-dialog"></a>폭포 대화 상자 정의

질문을 하려면 최소한 두 단계 **폭포** 대화 상자가 필요합니다. 이 예제의 경우 두 단계 **폭포** 대화 상자를 구성하였습니다. 여기서 첫 번째 단계는 사용자에게 이름을 물어보고, 두 번째 단계는 사용자에게 이름을 사용하여 인사합니다. 

# <a name="ctabcstab"></a>[C#](#tab/cstab)

봇의 생성자를 수정하여 대화 상자를 추가합니다.
```csharp
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hello ${userName}!`);
        await dc.end(); // Ends the dialog
    }
]);
```

---

`Prompts` 라이브러리와 함께 제공되는 `textPrompt` 메서드를 사용하여 질문합니다. `Prompts` 라이브러리는 다양한 형식의 정보에 대해 사용자에게 물을 수 있도록 프롬프트 집합을 제공합니다. 다른 프롬프트 형식에 대한 자세한 내용은 [사용자에게 입력할 프롬프트 창 표시](~/v4sdk/bot-builder-prompts.md)를 참조하세요.

작업할 프롬프트의 경우 dialogId `textPrompt`를 사용하는 `dialogs` 개체에 프롬프트를 추가하고 `TextPrompt()` 생성자를 통해 이를 만들어야 합니다.

# <a name="ctabcstab"></a>[C#](#tab/cstab)

```cs
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
    // add the prompt, of type TextPrompt
    dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
}

```
사용자가 질문에 답변하면 응답은 2단계의 `args` 매개 변수에서 찾을 수 있습니다.

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```
사용자가 질문에 답변하면 응답은 2단계의 `results` 매개 변수에서 찾을 수 있습니다. 이 경우 `results`는 지역 변수 `userName`에 할당됩니다. 자습서의 뒷부분에서 사용자가 선택한 저장소에 사용자 입력을 유지하는 방법을 살펴보겠습니다.

---


질문할 `dialogs`를 정의했으므로 대화 상자에서 호출하여 프롬프트 프로세스를 시작해야 합니다.

## <a name="start-the-dialog"></a>대화 상자 시작

# <a name="ctabcstab"></a>[C#](#tab/cstab)

봇 논리를 다음과 같이 수정합니다.

```cs

public async Task OnTurn(ITurnContext context)
{
    // We'll cover state later, in the next tutorial
    var state = ConversationState<Dictionary<string, object>>.Get(context);
    var dc = dialogs.CreateContext(context, state);
    if (context.Activity.Type == ActivityTypes.Message)
    {
        await dc.Continue();
        
        if(!context.Responded)
        {
            await dc.Begin("greetings");
        }
    }
}
```

봇 논리는 `OnTurn()` 메서드에 포함됩니다. 사용자가 “Hi”라고 말하면 봇이 `greetings` 대화 상자를 시작합니다. `greetings` 대화 상자의 첫 번째 단계로 사용자에게 이름을 묻는 메시지가 표시됩니다. 사용자가 메시지 작업으로 이름과 함께 회신을 보내면 결과가 `dc.Continue()` 메서드를 통해 폭포의 2단계로 전송됩니다. 정의한 대로 폭포의 2단계는 사용자에게 이름을 사용하여 인사하고 대화 상자를 종료합니다. 

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

**기본** 봇의 `processActivity()` 메서드를 다음과 같이 수정합니다.

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi'.");
            }
        }
    });
});
```

봇 논리는 `processActivity()` 메서드 내에 포함됩니다. 사용자가 “Hi”라고 말하면 봇이 `greetings` 대화 상자를 시작합니다. `greetings` 대화 상자의 첫 번째 단계로 사용자에게 이름을 묻는 메시지가 표시됩니다. 사용자가 작업의 `text` 메시지로 자신의 이름과 함께 회신을 보냅니다. 메시지가 예상된 의도와 일치하지 않았고 봇이 사용자에게 회신을 보내지 않았으므로 결과가 `dc.continue()` 메서드를 통해 폭포의 2단계로 전송됩니다. 정의한 대로 폭포의 2단계는 사용자에게 이름을 사용하여 인사하고 대화 상자를 종료합니다. 예를 들어 2단계에서 사용자에게 인사말 메시지를 보내지 않았다면 `processActivity` 메서드는 사용자에게 전송된 *기본 메시지*를 통해 종료됩니다.

---



## <a name="define-a-more-complex-waterfall-dialog"></a>더 복잡한 폭포 대화 상자 정의

지금까지 폭포 대화 상자의 작동 원리 및 빌드 방법에 대해 다루었습니다. 테이블을 예약하기 위한 더 복잡한 대화 상자를 시도해 보겠습니다.

테이블 예약 요청을 관리하려면 **폭포** 대화 상자를 4단계로 정의해야 합니다. 이 대화에서는 `TextPrompt` 외에 `DatetimePrompt` 및 `NumberPrompt`를 추가로 사용합니다.



# <a name="ctabcstab"></a>[C#](#tab/cstab)

Echo Bot 템플릿을 사용하여 시작하고 봇 이름을 CafeBot으로 바꿉니다. `DialogSet` 및 일부 정적 멤버 변수를 추가합니다.

```cs

namespace CafeBot
{
    public class CafeBot : IBot
    {
        private readonly DialogSet dialogs;

        // Usually, we would save the dialog answers to our state object, which will be covered in a later tutorial.
        // For purpose of this example, let's use the three static variables to store our reservation information.
        static DateTime reservationDate;
        static int partySize;
        static string reservationName;

        // the rest of the class definition is omitted here
        // but is discussed in the rest of this article
    }
}
```

그런 다음, `reserveTable` 대화 상자를 정의합니다. 봇 클래스 생성자 내에 대화 상자를 추가할 수 있습니다.
```cs
public CafeBot()
{
    dialogs = new DialogSet();

    // Define our dialog
    dialogs.Add("reserveTable", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Context.SendActivity("Welcome to the reservation service.");

            await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
        },
        async(dc, args, next) =>
        {
            var dateTimeResult = ((DateTimeResult)args).Resolution.First();

            reservationDate = Convert.ToDateTime(dateTimeResult.Value);
            
            // Ask for next info
            await dc.Prompt("partySizePrompt", "How many people are in your party?");

        },
        async(dc, args, next) =>
        {
            partySize = (int)args["Value"];

            // Ask for next info
            await dc.Prompt("textPrompt", "Whose name will this be under?");
        },
        async(dc, args, next) =>
        {
            reservationName = args["Text"];
            string msg = "Reservation confirmed. Reservation details - " +
            $"\nDate/Time: {reservationDate.ToString()} " +
            $"\nParty size: {partySize.ToString()} " +
            $"\nReservation name: {reservationName}";
            await dc.Context.SendActivity(msg);
            await dc.End();
        }
    });

     // Add a prompt for the reservation date
     dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
     // Add a prompt for the party size
     dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
     // Add a prompt for the user's name
     dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
}
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

`reserveTable` 대화 상자는 다음과 같습니다.

```javascript
// Reserve a table:
// Help the user to reserve a table
var reservationInfo = {};

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
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;
        
        // Reservation confirmation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

`reserveTable` 대화 상자의 대화 흐름은 사용자에게 폭포의 처음 세 단계를 통해 3가지 질문을 합니다. 4단계는 마지막 질문에 대한 답변을 처리하고 사용자에게 예약 확인을 전송합니다.



# <a name="ctabcstab"></a>[C#](#tab/cstab)
`reserveTable` 대화 상자의 각 폭포 단계는 사용자에게 정보를 요청하는 프롬프트를 사용합니다. 다음 코드는 대화 상자 집합에 프롬프트를 추가하는 데 사용되었습니다.

```cs
dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

이 폭포를 작업하기 위해 이러한 프롬프트를 `dialogs` 개체에 추가해야 합니다.

```javascript
// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt());
```

---

이제 봇 논리에 이를 연결할 준비가 되었습니다.

## <a name="start-the-dialog"></a>대화 상자 시작

# <a name="ctabcstab"></a>[C#](#tab/cstab)
다음 코드를 포함하도록 봇의 `OnTurn`을 수정합니다.
```cs
public async Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // The type parameter PropertyBag inherits from 
        // Dictionary<string,object>
        var state = ConversationState<Dictionary<string, object>>.Get(context);
        var dc = dialogs.CreateContext(context, state);
        await dc.Continue();

        // Additional logic can be added to enter each dialog depending on the message received
        
        if(!context.Responded)
        {
            if (context.Activity.Text.ToLowerInvariant().Contains("reserve table"))
            {
                await dc.Begin("reserveTable");
            }
            else
            {
                await context.SendActivity($"You said '{context.Activity.Text}'");
            }
        }
    }
}
```


**Startup.cs**에서 `EchoState` 대신 `Dictionary<string,object>`에서 파생된 클래스를 사용하도록 ConversationState 미들웨어의 초기화를 변경합니다.

예를 들어 `Configure()`에서는 다음과 같습니다.
```cs
options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

이 요청에 대한 사용자 의도를 캡처하려면 코드 몇 줄을 `processActivity()` 메서드에 추가합니다. 봇의 `processActivity()` 메서드를 다음과 같이 수정합니다.

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
            else if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi' or 'Reserve table'.");
            }
        }
    });
});
```

실행하면 사용자가 `reserve table` 문자열이 포함된 메시지를 보낼 때마다 봇은 `reserveTable` 대화를 시작합니다.

---



## <a name="next-steps"></a>다음 단계

이 자습서에서 봇은 변수에 대한 사용자의 입력을 봇 내에 저장하고 있습니다. 이 정보를 저장하거나 유지하려는 경우 미들웨어 레이어에 상태를 추가해야 합니다. 다음 자습서에서 사용자 상태 데이터를 유지하는 방법에 대해 더 자세히 살펴보겠습니다. 

> [!div class="nextstepaction"]
> [사용자 데이터 유지](bot-builder-tutorial-persist-user-inputs.md)
