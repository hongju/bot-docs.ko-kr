---
title: Azure CLI를 사용하여 봇 배포 | Microsoft Docs
description: Azure 클라우드에 봇을 배포합니다.
keywords: 봇 배포, Azure 배포, 봇 게시, az deploy bot, Visual Studio 배포 봇, msbot publish, msbot clone
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/11/2018
ms.openlocfilehash: a7cb9cb1e3df14f2f46bc5a4c3a633f5212b5dfd
ms.sourcegitcommit: 0b421ff71617f03faf55ea175fb91d1f9e348523
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/12/2018
ms.locfileid: "53286619"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Azure CLI를 사용하여 봇 배포

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

봇을 만들어 로컬로 테스트한 후에는 어디서나 액세스할 수 있도록 Azure에 배포할 수 있습니다. 봇이 Azure에 배포되면 사용하는 서비스에 대한 비용을 지불해야 합니다. [청구 및 비용 관리](https://docs.microsoft.com/en-us/azure/billing/) 문서는 Azure 청구를 이해하고, 사용량과 비용을 모니터링하며, 계정과 구독을 관리하는 데 도움이 됩니다.

이 문서에서는 `msbot` 도구를 사용하여 C# 및 JavaScript 봇을 Azure에 배포하는 방법을 보여 줍니다. 단계를 수행하기 전에 이 문서를 참조하면 봇 배포와 관련된 내용을 완전히 이해할 수 있습니다.


## <a name="prerequisites"></a>필수 조건
- Azure 구독이 아직 없는 경우 시작하기 전에 [체험 계정](https://azure.microsoft.com/free/)을 만듭니다.
- [.NET Core SDK](https://dotnet.microsoft.com/download) v2.2 이상을 설치합니다. 
- 최신 버전의 [Azure CLI 도구](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)를 설치합니다.
- `az` 도구에 대한 최신 `botservice` 확장을 설치합니다. 
  - 먼저 `az extension remove -n botservice` 명령을 사용하여 이전 버전을 제거합니다. 다음으로 `az extension add -n botservice` 명령을 사용하여 최신 버전을 설치합니다.
- 최신 버전의 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 도구를 설치합니다.
  - 복제 작업에 LUIS 또는 디스패치 리소스가 포함되는 경우 [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation)가 필요합니다.
  - 복제 작업에 QnA Maker 리소스가 포함되는 경우 [QnA Maker CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli)가 필요합니다.
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)를 설치합니다.
- [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)를 설치 및 구성합니다.
- [.bot](v4sdk/bot-file-basics.md) 파일에 대한 지식이 필요합니다.

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>az cli를 사용하여 JavaScript 및 C# 봇 배포
이미 봇을 만들었고, 이제는 이 봇을 Azure에 배포하려고 합니다. 이러한 단계에서는 필요한 Azure 리소스를 만들고, `msbot connect` 명령 또는 Bot Framework Emulator를 사용하여 .bot 파일의 서비스 참조를 업데이트했다고 가정합니다. .bot 파일이 최신 파일이 아닌 경우 배포 프로세스가 오류 또는 경고 없이 완료될 수 있지만 배포된 봇은 작동하지 않습니다.

명령 프롬프트를 열어 Azure Portal에 로그인합니다.

```cmd
az login
```
로그인할 수 있는 브라우저 창이 열립니다. 

### <a name="set-the-subscription"></a>구독 설정 
다음 명령을 사용하여 구독을 설정합니다.

```cmd
az account set --subscription "<azure-subscription>"
``` 

봇을 배포하는 데 사용할 구독이 확실하지 않은 경우 `az account list` 명령을 사용하여 계정에 대한 `subscriptions`의 목록을 볼 수 있습니다.

봇 폴더로 이동합니다. 
```cmd 
cd <local-bot-folder>
```

### <a name="azure-subscription-account"></a>Azure 구독 계정
먼저 Azure에 로그인하는 데 사용하는 이메일 계정 유형에 따라 적용되는 지침을 참조한 후에 계속 진행합니다.

**MSA 이메일 계정**

[MSA](https://en.wikipedia.org/wiki/Microsoft_account) 이메일 계정을 사용하는 경우 `msbot clone services` 명령에 사용할 appId 및 appSecret을 만들어야 합니다. 

- [애플리케이션 등록 포털](https://apps.dev.microsoft.com/)로 이동합니다. **앱 추가**를 클릭하여 애플리케이션을 등록하고, **애플리케이션 ID**를 만들고, **새 암호 생성**을 수행합니다. 
- `msbot clone services` 명령에 사용할 수 있도록 방금 생성한 애플리케이션 ID와 새 암호를 모두 저장합니다. 
- 배포하려면 봇에 적용되는 명령을 사용합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

**회사 또는 학교 계정**

회사 또는 학교에서 제공한 이메일 계정을 사용하여 Azure에 로그인하는 경우 애플리케이션 ID와 암호를 만들 필요가 없습니다. 배포하려면 봇에 적용되는 명령을 사용합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>"`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>"`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="save-the-secret-used-to-decrypt-bot-file"></a>.bot 파일의 암호를 해독하는 데 사용되는 비밀 저장
배포 프로세스에서 _새 .bot 파일을 만들고 비밀을 사용하여 암호화_한다는 점에 유의해야 합니다. 봇이 배포되는 동안 명령줄에서 .bot 파일 비밀을 저장하라는 다음 메시지가 표시됩니다. 

`The secret used to decrypt myAzBot.bot is:`
`hT6U1SIPQeXlebNgmhHYxcdseXWBZlmIc815PpK0WWA=`

`NOTE: This secret is not recoverable and you should save it in a safe place according to best security practices.
      Copy this secret and use it to open the <file.bot> the first time.`
      
나중에 사용할 수 있도록 .bot 파일 비밀을 저장합니다. 암호화된 새 .bot 파일은 Azure Portal에서 botFileSecret과 함께 사용됩니다. 나중에 봇 파일 이름 또는 비밀을 변경해야 하는 경우 포털의 **App Service 설정 -> 애플리케이션 설정** 섹션으로 차례로 이동합니다. appsetings.json 또는 .env 파일에서 봇 파일 이름이 만든 최신 봇 파일로 업데이트됩니다.  

### <a name="test-your-bot"></a>봇 테스트
에뮬레이터에서 프로덕션 엔드포인트를 사용하여 앱을 테스트합니다. 로컬로 테스트하려면 봇이 로컬 머신에서 실행되고 있는지 확인합니다. 

### <a name="to-update-your-bot-code-in-azure"></a>Azure에서 봇 코드를 업데이트하려면
`msbot clone services` 명령은 Azure에서 봇 코드를 업데이트하는 데 사용하지 마세요. 아래와 같이 `az bot publish` 명령을 사용해야 합니다.

```cmd
az bot publish --name "<your-azure-bot-name>" --proj-file "<your-proj-file>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| 인수        | 설명 |
|----------------  |-------------|
| `name`      | 봇이 Azure에 처음 배포될 때 사용한 이름입니다.|
| `proj-file` | C# 봇의 경우 .csproj 파일입니다. JS/TS 봇의 경우 로컬 봇의 시작 프로젝트 파일 이름(예: index.js 또는 index.ts)입니다.|
| `resource-group` | `msbot clone services` 명령이 사용한 Azure 리소스 그룹입니다.|
| `code-dir`  | 로컬 봇 폴더를 가리킵니다.|



## <a name="additional-resources"></a>추가 리소스

봇이 배포되면 일반적으로 Azure Portal에 만들어지는 리소스는 다음과 같습니다.

| 리소스      | 설명 |
|----------------|-------------|
| Web App 봇 | Azure App Service에 배포되는 Azure Bot Service 봇입니다.|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| 웹 애플리케이션을 구축하고 호스팅할 수 있습니다.|
| [App Service 계획](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| 실행할 웹앱에 대한 컴퓨팅 리소스 세트를 정의합니다.|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| 원격 분석을 수집하고 분석하는 도구를 제공합니다.|
| [Storage 계정](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| 고가용성, 보안, 내구성, 확장성 및 중복성이 뛰어난 클라우드 스토리지를 제공합니다.|

`az bot` 명령에 대한 설명서를 보려면 [참조](https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest) 항목을 참조하세요.

Azure 리소스 그룹에 익숙하지 않은 경우 이 [용어](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology) 항목을 참조하세요.

## <a name="next-steps"></a>다음 단계
> [!div class="nextstepaction"]
> [지속적인 배포 설정](bot-service-build-continuous-deployment.md)
