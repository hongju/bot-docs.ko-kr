---
title: Cortana Skill을 사용하여 음성 지원 봇 빌드 | Microsoft Docs
description: Cortana Skill을 사용하여 음성 지원 봇을 빌드하는 방법과 Node.js용 Bot Builder SDK에 대해 알아봅니다.
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e728a3999c484d19a78f03bd8eb7b8bd8833c39f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998040"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>Cortana Skill을 사용하여 음성 지원 봇 빌드

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-cortana-skill.md)

Node.js용 Bot Builder SDK를 사용하면 Cortana Skill로 Cortana 채널에 연결하여 음성 지원 봇을 빌드할 수 있습니다. Cortana Skill을 사용하면 Cortana를 통해 사용자의 음성 입력에 응답하는 기능을 제공할 수 있습니다.

> [!TIP]
> 기술 정의 및 수행할 수 있는 작업에 대한 자세한 내용은 [Cortana Skill 키트][CortanaGetStarted]를 참조하세요.

Bot Framework를 사용하여 Cortana Skill을 만드는 경우 Cortana 관련 지식이 거의 필요하지 않으며, 주로 봇 빌드로 이루어집니다. 그 동안 만들어 온 다른 봇과의 주요 차이점 중 하나는 Cortana는 시각적 구성 요소와 오디오 구성 요소가 모두 포함된다는 점입니다. 시각적 구성 요소의 경우, Cortana는 카드 등의 콘텐츠 렌더링을 위한 캔버스 영역을 제공합니다. 오디오 구성 요소의 경우, 봇 메시지에 텍스트 또는 SSML을 제공하면 Cortana가 사용자에게 읽어주므로 봇에 음성을 제공합니다. 

> [!NOTE]
> Cortana는 다양한 디바이스에서 사용할 수 있습니다. 일부 장치에는 화면이 있고, 독립 실행형 스피커와 같은 장치에는 화면이 없을 수 있습니다. 봇이 두 시나리오를 모두 처리할 수 있도록 해야 합니다. 디바이스 정보를 확인하는 방법을 알아보려면 참조 [Cortana 관련 엔터티][CortanaSpecificEntities]를 참조하세요.

## <a name="adding-speech-to-your-bot"></a>봇에 음성 추가

봇의 음성 메시지는 SSML(Speech Synthesis Markup Language)로 표현됩니다. Bot Builder SDK를 사용하면 봇 응답에 SSML을 포함하여 봇이 보여 주는 내용뿐 아니라 봇이 말하는 내용도 제어할 수 있습니다.

### <a name="sessionsay"></a>session.say

봇은 **session.send** 대신 **session.say** 메서드를 사용하여 사용자에게 말합니다. 여기에는 SSML 출력은 물론 카드와 같은 첨부 파일을 보내는 선택적인 매개 변수도 포함됩니다. 

메서드의 형식은 다음과 같습니다.

```session.say(displayText: string, speechText: string, options?: object)```

| 매개 변수 | 설명 |
|------|------|
| **displayText** | Cortana의 UI에 표시되는 텍스트 메시지입니다.|
| **speechText** | Cortana가 사용자에게 읽어주는 텍스트나 SSML입니다. |
| **options** | 첨부 파일이나 입력 힌트를 포함할 수 있는 [IMessage][IMessage] 개체입니다. 입력 힌트는 봇이 입력을 수락, 예상 또는 무시하는지를 나타냅니다. 카드 첨부 파일은 Cortana의 캔버스에서 **displayText** 정보 아래 표시됩니다.   |

**inputHint** 속성을 사용하면 봇이 입력을 기다리고 있는지 여부를 Cortana에 알려줄 수 있습니다. 기본 제공 프롬프트는 사용하는 경우에는 이 값이 기본값 **expectingInput**으로 자동 설정됩니다.


| 값 | 설명 |
|------|------|
| **acceptingInput** | 봇이 소극적으로 입력을 받을 준비가 되었지만 응답을 기다리고 있지는 않습니다. Cortana는 사용자가 마이크 단추를 길게 누를 경우 사용자 입력을 허용합니다.|
| **expectingInput** | 봇이 사용자 응답을 적극적으로 필요로 함을 나타냅니다. Cortana는 사용자가 마이크에 말하는 내용을 수신 대기합니다.  |
| **ignoringInput** | Cortana가 입력을 무시합니다. 봇은 요청을 적극적으로 처리 중이며 요청이 완료될 때까지 사용자 입력을 무시하는 경우 이 힌트를 보낼 수 있습니다.  |


