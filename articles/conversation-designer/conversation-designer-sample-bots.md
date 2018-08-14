---
title: 샘플 봇에서 대화 디자이너 봇 만들기 | Microsoft Docs
description: 이러한 샘플 봇 중 하나를 사용하여 새 봇을 시작합니다.
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 0ebf0d1b90b03789d8a77710631c83f0914099c5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302523"
---
# <a name="create-a-conversation-designer-bot-from-a-sample-bots"></a>샘플 봇에서 대화 디자이너 봇 만들기

대화 디자이너는 봇을 정의하기 위한 선언적 모델인 시각적 봇 작성기를 제공하는 강력한 프레임워크이자 봇 생성을 간소화하는 런타임입니다. 봇 생성을 간소화하는 한 가지 방법은 기존 봇에서 봇을 시작하는 것입니다.

여러 샘플 봇 중 하나 또는 이전에 내보낸 대화 디자이너 봇에서 대화 디자이너 봇을 만들 수 있습니다. 각 샘플 봇은 완벽하게 작동하는 봇입니다. 이는 고유한 봇을 빌드하거나 시나리오에 맞게 샘플 봇을 사용자 지정할 수 있는 견고한 시작점이 됩니다.

## <a name="the-sample-bots"></a>샘플 봇

대화 디자이너에는 **샘플 봇** 집합이 함께 제공됩니다. 이러한 **샘플 봇** 중 하나를 기반으로 봇 빌드를 시작할 수 있습니다. 이러한 **샘플 봇**의 대부분은 완벽하게 작동하는 봇의 작은 시나리오에 적용됩니다. 이러한 모든 샘플 봇이 함께 작동하는지 확인하려면 **완벽한 Contoso Cafe 봇**을 만듭니다. 

아래 표에는 생성 가능한 봇의 유형과 각 봇에 대한 간략한 설명, 관련 시나리오가 나열되어 있습니다. 

