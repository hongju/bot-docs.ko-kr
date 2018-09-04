---
title: 사용자 입력 변환 | Microsoft Docs
description: 사용자 입력을 자동으로 봇의 네이티브 언어로 번역하고 사용자의 언어로 다시 번역하는 방법.
keywords: 번역, 번역하기, 다국어, microsoft translator
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/06/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6304e328e523e73894473620fc1fb7656a8776bf
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756379"
---
# <a name="translate-user-input"></a>사용자 입력 변환 

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇은 [Microsoft Translator](https://www.microsoft.com/en-us/translator/)를 사용하여 봇이 이해하는 언어로 메시지를 자동으로 번역하고 필요에 따라 봇의 응답을 사용자의 언어로 다시 번역할 수 있습니다. 봇에 번역을 추가하여 봇의 중요한 핵심 프로그래밍 부분을 변경하지 않으면서 대상 사용자를 광범위하게 확장할 수 있습니다.
<!-- 
- [Get a Text Services key](#get-a-text-services-key)
- [Installing Packages](#installing-packages)
- [Combine translation with LUIS or QnA Maker](#using-luis)
- [Combine translation with QnA Maker](#using-qna-maker)
-->

이 자습서에서는 번역을 위해 Microsoft Translator 서비스를 사용하고 간단한 봇에 추가하여 작동 방식을 보여 줍니다.

## <a name="get-a-text-services-key"></a>텍스트 서비스 키 가져오기

먼저 Translator Text 서비스를 사용하기 위한 키가 필요합니다. Azure Portal에서 [평가판 키](https://www.microsoft.com/en-us/translator/trial.aspx#get-started)를 가져올 수 있습니다.

## <a name="installing-packages"></a>패키지 설치

봇에 번역을 추가하는 데 필요한 패키지가 있는지 확인합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

다음과 같은 NuGet 패키지의 시험판 버전에 [참조를 추가](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)합니다.

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai.Translation`(번역에 필수)

번역을 LUIS(Language Understanding)와 결합하고 다음에 참조를 추가하려는 경우:

* `Microsoft.Bot.Builder.Ai.Luis`(LUIS에 필수)

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

이러한 서비스 중 하나를 botbuilder-ai 패키지를 사용하여 봇에 추가할 수 있습니다. npm을 통해 프로젝트에 이 패키지를 추가할 수 있습니다.
* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---

## <a name="configure-translation"></a>번역 구성

다음으로, 간단히 봇의 미들웨어 스택에 추가하여 사용자에게서 받은 모든 메시지에 대한 번역기를 호출하도록 봇을 구성할 수 있습니다. 미들웨어는 번역 결과를 사용하여 컨텍스트 개체를 사용하는 사용자의 메시지를 수정합니다.


# <a name="ctabcs"></a>[C#](#tab/cs)

v4 SDK에서 EchoBot 샘플을 시작하고, `Startup.cs` 파일에서 `ConfigureServices` 메서드를 업데이트하여 봇에 `TranslationMiddleware`를 추가합니다. 그러면 사용자로부터 받은 모든 메시지를 번역하도록 봇을 구성합니다. <!--, by simply adding it to your bot's middleware set. The middleware stores the translation results on the context object. -->
-   Using 문을 사용합니다.
-   번역 미들웨어를 포함하도록 `ConfigureServices` 메서드를 업데이트합니다.

    여기에 나오는 코드 조각은 대부분의 주석을 제거하고 예외 처리 미들웨어를 제거하여 간소화되었습니다.

**Startup.cs**
```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai.Translation;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
```
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;
        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", false));

    });
}


```

> [!TIP] 
> BotBuilder SDK는 방금 전송한 메시지를 기반으로 하는 사용자의 언어를 자동으로 검색합니다. 이 기능을 재정의하려면 추가 콜백 매개 변수를 제공하여 사용자의 언어 검색 및 변경을 위한 고유한 논리를 추가할 수 있습니다.  



사용자가 말한 내용 뒤에 "You sent"를 보내는 `EchoBot.cs`에서 코드를 살펴봅니다.

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Threading.Tasks;

namespace Microsoft.Bot.Samples
{
    public class EchoBot : IBot
    {
        public EchoBot() { }

        public async Task OnTurn(ITurnContext context)
        {
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                // echo back the user's input.
                    await context.SendActivity($"You sent '{context.Activity.Text}'");


                case ActivityTypes.ConversationUpdate:
                    foreach (var newMember in context.Activity.MembersAdded)
                    {
                        if (newMember.Id != context.Activity.Recipient.Id)
                        {
                            await context.SendActivity("Hello and welcome to the echo bot.");
                        }
                    }
                    break;
            }
        }
        
    }
}
```

번역 미들웨어를 추가하면 선택적 매개 변수는 응답을 사용자의 언어로 다시 번역할지 여부를 지정합니다. `Startup.cs`에서 사용자 메시지를 봇의 언어로 번역하도록 `false`를 지정했습니다.

```cs
// The first parameter is a list of languages the bot recognizes
// The second parameter is your API key
// The third parameter specifies whether to translate the bot's replies back to the user's language
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", false));
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

echo 봇을 사용하여 번역 미들웨어를 설정하려면 다음을 app.js로 붙여넣습니다.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const { LanguageTranslator, LocaleConverter } = require('botbuilder-ai');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'] 
});
adapter.use(languageTranslator);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            await context.sendActivity(`You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---

## <a name="run-the-bot-and-see-translated-input"></a>봇 실행 및 번역된 입력 확인

봇을 실행하고, 다른 언어로 몇 가지 메시지를 입력합니다. 봇에서 사용자 메시지를 번역하고 해당 응답에서 번역을 나타내는 것을 볼 수 있습니다.

![봇에서 언어 검색 및 입력 번역](./media/how-to-bot-translate/bot-detects-language-translates-input.png)




## <a name="invoke-logic-in-the-bots-native-language"></a>봇의 네이티브 언어로 논리 호출

이제 영어 단어를 확인하는 논리를 추가합니다. 사용자가 다른 언어로 "help" 또는 "cancel"을 말하면 봇은 영어로 번역하고 영어 단어 "help" 또는 "cancel"을 확인하는 논리가 호출됩니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
`EchoBot.cs`에서 봇의 `OnTurn` 메서드에 있는 메시지 활동에 대한 `case` 문을 업데이트합니다.
```cs
case ActivityTypes.Message:
    // check the message text before calling context.SendActivity
    var messagetext = context.Activity.Text.Trim().ToLower();
    if (string.Equals("help", messagetext))
    {
        await context.SendActivity("You asked for help.");
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
if (context.activity.type === 'message') {
    // check the message text before calling context.sendActivity
    var messagetext = context.activity.text.trim().toLowerCase();
    if(messagetext === "help"){
        await context.sendActivity("you said help");
    }
}
```

---

![봇에서 프랑스어로 help 검색](./media/how-to-bot-translate/bot-detects-help-french.png)



## <a name="translate-replies-back-to-the-users-language"></a>응답을 사용자의 언어로 다시 번역

마지막 생성자 매개 변수를 `true`로 설정하여 응답을 사용자의 언어로 번역할 수도 있습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
`Startup.cs`에서 `ConfigureServices` 메서드의 다음 줄을 업데이트합니다.
```cs
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "TRANSLATION-SUBSCRIPTION-KEY", true));
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);
```

---

## <a name="run-the-bot-to-see-replies-in-the-users-language"></a>봇을 실행하여 사용자의 언어로 응답 확인

봇을 실행하고, 다른 언어로 몇 가지 메시지를 입력합니다. 사용자의 언어를 검색하고 응답을 번역하는 것을 볼 수 있습니다.

![봇에서 언어 검색 및 응답 번역](./media/how-to-bot-translate/bot-detects-language-translates-response.png)


## <a name="adding-logic-for-detecting-or-changing-the-user-language"></a>사용자 언어 검색 또는 변경을 위한 논리 추가

Botbuilder SDK에서 사용자의 언어를 자동으로 검색하게 두는 대신 콜백을 제공하여 사용자의 언어 확인 또는 사용자의 언어가 변경되는 경우의 확인을 위해 사용자 고유의 논리를 추가할 수 있습니다.


다음 예제에서 `CheckUserChangedLanguage` 콜백은 언어를 변경하기 위해 특정 사용자 메시지를 확인합니다. 

# <a name="ctabcs"></a>[C#](#tab/cs)
`Startup.cs`에서 번역 미들웨어에 콜백을 추가합니다.
```csharp
middleware.Add(new TranslationMiddleware(new string[] { "en" },
     "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", null,
     TranslatorLocaleHelper.GetActiveLanguage,
     TranslatorLocaleHelper.CheckUserChangedLanguage));
```
`TranslatorLocalHelper.cs` 파일을 추가하고 TranslatorLocalHelper 클래스 정의에 다음을 추가합니다.
```csharp
    class CurrentUserState
    {
        public string Language { get; set; }
    }

    public static void SetLanguage(ITurnContext context, string language) =>
        context.GetConversationState<CurrentUserState>().Language = language;

    public static bool IsSupportedLanguage(string language) =>
        _supportedLanguages.Contains(language);

    public static async Task<bool> CheckUserChangedLanguage(ITurnContext context)
    {
        bool changeLang = false;
        
        // use a specific message from user to change language
        if (context.Activity.Type == ActivityTypes.Message)
        {
            var messageActivity = context.Activity.AsMessageActivity();
            if (messageActivity.Text.ToLower().StartsWith("set my language to"))
            {
                changeLang = true;
            }
            if (changeLang)
            {
                var newLang = messageActivity.Text.ToLower().Replace("set my language to", "").Trim();
                if (!string.IsNullOrWhiteSpace(newLang)
                        && IsSupportedLanguage(newLang))
                {
                    SetLanguage(context, newLang);
                    await context.SendActivity($@"Changing your language to {newLang}");
                }
                else
                {
                    await context.SendActivity($@"{newLang} is not a supported language.");
                }
                // return true to intercept message from further processesing
                return true;
            }
        }

        return false;
    }

    public static string GetActiveLanguage(ITurnContext context)
    {

        if (context.Activity.Type == ActivityTypes.Message
            && context.GetConversationState<CurrentUserState>() != null
            && context.GetConversationState<CurrentUserState>().Language != null)
        {
            return context.GetConversationState<CurrentUserState>().Language;
        }

        return "en";
    }

```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// When the user inputs 'set my language to fr'
// The bot will automatically change all text to French
async function setUserLanguage(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my language to')) {
        state.language = context.activity.text.toLowerCase().replace('set my language to', '').trim();
        await context.sendActivity(`Setting your language to ${state.language}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    setUserLanguage: setUserLanguage,
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`You said: ${context.activity.text}`)
        }
    });
});

