---
title: 사용자 입력을 위한 프롬프트 | Microsoft Docs
description: 프롬프트를 사용하여 Node.js용 Bot Framework SDK를 통해 사용자 입력을 수집하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1cad11c8b1dde800543c919ab579b0112e7d3036
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404988"
---
# <a name="prompt-for-user-input"></a>사용자 입력을 위한 프롬프트

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Node.js용 Bot Framework SDK는 사용자의 입력을 수집하는 과정을 단순화하는 기본 제공 프롬프트 세트를 제공합니다. 

‘프롬프트’는 봇에 사용자의 입력이 필요할 때마다 사용됩니다.  프롬프트를 사용하면 프롬프트를 폭포로 연결하여 사용자에게 일련의 입력을 요청할 수 있습니다. 프롬프트를 [폭포](bot-builder-nodejs-dialog-waterfall.md)와 함께 사용하면 봇에서 [대화 흐름을 관리](bot-builder-nodejs-manage-conversation-flow.md)할 수 있습니다. 

이 문서에서는 프롬프트의 작동 방식 및 프롬프트를 사용하여 사용자의 정보를 수집하는 방법을 알아봅니다.

## <a name="prompts-and-responses"></a>프롬프트 및 응답

사용자의 입력이 필요할 때마다 프롬프트를 보내고, 사용자가 입력으로 응답할 때까지 기다린 다음, 입력을 처리하고 응답을 사용자에게 보낼 수 있습니다.

다음 코드 샘플은 이름을 묻는 메시지를 사용자에게 표시하고 인사말 메시지로 응답합니다.

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

이 기본 구조를 사용하면 봇에 필요한 만큼 프롬프트와 응답을 추가하여 대화 흐름을 모델링할 수 있습니다.

## <a name="prompt-results"></a>프롬프트 결과 

