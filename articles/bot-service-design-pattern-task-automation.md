---
title: 태스크 자동화 봇 만들기 | Microsoft Docs
description: 사용자의 추가 개입 없이 태스크를 수행하는 봇을 디자인하는 방법을 알아봅니다.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 2/13/2018
ms.openlocfilehash: c14814dc7e2c83f740202db90b7e41efcdfb66a5
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224408"
---
# <a name="create-task-automation-bots"></a>태스크 자동화 봇 만들기

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

태스크 자동화 봇을 사용하면 사람의 도움 없이 특정 태스크이나 태스크 집합을 완료할 수 있습니다. 이러한 유형의 봇은 일반적인 앱 또는 웹 사이트와 비슷한 경우가 많으며 주로 풍부한 사용자 컨트롤 및 텍스트를 통해 사용자와 소통합니다. 사용자와의 대화를 보강하기 위해 자연어 이해 기능을 포함할 수도 있습니다. 

## <a name="example-use-case-password-reset"></a>사용 사례 예제: 암호 재설정

태스크 봇의 특성을 더 잘 이해하기 위해서는 사용 사례 예제: 암호 재설정을 살펴보세요. 

Contoso 회사는 매일 직원으로부터 암호를 재설정해야 한다는 몇 번의 지원 센터 전화 문의를 받고 있습니다. Contoso는 지원 센터 상담원이 좀 더 복잡한 문제를 해결하는 데 시간을 투입할 수 있도록 직원 암호를 재설정하는 간단하면서 반복적인 태스크를 자동화하려고 합니다. 

Contoso의 숙련된 개발자인 John은 암호 재설정 태스크를 자동화하는 봇을 만들기로 결정했습니다. 새 앱 또는 웹 사이트를 만들 때와 같이, 먼저 봇의 디자인 사양을 작성했습니다. 

::: moniker range="azure-bot-service-3.0"

### <a name="navigation-model"></a>탐색 모델

해당 사양에는 탐색 모델 정의가 포함됩니다.

![다이얼로그 구조](~/media/bot-service-design-pattern-task-automation/simple-task1.png)

사용자는 `RootDialog`에서 시작합니다. 암호 재설정을 요청하면  
`ResetPasswordDialog`로 리디렉션됩니다. 봇은 `ResetPasswordDialog`를 사용하여 사용자에게 두 가지 정보인 전화번호와 생년월일을 묻는 메시지를 표시합니다. 

> [!IMPORTANT]
> 이 문서에 설명된 봇 디자인은 예제 목적으로만 제공됩니다. 실제 시나리오에서는 암호 재설정 봇이 보다 강력한 ID 확인 프로세스를 구현할 가능성이 높습니다.

### <a name="dialogs"></a>다이얼로그

다음으로, 사양에는 각 다이얼로그의 모양 및 각 기능이 설명됩니다. 

#### <a name="root-dialog"></a>루트 다이얼로그

루트 다이얼로그는 두 가지 옵션을 제공합니다. 

1. **암호 변경**은 사용자가 현재 암호를 알고 있고 단순히 이 암호를 변경하려고 하는 시나리오입니다.
2. **암호 재설정**은 사용자가 암호를 잊었거나 잃어버렸으므로 새 암호를 생성해야 하는 시나리오입니다.

> [!NOTE]
> 간단히 하기 위해 이 문서에서는 **암호 재설정** 흐름만 설명합니다.

다음 스크린샷에 표시된 대로 사양에서는 루트 다이얼로그에 대해 설명합니다.

![다이얼로그 구조](~/media/bot-service-design-pattern-task-automation/simple-task2.png)

#### <a name="resetpassword-dialog"></a>ResetPassword 다이얼로그

사용자가 루트 다이얼로그에서 **암호 재설정**을 선택하면 `ResetPassword` 다이얼로그가 호출됩니다. 
그러면 `ResetPassword` 다이얼로그는 두 개의 다른 다이얼로그를 호출합니다. 
먼저 사용자의 전화번호를 수집하는 `PromptStringRegex` 다이얼로그를 호출합니다. 
그런 후 사용자의 생년월일을 수집하는 `PromptDate` 다이얼로그를 호출합니다. 