```

---

## <a name="combining-luis-or-qna-with-translation"></a>번역과 LUIS 또는 QnA 결합

봇에서 LUIS 또는 QnA maker와 같은 다른 서비스와 번역을 결합하는 경우 봇의 네이티브 언어를 기대하는 다른 미들웨어에 전달하기 전에 메시지가 번역되도록 번역 미들웨어를 먼저 추가합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;

        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes. Input in these language won't be translated.
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", false));

        // Add LUIS middleware after translation middleware to pass translated messages to LUIS
        middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel("luisAppId", "subscriptionId", new Uri("luisModelBaseUrl"))));
    });
}
```

봇 코드에서 LUIS 결과는 이미 봇의 네이티브 언어로 번역된 입력을 기반으로 합니다. LUIS 앱의 결과를 확인하도록 봇 코드를 수정합니다.

```cs
public async Task OnTurn(ITurnContext context)
{
    switch (context.Activity.Type)
    {
        case ActivityTypes.Message:
            // check LUIS results
            var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
            (string key, double score) topItem = luisResult.GetTopScoringIntent();
            await context.SendActivity($"The top intent was: '{topItem.key}', with score {topItem.score}");

            await context.SendActivity($"You sent '{context.Activity.Text}'");

        case ActivityTypes.ConversationUpdate:
            foreach (var newMember in context.Activity.MembersAdded)
            {
                if (newMember.Id != context.Activity.Recipient.Id)
                {
                    await context.SendActivity("Hello and welcome to the echo bot.");
                }
            }
            break;
        }
    }  
}          
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false
});
adapter.use(languageTranslator);

// Add Luis recognizer middleware
const luisRecognizer = new LuisRecognizer({
    appId: "xxxxxx",
    subscriptionKey: "xxxxxxx"
});
adapter.use(luisRecognizer);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            let results = luisRecognizer.get(context);
            for (const intent in results.intents) {
                await context.sendActivity(`intent: ${intent.toString()} : ${results.intents[intent.toString()]}`)
            }
        }
    });
});
```

