---
title: Bot service 질문과 대답 | Microsoft Docs
description: Bot Framework의 요소 및 새 기능이 사용 가능해지는 시점에 대한 질문과 대답 목록입니다.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/28/2018
ms.openlocfilehash: 2a78ec6f8c453bfcfa3b6aba0d73257f35db76b8
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997220"
---
# <a name="bot-framework-frequently-asked-questions"></a>Bot Framework 질문과 대답

이 문서에는 Bot Framework에 대한 몇 가지 질문과 대답이 포함되어 있습니다.

## <a name="background-and-availability"></a>배경 및 가용성
### <a name="why-did-microsoft-develop-the-bot-framework"></a>Microsoft는 Bot Framework를 왜 개발했나요?

CUI(대화 사용자 인터페이스)는 곧 사용 가능해질 예정이지만 현재 소수의 개발자만 새로운 대화 환경을 만들거나 사용자가 즐길 수 있는 대화 인터페이스로 기존 응용 프로그램 및 서비스를 설정하는 데 필요한 전문 지식과 도구를 보유하고 있는 상황입니다. Microsoft는 Bot Framework를 만들어 개발자가 뛰어난 봇을 쉽게 개발하고 Microsoft의 프리미어 채널에 포함하여 대화가 진행되는 어디에서든지 사용자에게 봇을 연결할 수 있도록 하고 있습니다.

