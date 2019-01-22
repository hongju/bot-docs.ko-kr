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
ms.date: 01/07/2019
ms.openlocfilehash: 3ebc13cf9e2d111d716d081c36f125d28a441811
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360743"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Azure CLI를 사용하여 봇 배포

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

봇을 만들어 로컬로 테스트한 후에는 어디서나 액세스할 수 있도록 Azure에 배포할 수 있습니다. 봇이 Azure에 배포되면 사용하는 서비스에 대한 비용을 지불해야 합니다. [청구 및 비용 관리](https://docs.microsoft.com/en-us/azure/billing/) 문서는 Azure 청구를 이해하고, 사용량과 비용을 모니터링하며, 계정과 구독을 관리하는 데 도움이 됩니다.

이 문서에서는 `az` 및 `msbot` cli를 사용하여 C# 및 JavaScript 봇을 Azure에 배포하는 방법을 보여 줍니다. 단계를 수행하기 전에 이 문서를 참조하면 봇 배포와 관련된 내용을 완전히 이해할 수 있습니다.

## <a name="prerequisites"></a>필수 조건

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>az cli를 사용하여 JavaScript 및 C# 봇 배포

이미 로컬에서 봇을 만들고 테스트했고, 이제는 이 봇을 Azure에 배포하려고 합니다. 이러한 단계에서는 필요한 Azure 리소스를 만들었다고 가정합니다.

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

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>다운로드한 .bot 파일을 암호 해독하여 프로젝트에 사용

.bot 파일의 민감한 정보는 암호화되어 있습니다.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

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

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## <a name="additional-resources"></a>추가 리소스

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>다음 단계
> [!div class="nextstepaction"]
> [지속적인 배포 설정](bot-service-build-continuous-deployment.md)
