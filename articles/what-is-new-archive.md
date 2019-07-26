---
title: 새로운 기능 | Microsoft Docs
description: Bot Framework의 새로운 기능에 대해 알아봅니다.
keywords: bot framework, azure bot service
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b7342274e37ff33afb8695e8f25fbf0fa707178
ms.sourcegitcommit: b053c0ca7f2e9e60679f7e82e583c57ae83fcb50
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/19/2019
ms.locfileid: "68336754"
---
# <a name="whats-new-in-bot-framework-may-2019"></a>Bot Framework의 새로운 기능(2019 년 5월)

|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|SDK) |[4.4.3][1] | [4.4.0][2] | [4.4.0b1(미리 보기)][3] | [4.0.0a6(미리 보기)][3a]|
|Docs | [docs][5] |[docs][5] |  | |
|샘플 |[.NET Core][6], [WebAPI][10] |[Node.js][7] , [TypeScript][8], [es6][9]  | [Python][111] | | 

[1a]:https://github.com/microsoft/botframework-sdk/#readme
[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[4]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_typescript
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[111]:https://github.com/Microsoft/botbuilder-python/tree/master/samples

<a name="V4-whats-new"></a>
## <a name="bot-framework-sdk-new-in-preview"></a>Bot Framework SDK(신규! 미리 보기 상태)

- [적응형 대화 상자][47] | [docs][48] | [C# 샘플][49]: 적응형 대화는 개발자가 대화 진행 시 동적으로 변경될 수 있는 대화를 빌드할 수 있게 해줍니다.  기존에는 개발자가 대화의 전체 흐름을 사전에 매핑했기 때문에 대화의 유연성이 제한되었습니다.  적응형 대화는 개발자가 더 유연성을 발휘하고, 컨텍스트 변화에 대응하고, 대화 진행 시 대화에 새 단계 또는 전체 하위 대화를 삽입할 수 있게 해줍니다. 

- [언어 생성][43] | [docs][44] | [C# 샘플][45]: 언어 생성은 개발자가 코드 및 리소스 파일에서 포함된 문자열을 추출하여 언어 생성 런타임 및 파일 형식을 통해 관리할 수 있게 해줍니다.  언어 생성은 고객이 특정 문구에 여러 변형을 정의하고, 컨텍스트에 따라 단순한 식을 실행하고, 대화 메모리를 참조하고, 기능을 점차 추가하여 좀 더 자연스러운 대화 환경을 구축할 수 있게 해줍니다.

- [공통 식 언어][40]| [api][41]: 적응형 대화와 언어 생성은 모두 공통 식 언어를 사용하여 봇 대화를 지원합니다.

[40]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/common-expression-language#readme
[41]:https://github.com/Microsoft/BotBuilder-Samples/blob/master/experimental/common-expression-language/api-reference.md
[43]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation#readme
[44]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/docs
[45]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/csharp_dotnetcore
[46]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/javascript_nodejs/13.core-bot
[47]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme
[48]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs
[49]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore
[50]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/declarative

## <a name="botkit"></a>Botkit
[Botkit][100]는 주요 메시징 플랫폼용 챗봇, 앱 및 사용자 지정 통합을 빌드하기 위한 개발자 도구이자 SDK입니다. Botkit은 `hear()` 트리거, `ask()` 질문 및 `say()` 답변을 자동 처리합니다. 개발자는 이러한 구문을 사용하여 최신 버전의 Bot Framework SDK와 상호 호환 가능한 대화를 구축합니다. 

또한 Botkit에는 Javascript 봇 애플리케이션이 메시징 플랫폼과 직접 통신할 수 있게 해주는 [Slack][102], [Webex Teams][103], [Google 행아웃][104], [Facebook Messenger][105], [Twilio][106], [웹 채팅][107] 등의 6개 플랫폼 어댑터가 포함되어 있습니다.

Botkit는 Microsoft Bot Framework의 일부이며 [MIT 오픈 소스 라이선스][101]로 릴리스됩니다.

[100]:https://github.com/howdyai/botkit#readme
[101]:https://github.com/howdyai/botkit/blob/master/LICENSE.md
[102]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-slack#readme
[103]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-webex#readme
[104]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-hangouts#readme
[105]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-facebook#readme
[106]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-twilio-sms#readme
[107]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-web#readme

## <a name="bot-framework-solutions-new-in-preview"></a>Bot Framework Solutions(신규! 미리 보기 상태)

[Bot Framework Solutions 리포지토리](https://github.com/Microsoft/AI#readme)에는 템플릿 세트, 솔루션 가속기 및 도우미 같은 고급 대화 환경을 구축하는 데 도움이 되는 기술이 제공됩니다.

| Name | 설명 |  
|:------------|:------------| 
|[**가상 도우미**](https://github.com/Microsoft/AI/tree/master/docs#virtual-assistant) | 고객은 자신의 브랜드에 맞고 사용자 개개인에게 맞춤형으로 제공되며 광범위한 캔버스 및 디바이스에서 사용 가능한 대화형 도우미를 제공할 필요성을 크게 느끼고 있습니다. <br/><br/> 엔터프라이즈 템플릿은 기본 대화 의도, 디스패치 통합, QnA Maker, Application Insights 및 자동 배포 기능을 포함하고 있어 새로운 봇 프로젝트 생성 과정을 크게 간소화합니다.|
|[**기술**](https://github.com/Microsoft/AI/blob/master/docs/overview/skills.md)| 개발자는 다시 사용할 수 있는 대화형 기능, 즉 기술을 연결하여 대화형 환경을 구성할 수 있습니다. 기술은 그 자체로 봇이며 원격으로 호출됩니다. 기술 개발자 템플릿(.NET, TS)을 활용하면 새 기술을 용이하게 만들 수 있습니다. 
|[**분석**](https://github.com/Microsoft/AI/blob/master/docs/readme.md#analytics)| Conversational AI Analytics 솔루션을 사용하여 봇의 상태와 동작에 대한 핵심 인사이트를 확보하세요. 사용 가능한 원격 분석, 샘플 Application Insights 쿼리 및 Power BI 대시보드를 검토하여 사용자와 봇의 대화를 광범위하게 이해하세요. |

## <a name="azure-bot-service"></a>Azure Bot Service
Azure Bot Service는 데이터에 대한 완전한 소유권과 제어권으로 인텔리전트 엔터프라이즈급 봇을 호스트할 수 있게 해줍니다. 개발자는 Skype, Microsoft Teams, Cortana, Web Chat 등에서 봇을 등록하고 사용자에게 연결할 수 있습니다. [Azure][27] |  [docs][28] | [채널에 연결][29] 

* **Direct Line JS 클라이언트**: WebChat 클라이언트를 사용하고 있지 않으며 Azure Bot Service에서 Direct Line 채널을 사용하려면 사용자 지정 애플리케이션에서 Direct Line JS 클라이언트를 사용하면 됩니다. 자세한 내용을 보려면 [GitHub][30]로 이동하세요.

<a name="ABS-whats-new"></a>

* **신규! Direct Line Speech 채널**: 클라이언트와 봇 애플리케이션 간에 양방향으로 스트리밍되는 음성과 텍스트를 구현하는 채널을 제공하기 위해 Bot Framework와 Microsoft의 음성 서비스를 결합했습니다.  자세한 내용은 [봇에 음성 채널](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0)을 추가하는 방법을 참조하세요.

[27]:https://azure.microsoft.com/services/bot-service/
[28]:https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[29]:https://docs.microsoft.com/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0
[30]:https://github.com/Microsoft/BotFramework-DirectLineJS/blob/master/README.md


## <a name="bot-framework-emulator"></a>Bot Framework Emulator
[Bot Framework Emulator][60]는 봇 개발자가 Bot Framework SDK를 사용하여 빌드된 봇을 테스트 및 디버그할 수 있게 해주는 플랫폼 간 데스크톱 애플리케이션입니다. Bot Framework Emulator를 사용하여 머신의 로컬에서 실행 중인 봇을 테스트하거나 원격에서 실행 중인 봇에 연결할 수 있습니다.

- [최신 버전 다운로드][61] | [Docs][62]

<a name="Emulator-whats-new"></a>
### <a name="bot-inspector-new-in-preview"></a>봇 검사기(신규! 미리 보기 상태)

Bot Framework Emulator에서 베타 버전의 새로운 봇 검사기 기능을 출시했습니다. 이는 Microsoft Teams, Slack, Cortana, Facebook Messenger, Skype 등과 같은 채널에서 Bot Framework SDK v4 봇을 디버그하거나 테스트하는 기능입니다. 대화 중에 메시지가 Bot Framework Emulator에 미러링되므로 봇이 수신한 메시지 데이터를 검사할 수 있습니다. 또한 특정 순서에 채널과 봇 사이의 봇 상태 스냅샷도 렌더링됩니다. [봇 검사기](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md)에 대해 자세히 알아보세요.

[60]:https://github.com/Microsoft/BotFramework-Emulator#readme
[61]:https://github.com/Microsoft/BotFramework-Emulator/releases/latest
[62]:https://docs.microsoft.com/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0


## <a name="related-services"></a>관련 서비스

### <a name="language-understanding"></a>언어 이해 
자연어 환경을 구축하는 기계 학습 기반 서비스입니다. 지속적으로 향상되는 엔터프라이즈 수준에 적합한 사용자 맞춤형 모델을 신속하게 만들어 보세요. [LUIS(Language Understanding Service)][30]를 사용하면 애플리케이션에서 사람의 말을 통해 무엇을 원하는지 이해할 수 있습니다.

<a name="LUIS-whats-new"></a>

- **신규! 역할, 외부 엔터티 및 동적 엔터티**: LUIS에는 개발자가 텍스트에서 자세한 정보를 추출할 수 있게 해주는 몇 가지 기능이 추가되었으므로 이제 사용자가 더 뛰어난 인텔리전트 솔루션을 더 간편하게 빌드할 수 있습니다. 또한 LUIS는 모든 엔터티 유형으로 역할을 확장했으므로 동일한 엔터티를 컨텍스트에 따라 여러 가지 하위 유형으로 분류할 수 있습니다. 이제 개발자는 동적 목록 및 외부 엔터티를 통해 런타임에 모델을 식별하고 업데이트하는 등 LUIS로 수행 가능한 작업을 더 세부적으로 제어할 수 있습니다. 동적 목록은 사용자 관련 정보가 정확하게 일치될 수 있도록 예측 시간에 추가하여 엔터티를 나열하는 데 사용됩니다. 별도의 보조 엔터티 추출기가 외부 엔터티를 사용하여 실행되며, 해당 정보를 다른 모델을 위한 강력한 신호로 LUIS에 추가할 수 있습니다.

- **신규! Analytics 대시보드**: LUIS는 더 자세하고 시각적으로 풍부하며 포괄적인 분석 대시보드를 제공합니다. 사용자 친화적인 디자인은 애플리케이션을 설계할 때 대부분의 사용자가 마주하는 일반적인 문제를 해결하는 방법에 대한 간단한 설명을 제시하여 사용자가 모델의 품질, 잠재적 데이터 문제 및 모범 사례 도입 지침에 대한 자세한 인사이트를 확보할 수 있게 해줍니다.

[문서][31] | [Add language understanding to your bot][32] 

[18]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#readme
[19]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#readme
[30]:https://www.luis.ai
[31]:https://docs.microsoft.com/azure/cognitive-services/LUIS/Home
[32]:https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=csharp

### <a name="qna-maker"></a>QnA Maker
[QnA Maker][33]는 데이터 위에 대화형 질문 및 답변 레이어를 만드는 클라우드 기반 API 서비스입니다. QnA Maker를 사용하면 몇 분 안에 FAQ URL, 구조화된 문서, 제품 설명서 또는 편집 콘텐츠에 따라 간단한 질문 및 대답 봇을 빌드하고, 학습시키고, 게시할 수 있습니다.

<a name="QnA-whats-new"></a>

- **신규! 추출 파이프라인**: 이제 URL, 파일 및 SharePoint에서 계층 구조 정보를 추출할 수 있습니다.
- **신규! 인텔리전스**: 상황에 맞는 순위 지정 모델, 활성 학습 제안
- **신규! 대화**: QnA Maker의 다중 순서 대화입니다.

[문서][34]  | [add qnamaker to your bot][35] 

[33]:https://www.qnamaker.ai/
[34]:https://aka.ms/qnamaker-docs-home
[35]:https://docs.microsoft.com/azure/bot-service/bot-builder-howto-qna?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=cs

### <a name="speech-services"></a>Speech Services
[Speech Services](https://docs.microsoft.com/azure/cognitive-services/speech-service/)는 오디오를 텍스트로 변환하고, 음성 번역 및 텍스트 음성 변환을 수행합니다. Speech Services를 사용하면 봇에 음성을 통합하고, 사용자 지정 실행 단어를 만들고, 여러 언어로 제작할 수 있습니다.

### <a name="adaptive-cards"></a>Adaptive Cards
[적응형 카드](https://adaptivecards.io)는 개발자가 공통의 일관된 방식으로 카드 콘텐츠를 교환할 수 있게 해주는 개방형 표준이며, Bot Framework 개발자가 뛰어난 채널 간 대화 환경을 만드는 데 사용됩니다.

## <a name="additional-information"></a>추가 정보
- 자세한 내용은 [GitHub 페이지](https://github.com/Microsoft/botframework/blob/master/whats-new.md#whats-new)를 방문하세요.
