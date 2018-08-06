---
title: 대화 디자이너 | Microsoft Docs
description: 대화 디자이너에 대해 알아봅니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 42e9e8137619fff2ca86f82fea1fe81aa348449b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39111560"
---
# <a name="conversation-designer"></a>대화 디자이너
> [!IMPORTANT]
> 대화 디자이너는 일부 고객에게 제공되지 않습니다. 대화 디자이너의 가용성에 대한 자세한 내용은 올해 하반기에 제공됩니다.

대화 디자이너는 봇을 정의하기 위한 선언적 모델인 시각적 개체 봇 작성기를 제공하는 강력한 프레임워크이며 봇을 간소화하는 런타임입니다.

기존 비즈니스 논리를 사용하여 사용자 환경 및 통합에 대한 초점을 유지 관리하는 동시에 유용한 대화형 봇을 빌드하기 위해 대화형 컨트롤, 언어 이해, 대화 상자 및 언어 생성을 통합하는 통합된 접근 방법이 필요합니다. 대화 디자이너를 사용하면 작성, 디버그, LUIS, 분석, 음성 인식 및 기존 Bot Framework SDK 봇과의 호환성에 걸쳐 통합된 경로를 사용하여 봇을 빌드할 수 있습니다.

대화 디자이너를 사용하여 사용 가능한 입출력 양상에 쉽게 적용하고 다음과 같은 기능을 활용하는 봇을 빌드할 수 있습니다. 

- 상태 관리 오버헤드를 최소화하는 강력한 런타임
- 대화를 편리하게 시각적 모델링할 수 있는 기본 제공 대화 상태
- 코드와 봇 UI의 분리
- <a href="https://luis.ai" target="_blank">LUIS</a> 및 <a href="https://www.microsoft.com/cognitive-services/en-us/speech-api" target="_blank">음성 인식</a>과 같은 AI 프레임워크에 대한 기본 제공 지원
- 간단한 조건부 응답 템플릿을 사용하는 언어 생성에 대한 기본 제공 지원
- [적응 카드](conversation-designer-adaptive-cards.md)에 대한 지원

## <a name="prerequisites"></a>필수 조건

- 대화 디자이너에는 Azure 구독이 필요합니다. <a href="https://azure.microsoft.com/en-us/" target="_blank">여기</a>에서 시작하면 됩니다.
- 아직 수행하지 않았다면 이 기능을 사용하여 계정을 만든 후에 적어도 한 번 [LUIS 포털](https://luis.ai)에 로그인해야 합니다.
- JavaScript 프로그래밍에 익숙해야 합니다. 사용자 지정 스크립트 함수는 JavaScript로 작성됩니다.
- Microsoft Edge 또는 Google Chrome

## <a name="next-steps"></a>다음 단계
> [!div class="nextstepaction"]
> [봇 만들기](conversation-designer-create-bot.md)

## <a name="additional-resources"></a>추가 리소스
대화 디자이너를 사용하여 봇을 빌드하는 방법에 대해 자세히 알아봅니다.
- [Language Understanding](conversation-designer-luis.md): 봇에 대한 Language Understanding을 설정합니다.
- [대화 상자](conversation-designer-dialogues.md): 봇에 대한 대화 상자를 설정합니다.
- [응답 템플릿](conversation-designer-response-templates.md): 봇에 대한 응답 템플릿을 설정하는 방법에 대한 빠른 팁입니다.
- [적응 카드](conversation-designer-adaptive-cards.md): 적응 카드를 사용하여 봇에 대해 풍부한 시각적 개체 상호 작용을 설정합니다.
