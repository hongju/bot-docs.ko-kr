---
title: 엔터프라이즈 봇 템플릿 배포 | Microsoft Docs
description: 엔터프라이즈 봇에 지원되는 모든 Azure 리소스를 배포하는 방법을 알아봅니다.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e888b2473269cf576fd9edda0d99a30aa6212f7b
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/07/2019
ms.locfileid: "57571880"
---
# <a name="enterprise-bot-template---getting-started"></a>Enterprise Bot Template - 시작

> [!NOTE]
> 이 토픽은 SDK v4 버전에 적용됩니다. 

## <a name="prerequisites"></a>필수 조건

다음을 설치합니다.
- [Enterprise Bot Template VSIX](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4enterprise)
- [.NET Core SDK](https://www.microsoft.com/net/download)(최신 버전)
- [Node 패키지 관리자](https://nodejs.org/en/)
- [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0)(최신 버전)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Bot Service CLI 도구](https://github.com/Microsoft/botbuilder-tools)(최신 버전)
    ```shell
    npm install -g ludown luis-apis qnamaker botdispatch msbot chatdown
    ```
- [LuisGen](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/LUISGen/src/npm/readme.md)
    ```shell
    dotnet tool install -g luisgen
    ```

## <a name="create-your-bot-project"></a>봇 프로젝트 만들기
1. Visual Studio에서 **파일 > 새 프로젝트**를 차례로 클릭합니다.
1. **봇** 아래에서 **Enterprise Bot Template**을 선택합니다.

![새 프로젝트 템플릿 파일](media/enterprise-template/new_project.jpg)

1. 프로젝트 이름을 지정하고 **만들기**를 클릭합니다.
1. 프로젝트를 마우스 오른쪽 단추로 클릭하고, **빌드**를 클릭하여 NuGet 패키지를 복원합니다.

## <a name="deploy-your-bot"></a>봇 배포

엔터프라이즈 템플릿 봇에 필요한 엔드투엔드 작업용 Azure 서비스는 다음과 같습니다.
- Azure 웹앱
- Azure Storage 계정(대화 내용)
- Azure Application Insights(원격 분석)
- Azure CosmosDb(대화 상태 스토리지)
- Azure Cognitive Services - Language Understanding
- Azure Cognitive Services - QnA Maker(Azure Search, Azure Web App 포함)

제공된 배포 스크립트를 사용하여 이러한 서비스를 배포하는 데 도움이 되는 단계는 다음과 같습니다.

1. LUIS 작성 키를 검색합니다.
   - [이 설명서](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions) 페이지에서 배포 지역에 적합한 LUIS 포털을 검토합니다. [www.luis.ai](www.luis.ai)는 미국 지역을 참조하며 이 포털에서 검색된 작성 키는 유럽 배포에서 작동하지 않습니다.
   - 로그인한 후에 오른쪽 위 모서리에서 이름을 클릭합니다.
   - **설정**을 선택하고, 나중에 사용할 수 있도록 작성 키를 적어 둡니다.
    
    ![LUIS 작성 키 스크린샷](./media/enterprise-template/luis_authoring_key.jpg)

1. 명령 프롬프트를 엽니다.
1. Azure CLI를 사용하여 Azure 계정에 로그인합니다. 액세스 권한이 있는 구독 목록은 Azure Portal [구독](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) 페이지에서 찾을 수 있습니다.
    ```shell
    az login
    az account set --subscription "YOUR_SUBSCRIPTION_NAME"
    ```

1. msbot clone services 명령을 실행하여 서비스를 배포하고 프로젝트에서 .bot 파일을 구성합니다. **참고: 배포가 완료되면 나중에 사용할 수 있도록 명령 프롬프트 창에 표시된 봇 파일 비밀을 적어 두어야 합니다.**

    ```shell
    msbot clone services --name "YOUR-BOT-NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\LOCALE_FOLDER" --location "REGION"
    ```

    > **참고**:
    >- **YOUR-BOT-NAME** 매개 변수는 **글로벌로 고유**해야 하며, 소문자, 숫자 및 대시("-")만 포함할 수 있습니다.
    >- 이 단계에서 제공하는 배포 지역이 LUIS 작성 키 포털의 지역과 일치하는지 확인합니다(예: luis.ai의 경우 westus 또는 eu.luis.ai의 경우 westeurope).
    >- 일부 사용자가 MSA AppId 및 암호를 프로비저닝할 때 발생할 수 있는 알려진 문제가 있습니다. `ERROR: Unable to provision MSA id automatically. Please pass them in as parameters and try again` 오류가 표시되면 [https://apps.dev.microsoft.com](https://apps.dev.microsoft.com)으로 이동하여 새 애플리케이션을 수동으로 만들고 AppId 및 암호/비밀을 적어 둡니다. 위의 msbot clone services 명령을 다시 실행하여 방금 검색한 값이 있는 두 개의 새 인수(`--appId` 및 `--appSecret`)를 제공합니다. 셸에서 명령으로 해석할 수 있는 암호에서 특수 문자를 이스케이프해야 할 수 있습니다.
    >   - *Windows 명령 프롬프트*의 경우 appSecret을 큰따옴표로 묶습니다. 예: `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret "YOUR_APP_SECRET"`
    >   - *Windows PowerShell의 경우 --% 인수 뒤에 appSecret을 전달합니다. 예: `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --% --appSecret "YOUR_APP_SECRET"`
    >   - *MacOS 또는 Linux*의 경우 appSecret을 작은따옴표로 묶습니다. 예: `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret 'YOUR_APP_SECRET'`

1. 배포가 완료되면 봇 파일 비밀을 사용하여 **appsettings.json**을 업데이트합니다. 
    
    ```
    "botFilePath": "./YOUR_BOT_FILE.bot",
    "botFileSecret": "YOUR_BOT_SECRET",
    ```
1. 다음 명령을 실행하고, Application Insights 인스턴스에 대한 InstrumentationKey를 검색합니다.
    ```
    msbot list --bot YOUR_BOT_FILE.bot --secret "YOUR_BOT_SECRET"
    ```

1. 계측 키를 사용하여 **appdetettings.json**을 업데이트합니다.

    ```
    "ApplicationInsights": {
        "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
    }
    ```

## <a name="test-your-bot"></a>봇 테스트

완료되면 개발 환경 내에서 봇 프로젝트를 실행하고, **Bot Framework Emulator**를 엽니다. Emulator 내에서 **파일 > 봇 열기**를 차례로 클릭하고, 디렉터리의 .bot 파일로 이동합니다.

![Emulator 개발 엔드포인트 스크린샷](./media/enterprise-template/development_endpoint.jpg)

대화가 시작되면 소개 메시지를 받아야 합니다.

```hi```를 입력하여 모든 항목이 작동하는지 확인합니다.

## <a name="publish-your-bot"></a>봇 게시

테스트는 엔드투엔드 로컬로 수행할 수 있습니다. 추가 테스트를 위해 봇을 Azure에 배포할 준비가 되면 다음 명령을 사용하여 소스 코드를 게시할 수 있습니다.

```shell
az bot publish -g YOUR-BOT-NAME -n YOUR-BOT-NAME --proj-name YOUR-BOT-NAME.csproj --version v4
```

## <a name="view-your-bot-analytics"></a>봇 분석 보기
Enterprise Bot Template에는 Application Insights 서비스에 연결하여 대화 분석을 제공하는 미리 구성된 Power BI 대시보드가 포함되어 있습니다. 봇이 로컬로 테스트되면 이 대시보드를 열어 데이터를 볼 수 있습니다. 

1. [여기](https://github.com/Microsoft/AI/blob/master/solutions/analytics/ConversationalAnalyticsSample_02132019.pbit)에서 Power BI 대시보드(.pbit 파일)를 다운로드합니다.
1. [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/)에서 대시보드를 엽니다.
1. Application Insights 애플리케이션 ID(.bot 파일에 있음)를 입력합니다.

    ![봇 파일에서 AppInsights appId를 찾을 수 있는 위치](./media/enterprise-template/appInsights_appId.jpg)

1. Power BI 대시보드 기능에 대한 자세한 내용은 [여기](https://github.com/Microsoft/AI/tree/master/solutions/analytics)를 참조하세요.

# <a name="next-steps"></a>다음 단계

즉시 사용할 수 있는 봇이 성공적으로 배포되면 봇을 시나리오 및 요구 사항에 맞게 사용자 지정할 수 있습니다. [봇 사용자 지정](bot-builder-enterprise-template-customize.md)으로 계속 진행하세요.
