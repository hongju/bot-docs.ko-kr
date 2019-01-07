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
ms.date: 12/14/2018
ms.openlocfilehash: 19960940a40fa291534bc1f88290bc6a7da109e0
ms.sourcegitcommit: 8c10aa7372754596a3aa7303a3a893dd4939f7e9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53654338"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Azure CLI를 사용하여 봇 배포

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

봇을 만들어 로컬로 테스트한 후에는 어디서나 액세스할 수 있도록 Azure에 배포할 수 있습니다. 봇이 Azure에 배포되면 사용하는 서비스에 대한 비용을 지불해야 합니다. [청구 및 비용 관리](https://docs.microsoft.com/en-us/azure/billing/) 문서는 Azure 청구를 이해하고, 사용량과 비용을 모니터링하며, 계정과 구독을 관리하는 데 도움이 됩니다.

이 문서에서는 `az` 및 `msbot` cli를 사용하여 C# 및 JavaScript 봇을 Azure에 배포하는 방법을 보여 줍니다. 단계를 수행하기 전에 이 문서를 참조하면 봇 배포와 관련된 내용을 완전히 이해할 수 있습니다.


## <a name="prerequisites"></a>필수 조건
- Azure 구독이 아직 없는 경우 시작하기 전에 [체험 계정](https://azure.microsoft.com/free/)을 만듭니다.
- 최신 버전의 [Azure CLI 도구](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)를 설치합니다.
- `az` 도구에 대한 최신 `botservice` 확장을 설치합니다.
  - 먼저 `az extension remove -n botservice` 명령을 사용하여 이전 버전을 제거합니다. 다음으로 `az extension add -n botservice` 명령을 사용하여 최신 버전을 설치합니다.
- 최신 버전의 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 도구를 설치합니다.
- 최근에 출시된 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) 버전을 설치합니다.
- [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)를 설치 및 구성합니다.
- [.bot](v4sdk/bot-file-basics.md) 파일에 대한 지식이 필요합니다.

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>az cli를 사용하여 JavaScript 및 C# 봇 배포

이미 로컬에서 봇을 만들고 테스트했고, 이제는 이 봇을 Azure에 배포하려고 합니다. 이러한 단계에서는 필요한 Azure 리소스를 만들었다고 가정합니다.

명령 프롬프트를 열어 Azure Portal에 로그인합니다.

```cmd
az login
```

로그인할 수 있는 브라우저 창이 열립니다.

### <a name="set-the-subscription"></a>구독 설정

사용할 기본 구독을 설정합니다.

```cmd
az account set --subscription "<azure-subscription>"
```

봇을 배포하는 데 사용할 구독이 확실하지 않은 경우 `az account list` 명령을 사용하여 계정에 대한 `subscriptions`의 목록을 볼 수 있습니다.

봇 폴더로 이동합니다.

```cmd
cd <local-bot-folder>
```

### <a name="create-a-web-app-bot"></a>웹앱 봇 만들기

봇을 게시할 봇 리소스를 만듭니다.

먼저 Azure에 로그인하는 데 사용하는 이메일 계정 유형에 따라 적용되는 지침을 참조한 후에 계속 진행합니다.

#### <a name="msa-email-account"></a>MSA 이메일 계정

[MSA](https://en.wikipedia.org/wiki/Microsoft_account) 이메일 계정을 사용 중인 경우 애플리케이션 등록 포털에서 `az bot create`에 사용할 앱 ID와 앱 암호를 만들어야 합니다.

1. [**애플리케이션 등록 포털**](https://apps.dev.microsoft.com/)로 이동합니다.
1. **앱 추가**를 클릭하여 애플리케이션을 등록하고, **애플리케이션 ID**를 만들고, **새 암호 생성**을 수행합니다. 애플리케이션 및 암호가 이미 있지만 암호가 기억 나지 않는 경우 애플리케이션 비밀 섹션에서 새 암호를 생성해야 합니다.
1. `az bot create` 명령에 사용할 수 있도록 방금 생성한 애플리케이션 ID와 새 암호를 모두 저장합니다.  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| 옵션 | 설명 |
|:---|:---|
| --name | Azure에서 봇을 배포하는 데 사용되는 고유 이름입니다. 로컬 봇과 동일한 이름일 수도 있습니다. 이름에 공백 또는 밑줄을 넣지 마세요. |
| --location | 봇 서비스 리소스를 만드는 데 사용되는 지리적 위치입니다. 예를 들어 `eastus`, `westus`, `westus2` 등입니다. |
| --lang | 봇을 만드는 데 사용할 언어는 `Csharp` 또는 `Node`입니다. 기본값은 `Csharp`입니다. |
| --resource-group | 봇을 만들 리소스 그룹의 이름입니다. `az configure --defaults group=<name>`을 사용하여 기본 그룹을 구성할 수 있습니다. |
| --appid | 봇에서 사용할 Microsoft 계정 ID(MSA ID)입니다. |
| --password | 봇의 Microsoft 계정(MSA) 암호입니다. |

#### <a name="business-or-school-account"></a>회사 또는 학교 계정

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| 옵션 | 설명 |
|:---|:---|
| --name | Azure에서 봇을 배포하는 데 사용되는 고유 이름입니다. 로컬 봇과 동일한 이름일 수도 있습니다. 이름에 공백 또는 밑줄을 넣지 마세요. |
| --location | 봇 서비스 리소스를 만드는 데 사용되는 지리적 위치입니다. 예를 들어 `eastus`, `westus`, `westus2` 등입니다. |
| --lang | 봇을 만드는 데 사용할 언어는 `Csharp` 또는 `Node`입니다. 기본값은 `Csharp`입니다. |
| --resource-group | 봇을 만들 리소스 그룹의 이름입니다. `az configure --defaults group=<name>`을 사용하여 기본 그룹을 구성할 수 있습니다. |

### <a name="download-the-bot-from-azure"></a>Azure에서 봇 다운로드

그런 다음, 방금 만든 봇을 다운로드합니다. 이 명령은 save-path 아래에 하위 디렉터리를 만드는데 지정된 경로가 이미 있어야 합니다.

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| 옵션 | 설명 |
|:---|:---|
| --name | Azure에서 봇의 이름입니다. |
| --resource-group | 봇이 속하는 리소스 그룹의 이름입니다. |
| --save-path | 봇 코드를 다운로드할 기존 디렉터리입니다. |

### <a name="decrypt-the-downloaded-bot-file"></a>다운로드된 .bot 파일의 암호 해독

.bot 파일의 민감한 정보는 암호화되어 있습니다.

암호화 키를 가져옵니다.

1. [Azure Portal](http://portal.azure.com/)에 로그인합니다.
1. 봇의 Web App 봇 리소스를 엽니다.
1. 봇의 **애플리케이션 설정**을 엽니다.
1. **애플리케이션 설정** 창에서 **애플리케이션 설정**이 나올 때까지 아래로 스크롤합니다.
1. **botFileSecret**을 찾고 해당 값을 복사합니다.

.bot 파일의 암호를 해독합니다.

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| 옵션 | 설명 |
|:---|:---|
| --bot | 다운로드된 .bot 파일의 상대 경로입니다. |
| --secret | 암호화 키입니다. |

### <a name="use-the-downloaded-bot-file-in-your-project"></a>프로젝트에서 다운로드한 .bot 파일 사용

암호를 해독한 .bot 파일을 로컬 봇 프로젝트가 포함된 디렉터리에 복사합니다.

이 새.bot 파일을 사용하도록 봇을 업데이트합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**appsettings.json**에서 새 .bot 파일을 가리키도록 **botFilePath** 속성을 업데이트합니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**.env**에서 새 .bot 파일을 가리키도록 **botFilePath** 속성을 업데이트합니다.

---

### <a name="update-the-bot-file"></a>.bot 파일 업데이트

봇이 LUIS, QnA Maker 또는 Dispatch 서비스를 사용하는 경우 이에 대한 참조를 .bot 파일에 추가해야 합니다. 그렇지 않을 경우 이 단계를 건너뛸 수 있습니다.

1. 새 .bot 파일을 사용하여 BotFramework Emulator에서 봇을 엽니다. 이 봇은 로컬에서 실행할 필요가 없습니다.
1. **BOT EXPLORER** 패널에서 **SERVICES** 섹션을 확장합니다.
1. LUIS 앱에 대한 참조를 추가하려면 **SERVICES** 오른쪽에 있는 더하기 기호(+)를 클릭합니다.
   1. **LUIS(Language Understanding) 추가**를 선택합니다.
   1. Azure 계정에 로그인하라는 메시지가 표시되면 로그인합니다.
   1. 액세스할 수 있는 LUIS 애플리케이션의 목록이 표시됩니다. 봇에 사용할 애플리케이션을 선택합니다.
1. QnA Maker 기술 자료에 대한 참조를 추가하려면 **SERVICES** 오른쪽에 있는 더하기 기호(+)를 클릭합니다.
   1. **QnA Maker 추가**를 선택합니다.
   1. Azure 계정에 로그인하라는 메시지가 표시되면 로그인합니다.
   1. 액세스할 수 있는 기술 자료의 목록이 표시됩니다. 봇에 사용할 애플리케이션을 선택합니다.
1. Dispatch 모델에 대한 참조를 추가하려면 **SERVICES** 오른쪽에 있는 더하기 기호(+)를 클릭합니다.
   1. **Dispatch 추가**를 선택합니다.
   1. Azure 계정에 로그인하라는 메시지가 표시되면 로그인합니다.
   1. 액세스할 수 있는 Dispatch 모델의 목록이 표시됩니다. 봇에 사용할 애플리케이션을 선택합니다.

### <a name="test-your-bot-locally"></a>로컬에서 봇 테스트

현 상태에서 봇은 이전의 .bot 파일과 동일한 방식으로 작동해야 합니다. 새 .bot 파일을 사용하여 예상대로 작동하는지 확인하세요.

### <a name="publish-your-bot-to-azure"></a>Azure에 봇 게시

<!-- TODO: re-encrypt your .bot file? -->

Azure에 로컬 봇을 게시하세요. 이 단계는 시간이 약간 걸릴 수 있습니다.

```cmd
az bot publish --name <bot-resource-name> --proj-file "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

<!-- Question: What should --proj-file be for a Node project? -->

| 옵션 | 설명 |
|:---|:---|
| --name | Azure에서 봇의 리소스 이름입니다. |
| --proj-file | 게시해야 하는 시작 프로젝트 파일 이름(.csproj 제외)입니다. 예:  EnterpriseBot. |
| --resource-group | 리소스 그룹의 이름입니다. |
| --code-dir | 업로드할 봇 코드가 있는 디렉터리입니다. |

"배포 성공!" 메시지와 함께 이 작업이 완료되면 봇이 Azure에 배포됩니다.

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

암호화 키 설정을 지웁니다.

1. [Azure Portal](http://portal.azure.com/)에 로그인합니다.
1. 봇의 Web App 봇 리소스를 엽니다.
1. 봇의 **애플리케이션 설정**을 엽니다.
1. **애플리케이션 설정** 창에서 **애플리케이션 설정**이 나올 때까지 아래로 스크롤합니다.
1. **botFileSecret**을 찾아 삭제합니다.

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