| 봇 이름 | 시나리오 | 도입된 기능 |
| ---- | ---- | ---- |
| **기본 봇** | 몇 가지 기본적인 기능이 연결된 봇입니다. 예를 들어,이 봇은 대화에 새로운 사용자가 추가될 때 환영 메시지를 표시하고, 인사말에 응답하고, 사용자가 봇이 이해하지 못하는 것을 묻는 경우 대체 동작을 실행합니다. | - [작업](conversation-designer-tasks.md) <br/>- 작업 및 동작 트리거 <br/>- [Language Understanding 트리거](conversation-designer-luis.md)<br/>- [스크립트 트리거](conversation-designer-code-recognizer.md)<br/>- [회신 작업](conversation-designer-reply.md) |
| **Echo 봇** | 사용자의 메시지를 되돌려 보내는 **기본 봇**입니다.  봇에게 어떤 메시지를 보내든 "당신이 말한 것은 이렇습니다"라며 메시지를 되돌려 보냅니다. | - [스크립트 트리거](conversation-designer-code-recognizer.md)<br/>- 코드에서 컨텍스트 개체 해석<br/>- 코드에서 엔터티 만들기<br>- 봇의 회신에서 엔터티 사용 |
| **QnA 봇** | [QnAMaker.ai](http://qnamaker.ai)를 사용하여 한 차례 질문 및 답변 환경을 수행할 수 있는 봇입니다. 이 샘플이 작동하려면 [QnA Maker](http://qnamaker.ai) 계정을 설정하고 몇 가지 질문과 답변을 교육시켜야 합니다. 그런 다음, 이 **QnA 봇**을 열고 **스크립트** 페이지로 이동하여 이러한 두 자리 표시자를 QnA Maker의 정보 `<INSERT YOUR KB ID>` 및 `<INSERT YOUR SUBSCRIPTION KEY>`로 바꿉니다. 이러한 두 가지 정보를 입력한 후에는 봇을 [저장](conversation-designer-save-bot.md)한 다음, 봇을 [테스트](conversation-designer-debug-bot.md)할 수 있습니다. | - [스크립트 트리거](conversation-designer-code-recognizer.md)<br/>- 코드에서 HTTP 요청 만들기<br/>- 사용자 지정 NLP/LU 서비스에 연결(샘플의 QnAMaker.ai) |
| **대화 봇** | 간단한 대화에 참여하고 사용자 이름과 같은 컨텍스트를 기억할 수 있는 봇입니다. | - [대화 상자 작업](conversation-designer-dialogues.md)<br/>- 기본 [대화 상자 상태](conversation-designer-dialogues.md#dialogue-states) 및 속성<br/>- [프롬프트 대화 상자](conversation-designer-dialogues.md#prompt-state) 소개(프롬프트 및 재프롬프트 동작 정의)<br/>프롬프트 대화 상자에서 레이블이 지정된 엔터티로 - [의도 만들기](conversation-designer-luis.md#create-a-new-intent)<br/>- 봇의 회신에서 Language Understanding이 생성한 엔터티 사용<br/>- [카드](conversation-designer-adaptive-cards.md)를 사용하여 최종 사용자 환경 개선<br/>- [입력 양식](conversation-designer-adaptive-cards.md#input-form)에 봇 엔터티 바인딩<br/>- [응답 템플릿](conversation-designer-response-templates.md) 봇 자산 생성 및 사용 |
| **메모리 봇** | 사용자의 기본 설정을 요청하고 나중에 대화의 정보를 불러올 수 있는 봇입니다. "통신 기본 설정 지정"과 같은 내용을 말할 때 이 봇은 "통화" 또는 "문자" 중 어떤 방법으로 연락을 받을지 묻습니다. 또한 이 봇에 "카페 위치 찾기" 또는 "워싱턴주 레드몬드의 Contoso Cafe 길 찾기"와 같은 말로 Contoso Cafe 위치 정보를 요청할 수 있습니다. 봇이 정보를 찾으면 통화나 문자를 통해 찾은 정보를 알려줍니다. <br/><br/>통신 기본 설정이 이미 설정된 경우 봇은 해당 기본 설정만 사용하고 그렇지 않으면 사용자에게 기본 설정을 묻습니다. | - [대화 상자 작업](conversation-designer-dialogues.md)<br/>- [`context.global`](conversation-designer-context-object.md#context-properties)을 사용하여 봇 메모리에 데이터 저장<br/>- 메모리에서 대화 상자에서 사용할 컨텍스트 정보 불러오기 |
| **테이블 예약 봇** | 사용자가 Contoso Cafe 매장에서 테이블을 예약하는 데 도움을 주기 위해 사용자와 여러 차례 대화에 참여할 수 있는 봇입니다. <br/><br/>테이블을 예약하려면 (1) *인원 수*, (2) *날짜/시간* 및 (3) *위치*와 같은 세 가지 정보가 필요합니다. <br/><br/>“화요일 오후 2시, 레드몬드 매장에 4인 테이블을 예약하고 싶습니다.”라고 말하면 한 메시지에 세 가지 정보를 모두 제공할 수 있습니다. 세 가지의 정보 중 어느 하나라도 생략하면 봇이 해당 정보를 묻습니다. | - [대화 상자 작업](conversation-designer-dialogues.md)<br/>- 다른 대화 상자를 참조하는 복잡한 대화 상자 작업<br/>- 대화 상자를 함께 구성<br/>- LUIS의 모든 기능을 사용하여 봇의 대화 상자를 설정하여 정방향 슬롯 채우기 및 수정 환경과 같은 작업을 관리<br/>- [카드](conversation-designer-adaptive-cards.md)를 사용하여 최종 사용자 환경 개선<br/>- [입력 양식](conversation-designer-adaptive-cards.md#input-form)에 봇 엔터티 바인딩<br/>- [조건부 응답 템플릿](conversation-designer-response-templates.md#conditional-response-templates) 봇 자산 생성 및 사용 |
| **검색 구체화 봇** | 이전 대화에서 가져온 컨텍스트 정보를 사용하여 검색을 구체화할 수 있는 봇입니다. 사용자는 "이번 주말에 시애틀 매장이 문을 여나요?"와 같이 말한 다음, “렌턴 매장은 어떤가요?”라고 말할 수 있습니다. 그러면 봇이 두 번째 질문도 이번 주말에 대한 내용임을 알 수 있습니다. “이번 주 금요일 오후 8시에 문을 여는 매장은 어디인가요?” 다음에 “오후 10시에 여나요?” 등과 같은 질문을 시도해 볼 수도 있습니다. | - [대화 상자 작업](conversation-designer-dialogues.md)<br/>- [`context.global`](conversation-designer-context-object.md#context-properties)을 사용하여 봇의 메모리에 데이터 저장<br/>- 봇 메모리를 사용하여 컨텍스트에 사용자와의 대화가 정방향으로 제공되도록 함<br/>- 메모리에서 대화 상자에서 사용할 컨텍스트 정보 불러오기 |
| **샌드위치 주문 봇** | 샌드위치 주문 시 사용자에게 입력을 요구하는 메시지를 표시하는 여러 방법을 보여 주는 봇입니다. 예를 들어,이 봇은 샌드위치를 주문하도록 도와줍니다. <br/><br/>샌드위치를 주문하려면 봇이 (1) *샌드위치 크기*, (2) *빵 유형*, (3) *프로틴 옵션* 및 (4) *샌드위치 토핑*과 같은 네 가지 정보를 알아야 합니다. <br/><br/>**테이블 예약 봇**과 마찬가지로, 하나의 메시지에 네 가지 필수 정보를 모두 제공하거나 "샌드위치를 주문하고 싶습니다." 또는 "큰 샌드위치를 주문할 수 있나요?"와 같이 정보를 한 번에 하나씩 제공할 수 있습니다. 그러면 봇이 주문을 처리하는 데 필요한 나머지 정보를 요청할 것입니다. | - [대화 상자 작업](conversation-designer-dialogues.md)<br/>- 여러 방법으로 정보를 수집하도록 [프롬프트](conversation-designer-response-templates.md) 설정 |
| **Contoso Cafe 봇** | Contoso Cafe를 위해 제작된 완전한 기능의 봇입니다. 이 봇은 다른 샘플 봇이 수행할 수 있는 모든 작업을 수행할 수 있습니다. 사용자가 상호 작용할 수 있는 서식 있는 카드를 사용하여 환영 메시지를 제공합니다. 도움말, 인사말 및 대체 메시지를 지원합니다. 사용자는 이를 통해 테이블을 예약하거나 카페 위치를 찾거나 샌드위치를 주문할 수 있습니다. | - 대화 디자이너의 모든 기능<br/>- [`context.global`](conversation-designer-context-object.md#context-properties)을 사용하여 봇의 메모리에 데이터 저장<br/>- [카드](conversation-designer-adaptive-cards.md)를 사용하여 사용자 입력에 따라 특정 봇 작업으로 트리거(환영 메시지에서 단추가 있는 카드 참조)<br/>- 여러 가지 [작업](conversation-designer-tasks.md)을 수행할 수 있는 복잡한 봇 빌드. |

## <a name="explore-sample-bots"></a>샘플 봇 살펴보기

현재 사용 중인 샘플 봇이 원하는 것과 다른 경우 언제든지 다른 샘플 봇으로 전환할 수 있습니다. **샘플 봇 탐색** 페이지에서 모든 샘플 봇의 목록을 찾을 수 있습니다.

> [!WARNING] 
> 샘플 봇을 가져오면 현재 봇이 샘플 봇으로 바뀝니다. 현재 봇의 사용자 지정을 유지하려면 .zip 파일로 [내보내기](conversation-designer-export-import-bot.md#export-a-conversation-designer-bot)합니다.

다른 샘플 봇으로 전환하려면 다음을 수행합니다.
1. 현재 봇에서 **샘플 봇 탐색**을 클릭합니다. 이 옵션은 왼쪽 탐색 패널 맨 아래에 있습니다. 
2. **샘플 봇** 목록에서 봇을 선택하고 **가져오기**를 클릭합니다.
3. 현재 봇에 작업 내용을 저장하려면 **백업 및 가져오기** 옵션을 선택합니다. 이 옵션은 현재 봇을 로컬 컴퓨터에 저장한 다음, 새 **샘플 봇**을 가져옵니다. 그렇지 않으면 **백업하지 않고 가져오기**를 선택합니다.

이제 현재 봇이 방금 전에 선택한 새 샘플 봇으로 바뀝니다. 

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [봇 저장](conversation-designer-save-bot.md)
