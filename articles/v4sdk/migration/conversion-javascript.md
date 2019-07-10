---
title: 기존 v3 JavaScript 봇을 새 v4 프로젝트로 마이그레이션 | Microsoft Docs
description: 기존 v3 JavaScript 봇을 가져오고, 새 프로젝트를 사용하여 v4 SDK로 마이그레이션합니다.
keywords: JavaScript, 봇 마이그레이션, 대화 상자, v3 봇
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 573dabba7a16f88db890f0d095a2d4a0f983660c
ms.sourcegitcommit: 41c8caf0e0c849beeeb50cdccf6dbc1ba7cce442
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/24/2019
ms.locfileid: "67344587"
---
# <a name="migrate-a-sdk-v3-javascript-bot-to-v4"></a>SDK v3 Javascript 봇을 v4로 마이그레이션

이 문서에서는 v3 SDK JavaScript [core-MultiDialogs-v3](https://aka.ms/v3-js-core-multidialog-migration-sample) 봇을 새 v4 JavaScript 봇에 포팅합니다.
이 변환은 다음 단계로 구분됩니다.

1. 새 프로젝트를 만들고 종속성을 추가합니다.
1. 진입점을 업데이트하고 상수를 정의합니다.
1. 대화를 만들고 SDK v4를 사용하여 다시 구현합니다.
1. 대화를 실행하도록 봇 코드를 업데이트합니다.
1. **store.js** 유틸리티 파일을 포팅합니다.

이 프로세스가 완료되면 v4 봇이 작동합니다. 변환된 봇의 복사본도 [core-MultiDialogs-v4](https://aka.ms/v4-js-core-multidialog-migration-sample) 리포지토리 샘플에 있습니다.

Bot Framework SDK v4는 SDK v3와 동일한 기본 REST API를 기반으로 합니다. 그러나 SDK v4는 이전 버전의 SDK를 리팩터링하여 개발자가 자신의 봇을 더 유연하게 제어할 수 있도록 합니다. SDK의 주요 변경 내용은 다음과 같습니다.

- 상태가 상태 관리 개체 및 속성 접근자를 통해 관리됩니다.
- 턴을 처리하는 방법, 즉 봇에서 사용자 채널로부터 들어오는 활동을 받고 응답하는 방법이 변경되었습니다.
- v4는 `session` 개체를 사용하지 않습니다. 대신 들어오는 활동에 대한 정보를 포함하고 응답 활동을 다시 보내는 데 사용할 수 있는 _턴 컨텍스트_ 개체를 사용합니다.
- 새 대화 라이브러리는 v3의 대화 라이브러리와 매우 다릅니다. 구성 요소 및 폭포 대화를 사용하여 이전 대화를 새 대화 시스템으로 변환해야 합니다.

<!-- TODO
For more information about specific changes, see [differences between the v3 and v4 JavaScript SDK](???.md).
-->

> [!NOTE]
> 마이그레이션의 일환으로 일부 코드도 정리했지만 v3 논리에 대한 변경 내용만 마이그레이션 프로세스의 일부로 강조 표시합니다.

## <a name="prerequisites"></a>필수 조건

- Node.js
- Visual Studio Code
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)

## <a name="about-this-bot"></a>이 봇에 대한 정보

마이그레이션하는 봇은 여러 대화를 사용하여 대화 흐름을 관리하는 방법을 보여 줍니다. 봇에서는 항공편 또는 호텔 정보를 조회할 수 있습니다.

- 기본 대화는 사용자에게 찾고 있는 정보의 유형을 묻습니다.
- 호텔 대화는 사용자에게 검색 매개 변수를 묻는 메시지를 표시한 다음, 모의 검색을 수행합니다.
- 항공편 대화는 봇에서 catch하여 정상적으로 처리하는 오류를 생성합니다.

## <a name="create-and-open-a-new-v4-bot-project"></a>새 v4 봇 프로젝트 만들기 및 열기

1. 봇 코드를 포팅하는 v4 프로젝트가 필요합니다. 프로젝트를 로컬로 만들려면 [JavaScript용 Bot Framework SDK를 사용하여 봇 만들기](../../javascript/bot-builder-javascript-quickstart.md)를 참조하세요.

    > [!TIP]
    > Azure에서도 프로젝트를 만들 수 있습니다. [Azure Bot Service로 봇 만들기](../../bot-service-quickstart.md)를 참조하세요.
    > 그러나 이러한 두 가지 방법에는 파일을 지원하는 데 약간의 차이가 있습니다. 이 문서에서 사용하는 v4 프로젝트는 로컬 프로젝트로 만들어졌습니다.

1. 그런 다음, Visual Studio Code에서 프로젝트를 엽니다.

## <a name="update-the-packagejson-file"></a>package.json 파일 업데이트

1. Visual Studio Code의 터미널 창에서 `npm i botbuilder-dialogs`를 입력하여 종속성을 **botbuilder-dialogs** 패키지에 추가합니다.

1. **./package.json**을 편집하고, 필요에 따라 `name`, `version`, `description` 및 다른 속성을 업데이트합니다.

## <a name="update-the-v4-app-entry-point"></a>v4 앱 진입점 업데이트

v4 템플릿은 앱 진입점에 대한 **index.js** 파일과 봇 관련 논리에 대한 **bot.js** 파일을 만듭니다. 이후 단계에서는 **bot.js** 파일의 이름을 **bots/reservationBot.js**로 바꾸고, 각 대화에 대한 클래스를 추가합니다.

봇 앱에 대한 진입점인 **./index.js**를 편집합니다. 여기에는 HTTP 서버를 설정하는 v3 **app.js** 파일의 일부가 포함됩니다.

1. `BotFrameworkAdapter` 외에도 **botbuilder** 패키지에서 `MemoryStorage` 및 `ConversationState`를 가져옵니다. 또한 봇 및 기본 대화 모듈도 가져옵니다. (곧 만들겠지만 여기서 참조할 필요가 있습니다.)

    ```javascript
    // Import required bot services.
    // See https://aka.ms/bot-services to learn more about the different parts of a bot.
    const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');

    // This bot's main dialog.
    const { MainDialog } = require('./dialogs/main')
    const { ReservationBot } = require('./bots/reservationBot');
    ```

1. 어댑터에 대한 `onTurnError` 처리기를 정의합니다.

    ```javascript
    // Catch-all for errors.
    adapter.onTurnError = async (context, error) => {
        const errorMsg = error.message ? error.message : `Oops. Something went wrong!`;
        // This check writes out errors to console log .vs. app insights.
        console.error(`\n [onTurnError]: ${ error }`);
        // Clear out state
        await conversationState.delete(context);
        // Send a message to the user
        await context.sendActivity(errorMsg);
    };
    ```

    v4에서는 _봇 어댑터_를 사용하여 들어오는 활동을 봇으로 라우팅합니다. 이 어댑터를 사용하면 턴이 완료되기 전에 오류를 catch하고 대응할 수 있습니다. 여기서는 애플리케이션 오류가 발생하면 대화 상태를 지웁니다. 그러면 모든 대화가 다시 설정되고 봇이 손상된 대화 상태로 유지되지 않도록 합니다.

1. 봇을 만드는 템플릿 코드를 이 코드로 바꿉니다.

    ```javascript
    // Define state store for your bot.
    const memoryStorage = new MemoryStorage();

    // Create conversation state with in-memory storage provider.
    const conversationState = new ConversationState(memoryStorage);

    // Create the base dialog and bot
    const dialog = new MainDialog();
    const reservationBot = new ReservationBot(conversationState, dialog);
    ```

    이제 메모리 내 스토리지 계층이 `MemoryStorage` 클래스에서 제공되므로 대화 상태 관리 개체를 명시적으로 만들어야 합니다.

    대화 정의 코드가 곧 정의될 `MainDialog` 클래스로 이동되었습니다. 또한 봇 정의 코드를 `ReservationBot` 클래스로 마이그레이션합니다.

1. 마지막으로, 어댑터를 사용하여 활동을 봇으로 라우팅하도록 서버의 요청 처리기를 업데이트합니다.

    ```javascript
    // Listen for incoming requests.
    server.post('/api/messages', (req, res) => {
        adapter.processActivity(req, res, async (context) => {
            // Route incoming activities to the bot.
            await reservationBot.run(context);
        });
    });
    ```

    v4에서 봇은 턴에 대한 활동을 받는 `run` 메서드를 정의하는 `ActivityHandler`에서 파생됩니다.

## <a name="add-a-constants-file"></a>상수 파일 추가

봇에 대한 식별자를 저장하는 **./const.js** 파일을 만듭니다.

```javascript
module.exports = {
    MAIN_DIALOG: 'mainDialog',
    INITIAL_PROMPT: 'initialPrompt',
    HOTELS_DIALOG: 'hotelsDialog',
    INITIAL_HOTEL_PROMPT: 'initialHotelPrompt',
    CHECKIN_DATETIME_PROMPT: 'checkinTimePrompt',
    HOW_MANY_NIGHTS_PROMPT: 'howManyNightsPrompt',
    FLIGHTS_DIALOG: 'flightsDialog',
};
```

v4에서 ID는 대화 및 프롬프트 개체에 할당되고, 대화 및 프롬프트는 ID로 호출됩니다.

## <a name="create-new-dialog-files"></a>새 대화 파일 만들기

다음 파일을 만듭니다.

| 파일 이름 | 설명 |
|:---|:---|
| **./dialogs/flights.js** | `hotels` 대화에 대한 마이그레이션된 논리가 포함됩니다. |
| **./dialogs/hotels.js** | `flights` 대화에 대한 마이그레이션된 논리가 포함됩니다. |
| **./dialogs/main.js** | 봇에 대한 마이그레이션된 논리가 포함되며, _루트_ 대화를 대신합니다. |

지원 대화는 마이그레이션하지 않았습니다. v4에서 도움말 대화를 구현하는 방법에 대한 예제는 [사용자 중단 처리](../bot-builder-howto-handle-user-interrupt.md?tabs=javascript)를 참조하세요.

### <a name="implement-the-main-dialog"></a>기본 대화 구현

v3에서 모든 봇은 대화 시스템을 기반으로 하여 만들어졌습니다. 이제 v4에서는 봇 논리와 대화 논리가 분리되었습니다. v3 봇에서 _루트 대화_를 가져와서 이를 대신하는 `MainDialog` 클래스를 만들었습니다.

**./dialogs/main.js**를 편집합니다.

1. 대화에 필요한 클래스 및 상수를 가져옵니다.

    ```javascript
    const { DialogSet, DialogTurnStatus, ComponentDialog, WaterfallDialog,
        ChoicePrompt } = require('botbuilder-dialogs');
    const { FlightDialog } = require('./flights');
    const { HotelsDialog } = require('./hotels');
    const { MAIN_DIALOG,
        INITIAL_PROMPT,
        HOTELS_DIALOG,
        FLIGHTS_DIALOG
    } = require('../const');
    ```

1. `MainDialog` 클래스를 정의하고 내보냅니다.

    ```javascript
    const initialId = 'mainWaterfallDialog';

    class MainDialog extends ComponentDialog {
        constructor() {
            super(MAIN_DIALOG);

            // Create a dialog set for the bot. It requires a DialogState accessor, with which
            // to retrieve the dialog state from the turn context.
            this.addDialog(new ChoicePrompt(INITIAL_PROMPT, this.validateNumberOfAttempts.bind(this)));
            this.addDialog(new FlightDialog(FLIGHTS_DIALOG));

            // Define the steps of the base waterfall dialog and add it to the set.
            this.addDialog(new WaterfallDialog(initialId, [
                this.promptForBaseChoice.bind(this),
                this.respondToBaseChoice.bind(this)
            ]));

            // Define the steps of the hotels waterfall dialog and add it to the set.
            this.addDialog(new HotelsDialog(HOTELS_DIALOG));

            this.initialDialogId = initialId;
        }
    }

    module.exports.MainDialog = MainDialog;
    ```

    그러면 다른 대화가 선언되고 기본 대화에서 직접 참조한다는 메시지가 표시됩니다.

    - 이 대화에 대한 단계가 포함된 기본 폭포 대화. 구성 요소 대화가 시작되면 _초기 대화_가 시작됩니다.
    - 사용자에게 수행할 작업을 묻는 데 사용할 선택 프롬프트. 유효성 검사기를 사용하여 선택 프롬프트를 만들었습니다.
    - 두 개의 자식 대화(항공편 및 호텔).

1. `run` 도우미 메서드를 클래스에 추가합니다.

    ```javascript
    /**
     * The run method handles the incoming activity (in the form of a TurnContext) and passes it through the dialog system.
     * If no dialog is active, it will start the default dialog.
     * @param {*} turnContext
     * @param {*} accessor
     */
    async run(turnContext, accessor) {
        const dialogSet = new DialogSet(accessor);
        dialogSet.add(this);

        const dialogContext = await dialogSet.createContext(turnContext);
        const results = await dialogContext.continueDialog();
        if (results.status === DialogTurnStatus.empty) {
            await dialogContext.beginDialog(this.id);
        }
    }
    ```

    v4에서 봇은 먼저 대화 컨텍스트를 만든 다음, `continueDialog`를 호출하여 대화 시스템과 상호 작용합니다. 활성 대화가 있으면 제어가 전달되고, 그렇지 않으면 이 호출이 반환됩니다. `empty`의 결과에서 활성 대화가 없음을 나타내므로 여기서 기본 대화가 다시 시작됩니다.

    `accessor` 매개 변수는 대화 상태 속성에 대한 접근자를 전달합니다. _대화 스택_에 대한 상태가 이 속성에 저장됩니다. v4에서 상태 및 대화가 작동하는 방법에 대한 자세한 내용은 [상태 관리](../bot-builder-concept-state.md) 및 [대화 라이브러리](../bot-builder-concept-dialog.md)를 참조하세요.

1. 기본 대화의 폭포 단계와 선택 프롬프트에 대한 유효성 검사기를 클래스에 추가합니다.

    ```javascript
    async promptForBaseChoice(stepContext) {
        return await stepContext.prompt(
            INITIAL_PROMPT, {
                prompt: 'Are you looking for a flight or a hotel?',
                choices: ['Hotel', 'Flight'],
                retryPrompt: 'Not a valid option'
            }
        );
    }

    async respondToBaseChoice(stepContext) {
        // Retrieve the user input.
        const answer = stepContext.result.value;
        if (!answer) {
            // exhausted attempts and no selection, start over
            await stepContext.context.sendActivity('Not a valid option. We\'ll restart the dialog ' +
                'so you can try again!');
            return await stepContext.endDialog();
        }
        if (answer === 'Hotel') {
            return await stepContext.beginDialog(HOTELS_DIALOG);
        }
        if (answer === 'Flight') {
            return await stepContext.beginDialog(FLIGHTS_DIALOG);
        }
        return await stepContext.endDialog();
    }

    async validateNumberOfAttempts(promptContext) {
        if (promptContext.attemptCount > 3) {
            // cancel everything
            await promptContext.context.sendActivity('Oops! Too many attempts :( But don\'t worry, I\'m ' +
                'handling that exception and you can try again!');
            return await promptContext.context.endDialog();
        }

        if (!promptContext.recognized.succeeded) {
            await promptContext.context.sendActivity(promptContext.options.retryPrompt);
            return false;
        }
        return true;
    }
    ```

    폭포의 첫 번째 단계는 선택 프롬프트(자체가 대화임)를 시작하여 사용자에게 선택을 요청합니다. 폭포의 두 번째 단계는 선택 프롬프트의 결과를 사용합니다. 자식 대화를 시작하거나(선택한 경우) 기본 대화를 종료합니다(사용자가 선택하지 못한 경우).

    선택 프롬프트는 사용자가 올바르게 선택한 경우 사용자의 선택을 반환하거나 사용자에게 다시 선택하도록 다시 요청합니다. 유효성 검사기는 사용자에게 프롬프트를 연속적으로 표시한 횟수를 확인하고 3회의 시도가 실패하면 프롬프트가 실패하고 제어를 기본 폭포 대화에 반환합니다.

### <a name="implement-the-flights-dialog"></a>항공편 대화 구현

v3 봇에서 항공편 대화는 봇에서 대화 오류를 처리하는 방식을 보여 주는 스텁이었습니다. 여기서도 마찬가지입니다.

**./dialogs/flights.js**를 편집합니다.

```javascript
const { ComponentDialog, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'flightsWaterfallDialog';

class FlightDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(initialId, [
            async () => {
                throw new Error('Flights Dialog is not implemented and is instead ' +
                    'being used to show Bot error handling');
            }
        ]));
    }
}

exports.FlightDialog = FlightDialog;
```

### <a name="implement-the-hotels-dialog"></a>호텔 대화 구현

호텔 대화의 전체 흐름(목적지 요청, 날짜 요청, 숙박 일 수 요청)을 동일하게 유지한 다음, 사용자에게 검색과 일치하는 옵션 목록을 표시합니다.

**./dialogs/hotels.js**를 편집합니다.

1. 대화에 필요한 클래스 및 상수를 가져옵니다.

    ```javascript
    const { ComponentDialog, WaterfallDialog, TextPrompt, DateTimePrompt } = require('botbuilder-dialogs');
    const { AttachmentLayoutTypes, CardFactory } = require('botbuilder');
    const store = require('../store');
    const {
        INITIAL_HOTEL_PROMPT,
        CHECKIN_DATETIME_PROMPT,
        HOW_MANY_NIGHTS_PROMPT
    } = require('../const');
    ```

1. `HotelsDialog` 클래스를 정의하고 내보냅니다.

    ```javascript
    const initialId = 'hotelsWaterfallDialog';

    class HotelsDialog extends ComponentDialog {
        constructor(id) {
            super(id);

            // ID of the child dialog that should be started anytime the component is started.
            this.initialDialogId = initialId;

            // Register dialogs
            this.addDialog(new TextPrompt(INITIAL_HOTEL_PROMPT));
            this.addDialog(new DateTimePrompt(CHECKIN_DATETIME_PROMPT));
            this.addDialog(new TextPrompt(HOW_MANY_NIGHTS_PROMPT));

            // Define the conversation flow using a waterfall model.
            this.addDialog(new WaterfallDialog(initialId, [
                this.destinationPromptStep.bind(this),
                this.destinationSearchStep.bind(this),
                this.checkinPromptStep.bind(this),
                this.checkinTimeSetStep.bind(this),
                this.stayDurationPromptStep.bind(this),
                this.stayDurationSetStep.bind(this),
                this.hotelSearchStep.bind(this)
            ]));
        }
    }

    exports.HotelsDialog = HotelsDialog;
    ```

1. 대화 단계에서 사용할 몇 가지 도우미 함수를 클래스에 추가합니다.

    ```javascript
    addDays(startDate, days) {
        const date = new Date(startDate);
        date.setDate(date.getDate() + days);
        return date;
    };

    createHotelHeroCard(hotel) {
        return CardFactory.heroCard(
            hotel.name,
            `${hotel.rating} stars. ${hotel.numberOfReviews} reviews. From ${hotel.priceStarting} per night.`,
            CardFactory.images([hotel.image]),
            CardFactory.actions([
                {
                    type: 'openUrl',
                    title: 'More details',
                    value: `https://www.bing.com/search?q=hotels+in+${encodeURIComponent(hotel.location)}`
                }
            ])
        );
    }
    ```

    `createHotelHeroCard`는 호텔에 대한 정보가 포함된 영웅 카드를 만듭니다.

1. 대화에서 사용되는 폭포 단계를 클래스에 추가합니다.

    ```javascript
    async destinationPromptStep(stepContext) {
        await stepContext.context.sendActivity('Welcome to the Hotels finder!');
        return await stepContext.prompt(
            INITIAL_HOTEL_PROMPT, {
                prompt: 'Please enter your destination'
            }
        );
    }

    async destinationSearchStep(stepContext) {
        const destination = stepContext.result;
        stepContext.values.destination = destination;
        await stepContext.context.sendActivity(`Looking for hotels in ${destination}`);
        return stepContext.next();
    }

    async checkinPromptStep(stepContext) {
        return await stepContext.prompt(
            CHECKIN_DATETIME_PROMPT, {
                prompt: 'When do you want to check in?'
            }
        );
    }

    async checkinTimeSetStep(stepContext) {
        const checkinTime = stepContext.result[0].value;
        stepContext.values.checkinTime = checkinTime;
        return stepContext.next();
    }

    async stayDurationPromptStep(stepContext) {
        return await stepContext.prompt(
            HOW_MANY_NIGHTS_PROMPT, {
                prompt: 'How many nights do you want to stay?'
            }
        );
    }

    async stayDurationSetStep(stepContext) {
        const numberOfNights = stepContext.result;
        stepContext.values.numberOfNights = parseInt(numberOfNights);
        return stepContext.next();
    }

    async hotelSearchStep(stepContext) {
        const destination = stepContext.values.destination;
        const checkIn = new Date(stepContext.values.checkinTime);
        const checkOut = this.addDays(checkIn, stepContext.values.numberOfNights);

        await stepContext.context.sendActivity(`Ok. Searching for Hotels in ${destination} from 
            ${checkIn.toDateString()} to ${checkOut.toDateString()}...`);
        const hotels = await store.searchHotels(destination, checkIn, checkOut);
        await stepContext.context.sendActivity(`I found in total ${hotels.length} hotels for your dates:`);

        const hotelHeroCards = hotels.map(this.createHotelHeroCard);

        await stepContext.context.sendActivity({
            attachments: hotelHeroCards,
            attachmentLayout: AttachmentLayoutTypes.Carousel
        });

        return await stepContext.endDialog();
    }
    ```

    v3 호텔 대화의 단계에서 v4 호텔 대화의 폭포 단계로 마이그레이션했습니다.

## <a name="update-the-bot"></a>봇 업데이트

v4에서 봇은 대화 시스템 외부의 활동에 대응할 수 있습니다. `ActivityHandler` 클래스는 일반적인 유형의 활동에 대한 처리기를 정의하여 코드를 더 쉽게 관리할 수 있도록 합니다.

**./bot.js**의 이름을 **./bots/reservationBot.js**로 바꾸고 편집합니다.

1. 이 파일은 봇의 기본 구현을 제공하는 `ActivityHandler`를 이미 가져옵니다.

    ```javascript
    const { ActivityHandler } = require('botbuilder');
    ```

1. 클래스의 이름을 `ReservationBot`으로 변경합니다.

    ```javascript
    class ReservationBot extends ActivityHandler {
        // ...
    }

    module.exports.ReservationBot = ReservationBot;
    ```

1. 받는 개체를 수락하도록 생성자의 시그니처를 업데이트합니다.

    ```javascript
    /**
     *
     * @param {ConversationState} conversationState
     * @param {Dialog} dialog
     * @param {any} logger object for logging events, defaults to console if none is provided
    */
    constructor(conversationState, dialog, logger) {
        super();
        // ...
    }
    ```

1. 생성자에서 null 매개 변수 검사를 추가하고 클래스 생성자 속성을 정의합니다.

    ```javascript
    if (!conversationState) throw new Error('[DialogBot]: Missing parameter. conversationState is required');
    if (!dialog) throw new Error('[DialogBot]: Missing parameter. dialog is required');
    if (!logger) {
        logger = console;
        logger.log('[DialogBot]: logger not passed in, defaulting to console');
    }

    this.conversationState = conversationState;
    this.dialog = dialog;
    this.logger = logger;
    this.dialogState = this.conversationState.createProperty('DialogState');
    ```

    그러면 대화 스택의 상태를 저장할 대화 상태 속성 접근자가 만들어집니다.

1. 생성자에서 `onMessage` 처리기를 업데이트하고, `onDialog` 처리기를 추가합니다.

    ```javascript
    this.onMessage(async (context, next) => {
        this.logger.log('Running dialog with Message Activity.');

        // Run the Dialog with the new message Activity.
        await this.dialog.run(context, this.dialogState);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });

    this.onDialog(async (context, next) => {
        // Save any state changes. The load happened during the execution of the Dialog.
        await this.conversationState.saveChanges(context, false);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler`는 메시지 활동을 `onMessage`로 라우팅합니다. 이 봇은 대화를 통해 모든 사용자 입력을 처리합니다.

    `ActivityHandler`는 턴이 종료될 때 `onDialog`를 호출하여 제어를 어댑터에 반환합니다. 따라서 턴을 종료하기 전에 상태를 명시적으로 저장해야 합니다. 그렇지 않으면 상태 변경 내용이 저장되지 않고 대화가 제대로 실행되지 않습니다.

1. 마지막으로, 생성자에서 `onMembersAdded` 처리기를 업데이트합니다.

    ```javascript
    this.onMembersAdded(async (context, next) => {
        const membersAdded = context.activity.membersAdded;
        for (let cnt = 0; cnt < membersAdded.length; ++cnt) {
            if (membersAdded[cnt].id !== context.activity.recipient.id) {
                await context.sendActivity('Hello and welcome to Contoso help desk bot.');
            }
        }
        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    봇 이외의 참가자가 대화에 추가되었음을 나타내는 대화 업데이트 활동을 받으면 `ActivityHandler`에서 `onMembersAdded`를 호출합니다. 사용자가 대화에 참여하면 인사말 메시지를 보내도록 이 메서드를 업데이트합니다.

## <a name="create-the-store-file"></a>저장소 파일 만들기

호텔 대화에서 사용되는 **./store.js** 파일을 만듭니다. `searchHotels`는 v3 봇에서 사용하는 것과 동일한 모의 호텔 검색 함수입니다.

```javascript
module.exports = {
    searchHotels: destination => {
        return new Promise(resolve => {

            // Filling the hotels results manually just for demo purposes
            const hotels = [];
            for (let i = 1; i <= 5; i++) {
                hotels.push({
                    name: `${destination} Hotel ${i}`,
                    location: destination,
                    rating: Math.ceil(Math.random() * 5),
                    numberOfReviews: Math.floor(Math.random() * 5000) + 1,
                    priceStarting: Math.floor(Math.random() * 450) + 80,
                    image: `https://placeholdit.imgix.net/~text?txtsize=35&txt=Hotel${i}&w=500&h=260`
                });
            }

            hotels.sort((a, b) => a.priceStarting - b.priceStarting);

            // complete promise with a timer to simulate async response
            setTimeout(() => { resolve(hotels); }, 1000);
        });
    }
};
```

## <a name="test-the-bot-in-the-emulator"></a>Emulator에서 봇 테스트

이 시점에서 봇을 로컬로 실행하고 Emulator를 사용하여 이 봇에 연결할 수 있어야 합니다.

1. 샘플을 머신에서 로컬로 실행합니다.
    Visual Studio Code에서 디버깅 세션을 시작하면 봇을 테스트하면서 로깅 정보를 디버그 콘솔로 보냅니다.
1. 에뮬레이터를 시작하고 봇에 연결합니다.
1. 기본, 항공편 및 호텔 대화를 테스트하기 위한 메시지를 보냅니다.

## <a name="additional-resources"></a>추가 리소스

v4 개념 항목:

- [봇 작동 방식](../bot-builder-basics.md)
- [상태 관리](../bot-builder-concept-state.md)
- [다이얼로그 라이브러리](../bot-builder-concept-dialog.md)

v4 방법 항목:

- [문자 메시지 보내기 및 받기](../bot-builder-howto-send-messages.md)
- [사용자 및 대화 데이터 저장](../bot-builder-howto-v4-state.md)
- [순차적 대화 흐름 구현](../bot-builder-dialog-manage-conversation-flow.md)
- [에뮬레이터를 사용하여 디버그](../../bot-service-debug-emulator.md)
- [봇에 원격 분석 추가](../bot-builder-telemetry.md)
