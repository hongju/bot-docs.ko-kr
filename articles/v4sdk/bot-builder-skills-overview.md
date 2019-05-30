---
title: Bot Framework 기술 개요 | Microsoft Docs
description: Bot Framework 기술에 대해 자세히 알아보기
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8ed5841f9bfe874de26f1aecbb0e4460e541668b
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215294"
---
# <a name="virtual-assistant---skills-overview"></a>가상 도우미 - 기술 개요

> [!NOTE]
> 이 토픽은 SDK v4 버전에 적용됩니다. 

## <a name="overview"></a>개요

개발자는 다시 사용할 수 있는 대화형 기능, 즉 기술을 연결하여 대화형 환경을 구성할 수 있습니다.

엔터프라이즈 차원에서는 서로 다른 팀이 보유한 여러 하위 봇을 하나로 모으거나, 다른 개발자들이 제공한 공통 기능을 더 광범위하게 활용하는 단일 상위 봇을 만들 수도 있습니다. 이 기술 미리 보기를 통해 개발자들은 새 봇을 만들고(보통은 가상 도우미 템플릿을 통해) 모든 디스패치 및 구성 변경 내용을 통합하는 단일 명령줄 작업을 통해 기술을 추가/제거할 수 있습니다.     

기술은 그 자체로 봇이며 원격으로 호출됩니다. 기술 개발자 템플릿(.NET, TS)을 활용하면 새 기술을 용이하게 만들 수 있습니다.

기술의 핵심 설계 목표는 일관된 작업 프로토콜을 유지 관리하고 가능한 일반 V4 SDK 봇에 근접한 개발 환경을 보장하는 것이었습니다. 

![기술 시나리오](./media/enterprise-template/skills-scenarios.png)

## <a name="bot-framework-skills"></a>Bot Framework 기술

현재 Microsoft Graph를 통해 다음 Bot Framework 기술이 제공되며 여러 언어로 사용할 수 있습니다.

![기술 시나리오](./media/enterprise-template/skills-at-build.png)

| Name | 설명 |
| ---- | ----------- |
|[달력 기술](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-calendar.md)|도우미에 달력 기능을 추가합니다. Microsoft Graph 및 Google에서 제공합니다.|
|[이메일 기술](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-email.md)|도우미에 이메일 기능을 추가합니다. Microsoft Graph 및 Google에서 제공합니다.|
|[할 일 기술](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-todo.md)|도우미에 작업 관리 기능을 추가합니다. Microsoft Graph에서 제공합니다.|
|[관심 지점 기술](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-pointofinterest.md)|관심 지점 및 지침을 찾습니다. Azure Maps 및 FourSquare에서 제공합니다.|
|[자동차 기술](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/automotive.md)|자동차 기능 제어를 위한 수평적 업계 기술입니다.|
|[실험 기술](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/experimental.md)|뉴스, 레스토랑 예약 및 날씨|

## <a name="getting-started"></a>시작하기

기존 기술을 활용하고 자체 기술을 빌드하는 방법은 [시작하기](https://github.com/Microsoft/AI/tree/master/docs#tutorials)를 참조하세요.
