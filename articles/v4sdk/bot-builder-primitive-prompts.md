---
title: 기본 프롬프트를 사용하여 대화 흐름 관리 | Microsoft Docs
description: 봇 작성기 SDK에서 기본 프롬프트를 사용하여 대화 흐름을 관리하는 방법을 알아봅니다.
keywords: 대화 흐름, 프롬프트
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1514e1bcafc87be9e8449382bca7aa156e512ed9
ms.sourcegitcommit: e8c513d3af5f0c514cadcbcd0a737a7393405afa
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/20/2018
ms.locfileid: "40228352"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>사용자 고유의 프롬프트를 사용하여 사용자 입력 요청
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇과 사용자 간의 대화에는 종종 사용자에게 정보를 요청하고(프롬프트), 사용자의 응답을 구문 분석한 다음, 해당 정보에 대한 작업을 수행하는 것이 포함됩니다. [다이얼로그 라이브러리를 사용하여 사용자에게 요청하는 방법](bot-builder-prompts.md)에 관한 항목에서 **다이얼로그** 라이브러리를 사용하여 사용자에게 입력을 요청하는 방법에 대해 자세히 설명합니다. 무엇보다도 **다이얼로그** 라이브러리는 현재 대화와 현재 질문을 추적합니다. 또한 텍스트, 숫자, 날짜 및 시간 등과 같은 다양한 유형의 정보를 요청하는 메서드를 제공합니다. 

특정 상황에서 기본 제공 **다이얼로그**는 봇에 적합한 솔루션이 아닐 수도 있습니다. **다이얼로그**는 간단한 봇에 너무 많은 오버헤드를 추가하거나, 너무 딱딱하거나, 그렇지 않은 경우 봇에서 수행해야 작업을 달성하지 못할 수 있습니다. 이러한 경우 라이브러리를 건너뛰고 사용자 고유의 프롬프트 논리를 구현할 수 있습니다. 이 항목에서는 기본 *에코 봇*을 설정하여 사용자 고유의 프롬프트를 사용하여 대화를 관리할 수 있는 방법을 보여 줍니다.

## <a name="track-prompt-states"></a>프롬프트 상태 추적

프롬프트 기반 대화에서 현재 대화의 위치와 현재 요청 중인 질문을 추적해야 합니다. 코드에서 이는 두 개의 플래그를 관리하는 것으로 해석됩니다. 

예를 들어 추적하려는 두 개의 속성을 만들 수 있습니다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

여기에는 프롬프트 정보에 중첩된 사용자 프로필 클래스가 있으며, 이러한 클래스는 모두 봇 [상태](bot-builder-howto-v4-state.md)에 함께 저장됩니다.

