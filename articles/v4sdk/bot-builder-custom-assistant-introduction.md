---
title: 사용자 지정 도우미 개요 | Microsoft Docs
description: 사용자 고유의 사용자 지정 도우미를 만드는 것에 대해 알아봅니다.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 13/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ddb1a7f8f189705de6b40bf2802a7bc2d1b49135
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46708747"
---
## <a name="custom-assistant-overview"></a>사용자 지정 도우미 개요

## <a name="overview"></a>개요

광범위한 대화형 캔버스 및 장치에 사용할 수 있으며, 해당 브랜드에 맞게 조정되고, 그들의 고객들에게 개인화된 대화형 도우미를 고객과 파트너에게 제공해야 할 필요성을 봐왔습니다. Bot Framework SDK에 대한 Microsoft 오픈 소스 접근 방식에 이어서, 오픈 소스 사용자 지정 개인 도우미는 기본 기능 집합에 빌드된 최종 사용자 환경에 대한 전체 제어를 제공합니다. 또한 진정으로 통합된 지능형 환경을 위해 최종 사용자에 대한 정보 및 모든 장치/에코 시스템 정보를 환경에 적용할 수 있습니다.

우리는 우리의 고객이 자신들의 고객 관계 및 정보를 직접 소유하고 강화해야 한다고 굳게 믿고 있습니다. 따라서 사용자 지정 도우미는 고객 및 파트너에게 완벽한 사용자 환경 제어를 제공합니다. 이름, 음성 및 성격을 조직의 요구에 맞게 변경할 수 있습니다. 사용자 지정 도우미 솔루션으로 사용자 고유의 도우미 만들기가 간소화되어 몇 분만에 시작할 수 있습니다. 

사용자 지정 개인 도우미 기능의 범위는 광범위하며, 일반적으로 최종 사용자에게 다양한 기능을 제공합니다. 개발자 생산성을 높이고 재사용이 가능한 대화형 환경의 활발한 에코 시스템을 활성화하기 위해 다시 사용할 수 있는 대화형 기술의 초기 예제를 개발자에게 제공하고 있습니다. 이러한 기술을 대화형 응용 프로그램에 추가하여 관심 지점 찾기, 일정, 태스크, 이메일 및 다른 많은 시나리오와의 상호 작용처럼 특정 대화 환경을 조명할 수 있습니다. 기술은 완전히 사용자 지정할 수 있으며 여러 언어, 대화 상자 및 코드를 위해 여러 언어 모델로 구성됩니다.

이번에는 첫 번째 환경을 구현하기 위해 초기 미리 보기를 실행하고 오픈 소스 리포지토리의 초기 고객 및 파트너와 밀접한 협업을 진행하며, 다음 몇 달 동안 더 광범위하게 제공할 예정입니다. 

![사용자 지정 도우미 다이어그램](media/enterprise-template/CustomAssistantDiagram.jpg)

## <a name="complete-control-of-the-user-experience"></a>사용자 환경의 완벽한 제어

사용자는 최종 사용자 환경의 모든 측면을 소유하며 제어합니다. 여기에는 브랜딩, 이름, 음성, 성격, 응답 및 아바타가 포함됩니다. 필요에 따라 사용자가 조정할 수 있도록 사용자 지정 도우미 및 지원 기술에 대한 소스 코드가 전부 제공됩니다.

## <a name="complete-ownership-and-control-of-data"></a>완전한 소유권 및 데이터 제어

