---
title: 엔터프라이즈 봇 템플릿 배포 | Microsoft Docs
description: 엔터프라이즈 봇에 지원되는 모든 Azure 리소스를 배포하는 방법을 알아봅니다.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a4ac88872e11cd32a9de96d52dcf9e917bb3488f
ms.sourcegitcommit: 87b5b0ca9b0d5e028ece9f7cc4948c5507062c2b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/24/2018
ms.locfileid: "47029801"
---
# <a name="enterprise-bot-template---deploying-your-bot"></a>엔터프라이즈 봇 템플릿 - 봇 배포

> [!NOTE]
> 이 항목은 SDK v4 버전에 적용됩니다. 

## <a name="prerequisites"></a>필수 조건

- [노드 패키지 관리자](https://nodejs.org/en/)가 설치되어 있는지 확인합니다.

- Azure Bot Service 명령줄(CLI) 도구를 설치합니다. 최신 버전이 있는지 확인하기 전에 도구를 사용한 경우에도 이 작업을 수행해야 합니다.

```shell
npm install -g ludown luis-apis qnamaker botdispatch msbot luisgen chatdown
```

- [여기](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest)에서 Azure 명령줄(CLI) 도구를 설치합니다.

- Bot Service용 AZ 확장을 설치합니다.
```shell
az extension add -n botservice
```

## <a name="configuration"></a>구성

- LUIS 제작 키 검색
   - [이 설명서](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions) 페이지에서 배포하려는 지역에 대한 올바른 LUIS 포털을 검토합니다. 
   - 로그인한 후에 오른쪽 위 모서리에서 이름을 클릭합니다.
   - 설정을 선택하고, 다음 단계를 위해 제작 키를 적어 둡니다.

## <a name="deployment"></a>배포

>Azure 구독이 여러 개이고 배포에서 올바른 Azure 구독을 선택하려면 계속하기 전에 다음 명령을 실행합니다.

 Azure 계정에 대한 브라우저 로그인 프로세스를 수행합니다.
```shell
az login
az account list
az account set --subscription "YOUR_SUBSCRIPTION_NAME"
```

엔터프라이즈 템플릿 봇에 필요한 종단 간 작업에 대한 종속성은 다음과 같습니다.
- Azure 웹앱
- Azure Storage 계정(대화 내용)
- Azure Application Insights(원격 분석)
- Azure Cosmos DB(상태)
- Azure Cognitive Services - Language Understanding
- Azure Cognitive Services - QnAMaker(Azure Search, Azure Web App 포함)
- Azure Cognitive Services - Content Moderator(선택적 수동 단계)

새 봇 프로젝트에는 `msbot clone services` 명령을 사용하여 위의 모든 서비스를 Azure 구독에 자동으로 배포하고, 프로젝트의 .bot 파일이 봇에서 작업을 원활하게 수행할 수 있게 하는 키를 포함하여 모든 서비스로 업데이트되도록 하는 배포 방법이 있습니다.

> 배포된 후에는 만든 서비스에 대한 가격 책정 계층을 검토하고 시나리오에 맞게 조정합니다.

만든 프로젝트 내의 README.md에는 만든 봇 이름으로 업데이트된 msbot 복제 서비스 명령줄 예제가 포함되어 있으며, 일반 버전이 아래에 나와 있습니다. 이전 단계에서 제작 키를 업데이트하고 사용하려는 Azure 데이터 센터 위치(예: westus 또는 westeurope)를 선택해야 합니다.

> 이전 단계에서 검색한 LUIS 제작 키가 아래에서 지정한 지역에 해당하는지 확인합니다.

```shell
msbot clone services --name "YOUR_BOT_NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\msbotClone" --location "westus"
```

msbot 도구는 위치와 SKU를 포함한 배포 계획을 간략하게 설명합니다. 검토한 후에 계속 진행하세요.

![배포 확인](./media/enterprise-template/EnterpriseBot-ConfirmDeployment.png)

>배포가 완료되면 나중의 단계에서 필요하므로 제공된 .bot 파일 비밀을 **반드시** 적어 둡니다.

- `appsettings.json` 파일을 새로 만든 .bot 파일 이름과 .bot 파일 비밀로 업데이트합니다.
- 다음 명령을 실행하고, Application Insights 인스턴스에 대한 InstrumentationKey를 검색하고, `appsettings.json` 파일의 InstrumentationKey를 업데이트합니다.

`msbot list --bot YOURBOTFILE.bot --secret YOUR_BOT_SECRET`

        {
          "botFilePath": ".\\YOURBOTFILE.bot",
          "botFileSecret": "YOUR_BOT_SECRET",
          "ApplicationInsights": {
            "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
          }
        }

## <a name="testing"></a>테스트

완료되면 개발 환경 내에서 봇 프로젝트를 실행하고 Bot Framework 에뮬레이터를 엽니다. 에뮬레이터의 [파일] 메뉴에서 [봇 열기]를 선택하고, 디렉터리의 .bot 파일로 이동합니다.

그런 다음, ```hi```을 입력하여 모든 항목이 작동하는지 확인합니다.

## <a name="deploy-to-azure"></a>Deploy to Azure

종단 간 테스트를 로컬로 수행할 수 있습니다. 봇을 Azure에 배포하여 추가 테스트를 수행할 준비가 되면 다음 명령을 사용하여 소스 코드를 게시할 수 있으며, 소스 코드 업데이트를 푸시하려고 할 때마다 이를 실행할 수 있습니다.

```shell
az bot publish -g YOUR_BOT_NAME -n YOUR_BOT_NAME --proj-file YOUR_BOT_NAME.csproj --sdk-version v4
```

## <a name="enabling-more-scenarios"></a>추가 시나리오 사용

봇 프로젝트는 다음 단계를 통해 사용하도록 설정할 수 있는 추가 기능을 제공합니다.

### <a name="authentication"></a>인증

인증을 사용하도록 설정하려면 Azure Portal의 봇 설정 내에 [인증 연결 이름]을 구성한 후 다음 단계를 수행합니다. 자세한 내용은 [이 설명서](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0)에 있습니다.

MainDialog 생성자에서 `SignInDialog`를 등록합니다.
    
`AddDialog(new SignInDialog(_services.AuthConnectionName));`

코드에서 원하는 위치에 다음을 추가하여 간단한 로그인 흐름을 테스트합니다.
    
`var signInResult = await dc.BeginAsync(SignInDialog.Name);`

### <a name="content-moderation"></a>콘텐츠 조정

콘텐츠 조정은 봇에 보낸 메시지에서 PII(개인 식별 정보) 및 성인용 콘텐츠를 식별하는 데 사용할 수 있습니다. 이 기능을 사용하도록 설정하려면 Azure Portal로 이동하여 새 콘텐츠 조정자 서비스를 만듭니다. 구독 키와 지역을 수집하여 .bot 파일을 구성합니다. 

> 이 단계는 앞으로 자동화됩니다.

시작할 때마다 콘텐츠 조정을 사용하도록 설정하려면 시작 시 service.AddBot<>() 메서드의 아래쪽에 다음 코드를 추가합니다. 콘텐츠 조정의 결과는 봇 상태를 통해 액세스할 수 있습니다. 
    
```
    // Content Moderation Middleware (analyzes incoming messages for inappropriate content including PII, profanity, etc.)
    var moderatorService = botConfig.Services.Where(s => s.Name == ContentModeratorMiddleware.ServiceName).FirstOrDefault();
    if (moderatorService != null)
    {
        var moderator = moderatorService as GenericService;
        var moderatorKey = moderator.Configuration["subscriptionKey"];
        var moderatorRegion = moderator.Configuration["region"];
        var moderatorMiddleware = new ContentModeratorMiddleware(moderatorKey, moderatorRegion);
        options.Middleware.Add(moderatorMiddleware);
    }
```
대화 스택 내에서 이 조정을 호출하여 미들웨어 결과에 액세스합니다.
```     
    var cm = dc.Context.TurnState.Get<Microsoft.CognitiveServices.ContentModerator.Models.Screen>(ContentModeratorMiddleware.TextModeratorResultKey);
```

## <a name="customize-your-bot"></a>봇 사용자 지정

즉시 사용할 수 있는 봇을 성공적으로 배포했는지 확인한 후 시나리오 및 필요에 맞게 봇을 사용자 지정할 수 있습니다. [봇 사용자 지정](bot-builder-enterprise-template-customize.md)으로 계속 진행하세요.