---
title: .bot 파일을 사용하여 리소스 관리 | Microsoft Docs
description: 봇 파일의 목적 및 사용 방법을 설명합니다.
keywords: 봇 파일, .bot, .bot 파일, 봇 리소스, 봇 리소스 관리
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/30/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 14552c55da4b1f9b581b81917496de179e92762b
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/18/2019
ms.locfileid: "58811505"
---
# <a name="manage-bot-resources"></a>봇 리소스 관리

봇은 일반적으로 [LUIS.ai](https://luis.ai) 또는 [QnaMaker.ai](https://qnamaker.ai)와 같은 다른 서비스를 사용합니다. 봇을 개발하는 경우 모두 리소스를 추적할 수 있어야 합니다. appsettings.json, web.config 또는 .env와 같은 다양한 방법을 사용할 수 있습니다. 

> [!IMPORTANT]
> .bot 파일은 Bot Framework SDK 4.3 릴리스 이전에 리소스를 관리하기 위한 메커니즘으로 제공되었습니다. 그러나 앞으로 이러한 리소스를 관리하려면 appsettings.json 또는 .env 파일을 사용하는 것이 좋습니다. .bot 파일이 **_더 이상 사용되지 않더라도_** 현재 .bot 파일을 사용하는 봇은 계속 작동합니다. .bot 파일을 사용하여 리소스를 관리한 경우 적용되는 단계에 따라 설정을 마이그레이션합니다. 

## <a name="migrating-settings-from-bot-file"></a>.bot 파일에서 설정 마이그레이션
다음 섹션에서는 .bot 파일에서 설정을 마이그레이션하는 방법에 대해 설명합니다. 사용자에게 적용되는 시나리오를 따르세요.

**시나리오 1: .bot 파일이 있는 로컬 봇**

이 시나리오에서는 .bot 파일을 사용하는 로컬 봇이 있지만 _봇이 Azure Portal로 마이그레이션되지 않았습니다_. 아래 단계에 따라 설정을 .bot 파일에서 appsettings.json 또는 .env 파일로 마이그레이션합니다.

- .bot 파일이 암호화되어 있으면 다음 명령을 사용하여 암호를 해독해야 합니다.

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear` command.
```

- 암호 해독된 .bot 파일을 열고, 값을 복사하여 appsettings.json 또는 .env 파일에 추가합니다.
- appsettings.json 또는 .env 파일에서 설정을 읽도록 코드를 업데이트합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`ConfigureServices` 메서드에서 ASP.NET Core를 통해 제공되는 구성 개체를 사용합니다. 예를 들어 다음과 같습니다. 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

JavaScript에서 `process.env` 개체의 .env 변수를 참조합니다. 예를 들어 다음과 같습니다.
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

필요한 경우 appsettings.json 또는 .env 파일을 사용하여 리소스를 프로비저닝하고 봇에 연결합니다.

**시나리오 2: .bot 파일을 사용하여 Azure에 배포된 봇**

이 시나리오에서는 이미 .bot 파일을 사용하여 봇을 Azure Portal에 배포했으며, 이제 설정을 .bot 파일에서 appsettings.json 또는 .env 파일로 마이그레이션하려고 합니다.

- Azure Portal에서 봇 코드를 다운로드합니다. 코드가 다운로드되면 MicrosoftAppId, MicrosoftAppPassword 및 해당 추가 설정이 배치될 appsettings.json 또는 .env 파일을 포함하도록 요구하는 메시지가 표시됩니다. 
- _다운로드_한 appsettings.json 또는 .env 파일을 열고, 해당 파일의 설정을 _로컬_ appsettings.json 또는 .env 파일에 복사합니다. 로컬 appsettings.json 또는 .env 파일에서 botSecret 및 botFilePath 항목을 제거하는 것을 잊지 마세요.
- appsettings.json 또는 .env 파일에서 설정을 읽도록 코드를 업데이트합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
`ConfigureServices` 메서드에서 ASP.NET Core를 통해 제공되는 구성 개체를 사용합니다. 예를 들어 다음과 같습니다. 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
JavaScript에서 `process.env` 개체의 .env 변수를 참조합니다. 예를 들어 다음과 같습니다.
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

또한 **Azure Portal**의 **애플리케이션 설정** 섹션에서 `botFilePath` 및 `botFileSecret`을 제거해야 합니다.

_필요한 경우_ appsettings.json 또는 .env 파일을 사용하여 리소스를 프로비저닝하고 봇에 연결합니다.

**시나리오 3: appsettings.json 또는 .env 파일을 사용하는 봇**

이 시나리오에서는 SDK 4.3을 사용하여 봇을 처음부터 개발하기 시작하고 마이그레이션할 기존 .bot 파일이 없는 경우를 처리합니다. 봇에서 사용하려는 모든 설정은 아래와 같이 appsettings.json 또는 .env 파일에 있습니다.

```JSON
{
  "MicrosoftAppId": "<your-AppId>",
  "MicrosoftAppPassword": "<your-AppPwd>"
}
```

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

C# 코드에서 위의 설정을 읽으려면 ASP.NET Core를 통해 제공되는 구성 개체를 사용합니다. 예를 들어 다음과 같습니다. **Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
JavaScript에서 `process.env` 개체의 .env 변수를 참조합니다(예: **index.js**).
```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```

---

필요한 경우 appsettings.json 또는 .env 파일을 사용하여 리소스를 프로비저닝하고 봇에 연결합니다.


## <a name="faq"></a>FAQ
**Q:** 새 V4 봇을 Azure Portal에 만들려고 합니다. .bot 파일을 제거하면 Azure Portal 환경이 어떻게 변경되나요?

**A:** 봇을 Azure Portal에 만들면 .bot 파일이 만들어지지 않습니다. **Azure Portal**의 **애플리케이션 설정** 섹션을 사용하여 ID/키를 찾을 수 있습니다. 코드가 다운로드되면 이러한 설정이 appsettings.json 또는 .env 파일에 저장됩니다. 봇에서 개별 서비스를 호출하기 전에 설정을 읽을 수 있도록 코드를 업데이트합니다. 봇 코드가 업데이트되면 az bot publish를 사용하여 봇을 배포할 수 있습니다.

**Q:** V3 봇의 경우는 어떨까요?

**A:** V3 봇의 시나리오는 봇 파일이 없는 V4 봇의 시나리오와 비슷합니다. 배포 프로세스는 계속 작동합니다. 

## <a name="additional-resources"></a>추가 리소스
- 봇을 배포하는 단계는 [배포](../bot-builder-deploy-az-cli.md) 문서를 참조하세요.
- 키와 비밀을 보호하려면 Azure Key Vault를 사용하는 것이 좋습니다. Azure Key Vault는 봇의 엔드포인트 및 작성 키와 같은 비밀을 안전하게 저장하고 액세스할 수 있는 도구입니다. [키 관리](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-whatis) 솔루션을 제공하고 암호화 키를 쉽게 만들고 제어할 수 있으므로 비밀을 완벽하게 제어할 수 있습니다.


<!--

# Manage resources with a .bot file

Bots usually consume lots of different services, such as [LUIS.ai](https://luis.ai) or [QnaMaker.ai](https://qnamaker.ai). When you are developing a bot, there is no uniform place to store the metadata about the services that are in use.  This prevents us from building tooling that looks at a bot as a whole.

To address this problem, we have created a **.bot file** to act as the place to bring all service references together in one place to 
enable tooling.  For example, the Bot Framework Emulator ([V4](https://aka.ms/Emulator-wiki-getting-started)) uses a  .bot file to create a unified view over the connected services your bot consumes.  

With a .bot file, you can register services like:

* **Localhost** local debugger endpoints
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/) Azure Bot Service registrations.
* [**LUIS.AI**](https://www.luis.ai/) LUIS gives your bot the ability to communicate with people using natural language.. 
* [**QnA Maker**](https://qnamaker.ai/) Build, train and publish a simple question and answer bot based on FAQ URLs, structured documents or editorial content in minutes.
* [**Dispatch**](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) models for dispatching across multiple services.
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/) for insights and bot analytics.
* [**Azure Blob Storage**](https://azure.microsoft.com/en-us/services/storage/blobs/) for bot state persistence. 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) - globally distributed, multi-model database service to persist bot state.

Apart from these, your bot might rely on other custom services. You can leverage the [generic service](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md) capability to connect a generic service configuration.

## When is a .bot file created? 
- If you create a bot using [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D), a .bot file is automatically created for you with list of connected services provisioned. The .bot is encrypted by default.
- If you create a bot using Bot Framework V4 SDK [Template](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) for Visual Studio or using Bot Builder [Yeoman Generator](https://www.npmjs.com/package/generator-botbuilder), a .bot file is automatically created. No connected services are provisioned in this flow and the bot file is not encrypted.
- If you are starting with [BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples), every sample for Bot Framework V4 SDK includes a .bot file and the .bot file is not encrypted. 
- You can also create a bot file using the [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) tool.

## What does a bot file look like? 
Take a look at a sample [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) file.
To learn about encrypting and decrypting the .bot file, see [Bot Secrets](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md).

## Why do I need a .bot file?

A .bot file is **not** a requirement to build bots with Bot Framework SDK. You can continue to use appsettings.json, web.config, env, 
keyvault or any mechanism you see fit to keep track of service references and keys that your bot depends on. However, to test
the bot using the Emulator, you'll need a .bot file. The good news is that Emulator can create a .bot file for testing. To do that, 
start the Emulator, click on the **create a new bot configuration** link on the Welcome page. In the dialog box that appears, type a **Bot name** and an **Endpoint URL**. Then connect.

The advantages of using .bot file are:
- Provides a standard way of storing resources regardless of the language/platform you use.   
- Bot Framework Emulator and CLI tools rely on and work great with tracking connected services in a consistent format (in a .bot file) 
- Elegant tooling solutions around services creation and management is harder without a well defined schema (.bot file).  


## Using .bot file in your Bot Framework SDK bot

You can use the .bot file to get service configuration information in your bot's code. The BotFramework-Configuration library available 
for [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) and [JS](https://www.npmjs.com/package/botframework-config) helps you load a bot file and supports several methods to query and get the appropriate service configuration information.

## Additional resources
Refer to [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) readme file for more information on using a bot file.

-->

