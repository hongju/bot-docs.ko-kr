---
title: Node.js를 사용하여 Cortana Skill 구축 | Microsoft Docs
description: Node.js용 Bot Framework SDK에서 Cortana Skill을 빌드하기 위한 핵심 개념을 알아봅니다.
keywords: Bot Framework, Cortana Skill, 음성, Node.js, Bot Builder, SDK, 주요 개념, 핵심 개념
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/10/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5de773f6f8f4d46c0c1fe880588f2530c3c68f56
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/18/2019
ms.locfileid: "59746067"
---
# <a name="key-concepts-for-building-a-bot-for-cortana-skills-using-nodejs"></a>Node.js를 사용하여 Cortana Skill용 봇을 빌드하기 위한 주요 개념
 
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!NOTE]
> 이 문서는 예비적인 내용이며 업데이트될 예정입니다.

이 문서에서는 Node.js용 Bot Framework SDK에서 Cortana Skill을 빌드하기 위한 주요 개념을 소개합니다. 

## <a name="what-is-a-cortana-skill"></a>Cortana Skill이란?
Cortana Skill은 Windows 10에 기본 제공되는 것과 같은 Cortana 클라이언트를 사용하여 호출할 수 있는 봇입니다. 사용자는 봇과 연관된 키워드나 구문을 말하여 봇을 시작합니다. Bot Framework 포털을 사용하여 봇을 시작하는 데 사용할 키워드를 구성합니다. 

Cortana는 텍스트 대화뿐 아니라 음성 메시지도 주고받을 수 있는 음성 지원 채널로 생각할 수 있습니다. Cortana Skill로 게시되는 봇은 음성뿐만 아니라 텍스트도 사용할 수 있도록 설계해야 합니다. Bot Framework는 봇이 보내는 음성 메시지를 정의하도록 SSML(Speech Synthesis Markup Language)을 지정하는 메서드를 제공합니다.

## <a name="acknowledge-user-utterances"></a>사용자 발언 확인 

<!-- Establishing conversational understanding -->
<!-- Placeholder: In this section, describe how you have to write your speech to sound natural -->


음성 지원 봇을 만들 때는 대화의 공통 기반과 상호 이해를 구축하려고 해야 합니다. 봇은 사용자의 발언을 들었고 이해했다는 것을 나타내서 사용자의 발언을 "접지(ground)"해야 합니다.

사용자는 시스템이 자신의 발언을 접지(ground)하지 못하면 혼란스러워합니다. 예를 들어, 다음 대화에서 봇이 "다음은 뭔가요?"라고 질문하면 약간 혼란스러울 수 있습니다.

> **Cortana**: 프로필 몇 개를 더 검토하시겠습니까?  
> **사용자**:  아니요.  
> **Cortana**: 다음 작업

봇이 확인의 의미로 "오케이"를 추가하면 사용자에게 보다 친절해집니다.

> **Cortana**: 프로필 몇 개를 더 검토하시겠습니까?  
> **사용자**:  아니요.  
> **Cortana**: **알겠습니다**. 다음은 무엇인가요?

접지의 정도(약한 정도에서 강한 정도)

1. 지속적인 주목
2. 다음 관련 기여
3. 확인: 최소한의 응답 또는 컨티뉴어: "예", "응", "오케이", "좋아"
4. 데모: 발언을 재구성, 완성하여 이해했다는 것을 나타냅니다.
5. 디스플레이: 발언의 전체 또는 일부를 반복합니다.

### <a name="acknowledgement-and-next-relevant-contribution"></a>확인 및 다음 관련 기여

> **사용자**: 5월에 여행을 가려고 합니다.  
> **Cortana**: **알겠습니다**. 5월 며칠에 여행을 가시겠습니까?  
> **사용자**: 음, 12일에서 15일까지요.  
> **Cortana**: **알겠습니다**. 어느 도시로 가시겠습니까?  