```csharp
// Where user information will be stored
public class UserProfile
{
    public string userName = null;
    public string workPlace = null;
    public int age = 0;
}

// Flags to keep track of our prompts, and the 
// user profile object for this conversation
public class PromptInformation
{
    public string topicTitle = null;
    public string prompt = null;
    public UserProfile userProfile = null;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

이러한 상태는 단지 현재의 항목과 프롬프트만 계속 추적합니다. 클라우드에 배포할 때 이러한 플래그가 예상대로 작동하도록 하기 위해 [대화 상태](bot-builder-howto-v4-state.md)에 저장합니다. 이 코드는 기본 봇 논리 내에 배치합니다.

**app.js**
```javascript
// Define a topicStates object if it doesn't exist in the convoState.
if(!convo.topicStates){
    convo.topicStates = {
        "topicTitle": undefined, // Current conversation topic in progress
        "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
    };
}
```

---

## <a name="manage-a-topic-of-conversation"></a>대화 항목 관리

사용자로부터 정보를 수집하는 것과 같은 순차적 대화에서는 사용자가 대화에 언제 들어 왔는지, 그리고 대화에서 사용자가 어디에 있는지를 알아야 합니다. 위에서 정의한 프롬프트 플래그를 설정하고 확인한 다음, 그에 따라 작동함으로써 기본 봇 설정 처리기에서 이를 추적할 수 있습니다. 아래 샘플에서는 이러한 플래그를 사용하여 대화를 통해 사용자의 프로필 정보를 수집하는 방법을 보여 줍니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Get our current state, as defined above
var convoState = context.GetConversationState<PromptInformation>();

if (convoState.userProfile == null)
{
    await context.SendActivity("Welcome new user, please fill out your profile information.");
    convoState.topicTitle = "profileTopic"; // Start the userProfile topic
    convoState.userProfile = new UserProfile();
}

// Start or continue a conversation if we are in one
if (convoState.topicTitle == "profileTopic")
{
    if (convoState.userProfile.userName == null && convoState.prompt == null)
    {
        convoState.prompt = "askName";
        await context.SendActivity("What is your name?");
    }
    else if (convoState.prompt == "askName")
    {
        // Save the user's response
        convoState.userProfile.userName = context.Activity.Text;

        // Ask next question
        convoState.prompt = "askAge";
        await context.SendActivity("How old are you?");
    }
    else if (convoState.prompt == "askAge")
    {
        // Save user's response
        if (!Int32.TryParse(context.Activity.Text, out convoState.userProfile.age))
        {
            // Failed to convert to int
            await context.SendActivity("Failed to convert string to int");
        }
        else
        {
            // Ask next question
            convoState.prompt = "workPlace";
            await context.SendActivity("Where do you work?");
        }

    }
    else if (convoState.prompt == "workPlace")
    {
        // Save user's response
        convoState.userProfile.workPlace = context.Activity.Text;

        // Done
        convoState.topicTitle = null; // Reset conversation flag
        convoState.prompt = null;     // Reset the prompt flag
        
        await context.SendActivity("Thank you. Your profile is complete.");
    }
    else // Somehow our flags got inappropriately set
    {
        await context.SendActivity("Looks like something went wrong, let's start over");
        convoState.userProfile = null;
        convoState.prompt = null;
        convoState.topicTitle = null;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**
```javascript
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);

        // Defined flags to manage the conversation flow and prompt states
        // convo.topicStates.topicTitle - Current conversation topic in progress
        // convo.topicStates.prompt - Current prompt state in progress - indicate what question is being asked.
        
        if (isMessage) {
            // Define a topicStates object if it doesn't exist in the convoState.
            if(!convo.topicStates){
                convo.topicStates = {
                    "topicTitle": undefined, // Current conversation topic in progress
                    "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
                };
            }
            
            // If user profile is not defined then define it.
            if(!convo.userProfile){
                
                await context.sendActivity(`Welcome new user, please fill out your profile information.`);
                convo.topicStates.topicTitle = "profileTopic"; // Start the userProfile topic
                convo.userProfile = { // Define the user's profile object
                    "userName": undefined,
                    "age": undefined,
                    "workPlace": undefined
                }; 
            }

            // Start or continue a conversation if we are in one
            if(convo.topicStates.topicTitle == "profileTopic"){
                if(!convo.userProfile.userName && !convo.topicStates.prompt){
                    convo.topicStates.prompt = "askName";
                    await context.sendActivity("What is your name?");
                }
                else if(convo.topicStates.prompt == "askName"){
                    // Save the user's response
                    convo.userProfile.userName = context.activity.text; 

                    // Ask next question
                    convo.topicStates.prompt = "askAge";
                    await context.sendActivity("How old are you?");
                }
                else if(convo.topicStates.prompt == "askAge"){
                    // Save user's response
                    convo.userProfile.age = context.activity.text;

                    // Ask next question
                    convo.topicStates.prompt = "workPlace";
                    await context.sendActivity("Where do you work?");
                }
                else if(convo.topicStates.prompt == "workPlace"){
                    // Save user's response
                    convo.userProfile.workPlace = context.activity.text;

                    // Done
                    convo.topicStates.topicTitle = undefined; // Reset conversation flag
                    convo.topicStates.prompt = undefined;     // Reset the prompt flag
                    await context.sendActivity("Thank you. Your profile is complete.");
                }
            }

            // Check for valid intents
            else if(context.activity.text && context.activity.text.match(/hi/ig)){
                await context.sendActivity(`Hi ${convo.userProfile.userName}.`);
            }
            else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

    });
});

