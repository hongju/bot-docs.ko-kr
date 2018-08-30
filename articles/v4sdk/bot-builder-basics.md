---
title: Bot Builder 기본 사항 | Microsoft Docs
description: Bot Builder SDK 기본 구조입니다.
keywords: 순서 컨텍스트, 봇 구조, 입력 받기
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 34564b411f911ae82197d5a34cb954a103abe70b
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905417"
---
# <a name="basic-bot-structure"></a>기본 봇 구조

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Azure Bot Service 및 Bot Builder SDK는 봇을 빌드하고 디버그하는 데 도움이 되는 라이브러리, 샘플 및 도구를 제공합니다. 그러나 그 중 하나에 대해 자세히 살펴보기 전에 봇의 기본 구조 및 작동 방식을 이해해야 합니다. 이러한 개념은 선택한 프로그래밍 언어에 상관없이 적용됩니다. 전체에 걸쳐 자세한 콘텐츠에 대한 링크가 제공됩니다. 이 문서에서는 봇의 작동 방식에 대한 초기 멘탈 프레임워크를 제공합니다.

처음부터 봇의 기본 구조를 살펴보겠습니다.

## <a name="creation-of-your-bot"></a>봇 만들기

봇은 [Azure Portal](~/bot-service-quickstart.md) 내에서, [Visual Studio](~/dotnet/bot-builder-dotnet-sdk-quickstart.md)에서 또는 [JavaScript](~/javascript/bot-builder-javascript-quickstart.md), [Java](~/java/bot-builder-java-quickstart.md) 또는 [Python](~/python/bot-builder-python-quickstart.md)에 대한 명령줄 도구를 통해 다양한 방법으로 만들어질 수 있습니다. 만들어진 후 로컬 머신, Azure 또는 다른 클라우드 서비스에서 봇을 실행할 수 있습니다. 실행되는 위치 또는 빌드되는 방법에 관계없이 매우 비슷하게 작동합니다.

## <a name="interaction-with-your-bot"></a>봇과의 상호 작용

봇은 기본적으로 웹 사이트 또는 앱이 수행하는 방식으로 표시되는 UI가 없으며, 대신 사용자는 [대화](~/v4sdk/bot-concepts.md#activities-and-conversations)를 통해 상호 작용합니다. 봇에 연결하는 데 사용되는 응용 프로그램에 따라([채널](~/v4sdk/bot-concepts.md)이라고 하지만 여기에서는 다루지 않음) 특정 정보가 사용자와 봇 간에 전송됩니다.

정보의 각 단위는 봇 내에서 **작업**이라고 하며, 작업은 다양한 양식으로 제공될 수 있습니다. 작업은 **메시지**라고 하는 사용자의 통신 또는 몇몇 다른 [작업 유형](~/bot-service-activities-entities.md)에서 래핑된 추가 정보를 포함합니다. 이 추가 정보는 새 파티가 대화에 조인하거나 떠나는 경우, 대화가 종료하는 경우 등을 포함할 수 있습니다. 사용자의 연결에서 이러한 유형의 통신은 사용자가 작업을 수행할 필요 없이 기본 시스템에서 전송됩니다.

봇은 올바른 형식으로 작업 개체에서 통신을 받고 래핑하여 봇 코드에 제공합니다. 다른 모든 작업 유형은 유용한 정보를 제공하지만 가장 흥미롭고 가장 일반적인 작업은 사용자로부터의 **메시지** 작업입니다.

봇에서 받는 각 작업은 순서대로 시작합니다. 곧 자세히 설명하겠습니다.

## <a name="receiving-user-input"></a>사용자 입력 받기

사용자로부터 메시지 작업을 받으면 보내는 것을 이해해야 합니다. 가장 간단한 방법은 단순히 들어오는 메시지 텍스트를 문자열과 일치하도록 하는 것입니다. 문자열에 기반하여 봇이 달성하기 위해 시도하는 것에 따라 작업을 수행하도록 선택할 수 있습니다. 이는 사용자에 대한 응답, 일부 변수 또는 리소스 업데이트, [저장소](~/v4sdk/bot-builder-storage-concept.md)에 저장 또는 유사한 처리를 포함할 수 있습니다.

[LUIS](~/v4sdk/bot-builder-concept-luis.md) 또는 [QnA Maker](~/v4sdk/bot-builder-howto-qna.md)를 사용하는 것과 같은 사용자 입력을 인식하는 더욱 복잡한 방법이 있지만 문자열 일치가 가장 간단합니다.

## <a name="defining-a-turn"></a>순서 정의

[!INCLUDE [Define a turn](~/includes/snippet-definition-turn.md)]

작업 처리는 [작업 처리](~/v4sdk/bot-builder-concept-activity-processing.md)에 자세히 설명된 대로 **어댑터**에서 관리됩니다. 어댑터에서 작업을 받으면 **순서 컨텍스트**를 만들어 작업에 대한 정보를 제공하고 처리 중인 현재 순서에 컨텍스트를 제공합니다. 해당 순서 컨텍스트는 순서의 기간 동안 존재한 다음, 순서의 끝을 표시하여 삭제됩니다.

[순서 컨텍스트](~/v4sdk/bot-builder-concept-activity-processing.md#turn-context)는 몇 가지 정보를 포함하며 봇의 모든 계층에서 동일한 개체가 사용됩니다. 이 순서 컨텍스트 개체는 나중의 순서에서 필요할 수 있는 정보를 저장하는 데 사용될 수 있으며, 사용되어야 하므로 유용합니다.

> [!IMPORTANT]
> 모든 **순서**는 자체적으로 실행되어 서로 독립적이며, 겹칠 수 있습니다. 봇은 다양한 채널의 다양한 사용자에서 한 번에 여러 순서를 처리할 수 있습니다. 각 순서는 고유한 순서 컨텍스트를 갖지만 일부 상황에서 소개하는 복잡성을 고려할 가치가 있습니다.

## <a name="where-to-go-from-here"></a>여기에서 이동할 위치

이 문서는 [작업이 처리되는](~/v4sdk/bot-builder-concept-activity-processing.md) 방식, 다양한 [대화 형식](~/v4sdk/bot-builder-conversations.md), 인텔리전트 대화에 대한 [대화의 상태](~/v4sdk/bot-builder-storage-concept.md) 트랙 유지 등과 같은 많은 세부 정보를 언급하지 않았습니다. 개념 항목의 나머지 부분은 기본 사항을 빌드하고, 봇과 Azure Bot Service를 이해하는 데 필요한 나머지 개념을 다룹니다. 다음 단계 섹션을 따라 정보를 구축하거나 호기심을 불러일으키는 항목으로 이동하고, [빠른 시작](~/bot-service-quickstart.md)을 시도하여 첫 번째 봇을 빌드하거나 봇을 [개발](~/v4sdk/bot-builder-howto-send-messages.md)하는 방법에 대해 자세히 알아볼 수 있습니다.

## <a name="next-steps"></a>다음 단계

다음으로 Bot Connector Service를 통해 봇은 다른 플랫폼의 사용자와 통신할 수 있습니다.

> [!div class="nextstepaction"]
> [채널 및 Bot Connector Service](~/v4sdk/bot-concepts.md)
