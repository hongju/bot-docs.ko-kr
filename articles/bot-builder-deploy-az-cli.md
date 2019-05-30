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
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c17e830c61036a6551fa7f3dbab79f83bda38123
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/24/2019
ms.locfileid: "66214312"
---
# <a name="deploy-your-bot"></a>봇 배포

[!INCLUDE [applies-to](./includes/applies-to.md)]

이 문서에서는 Azure에 봇을 배포하는 방법을 보여줍니다. 단계를 수행하기 전에 이 문서를 참조하면 봇 배포와 관련된 내용을 완전히 이해할 수 있습니다.

## <a name="prerequisites"></a>필수 조건
- Azure 구독이 없는 경우 시작하기 전에 [계정](https://azure.microsoft.com/free/)을 만드세요.
- 로컬 머신에서 개발한 CSharp, JavaScript 또는 TypeScript 봇이 있어야 합니다.
- [Azure cli](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) 최신 버전

## <a name="1-prepare-for-deployment"></a>1. 배포 준비
Visual Studio 또는 Yeoman 템플릿을 사용하여 봇을 만들 때 생성된 소스 코드에는 ARM 템플릿이 있는 `deploymentTemplates` 폴더가 있습니다. 여기서 설명하는 배포 프로세스에서는 ARM 템플릿을 사용하여 Azure CLI를 통해 Azure의 봇에 필요한 리소스를 프로비전합니다. 

> [!IMPORTANT]
> Bot Framework SDK 4.3 릴리스부터는 appsettings.json 또는 .env 파일을 사용하여 리소스를 관리할 수 있도록 .bot 파일이 _사용되지 않습니다_. .bot 파일에서 appsettings.json 또는 .env 파일로 설정을 마이그레이션하는 방법에 대한 자세한 내용은 [봇 리소스 관리](v4sdk/bot-file-basics.md)를 참조하세요.

### <a name="login-to-azure"></a>Azure에 로그인

이미 로컬에서 봇을 만들고 테스트했고, 이제는 이 봇을 Azure에 배포하려고 합니다. 명령 프롬프트를 열어 Azure Portal에 로그인합니다.

```cmd
az login
```
로그인할 수 있는 브라우저 창이 열립니다.

### <a name="set-the-subscription"></a>구독 설정
사용할 기본 구독을 설정합니다.

```cmd
az account set --subscription "<azure-subscription>"
```

봇을 배포하는 데 사용할 구독이 확실하지 않은 경우 `az account list` 명령을 사용하여 계정에 대한 구독 목록을 볼 수 있습니다. 봇 폴더로 이동합니다.

### <a name="create-an-app-registration"></a>앱 등록 만들기
애플리케이션을 등록하면 Azure AD를 사용하여 사용자를 인증하고 사용자 리소스에 대한 액세스를 요청할 수 있습니다. 봇이 Bot Framework Service에 액세스하여 인증된 메시지를 보내고 받으려면 Azure에서 등록된 앱이 필요합니다. Azure CLI를 통해 등록을 만들려면 다음 명령을 수행합니다.

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| 옵션   | 설명 |
|:---------|:------------|
| display-name | 애플리케이션의 표시 이름입니다. |
| 암호 | 앱 암호, 즉 '클라이언트 비밀'입니다. 암호는 길이가 16자 이상이며 알파벳 대문자 또는 소문자 1자 이상과, 특수 문자 1자 이상을 포함해야 합니다.|
| available-to-other-tenants| 모든 Azure AD 테넌트에서 애플리케이션을 사용할 수 있습니다. 봇이 Azure Bot Service 채널에서 작동하려면 이 값이 `true`여야 합니다.|

위의 명령은 `appId` 키가 있는 JSON을 출력하고, `appId` 매개 변수에 사용되는 ARM 배포를 위해 이 키 값을 저장합니다. 제공된 암호는 `appSecret` 매개 변수에 사용됩니다.

새 리소스 그룹 또는 기존 리소스 그룹에 봇을 배포할 수 있습니다. 본인에게 가장 적합한 옵션을 선택합니다.

# <a name="deploy-via-arm-template-with-new-resource-grouptabnewrg"></a>[ARM 템플릿을 통해 배포(**새** 리소스 그룹 포함)](#tab/newrg)

### <a name="create-azure-resources"></a>Azure 리소스 만들기

새 리소스 그룹을 Azure에 만들고 ARM 템플릿을 사용하여 그 안에 지정된 리소스를 만듭니다. 이 경우 새 App Service 계획, 웹앱 및 봇 채널 등록을 제공합니다.

```cmd
az deployment create --name "<name-of-deployment>" --template-file "template-with-new-rg.json" --location "location-name" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" botSku=F0 newAppServicePlanName="<name-of-app-service-plan>" newWebAppName="<name-of-web-app>" groupName="<new-group-name>" groupLocation="<location>" newAppServicePlanLocation="<location>"
```

| 옵션   | 설명 |
|:---------|:------------|
| 이름 | 배포의 식별 이름입니다. |
| template-file | ARM 템플릿의 경로입니다. 프로젝트의 `deploymentTemplates` 폴더에서 제공하는 `template-with-new-rg.json` 파일을 사용할 수 있습니다. |
| location |위치입니다. 값 출처: `az account list-locations`. `az configure --defaults location=<location>`을 사용하여 기본 위치를 구성할 수 있습니다. |
| 매개 변수 | 배포 매개 변수 값을 제공합니다. `az ad app create` 명령 실행에서 가져온 `appId` 값입니다. `appSecret`은 이전 단계에서 제공한 암호입니다. `botId` 매개 변수는 전역적으로 고유해야 하며 변경이 불가능한 봇 ID로 사용됩니다. 또한 변경 가능한 봇의 표시 이름을 구성하는 데도 사용됩니다. `botSku`는 가격 책정 계층으로, F0(무료) 또는 S1(표준)일 수 있습니다. `newAppServicePlanName`은 App Service 계획의 이름입니다. `newWebAppName`은 만들고 있는 웹앱의 이름입니다. `groupName`은 만들고 있는 Azure 리소스 그룹의 이름입니다. `groupLocation`은 Azure 리소스 그룹의 위치입니다. `newAppServicePlanLocation`은 App Service 계획의 위치입니다. |

# <a name="deploy-via-arm-template-with-existing--resource-grouptaberg"></a>[ARM 템플릿을 통해 배포(**기존** 리소스 그룹 포함)](#tab/erg)

### <a name="create-azure-resources"></a>Azure 리소스 만들기

기존 리소스 그룹을 사용할 경우 기존 App Service 계획을 사용하거나 새로 만들 수 있습니다. 두 옵션에 대한 단계는 다음과 같습니다. 

**옵션 1: 기존 App Service 계획 사용** 

이 예에서는 기존 App Service 계획을 사용하지만 새 웹앱과 봇 채널 등록을 만듭니다. 

_참고: botId 매개 변수는 전역적으로 고유해야 하며 변경이 불가능한 봇 ID로 사용됩니다. 또한 변경 가능한 봇의 표시 이름을 구성하는 데도 사용됩니다._

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" existingAppServicePlan="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

**옵션 2: 새 App Service 계획** 

이 경우 새 App Service 계획, 웹앱 및 봇 채널 등록을 만듭니다. 

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" newAppServicePlanName="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

| 옵션   | 설명 |
|:---------|:------------|
| 이름 | 배포의 식별 이름입니다. |
| resource-group | Azure 리소스 그룹의 이름입니다. |
| template-file | ARM 템플릿의 경로입니다. 프로젝트의 `deploymentTemplates` 폴더에서 제공하는 `template-with-preexisting-rg.json` 파일을 사용할 수 있습니다. |
| location |위치입니다. 값 출처: `az account list-locations`. `az configure --defaults location=<location>`을 사용하여 기본 위치를 구성할 수 있습니다. |
| 매개 변수 | 배포 매개 변수 값을 제공합니다. `az ad app create` 명령 실행에서 가져온 `appId` 값입니다. `appSecret`은 이전 단계에서 제공한 암호입니다. `botId` 매개 변수는 전역적으로 고유해야 하며 변경이 불가능한 봇 ID로 사용됩니다. 또한 변경 가능한 봇의 표시 이름을 구성하는 데도 사용됩니다. `newWebAppName`은 만들고 있는 웹앱의 이름입니다. `newAppServicePlanName`은 App Service 계획의 이름입니다. `newAppServicePlanLocation`은 App Service 계획의 위치입니다. |

---

### <a name="retrieve-or-create-necessary-iiskudu-files"></a>필요한 IIS/Kudu 파일을 만들거나 검색

**C# 봇의 경우**

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

--code-dir에 상대적인 .csproj 파일 경로를 제공해야 합니다. --proj-file-path 인수를 사용하면 됩니다. 이 명령은 --code-dir 및 --proj-file-path를 "./MyBot.csproj"로 확인합니다.

**JavaScript 봇의 경우**

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

이 명령은 Node.js가 Azure App Services에서 IIS 작업을 위해 필요한 web.config를 가져옵니다. web.config가 봇의 루트에 저장되었는지 확인합니다.

**TypeScript 봇의 경우**

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

이 명령은 위의 JavaScript와 유사하게 작동하지만 Typescript 봇에 사용됩니다.

### <a name="zip-up-the-code-directory-manually"></a>수동으로 코드 디렉터리 압축

구성되지 않은 [zip 배포 API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url)를 사용하여 봇 코드를 배포할 때는 웹앱/Kudu가 다음과 같이 작동합니다.

_Kudu는 기본적으로 Zip 파일을 통한 배포가 실행 준비되었으며 배포 중에 npm install, dotnet restore/dotnet publish 같은 추가 빌드 단계가 필요하지 않다고 가정합니다._

이 때문에 빌드 코드와 모든 필요한 종속성을 웹앱에 배포할 zip 파일에 포함해야 하며, 그렇지 않으면 봇이 의도대로 작동하지 않게 됩니다.

> [!IMPORTANT]
> 프로젝트 파일을 압축하기 전에 올바른 폴더 _안_에 있어야 합니다. 
> - C# 봇의 경우 .csproj 파일이 있는 폴더입니다. 
> - JS 봇의 경우 app.js 또는 index.js 파일이 있는 폴더입니다. 
>
> 루트 폴더 위치가 올바르지 않을 경우 **봇이 Azure Portal에서 실행되지 못하게 됩니다**.

## <a name="2-deploy-code-to-azure"></a>2. Azure에 코드 배포
이제 Azure Web App에 코드를 배포할 준비가 되었습니다. 명령줄에서 다음 명령을 실행하여 웹앱에 대해 kudu zip 푸시 배포를 사용하는 배포를 수행합니다.

```cmd
az webapp deployment source config-zip --resource-group "<new-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| 옵션   | 설명 |
|:---------|:------------|
| resource-group | 이전에 만든 Azure의 리소스 그룹 이름입니다. |
| 이름 | 이전에 사용한 웹앱의 이름입니다. |
| src  | 만든 압축 파일의 경로입니다. |

## <a name="3-test-in-web-chat"></a>3. 웹 채팅에서 테스트
- Azure Portal에서 웹앱 봇 블레이드로 이동합니다.
- **봇 관리** 섹션에서 **웹 채팅에서 테스트**를 클릭합니다. Azure Bot Service가 웹 채팅 컨트롤을 로드하고 봇에 연결합니다.
- 배포가 성공하면 몇 초 동안 기다린 후 필요에 따라 웹앱을 다시 시작하여 캐시를 지웁니다. 웹앱 봇 블레이드로 돌아가서 Azure Portal에 제공된 웹 채팅을 사용하여 테스트합니다.

## <a name="additional-information"></a>추가 정보
봇이 Azure에 배포되면 사용하는 서비스에 대한 비용을 지불해야 합니다. [청구 및 비용 관리](https://docs.microsoft.com/en-us/azure/billing/) 문서는 Azure 청구를 이해하고, 사용량과 비용을 모니터링하며, 계정과 구독을 관리하는 데 도움이 됩니다.

## <a name="next-steps"></a>다음 단계
> [!div class="nextstepaction"]
> [지속적인 배포 설정](bot-service-build-continuous-deployment.md)