기본 제공 프롬프트는 `results.response` 필드로 사용자의 응답을 반환하는 [대화 상자](bot-builder-nodejs-dialog-overview.md)로 구현됩니다. JSON 개체의 경우 응답은 `results.response.entity` 필드로 반환됩니다. 모든 형식의 [대화 상자 처리기](bot-builder-nodejs-dialog-overview.md#dialog-handlers)가 프롬프트 결과를 받을 수 있습니다. 봇이 응답을 받으면 이 응답을 사용하거나, [`session.endDialogWithResult`][EndDialogWithResult] 메서드를 호출하여 호출 대화 상자에 다시 전달할 수 있습니다.

다음 코드 샘플은 `session.endDialogWithResult` 메서드를 사용하여 프롬프트 결과를 호출 대화 상자에 반환하는 방법을 보여 줍니다. 이 예제에서 `greetings` 대화 상자는 `askName` 대화 상자가 반환하는 프롬프트 결과를 사용하여 이름별로 사용자에게 인사말을 제공합니다.

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="prompt-types"></a>프롬프트 형식
Node.js용 Bot Framework SDK는 다양한 형식의 기본 제공 프롬프트를 포함합니다. 

|**프롬프트 형식**     | **설명** |     
| ------------------ | --------------- |
|[Prompts.text](#promptstext) | 사용자에게 텍스트 문자열을 입력하도록 요청합니다. |     
|[Prompts.confirm](#promptsconfirm) | 사용자에게 작업을 확인하도록 요청합니다.| 
|[Prompts.number](#promptsnumber) | 사용자가 숫자를 입력하도록 요청합니다.     |
|[Prompts.time](#promptstime) | 사용자에게 시간 또는 날짜/시간을 요청합니다.      |
|[Prompts.choice](#promptschoice) | 사용자에게 옵션 목록에서 선택하도록 요청합니다.    |
|[Prompts.attachment](#promptsattachment) | 사용자에게 사진이나 비디오를 업로드하도록 요청합니다.|       

다음 섹션에서는 각 형식의 프롬프트에 대한 자세한 정보를 제공합니다.

### <a name="promptstext"></a>Prompts.text

[Prompts.text()][PromptsText] 메서드를 사용하여 사용자에게 **텍스트 문자열**을 요청할 수 있습니다. 프롬프트는 사용자의 응답을 [IPromptTextResult][IPromptTextResult]로 반환합니다.

```javascript
builder.Prompts.text(session, "What is your name?");
```

### <a name="promptsconfirm"></a>Prompts.confirm

[Prompts.confirm()][PromptsConfirm] 메서드를 사용하여 사용자에게 **예/아니요** 응답으로 작업을 확인하도록 요청합니다. 프롬프트는 사용자의 응답을 [IPromptConfirmResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html)로 반환합니다.

```javascript
builder.Prompts.confirm(session, "Are you sure you wish to cancel your order?");
```

### <a name="promptsnumber"></a>Prompts.number

[Prompts.number()][PromptsNumber] 메서드를 사용하여 사용자에게 **숫자**를 요청할 수 있습니다. 프롬프트는 사용자의 응답을 [IPromptNumberResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html)로 반환합니다.

```javascript
builder.Prompts.number(session, "How many would you like to order?");
```

### <a name="promptstime"></a>Prompts.time

[Prompts.time()][PromptsTime] 메서드를 사용하여 사용자에게 **시간** 또는 **날짜/시간**을 요청할 수 있습니다. 프롬프트는 사용자의 응답을 [IPromptTimeResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html)로 반환합니다. 프레임워크는 [Chrono](https://github.com/wanasit/chrono) 라이브러리를 사용하여 사용자 응답을 구문 분석하고 상대적 응답(예: “5분 내”) 및 절대적 응답(예: “6월 6일 오후 2시”)을 둘 다 지원합니다.

사용자의 응답을 나타내는 [results.response](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html#response) 필드에는 날짜 및 시간을 지정하는 [엔터티](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html) 개체가 포함됩니다. 날짜 및 시간을 JavaScript `Date` 개체로 확인하려면 [EntityRecognizer.resolveTime()](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime) 메서드를 사용합니다.

> [!TIP] 
> 사용자가 입력하는 시간은 봇을 호스트하는 서버의 표준 시간대에 따라 UTC 시간으로 변환됩니다. 서버가 사용자와 다른 표준 시간대에 있을 수 있으므로 표준 시간대를 고려해야 합니다. 날짜 및 시간을 사용자의 현지 시간으로 변환하려면 사용자에게 사용자의 표준 시간대를 요청하는 것이 좋습니다.

```javascript
bot.dialog('createAlarm', [
    function (session) {
        session.dialogData.alarm = {};
        builder.Prompts.text(session, "What would you like to name this alarm?");
    },
    function (session, results, next) {
        if (results.response) {
            session.dialogData.name = results.response;
            builder.Prompts.time(session, "What time would you like to set an alarm for?");
        } else {
            next();
        }
    },
    function (session, results) {
        if (results.response) {
            session.dialogData.time = builder.EntityRecognizer.resolveTime([results.response]);
        }

        // Return alarm to caller  
        if (session.dialogData.name && session.dialogData.time) {
            session.endDialogWithResult({ 
                response: { name: session.dialogData.name, time: session.dialogData.time } 
            }); 
        } else {
            session.endDialogWithResult({
                resumed: builder.ResumeReason.notCompleted
            });
        }
    }
]);
```

### <a name="promptschoice"></a>Prompts.choice

[Prompts.choice()][PromptsChoice] 메서드를 사용하여 사용자에게 **옵션 목록에서 선택**하도록 요청할 수 있습니다. 사용자는 선택하는 옵션과 연결된 번호를 입력하거나 선택하는 옵션의 이름을 입력하여 선택을 전달할 수 있습니다. 옵션 이름의 전체 및 부분 일치가 둘 다 지원됩니다. 프롬프트는 사용자의 응답을 [IPromptChoiceResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html)로 반환합니다. 

사용자에게 표시되는 목록 스타일을 지정하려면 [IPromptOptions.listStyle](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptoptions.html#liststyle) 속성을 설정합니다. 다음 표에서는 이 속성의 `ListStyle` 열거형 값을 보여 줍니다.


`ListStyle` 열거형 값은 다음과 같습니다.

| 인덱스 | 이름 | 설명 |
| ---- | ---- | ---- |
| 0 | 없음 | 렌더링되는 목록이 없습니다. 목록이 프롬프트의 일부로 포함될 경우 사용됩니다. |
| 1 | inline | 선택 항목이 다음 형식의 인라인 목록으로 렌더링됩니다. “1. 빨강, 2. 녹색 또는 3. 파랑”. |
| 2 | list | 선택 항목이 번호 매기기 목록으로 렌더링됩니다. |
| 3 | button | 선택 항목이 단추를 지원하는 채널의 단추로 렌더링됩니다. 다른 채널의 경우 텍스트로 렌더링됩니다. |
| 4 | auto | 옵션의 채널 및 수에 따라 스타일이 자동으로 선택됩니다. | 

`builder` 개체에서 이 열거형에 액세스하거나 인덱스를 제공하여 `ListStyle`을 선택할 수 있습니다. 예를 들어, 다음 코드 조각에서 두 문은 모두 같은 작업을 수행합니다.

```javascript
// ListStyle passed in as Enum
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: builder.ListStyle.button });

// ListStyle passed in as index
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: 3 });
```

옵션 목록을 지정하려면 파이프로 구분된(`|`) 문자열, 문자열 배열 또는 개체 맵을 사용할 수 있습니다.

파이프로 구분된 문자열: 

```javascript
builder.Prompts.choice(session, "Which color?", "red|green|blue");
```

문자열 배열:

```javascript
builder.Prompts.choice(session, "Which color?", ["red","green","blue"]);
```

개체 맵: 

```javascript
var salesData = {
    "west": {
        units: 200,
        total: "$6,000"
    },
    "central": {
        units: 100,
        total: "$3,000"
    },
    "east": {
        units: 300,
        total: "$9,000"
    }
};

bot.dialog('getSalesData', [
    function (session) {
        builder.Prompts.choice(session, "Which region would you like sales for?", salesData); 
    },
    function (session, results) {
        if (results.response) {
            var region = salesData[results.response.entity];
            session.send(`We sold ${region.units} units for a total of ${region.total}.`); 
        } else {
            session.send("OK");
        }
    }
]);
```

### <a name="promptsattachment"></a>Prompts.attachment

[Prompts.attachment()][PromptsAttachment] 메서드를 사용하여 사용자에게 이미지나 동영상 등의 파일을 업로드하도록 요청할 수 있습니다. 프롬프트는 사용자의 응답을 [IPromptAttachmentResult](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html)로 반환합니다.

```javascript
builder.Prompts.attachment(session, "Upload a picture for me to transform.");
```

## <a name="next-steps"></a>다음 단계

이제 사용자에게 폭포를 단계별로 안내하고 정보를 요청하는 메시지를 표시하는 방법을 알았으므로, 대화 흐름을 더 효율적으로 관리하는 방법을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [대화 흐름 관리](bot-builder-nodejs-dialog-manage-conversation-flow.md)


[SendAttachments]: bot-builder-nodejs-send-receive-attachments.md
[SendCardWithButtons]: bot-builder-nodejs-send-rich-cards.md
[RecognizeUserIntent]: bot-builder-nodejs-recognize-intent-messages.md
[SaveUserData]: bot-builder-nodejs-save-user-data.md

[UniversalBot]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[Session]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session


[SendTyping]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping

[EndDialogWithResult]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#enddialogwithresult

[IPromptResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html

[Result_Response]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#response

[ResumeReason]: https://docs.botframework.com/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html

[Result_Resumed]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed

[entity]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html

[ResolveTime]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime

[PromptsRef]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html

[PromptsText]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#text

[IPromptTextResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttextresult.html

[PromptsConfirm]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#confirm

[IPromptConfirmResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html

[PromptsNumber]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#number

[IPromptNumberResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html

[PromptsTime]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#time

[IPromptTimeResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html

[PromptsChoice]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice

[IPromptChoiceResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html

[PromptsAttachment]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#attachment

[IPromptAttachmentResult]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html