---

## <a name="bypass-translation-for-specified-patterns"></a>특정 패턴에 대한 번역 바이패스
적절한 이름과 같은 봇에서 번역하길 원치 않는 특정 단어가 있을 수 있습니다. 번역해서는 안 되는 패턴을 나타내기 위한 정규식을 제공할 수 있습니다. 예를 들어 사용자가 봇의 비 네이티브 언어로 "My name is ..."를 말하고, 해당 이름을 번역하지 않으려는 경우 이를 지정하는 패턴을 사용할 수 있습니다.

# <a name="ctabcs"></a>[C#](#tab/cs)
Startup.cs
```cs
// Pattern representing input to not translate
Dictionary<string, List<string>> patterns = new Dictionary<string, List<string>>();
// single pattern for fr language, to avoid translating what follows "my name is"
patterns.Add("fr", new List<string> { "mon nom est (.+)" });

middleware.Add(new TranslationMiddleware(new string[] { "en" },
    "<YOUR API KEY>", patterns,
    TranslatorLocaleHelper.GetActiveLanguage,
    TranslatorLocaleHelper.CheckUserChangedLanguage));

```
<!-- TODO: ADD more explanation (both of these callbacks are run every time), fix image by debugging regex for l'etat -->

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR API KEY>",
    // single pattern for fr language, to avoid translating what follows "my name is"
    noTranslatePatterns: new Set("fr", "/mon nom est (.+)/"),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false,
    
});
adapter.use(languageTranslator);

