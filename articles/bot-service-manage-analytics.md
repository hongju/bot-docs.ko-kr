---
title: 봇 분석 | Microsoft Docs
description: Bot Framework에서 데이터 수집 및 분석을 사용하여 봇의 분석 능력을 개선하는 방법을 알아봅니다.
keywords: 봇 분석, application insights, 트래픽, 대기 시간, 통합, AppInsights
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/04/2018
ms.openlocfilehash: 2f7474500af4305f4c51193a2a5af264d419569b
ms.sourcegitcommit: 91156d0866316eda8d68454a0c4cd74be5060144
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/07/2018
ms.locfileid: "53010518"
---
# <a name="bot-analytics"></a>봇 분석

분석은 [Application Insights](/azure/application-insights/app-insights-analytics)의 확장입니다. Application Insights는 트래픽, 대기 시간 및 통합과 같은 **서비스 수준** 및 계측 데이터를 제공합니다. 또한 분석은 사용자에 대한 **대화 수준** 보고, 메시지 및 채널 데이터를 제공합니다.

## <a name="view-analytics-for-a-bot"></a>봇의 분석 보기

분석에 액세스하려면 Azure Portal에서 봇을 열고 **분석**을 클릭합니다.

데이터가 너무 많나요? 봇에 연결된 Application Insights에 대해 [샘플링](/azure/application-insights/app-insights-sampling)을 사용하도록 설정하고 구성할 수 있습니다. 이렇게 하면 통계적으로 정확한 분석을 유지하면서 원격 분석 트래픽 및 스토리지를 줄입니다.

### <a name="specify-channel"></a>채널 지정

아래 그래프에 표시되는 채널 중에서 선택합니다. 봇이 채널에서 사용되도록 설정되지 않으면 해당 채널에서 데이터가 제공되지 않습니다.

![채널 선택](~/media/analytics-channels.png)

* 차트에 채널을 포함하려면 해당 확인란을 선택합니다.
* 차트에서 채널을 제거하려면 해당 확인란을 선택 취소합니다.

### <a name="specify-time-period"></a>기간 지정

분석은 지난 90일 동안만 사용할 수 있습니다. 데이터 수집은 Application Insights가 사용되도록 설정되었을 때 시작되었습니다.

![기간 선택](~/media/analytics-timepick.png)

드롭다운 메뉴를 클릭하고 그래프에 표시해야 하는 기간을 클릭합니다.
전체 시간 프레임을 변경하면 그래프의 시간 증분(x축)도 그에 따라 변경됩니다.

### <a name="grand-totals"></a>총합계

지정된 시간 프레임 동안 활성 사용자 및 송수신된 활동의 총 수입니다.
대시 `--`는 활동이 없음을 나타냅니다.

### <a name="retention"></a>보존

재방문 주기는 하나의 메시지를 보낸 후 나중에 다시 돌아와 다른 메시지를 보낸 사용자 수를 추적합니다.
이 차트는 10일 간의 규칙적인 기간을 포함합니다. 따라서 시간 프레임을 변경해도 결과는 영향을 받지 않습니다.

![재방문 주기 차트](~/media/analytics-retention.png)

가능한 가장 최근 날짜는 2일 전입니다. 즉, 사용자가 그저께 메시지를 보낸 후 어제 *돌아온* 경우입니다.

### <a name="user"></a>사용자

사용자 그래프는 지정된 시간 프레임 동안 봇이 각 채널을 사용하여 액세스한 사용자 수를 추적합니다.

![사용자 그래프](~/media/analytics-users.png)

* 백분율 차트는 각 채널을 사용한 사용자의 비율을 보여 줍니다.
* 꺾은선형 그래프는 특정 시간에 봇에 액세스하고 있던 사용자 수를 나타냅니다.
* 꺾은선형 그래프의 범례는 어떤 색이 어떤 채널을 나타내는지 지정하고, 지정된 기간 동안의 사용자 총 수를 포함합니다.

### <a name="activities"></a>활동

활동 그래프는 지정된 시간 프레임 동안 얼마나 많은 활동이 어떤 채널을 사용하여 송수신되었는지를 추적합니다.

![활동 그래프](~/media/analytics-activities.png)

* 백분율 차트는 각 채널을 통해 전달된 활동의 비율을 보여 줍니다.
* 꺾은선형 그래프는 지정된 시간 프레임 동안 송수신된 활동 수를 나타냅니다.
* 꺾은선형 그래프의 범례는 어떤 색이 각 채널을 나타내는지를 지정하고, 지정된 기간 동안 해당 채널에서 송수신된 총 활동 수를 나타냅니다.

## <a name="enable-analytics"></a>분석 사용

분석은 Application Insights가 사용되도록 설정되고 구성될 때까지 사용할 수 없습니다. Application Insights는 사용되도록 설정되는 즉시, 데이터 수집을 시작합니다. 예를 들어, Application Insights가 6개월 된 봇에 대해 1주일 전부터 사용되도록 설정된 경우 1주일 동안의 데이터를 수집하게 됩니다.

> [!NOTE]
> 분석을 사용하려면 Azure 구독 및 Application Insights [리소스](/azure/application-insights/app-insights-create-new-resource)가 둘 다 필요합니다.
Application Insights에 액세스하려면 [Azure Portal](https://portal.azure.com/)에서 봇을 열고 **설정**을 클릭합니다.

봇 리소스를 만들 때 Application Insights를 추가할 수 있습니다.

나중에 Application Insights 리소스를 만들어 봇에 연결할 수도 있습니다.

1. Application Insights [리소스](/azure/application-insights/app-insights-create-new-resource)를 만듭니다.
2. 대시보드에서 봇을 엽니다. **설정**을 클릭하고 **분석** 섹션까지 아래로 스크롤합니다.
3. 봇을 Application Insights에 연결하기 위한 정보를 입력합니다. 모든 필드는 필수입니다.

![Insights 연결](~/media/analytics-enable.png)

<!--Snip: As of 12/04/2018, parts of this appear to be out of date. However, ~/bot-service-resources-app-insights-keys.md appears to be up to date.

### AppInsights Instrumentation Key

To find this value, open the Application Insights resource for your bot and navigate to **Configure** > **Properties**.

### AppInsights API key

Provide an Azure App Insights API key. Learn how to [generate a new API key](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID). Only **Read** permission is required.

### AppInsights Application ID

To find this value, open Application Insights and navigate to **Configure** > **API Access**.

/Snip-->

이러한 값을 찾는 방법에 대한 자세한 내용은 [Application Insights 키](~/bot-service-resources-app-insights-keys.md)를 참조하세요.

## <a name="additional-resources"></a>추가 리소스
* [Application Insights 키](~/bot-service-resources-app-insights-keys.md)