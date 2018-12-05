---
title: 가상 도우미 개요 | Microsoft Docs
description: 사용자 고유의 가상 도우미를 만드는 방법 알아보기
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 13/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3a3074ff9e8de36d7a7cf7b86fef62979bc53fa8
ms.sourcegitcommit: 958a28bbab7dd29b384bb2e2d58d866e88f53316
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/28/2018
ms.locfileid: "52500657"
---
# <a name="virtual-assistant-solution-overview"></a>가상 도우미 솔루션 개요

## <a name="overview"></a>개요
광범위한 대화형 캔버스 및 장치에 사용할 수 있고, 브랜드에 맞게 조정되고, 고객에 맞게 개인화된 대화형 도우미를 요구하는 고객과 파트너가 매우 많습니다. Bot Framework SDK에 대한 Microsoft의 오픈 소스 접근 방식을 계속 이어가는 오픈 소스 사용자 지정 개인 도우미 솔루션은 기본 기능 집합에 빌드된 최종 사용자 환경에 대한 전체 제어를 제공합니다. 또한 진정으로 통합된 지능형 환경을 위해 최종 사용자에 대한 정보 및 모든 장치/에코 시스템 정보를 환경에 적용할 수 있습니다.

우리는 고객이 자신들의 고객 관계 및 인사이트를 직접 소유하고 강화해야 한다고 굳게 믿고 있습니다. 따라서 가상 도우미는 GitHub에 제공되는 오픈 소스 코드를 통해 고객과 파트너에게 완전한 사용자 환경 제어를 제공합니다. 이름, 음성 및 성격을 조직의 요구에 맞게 변경할 수 있습니다. 가상 도우미 솔루션은 사용자 고유의 도우미 만들기를 간소화하여 불과 몇 분 안에 시작하여 통합형 개발 도구 사용을 확장할 수 있습니다.

가상 도우미 기능의 범위는 광범위하며, 일반적으로 최종 사용자에게 다양한 기능을 제공합니다. 개발자 생산성을 높이고 재사용 가능한 대화형 환경의 활발한 에코 시스템을 활성화하기 위해, 재사용 가능한 대화형 기술의 초기 예제가 개발자에게 제공됩니다. 이러한 기술을 대화형 응용 프로그램에 추가하여 관심 지점 찾기, 일정, 태스크, 이메일 및 다른 많은 시나리오와의 상호 작용처럼 특정 대화 환경을 조명할 수 있습니다. 기술은 완전히 사용자 지정할 수 있으며 여러 언어, 대화 상자 및 코드를 위해 여러 언어 모델로 구성됩니다.

현재 오픈 소스 리포지토리에 있는 초기 고객 및 파트너와 긴밀하게 협력하여 첫 번째 환경을 구현하고 있습니다.

![가상 도우미 다이어그램](./media/enterprise-template/customassistantdiagram.jpg)

## <a name="whats-in-the-box"></a>가상 도우미의 기능 

가상 도우미는 다중 언어의 기본 대화 의도, 디스패치, QnA 및 대화 인사이트를 포함하여 대화형 환경에 필요한 기본 기능을 제공하는 [엔터프라이즈 템플릿](./bot-builder-enterprise-template-overview.md) 기능을 기반으로 합니다. 현재 다음과 같은 도우미 관련 기능이 제공되고 더 많은 기능이 추가될 예정이며, 고객 및 파트너와 긴밀하게 협력하여 로드맵을 알려드릴 것입니다.