다음 예제는 Cortana가 일반 텍스트나 SSML을 읽는 방법을 보여줍니다.

```javascript
// Have Cortana read plain text
session.say('This is the text that Cortana displays', 'This is the text that is spoken by Cortana.');

// Have Cortana read SSML
session.say('This is the text that Cortana displays', '<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">This is the text that is spoken by Cortana.</speak>');
```

이 예제에서는 사용자 입력이 필요함을 Cortana에 알리는 방법을 보여 줍니다. 마이크가 열려 있습니다.
```javascript
// Add an InputHint to let Cortana know to expect user input
session.say('Hi there', 'Hi, what’s your name?', {
    inputHint: builder.InputHint.expectingInput
});
```
<!-- TODO: tip about time limit and batching -->


### <a name="prompts"></a>프롬프트

**session.say()** 메서드를 사용하는 것 외에도 **speak** 및 **retrySpeak** 옵션을 사용하여 기본 제공 프롬프트에 텍스트나 SSML을 전달할 수 있습니다.  

```javascript
builder.Prompts.text(session, 'text based prompt', {                                    
    speak: 'Cortana reads this out initially',                                               
    retrySpeak: 'This message is repeated by Cortana after waiting a while for user input',  
    inputHint: builder.InputHint.expectingInput                                              
});
```



<!-- TODO: Link to SSML library -->

사용자에게 선택 항목 목록을 제공하려면 **Prompts.choice**를 사용합니다. **synonyms** 옵션을 사용하면 사용자의 발언을 보다 유연하게 인식할 수 있습니다. **value** 옵션은 **results.response.entity**에 반환됩니다. **action** 옵션은 봇이 선택 항목에 표시하는 레이블을 지정합니다.

**Prompts.choice**는 서수 선택을 지원합니다. 즉 사용자가 "the first", "the second" 또는 "the third"라고 말해서 목록에서 항목을 선택할 수 있습니다. 예를 들어 다음 프롬프트에서 사용자가 Cortana에게 "the second option"을 요청하면 프롬프트에 8이라는 값이 반환됩니다.

```javascript
        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|ate|8 sided|8 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') // use helper function to format SSML
        });
```

이전 예제에서 프롬프트의 **speak** 속성에 대한 SSML은 다음 형식으로 지역화된 프롬프트 파일에 저장된 문자열을 사용하여 형성됩니다. 

```json
{
    "choose_sides": "__Number of Sides__",
    "choose_sides_ssml": [
        "How many sides on the dice?",
        "Pick your poison.",
        "All the standard sizes are supported."
    ]
}
```


그런 다음, 도우미 함수가 SSML(Speech Synthesis Markup Language) 문서의 필수 루트 요소를 빌드합니다. 

```javascript

module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}
```

