---
title: 봇 배포 | Microsoft Docs
description: Azure 클라우드에 봇을 배포합니다.
keywords: 봇 배포, Azure 봇 배포, 봇 게시
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 04/12/2019
ms.openlocfilehash: 4532fe55705524573de55017e633289255a20ab9
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/18/2019
ms.locfileid: "59508220"
---
# <a name="deploy-your-bot"></a>봇 배포

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

봇을 만들어 로컬로 테스트한 후에는 어디서나 액세스할 수 있도록 Azure에 배포할 수 있습니다. 봇이 Azure에 배포되면 사용하는 서비스에 대한 비용을 지불해야 합니다. [청구 및 비용 관리](https://docs.microsoft.com/en-us/azure/billing/) 문서는 Azure 청구를 이해하고, 사용량과 비용을 모니터링하며, 계정과 구독을 관리하는 데 도움이 됩니다.

이 문서에서는 C# 및 JavaScript 봇을 Azure에 배포하는 방법을 보여 줍니다. 단계를 수행하기 전에 이 문서를 참조하면 봇 배포와 관련된 내용을 완전히 이해할 수 있습니다.

## <a name="prerequisites"></a>필수 조건
- [Azure 구독](http://portal.azure.com)이 없는 경우 시작하기 전에 체험 계정을 만듭니다.
- 로컬 머신에서 개발한 [**CSharp**](./dotnet/bot-builder-dotnet-sdk-quickstart.md) 또는 [**JavaScript**](./javascript/bot-builder-javascript-quickstart.md) 봇이 있어야 합니다.

## <a name="1-prepare-for-deployment"></a>1. 배포 준비
배포 프로세스를 수행하려면 Azure에 대상 웹앱 봇이 있어야 로컬 봇이 배포될 수 있습니다. Azure에서 대상 웹앱 봇과 이 봇에 프로비저닝되는 리소스는 로컬 봇에서 배포용으로 사용합니다. 로컬 봇에는 필요한 Azure 리소스 중 일부가 프로비저닝되어 있지 않기 때문에 이 항목이 필요합니다. 대상 웹앱 봇을 만들면 다음 리소스가 자동으로 프로비저닝됩니다.
-   웹앱 봇 – 로컬 봇을 배포할 때 사용할 봇입니다.
-   App Service 플랜 - App Service 앱이 실행해야 하는 리소스를 제공합니다.
-   App Service - 웹 애플리케이션을 호스팅하기 위한 서비스
-   스토리지 계정 - Blob, 파일, 큐, 테이블, 디스크 등, 모든 Azure Storage 데이터 개체가 포함됩니다.

대상 웹앱 봇을 만드는 동안 봇의 앱 ID 및 암호도 생성됩니다. Azure에서 앱 ID 및 암호는 [서비스 인증 및 권한 부여](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization)를 지원합니다. 이 정보 중 일부를 검색하여 로컬 봇 코드에서 사용하게 됩니다. 

> [!IMPORTANT]
> Azure Portal에서 사용되는 봇 템플릿에 대한 프로그래밍 언어는 봇을 작성할 때 사용한 프로그래밍 언어와 일치해야 합니다.

이미 Azure에서 사용하려는 봇을 만든 경우 새 웹앱 봇을 만드는 것은 선택 사항입니다.

1. [Azure Portal](https://portal.azure.com)에 로그인합니다.
1. Azure Portal의 왼쪽 위 모서리에 있는 **새 리소스 만들기** 링크를 클릭하고 **AI + Machine Learning > 웹앱 봇**을 선택합니다.
1. 웹앱 봇에 대한 정보가 포함된 새 블레이드가 열립니다. 
1. **Bot Service** 블레이드에서 봇에 대해 요청되는 정보를 제공합니다.
1. **만들기**를 클릭하여 서비스를 만들고 봇을 클라우드에 배포합니다. 이 프로세스에는 몇 분 정도 걸릴 수 있습니다.

### <a name="download-the-source-code"></a>소스 코드 다운로드
대상 웹앱 봇을 만든 후에는 Azure Portal에서 로컬 머신으로 봇 코드를 다운로드해야 합니다. 이 코드는 appsettings.json 또는 .env 파일에 있는 서비스 참조(예: MicrosoftAppID, MicrosoftAppPassword, LUIS 또는 QnA)를 가져오기 위해 다운로드하는 것입니다. 

1. **봇 관리** 섹션에서 **빌드**를 클릭합니다.
1. 오른쪽 창에서 **Bot 소스 코드 다운로드** 링크를 클릭합니다.
1. 표시되는 메시지에 따라 코드를 다운로드한 다음, 폴더의 압축을 풉니다.
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="update-your-local-appsettingsjson-or-env-file"></a>로컬 appsettings.json 또는 .env 파일 업데이트

다운로드한 appsettings.json 또는 .env 파일을 엽니다. 나열되는 **모든** 항목을 복사하여 _로컬_ appsettings.json 또는 .env 파일에 추가합니다. 모든 중복 서비스 항목을 확인하거나 서비스 ID를 복제합니다. 봇이 종속된 추가 서비스 참조를 유지합니다.

파일을 저장합니다.

### <a name="update-local-bot-code"></a>로컬 봇 코드 업데이트
.bot 파일을 사용하는 대신 appsettings.json 또는 .env 파일을 사용하도록 로컬 Startup.cs 또는 index.js 파일을 업데이트합니다. .bot 파일은 더 이상 사용되지 않으며, .bot 파일 대신 appsettings.json 또는 .env 파일을 모두 사용하도록 VSIX 템플릿, Yeoman 생성기, 샘플 및 나머지 문서를 업데이트합니다. 그동안 봇 코드를 변경해야 합니다. 

appsettings.json 또는 .env 파일에서 설정을 읽도록 코드를 업데이트합니다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
`ConfigureServices` 메서드에서 ASP.NET Core를 통해 제공되는 구성 개체를 사용합니다. 예를 들어 다음과 같습니다. 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="jstabjs"></a>[JS](#tab/js)

JavaScript에서 `process.env` 개체의 .env 변수를 참조합니다. 예를 들어 다음과 같습니다.
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

- 파일을 저장하고 봇을 테스트합니다.

### <a name="setup-a-repository"></a>리포지토리 설정

지속적인 배포를 지원하려면 즐겨찾는 Git 원본 제어 공급자를 사용하여 Git 리포지토리를 만듭니다. 코드를 리포지토리에 커밋합니다.

[리포지토리 준비](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment#prepare-your-repository)에 설명된 대로 리포지토리 루트에 프로젝트의 올바른 파일에 있는지 확인합니다.

### <a name="update-app-settings-in-azure"></a>Azure에서 앱 설정 업데이트
로컬 봇은 암호화된 .bot 파일을 사용하지 않지만 Azure Portal이 암호화된 .bot 파일을 사용하도록 구성된 _경우_에는 그렇지 않습니다. 이 문제는 Azure 봇 설정에 저장된 **botFileSecret**를 제거하여 해결할 수 있습니다.
1. Azure Portal에서 봇용 **웹앱 봇** 리소스를 엽니다.
1. 봇의 **애플리케이션 설정**을 엽니다.
1. **애플리케이션 설정** 창에서 **애플리케이션 설정**까지 아래로 스크롤합니다.
1. 봇에 **botFileSecret** 및 **botFilePath** 항목이 있는지 확인합니다. 있는 경우 해당 항목을 삭제합니다.
1. 변경 내용을 저장합니다.

## <a name="2-deploy-using-azure-deployment-center"></a>2. Azure 배포 센터를 사용하여 배포

이제 Azure에 봇 코드를 업로드해야 합니다. [Azure App Service에 지속적인 배포](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment) 항목의 지침을 따릅니다.

`App Service Kudu build server`를 사용하여 빌드하는 것이 좋습니다.

지속적인 배포를 구성하면 리포지토리에 커밋한 변경 사항이 게시됩니다. 그러나 봇에 서비스를 추가하는 경우에는 이와 관련된 항목을 .bot 파일에 추가해야 합니다.

## <a name="3-test-your-deployment"></a>3. 배포 테스트

배포가 성공하면 몇 초 동안 기다린 후 필요에 따라 웹앱을 다시 시작하여 캐시를 지웁니다. 웹앱 봇 블레이드로 돌아가서 Azure Portal에 제공된 웹 채팅을 사용하여 테스트합니다.

## <a name="additional-resources"></a>추가 리소스
- [연속 배포의 일반 문제를 조사하는 방법](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)

