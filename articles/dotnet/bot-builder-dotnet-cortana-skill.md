---
title: .NET을 사용하여 Cortana 기술 빌드 | Microsoft Docs
description: .NET용 Bot Builder SDK에서 Cortana 기술을 빌드하는 경우의 핵심 개념을 알아봅니다.
keywords: Bot Framework, Cortana 기술, 음성, .NET, Bot Builder, SDK, 주요 개념, 핵심 개념
author: DeniseMak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e6bfd890944cfea052e07ee99451ab90db75415b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300696"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>Cortana 기술을 사용하여 음성 지원 봇 빌드
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.js](../nodejs/bot-builder-nodejs-cortana-skill.md)


.NET용 Bot Builder SDK를 사용하면 Cortana 기술로 Cortana 채널에 연결하여 음성 지원 봇을 빌드할 수 있습니다. 


> [!TIP]
> 기술 정의 및 수행할 수 있는 작업에 대한 자세한 내용은 [Cortana 기술 키트][CortanaGetStarted]를 참조하세요.

Bot Framework를 사용하여 Cortana 기술을 만드는 경우 Cortana 관련 지식이 거의 필요하지 않으며, 주로 봇 빌드로 이루어집니다. 이전에 만든 다른 봇과의 주요 차이점 중 하나는 Cortana에는 시각적 구성 요소와 오디오 구성 요소가 둘 다 포함된다는 것입니다. 시각적 구성 요소의 경우, Cortana는 카드 등의 콘텐츠 렌더링을 위한 캔버스 영역을 제공합니다. 오디오 구성 요소의 경우, 봇 메시지에 텍스트 또는 SSML을 제공하면 Cortana가 사용자에게 읽어주므로 봇에 음성을 제공합니다. 

> [!NOTE]
> Cortana는 다양한 장치에서 사용할 수 있습니다. 일부 장치에는 화면이 있고, 독립 실행형 스피커와 같은 장치에는 화면이 없을 수 있습니다. 봇이 두 시나리오를 모두 처리할 수 있도록 해야 합니다. 장치 정보를 확인하는 방법을 알아보려면 참조 [Cortana 관련 엔터티][CortanaSpecificEntities]를 참조하세요.

## <a name="adding-speech-to-your-bot"></a>봇에 음성 추가

봇의 음성 메시지는 SSML(Speech Synthesis Markup Language)로 표현됩니다. Bot Builder SDK를 사용하면 봇 응답에 SSML을 포함하여 봇이 보여 주는 내용뿐 아니라 봇이 말하는 내용도 제어할 수 있습니다.  봇이 사용자 입력을 허용하거나, 필요로 하거나, 무시하는지 지정하여 Cortana 마이크의 상태를 제어할 수도 있습니다.

`IMessageActivity` 개체의 `Speak` 속성을 설정하여 Cortana가 말할 메시지를 지정합니다. 일반 텍스트를 지정하는 경우 Cortana가 단어의 발음 방법을 결정합니다. 

```cs
Activity reply = activity.CreateReply("This is the text that Cortana displays."); 
reply.Speak = "This is the text that Cortana will say.";
```

