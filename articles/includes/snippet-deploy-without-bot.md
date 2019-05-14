---
ms.openlocfilehash: 4b5181babf728861107a0c7bc28f844491761a7a
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033887"
---
배포를 시작하기 전에 [Azure cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) 및 [dotnet cli](https://dotnet.microsoft.com/download)의 최신 버전이 있는지 확인합니다. dotnet cli가 없으면 위에서 제공하는 링크에서 .Net Core 런타임 옵션을 사용하여 설치합니다. 

### <a name="login-to-azure-cli-and-set-your-subscription"></a>Azure CLI에 로그인하여 구독을 설정합니다.
이미 로컬에서 봇을 만들고 테스트했고, 이제는 이 봇을 Azure에 배포하려고 합니다. 명령 프롬프트를 열어 Azure Portal에 로그인합니다.

```cmd
az login
```
### <a name="set-the-subscription"></a>구독 설정

사용할 기본 구독을 설정합니다.

```cmd
az account set --subscription "<azure-subscription>"
```

봇을 배포하는 데 사용할 구독이 확실하지 않은 경우 `az account list` 명령을 사용하여 계정에 대한 구독 목록을 볼 수 있습니다.

봇 폴더로 이동합니다.
`cd <local-bot-folder>`

### <a name="create-a-web-app-bot-in-azure"></a>Azure에서 웹앱 봇 만들기 

로컬 봇을 게시할 리소스 그룹이 아직 없는 경우 새로 만듭니다.

```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| 옵션     | 설명 |
|:-----------|:---|
| 이름     | 리소스 그룹의 고유한 이름입니다. 이름에 공백 또는 밑줄을 넣지 마세요. |
| location | 리소스 그룹을 만드는 데 사용되는 지리적 위치입니다. 예를 들어 `eastus`, `westus`, `westus2` 등입니다. 위치 목록에는 `az account list-locations`를 사용합니다. |

그런 다음, 봇을 게시할 봇 리소스를 만듭니다. 그러면 Azure에서 필요한 리소스가 프로비전되고 봇 웹앱이 만들어지며, 이 웹앱을 로컬 봇으로 덮어쓸 것입니다. 

먼저 Azure에 로그인하는 데 사용하는 이메일 계정 유형에 따라 적용되는 지침을 참조한 후에 계속 진행합니다.

#### <a name="msa-email-account"></a>MSA 이메일 계정
MSA 이메일 계정을 사용 중인 경우 애플리케이션 등록 포털에서 `az bot create` 명령에 사용할 앱 ID와 앱 암호를 만들어야 합니다.
1. [**애플리케이션 등록 포털**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)로 이동합니다.
1. **앱 추가**를 클릭하여 애플리케이션을 등록하고, **애플리케이션 ID**를 만들고, **새 암호 생성**을 수행합니다. 애플리케이션 및 암호가 이미 있지만 암호가 기억 나지 않는 경우 애플리케이션 비밀 섹션에서 새 암호를 생성해야 합니다.
1. `az bot create` 명령에 사용할 수 있도록 방금 생성한 애플리케이션 ID와 새 암호를 모두 저장합니다.  

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| 옵션 | 설명 |
|:---|:---|
| 이름 | Azure에서 봇을 배포하는 데 사용되는 고유 이름입니다. 로컬 봇과 동일한 이름일 수도 있습니다. 이름에 공백 또는 밑줄을 넣지 마세요. |
| location | 봇 서비스 리소스를 만드는 데 사용되는 지리적 위치입니다. 예를 들어 `eastus`, `westus`, `westus2` 등입니다. |
| resource-group | 봇을 만들 리소스 그룹의 이름입니다. `az configure --defaults group=<name>`을 사용하여 기본 그룹을 구성할 수 있습니다. |
| appId | 봇에서 사용할 Microsoft 계정 ID(MSA ID)입니다. |
| 암호 | 봇의 Microsoft 계정(MSA) 암호입니다. |

#### <a name="business-or-school-account"></a>회사 또는 학교 계정

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```
| 옵션 | 설명 |
|:---|:---|
| 이름 | Azure에서 봇을 배포하는 데 사용되는 고유 이름입니다. 로컬 봇과 동일한 이름일 수도 있습니다. 이름에 공백 또는 밑줄을 넣지 마세요. |
| location | 봇 서비스 리소스를 만드는 데 사용되는 지리적 위치입니다. 예를 들어 `eastus`, `westus`, `westus2` 등입니다. |
| lang | 봇을 만드는 데 사용할 언어는 `Csharp` 또는 `Node`입니다. 기본값은 `Csharp`입니다. |
| resource-group | 봇을 만들 리소스 그룹의 이름입니다. `az configure --defaults group=<name>`을 사용하여 기본 그룹을 구성할 수 있습니다. |

#### <a name="update-appsettingsjson-or-env-file"></a>appsettings.json 또는 .env 파일 업데이트
봇을 만든 후 콘솔 창에 다음 정보가 표시됩니다. 

```JSON
{
  "appId": "as234-345b-4def-9047-a8a44b4s",
  "appPassword": "34$#w%^$%23@334343",
  "endpoint": "https://mybot.azurewebsites.net/api/messages",
  "id": "mybot",
  "name": "mybot",
  "resourceGroup": "botresourcegroup",
  "serviceName": "mybot",
  "subscriptionId": "234532-8720-5632-a3e2-a1qw234",
  "tenantId": "32f955bf-33f1-43af-3ab-23d009defs47",
  "type": "abs"
}
```

`appId` 및 `appPassword` 값을 복사하여 appsettings.json 또는 .env 파일에 붙여넣어야 합니다. 예: 

```JSON
{
  MicrosoftAppId: "as234-345b-4def-9047-a8a44b4s",
  MicrosoftAppPassword: "34$#w%^$%23@334343"
}
```
appsettings.json 또는 .env 파일에 봇에 대해 프로비저닝한 다른 서비스용 추가 키가 있는 경우 해당 항목을 삭제하지 않습니다.

파일을 저장합니다.

이제 봇을 만들 때 사용한 프로그래밍 언어(**C#** 또는 **JS**)에 따라 해당하는 단계를 따릅니다.

**C# 봇:** 

명령 프롬프트를 열고 프로젝트 폴더로 이동합니다. 명령줄에서 다음 명령을 실행합니다.

| Task | 명령 |
|:-----|:--------|
| 1. 프로젝트 종속성 복원 | `dotnet restore`|
| 2. 프로젝트 빌드     | `dotnet build` |
| 3. 프로젝트 파일 압축 | 아무 유틸리티나 사용하여 프로젝트 파일을 압축합니다. .csproj 파일이 있는 폴더로 이동하여 이 수준의 모든 파일과 폴더를 선택하여 압축 폴더를 만듭니다. |
| 4. 빌드 배포 설정 구성 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|
| 5. 스크립트 생성기 인수 설정 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_SCRIPT_GENERATOR_ARGS="--aspNetCore mybot.csproj"`|

**JS 봇**
1. [여기](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps)에서 web.config를 다운로드하여 프로젝트 폴더에 저장합니다. 
1. 파일을 편집하고 모든 "server.js"를 "index.js"로 바꿉니다. 
1. 파일을 저장합니다.

명령 프롬프트를 열고 프로젝트 폴더로 이동합니다. 명령줄에서 다음 명령을 실행합니다.

| Task | 명령 |
|:-----|:--------|
| 1. 노드 모듈 설치 | `npm install` |
| 2. 프로젝트 파일 압축 | 아무 유틸리티나 사용하여 프로젝트 파일을 압축합니다. .csproj 파일이 있는 폴더로 이동하여 이 수준의 모든 파일과 폴더를 선택하여 압축 폴더를 만듭니다. |
| 3. 빌드 배포 설정 구성 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|