### <a name="what-is-the-v4-sdk"></a>V4 SDK란?
Bot Builder v4 SDK는 이전 Bot Builder SDK에서 얻은 의견 및 학습 결과를 토대로 구축되었습니다. 이 키트는 봇 빌딩 블록의 풍부한 구성 요소화를 지원하면서 적절한 수준의 추상화를 제공합니다. 간단한 봇에서 시작하고 확장 가능한 모듈식 프레임워크를 사용하여 봇을 좀 더 정교하게 확장할 수 있습니다. GitHub에서 SDK에 대한 [FAQ](https://github.com/Microsoft/botbuilder-dotnet/wiki/FAQ)를 찾을 수 있습니다.


## <a name="channels"></a>채널
### <a name="when-will-you-add-more-conversation-experiences-to-the-bot-framework"></a>언제 Bot Framework에 더 많은 대화 환경을 추가할 예정인가요?

추가 채널을 비롯하여 Bot Framework를 지속적으로 개선할 예정이지만 현재는 일정을 확인할 수 없습니다.  
프레임워크에 특정 채널을 추가하려는 경우 [알려주세요][Support].

### <a name="i-have-a-communication-channel-id-like-to-be-configurable-with-bot-framework-can-i-work-with-microsoft-to-do-that"></a>Bot Framework를 사용하여 구성하려는 통신 채널이 있습니다. Microsoft와 함께 이 작업을 수행할 수 있나요?

개발자가 Bot Framework에 새 채널을 추가할 수 있도록 하는 일반적인 메커니즘을 제공하고 있지는는 않지만 [직접 회선 API][DirectLineAPI]를 통해 앱에 봇을 연결할 수 있습니다. 통신 채널의 개발자이며 Microsoft와 협력하여 Bot Framework에서 채널을 사용하도록 설정하려는 경우 [의견을 알려주세요][Support].

### <a name="if-i-want-to-create-a-bot-for-skype-what-tools-and-services-should-i-use"></a>Skype용 봇을 만들려고 하는 경우 어떤 도구와 서비스를 사용해야 하나요?

Bot Framework는 Skype 및 기타 다양한 채널을 위한 응답성과 성능이 뛰어나고 확장이 가능한 고품질 봇을 빌드, 연결 및 배포하도록 디자인되었습니다. 이 SDK를 사용하여 풍부한 오디오 및 비디오 환경과 같은 Skype 관련 봇 상호 작용 뿐만 아니라 텍스트/sms, 이미지, 단추 및 카드 사용 가능 봇(오늘날 대화 환경에서 봇 상호 작용의 대부분을 구성)을 만들 수 있습니다.

유용한 봇이 이미 있고 대상 사용자를 Skype 사용자로 확장하려는 경우 REST API용 Bot Builder를 통해 Skype(또는 기타 지원되는 채널)에 쉽게 연결할 수 있습니다.

## <a name="security-and-privacy"></a>보안 및 개인 정보
### <a name="do-the-bots-registered-with-the-bot-framework-collect-personal-information-if-yes-how-can-i-be-sure-the-data-is-safe-and-secure-what-about-privacy"></a>Bot Framework에 등록된 봇이 개인 정보를 수집하나요? 그렇다면 데이터가 안전하고 보안이 유지되는지 어떻게 확신할 수 있나요? 개인 정보 보호의 경우는 어떤가요?

각 봇에는 자체 서비스가 있고 이러한 서비스의 개발자는 개발자 준수 사항에 따라 서비스 약관 및 개인정보처리방침을 제공해야 합니다.  이 정보는 봇 디렉터리의 봇 카드에서 액세스할 수 있습니다.

I/O 서비스를 제공하기 위해 Bot Framework는 사용자가 사용한 채팅 서비스에서 봇으로 메시지 및 메시지 콘텐츠(사용자 ID 포함)를 전송합니다.

### <a name="can-i-host-my-bot-on-my-own-servers"></a>내 서버에서 내 봇을 호스트할 수 있나요?
예. 봇은 인터넷상의 어느 곳에서나 호스팅할 수 있습니다. 자체 서버, Azure 또는 다른 데이터 센터에서 호스팅할 수 있습니다. 유일한 요구 사항은 봇이 공개적으로 액세스 가능한 HTTPS 엔드포인트를 공개해야 한다는 것입니다.

### <a name="how-do-you-ban-or-remove-bots-from-the-service"></a>서비스에서 봇을 금지하거나 제거하려면 어떻게 하나요?

사용자는 디렉터리에 있는 봇의 연락처 카드를 통해 오작동하는 봇을 보고할 수 있습니다. 개발자는 서비스에 참여하려면 Microsoft 서비스 약관을 준수해야 합니다.

### <a name="which-specific-urls-do-i-need-to-whitelist-in-my-corporate-firewall-to-access-bot-framework-services"></a>Bot Framework Service에 액세스하기 위해 회사 방화벽에서 허용 목록에 포함해야 하는 특정 URL이 있나요?
봇에서 인터넷으로 보내는 트래픽을 차단하는 아웃바운드 방화벽이 있는 경우, 해당 방화벽에서 다음 URL을 허용 목록에 포함해야 합니다.
- login.botframework.com(봇 인증)
- login.microsoftonline.com(봇 인증)
- westus.api.cognitive.microsoft.com(Luis.ai NLP 통합용)
- state.botframework.com(프로토타입 생성을 위한 봇 상태 저장소)
- cortanabfchanneleastus.azurewebsites.net(Cortana 채널)
- cortanabfchannelwestus.azurewebsites.net(Cortana 채널)
- *.botframework.com(채널)

### <a name="can-i-block-all-traffic-to-my-bot-except-traffic-from-the-bot-connector-service"></a>제 봇으로 들어오는 트래픽 중에서 Bot Connector Service에서 들어오는 트래픽을 제외한 모든 트래픽을 차단할 수 있나요?
아니요. 이러한 종류의 IP 주소 또는 DNS 허용 목록은 허용되지 않습니다. Bot Framework Connector Service는 전 세계 Azure 데이터 센터에서 호스팅되고 Azure IP 목록은 지속적으로 변경됩니다. 허용 목록에 추가한 특정 Azure IP 주소가 다음 날 Azure IP 주소가 변경되면서 차단될 수도 있습니다.
 
### <a name="what-keeps-my-bot-secure-from-clients-impersonating-the-bot-framework-connector-service"></a>Bot Framework Connector Service를 가장하는 클라이언트로부터 내 봇을 보호하기 위해 필요한 것은 무엇인가요?
1. 봇에 대한 모든 요청에 첨부되는 보안 토큰에는 ServiceUrl이 인코딩되어 있으므로, 공격자가 토큰에 액세스하더라도 대화를 새 ServiceUrl로 리디렉션할 수 없습니다. 이는 SDK의 모든 구현에 의해 적용되며 인증 [참조](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-authentication?view=azure-bot-service-3.0#bot-to-connector) 자료에 문서화되어 있습니다.

2. 들어오는 토큰이 없거나 형식이 잘못된 경우 Bot Framework SDK는 응답에서 토큰을 생성하지 않습니다. 이렇게 하면 봇이 잘못 구성된 경우 피해를 줄일 수 있습니다.
3. 봇 내부에서 토큰에 제공된 ServiceUrl을 수동으로 확인할 수 있습니다. 이 방법은 가능하기는 하지만 서비스 토폴로지가 변경될 경우 봇이 더 취약해지므로 권장하지는 않습니다.


이는 봇에서 인터넷으로의 아웃바운드 연결입니다. Bot Framework Connector Service가 봇과 통신할 때 사용하는 IP 주소 또는 DNS 이름 목록이 없습니다. 인바운드 IP 주소 허용 목록은 지원되지 않습니다.

## <a name="rate-limiting"></a>속도 제한
### <a name="what-is-rate-limiting"></a>속도 제한이란?
어떤 단일 봇도 다른 봇의 성능에 나쁜 영향을 미칠 수 없도록 Bot Framework 서비스는 서비스 자체 및 해당 고객을 악의적인 호출 패턴(예: 서비스 거부 공격)에서 보호해야 합니다. 이러한 종류의 보호를 위해 모든 끝점에 속도 제한(제한이라고도 함)을 추가했습니다. 속도 제한을 적용하여 봇이 특정 호출을 수행할 수 있는 빈도를 제한할 수 있습니다. 예를 들어, 속도 제한을 사용하도록 설정한 경우 봇이 많은 수의 작업을 게시하고 싶다면 시간이 지나면서 공간을 확보해야 할 것입니다. 속도 제한을 적용하는 목적이 봇의 총 용량을 제한하려는 것은 아닙니다. 사람의 대화 패턴을 따르지 않는 대화형 인프라가 남용되는 것을 방지하도록 디자인된 것입니다.

### <a name="how-will-i-know-if-im-impacted"></a>영향을 받는지 여부를 어떻게 알 수 있나요?
높은 용량에서도 속도 제한을 경험할 가능성은 낮습니다. 대부분의 속도 제한은 작업의 대량 전송(봇 또는 클라이언트로부터), 과도한 부하 테스트 또는 버그로 인해서만 발생합니다. 요청이 제한될 경우 요청 다시 시도가 성공하기 전에 대기할 시간(초)을 나타내는 Retry-After 헤더와 함께 HTTP 429(너무 많은 요청) 응답이 반환됩니다. Azure Application Insights를 통해 봇에 대해 분석 기능을 사용하도록 설정하여 이 정보를 수집할 수 있습니다. 또는 봇의 코드를 로그 메시지에 추가할 수 있습니다. 

### <a name="how-does-rate-limiting-occur"></a>속도 제한은 어떻게 발생하나요?
다음과 같은 경우에 발생할 수 있습니다.
-   봇이 메시지를 너무 자주 전송하는 경우
-   봇의 클라이언트가 메시지를 너무 자주 전송하는 경우
-   직접 회선 클라이언트가 새 웹 소켓을 너무 자주 요청하는 경우

### <a name="what-are-the-rate-limits"></a>속도 제한이란?
Microsoft는 서비스 및 사용자를 보호하면서 동시에 속도 제한을 계속해서 가능한 한 관대하게 조정하고 있습니다. 때때로 임계값이 변경되므로 지금은 수치를 게시하지 않습니다. 속도 제한의 영향을 받는 경우 [bf-reports@microsoft.com](mailto://bf-reports@microsoft.com)으로 자유롭게 연락해주세요.

## <a name="related-services"></a>관련 서비스
### <a name="how-does-the-bot-framework-relate-to-cognitive-services"></a>Bot Framework는 Cognitive Services와 어떤 관계가 있나요?

Bot Framework와 [Cognitive Services](http://www.microsoft.com/cognitive) 둘 다 [Microsoft Build 2016](http://build.microsoft.com)에 도입된 새로운 기능으로, GA에서는 Cortana Intelligence Suite에도 통합될 예정입니다. 이러한 두 서비스는 수년 간의 연구를 통해 작성되었으며 인기 있는 Microsoft 제품에서 사용됩니다. 이러한 기능은 'Cortana Intelligence'와 결합되어 모든 조직이 데이터, 클라우드 및 인텔리전스의 능력을 활용하여 자체적인 지능형 시스템을 구축하도록 함으로써 새로운 기회를 열고, 비즈니스 속도를 향상시키고, 고객 서비스 분야에서 선두자리를 유지할 수 있도록 합니다.

### <a name="what-is-cortana-intelligence"></a>Cortana Intelligence란?

Cortana Intelligence는 데이터를 지능형 작업으로 변환하는 완전히 관리되는 빅 데이터, 고급 분석 및 인텔리전스 제품군입니다.  
Microsoft에서 수년에 걸쳐 진행된 연구 및 혁신을 토대로 구현된 기술(클라우드의 다양한 고급 분석, 기계 학습, 빅 데이터 저장소 및 처리)을 함께 제공하는 포괄적인 제품군입니다.

* 모든 데이터를 수집하고, 관리하고, 저장할 수 있으며, 이러한 데이터는 시간이 지나면서 확장 가능한 보안 방식으로 매끄럽고 경제적으로 증가될 수 있습니다.
* 가장 까다로운 문제에 대한 의사 결정을 예측하고, 규정하고, 자동화할 수 있는 클라우드 기반의 편리하고 실행 가능한 분석 기능을 제공합니다.
* Cognitive Services 및 에이전트를 통해 지능형 시스템을 사용하여 주변 세계를 상황을 고려해서 자연스러운 방식으로 보고, 듣고, 해석하고, 이해할 수 있습니다.

Microsoft는 Cortana Intelligence를 사용하여 엔터프라이즈 고객이 새로운 기회를 창출하고, 비즈니스 속도를 높이고, 해당 분야에서 선두주자가 될 수 있도록 지원하려고 합니다.

### <a name="what-is-the-direct-line-channel"></a>직접 회선 채널이란?

직접 회선은 봇을 서비스, 모바일 앱 또는 웹 페이지에 추가할 수 있는 REST API입니다.

모든 언어에서 직접 회선 API용 클라이언트를 작성할 수 있습니다. 간단히 [직접 회선 프로토콜][DirectLineAPI]을 코딩하고, 직접 회선 구성 페이지에서 암호를 생성하고, 코드가 있는 어디에서든지 봇과 통신합니다.

직접 회선은 다음에 적합합니다.

* IOS, Android 및 Windows Phone 등의 모바일 앱
* Windows, OSX 등의 데스크톱 응용 프로그램
* [포함 가능한 웹 채팅 채널][WebChat]이 제공하는 것 이상의 사용자 지정이 필요한 웹 페이지
* 서비스 간 응용 프로그램

[DirectLineAPI]: https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts
[Support]: bot-service-resources-links-help.md
[WebChat]: bot-service-channel-connect-webchat.md