피치, 톤, 강조를 자세히 제어하려면 `Speak` 속성의 형식을 [SSML(Speech Synthesis Markup Language)](http://www.w3.org/TR/speech-synthesis/)로 지정합니다.  

다음 코드 예제에서는 단어 “텍스트”를 보통 수준의 강조로 말하도록 지정합니다.
```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "<speak version=\"1.0\" xmlns=\"http://www.w3.org/2001/10/synthesis\" xml:lang=\"en-US\">This is the <emphasis level=\"moderate\">text</emphasis> that will be spoken.</speak>";
```


**InputHint** 속성은 봇에 입력이 필요한지 여부를 Cortana에 알리는 데 도움이 됩니다. 기본값은 프롬프트를 위한 **ExpectingInput** 및 다른 형식의 응답을 위한 **AcceptingInput**입니다.


| 값 | 설명 |
|------|------|
| **AcceptingInput** | 봇이 소극적으로 입력을 받을 준비가 되었지만 응답을 기다리고 있지는 않습니다. Cortana는 사용자가 마이크 단추를 길게 누를 경우 사용자 입력을 허용합니다.|
| **ExpectingInput** | 봇이 사용자 응답을 적극적으로 필요로 함을 나타냅니다. Cortana는 사용자가 마이크에 말하는 내용을 수신 대기합니다.  |
| **IgnoringInput** | Cortana가 입력을 무시합니다. 봇은 요청을 적극적으로 처리 중이며 요청이 완료될 때까지 사용자 입력을 무시하는 경우 이 힌트를 보낼 수 있습니다.  |

<!-- TODO: tip about time limit and batching -->

이 예제에서는 사용자 입력이 필요함을 Cortana에 알리는 방법을 보여 줍니다. 마이크가 열려 있습니다.
```cs
// Add an InputHint to let Cortana know to expect user input
Activity reply = activity.CreateReply("This is the text that will be displayed."); 
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.ExpectingInput;
```



## <a name="display-cards-in-cortana"></a>Cortana에서 카드 표시

음성 응답 외에도 Cortana에서 카드 첨부 파일을 표시할 수 있습니다. Cortana는 다음과 같은 서식 있는 카드를 지원합니다.

| 카드 종류 | 설명 |
|----|----|
| [HeroCard][heroCard] | 일반적으로 하나의 큰 이미지, 하나 이상의 단추 및 텍스트를 포함하는 카드입니다. |
| [ThumbnailCard][thumbnailCard] | 일반적으로 하나의 미리 보기 이미지, 하나 이상의 단추 및 텍스트를 포함하는 카드입니다. |
| [ReceiptCard][receiptCard] | 봇이 사용자에게 확인 메일을 제공할 수 있는 카드입니다. 일반적으로 확인 메일, 세금 및 총 정보에 포함할 항목 목록과 기타 텍스트를 포함합니다. |
| [SignInCard][signinCard] | 봇이 사용자가 로그인하도록 요청할 수 있는 카드입니다. 일반적으로 사용자가 로그인 프로세스를 시작하기 위해 클릭할 수 있는 하나 이상의 단추와 텍스트를 포함합니다. |


이러한 카드가 Cortana 내에서 어떻게 표시되는지 확인하려면 [카드 디자인 모범 사례][CardDesign]를 참조하세요. 봇에서 서식 있는 카드를 사용하는 방법의 예는 [메시지에 서식 있는 카드 첨부 파일 추가](bot-builder-dotnet-add-rich-card-attachments.md)를 참조하세요. 

<!--
The following code demonstrates how to add the `Speak` and `InputHint` properties to a message containing a `HeroCard`.
-->


## <a name="sample-rollerskill"></a>샘플: RollerSkill
다음 섹션의 코드는 주사위 굴리기에 대한 샘플 Cortana 기술에서 가져온 것입니다. [BotBuilder 샘플 리포지토리](https://github.com/Microsoft/BotBuilder-Samples/)에서 봇의 전체 코드를 다운로드합니다.

Cortana에 해당 [호출 이름][InvocationNameGuidelines]을 말하여 기술을 호출합니다. 롤러 기술의 경우, [Cortana 채널에 봇을 추가][CortanaChannel]하고 Cortana 기술로 등록한 후 Cortana에 “Ask Roller” 또는 “Ask Roller to roll dice”라고 말해 기술을 호출할 수 있습니다.

### <a name="explore-the-code"></a>코드 탐색



적절한 대화 상자를 호출하기 위해 `RootDispatchDialog.cs`에 정의된 활동 처리기는 정규식을 사용하여 사용자 입력과 일치시킵니다. 예를 들어, 다음 예제의 처리기는 사용자가 “I'd like to roll some dice”와 같이 말할 경우 트리거됩니다. 유사한 발화도 대화 상자를 트리거하도록 동의어가 정규식에 포함됩니다.
```cs
        [RegexPattern("(roll|role|throw|shoot).*(dice|die|dye|bones)")]
        [RegexPattern("new game")]
        [ScorableGroup(1)]
        public async Task NewGame(IDialogContext context, IActivity activity)
        {
            context.Call(new CreateGameDialog(), AfterGameCreated);
        }
```

`CreateGameDialog` 대화 상자는 봇이 재생할 사용자 지정 게임을 설정합니다. `PromptDialog`를 사용하여 원하는 주사위의 면 수와 주사위를 굴릴 횟수를 사용자에게 묻습니다. 프롬프트를 초기화하는 데 사용되는 `PromptOptions` 개체에는 프롬프트 음성 버전의 `speak` 속성이 포함되어 있습니다.

```cs
    [Serializable]
    public class CreateGameDialog : IDialog<GameData>
    {
        public async Task StartAsync(IDialogContext context)
        {
            context.UserData.SetValue<GameData>(Utils.GameDataKey, new GameData());

            var descriptions = new List<string>() { "4 Sides", "6 Sides", "8 Sides", "10 Sides", "12 Sides", "20 Sides" };
            var choices = new Dictionary<string, IReadOnlyList<string>>()
             {
                { "4", new List<string> { "four", "for", "4 sided", "4 sides" } },
                { "6", new List<string> { "six", "sex", "6 sided", "6 sides" } },
                { "8", new List<string> { "eight", "8 sided", "8 sides" } },
                { "10", new List<string> { "ten", "10 sided", "10 sides" } },
                { "12", new List<string> { "twelve", "12 sided", "12 sides" } },
                { "20", new List<string> { "twenty", "20 sided", "20 sides" } }
            };

            var promptOptions = new PromptOptions<string>(
                Resources.ChooseSides,
                choices: choices,
                descriptions: descriptions,
                speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseSidesSSML))); // spoken prompt

            PromptDialog.Choice(context, this.DiceChoiceReceivedAsync, promptOptions);
        }

        private async Task DiceChoiceReceivedAsync(IDialogContext context, IAwaitable<string> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                int sides;
                if (int.TryParse(await result, out sides))
                {
                    game.Sides = sides;
                    context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
                }

                var promptText = string.Format(Resources.ChooseCount, sides);

                var promptOption = new PromptOptions<long>(promptText, choices: null, speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseCountSSML)));

                var prompt = new PromptDialog.PromptInt64(promptOption);
                context.Call<long>(prompt, this.DiceNumberReceivedAsync);
            }
        }

        private async Task DiceNumberReceivedAsync(IDialogContext context, IAwaitable<long> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                game.Count = await result;
                context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
            }

            context.Done(game);
        }
    }
```

`PlayGameDialog`는 결과를 `HeroCard`에 표시하고 `Speak` 메서드를 사용해서 말할 음성 메시지를 작성하여 결과를 렌더링합니다.

```cs
   [Serializable]
    public class PlayGameDialog : IDialog<object>
    {
        private const string RollAgainOptionValue = "roll again";

        private const string NewGameOptionValue = "new game";

        private GameData gameData;

        public PlayGameDialog(GameData gameData)
        {
            this.gameData = gameData;
        }

        public async Task StartAsync(IDialogContext context)
        {
            if (this.gameData == null)
            {
                if (!context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out this.gameData))
                {
                    // User started session with "roll again" so let's just send them to
                    // the 'CreateGameDialog'
                    context.Done<object>(null);
                }
            }

            int total = 0;
            var randomGenerator = new Random();
            var rolls = new List<int>();

            // Generate Rolls
            for (int i = 0; i < this.gameData.Count; i++)
            {
                var roll = randomGenerator.Next(1, this.gameData.Sides);
                total += roll;
                rolls.Add(roll);
            }

            // Format rolls results
            var result = string.Join(" . ", rolls.ToArray());
            bool multiLine = rolls.Count > 5;

            var card = new HeroCard()
            {
                Subtitle = string.Format(
                    this.gameData.Count > 1 ? Resources.CardSubtitlePlural : Resources.CardSubtitleSingular,
                    this.gameData.Count,
                    this.gameData.Sides),
                Buttons = new List<CardAction>()
                {
                    new CardAction(ActionTypes.ImBack, "Roll Again", value: RollAgainOptionValue),
                    new CardAction(ActionTypes.ImBack, "New Game", value: NewGameOptionValue)
                }
            };

            if (multiLine)
            {
                card.Text = result;
            }
            else
            {
                card.Title = result;
            }

            var message = context.MakeMessage();
            message.Attachments = new List<Attachment>()
            {
                card.ToAttachment()
            };

            // Determine bots reaction for speech purposes
            string reaction = "normal";

            var min = this.gameData.Count;
            var max = this.gameData.Count * this.gameData.Sides;
            var score = total / max;
            if (score == 1)
            {
                reaction = "Best";
            }
            else if (score == 0)
            {
                reaction = "Worst";
            }
            else if (score <= 0.3)
            {
                reaction = "Bad";
            }
            else if (score >= 0.8)
            {
                reaction = "Good";
            }

            // Check for special craps rolls
            if (this.gameData.Type == "Craps")
            {
                switch (total)
                {
                    case 2:
                    case 3:
                    case 12:
                        reaction = "CrapsLose";
                        break;
                    case 7:
                        reaction = "CrapsSeven";
                        break;
                    case 11:
                        reaction = "CrapsEleven";
                        break;
                    default:
                        reaction = "CrapsRetry";
                        break;
                }
            }

            // Build up spoken response
            var spoken = string.Empty;
            if (this.gameData.Turns == 0)
            {
                spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"Start{this.gameData.Type}GameSSML"));
            }

            spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"{reaction}RollReactionSSML"));

            message.Speak = SSMLHelper.Speak(spoken);

            // Increment number of turns and store game to roll again
            this.gameData.Turns++;
            context.UserData.SetValue<GameData>(Utils.GameDataKey, this.gameData);

            // Send card and bots reaction to user.
            message.InputHint = InputHints.AcceptingInput;
            await context.PostAsync(message);

            context.Done<object>(null);
        }
    }
```
## <a name="next-steps"></a>다음 단계

봇이 로컬에서 실행되거나 클라우드에 배포된 경우 Cortana에서 호출할 수 있습니다. Cortana 기술을 사용해 보는 데 필요한 단계는 [Cortana 기술 테스트](../bot-service-debug-cortana-skill.md)를 참조하세요.


## <a name="additional-resources"></a>추가 리소스
* [Cortana 기술 키트][CortanaGetStarted]
* [메시지에 음성 추가](bot-builder-dotnet-text-to-speech.md)
* [SSML 참조][SSMLRef]
* [Cortana 음성 디자인 모범 사례][VoiceDesign]
* [Cortana 카드 디자인 모범 사례][CardDesign]
* [Cortana 개발자 센터][CortanaDevCenter]
* [Cortana 테스트 및 디버그 모범 사례][Cortana-TestBestPractice]
* <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Builder SDK 참조</a>

[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/

[SSMLRef]: https://aka.ms/cortana-ssml
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

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 