> [!NOTE]
> 이 예제에서 John은 두 개의 별도 다이얼로그를 사용하여 사용자의 전화번호와 생년월일을 수집하는 논리를 구현하기로 했습니다. 이 접근 방식은 각 다이얼로그에 필요한 코드를 단순화할 뿐만 아니라 앞으로 이러한 다이얼로그가 다른 시나리오에서도 사용될 가능성을 높여줍니다. 

사양에는 `ResetPassword` 다이얼로그에 대해 설명됩니다.

![다이얼로그 구조](~/media/bot-service-design-pattern-task-automation/simple-task3.png)

#### <a name="promptstringregex-dialog"></a>PromptStringRegex 다이얼로그

`PromptStringRegex` 다이얼로그는 사용자에게 전화번호를 입력하라는 메시지를 표시하고 사용자가 제공하는 전화번호가 예상되는 형식과 일치하는지 확인합니다. 
또한 사용자가 잘못된 입력을 반복적으로 제공하는 시나리오도 고려합니다. 
사양에는 `PromptStringRegex` 다이얼로그가 설명됩니다.

![다이얼로그 구조](~/media/bot-service-design-pattern-task-automation/simple-task4.png)

### <a name="prototype"></a>원형 구축

마지막으로, 사양에서는 암호 재설정 태스크를 성공적으로 완료하기 위해 봇과 소통하는 사용자의 예제를 제공합니다.

![다이얼로그 구조](~/media/bot-service-design-pattern-task-automation/simple-task5.png)

::: moniker-end 

## <a name="bot-app-or-website"></a>봇, 앱 또는 웹 사이트?

태스크 자동화 봇이 앱이나 웹 사이트와 매우 유사하다면 왜 앱이나 웹 사이트를 빌드하지 않는 것인지 의문이 생길 수 있습니다. 특정 시나리오에 따라, 봇 대신 앱이나 웹 사이트를 빌드하는 것이 훨씬 더 적절할 수 있습니다. [Bot Framework 직접 회선 API][directLineAPI] 또는 <a href="https://aka.ms/BotFramework-WebChat" target="_blank">웹 채팅 컨트롤</a>을 사용하여 앱에 봇을 포함하도록 선택할 수도 있습니다. 앱 컨텍스트에서 봇을 구현하면 풍부한 앱 환경과 대화형 환경이라는 두 가지 장점을 한 곳에서 제공할 수 있습니다. 

그렇지만 대부분의 경우 앱이나 웹 사이트를 빌드하는 것이 봇을 빌드하는 것보다 훨씬 더 복잡하고 비용도 많이 들 수 있습니다. 앱 또는 웹 사이트는 종종 여러 클라이언트와 플랫폼을 지원해야 하고, 패키징 및 배포도 까다롭고 시간이 많이 소요되며, 앱을 다운로드하여 설치하는 사용자 환경이 편리하지 않은 경우도 많습니다. 이러한 이유로, 봇은 종종 당면한 문제를 해결할 수 있는 훨씬 더 간단한 방법을 제공할 수 있습니다. 

또한 봇은 쉽게 확장될 수 있는 유연성도 제공합니다. 예를 들어, 개발자는 암호 재설정 봇에 자연어 및 음성 기능을 추가하여 전화 통화를 통해 액세스하거나 문자 메시지 지원을 추가할 수도 있습니다. 회사는 건물 전체에 키오스크를 설치하고 해당 환경에 암호 재설정 봇을 포함할 수 있습니다.

::: moniker range="azure-bot-service-3.0"
<!-- TODO: SimpleTaskAutomation no longer exists
## Sample code

For a complete sample that shows how to implement simple task automation using the Bot Framework SDK for .NET, see the <a href="https://aka.ms/capability-SimpleTaskAutomation" target="_blank">Simple Task Automation sample</a> in GitHub.

For a complete sample that shows how to implement simple task automation using the Bot Framework SDK for Node.js, see the <a href="https://aka.ms/capability-SimpleTaskAutomation" target="_blank">Simple Task Automation sample</a> in GitHub.
-->

## <a name="additional-resources"></a>추가 리소스

- [다이얼로그](~/dotnet/bot-builder-dotnet-dialogs.md)
- [다이얼로그를 사용하여 대화 흐름 관리(.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [다이얼로그를 사용하여 대화 흐름 관리(Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)

::: moniker-end

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