> [!TIP]
> [Roller 샘플 스킬](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill)에서 봇의 SSML 기반 응답을 작성하는 작은 유틸리티 모듈(ssml.js)을 찾을 수 있습니다.
> 형식이 올바른 SSML을 손쉽게 만들 수 있는 몇 가지 유용한 SSML 라이브러리가 [npm](https://www.npmjs.com/search?q=ssml)을 통해 제공됩니다.

## <a name="display-cards-in-cortana"></a>Cortana에서 카드 표시

음성 응답 외에도 Cortana에서 카드 첨부 파일을 표시할 수 있습니다. Cortana는 다음과 같은 리치(rich) 카드를 지원합니다.
* [HeroCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html)
* [ReceiptCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html)
* [ThumbnailCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html)

이러한 카드가 Cortana 내에서 어떻게 표시되는지 확인하려면 [카드 디자인 모범 사례][CardDesign]를 참조하세요. 봇에 리치(rich) 카드를 추가하는 방법에 대한 예제는 [리치 카드 보내기](bot-builder-nodejs-send-rich-cards.md)를 참조하세요. 

다음 코드는 **speak** 및 **inputHint** 속성을 히어로(Hero) 카드를 포함하는 메시지에 추가하는 방법을 보여줍니다.

```javascript 

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play yahtzee', 'Play Yahtzee')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'I\'m roller, the dice rolling bot. You can say \'roll some dice\''))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput); // Tell Cortana to accept input
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });


/** This helper function builds the required root element of a Speech Synthesis Markup Language (SSML) document. */
module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}

```
## <a name="sample-rollerskill"></a>샘플: RollerSkill
다음 섹션의 코드는 주사위 굴리기에 대한 샘플 Cortana Skill에서 가져온 것입니다. [BotBuilder 샘플 리포지토리](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill)에서 봇의 전체 코드를 다운로드합니다.

Cortana에 해당 [호출 이름][InvocationNameGuidelines]을 말하여 기술을 호출합니다. 롤러 기술의 경우, [Cortana 채널에 봇을 추가][CortanaChannel]하고 Cortana Skill로 등록한 후 Cortana에 “Ask Roller” 또는 “Ask Roller to roll dice”라고 말해 기술을 호출할 수 있습니다.

### <a name="explore-the-code"></a>코드 탐색

RollerSkill 샘플은 사용자에게 선택 가능한 옵션을 알려주는 단추가 있는 카드를 여는 것으로 시작됩니다.

```javascript
/**
 *   Create your bot with a default message handler that receive messages from the user.
 * - This function is be called anytime the user's utterance isn't
 *   recognized by the other handlers in the bot.
 */
var bot = new builder.UniversalBot(connector, function (session) {
    // Just redirect to our 'HelpDialog'.
    session.replaceDialog('HelpDialog');
});

//...

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play craps', 'Play Craps')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'help_ssml'))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput);
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });
```

### <a name="prompt-the-user-for-input"></a>사용자에게 입력 요구

다음 대화 상자는 봇이 플레이할 사용자 지정 게임을 설정합니다.  사용자에게 원하는 주사위의 면 수를 질문한 다음, 몇 번을 굴려야 하는지 질문합니다. 게임 구조가 구축되면 별도의 'PlayGameDialog'로 전달됩니다.

dialog를 시작하기 위해 이 dialog의 **triggerAction()** 처리기에서 사용자가 "I'd like to roll some dice"(주사위를 굴리겠습니다)와 같은 말을 하도록 허용합니다. 정규식을 사용하여 사용자의 입력과 일치시키지만 간편하게 [LUIS 의도](./bot-builder-nodejs-recognize-intent-luis.md)를 사용할 수도 있습니다. 


```javascript


bot.dialog('CreateGameDialog', [
    function (session) {
        // Initialize game structure.
        // - dialogData gives us temporary storage of this data in between
        //   turns with the user.
        var game = session.dialogData.game = { 
            type: 'custom', 
            sides: null, 
            count: null,
            turns: 0
        };

        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '6', action: { title: '6 Sides' }, synonyms: 'six|sex|6 sided|6 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|8 sided|8 sides' },
            { value: '10', action: { title: '10 Sides' }, synonyms: 'ten|10 sided|10 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') 
        });
    },
    function (session, results) {
        // Store users input
        // - The response comes back as a find result with index & entity value matched.
        var game = session.dialogData.game;
        game.sides = Number(results.response.entity);

        /**
         * Ask for number of dice.
         */
        var prompt = session.gettext('choose_count', game.sides);
        builder.Prompts.number(session, prompt, {
            speak: speak(session, 'choose_count_ssml'),
            minValue: 1,
            maxValue: 100,
            integerOnly: true
        });
    },
    function (session, results) {
        // Store users input
        // - The response is already a number.
        var game = session.dialogData.game;
        game.count = results.response;

        /**
         * Play the game we just created.
         * 
         * replaceDialog() ends the current dialog and start a new
         * one in its place. We can pass arguments to dialogs so we'll pass the
         * 'PlayGameDialog' the game we created.
         */
        session.replaceDialog('PlayGameDialog', { game: game });
    }
]).triggerAction({ matches: [
    /(roll|role|throw|shoot).*(dice|die|dye|bones)/i,
    /new game/i
 ]});
```

### <a name="render-results"></a>렌더링 결과

 이 dialog는 주요 게임 루프입니다. 봇이 게임 구조를 **session.conversationData**에 저장하기 때문에 사용자가 "roll again"(다시 굴려)이라고 말하면 같은 주사위 세트를 다시 굴릴(roll) 수 있습니다.

```javascript

bot.dialog('PlayGameDialog', function (session, args) {
    // Get current or new game structure.
    var game = args.game || session.conversationData.game;
    if (game) {
        // Generate rolls
        var total = 0;
        var rolls = [];
        for (var i = 0; i < game.count; i++) {
            var roll = Math.floor(Math.random() * game.sides) + 1;
            if (roll > game.sides) {
                // Accounts for 1 in a million chance random() generated a 1.0
                roll = game.sides;
            }
            total += roll;
            rolls.push(roll);
        }

        // Format roll results
        var results = '';
        var multiLine = rolls.length > 5;
        for (var i = 0; i < rolls.length; i++) {
            if (i > 0) {
                results += ' . ';
            }
            results += rolls[i];
        }

        // Render results using a card
        var card = new builder.HeroCard(session)
            .subtitle(game.count > 1 ? 'card_subtitle_plural' : 'card_subtitle_singular', game)
            .buttons([
                builder.CardAction.imBack(session, 'roll again', 'Roll Again'),
                builder.CardAction.imBack(session, 'new game', 'New Game')
            ]);
        if (multiLine) {
            //card.title('card_title').text('\n\n' + results + '\n\n');
            card.text(results);
        } else {
            card.title(results);
        }
        var msg = new builder.Message(session).addAttachment(card);

        // Determine bots reaction for speech purposes
        var reaction = 'normal';
        var min = game.count;
        var max = game.count * game.sides;
        var score = total/max;
        if (score == 1.0) {
            reaction = 'best';
        } else if (score == 0) {
            reaction = 'worst';
        } else if (score <= 0.3) {
            reaction = 'bad';
        } else if (score >= 0.8) {
            reaction = 'good';
        }

        // Check for special craps rolls
        if (game.type == 'craps') {
            switch (total) {
                case 2:
                case 3:
                case 12:
                    reaction = 'craps_lose';
                    break;
                case 7:
                    reaction = 'craps_seven';
                    break;
                case 11:
                    reaction = 'craps_eleven';
                    break;
                default:
                    reaction = 'craps_retry';
                    break;
            }
        }

        // Build up spoken response
        var spoken = '';
        if (game.turn == 0) {
            spoken += session.gettext('start_' + game.type + '_game_ssml') + ' ';
        } 
        spoken += session.gettext(reaction + '_roll_reaction_ssml');
        msg.speak(ssml.speak(spoken));

        // Increment number of turns and store game to roll again
        game.turn++;
        session.conversationData.game = game;

        /**
         * Send card and bot's reaction to user. 
         */

        msg.inputHint(builder.InputHint.acceptingInput);
        session.send(msg).endDialog();
    } else {
        // User started session with "roll again" so let's just send them to
        // the 'CreateGameDialog'
        session.replaceDialog('CreateGameDialog');
    }
}).triggerAction({ matches: /(roll|role|throw|shoot) again/i });
```

## <a name="next-steps"></a>다음 단계
봇이 로컬에서 실행되거나 클라우드에 배포된 경우 Cortana에서 호출할 수 있습니다. Cortana 기술을 사용해 보는 데 필요한 단계는 [Cortana Skill 테스트](../bot-service-debug-cortana-skill.md)를 참조하세요.


## <a name="additional-resources"></a>추가 리소스
* [Cortana Skill 키트][CortanaGetStarted]
* [메시지에 음성 추가](bot-builder-nodejs-text-to-speech.md)
* [SSML 참조][SSMLRef]
* [Cortana 음성 디자인 모범 사례][VoiceDesign]
* [Cortana 카드 디자인 모범 사례][CardDesign]
* [Cortana 개발자 센터][CortanaDevCenter]
* [Cortana 테스트 및 디버그 모범 사례][Cortana-TestBestPractice]


[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/


[SSMLRef]: https://aka.ms/cortana-ssml
[IMessage]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage.html
[Send]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines
[VoiceDesign]: https://aka.ms/cortana-design-voice
[CardDesign]: https://aka.ms/cortana-design-card
[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-Publish]: https://aka.ms/cortana-publish


[CortanaTry]: https://aka.ms/try-cortana-bot
[CortanaChannel]: https://aka.ms/bot-cortana-channel
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice
