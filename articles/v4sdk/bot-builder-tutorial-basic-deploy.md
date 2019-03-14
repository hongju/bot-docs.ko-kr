---
title: 기본 봇 만들기 및 배포 자습서 | Microsoft Docs
description: 기본 봇을 만들어서 Azure에 배포하는 방법을 알아봅니다.
keywords: echo 봇, 배포, azure, 자습서
author: Ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/9/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7927ab97dc88657a198c8f1d8e56bcb1ddf0fabe
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568240"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>자습서: 기본 봇 만들기 및 배포

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

이 자습서에서는 Bot Framework SDK를 사용하여 기본 봇을 만들고 Azure에 배포하는 방법을 안내합니다. 이미 기본 봇을 만들어서 로컬로 실행 중인 경우 [봇 배포](#deploy-your-bot) 섹션으로 건너뛰세요.

이 자습서에서는 다음 방법에 대해 알아봅니다.

> [!div class="checklist"]
> * 기본 Echo 봇 만들기
> * 로컬로 봇을 실행하고 상호 작용
> * 봇 게시

Azure 구독이 아직 없는 경우 시작하기 전에 [무료 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 을 만듭니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

---

## <a name="deploy-your-bot"></a>봇 배포

이제 Azure에 봇을 배포합니다.

### <a name="prerequisites"></a>필수 조건

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]

### <a name="login-to-azure-cli-and-set-your-subscription"></a>Azure CLI에 로그인하여 구독을 설정합니다.

이미 로컬에서 봇을 만들고 테스트했고, 이제는 이 봇을 Azure에 배포하려고 합니다.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### <a name="create-a-web-app-bot"></a>웹앱 봇 만들기

봇을 게시할 리소스 그룹이 아직 없는 경우 새로 만듭니다.

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

먼저 Azure에 로그인하는 데 사용하는 이메일 계정 유형에 따라 적용되는 지침을 참조한 후에 계속 진행합니다.

#### <a name="msa-email-account"></a>MSA 이메일 계정

[MSA](https://en.wikipedia.org/wiki/Microsoft_account) 이메일 계정을 사용 중인 경우 애플리케이션 등록 포털에서 `az bot create`에 사용할 앱 ID와 앱 암호를 만들어야 합니다.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### <a name="business-or-school-account"></a>회사 또는 학교 계정

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### <a name="download-the-bot-from-azure"></a>Azure에서 봇 다운로드

그런 다음, 방금 만든 봇을 다운로드합니다. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

[!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>다운로드한 .bot 파일을 암호 해독하여 프로젝트에 사용

.bot 파일의 중요한 정보가 암호화되어 있으므로, 쉽게 사용할 수 있도록 암호 해독합니다. 

먼저 다운로드한 봇 디렉터리로 이동합니다.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="test-your-bot-locally"></a>로컬에서 봇 테스트

현 상태에서 봇은 이전의 `.bot` 파일과 동일한 방식으로 작동해야 합니다. 새 `.bot` 파일을 사용하여 예상대로 작동하는지 확인하세요.

이제 에뮬레이터에 *프로덕션* 엔드포인트가 보일 것입니다. 보이지 않으면 여전히 기존 `.bot` 파일을 사용 중일 것입니다.

### <a name="publish-your-bot-to-azure"></a>Azure에 봇 게시

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

이제 웹 채팅에서 봇을 테스트할 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>다음 단계
> [!div class="nextstepaction"]
> [봇에 서비스 추가](bot-builder-tutorial-add-qna.md)

