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
ms.date: 02/07/2019
ms.openlocfilehash: b4c3b982bf061b3a24c6d240b05dc40b0cf07816
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971430"
---
# <a name="deploy-your-bot"></a>봇 배포 

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

봇을 만들어 로컬로 테스트한 후에는 어디서나 액세스할 수 있도록 Azure에 배포할 수 있습니다. 봇이 Azure에 배포되면 사용하는 서비스에 대한 비용을 지불해야 합니다. [청구 및 비용 관리](https://docs.microsoft.com/en-us/azure/billing/) 문서는 Azure 청구를 이해하고, 사용량과 비용을 모니터링하며, 계정과 구독을 관리하는 데 도움이 됩니다.

이 문서에서는 C# 및 JavaScript 봇을 Azure에 배포하는 방법을 보여 줍니다. 단계를 수행하기 전에 이 문서를 참조하면 봇 배포와 관련된 내용을 완전히 이해할 수 있습니다.

## <a name="prerequisites"></a>필수 조건
- 최신 버전의 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 도구를 설치합니다.
- 로컬 머신에서 개발한 [CSharp](./dotnet/bot-builder-dotnet-sdk-quickstart.md) 또는 [JavaScript](./javascript/bot-builder-javascript-quickstart.md) 봇이 있어야 합니다. 

## <a name="create-a-web-app-bot-in-azure"></a>Azure에서 웹앱 봇 만들기
이미 Azure에서 사용하려는 봇을 만든 경우 이 섹션은 선택 사항입니다.

1. [Azure Portal](https://portal.azure.com)에 로그인합니다.
1. Azure Portal의 왼쪽 위 모서리에 있는 **새 리소스 만들기** 링크를 클릭하고 **AI + Machine Learning > 웹앱 봇**을 선택합니다.
1. 웹앱 봇에 대한 정보가 포함된 새 블레이드가 열립니다. 
1. **Bot Service** 블레이드에서 봇에 대해 요청되는 정보를 제공합니다.
1. **만들기**를 클릭하여 서비스를 만들고 봇을 클라우드에 배포합니다. 이 프로세스에는 몇 분 정도 걸릴 수 있습니다.

## <a name="download-the-source-code"></a>소스 코드 다운로드
1. **봇 관리** 섹션에서 **빌드**를 클릭합니다.
1. 오른쪽 창에서 **Bot 소스 코드 다운로드** 링크를 클릭합니다.
1. 표시되는 메시지에 따라 코드를 다운로드한 다음, 폴더의 압축을 풉니다.

## <a name="decrypt-the-bot-file"></a>.bot 파일 암호 해독
Azure Portal에서 다운로드한 소스 코드에는 암호화된 .bot 파일이 포함되어 있습니다. 값을 로컬 .bot 파일에 복사하려면 암호를 해독해야 합니다.  

1. Azure Portal에서 봇용 웹앱 봇 리소스를 엽니다.
1. 봇의 **애플리케이션 설정**을 엽니다.
1. **애플리케이션 설정** 창에서 **애플리케이션 설정**까지 아래로 스크롤합니다.
1. **botFileSecret**을 찾고 해당 값을 복사합니다.

`msbot cli`을 사용하여 파일의 암호를 해독합니다.
```
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

## <a name="update-the-bot-file"></a>.bot 파일 업데이트
암호를 해독한 .bot 파일을 엽니다. `services` 섹션에 나열된 항목을 복사하여 로컬 .bot 파일에 추가합니다. 예: 

```
{
   "type": "endpoint",
   "name": "production",
   "endpoint": "https://<something>.azurewebsites.net/api/messages",
   "appId": "<App Id>",
   "appPassword": "<App Password>",
   "id": "2
}
```

파일을 저장합니다.
 
## <a name="setup-a-repository"></a>리포지토리 설정
즐겨찾는 Git 원본 제어 공급자를 사용하여 Git 리포지토리를 만듭니다. 코드를 리포지토리에 커밋합니다.
 
## <a name="update-app-settings-in-azure"></a>Azure에서 앱 설정 업데이트
1. Azure Portal에서 봇용 **웹앱 봇** 리소스를 엽니다.
1. 봇의 **애플리케이션 설정**을 엽니다.
1. **애플리케이션 설정** 창에서 **애플리케이션 설정**까지 아래로 스크롤합니다.
1. **botFileSecret**을 찾아 삭제합니다.
1. 리포지토리에 체크 인한 파일과 일치하도록 봇 파일의 이름을 업데이트합니다.
1. 변경 내용을 저장합니다.


## <a name="deploy-using-azure-deployment-center"></a>Azure 배포 센터를 사용하여 배포
이제 Git 리포지토리를 Azure App Services에 연결해야 합니다. [지속적인 배포 설정](https://docs.microsoft.com/en-us/azure/app-service/deploy-continuous-deployment) 항목의 지침을 따릅니다. `App Service Kudu build server`를 사용하여 빌드하는 것이 좋습니다.

## <a name="test-your-deployment"></a>배포 테스트
배포가 성공하면 몇 초 동안 기다린 후 필요에 따라 웹앱을 다시 시작하여 캐시를 지웁니다. 웹앱 봇 블레이드로 돌아가서 Azure Portal에 제공된 웹 채팅을 사용하여 테스트합니다.

## <a name="additional-resources"></a>추가 리소스
- [연속 배포의 일반 문제를 조사하는 방법](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)

<!--

## Prerequisites

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## Deploy JavaScript and C# bots using az cli

You've already created and tested a bot locally, and now you want to deploy it to Azure. These steps assume that you have created the required Azure resources.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### Create a Web App Bot

If you don't already have a resource group to which to publish your bot, create one:

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

Before proceeding, read the instructions that apply to you based on the type of email account you use to log in to Azure.

#### MSA email account

If you are using an [MSA](https://en.wikipedia.org/wiki/Microsoft_account) email account, you will need to create the app ID and app password on the Application Registration Portal to use with `az bot create` command.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### Business or school account

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### Download the bot from Azure

Next, download the bot you just created. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

### Decrypt the downloaded .bot file and use in your project

The sensitive information in the .bot file is encrypted.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### Update the .bot file

If your bot uses LUIS, QnA Maker, or Dispatch services, you will need to add references to them to your .bot file. Otherwise, you can skip this step.

1. Open your bot in the BotFramework Emulator, using the new .bot file. The bot does not need to be running locally.
1. In the **BOT EXPLORER** panel, expand the **SERVICES** section.
1. To add references to LUIS apps, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Language Understanding (LUIS)**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of LUIS applications you have access to. Select the ones for your bot.
1. To add references to a QnA Maker knowledge base, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add QnA Maker**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of knowledge bases you have access to. Select the ones for your bot.
1. To add references to Dispatch models, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Dispatch**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of Dispatch models you have access to. Select the ones for your bot.

### Test your bot locally

At this point, your bot should work the same way it did with the old .bot file. Make sure that it works as expected with the new .bot file.

### Publish your bot to Azure

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]


[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## Additional resources

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## Next steps
> [!div class="nextstepaction"]
> [Set up continous deployment](bot-service-build-continuous-deployment.md)

-->