```

---

위의 샘플 코드는 사용자 프로필이 메모리에 정의되어 있는지 확인합니다. 그렇지 않고 새 사용자인 경우 해당 항목을 자동으로 시작하도록 플래그를 설정합니다. 그런 다음, 현재 질문의 값으로 `prompt` 플래그를 설정하여 대화를 진행하는 방법을 보여 줍니다. 이 플래그를 적절한 값으로 설정하면 조건부 절에서 각 설정마다 사용자의 응답을 포착하여 적절하게 처리합니다. 

마지막으로, 봇이 루프에 갇히지 않도록 정보를 요청할 때 이러한 플래그를 다시 설정해야 합니다. 이 기본 설정을 확장하여 사용자 입력에 따라 다른 플래그를 정의하거나 대화를 분기하여 봇에 필요한 대로 더 복잡한 대화 흐름을 관리할 수 있습니다.

## <a name="manage-multiple-topics-of-conversations"></a>여러 대화 항목 관리

하나의 대화 항목을 처리할 수 있으면 봇 논리를 확장하여 여러 대화 항목을 처리할 수 있습니다. 하나의 대화 항목과 마찬가지로 특정 항목을 트리거하는 조건을 확인한 다음, 적절한 경로를 선택함으로써 여러 항목을 간단하게 처리할 수 있습니다.

위의 예제에서는 테이블을 예약하거나 저녁 식사를 주문하는 것과 같은 다른 함수와 항목을 허용하도록 리팩터링할 수 있습니다. 대화를 추적하는 데 필요한 대로 추가 정보를 사용자 프로파일 또는 항목 상태 플래그에 추가할 수 있습니다.

대화의 여러 항목을 더 효율적으로 관리하는 데 도움이 되는 방법 중 하나는 주 메뉴를 제공하는 것입니다. [제안된 작업](bot-builder-howto-add-suggested-actions.md)을 사용하여 사용자가 참여할 수 있는 항목을 선택한 다음, 해당 항목으로 이동할 수 있습니다. 또한 코드를 독립적인 함수로 분할하면 대화 흐름을 더 쉽게 수행할 수 있습니다.

## <a name="validate-user-input"></a>사용자 입력 유효성 검사

**다이얼로그** 라이브러리는 사용자 입력의 유효성을 검사하는 기본 제공 방법을 제공하지만, 고유한 프롬프트를 사용하여 유효성 검사를 수행할 수도 있습니다. 예를 들어 사용자의 연령을 요청하는 경우 응답으로 "Bob"과 같은 문자열이 아닌 숫자를 얻으려고 합니다.

숫자 또는 날짜/시간에 대한 구문 분석은 이 주제의 범위를 벗어나는 복잡한 작업입니다. 다행히도 활용할 수 있는 라이브러리가 있습니다. 이 정보를 올바르게 구문 분석하기 위해 [Microsoft의 텍스트 인식기](https://github.com/Microsoft/Recognizers-Text) 라이브러리를 사용합니다. 이 패키지는 NuGet을 통해 사용하거나 리포지토리에서 다운로드할 수 있습니다. 또한 **다이얼로그** 라이브러리에도 포함되어 있습니다. 주목할 만한 가치가 있지만 여기서는 해당 라이브러리를 사용하지 않습니다.

이 라이브러리는 일반 언어 또는 복잡한 응답(예: 날짜, 시간 또는 전화 번호)을 구문 분석하는 데 특히 유용합니다. 이 샘플에서는 저녁 식사 예약 파티 크기에 대한 숫자만의 유효성을 검사하지만 심층적인 유효성 검사를 위해 동일한 개념을 확장할 수 있습니다. 이 라이브러리에 익숙하지 않은 경우 해당 사이트의 콘텐츠를 검토하여 제공해야 하는 콘텐츠를 확인합니다.

다음 샘플에서는 `RecognizeNumber`만 사용합니다. 라이브러리의 다른 인식기 메서드를 사용하는 방법에 대한 자세한 내용은 해당 [리포지토리의 설명서](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md)에서 찾을 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

인식기 라이브러리를 사용하려면 해당 라이브러리를 using 문에 추가합니다.

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq; // Used to get the first result from the recognizer
```

그런 다음, 유효성 검사를 실제로 수행하는 함수를 정의합니다.

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(input, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        Int32.TryParse(result.First().Text, out int partySize);
        
        if (partySize < 6)
        {
            throw new Exception("Party size too small.");
        }
        else if (partySize > 20)
        {
            throw new Exception("Party size too big.");
        }

        // If we got through this, the number is valid
        return true;
    }
    catch (Exception)
    {
        await context.SendActivity("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

인식기 라이브러리를 사용하려면 **app.js**에 해당 라이브러리가 필요합니다.

```javascript
// Required packages for this bot
var Recognizers = require('@microsoft/recognizers-text-suite');
```

그런 다음, 유효성 검사를 실제로 수행하는 함수를 정의합니다.

```javascript
// Support party size between 6 and 20 only
async function validatePartySize(context, input){
    try {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        var value = parseInt(results[0].resolution.value);

        if(value < 6) {
            throw new Error(`Party size too small.`);
        }
        else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    }
    catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

유효성을 검사하려는 프롬프트 단계 내에서 다음 프롬프트로 이동하기 전에 유효성 검사 함수를 호출합니다. 유효성 검사가 실패하면 질문을 다시 요청합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
if (convoState.prompt == "partySize")
{
    if (await ValidatePartySize(context, context.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which 
        // is a new class we've added to our state
        convoState.ReservationInfo.partySize = context.Activity.Text;

        // Ask next question
        convoState.prompt = "reserveName";
        await context.SendActivity("Who's name will this be under?");
    }
    else
    {
        // Ask again
        await context.SendActivity("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
// ...
if(convo.topicStates.prompt == "partySize"){
    if(await validatePartySize(context, context.activity.text)){
        // Save user's response
        convo.reservationInfo.partySize = context.activity.text;
        
        // Ask next question
        topicStates.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    }
    else {
        // Ask again
        await context.sendActivity("How many people are in your party?");
    }
}
// ...
```

---

## <a name="next-step"></a>다음 단계

이제 프롬프트 상태를 직접 관리하는 방법을 이해했으므로 **다이얼로그** 라이브러리를 활용하여 사용자 입력을 요청하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [다이얼로그를 사용하여 사용자 입력 요청](bot-builder-prompts.md)
