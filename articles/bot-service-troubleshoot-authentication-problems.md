---
title: Bot Framework 인증 문제 해결 | Microsoft Docs
description: 봇 관련 인증 오류를 해결하는 방법을 알아봅니다.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/17
ms.openlocfilehash: 0fdd196716c0fffb36583c0df894481b032dd83e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999412"
---
# <a name="troubleshooting-bot-framework-authentication"></a>Bot Framework 인증 문제 해결

이 가이드를 통해 일련의 시나리오를 평가하여 문제의 위치를 확인하는 방식으로 봇 관련 인증 문제를 해결할 수 있습니다. 

> [!NOTE]
> 이 가이드의 모든 단계를 완료하려면 [Bot Framework Emulator][Emulator]를 다운로드하여 사용해야 하고 <a href="https://dev.botframework.com" target="_blank">Bot Framework 포털</a>에서 봇의 등록 설정에 액세스할 수 있어야 합니다.

## <a id="PW"></a> 앱 ID 및 암호

봇 보안은 Bot Framework에 봇을 등록할 때 받은 **Microsoft 앱 ID** 및 **Microsoft 앱 암호**를 통해 구성됩니다. 일반적으로 이러한 값은 봇의 구성 파일 내에서 지정되고 Microsoft 계정 서비스에서 액세스 토큰을 검색하는 데 사용됩니다. 

아직 봇을 배포하지 않은 경우 [봇을 azure에 배포](~/bot-builder-howto-deploy-azure.md)하여 인증에 사용할 수 있도록 **Microsoft 앱 ID** 및 **Microsoft 앱 암호**를 가져옵니다. 

> [!NOTE]
> 이미 배포된 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

## <a name="step-1-disable-security-and-test-on-localhost"></a>1단계: localhost에서 보안 및 테스트 사용 안 함

이 단계에서는 보안이 사용되지 않을 경우 봇이 localhost에서 액세스 가능하고 작동하는지 확인합니다. 

> [!WARNING]
> 봇에 대한 보안을 사용하지 않으면 알 수 없는 공격자가 사용자를 가장할 수 있습니다. 보호된 디버깅 환경에서 운영 중인 경우에만 다음 절차를 구현합니다.

### <a id="disable-security-localhost"></a> 보안 사용 안 함

봇에 대한 보안을 사용하지 않으려면 해당 구성 설정을 편집하여 앱 ID 및 암호의 값을 제거합니다. 

::: moniker range="azure-bot-service-3.0"

.NET용 Bot Builder SDK를 사용 중인 경우 Web.config 파일에서 다음 설정을 편집합니다. 

```xml
<appSettings>
  <add key="MicrosoftAppId" value="" />
  <add key="MicrosoftAppPassword" value="" />
</appSettings>
```

Node.js용 Bot Builder SDK를 사용 중인 경우 다음 값을 편집하거나 해당 환경 변수를 업데이트합니다.

```javascript
var connector = new builder.ChatConnector({
  appId: null,
  appPassword: null
});
```

::: moniker-end

::: moniker range="azure-bot-service-4.0"

.NET용 Bot Builder SDK를 사용 중인 경우 `appsettings.config` 파일에서 다음 설정을 편집합니다.

```xml
<appSettings>
  <add key="MicrosoftAppId" value="" />
  <add key="MicrosoftAppPassword" value="" />
</appSettings>
```

Node.js용 Bot Builder SDK를 사용 중인 경우 다음 값을 편집하거나 해당 환경 변수를 업데이트합니다.

```javascript
const adapter = new BotFrameworkAdapter({
    appId: null,
    appPassword: null
});
```

구성을 위해 `.bot` 파일을 사용하는 경우 `appId` 및 `appPassword`를 `""`로 업데이트할 수 있습니다.

::: moniker-end

### <a name="test-your-bot-on-localhost"></a>localhost에서 봇 테스트 

그런 다음, Bot Framework Emulator를 사용하여 localhost에서 봇을 테스트합니다.

