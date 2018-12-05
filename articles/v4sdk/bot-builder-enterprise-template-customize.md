---
title: 엔터프라이즈 봇 사용자 지정 | Microsoft Docs
description: 엔터프라이즈 템플릿에서 만든 봇을 사용자 지정하는 방법을 알아봅니다.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 319700f8b7b236ce74058bac5fabb84f21e04d69
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/28/2018
ms.locfileid: "52452015"
---
# <a name="enterprise-bot-template---customize-your-bot"></a>엔터프라이즈 봇 - 봇 사용자 지정

> [!NOTE]
> 이 토픽은 SDK v4 버전에 적용됩니다. 

## <a name="net"></a>.NET
엔터프라이즈 봇 템플릿이 [여기](bot-builder-enterprise-template-deployment.md)의 지침에서 나열한 대로 종단 간에 작동하도록 배포 및 테스트한 후에는 시나리오 및 필요에 따라 봇을 쉽게 사용자 지정할 수 있습니다. 템플릿의 목표는 대화형 환경을 구축할 수 있는 견고한 기반을 제공하는 것입니다.

## <a name="project-structure"></a>프로젝트 구조

봇의 폴더 구조는 아래에 나와 있으며, 봇 프로젝트를 구조화하고 들어오는 메시지를 처리하는 데 권장되는 모범 사례를 나타냅니다.

    | - YourBot.bot         // The .bot file containing all of your Bot configuration including dependencies
    | - README.md           // README file containing links to documentation
    | - Program.cs          // Default Program.cs file
    | - Startup.cs          // Core Bot Initialisation including Bot Configuration LUIS, Dispatcher, etc. 
    | - <BOTNAME>State.cs   // The Root State class for your Bot
    | - appsettings.json    // References above .bot file for Configuration information. App Insights key
    | - CognitiveModels     
        | - LUIS            // .LU file containing base conversational intents (Greeting, Help, Cancel)
        | - QnA             // .LU file containing example QnA items
    | - DeploymentScripts   // msbot clone recipes for deployment
        | - de              // Deployment files for German
        | - en              // Deployment files for English        
        | - es              // Deployment files for Spanish
        | - fr              // Deployment files for French
        | - it              // Deployment files for Italian
        | - zh              // Deployment files for Chinese
    | - Dialogs             // All Bot dialogs sit under this folder
        | - Main            // Root Dialog for all messages
            | - MainDialog.cs       // Dialog Logic
            | - MainResponses.cs    // Dialog responses
            | - Resources           // Adaptive Card JSON, Resource File
        | - Onboarding
            | - OnboardingDialog.cs       // Onboarding dialog Logic
            | - OnboardingResponses.cs    // Onboarding dialog responses
            | - OnboardingState.cs        // Localised dialog state
            | - Resources                 Resource File
        | - Cancel
        | - Escalate
        | - Signin
    | - Middleware          // Telemetry, Content Moderator
    | - ServiceClients      // SDK libraries, example GraphClient provided for Auth example
   
## <a name="update-introduction-message"></a>소개 메시지 업데이트

