---
title: 대화 디자이너 봇 저장 | Microsoft Docs
description: 대화 디자이너 봇에서 언어 이해 모델을 저장 및 학습하고 음성 인식의 초기화를 설정하는 방법을 알아봅니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 3a911158c379f879c0be604fb5e8ba30ab22ee44
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304122"
---
# <a name="saving-your-conversation-designer-bot"></a>대화 디자이너 봇 저장
> [!IMPORTANT]
> 대화 디자이너는 일부 고객에게 제공되지 않습니다. 대화 디자이너의 가용성에 대한 자세한 내용은 올해 하반기에 제공됩니다.

대화 디자이너에서 작업하는 동안 모든 작업은 브라우저 메모리에 캐시됩니다. 변경 내용을 커밋하려면 왼쪽 탐색 메뉴의 왼쪽 위 모서리에 있는 **저장** 단추를 클릭합니다. 작업 손실을 방지하기 위해 작업을 자주 저장하는 것이 좋습니다. **저장** 단추를 클릭하는 것 외에도, **CTRL + S** 키보드 바로 가기를 사용하여 작업을 저장할 수 있습니다.

## <a name="trains-luis-and-primes-speech-recognition"></a>LUIS 학습 및 음성 인식 초기화 설정

**저장** 단추를 클릭하면 봇에 변경 내용이 저장되고 몇 가지 추가 작업이 수행됩니다. 키보드 바로 가기와 달리, **저장** 단추를 클릭하면 대화 디자이너가 다음 작업을 수행하도록 지시하게 됩니다.

- 봇에 대한 모든 새 LUIS 의도 및 엔터티를 학습하고 Bot Service에서 LUIS 모델을 로컬로 게시합니다(필요한 경우). 이러한 의도는 대화 디자이너 또는 봇의 해당하는 [LUIS](https://luis.ai) 앱에 추가될 수 있습니다.
- 새 LUIS 모델을 사용하도록 대화 런타임을 업데이트합니다.
- 예제 발언을 준비하고 Microsoft Cognitive Services에 전송하여 음성 인식을 초기화하면 이 봇의 음성 인식 정확도가 크게 향상됩니다.

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [봇 테스트](conversation-designer-debug-bot.md)
