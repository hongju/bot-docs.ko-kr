---
title: 정보 봇 시나리오 | Microsoft Docs
description: Bot Framework를 사용하여 정보 봇 시나리오를 살펴봅니다.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7c85227d062f32f9f1e8096f1dad83c09ed85a9f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303458"
---
# <a name="information-bot-scenario"></a>정보 봇 시나리오
이 정보 봇은 Cognitive Services QnA Maker를 사용하여 기술 자료 집합이나 FAQ에 정의된 질문에 답변하고 Azure Search를 사용하여 제한 없이 더 많은 질문에 답변할 수 있습니다.

종종 정보는 검색을 통해 쉽게 표시될 수 있는 SQL Server와 같은 구조화된 데이터 저장소에 포함되어 있습니다. 간단한 대화형 명령을 사용하여 고객의 주문 상태를 조회한다고 가정합니다. Cognitive Services QnA Maker를 사용하여 고객 조회, 고객의 가장 최근 주문 검토 등과 같은 유효한 검색 옵션의 집합이 표시됩니다. 정의된 QnA 형식을 사용하여 사용자는 SQL Database에 저장된 데이터를 조회할 수 있는 Azure Search에서 지원되는 질문에 쉽게 답할 수 있습니다.

![정보 봇 다이어그램](~/media/scenarios/bot-service-scenario-informational-bot.png)

다음은 정보 봇의 논리 흐름입니다.

1. 직원 정보 봇을 시작합니다.
2. Azure Active Directory가 직원의 ID를 확인합니다.
3. 직원은 봇에게 어떤 유형의 쿼리가 지원되는지 질문할 수 있습니다.
4. Cognitive Services가 QnA Maker로 빌드된 FAQ 봇을 반환합니다.
5. 직원이 유효한 쿼리를 정의합니다.
6. 봇이 쿼리를 Azure Search에 제출하고 응용 프로그램 데이터에 대한 정보가 반환됩니다.
7. Application Insights가 봇 성능과 사용을 통한 개발에 도움이 되도록 런타임 원격 분석 데이터를 수집합니다.

## <a name="sample-bot"></a>샘플 봇
C#으로 작성된 샘플 봇은 SQL Database 인스턴스에서 Azure Search로 인덱싱된 데이터와 작업하는 Microsoft Azure에서 실행됩니다. 봇은 Cognitive Services: QnA Maker를 사용하여 질문(답변)을 표현하는 방법에 대한 정보로 질문할 수 있는 질문 목록을 노출합니다. 그러면 봇 사용자는 인덱싱된 데이터베이스의 광범위한 영역 또는 특정 영역에서 Azure Search를 통해 데이터를 조회하는 쿼리를 입력할 수 있습니다. 샘플은 고객 및 주문 정보가 포함된 간단한 데이터베이스를 제공합니다. Application Insights는 봇 사용량을 추적하고 봇의 예외 상황을 모니터링하는 데 도움을 줍니다. 봇은 Azure AD 앱을 기반으로 게시되기 때문에 정보에 액세스할 수 있는 사람을 제한할 수 있습니다.

이 샘플 봇의 소스 코드는 [일반적인 Bot Framework 시나리오에 대한 샘플](https://aka.ms/bot/scenarios)에서 다운로드하거나 복제할 수 있습니다.

## <a name="components-youll-use"></a>사용할 구성 요소
정보 봇은 다음 구성 요소를 사용합니다.
-   인증에 Azure AD
-   Cognitive Services: QnA Maker
-   Azure Search
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure AD(Azure Active Directory)
Azure Active Directory(Azure AD)는 Microsoft의 다중 테넌트 클라우드 기반 디렉터리 및 ID 관리 서비스입니다. Azure AD를 사용하면 전 세계 수백만 개의 조직에서 사용하는 세계 정상급 ID 관리 솔루션과 쉽고 빠르고 통합할 수 있기 때문 봇 개발자가 봇을 구축하는 데 집중할 수 있습니다. Azure AD 앱을 정의함으로써 자체의 복잡한 인증 및 권한 부여 시스템을 구현하지 않고도, 봇에 액세스할 수 있는 사용자와 노출된 데이터를 제어할 수 있습니다.

### <a name="cognitive-services-qna-maker"></a>Cognitive Services: QnA Maker
Cognitive Services QnA Maker는 사용자가 봇에서 쿼리할 수 있는 FAQ 데이터 소스를 제공하는 데 도움이 됩니다. 다른 시스템에 저장된 방대한 양의 정보에 접근하는 경우 사용자가 정보 소스를 필터링하고 설정하도록 돕는 데 유용합니다. 단일 SQL 데이터베이스는 엄청난 양의 정보를 포함할 수 있고 자유 형식 검색을 적용하면 너무 많은 정보가 검색됩니다. QnA Maker를 처음 사용하면 봇 사용자를 위한 로드맵을 정의하여 지능적으로 질문하는 방법을 알 수 있고, 그런 다음, Azure Search를 통해 검색할 수 있습니다.

### <a name="azure-search"></a>Azure Search
Azure Search는 검색 인덱스를 신속하게 준비하여 사용할 수 있는 앱용 클라우드 검색 서비스입니다. Microsoft Azure를 기반으로 실행되기 때문에 사용량에 따라 쉽게 확장 및 축소할 수 있습니다. 검색 순위를 효과적으로 제어하고 데이터베이스에 숨겨진 데이터를 겉으로 드러내서 검색 결과를 비즈니스 목표에 연결할 수 있습니다.

### <a name="application-insights"></a>Application Insights
Application Insights는 APM(응용 프로그램 성능 관리) 및 즉각적인 분석을 통해 실행 가능한 통찰력을 가져오도록 도와줍니다. 기본적으로 다양한 성능 모니터링, 강력한 경고 및 사용이 간편한 대시보드를 통해 봇을 사용할 수 있도록 보장하고 예상대로 수행할 수 있습니다. 문제가 있는지 신속하게 확인한 다음, 근본 원인을 분석하여 문제를 찾아 해결할 수 있습니다.

## <a name="next-steps"></a>다음 단계
다음으로, 사물 인터넷 봇 시나리오에 대해 알아봅니다.

> [!div class="nextstepaction"]
> [사물 인터넷 봇 시나리오](bot-service-scenario-internet-things.md)