소개 메시지는 [적응형 카드](https://www.adaptivecards.io)를 사용합니다. 봇에 대한 이러한 소개를 사용자 지정하려면 Dialogs/Main/Resources 폴더에서 ```Intro.json```이라는 JSON 파일을 찾습니다. [적응형 카드 시각화 도우미](http://adaptivecards.io/visualizer)를 사용하여 봇의 요구 사항에 맞게 적응형 카드를 수정합니다.

## <a name="update-bot-responses"></a>봇 응답 업데이트

프로젝트 내의 각 대화에는 지원 리소스(.resx) 파일 내에 저장되는 일단의 응답이 있습니다. 이러한 응답은 각 대화의 Resources 폴더 아래에서 찾을 수 있습니다.

아래와 같이 Visual Studio 리소스 편집기에서 응답을 변경하여 봇이 응답하는 방식을 조정할 수 있습니다.

![봇 응답 사용자 지정](media/enterprise-template/EnterpriseBot-CustomisingResponses.png)

이 방법은 표준 리소스 파일 지역화 방법을 사용하여 다국어 응답을 지원합니다. 자세한 내용은 [여기](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization?view=aspnetcore-2.1)에 있습니다.

## <a name="updating-your-cognitive-models"></a>인지 모델 업데이트

엔터프라이즈 템플릿에는 기본적으로 두 가지 인지 모델, 즉 FAQ QnA Maker 기술 자료 샘플 및 일반 의도(인사말, 도움말, 취소 등)에 대한 LUIS 모델이 포함되어 있습니다. 이러한 모델은 사용자의 필요에 맞게 사용자 지정할 수 있습니다. 또한 새 LUIS 모델과 QnA Maker 기술 자료를 추가하여 봇의 기능을 확장할 수도 있습니다.

### <a name="updating-an-existing-luis-model"></a>기존 LUIS 모델 업데이트
엔터프라이즈 템플릿에 대한 기존 LUIS 모델을 업데이트하려면 다음 단계를 수행합니다.
1. [LUIS 포털](http://luis.ai)에서 LUIS 모델을 변경하거나 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 및 [Luis](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI 도구를 사용하여 변경합니다. 
2. 다음 명령을 실행하여 변경 내용을 반영하도록 디스패치 모델을 업데이트합니다(적절한 메시지 라우팅 보장).
```shell
    dispatch refresh --bot "YOUR_BOT.bot" --secret YOUR_SECRET
```
3. 업데이트된 각 모델에 대한 프로젝트 루트에서 다음 명령을 실행하여 연결된 해당 LuisGen 클래스를 업데이트합니다. 
```shell
    luis export version --appId [LUIS_APP_ID] --versionId [LUIS_APP_VERSION] --authoringKey [YOUR_LUIS_AUTHORING_KEY] | luisgen --cs [CS_FILE_NAME] -o "\Dialogs\Shared\Resources"
```

### <a name="updating-an-existing-qna-maker-knowledge-base"></a>기존 QnA Maker 기술 자료 업데이트
기존 QnA Maker 기술 자료를 업데이트하려면 다음 단계를 수행합니다.
1. [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 및 [QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI 도구 또는 [QnA Maker 포털](https://qnamaker.ai)을 통해 QnA Maker 기술 자료를 변경합니다.
2. 다음 명령을 실행하여 변경 내용을 반영하도록 디스패치 모델을 업데이트합니다(적절한 메시지 라우팅 보장).
```shell
    dispatch refresh --bot "YOUR_BOT.bot" --secret YOUR_SECRET
```

### <a name="adding-a-new-luis-model"></a>새 LUIS 모델 추가

프로젝트에 새 LUIS 모델을 추가하려는 시나리오에서 추가 모델을 인식할 수 있도록 봇 구성 및 디스패처를 업데이트해야 합니다. 
1. LuDown/LUIS CLI 도구 또는 LUIS 포털을 통해 LUIS 모델을 만듭니다.
2. 다음 명령을 실행하여 새 LUIS 앱을 .bot 파일에 연결합니다.
```shell
    msbot connect luis --appId [LUIS_APP_ID] --authoringKey [LUIS_AUTHORING_KEY] --subscriptionKey [LUIS_SUBSCRIPTION_KEY] 
```
3. 다음 명령을 사용하여 이 새 LUIS 모델을 디스패처에 추가합니다
```shell
    dispatch add -t luis -id LUIS_APP_ID -bot "YOUR_BOT.bot" -secret YOURSECRET
```
4. 다음 명령을 통해 LUIS 모델 변경 내용을 반영하도록 디스패치 모델을 새로 고칩니다.
```shell
    dispatch refresh -bot "YOUR_BOT.bot" -secret YOUR_SECRET
```

### <a name="adding-an-additional-qna-maker-knowledge-base"></a>추가 QnA Maker 기술 자료 추가

추가 QnA Maker 기술 자료를 봇에 추가하려는 일부 시나리오에서 이렇게 하려면 다음 단계를 수행하면 됩니다.

1. 도우미 디렉터리에서 실행된 다음 명령을 사용하여 JSON 파일에서 새 QnA Maker 기술 자료를 만듭니다.
```shell
qnamaker create kb --in <KB.json> --msbot | msbot connect qna --stdin --bot "YOUR_BOT.bot" --secret YOURSECRET
```
2. 다음 명령을 실행하여 변경 내용을 반영하도록 디스패치 모델을 업데이트합니다.
```shell
dispatch refresh --bot "YOUR_BOT.bot" --secret YOUR_SECRET
```
3. 새 QnA 원본을 반영하는 강력한 형식의 디스패치 클래스를 업데이트합니다.
```shell
msbot get dispatch --bot "YOUR_BOT.bot" | luis export version --stdin > dispatch.json
luisgen dispatch.json -cs Dispatch -o Dialogs\Shared
```
4.  제공된 예제에 따라 새 QnA 원본에 대해 해당하는 디스패치 의도를 포함하도록 `Dialogs\Main\MainDialog.cs` 파일을 업데이트합니다.

이제 여러 QnA 원본을 봇의 일부로 활용할 수 있어야 합니다.

## <a name="adding-a-new-dialog"></a>새 대화 추가

봇에 새 대화를 추가하려면 먼저 Dialogs 아래에 새 폴더를 만들고 이 클래스가 `EnterpriseDialog`에서 파생되는지 확인해야 합니다. 그런 다음, 대화 인프라를 연결해야 합니다. [온보딩] 대화 상자에는 참조할 수 있는 간단한 예제가 나와 있으며, 아래에 단계의 개요와 함께 발췌 부분이 표시됩니다.

- 생성자에 폭포 대화 추가
- 폭포 단계 정의
- 폭포 단계 만들기
- 폭포를 전달하는 AddDialog 호출
- 폭포에서 사용할 프롬프트를 전달하는 AddDialog 호출
- InitialDialogId를 구성 요소가 실행되도록 할 첫 번째 대화로 설정합니다.

```
    InitialDialogId = nameof(OnboardingDialog);

    var onboarding = new WaterfallStep[]
    {
        AskForName,
        AskForEmail,
        AskForLocation,
        FinishOnboardingDialog,
    };

    AddDialog(new WaterfallDialog(InitialDialogId, onboarding));
    AddDialog(new TextPrompt(DialogIds.NamePrompt));
    AddDialog(new TextPrompt(DialogIds.EmailPrompt));
    AddDialog(new TextPrompt(DialogIds.LocationPrompt));
```

그런 다음, 응답을 처리할 템플릿 관리자를 만들어야 합니다. 새 클래스를 만들고, TemplateManager에서 파생시킵니다. 예제는 OnboardingResponses.cs 파일에서 제공되며 발췌 부분은 다음과 같습니다.

```    
 ["default"] = new TemplateIdMap
            {
                { ResponseIds.EmailPrompt,
                    (context, data) =>
                    MessageFactory.Text(
                        text: OnboardingStrings.EMAIL_PROMPT,
                        ssml: OnboardingStrings.EMAIL_PROMPT,
                        inputHint: InputHints.ExpectingInput)
                },
                { ResponseIds.HaveEmailMessage,
                    (context, data) =>
                    MessageFactory.Text(
                        text: string.Format(OnboardingStrings.HAVE_EMAIL, data.email),
                        ssml: string.Format(OnboardingStrings.HAVE_EMAIL, data.email),
                        inputHint: InputHints.IgnoringInput)
                },
                { ResponseIds.HaveLocationMessage,
                    (context, data) =>
                    MessageFactory.Text(
                        text: string.Format(OnboardingStrings.HAVE_LOCATION, data.Name, data.Location),
                        ssml: string.Format(OnboardingStrings.HAVE_LOCATION, data.Name, data.Location),
                        inputHint: InputHints.IgnoringInput)
                },
                
                ...
```

응답을 렌더링하려면 템플릿 관리자 인스턴스를 사용하여 프롬프트에 대한 `ReplyWith` 또는 `RenderTemplate`를 통해 이러한 응답에 액세스할 수 있습니다. 예제는 다음과 같습니다.

```
Prompt = await _responder.RenderTemplate(sc.Context, sc.Context.Activity.Locale, OnboardingResponses.ResponseIds.NamePrompt)
await _responder.ReplyWith(sc.Context, OnboardingResponses.ResponseIds.HaveNameMessage, new { name });
```

대화 인프라의 마지막 부분은 대화에만 적용되는 State 클래스를 만드는 것입니다. 새 클래스를 만들고 `DialogState`에서 파생되는지 확인합니다.

대화가 완료되면 `AddDialog`를 사용하여 `MainDialog` 구성 요소에 대화를 추가해야 합니다. 새 대화를 사용하려면 `RouteAsync` 메서드 내에서 `dc.BeginDialogAsync()`를 호출하고, 원하는 경우 적절한 LUIS 의도를 사용하여 트리거합니다.

## <a name="conversational-insights-using-powerbi-dashboard-and-application-insights"></a>PowerBI 대시보드 및 Application Insights를 사용하는 대화형 인사이트
- 대화형 인사이트를 가져오려면 [PowerBI 대시보드를 사용하여 대화형 분석 구성](bot-builder-enterprise-template-powerbi.md)을 계속합니다.