기능 | 설명 |
------------ | -------------
온보딩 | 도우미가 사용자에게 인사하고 초기 정보를 수집할 수 있게 해주는 예제 온보딩 흐름.
이벤트 아키텍처 | 가상 도우미 컨텍스트의 이벤트는 도우미를 (웹 브라우저에 또는 자동차나 스피커 같은 장치에) 호스팅하는 클라이언트 응용 프로그램이 사용자 또는 장치 이벤트에 대한 정보를 교환하는 한편 장치 작업을 수행하는 이벤트를 수신할 수 있게 해줍니다.
연결된 계정 | 음성 중심 시나리오에서는 사용자가 음성 명령을 통해 시스템을 지원하기 위해 사용자 이름과 암호를 입력하는 것이 실용적이지 않습니다. 따라서 별도의 동반 환경을 통해 사용자가 로그인하여가상 도우미가 나중에 사용할 토큰을 검색할 수 있도록 권한을 부여할 수 있는 기회를 제공합니다.
기술 구현 지원 | 각 개발자가 직접 빌드해야 하는 광범위한 공통 기능 집합이 있습니다. 가상 도우미 솔루션은 구성만을 통해 새 기능을 가상 도우미에 연결할 수 있는 새로운 기술이 포함되어 있으며, 다운스트림 작업에 대한 토큰을 요청하는 기술에 대한 인증 메커니즘을 제공합니다.
관심 지점 기술 | PoI(관심 지점) 기술 미리 보기는 관심 지점을 찾아서 지침을 요청하는 포괄적인 언어 모델을 제공합니다. 이 기술은 현재 Azure Maps와 통합됩니다.
달력 기술 | 달력 기술 미리 보기는 일반적인 달력 관련 작업에 대한 포괄적인 언어 모델을 제공합니다. 이 기술은 현재 Microsoft Graph(Office 365/Outlook.com)에 통합되며 조만간 Google API를 지원할 예정입니다.
이메일 기술 | 이메일 기술 미리 보기는 일반적인 이메일 관련 작업에 대한 포괄적인 언어 모델을 제공합니다. 이 기술은 현재 Microsoft Graph(Office 365/Outlook.com)에 통합되며 조만간 Google API를 지원할 예정입니다.
ToDo 기술 | ToDo 기술 미리 보기는 일반적인 태스크 관련 작업에 대한 포괄적인 언어 모델을 제공합니다. 이 기술은 현재 OneNote에 통합되며 조만간 Microsoft Graph(outlookTask)를 지원할 예정입니다.
장치 통합 | 적응형 카드 및 Speech SDK와 함께 Azure Bot Service SDK(DirectLine)는 장치에 대한 간편한 플랫폼 간 통합을 지원합니다. Edge를 포함한 추가 장치 통합 예제 및 플랫폼이 계획되어 있습니다.
테스트 도구 | Bot Framework Emulator 외에도, 복잡한 인증 시나리오를 테스트할 수 있는 웹 채팅 기반 테스트 도구가 제공됩니다. 간단한 콘솔 기반 테스트 도구는 장치 통합의 용이성을 보여주는 메시지 교환 방법을 보여줍니다.
자동화된 배포 | 도우미에 필요한 모든 Azure 리소스(봇 등록, Azure App Service, LUIS, QnAMaker, Content Moderator, CosmosDB, Azure Storage 및 Application Insights)는 자동으로 배포됩니다. 또한 모든 기술에 대한 기본 LUIS, QnAMaker 및 디스패치 모델을 만들고, 학습하고, 게시하여 즉시 테스트할 수 있습니다.
자동차 언어 모델 | 전화기, 내비게이션, 자동차 실내 기능 제어 등의 핵심 도메인을 다루는 자동차 언어 모델이 조만간 제공될 예정입니다.

## <a name="example-scenarios"></a>예제 시나리오

가상 도우미는 다양한 산업 시나리오로 확장되며, 아래에 참조용 예제 시나리오가 제공되어 있습니다.

- 자동차 산업: 자동차에 통합된 음성 지원 개인 도우미 - 늦는 경우 회의 변경, 태스크 목록에 항목 추가, 이벤트(예: 엔진 켜기, 귀가 또는 자동 주행 속도 유지 장치 사용)에 따라 완료할 태스크를 자동차가 제안할 수 있는 사전 예방 환경과 같은 생산성 집중 시나리오와 함께 기존 자동차 작동(예: 네비게이션, 라디오)을 수행하는 기능을 최종 사용자에게 제공합니다. 적응형 카드는 PTT(말하려면 누르기) 또는 Wake Word(시동 단어) 상호 작용을 통해 수행되는 헤드 유닛과 음성 통합 내에 렌더링됩니다.

- 숙박: 호텔방 장치에 통합된 음성 지원 개인 도우미 - 컨시어지 및 지역 내 음식점 및 명소 찾기 기능을 포함한 광범위한 숙박용 시나리오(예: 숙박 기간 연장, 퇴실 지연 요청, 룸 서비스)를 제공합니다. 생산성과 연관된 선택적 계정은 제안된 경보 호출, 날씨 경고 및 체류 기간 동안의 패턴 학습과 같은 더 개인화된 환경을 제공합니다. 현재 TV 개인화의 진화는 오늘날 객실에서 영향을 받았습니다.

- 엔터프라이즈: 엔터프라이즈 장치 및 기존 대화 캔버스(예: Teams, 웹 채팅, Slack)에 통합된 음성 및 텍스트 지원 브랜드화된 직원 도우미 환경 - 직원이 자신의 일정을 관리하거나, 이용 가능한 회의실을 찾거나, 특정 기술을 보유한 사람을 찾거나, HR 관련 작업을 수행할 수 있도록 합니다.

## <a name="virtual-assistant-principles"></a>가상 도우미 원칙

### <a name="your-data-your-brand-and-your-experience"></a>데이터, 브랜드 및 환경
최종 사용자 환경의 모든 측면을 고객이 소유하고 제어합니다. 여기에는 브랜딩, 이름, 음성, 성격, 응답 및 아바타가 포함됩니다. 필요에 따라 고객이 조정할 수 있도록 가상 도우미 및 지원 기술에 대한 소스 코드가 전부 제공됩니다.

