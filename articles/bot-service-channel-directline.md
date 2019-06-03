---
title: Direct Line 채널 정보
titleSuffix: Bot Service
description: Direct Line 채널 기능
services: bot-service
author: ivorb
manager: kamrani
ms.service: bot-service
ms.subservice: bot-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: v-ivorb
ms.openlocfilehash: 0b108d90d18261cd22214db9a7926bdac1bfee40
ms.sourcegitcommit: a4181f35dbe6a8b107eea28122372f524e19880a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/02/2019
ms.locfileid: "65030196"
---
## <a name="about-direct-line"></a>Direct Line 정보

Bot Framework Direct Line 채널을 사용하면 봇을 모바일 앱, 웹 페이지 또는 기타 애플리케이션에 간편하게 통합할 수 있습니다.
Direct Line은 다음 세 가지 형태로 제공됩니다.
- Direct Line 서비스 - 대부분의 개발자를 위한 강력한 글로벌 서비스
- Direct Line App Service 확장 - 보안 및 성능 전담 Direct Line 기능(2019년 5월부터 프라이빗 미리 보기로 제공)
- Direct Line Speech – 고성능 음성 기능을 위해 최적화(2019년 5월부터 프라이빗 미리 보기로 제공)

각 제공과 솔루션 요구를 평가하여 가장 적합한 Direct Line 제공을 선택할 수 있습니다. 앞으로 이러한 제공은 간소화될 것입니다.

|                            | 직접 회선 | Direct Line App Service 확장 | Direct Line Speech |
|----------------------------|-------------|-----------------------------------|--------------------|
| GA 가용성 및 SLA    | 일반 공급 | 프라이빗 미리 보기, SLA 없음  | 프라이빗 미리 보기, SLA 없음 |
| 음성 인식 및 텍스트 음성 변환 성능 | Standard | Standard | 고성능 |
| 통합 OAuth           | 예 | 예 | 아니요 |
| 통합 원격 분석       | 예 | 예 | 아니요 |
| 레거시 웹 브라우저 지원 | 예 | 아니요 | 아니요 |
| Bot Framework SDK 지원 | 모든 v3, v4 | v4.5+ 필요 | v4.5+ 필요 |
| 클라이언트 SDK 지원    | JS, C# | JS, C# | C++, C#, Unity |
| 웹 채팅 작동  | 예 | 예 | 아니요|
| 대화 기록 API | 예 | 예| 아니요|
| VNET | 아니요 | 미리 보기* | 아니요 |

_* Direct Line App Service 확장은 VNET에서 사용할 수 있지만 아웃바운드 호출 제한은 아직 허용되지 않습니다._

## <a name="addtional-resources"></a>추가 리소스
- [직접 회선에 봇 연결](bot-service-channel-connect-directline.md)
- [Direct Line Speech에 봇 연결](bot-service-channel-connect-directlinespeech.md)
