---
title: 봇 리소스 관리 | Microsoft Docs
description: 봇 파일의 목적 및 사용 방법을 설명합니다.
keywords: 봇 파일, .bot, .bot 파일, msbot, 봇 리소스, 봇 리소스 관리
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a550de06be6584c5d9ba5eca4ffffffd49a518d2
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404298"
---
# <a name="manage-bot-resources"></a>봇 리소스 관리

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

봇은 일반적으로 [LUIS.ai](https://luis.ai) 또는 [QnaMaker.ai](https://qnamaker.ai)와 같은 다른 서비스를 사용합니다. 봇을 개발하는 경우 모두 리소스를 추적할 수 있어야 합니다. appsettings.json, web.config 또는 .env와 같은 다양한 방법을 사용할 수 있습니다. 

> [!IMPORTANT]
> .bot 파일은 Bot Framework SDK 4.3 릴리스 이전에 리소스를 관리하기 위한 메커니즘으로 제공되었습니다. 그러나 앞으로 이러한 리소스를 관리하려면 appsettings.json 또는 .env 파일을 사용하는 것이 좋습니다. .bot 파일이 **_더 이상 사용되지 않더라도_** 현재 .bot 파일을 사용하는 봇은 계속 작동합니다. .bot 파일을 사용하여 리소스를 관리한 경우 적용되는 단계에 따라 설정을 마이그레이션합니다. 

## <a name="migrating-settings-from-bot-file"></a>.bot 파일에서 설정 마이그레이션
다음 섹션에서는 .bot 파일에서 설정을 마이그레이션하는 방법에 대해 설명합니다. 사용자에게 적용되는 시나리오를 따르세요.

**시나리오 #1: .bot 파일이 있는 로컬 봇**

이 시나리오에서는 .bot 파일을 사용하는 로컬 봇이 있지만 _봇이 Azure Portal로 마이그레이션되지 않았습니다_. 아래 단계에 따라 설정을 .bot 파일에서 appsettings.json 또는 .env 파일로 마이그레이션합니다.

- .bot 파일이 암호화되어 있으면 다음 명령을 사용하여 암호를 해독해야 합니다.

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
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

*필요한 경우* appsettings.json 또는 .env 파일을 사용하여 리소스를 프로비저닝하고 봇에 연결합니다.

**시나리오 2: .bot 파일을 사용하여 Azure에 배포된 봇**

이 시나리오에서는 이미 .bot 파일을 사용하여 봇을 Azure Portal에 배포했으며, 이제 설정을 .bot 파일에서 appsettings.json 또는 .env 파일로 마이그레이션하려고 합니다.

- Azure Portal에서 봇 코드를 다운로드합니다. 코드가 다운로드되면 MicrosoftAppId, MicrosoftAppPassword 및 해당 추가 설정이 배치될 appsettings.json 또는 .env 파일을 포함하도록 요구하는 메시지가 표시됩니다. 
- _다운로드_ 한 appsettings.json 또는 .env 파일을 열고, 해당 파일의 설정을 _로컬_ appsettings.json 또는 .env 파일에 복사합니다. 로컬 appsettings.json 또는 .env 파일에서 botSecret 및 botFilePath 항목을 제거하는 것을 잊지 마세요.
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

*필요한 경우* appsettings.json 또는 .env 파일을 사용하여 리소스를 프로비저닝하고 봇에 연결합니다.

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

## <a name="additional-resources"></a>추가 리소스
- 봇을 배포하는 단계는 [배포](../bot-builder-deploy-az-cli.md) 문서를 참조하세요.
- [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview)를 사용하여 클라우드 애플리케이션 및 서비스에서 사용하는 암호화 키 및 비밀을 보호하고 관리하는 방법에 대해 알아봅니다.
