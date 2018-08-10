---
title: 지역화 지원 | Microsoft Docs
description: 사용자의 위치를 확인하고 .NET용 Bot Builder SDK를 사용하여 지역화 기능을 구현하는 방법을 알아봅니다.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8f58c8d0b884e929c575e01aa5e0fee5c21ea4e6
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303586"
---
# <a name="support-localization"></a>지역화 지원

Bot Builder에는 여러 언어로 사용자와 통신할 수 있는 봇을 빌드하기 위한 풍부한 지역화 시스템이 포함됩니다. 봇의 디렉터리 구조에 저장된 JSON 파일을 사용하여 봇의 프롬프트를 모두 지역화할 수 있습니다. LUIS 같은 시스템을 사용하여 자연어 처리를 수행할 경우 봇이 지원하는 언어별 모델이 포함된 [LuisRecognizer][LUISRecognizer]를 구성할 수 있으며, SDK가 사용자의 기본 설정 로캘과 일치하는 모델을 자동으로 선택합니다.

## <a name="determine-the-locale-by-prompting-the-user"></a>사용자에게 메시지를 표시할 로캘 확인
봇을 지역화하는 첫 번째 단계는 사용자의 기본 설정 언어를 식별하는 기능을 추가하는 것입니다. SDK는 사용자별로 이 기본 설정을 저장하고 검색할 수 있는 [session.preferredLocale()][preferredLocal] 메서드를 제공합니다. 다음 예제는 사용자에게 기본 설정 언어를 묻는 메시지를 표시하고 나서 선택 항목을 저장하는 대화 상자입니다.

``` javascript
bot.dialog('/localePicker', [
    function (session) {
        // Prompt the user to select their preferred locale
        builder.Prompts.choice(session, "What's your preferred language?", 'English|Español|Italiano');
    },
    function (session, results) {
        // Update preferred locale
        var locale;
        switch (results.response.entity) {
            case 'English':
                locale = 'en';
                break;
            case 'Español':
                locale = 'es';
                break;
            case 'Italiano':
                locale = 'it';
                break;
        }
        session.preferredLocale(locale, function (err) {
            if (!err) {
                // Locale files loaded
                session.endDialog(`Your preferred language is now ${results.response.entity}`);
            } else {
                // Problem loading the selected locale
                session.error(err);
            }
        });
    }
]);
```

## <a name="determine-the-locale-by-using-analytics"></a>분석을 사용하여 로캘 확인
사용자의 로캘을 확인하는 또 다른 방법은 [텍스트 분석 API](/azure/cognitive-services/cognitive-services-text-analytics-quick-start) 같은 서비스를 사용하여 사용자가 보낸 메시지의 텍스트를 기반으로 사용자의 언어를 자동으로 검색하는 것입니다.

아래 코드 조각은 이 서비스를 고유한 봇에 통합하는 방법을 보여 줍니다.
``` javascript
var request = require('request');

bot.use({
    receive: function (event, next) {
        if (event.text && !event.textLocale) {
            var options = {
                method: 'POST',
                url: 'https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/languages?numberOfLanguagesToDetect=1',
                body: { documents: [{ id: 'message', text: event.text }]},
                json: true,
                headers: {
                    'Ocp-Apim-Subscription-Key': '<YOUR API KEY>'
                }
            };
            request(options, function (error, response, body) {
                if (!error && body) {
                    if (body.documents && body.documents.length > 0) {
                        var languages = body.documents[0].detectedLanguages;
                        if (languages && languages.length > 0) {
                            event.textLocale = languages[0].iso6391Name;
                        }
                    }
                }
                next();
            });
        } else {
            next();
        }
    }
});
```

위의 코드 조각을 봇에 추가한 후 [session.preferredLocale()][preferredLocal]를 호출하면 검색된 언어가 자동으로 반환됩니다. `preferredLocale()`에 대한 검색 순서는 다음과 같습니다.
1. `session.preferredLocale()`를 호출하여 저장된 로캘입니다. 이 값은 `session.userData['BotBuilder.Data.PreferredLocale']`에 저장됩니다.
2. `session.message.textLocale`에 할당된 검색된 로캘입니다.
3. 봇의 구성된 기본 로캘입니다(예: 영어(‘en’)).

해당 생성자를 사용하여 봇의 기본 로캘을 구성할 수 있습니다.

```javascript
var bot = new builder.UniversalBot(connector, {
    localizerSettings: { 
        defaultLocale: "es" 
    }
});
```