```

---

![봇에서 패턴에 대한 번역 바이패스](./media/how-to-bot-translate/bot-no-translate-name-fr.png)

## <a name="localize-dates"></a>날짜 지역화

날짜를 지역화해야 하는 경우 `LocaleConverterMiddleware`를 추가할 수 있습니다. 예를 들어 봇에서 `MM/DD/YYYY` 형식의 날짜를 예상하는 것을 아는 경우 다른 로캘의 사용자가 `DD/MM/YYYY` 형식으로 날짜를 입력할 수 있으며, 로캘 변환기 미들웨어는 봇에서 예상하는 형식으로 날짜를 자동으로 변환할 수 있습니다.

> [!NOTE]
> 로캘 변환기 미들웨어는 날짜만 변환합니다. 번역 미들웨어의 결과를 알지 못합니다. 번역 미들웨어를 사용하는 경우 로캘 변환기와 결합하는 방법에 주의해야 합니다. 번역 미들웨어는 다른 텍스트 입력과 함께 텍스트 형식으로 된 일부 날짜를 번역하지만 날짜를 변환하지 않습니다.

예를 들어 다음 이미지는 영어에서 프랑스어로 번역한 후 사용자 입력을 다시 에코하는 봇을 보여줍니다. `LocaleConverterMiddleware`를 사용하지 않고 `TranslationMiddleware`를 사용합니다.

![날짜 변환 없이 날짜를 번역하는 봇](./media/how-to-bot-translate/locale-date-before.png)

다음은 `LocaleConverterMiddleware`가 추가되는 경우 동일한 봇을 보여 줍니다.

![날짜 변환 없이 날짜를 번역하는 봇](./media/how-to-bot-translate/locale-date-after.png)

로캘 변환기는 영어, 프랑스어, 독일어 및 중국어 로캘을 지원할 수 있습니다. <!-- TODO: ADD DETAIL ABOUT SUPPORTED LOCALES -->

# <a name="ctabcs"></a>[C#](#tab/cs)
Startup.cs
```cs
// Add locale converter middleware
middleware.Add(new LocaleConverterMiddleware(
    TranslatorLocaleHelper.GetActiveLocale,
    TranslatorLocaleHelper.CheckUserChangedLocale,
     "en-us", LocaleConverter.Converter));
```
TranslatorLocaleHelper.cs
```cs
// TranslatorLocaleHelper.cs
public static async Task<bool> CheckUserChangedLocale(ITurnContext context)
{
    bool changeLocale = false;//logic implemented by developper to make a signal for language changing 
    //use a specific message from user to change language
    if (context.Activity.Type == ActivityTypes.Message)
    {
        var messageActivity = context.Activity.AsMessageActivity();
        if (messageActivity.Text.ToLower().StartsWith("set my locale to"))
        {
            changeLocale = true;
        }
        if (changeLocale)
        {
            var newLocale = messageActivity.Text.ToLower().Replace("set my locale to", "").Trim(); //extracted by the user using user state 
            if (!string.IsNullOrWhiteSpace(newLocale)
                    && IsSupportedLanguage(newLocale))
            {
                SetLocale(context, newLocale);
                await context.SendActivity($@"Changing your language to {newLocale}");
            }
            else
            {
                await context.SendActivity($@"{newLocale} is not a supported locale.");
            }
            //intercepts message
            return true;
        }
    }

    return false;
}
public static string GetActiveLocale(ITurnContext context)
{
    if (currentLocale != null)
    {
        //the user has specified a different locale so update the bot state
        if (context.GetConversationState<CurrentUserState>() != null
            && currentLocale != context.GetConversationState<CurrentUserState>().Locale)
        {
            SetLocale(context, currentLocale);
        }
    }
    if (context.Activity.Type == ActivityTypes.Message
        && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Locale != null)
    {
        return context.GetConversationState<CurrentUserState>().Locale;
    }

    return "en-us";
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

<!-- this snippet only works if the user doesn't actually try to change their locale.  Emailed Mostafa about the issue 
It should change the locale after you type in 'set my locale to....' -->

```javascript
// Delegates for getting and setting user locale
function getUserLocale(context) {
    const state = conversationState.get(context)
    if (state.locale == undefined) {
        return 'en-us';
    } else {
        return state.locale;
    }
}

// Function to change the user's locale.
async function setUserLocale(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my locale to')) {        
        state.locale = context.activity.text.toLowerCase().replace('set my locale to', '').trim();
        await context.sendActivity(`Setting your locale to ${state.locale}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add locale converter middleware
// Will convert input to fr-fr
const localeConverter = new LocaleConverter({
    toLocale: 'fr-fr',
    setUserLocale: setUserLocale,
    getUserLocale: getUserLocale
});
adapter.use(localeConverter);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`The message that you sent was:  ${context.activity.text}`)
        }
    });
});

```

---