가상 도우미는 고객의 Azure 구독 내에서 배포됩니다. 따라서 도우미에서 생성된 모든 데이터(질문, 사용자 동작 등) 전체가 Azure 구독 내에 포함됩니다. 자세한 내용은 [Cognitive Services Azure Trusted Cloud](https://www.microsoft.com/en-us/trustcenter/cloudservices/cognitiveservices) 및 [보안 센터의 Azure 섹션](https://www.microsoft.com/en-us/TrustCenter/CloudServices/Azure)을 참조하세요.

### <a name="write-it-once-embed-it-anywhere"></a>한 번 작성하여 어디에나 포함
가상 도우미는 Microsoft 대화형 AI 플랫폼을 활용하므로 웹 채팅, FaceBook Messenger, Skype 등 모든 Bot Framework [채널](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0)을 통해 표시될 수 있습니다. 

또한 [직접 회선](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0) 채널을 통해 자동차, 스피커, 알람 시계 등의 장치를 포함한 모바일 장치 및 데스크톱으로 환경을 포함할 수 있습니다.

### <a name="enterprise-grade-solutions"></a>엔터프라이즈급 솔루션
가상 도우미 솔루션은 지원되는 Azure 구성 요소의 광범위한 집합과 함께 Azure Bot Service, Language Understanding Cognitive Service, 통합 음성을 기반으로 하며, 따라서 ISO 27018, HIPPA, PCI DSS, SOC 1, 2, 3 인증을 포함한 [Azure 글로벌 인프라](https://azure.microsoft.com/en-gb/global-infrastructure/)의 혜택을 누릴 수 있습니다.

또한 Language Understanding 지원은 [여기에 나열된](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-supported-languages) 광범위한 언어 집합을 지원하는 LUIS Cognitive Service에서 제공됩니다. [Translator Cognitive Service](https://azure.microsoft.com/en-us/services/cognitive-services/translator-text-api/)는 가상 도우미의 도달 범위를 더욱 확장하도록 추가 기계 번역 기능을 제공합니다.

### <a name="integrated-and-context-aware"></a>통합 및 컨텍스트 인식
가상 도우미는 고객의 장치 및 에코시스템으로 통합이 가능하여 진정으로 통합된 지능형 환경을 구현할 수 있습니다. 이 컨텍스트 인식을 통해 더 지능적인 환경을 개발할 수 있으며 가능한 더 많은 개인화를 제공합니다.

### <a name="3rd-party-assistant-integration"></a>타사 도우미 통합
가상 도우미를 사용하면 고객의 고유한 환경을 제공할 수 있을 뿐 아니라, 특정 질문 유형에 대해 최종 사용자가 선택한 디지털 도우미로 이전할 수 있습니다.

### <a name="flexible-integration"></a>유연한 통합
가상 도우미 아키텍처는 유연합니다. 기존 백 엔드 시스템 및 API와의 통합은 물론, 음성 또는 자연어 처리 기능을 기반으로 하는 장치에 대한 기존 투자와도 통합할 수 있습니다.

### <a name="adaptive-cards"></a>Adaptive Cards
[적응형 카드](https://adaptivecards.io/)는 가상 도우미가 텍스트 기반 응답과 함께 사용자 환경 요소(예: 카드, 이미지, 단추)를 반환하는 기능을 제공합니다. 장치 또는 대화 캔버스에 화면이 있는 경우 이러한 적응형 카드를 광범위한 장치 및 플랫폼에 렌더링하여 적절한 사용자 환경 지원을 제공할 수 있습니다. 적응형 카드의 예제는 [여기](https://docs.microsoft.com/en-us/adaptive-cards/rendering-cards/getting-started) 문서의 렌더링 옵션에 관한 정보와 함께 [여기](https://adaptivecards.io/samples/)에서 확인할 수 있습니다.

### <a name="skills"></a>기술
기본 도우미 외에도 각 개발자가 빌드하는 데 필요한 일반적인 기능의 광범위한 집합이 있습니다. 생산성은 각 조직이 언어 모델(LUIS), 대화 상자(코드), 통합(코드) 및 언어 생성(응답)을 만들어서 인기 있는 달력, 태스크 또는 이메일 환경을 활성화해야 하는 좋은 예입니다.

여러 언어를 지원해야 하므로 더욱 복잡해지고, 그 결과 자체 도우미를 빌드하는 모든 조직에 많은 작업이 필요하게 됩니다.

가상 도우미 솔루션에는 구성만 하면 새 기능을 사용자 지정 도우미에 연결할 수 있는 새로운 기술 기능이 포함되어 있습니다. 

각 기술(언어 모델, 대화 상자, 통합 코드 및 언어 생성)의 모든 측면은 가상 도우미와 마찬가지로 전체 소스 코드가 GitHub에서 제공되므로 개발자가 전적으로 사용자 지정할 수 있습니다.

## <a name="getting-started"></a>시작하기

가상 도우미 솔루션은 가상 도우미 팀에서 정기적으로 업데이트하는 [이 GitHub 리포지토리](https://github.com/Microsoft/AI/tree/master/solutions/Virtual-Assistant)에서 사용할 수 있습니다. 동일한 리포지토리에서 더 자세한 설명서를 사용할 수 있으며, GitHub 피드백 메커니즘을 통해 문제/피드백을 직접 제공할 수 있습니다. 논의하려는 주요 시나리오가 있는 경우 [이 양식](https://aka.ms/customassistantpreviewform)을 작성해 주시면 연락드리겠습니다.

