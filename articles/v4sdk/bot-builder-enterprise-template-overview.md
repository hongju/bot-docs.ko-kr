---
title: 엔터프라이즈 봇 템플릿 개요 | Microsoft Docs
description: 엔터프라이즈 봇 템플릿의 기능에 대해 알아봅니다.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 43bc3c7606a12084690d71f8b6ea2dc3b2e5984d
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/28/2018
ms.locfileid: "52451995"
---
# <a name="enterprise-bot-template"></a>엔터프라이즈 봇 템플릿 

> [!NOTE]
> 이 토픽은 SDK v4 버전에 적용됩니다. 

고품질 대화형 환경을 만드는 데에는 기본적인 기능 집합이 필요합니다. 뛰어난 대화형 환경을 성공적으로 구축할 수 있도록 하기 위해 엔터프라이즈 봇 템플릿을 만들었습니다. 이 템플릿은 대화형 환경 구축을 통해 식별한 모범 사례와 지원 구성 요소를 모두 제공합니다. 

이 템플릿은 새 봇 프로젝트를 만드는 과정을 크게 간소화합니다. 템플릿에서 [Bot Builder SDK v4](https://github.com/Microsoft/botbuilder) 및 [Bot Builder 도구](https://github.com/Microsoft/botbuilder-tools)를 활용하여 제공하는 기능은 다음과 같습니다.

기능 | 설명 |
------------ | -------------
소개 메시지 | 대화 시작 시 적응형 카드가 있는 소개 메시지입니다. 봇의 기능을 설명하며, 초기 질문을 안내하는 단추를 제공합니다. 개발자는 이 메시지를 적절하게 사용자 지정할 수 있습니다.
자동 입력 표시기  | 대화 중에 시각적 입력 표시기를 보내고 장기 실행 작업에 대해 반복합니다.
.bot 파일 구동 구성 | 봇에 대한 모든 구성 정보(예: LUIS, 디스패처 엔드포인트, Application Insights)는 .bot 파일에 래핑되어 봇 시작을 구동하는 데 사용됩니다.
기본 대화형 의도  | 영어, 프랑스어, 이탈리아어, 독일어, 스페인어 및 중국어로 작성된 기본 의도(인사말, 감사, 도움말, 취소, 등)입니다. 이러한 의도는 쉽게 수정할 수 있는 .LU(언어 이해) 파일로 제공됩니다.
기본 대화형 응답  | 기본 대화형 의도에 대한 응답은 별도의 View 클래스로 추상화됩니다. 이러한 응답은 앞으로 새 LG(언어 세대) 파일로 전환됩니다.
부적절한 콘텐츠 또는 PII(개인 식별 정보) 검색  |미들웨어 구성 요소에서 [Content Moderator](https://azure.microsoft.com/en-us/services/cognitive-services/content-moderator/)를 사용하여 들어오는 대화에서 부적절하거나 PII 데이터를 검색합니다.
스크립트(Transcript)  | Azure Storage에 저장된 모든 대화 내용입니다.
디스패처 | 지정된 발화가 LUIS + 코드에서 처리되어야 하는지, 아니면 QnA Maker에 전달되어야 하는지를 식별하는 통합 [디스패치](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) 모델입니다.
QnA Maker 통합  | 기존 데이터 원본(예: PDF 매뉴얼)을 활용할 수 있는 기술 자료에서 일반적인 질문에 답변할 수 있도록 [QnA Maker](https://www.qnamaker.ai)와 통합됩니다.
대화형 인사이트  | 모든 대화에 대한 원격 분석과 PowerBI 대시보드 예제를 수집하여 대화형 환경에 대한 인사이트로 시작할 수 있도록 [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/)와 통합됩니다.

또한 봇에 필요한 모든 Azure 리소스(봇 등록, Azure App Service, LUIS, QnA Maker, Content Moderator, CosmosDB, Azure Storage 및 Application Insights)는 자동으로 배포됩니다. 더욱이 기본 LUIS, QnA Maker 및 디스패치 모델을 생성, 학습 및 게시하여 기본 의도와 라우팅을 즉시 테스트할 수 있습니다.

템플릿을 만들고 배포 단계가 실행되면 F5 키를 눌러 종단 간 테스트를 수행할 수 있습니다. 이렇게 하면 대화형 환경을 시작하기 위한 견고한 기반을 제공하여 각 프로젝트에서 수행해야 하는 며칠 간의 노력을 줄이고 대화형 품질 표시 기준을 향상시킵니다.

시작하려면 [프로젝트 만들기](bot-builder-enterprise-template-create-project.md)로 계속 진행하세요. 위의 기능을 구동한 모범 사례와 학습에 대한 자세한 내용은 [템플릿 세부 정보](bot-builder-enterprise-template-overview-detail.md) 항목을 참조하세요. 

엔터프라이즈 봇 템플릿에 대한 전체 소스 코드는[이 GitHub 위치](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)에서 사용할 수 있습니다.
