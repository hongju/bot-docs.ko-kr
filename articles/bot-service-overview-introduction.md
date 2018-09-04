---
title: Azure Bot Service 소개 | Microsoft Docs
description: 봇의 빌드, 연결, 테스트, 배포, 모니터링 및 관리를 위한 Bot Service에 대해 알아봅니다.
keywords: 개요, 소개, SDK, 개요
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/27/2018
ms.openlocfilehash: 47dadde9a294855e3c39c1cbd17635b2839b856d
ms.sourcegitcommit: d2e0a1c7da19afc1254bc689bb345dc1804484e2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/28/2018
ms.locfileid: "43117044"
---
::: moniker range="azure-bot-service-3.0"

# <a name="about-azure-bot-service"></a>Azure Bot Service 정보

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Azure Bot Service는 지능형 봇을 모두 한 번에 빌드, 테스트, 배포 및 관리할 수 있는 도구를 제공합니다. 개발자는 SDK에서 제공하는 확장 가능한 모듈식 프레임워크를 통해 템플릿을 활용하여 음성, 언어 이해, 질문 및 답변 등을 제공하는 봇을 만들 수 있습니다.  

## <a name="what-is-a-bot"></a>봇이란?
봇은 사용자가 텍스트, 그래픽(카드) 또는 음성을 사용하여 대화형 방식으로 상호 작용하는 앱입니다. 간단한 질문 및 답변 대화 상자일 수도 있고, 패턴 일치, 상태 추적 및 기존 비즈니스 서비스와 잘 통합된 인공 지능 기술을 사용하여 지능형 방식으로 사람들이 서비스와 상호 작용할 수 있도록 하는 정교한 봇일 수도 있습니다. 봇의 [사례 연구](https://azure.microsoft.com/services/bot-service/)를 확인해 보세요.  

## <a name="building-a-bot"></a>봇 빌드 
C# 또는 Node.js에서 봇을 만들 때 즐겨 찾는 개발 환경이나 명령줄 도구를 사용할지 선택할 수 있습니다. 시작하기 위해 봇을 빌드하는 데 사용할 수 있는 봇 개발의 다양한 단계에 대한 도구를 제공합니다.    

![봇 개요](media/bot-service-overview.png) 

## <a name="plan"></a>계획 
코드를 작성하기 전에 모범 사례에 대한 봇 [디자인 지침](bot-service-design-principles.md) 을 검토하고 봇에 필요한 요구 사항을 확인합니다. 간단한 봇을 만들거나 음성, 언어 이해, QnA 또는 다양한 출처에서 정보를 추출하여 지능형 응답을 제공하는 기능 등 더 복잡한 기능을 포함시킬 수 있습니다.  

> [!TIP] 
>
> 다음과 같이 필요한 템플릿을 설치합니다.
>  - Bot Builder .NET SDK v3 [VSIX 템플릿](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3) 
>  - Bot Builder Node.js SDK v3 [Yeoman 템플릿](https://www.npmjs.com/package/generator-botbuilder) 
>
> 다음과 같은 도구를 설치합니다.
> - 봇 자산을 만들고 관리하는 [CLI 도구](https://github.com/Microsoft/botbuilder-tools)를 다운로드합니다. 이러한 도구는 명령줄에서 봇 구성 파일, LUIS 앱, QnA 기술 자료 등을 관리하는 데 유용합니다. 자세한 내용은 [추가 정보](https://github.com/Microsoft/botbuilder-tools/blob/master/README.md)에서 볼 수 있습니다.
> - 봇을 테스트하기 위한 [에뮬레이터](https://github.com/Microsoft/BotFramework-Emulator/releases)
>
> 필요한 경우 다음과 같은 봇 구성 요소를 사용합니다.  
> - 언어 이해를 봇에 추가하기 위한 [LUIS](https://www.luis.ai/)
> - 더 자연스러운 대화형 방식으로 사용자의 질문에 응답하기 위한 [QnA Maker](https://qnamaker.ai/)
> - 오디오를 텍스트로 변환하고, 의도를 이해하고, 텍스트를 다시 음성으로 변환하기 위한 [Speech](https://azure.microsoft.com/services/cognitive-services/speech/)  
> - 맞춤법 오류를 수정하고, 이름, 브랜드 이름 및 은어 간 차이점을 인식하기 위한 [맞춤법 검사](https://azure.microsoft.com/services/cognitive-services/spell-check/) 
> - 다양한 다른 지능형 구성 요소를 위한 [Cognitive Services](bot-service-concept-intelligence.md) 


## <a name="build-your-bot"></a>봇 빌드 
봇은 대화형 인터페이스를 구현하고 Bot Service와 통신하는 웹 서비스입니다. 이 솔루션은 여러 환경 및 언어에서 만들 수 있으며, Visual Studio 또는 Yeoman이나 Azure Portal 내에서 직접 손쉬운 시작 도구를 제공합니다. 사용할 수 있는 도구 및 서비스 중 일부가 아래에 나와 있습니다.

> [!TIP]
>
> [SDK](~/dotnet/bot-builder-dotnet-quickstart.md), [Azure Portal](bot-service-quickstart.md)을 사용하여 봇을 만들거나 [CLI 도구](~/bot-builder-create-templates.md)를 사용합니다.
>
> 구성 요소를 추가합니다. 
> - 언어 이해 모델 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home)를 추가합니다. 
> - 사용자 질문에 대답하기 위해 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home) 기술 자료를 추가합니다.  
> - 카드, 음성 또는 번역을 사용하여 사용자 환경을 개선합니다. 
> - Bot Builder SDK를 사용하여 봇에 논리를 추가합니다.   

## <a name="test-your-bot"></a>봇 테스트 
봇은 수많은 다양한 파트가 함께 작동하는 복잡한 앱입니다. 다른 복잡한 앱과 마찬가지로 흥미로운 버그가 발생하거나 봇이 예상과 다르게 동작할 수 있습니다. 게시하기 전에 봇을 테스트합니다.

> [!TIP] 
>
> - [에뮬레이터를 사용하여 봇 테스트](bot-service-debug-emulator.md)
> - [웹 채팅에서 봇 테스트](bot-service-manage-test-webchat.md)

## <a name="publish"></a>게시 
준비되면 봇을 Azure 또는 사용자 고유의 웹 서비스 또는 데이터 센터에 게시합니다. 봇을 로컬로 개발할 수 있도록 지속적인 배포를 설정할 수 있습니다. 이는 봇이 GitHub 또는 Visual Studio Team Services와 같은 소스 제어에 체크 인하는 경우에 유용합니다. 변경 내용을 소스 리포지토리로 다시 체크 인하면 변경 내용이 자동으로 Azure에 배포됩니다.

> [!Tip]
>
> - [Azure에 배포](bot-service-build-continuous-deployment.md)

## <a name="connect"></a>연결          
Facebook, Messenger, Kik, Skype, Slack, Microsoft Teams, Telegram, 텍스트/SMS, Twilio, Cortana 및 Skype와 같은 채널에 봇을 연결하여 상호 작용을 늘리고 더 많은 고객과 연결합니다.  
  
> [!TIP]
>
> - [추가할 채널 선택](bot-service-manage-channels.md)


## <a name="evaluate"></a>평가 
Azure Portal에서 수집된 데이터를 사용하여 봇의 성능과 기능을 향상시킬 기회를 확인합니다. 트래픽, 대기 시간 및 통합과 같은 서비스 수준 및 계측 데이터를 가져올 수 있습니다. 또한 분석은 사용자에 대한 대화 수준 보고, 메시지 및 채널 데이터를 제공합니다.

> [!Tip]
>
> - [분석 수집](bot-service-manage-analytics.md) 


::: moniker-end

::: moniker range="azure-bot-service-4.0"

# <a name="about-azure-bot-service"></a>Azure Bot Service 정보

[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure Bot Service는 지능형 봇을 모두 한 번에 빌드, 테스트, 배포 및 관리할 수 있는 도구를 제공합니다. 개발자는 SDK에서 제공하는 확장 가능한 모듈식 프레임워크를 통해 템플릿을 활용하여 음성 제공, 자연어 이해, 질문 및 답변 처리 등을 수행하는 봇을 만들 수 있습니다.  

## <a name="what-is-a-bot"></a>봇이란?

봇은 인간 사용자와 대화를 통해 통신할 수 있는 앱입니다. 소프트웨어는 텍스트, 음성, 그래픽 또는 메뉴를 통해 상호 작용하고 대화와 관련된 작업을 수행할 수 있습니다. 단순히 질문과 답변을 주고받거나 사람들이 자연스러운 방법으로 서비스와 상호 작용할 수 있게 해주는 정교한 봇일 수도 있습니다. 한편 봇은 기존 서비스와 잘 통합된 지능형 기술을 사용하고 있습니다.

봇은 시스템이 정보를 수집할 수 있게 해주거나, 컴퓨터처럼 느껴지기보다는 상호 작용을 하고 있다는 느낌을 더 많이 주는 사용자 환경을 제공합니다. 또한 저녁 예약을 받거나 사용자의 프로필 정보를 수집하는 직접적인 인간 상호 작용이 필요하지 않은 간단한 작업을 시스템(또는 다른 시스템과의 통합)에 전달합니다. 

## <a name="building-a-bot"></a>봇 빌드 

C#, JavaScript, Java 및 Python에서 봇을 만들 때 즐겨 찾는 개발 환경이나 명령줄 도구를 사용할지 선택할 수 있습니다. 시작하기 위해 봇을 빌드하는 데 사용할 수 있는 봇 개발의 다양한 단계에 대한 도구를 제공합니다.    

### <a name="design"></a>디자인

코드를 작성하기 전에 모범 사례에 대한 봇 [디자인 지침](bot-service-design-principles.md) 을 검토하고 봇에 필요한 요구 사항을 확인합니다. 간단한 봇을 만들거나 음성, 자연어 이해 또는 사용자의 질문에 답변하는 기능과 같은 보다 정교한 기능을 포함할 수 있습니다. 

### <a name="build"></a>빌드

봇은 대화형 인터페이스를 구현하고 Bot Service와 통신하는 웹 서비스입니다. 이 솔루션은 여러 환경 및 언어에서 만들 수 있으며, 쉽게 시작할 수 있게 해주는 위와 같은 템플릿을 제공합니다. [Azure Portal](bot-service-quickstart.md)에서 봇 개발을 시작하거나, 선택한 언어로 로컬 개발을 위해 아래에 나열된 템플릿을 사용할 수 있습니다.

| .NET 템플릿 | JavaScript 템플릿 | 
| --- | --- | 
| [VSIX 템플릿](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) | [Yeoman 템플릿](https://www.npmjs.com/package/generator-botbuilder). @preview를 사용하여 v4 템플릿을 가져옵니다. |


다음은 봇의 기능을 향상시키는 데 사용할 수 있는 추가 구성 요소입니다. 

| 기능 | 설명 | 링크 |
| --- | --- | --- |
| 자연어 처리 추가 | 봇이 자연어를 이해하고, 맞춤법 오류를 이해하고, 음성 기능을 사용하고, 사용자의 의도를 인식할 수 있게 해줍니다. | [LUIS 사용 방법](~/v4sdk/bot-builder-howto-v4-luis.md) 
| 질문에 답변 | 더 자연스럽고 대화하는 방식으로 사용자 질문에 답변하기 위해 기술 자료 추가 | [QnA Maker 사용 방법](~/v4sdk/bot-builder-howto-qna.md) 
| 여러 모델 관리 | LUIS 및 QnA Maker와 같은 둘 이상의 모델을 사용하는 경우, 봇 대화 중에 어느 모델을 사용해야 하는지 지능적으로 결정합니다. | [Dispatch 도구](~/v4sdk/bot-builder-tutorial-dispatch.md) |
| 카드 및 단추 추가 | 그래픽, 메뉴 및 카드와 같은 텍스트 이외의 다른 미디어를 사용하여 사용자 환경 향상 | [카드 추가 방법](v4sdk/bot-builder-howto-add-media-attachments.md) |

> [!NOTE]
> 위의 표는 포괄적인 목록은 아닙니다. 왼쪽 문서를 [메시지 보내기](~/v4sdk/bot-builder-howto-send-messages.md)부터 탐색하여 더 많은 봇 기능에 대해 알아봅니다.

또한 봇 자산을 생성, 관리 및 테스트할 수 있는 CLI 도구를 제공합니다. 이러한 도구는 명령줄에서 봇 구성 파일, LUIS 앱, QnA 기술 자료, 모의 대화 등을 관리할 수 있습니다. 자세한 내용은 [추가 정보](https://github.com/Microsoft/botbuilder-tools/blob/master/README.md)에서 볼 수 있습니다.

### <a name="test"></a>테스트 
봇은 수많은 다양한 파트가 함께 작동하는 복잡한 앱입니다. 다른 복잡한 앱과 마찬가지로 흥미로운 버그가 발생하거나 봇이 예상과 다르게 동작할 수 있습니다. 게시하기 전에 봇을 테스트합니다. 

[에뮬레이터를 사용하여 봇을 테스트](bot-service-debug-emulator.md)하거나 [웹 채팅에서 봇을 테스트](bot-service-manage-test-webchat.md)합니다.

### <a name="publish"></a>게시 
준비되면 [봇을 Azure](bot-builder-howto-deploy-azure.md) 또는 사용자 고유의 웹 서비스 또는 데이터 센터에 게시합니다.

### <a name="connect"></a>연결          
Facebook, Messenger, Kik, Skype, Slack, Microsoft Teams, Telegram, 텍스트/SMS, Twilio, Cortana 및 Skype와 같은 채널에 봇을 연결하여 상호 작용을 늘리고 더 많은 고객과 연결합니다.

[추가할 채널을 선택](bot-service-manage-channels.md)합니다.


### <a name="evaluate"></a>평가 
Azure Portal에서 수집된 데이터를 사용하여 봇의 성능과 기능을 향상시킬 기회를 확인합니다. 트래픽, 대기 시간 및 통합과 같은 서비스 수준 및 계측 데이터를 가져올 수 있습니다. 또한 분석은 사용자에 대한 대화 수준 보고, 메시지 및 채널 데이터를 제공합니다. 

[분석을 수집](bot-service-manage-analytics.md)하는 방법을 알아봅니다.


## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [봇 만들기](bot-service-quickstart.md)

::: moniker-end