### <a name="grounding-by-demonstration"></a>데모를 통한 접지

> **사용자**: 5월에 여행을 가려고 합니다.  
> **Cortana**: 그러면, 5월 **며칠**에 여행을 가시겠습니까?  
> **사용자**: 네, 12일에서 15일까지요.  
> **Cortana**: **그러면**, 어느 도시로 가시겠습니까?  
    
### <a name="closure"></a>Closure

동작을 수행하는 봇은 수행이 성공한 증거를 표현해야 합니다. 실패했거나 이해했다는 것을 나타내는 것도 중요합니다. 

* 비-음성 클로저: 엘리베이터 버튼을 누르면 버튼의 등이 켜집니다.  
다음은 2단계 프로세스입니다.
    * 프레젠테이션(단추를 누를 경우)
    * 승인(단추가 밝아질 경우)

## <a name="differences-in-content-presentation"></a>콘텐츠 표현의 차이점
Cortana는 다양한 디바이스에서 지원되지만 그 중 화면이 있는 디바이스는 일부입니다. 음성 인식 봇을 설계할 때는 음성 대화가 봇에 표시되는 텍스트 메시지와 동일하지 않은 경우가 종종 있다는 점을 고려해야 합니다.
<!-- If there are differences in what the bot will say, in the text vs the speak fields of a prompt or in a waterfall, for example, discuss them here.

## Speech

You bot uses the **session.say** method to speak to the user. The speak method has three overloads:
* If you pass only one parameter to **session.say**, it can be a text parameter.
* If you pass two parameters to **session.say**, it can take text and SSML.
* If you pass three parameters, the third parameter takes an options structure that specifies all the options you can pass to build an **IMessage** object.

```javascript
var bot = new builder.UniversalBot(connector, function (session) {
    session.say("Hello... I'm a decision making bot.'.", 
        ssml.speak("Hello. I can help you answer all of life's tough questions."));
    session.replaceDialog('rootMenu');
});

```
## Speech in messages

The **IMessage** object provides a **speak** property for SSML. It can be used to play a .wav file.

The **inputHint** property helps indicate to Cortana whether your bot is expecting input. If you're using a built-in prompt, this value is automatically set to the default of **expectingInput**.

The **inputHint** property can take the following values: 
* **expectingInput**: Indicates that the bot is actively expecting a response from the user. Cortana listens for the user to speak into the microphone.
* **acceptingInput**: Indicates that the bot is passively ready for input but is not waiting on a response. Cortana accepts input from the user if the user holds down the microphone button.
* **ignoringInput**: Cortana is ignoring input. Your bot may send this hint if it is actively processing a request and will ignore input from users until the request is complete.

Prompts must use the `speak:` option.

```javascript
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```

Prompts.number has *ordinal support*, meaning that you can say "the last", "the first", "the next-to-last" to choose an item in a list.

## Using synonyms

<!-- Axl Rose example -->
```javascript   
         var choices = [
            { 
                value: 'flipCoinDialog',
                action: { title: "Flip A Coin" },
                synonyms: 'toss coin|flip quarter|toss quarter'
            },
            {
                value: 'rollDiceDialog',
                action: { title: "Roll Dice" },
                synonyms: 'roll die|shoot dice|shoot die'
            },
            {
                value: 'magicBallDialog',
                action: { title: "Magic 8-Ball" },
                synonyms: 'shake ball'
            },
            {
                value: 'quit',
                action: { title: "Quit" },
                synonyms: 'exit|stop|end'
            }
        ];
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```

## <a name="configuring-your-bot"></a>봇 구성

## <a name="prompts"></a>프롬프트

## <a name="additional-resources"></a>추가 리소스

Cortana 설명서: [Cortana 기술 설명서](/cortana/skills/)

Cortana SSML 참조: [SSML(Speech Synthesis Markup Language) 참조](/cortana/skills/speech-synthesis-markup-language)
