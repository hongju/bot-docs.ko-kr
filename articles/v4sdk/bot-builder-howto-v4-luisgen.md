---
title: 입력한 LUIS 결과 추출 | Microsoft Docs
description: LUIS를 사용하여 Bot Builder SDK에서 엔터티를 추출하는 방법을 알아봅니다.
keywords: 의도, 엔터티, LUISGen, 추출
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/16/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6a88b0a7f44f43d0676ba88314fbba7c486e6be4
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301603"
---
# <a name="extract-intents-and-entities-using-luisgen"></a>LUISGen을 사용하여 의도 및 엔터티 추출

의도를 인식하는 것 외에, LUIS 앱은 사용자 요청 이행에 중요한 단어에 해당하는 엔터티도 추출할 수 있습니다. 예를 들어, 레스토랑 예약 예제에서 LUIS 앱은 사용자의 메시지에서 파티 규모, 예약 날짜 또는 식당 위치를 추출할 수 있습니다. 


[LUISGen 도구](https://github.com/Microsoft/botbuilder-tools/tree/master/LUISGen)를 사용하여 봇의 코드에서 LUIS로부터 엔터티를 보다 쉽게 추출하도록 하는 클래스를 생성합니다.

Node.js 명령줄에서 전역 경로에 `luisgen`을 설치합니다.
```
npm install -g luisgen
```

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="generate-a-luis-results-class"></a>LUIS 결과 클래스 생성

[CafeBot LUIS 샘플](https://aka.ms/contosocafebot-luis)을 다운로드하고 해당 루트 폴더에서 LUISGen을 실행합니다.

```
luisgen Assets\LU\models\LUIS\cafeLUISModel.json -cs ContosoCafeBot.CafeLUISModel
```

## <a name="examine-the-generated-code"></a>생성된 코드 검사
이렇게 하면 프로젝트에 추가할 수 있는 **cafeLUISModel.cs**가 생성됩니다. LUIS에서 강력한 형식의 결과를 가져오기 위한 `cafeLuisModel` 클래스를 제공합니다.

이 클래스는 LUIS 앱에 정의된 의도를 가져오기 위한 열거형을 포함합니다.
```cs
public enum Intent {
    Book_Table, 
    Greeting, 
    None, 
    Who_are_you_intent
};
```
또한 `Entities` 속성도 포함합니다. 사용자의 메시지에서 한 엔터티가 여러 번 발생할 수 있으므로 `_Entities` 클래스는 엔터티의 각 형식에 대한 배열을 정의합니다. 
```cs
public class _Entities
{
    // Simple entities
    public string[] partySize;

    // Built-in entities
    public Microsoft.Bot.Builder.Ai.LUIS.DateTimeSpec[] datetime;
    public double[] number;

    // Lists
    public string[][] cafeLocation;

    // Instance
    public class _Instance
    {
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] partySize;
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] datetime;
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] number;
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] cafeLocation;
    }
    [JsonProperty("$instance")]
    public _Instance _instance;
}
public _Entities Entities;
```

> [!NOTE]
> LUIS는 사용자의 발언에서 지정된 형식을 갖는 둘 이상의 엔터티를 검색할 수 있으므로 모든 엔터티 형식은 배열입니다. 예를 들어, 사용자가 "make reservations for 5pm tomorrow and 9pm next Saturday"라고 말할 경우 `datetime` 결과에서 "5pm tomorrow"와 "9pm next Saturday"가 둘 다 반환됩니다.
>

|엔터티 | type | 예 | 메모 |
|-------|-----|------|---|
|partySize| string[]| Party of `four`| 단순 엔터티는 문자열을 인식합니다. 이 예제에서 Entities.partySize[0]는 `"four"`입니다.
|Datetime| [DateTimeSpec](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.datetimespec?view=botbuilder-4.0.0-alpha)[]| reservation for at `9pm tomorrow`| 각 **DateTimeSpec** 개체에는 가능한 시간 값이 **timex** 형식으로 지정되는 timex 필드가 있습니다. timex에 대한 자세한 내용은 http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3을 참조하세요. 인식을 수행하는 라이브러리에 대한 자세한 내용은 https://github.com/Microsoft/Recognizers-Text를 참조하세요.
|number| double[]| Party of `four` which includes `2` children | `number`는 파티의 규모 뿐만 아니라 모든 명 수를 식별합니다. <br/> 발언 "Party of four which includes 2 children"에서 `Entities.number[0]`은 4이고 `Entities.number[1]`은 2입니다다.
|cafelocation| string[][] | Reservation at the `Seattle` location.| cafeLocation은 목록 엔터티로, 인식된 목록 멤버를 포함함을 의미합니다. 인식할 수 있는 엔터티가 둘 이상의 목록으로 이루어진 멤버인 경우 배열의 배열입니다. 예를 들어, "reservation in Washington"은 워싱턴 주 및 워싱턴 D.C.에 대한 목록에 해당할 수 있습니다.

`_Instance` 속성은 [InstanceData](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.instancedata?view=botbuilder-4.0.0-alpha)에 인식된 각 엔터티에 대한 자세한 정보를 제공합니다.

## <a name="check-intents-in-your-bot"></a>봇의 의도 확인
**CafeBot.cs**에서 `OnTurn` 내의 코드를 살펴보세요. 봇이 LUIS를 호출하고, 의도를 확인하여 시작할 다이얼로그를 결정하는 위치를 확인할 수 있습니다. [`Recognize`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer?view=botbuilder-4.0.0-alpha#methods) 호출에서 얻은 LUIS 결과는 `BookTable` 다이얼로그에 인수로 전달됩니다.



```cs
if(!context.Responded)
{
    // call LUIS and get results
    LuisRecognizer lRecognizer = createLUISRecognizer();
    // Use the generated class as the type parameter to Recognize()
    cafeLUISModel lResult = await lRecognizer.Recognize<cafeLUISModel>(utterance, ct);
    Dictionary<string,object> lD = new Dictionary<string,object>();
    if(lResult != null) {
        lD.Add("luisResult", lResult);
    }
    
    // top level dispatch
    switch (lResult.TopIntent().intent)
    {
        case cafeLUISModel.Intent.Greeting:
            await context.SendActivity("Hello!");
            if (userState.sendCards) await context.SendActivity(CreateCardResponse(context.Activity, createWelcomeCardAttachment()));
            break;

        case cafeLUISModel.Intent.Book_Table:
            await dc.Begin("BookTable", lD);
            break;

        case cafeLUISModel.Intent.Who_are_you_intent:
            await context.sendActivity("I'm the Contoso Cafe bot.");
            break;

        case cafeLUISModel.Intent.None:
        default:
            await getQnAResult(context);
            break;
    }
}
```

## <a name="extract-entities-in-a-dialog"></a>다이얼로그에서 엔터티 추출

이제 `Dialogs/BookTable.cs`를 살펴보겠습니다. `BookTable` 다이얼로그에는 각각이 `args`로 전달된 LUIS 결과에서 엔터티를 확인하는 폭포형 단계 시퀀스가 포함되어 있습니다. 해당 엔터티를 찾지 못하면 다이얼로그는 `next()`를 호출하여 프롬프트 단계를 건너뜁니다. 해당 엔터티를 찾으면 다이얼로그는 해당 엔터티를 묻는 프롬프트를 표시하고, 프롬프트에 대한 사용자 대답이 다음 폭포형 단계에서 수신됩니다.

```cs
    Dialogs.Add("BookTable",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                dc.ActiveDialog.State = new Dictionary<string, object>();
                IDictionary<string,object> state = dc.ActiveDialog.State;

                // add any LUIS entities to active dialog state.
                if(args.ContainsKey("luisResult")) {
                    cafeLUISModel lResult = (cafeLUISModel)args["luisResult"];
                    updateContextWithLUIS(lResult, ref state);
                }
                
                // prompt if we do not already have cafelocation
                if(state.ContainsKey("cafeLocation")) {
                    state["bookingLocation"] = state["cafeLocation"];
                    await next();
                } else {
                    await dc.Prompt("choicePrompt", "Which of our locations would you like?", promptOptions);
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("cafeLocation")) {
                    var choiceResult = (FoundChoice)args["Value"];
                    state["bookingLocation"] = choiceResult.Value;
                }
                bool promptForDateTime = true;
                if(state.ContainsKey("datetime")) {
                    // validate timex
                    var inputdatetime = new string[] {(string)state["datetime"]};
                    var results = evaluateTimeX((string[])inputdatetime);
                    if(results.Count != 0) {
                        var timexResolution = results.First().TimexValue;
                        var timexProperty = new TimexProperty(timexResolution.ToString());
                        var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                        state["bookingDateTime"] = bookingDateTime;
                        promptForDateTime = false;
                    }
                }
                // prompt if we do not already have date and time
                if(promptForDateTime) {
                    await dc.Prompt("timexPrompt", "When would you like to arrive? (We open at 4PM.)",
                                    new PromptOptions { RetryPromptString = "We only accept reservations for the next 2 weeks and in the evenings between 4PM - 8PM" });
                } else {
                    await next();
                }                       
                
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("datetime")) { 
                    var timexResult = (TimexResult)args;
                    var timexResolution = timexResult.Resolutions.First();
                    var timexProperty = new TimexProperty(timexResolution.ToString());
                    var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                    state["bookingDateTime"] = bookingDateTime;
                }
                // prompt if we already do not have party size
                if(state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = state["partySize"];
                    await next();
                } else {
                    await dc.Prompt("numberPrompt", "How many in your party?");
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = args["Value"];
                }

                await dc.Prompt("confirmationPrompt", $"Thanks, Should I go ahead and book a table for {state["bookingGuestCount"].ToString()} guests at our {state["bookingLocation"].ToString()} location for {state["bookingDateTime"].ToString()} ?");
            },
            async (dc, args, next) =>
            {
                var dialogState = dc.ActiveDialog.State;

                // TODO: Verify user said yes to confirmation prompt

                // TODO: book the table! 

                await dc.Context.SendActivity($"Thanks, I have {dialogState["bookingGuestCount"].ToString()} guests booked for our {dialogState["bookingLocation"].ToString()} location for {dialogState["bookingDateTime"].ToString()}.");
            }
        }
    );
}

// This helper method updates dialog state with any LUIS results
private void updateContextWithLUIS(cafeLUISModel lResult, ref IDictionary<string,object> dialogContext) {
    if(lResult.Entities.cafeLocation != null && lResult.Entities.cafeLocation.GetLength(0) > 0) {
        dialogContext.Add("cafeLocation", lResult.Entities.cafeLocation[0][0]);
    }
    if(lResult.Entities.partySize != null && lResult.Entities.partySize.GetLength(0) > 0) {
        dialogContext.Add("partySize", lResult.Entities.partySize[0][0]);
    } else {
        if(lResult.Entities.number != null && lResult.Entities.number.GetLength(0) > 0) {
            dialogContext.Add("partySize", lResult.Entities.number[0]);
        }
    }
    if(lResult.Entities.datetime != null && lResult.Entities.datetime.GetLength(0) > 0) {
        dialogContext.Add("datetime", lResult.Entities.datetime[0].Expressions[0]);
    }
}
```
## <a name="run-the-sample"></a>샘플 실행

Visual Studio 2017에서 `ContosoCafeBot.sln`을 열고 봇을 실행합니다. [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator)를 사용하여 샘플 봇에 연결합니다.

이 에뮬레이터에서 `reserve a table`라고 말하여 예약 다이얼로그를 시작합니다.

![봇 실행](media/how-to-luisgen/run-bot.png)

# <a name="typescripttabjs"></a>[TypeScript](#tab/js)

[CafeBot_LUIS 샘플](https://aka.ms/contosocafebot-typescript-luis-dialogs)을 다운로드하고 해당 루트 폴더에서 LUISGen을 실행합니다.

```
luisgen cafeLUISModel.json -ts CafeLUISModel
```

이렇게 하면 프로젝트에 추가할 수 있는 **CafeLUISModel.ts**가 생성됩니다. 생성된 파일에서 해당 형식을 사용하여 LUIS 인식기에서 형식화된 결과를 가져올 수 있습니다.


```typescript
// call LUIS and get typed results
await luisRec.recognize(context).then(async (res : any) => 
{    
    // get a typed result
    var typedresult = res as CafeLUISModel;   
    
```

## <a name="pass-the-typed-result-to-a-dialog"></a>형식화된 결과를 다이얼로그에 전달

**luisbot.ts**에서 코드를 검사합니다. `processActivity` 처리기에서 봇은 형식화된 결과를 다이얼로그에 전달합니다.

```typescript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';

        // Create dialog context 
        const state = conversationState.get(context);
        const dc = dialogs.createContext(context, state);
            
        if (!isMessage) {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }

        // Check to see if anyone replied. 
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {

                
                await luisRec.recognize(context).then(async (res : any) => 
                {    
                    var typedresult = res as CafeLUISModel;                
                    let topIntent = LuisRecognizer.topIntent(res);    
                    switch (topIntent)
                    {
                        case Intents.Book_Table: {
                            await context.sendActivity("Top intent is Book_Table ");                          
                            await dc.begin('reserveTable', typedresult);
                            break;
                        }
                        
                        case Intents.Greeting: {
                            await context.sendActivity("Top intent is Greeting");
                            break;
                        }
    
                        case Intents.Who_are_you_intent: {
                            await context.sendActivity("Top intent is Who_are_you_intent");
                            break;
                        }
                        default: {
                            await dc.begin('default', topIntent);
                            break;
                        }
                    }
    
                }, (err) => {
                    // there was some error
                    console.log(err);
                }
                );                                
            }
        }
    });
});
```

## <a name="check-for-existing-entities-in-a-dialog"></a>다이얼로그의 기존 엔터티 확인

**luisbot.ts**에서 `reserveTable` 다이얼로그는 `SaveEntities` 도우미 함수를 호출하여 LUIS 앱이 검색한 엔터티를 확인합니다. 엔터티가 발견되면 다이얼로그 상태에 저장됩니다. 다이얼로그의 각 폭포형 단계는 엔터티가 다이얼로그 상태에 저장되었는지 확인하고 저장되지 않았으면 프롬프트합니다.

```typescript
dialogs.add('reserveTable', [
    async function(dc, args, next){
        var typedresult = args as CafeLUISModel;

        // Call a helper function to save the entities in the LUIS result
        // to dialog state
        await SaveEntities(dc, typedresult);

        await dc.context.sendActivity("Welcome to the reservation service.");
        
        if (dc.activeDialog.state.dateTime) {
            await next();     
        }
        else {
            await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.dateTime) {
            // Save the dateTimePrompt result to dialog state
            dc.activeDialog.state.dateTime = result[0].value;
        }

        // If we don't have party size, ask for it next
        if (!dc.activeDialog.state.partySize) {
            await dc.prompt('textPrompt', "How many people are in your party?");
        } else {
            await next();
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.partySize) {
            dc.activeDialog.state.partySize = result;
        }
        // Ask for the reservation name next
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.Name = result;

        // Save data to conversation state
        var state = conversationState.get(dc.context);

        // Copy the dialog state to the conversation state
        state = dc.activeDialog.state;

        // TODO: Add in <br/>Location: ${state.cafeLocation}
        var msg = `Reservation confirmed. Reservation details:             
            <br/>Date/Time: ${state.dateTime} 
            <br/>Party size: ${state.partySize} 
            <br/>Reservation name: ${state.Name}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

`SaveEntities` 도우미 함수는 `datetime` 및 `partysize` 엔터티를 확인합니다. `datetime` 엔터티는 [미리 빌드된 엔터티](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)입니다.

```typescript
// Helper function that saves any entities found in the LUIS result
// to the dialog state
async function SaveEntities( dc: DialogContext<TurnContext>, typedresult) {
    // Resolve entities returned from LUIS, and save these to state
    if (typedresult.entities)
    {
        console.log(`Entities found.`);
        let datetime = typedresult.entities.datetime;

        if (datetime) {
            // Use the first date or time found in the utterance
            if (datetime[0].timex) {
                timexValues = datetime[0].timex;
                // Datetime results from LUIS are represented in timex format:
                // http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3                                
                // More information on the library which does the recognition can be found here: 
                // https://github.com/Microsoft/Recognizers-Text

                if (datetime[0].type === "datetime") {
                    // To parse timex, here you use the resolve and creator from
                    // @microsoft/recognizers-text-data-types-timex-expression
                    // The second parameter is an array of constraints the results must satisfy
                    var resolution = Resolver.evaluate(
                        // array of timex values to evaluate. There may be more than one: "today at 6" can be 6AM or 6PM.
                        timexValues,
                        // constrain results to times between 4pm and 8pm                        
                        [Creator.evening]);
                    if (resolution[0]) {
                        // toNaturalLanguage takes the current date into account to create a friendly string
                        dc.activeDialog.state.dateTime = resolution[0].toNaturalLanguage(new Date());
                        // You can also use resolution.toString() to format the date/time.
                    } else {
                        // time didn't satisfy constraint.
                        dc.activeDialog.state.dateTime = null;
                    }
                } 
                else  {
                    console.log(`Type ${datetime[0].type} is not yet supported. Provide both the date and the time.`);
                }
            }                                                
        }
        let partysize = typedresult.entities.partySize;
        if (partysize) {
            console.log(`partysize entity detected.${partysize}`);
            // use first partySize entity that was found in utterance
            dc.activeDialog.state.partySize = partysize[0];
        }
        let cafelocation = typedresult.entities.cafeLocation;

        if (cafelocation) {
            console.log(`location entity detected.${cafelocation}`);
            // use first cafeLocation entity that was found in utterance
            dc.activeDialog.state.cafeLocation = cafelocation[0][0];
        }
    } 
}
```

## <a name="run-the-sample"></a>샘플 실행

1. 설치된 TypeScript 컴파일러가 없으면 다음 명령을 사용하여 설치합니다.

```
npm install --global typescript
```

2. 봇을 실행하기 전에 샘플의 루트 디렉터리에서 `npm install`을 실행하여 종속성을 설치합니다.

```
npm install
```

3. 루트 디렉터리에서 `tsc`를 사용하여 샘플을 빌드합니다. 이렇게 하면 `luisbot.js`가 생성됩니다.

4. `lib` 디렉터리에서 `luisbot.js`를 실행합니다.

5. [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator)를 사용하여 샘플을 실행합니다.

6. 이 에뮬레이터에서 `reserve a table`라고 말하여 예약 다이얼로그를 시작합니다.

![봇 실행](media/how-to-luisgen/run-bot.png)

---


## <a name="additional-resources"></a>추가 리소스

LUIS의 자세한 배경 정보는 [Language Understanding](./bot-builder-concept-luis.md)을 참조하세요.


## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [디스패치 도구를 사용하여 LUIS 및 QnA 결합](./bot-builder-tutorial-dispatch.md)


