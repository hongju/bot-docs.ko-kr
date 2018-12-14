---
title: 봇 소스 코드 다운로드 및 재배포 | Microsoft Docs
description: Bot Service를 다운로드하고 게시하는 방법을 알아봅니다.
keywords: 원본 코드 다운로드, 재배포, 배포, zip 파일, 게시
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/26/2018
ms.openlocfilehash: 19dd474c16224cc811a214acea6e9cb51da95b3f
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332767"
---
# <a name="download-and-redeploy-bot-code"></a>봇 코드 다운로드 및 재배포
Azure Bot Service를 사용하면 봇의 전체 원본 프로젝트를 다운로드할 수 있으므로 원하는 IDE를 사용하여 로컬에서 작업할 수 있습니다. 코드 업데이트를 수행한 후 Azure Portal에 다시 변경 사항을 게시할 수 있습니다. Azure Portal 및 `az` cli를 사용하여 코드를 다운로드하는 방법을 보여드리겠습니다. 또한 Visual Studio 및 `az` cli 도구를 사용하여 업데이트된 봇 코드를 다시 배포하는 방법을 알아보겠습니다. 본인에게 가장 적합한 메서드를 선택할 수 있습니다.

## <a name="prerequisites"></a>필수 조건
-  [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)
- `az extension add -n botservice` 명령을 사용하여 az botservice 확장 설치

### <a name="download-code-using-the-azure-portal"></a>Azure Portal을 사용하여 코드 다운로드
[Azure Portal](https://portal.azure.com)에서 코드를 다운로드하려면 다음을 수행합니다.
1. 봇에 대한 블레이드를 엽니다.
1. **봇 관리** 섹션에서 **빌드**를 클릭합니다.
1. **소스 코드 다운로드**에서 **zip 파일 다운로드**를 클릭합니다.
1. Azure가 다운로드 URI를 준비할 때까지 기다린 후 알림에서 **zip 파일 다운로드**를 클릭합니다.
1. 로컬 디렉터리에 .zip 파일을 저장하고 압축을 풉니다.

C# 봇이 있는 경우 아래 표시된 것처럼 .bot 파일 정보를 포함하도록 `appsettings.json` 파일을 업데이트합니다.

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```

node.js 봇이 있는 경우 다음과 같은 항목이 포함된 `.env` 파일을 추가합니다.
```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

`botFilePath`는 봇의 이름을 참조하고 "yourbasicBot.bot"을 자체 봇 이름으로 바꿉니다. `botFileSecret` 키를 가져오려면 봇의 키를 생성하는 방법에 대한 [봇 파일 암호화](https://aka.ms/bot-file-encryption) 문서를 참조하세요.

다음으로, 기존 원본 파일을 편집하거나 프로젝트에 새 원본 파일을 추가하여 원본을 변경합니다. 에뮬레이터를 사용하여 코드를 테스트합니다. 수정된 코드를 Azure Portal에 다시 배포할 준비가 된 경우 아래 지침을 따르세요.

### <a name="publish-code-using-visual-studio"></a>Visual Studio를 사용하여 코드 게시
1. Visual Studio에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 **게시...** 를 클릭합니다. **게시** 창이 열립니다.

![Azure 게시](~/media/azure-bot-build/azure-csharp-publish.png)

2. 프로젝트의 프로필을 선택합니다.
3. 프로젝트에서 _publish.cmd_ 파일에 나열된 암호를 복사합니다.
4. **게시**를 클릭합니다.
5. 메시지가 표시되면 3단계에서 복사한 암호를 입력합니다.   

프로젝트가 구성된 후 프로젝트 변경 내용이 Azure에 게시됩니다. 

다음으로, `az` cli를 사용하여 코드를 다운로드하고 다시 배포하는 방법을 살펴보겠습니다.

### <a name="download-code-using-azure-cli"></a>Azure CLI를 사용하여 코드 다운로드

먼저 az cli 도구를 사용하여 Azure Portal에 로그인합니다.

```azcli
az login
```

고유한 임시 인증 코드를 묻는 메시지가 나타납니다. 로그인하려면 웹 브라우저를 사용하여 Microsoft [디바이스 로그인](https://microsoft.com/devicelogin)을 방문하고 계속하려면 CLI에서 제공하는 코드를 붙여넣습니다.

`az` cli를 사용하여 코드를 다운로드하려면 다음 명령을 사용합니다.
```azcli
az bot download --name "my-bot-name" --resource-group "my-resource-group"`
```
코드가 다운로드되면 다음을 수행합니다.
- C# 봇의 경우 아래 표시된 것처럼 .bot 파일 정보를 포함하도록 appsettings.json 파일을 업데이트합니다.

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```

- node.js 봇의 경우 다음과 같은 항목이 포함된 .env 파일을 추가합니다.

```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

다음으로, 기존 원본 파일을 편집하거나 프로젝트에 새 원본 파일을 추가하여 원본을 변경합니다. 에뮬레이터를 사용하여 코드를 테스트합니다. 수정된 코드를 Azure Portal에 다시 배포할 준비가 된 경우 아래 지침을 따르세요.

### <a name="login-to-azure-cli-by-running-the-following-command"></a>다음 명령을 실행하여 Azure CLI에 로그인합니다.
이미 로그인한 경우 이 단계를 건너뛸 수 있습니다.

```azcli
az login
```
고유한 임시 인증 코드를 묻는 메시지가 나타납니다. 로그인하려면 웹 브라우저를 사용하여 Microsoft [디바이스 로그인](https://microsoft.com/devicelogin)을 방문하고 계속하려면 CLI에서 제공하는 코드를 붙여넣습니다.

### <a name="publish-code-using-azure-cli"></a>Azure CLI를 사용하여 코드 게시
`az` cli를 사용하여 Azure에 다시 코드를 게시하려면 다음 명령을 사용합니다.
```azcli
az bot publish --name "my-bot-name" --resource-group "my-resource-group" --code-dir <path to directory> 
```

`code-dir` 옵션을 사용하여 사용할 디렉터리를 지정할 수 있습니다. 이 디릭터리가 지정되지 않은 경우 `az bot publish` 명령에서 게시할 로컬 디렉터리를 사용합니다.

## <a name="next-steps"></a>다음 단계
이제 Azure에 변경 내용을 다시 업로드하는 방법을 알았으므로 봇에 대한 지속적인 배포를 설정할 수 있습니다.

> [!div class="nextstepaction"]
> [지속적인 배포 설정](bot-service-build-continuous-deployment.md)