1. localhost에서 봇을 시작합니다.
2. Bot Framework Emulator를 시작합니다.
3. 에뮬레이터를 사용하여 봇에 연결합니다.
    - 에뮬레이터의 주소 표시줄에 `http://localhost:port-number/api/messages`를 입력합니다. 여기서 **port-number**는 애플리케이션이 실행 중인 브라우저에 표시되는 포트 번호와 일치합니다. 
    - **Microsoft 앱 ID** 및 **Microsoft 앱 암호** 필드가 둘 다 비어 있는지 확인합니다.
    - **Connect**를 클릭합니다.
4. 봇에 대한 연결을 테스트하려면 에뮬레이터에 일부 텍스트를 입력하고 Enter 키를 누릅니다.

봇이 입력에 응답하고 채팅 창에 오류가 없으면 보안이 사용되지 않을 때 봇이 localhost에서 액세스 가능하고 작동함을 확인했습니다. [2단계](#step-2)로 진행합니다.

채팅 창에서 하나 이상의 오류가 표시되면 오류를 클릭하여 자세한 내용을 확인합니다. 일반적인 문제는 다음과 같습니다.

* 에뮬레이터 설정은 봇의 잘못된 엔드포인트를 지정합니다. URL에 적절한 포트 번호를 포함하고 URL 끝에 적절한 경로를 포함했는지 확인합니다(예: `/api/messages`).
* 에뮬레이터 설정은 `https`로 시작하는 봇 엔드포인트를 지정합니다. Localhost에서 엔드포인트는 `http`로 시작해야 합니다.
* 에뮬레이터 설정은 **Microsoft 앱 ID** 필드 및/또는 **Microsoft 앱 암호** 필드의 값을 지정합니다. 두 필드가 모두 비어 있어야 합니다.
* 봇의 보안이 사용하지 않도록 설정되지 않았습니다. 봇이 앱 ID 또는 암호의 값을 지정하지 않았는지 [확인](#disable-security-localhost)합니다.

## <a id="step-2"></a> 2단계: 봇의 앱 ID 및 암호 확인

이 단계에서는 봇이 인증에 사용할 앱 ID 및 암호가 유효한지 확인합니다. (이러한 값을 모르는 경우, 지금 [값을 가져옵니다](#PW).) 

> [!WARNING]
> 다음 지침은 `login.microsoftonline.com`에 대한 SSL 확인을 사용하지 않도록 설정합니다. 이 절차는 보안 네트워크에서만 수행하고 이후 응용 프로그램의 암호를 변경하는 것이 좋습니다.

### <a name="issue-an-http-request-to-the-microsoft-login-service"></a>Microsoft 로그인 서비스에 대한 HTTP 요청 실행

이러한 지침은 [cURL](https://curl.haxx.se/download.html)을 사용하여 Microsoft 로그인 서비스에 대한 HTTP 요청을 실행하는 방법을 설명합니다. Postman과 같은 대체 도구를 사용할 수 있습니다. 요청이 Bot Framework [인증 프로토콜](~/rest-api/bot-framework-rest-connector-authentication.md)을 따르는지만 확인하세요.

봇의 앱 ID 및 암호가 유효한지 확인하려면 `APP_ID` 및 `APP_PASSWORD`를 봇의 앱 ID 및 암호로 바꾸고 **cURL**을 사용하여 다음 요청을 실행합니다.

```cmd
curl -k -X POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token -d "grant_type=client_credentials&client_id=APP_ID&client_secret=APP_PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default"
```

이 요청은 액세스 토큰에 대한 봇의 앱 ID 및 암호를 교환하려고 합니다. 요청에 성공하면 무엇보다 `access_token` 속성이 포함된 JSON 페이로드를 받게 됩니다. 

```json
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":0,"access_token":"eyJ0eXAJKV1Q..."}
```

요청에 성공하면 요청에서 지정한 앱 ID 및 암호가 유효한 것을 확인했습니다. [3단계](#step-3)로 진행합니다.

요청에 대한 응답으로 오류가 표시되면 응답을 검사하여 오류의 원인을 확인합니다. 응답이 앱 ID 또는 암호가 잘못되었다고 나타내면 Bot Framework 포털에서 [올바른 값을 가져오고](#PW) 새 값으로 요청을 실행하여 값이 유효한지 확인합니다. 

## 3단계: localhost에서 보안 및 테스트 사용<a id="step-3"></a>

현재 보안이 사용되지 않을 때 봇이 localhost에서 액세스 가능하고 작동함을 확인했고 봇이 인증에 사용할 앱 ID 및 암호가 유효한 것을 확인했습니다. 이 단계에서는 보안이 사용될 경우 봇이 localhost에서 액세스 가능하고 작동하는지 확인합니다.

### <a id="enable-security-localhost"></a> 보안 사용

봇이 localhost에서만 실행되는 경우에도 봇의 보안은 Microsoft 서비스를 사용합니다. 봇에 대한 보안을 사용하려면 해당 구성 설정을 편집하여 [2단계](#step-2)에서 확인한 값으로 앱 ID 및 암호를 채웁니다.

.NET용 Bot Builder SDK를 사용 중인 경우 `.bot` 또는 `appsettings.config` 파일에서 다음 설정을 채웁니다.

```xml
<appSettings>
  <add key="MicrosoftAppId" value="APP_ID" />
  <add key="MicrosoftAppPassword" value="PASSWORD" />
</appSettings>
```

Node.js용 Bot Builder SDK를 사용 중인 경우 다음 설정을 채우거나 해당 환경 변수를 업데이트합니다.

```javascript
var connector = new builder.ChatConnector({
  appId: 'APP_ID',
  appPassword: 'PASSWORD'
});
```

> [!NOTE]
> 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

### <a name="test-your-bot-on-localhost"></a>localhost에서 봇 테스트 

그런 다음, Bot Framework Emulator를 사용하여 localhost에서 봇을 테스트합니다.

1. localhost에서 봇을 시작합니다.
2. Bot Framework Emulator를 시작합니다.
3. 에뮬레이터를 사용하여 봇에 연결합니다.
    - 에뮬레이터의 주소 표시줄에 `http://localhost:port-number/api/messages`를 입력합니다. 여기서 **port-number**는 애플리케이션이 실행 중인 브라우저에 표시되는 포트 번호와 일치합니다. 
    - 봇의 앱 ID를 **Microsoft 앱 ID** 필드에 입력합니다.
    - 봇의 암호를 **Microsoft 앱 암호** 필드에 입력합니다.
    - **Connect**를 클릭합니다.
4. 봇에 대한 연결을 테스트하려면 에뮬레이터에 일부 텍스트를 입력하고 Enter 키를 누릅니다.

봇이 입력에 응답하고 채팅 창에 오류가 없으면 보안이 사용될 때 봇이 localhost에서 액세스 가능하고 작동함을 확인했습니다.  [4단계](#step-4)로 진행합니다.

채팅 창에서 하나 이상의 오류가 표시되면 오류를 클릭하여 자세한 내용을 확인합니다. 일반적인 문제는 다음과 같습니다.

* 에뮬레이터 설정은 봇의 잘못된 엔드포인트를 지정합니다. URL에 적절한 포트 번호를 포함하고 URL 끝에 적절한 경로를 포함했는지 확인합니다(예: `/api/messages`).
* 에뮬레이터 설정은 `https`로 시작하는 봇 엔드포인트를 지정합니다. Localhost에서 엔드포인트는 `http`로 시작해야 합니다.
* 에뮬레이터 설정에서 **Microsoft 앱 ID** 필드 및/또는 **Microsoft 앱 암호**에 유효한 값이 없습니다. 두 필드를 모두 입력해야 하고 각 필드에는 [2단계](#step-2)에서 확인한 해당 값을 포함해야 합니다.
* 봇의 보안이 사용하도록 설정되지 않았습니다. 봇 구성 설정이 앱 ID 및 암호의 값을 둘 다 지정하는지 [확인](#enable-security-localhost)합니다.

## 4단계: 클라우드에서 봇 테스트 <a id="step-4"></a>

현재 보안이 사용되지 않을 때 봇이 localhost에서 액세스 가능하고 작동함을 확인했고, 봇의 앱 ID 및 암호가 유효한 것을 확인했고, 보안이 사용될 때 봇이 localhost에서 액세스 가능하고 작동함을 확인했습니다. 이 단계에서는 봇을 클라우드에 배포하고 보안이 사용될 경우 봇이 클라우드에서 액세스 가능하고 작동하는지 확인합니다. 

### <a name="deploy-your-bot-to-the-cloud"></a>클라우드에 봇 배포

Bot Framework에서는 봇이 인터넷에서 액세스 가능해야 하므로, 봇을 Azure와 같은 클라우드 호스팅 플랫폼에 배포해야 합니다. [3단계](#step-3)의 설명대로 배포하기 전에 봇에 대한 보안을 사용하도록 설정해야 합니다.

> [!NOTE]
> 아직 클라우드 호스팅 공급자가 없는 경우, <a href="https://azure.microsoft.com/en-us/free/" target="_blank">체험 계정</a>으로 등록할 수 있습니다. 

Azure에 봇을 배포하면 애플리케이션에 대한 SSL이 자동으로 구성되므로 Bot Framework에서 요구하는 **HTTPS** 엔드포인트를 사용할 수 있습니다. 다른 클라우드 호스팅 공급자에 배포할 경우에는 봇에 **HTTPS** 엔드포인트가 포함되도록 애플리케이션에 대한 SSL이 구성되어 있는지 확인해야 합니다.

### <a name="test-your-bot"></a>봇 테스트 

보안을 사용하도록 설정하고 클라우드에서 봇을 테스트하려면 다음 단계를 수행합니다.

1. 봇이 성공적으로 배포되었고 실행 중인지 확인합니다. 
2. <a href="https://dev.botframework.com" target="_blank">Bot Framework 포털</a>에 로그인합니다.
3. **My Bots**(내 봇)를 클릭합니다.
4. 테스트할 봇을 선택합니다.
5. **테스트**를 클릭하여 포함된 웹 채팅 컨트롤에서 봇을 엽니다.
6. 봇에 대한 연결을 테스트하려면 웹 채팅 컨트롤에 일부 텍스트를 입력하고 Enter 키를 누릅니다.

채팅 창에 오류가 표시되면 오류 메시지를 사용하여 오류의 원인을 확인합니다. 일반적인 문제는 다음과 같습니다. 

* Bot Framework 포털에서 봇에 대한 **설정** 페이지에 지정된 **메시징 엔드포인트**가 올바르지 않습니다. URL 끝에 적절한 경로를 포함했는지 확인합니다(예: `/api/messages`).
* Bot Framework 포털에서 봇에 대한 **설정** 페이지에 지정된 **메시징 엔드포인트**가 `https`로 시작되지 않거나 Bot Framework에서 신뢰되지 않습니다. 봇에는 유효한 체인에서 신뢰할 수 있는 인증서가 있어야 합니다.
* 앱 ID 또는 암호의 값이 잘못되거나 누락된 상태로 봇이 구성되어 있습니다. 봇 구성 설정이 앱 ID 및 암호의 유효한 값을 지정하는지 [확인](#enable-security-localhost)합니다.

봇이 입력에 적절하게 응답하는 경우 보안을 사용하도록 설정하고 봇이 클라우드에서 액세스 가능하고 작동함을 확인했습니다. 이때 봇은 Skype, Facebook Messenger, Direct Line 등의 [채널에 안전하게 연결](~/bot-service-manage-channels.md)할 준비가 완료되었습니다.

## <a name="additional-resources"></a>추가 리소스

위의 단계를 완료한 후 여전히 문제가 발생하는 경우, 다음을 수행할 수 있습니다.

* Bot Framework Emulator 및 <a href="https://ngrok.com/" target="_blank">ngrok</a>를 사용하여 [클라우드에서 봇을 디버그](~/bot-service-debug-emulator.md)합니다.
* [Fiddler](https://www.telerik.com/fiddler) 같은 프록시 도구를 사용하여 봇에서 보내고 받는 HTTPS 트래픽을 검사합니다. *Fiddler는 Microsoft 제품이 아닙니다.*
* Bot Framework가 사용하는 인증 기술을 알아보려면 [Bot Connector 인증 가이드][BotConnectorAuthGuide]를 검토하세요.
* Bot Framework [지원][Support] 리소스를 사용하여 다른 사용자에게 도움을 요청합니다. 

[BotConnectorAuthGuide]: ~/rest-api/bot-framework-rest-connector-authentication.md
[Support]: bot-service-resources-links-help.md
[Emulator]: bot-service-debug-emulator.md
