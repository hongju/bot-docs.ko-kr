---
title: Application Insights 키 | Microsoft Docs
description: Application Insights 키를 가져와 봇에 원격 분석을 추가하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 87cdd27a3a413ae200273090d4a79b5edbc55dc1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996910"
---
# <a name="application-insights-keys"></a>Application Insights 키

Azure **Application Insights**는 Microsoft Azure 리소스에 애플리케이션에 대한 데이터를 표시합니다. 봇에 원격 분석을 추가하려면 봇에 대해 생성된 Application Insights 리소스 및 Azure 구독이 필요합니다. 이 리소스에서 봇을 구성할 다음 세 개의 키를 가져올 수 있습니다.

1. 계측 키
2. 애플리케이션 UI
3. API 키

이 항목에서는 이러한 Application Insights 키를 만드는 방법을 보여 줍니다.

> [!NOTE]
> 봇을 만들거나 등록하는 동안 **Application Insights** *켜기* 또는 *끄기* 옵션이 제공되었습니다. Application Insights를 *켠* 경우, 봇에 필요한 모든 Application Insights 키가 이미 있습니다. 그러나 Application Insights를 *끈* 경우 이 항목의 지침에 따라 해당 키를 수동으로 만들 수 있습니다.

## <a name="instrumentation-key"></a>계측 키

계측 키를 가져오려면 다음을 수행합니다.
1. [Azure portal](http://portal.azure.com)의 [모니터] 섹션에서 새 **Application Insights** 리소스를 만들거나 기존 리소스를 사용합니다.
![Application Insights 목록의 포털 화면 캡처](~/media/portal-app-insights-add-new.png)

2. Application Insights 리소스 목록에서 방금 만든 Application Insight 리소스를 클릭합니다.

3. **개요**를 클릭합니다.

4. **기본 정보** 블록을 확장하고 **계측 키**를 찾습니다. 
![개요의 포털 화면 캡처](~/media/portal-app-insights-instrumentation-key-dropdown.png)
![계측 키의 포털 화면 캡처](~/media/portal-app-insights-instrumentation-key.png)

5. **계측 키**를 복사하고 봇 설정의 **Application Insights 계측 키** 필드에 붙여넣습니다.

## <a name="application-id"></a>애플리케이션 UI

애플리케이션 ID를 가져오려면 다음을 수행합니다.
1. Application Insights 리소스에서 **API 액세스**를 클릭합니다.

2. **응용 프로그램 ID**를 복사하고 봇 설정의 **Application Insights 응용 프로그램 ID** 필드에 붙여넣습니다. 
![응용 프로그램 ID의 포털 화면 캡처](~/media/portal-app-insights-appid.png)

## <a name="api-key"></a>API 키

API 키를 가져오려면 다음을 수행합니다.
1. Application Insights 리소스에서 **API 액세스**를 클릭합니다.

2. **API 키 만들기**를 클릭합니다.

3. 간단한 설명을 입력하고 **원격 분석 읽기** 옵션을 선택한 다음, **키 생성** 단추를 클릭합니다.
![응용 프로그램 ID 및 API 키의 포털 화면 캡처](~/media/portal-app-insights-appid-apikey.png)

   > [!WARNING]
   > 이 **API 키**는 다시 표시되지 않으므로 키를 복사하고 저장합니다. 이 키를 분실하면 새로 만들어야 합니다.

4. 봇 설정의 **Application Insights API 키** 필드에 API 키를 복사합니다.

## <a name="additional-resources"></a>추가 리소스
이러한 필드를 봇 설정에 연결하는 방법에 대한 자세한 내용은 [분석 사용](~/bot-service-manage-analytics.md#enable-analytics)을 참조하세요.