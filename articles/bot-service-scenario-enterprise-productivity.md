---
title: 엔터프라이즈 생산성 봇 시나리오 | Microsoft Docs
description: Bot Framework를 사용하여 엔터프라이즈 생산성 봇 시나리오를 살펴봅니다.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0dc869aa1464c086b6596ee83d8e6e488d8a8a55
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302530"
---
# <a name="enterprise-productivity-bot-scenario"></a>엔터프라이즈 생산성 봇 시나리오
엔터프라이즈 봇은 Office 365 일정 및 기타 서비스와 봇을 통합하여 생산성을 높일 수 있는 방법을 보여 줍니다.

창을 열지 않고도 고객 정보에 빠르게 액세스할 수 있는 것이 바로 엔터프라이즈 생산성 봇의 핵심입니다. 판매 담당자는 간단한 채팅 명령을 사용하여 고객을 조회하고 Graph API 및 Office 365를 통해 다음 약속을 확인할 수 있습니다. 여기에서 Dynamics CRM에 저장되는 고객별 정보(예: 사례 접수 또는 새 계정 만들기)에 액세스할 수 있습니다.

![엔터프라이즈 봇 다이어그램](~/media/scenarios/bot-service-scenario-enterprise-bot.png)

엔터프라이즈 생산성 봇의 논리 흐름은 다음과 같습니다.

1. 직원이 엔터프라이즈 생산성 봇에 액세스합니다.
2. Azure Active Directory가 직원의 ID를 확인합니다.
3. 엔터프라이즈 생산성 봇은 Azure Graph를 통해 직원의 Office 365 일정을 쿼리할 수 있습니다.
4. 봇이 일정에서 수집된 데이터를 사용하여 Dynamics CRM의 사례 정보에 액세스합니다.
5. 정보가 직원에게 반환되며, 직원은 봇을 나가지 않고 데이터를 필터링할 수 있습니다.
6. Application Insights가 개발에 도움이 될 봇 성능 및 사용과 관련한 런타임 원격 분석 정보를 수집합니다.

이 샘플 봇의 소스 코드는 [일반적인 Bot Framework 시나리오에 대한 샘플](https://aka.ms/bot/scenarios)에서 다운로드하거나 복제할 수 있습니다.

## <a name="sample-bot"></a>샘플 봇
봇은 다양한 채널에서 액세스할 수 있기 때문에, 인증만 하면 자리에 있을 때는 회사 포털을 사용하고 이동 중에는 Skype를 사용하여 액세스할 수 있습니다. Azure AD를 통합하고 액세스할 수 있으면 엔터프라이즈 생산성 봇에서 사용자가 Azure AD에서 인증되었음을 알 수 있습니다. 여기에서 특정 고객과 다음 약속이 언제 있는지 봇에게 확인을 요청할 수 있습니다. 봇은 Graph API를 통해 Office 365를 쿼리하여 이 정보를 가져옵니다. 그런 다음, 앞으로 7일 이내에 약속이 있는 경우 CRM을 쿼리하여 고객에 대한 최근 사례를 찾습니다. 봇은 사례를 찾을 수 없다고 응답하거나 열려 있거나 닫힌 사례의 수로 응답합니다. 여기에서 유형별로 사례를 나열하도록 봇에게 요청하고, 개별 사례를 자세히 살펴볼 수 있습니다.

## <a name="components-youll-use"></a>사용할 구성 요소
엔터프라이즈 생산성 봇은 다음 구성 요소를 사용합니다.
-   인증용 Azure AD
-   Office 365에 대한 Graph API
-   Dynamics CRM
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure AD(Azure Active Directory)
Azure Active Directory(Azure AD)는 Microsoft의 다중 테넌트 클라우드 기반 디렉터리 및 ID 관리 서비스입니다. 봇 개발자인 Azure AD를 통해 전 세계 수백 만 개 조직에서 사용하는 세계 최고의 ID 관리 솔루션과 빠르고 쉽게 통합함으로써, 봇을 구축하는 데 집중할 수 있습니다. Azure AD 앱을 정의하면 자체의 복잡한 인증 및 권한 부여 시스템을 구현하지 않고도, 봇에 액세스할 수 있는 사용자와 노출되는 데이터를 제어할 수 있습니다.

### <a name="graph-api-to-office-365"></a>Office 365에 대한 Graph API
Microsoft Graph는 https://graph.microsoft.com의 단일 엔드포인트를 통해 Office 365의 여러 API 및 기타 Microsoft Cloud Services를 제공합니다. Microsoft Graph를 사용하면 사용자와 봇이 쿼리를 더 쉽게 실행할 수 있습니다. API는 Office 365에 포함된 Exchange Online, Azure Active Directory, SharePoint를 비롯한 여러 Microsoft Cloud Services의 데이터를 제공합니다. API를 사용하여 엔터티와 관계를 탐색할 수 있습니다. SDK 또는 REST 엔드포인트를 사용하는 봇뿐만 아니라 Android, iOS, Ruby, UWP, Xamarin 등이 기본 지원되는 다른 앱에서도 API를 사용할 수 있습니다.

### <a name="dynamics-crm"></a>Dynamics CRM
Dynamics CRM은 고객 참여 플랫폼입니다. 봇 및 CRM의 API를 사용하여 CRM에 저장된 다양한 데이터에 액세스할 수 있는 풍부한 대화형 봇을 빌드할 수 있습니다. 사례 생성, 상태 확인, 지식 관리 검색 등 Dynamics CRM의 강력한 기능이 봇에 제공됩니다.

### <a name="application-insights"></a>Application Insights
Application Insights는 APM(응용 프로그램 성능 관리) 및 즉각적인 분석을 통해 실행 가능한 통찰력을 가져오도록 도와줍니다. 기본적으로 다양한 성능 모니터링, 강력한 경고 및 사용이 간편한 대시보드를 통해 봇의 가용성과 올바른 작동을 보장할 수 있습니다. 문제가 있는지 신속하게 확인한 다음, 근본 원인을 분석하여 문제를 찾아 해결할 수 있습니다.

## <a name="next-steps"></a>다음 단계
다음으로, 정보 봇 시나리오에 대해 알아봅니다.

> [!div class="nextstepaction"]
> [정보 봇 시나리오](bot-service-scenario-informational.md)