## <a name="localize-prompts"></a>프롬프트 지역화
Bot Builder SDK의 기본 지역화 시스템은 파일을 기반으로 하고 이 시스템을 사용하면 봇에서 디스크에 저장된 JSON 파일을 사용하여 여러 언어를 지원할 수 있습니다. 기본적으로 지역화 시스템은 **./locale/<IETF TAG>/index.json** 파일에서 봇의 프롬프트를 검색합니다. 여기서 <IETF TAG>는 프롬프트를 찾을 기본 제공 로캘을 나타내는 유효한 [IETF 언어 태그][IEFT]입니다. 

다음 스크린샷은 세 가지 언어인 영어, 이탈리아어 및 스페인어를 지원하는 봇의 디렉터리 구조를 보여 줍니다.

![세 가지 로캘에 대한 디렉터리 구조](../media/locale-dir.png)

파일 구조는 지역화된 텍스트 문자열에 대한 메시지 ID의 간단한 JSON 맵입니다. 값이 문자열이 아닌 배열인 경우, [session.localizer.gettext()][GetText]를 사용하여 해당 값을 검색할 때 배열의 한 프롬프트가 임의로 선택됩니다. 

언어별 텍스트 대신에 [session.send()](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send) 호출로 메시지 ID를 전달하는 경우 봇은 메시지의 지역화된 버전을 자동으로 검색합니다.

```javascript
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("greeting");
        session.send("instructions");
        session.beginDialog('/localePicker');
    },
    function (session) {
        builder.Prompts.text(session, "text_prompt");
    }
]);
```

내부적으로 SDK는 [`session.preferredLocale()`][preferredLocale]를 호출하여 사용자의 기본 제공 로캘을 가져오고 나서 [`session.localizer.gettext()`][GetText] 호출에 이 로캘을 사용하여 메시지 ID를 지역화된 텍스트 문자열에 매핑합니다.  지역화 도구를 수동으로 호출해야 하는 경우가 있습니다. 예를 들어, [`Prompts.choice()`][promptsChoice]에 전달되는 열거형 값은 자동으로 지역화되지 않으므로 프롬프트를 호출하기 전에 지역화된 목록을 수동으로 검색해야 할 수 있습니다.

```javascript
var options = session.localizer.gettext(session.preferredLocale(), "choice_options");
builder.Prompts.choice(session, "choice_prompt", options);
```

기본 지역화 도구는 여러 파일에서 메시지 ID를 검색하고, ID를 찾을 수 없거나 지역화 파일이 제공되지 않은 경우에는 단순히 ID 텍스트를 반환하므로 지역화 파일을 투명하고 선택적으로 사용할 수 있습니다.  파일은 다음 순서로 검색됩니다.

1. [`session.preferredLocale()`][preferredLocale]에서 반환된 로캘 아래에서 **index.json** 파일이 검색됩니다.
2. 로캘에 **en-US** 같은 선택적 하위 태그가 포함된 경우, **en**의 루트 태그가 검색됩니다.
3. 봇의 구성된 기본 로캘이 검색됩니다.

## <a name="use-namespaces-to-customize-and-localize-prompts"></a>네임스페이스를 사용하여 프롬프트 사용자 지정 및 지역화
기본 지역화 도구는 프롬프트의 네임스페이스 지정을 지원하여 메시지 ID 간 충돌을 방지합니다.  봇은 네임스페이스가 지정된 프롬프트를 재정의하여 다른 네임스페이스에서 프롬프트를 사용자 지정하거나 내용을 바꿀 수 있습니다.  이 기능을 사용하여 SDK의 기본 제공 메시지를 사용자 지정하면 다른 언어에 대한 지원을 추가하거나 단순히 SDK의 현재 메시지 내용을 바꿀 수 있습니다.  예를 들어, 단순히 **BotBuilder.json**이라는 파일을 봇의 로캘 디렉터리에 추가하고 나서 `default_error` 메시지 ID의 항목을 추가하면 SDK의 기본 오류 메시지를 변경할 수 있습니다.

![로캘 네임스페이스 지정을 위한 BotBuilder.json](../media/locale-namespacing.png)


## <a name="additional-resources"></a>추가 리소스

인식기를 지역화하는 방법을 알아보려면 [의도 인식](bot-builder-nodejs-recognize-intent-messages.md)을 참조하세요.


[LUIS]: https://www.luis.ai/
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html
[LUISRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISSample]: https://github.com/Microsoft/BotBuilder/blob/master/Node/examples/basics-naturalLanguage/app.js
[DisambiguationSample]: https://github.com/Microsoft/BotBuilder/tree/master/Node/examples/feature-onDisambiguateRoute
[preferredLocal]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[preferredLocale]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[promptsChoice]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice
[GetText]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ilocalizer.html#gettext
[IEFT]: https://en.wikipedia.org/wiki/IETF_language_tag