사용자 지정 도우미는 사용자의 Azure 구독 내에서 배포됩니다. 따라서 도우미에서 생성된 모든 데이터(질문, 사용자 동작 등)은 사용자의 Azure 구독 내에 전적으로 포함됩니다. [Cognitive Services Azure Trusted Cloud](https://www.microsoft.com/en-us/trustcenter/cloudservices/cognitiveservices) 및 [보안 센터의 Azure 섹션](https://www.microsoft.com/en-us/TrustCenter/CloudServices/Azure)에서 자세한 내용을 더 구체적으로 확인하세요.

## <a name="your-assistant-anywhere"></a>어디서든 사용 가능한 당신의 도우미

사용자 지정 도우미는 Microsoft 대화형 AI 플랫폼을 활용하므로 웹 채팅, FaceBook Messenger, Skype 등 모든 Bot Framework [채널](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0)을 통해 표시될 수 있습니다. 또한 [직접 회선](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0) 채널을 통해 자동차, 스피커, 알람 시계 등의 장치를 포함한 모바일 장치 및 데스크톱으로 환경을 임베딩할 수 있습니다.

## <a name="built-on-enterprise-grade-technology"></a>엔터프라이즈급 기술로 빌드

사용자 지정 도우미 솔루션은 지원되는 Azure 구성 요소의 광범위한 집합과 함께 Azure Bot Service, Language Understanding Cognitive Service, 통합 음성을 기반으로 하여 [Azure 글로벌 인프라](https://azure.microsoft.com/en-gb/global-infrastructure/)의 혜택을 누릴 수 있습니다.

또한, Language Understanding 지원은 [여기에 나열된](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-supported-languages) 광범위한 언어 집합을 지원하는 LUIS Cognitive Service에서 제공됩니다. [Translator Cognitive Service](https://azure.microsoft.com/en-us/services/cognitive-services/translator-text-api/)는 사용자 지정 도우미의 도달 범위를 더욱 확장하도록 추가 기계 번역 기능을 제공합니다.

## <a name="integrated-and-context-aware"></a>통합 및 컨텍스트 인식

사용자 지정 도우미는 사용자의 장치 및 에코시스템으로 통합이 가능하여 진정으로 통합된 지능형 환경을 구현할 수 있습니다. 이 컨텍스트 인식을 통해 더 지능적인 환경을 개발할 수 있으며 가능한 더 많은 개인화를 제공합니다.

## <a name="3rd-party-assistant-integration"></a>타사 도우미 통합

사용자 지정 도우미를 사용하면 자신의 고유한 환경뿐 아니라, 특정 질문 유형에 대해 최종 사용자가 선택한 디지털 도우미로의 이전도 제공할 수 있습니다.

## <a name="flexible-integration"></a>유연한 통합

사용자 지정 도우미 아키텍처는 유연합니다. 기존 백 엔드 시스템 및 API와의 통합은 물론, 음성 또는 자연어 처리 기능을 기반으로 하는 장치에 대한 기존 투자와도 통합할 수 있습니다.

## <a name="adaptive-cards"></a>Adaptive Cards

[적응형 카드](https://adaptivecards.io/)는 사용자 지정 도우미가 텍스트 기반 응답과 함께 사용자 환경 요소(예: 카드, 이미지, 단추)를 반환하는 기능을 제공합니다. 장치 또는 대화 캔버스에 화면이 있는 경우 이러한 적응형 카드를 광범위한 장치 및 플랫폼에 렌더링하여 적절한 사용자 환경 지원을 제공할 수 있습니다. 적응형 카드의 예제는 [여기](https://docs.microsoft.com/en-us/adaptive-cards/rendering-cards/getting-started) 문서의 렌더링 옵션에 관한 정보와 함께 [여기](https://adaptivecards.io/samples/)에서 확인할 수 있습니다.


## <a name="skills"></a>기술

기본 도우미 외에도 각 개발자가 빌드하는 데 필요한 일반적인 기능의 광범위한 집합이 있습니다. 생산성은 각 조직이 언어 모델(LUIS), 대화 상자(코드), 통합(코드) 및 언어 생성(응답)을 생성하여 관심 지점, 이메일, 일정 또는 태스크와 같은 일반적인 시나리오를 활성화해야 하는 좋은 예입니다.

이는 여러 언어를 지원해야 하므로 더욱 복잡해지고, 그 결과 자체 도우미를 빌드하는 모든 조직에 많은 작업이 필요하게 됩니다.

사용자 지정 도우미 솔루션에는 구성만 하면 새 기능을 사용자 지정 도우미에 연결할 수 있는 새로운 기술 기능이 포함되어 있습니다. 

각 기술(언어 모델, 대화 상자, 통합 코드 및 언어 생성)의 모든 측면은 사용자 지정 도우미와 마찬가지로 전체 소스 코드가 GitHub에서 제공되므로 개발자가 전적으로 사용자 지정할 수 있습니다.

## <a name="example-scenarios"></a>예제 시나리오

사용자 지정 도우미는 다양한 산업 시나리오로 확장됩니다. 예제 시나리오는 참조용으로 다음에 나와있습니다.

- 자동차 산업: 자동차에 통합된 음성 지원 개인 도우미 - 늦는 경우 회의 변경, 태스크 목록에 항목 추가, 이벤트(예: 엔진 켜기, 귀가 또는 자동 주행 속도 유지 장치 사용)에 따라 완료할 태스크를 자동차가 제안할 수 있는 사전 예방 환경과 같은 생산성 집중 시나리오와 함께 기존 자동차 작동(예: 네비게이션, 라디오)을 수행하는 기능을 최종 사용자에게 제공합니다. 적응형 카드는 PTT(말하려면 누르기) 또는 Wake Word(시동 단어) 상호 작용을 통해 수행되는 헤드 유닛과 음성 통합 내에 렌더링됩니다.

- 숙박: 호텔방 장치에 통합된 음성 지원 개인 도우미 - 컨시어지 및 지역 내 음식점 및 명소 찾기 기능을 포함한 광범위한 숙박용 시나리오를 제공(예: 숙박 기간 연장, 퇴실 지연 요청, 룸 서비스)합니다. 생산성과 연관된 선택적 계정은 제안된 경보 호출, 날씨 경고 및 체류 기간 동안의 패턴 학습과 같은 더 개인화된 환경을 제공합니다. 현재 TV 개인화의 진화는 오늘날 객실에서 영향을 받았습니다.

- 엔터프라이즈: 엔터프라이즈 장치 및 기존 대화 캔버스(예: Teams, 웹 채팅, Slack)에 통합된 음성 및 텍스트 지원 브랜드화된 직원 도우미 환경 - 직원이 자신의 일정을 관리하거나, 이용 가능한 회의실을 찾거나, 특정 기술을 보유한 사람을 찾거나, HR 관련 작업을 수행할 수 있도록 합니다. 

## <a name="getting-started"></a>시작하기

이번에는 첫 번째 환경을 구현하기 위해 초기 미리 보기를 실행하고 오픈 소스 리포지토리의 초기 고객 및 파트너와 밀접한 협업을 진행하며, 다음 몇 달 동안 더 광범위하게 제공할 예정입니다. 자신의 관심사를 등록하거나 시작하려면 [이 양식](https://aka.ms/customassistantpreviewform)을 채우세요. 연락드리겠습니다.

