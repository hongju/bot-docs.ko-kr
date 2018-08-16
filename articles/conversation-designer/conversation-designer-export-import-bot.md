---
title: 대화 디자이너 봇 가져오기 및 내보내기 | Microsoft Docs
description: 대화 디자이너 봇을 가져오고 내보내는 방법을 알아봅니다.
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: ed055cb0f75d148e6a0dd3248366851901d9a675
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304187"
---
# <a name="export-and-import-a-conversation-designer-bot"></a>대화 디자이너 봇 내보내기 및 가져오기

대화 디자이너에서 봇은 .zip 파일로 컴퓨터로 내보낼 수 있습니다. 따라서 봇을 백업할 수 있습니다. 나중에 내보낸 .zip 파일 중 하나를 사용하여 이전 상태로 봇을 복원할 수 있습니다. 

## <a name="export-a-conversation-designer-bot"></a>대화 디자이너 봇 내보내기기

내보내기를 사용하면 로컬 컴퓨터에 대화 디자이너 봇의 현재 상태를 저장할 수 있습니다. 

대화 디자이너 봇을 내보내려면 다음을 수행합니다.
1. 탐색 패널의 오른쪽 위 모서리에서 줄임표(**...**)를 클릭합니다.
2. **내보내기**를 클릭합니다. 서버에서 필요한 정보를 수집하고 사용자가 .zip 파일을 저장하기 위한 옵션과 함께 반환합니다.
3. 내보낸 .zip 파일을 로컬 컴퓨터에 저장합니다.

이제 봇이 사용자 컴퓨터에 .zip 파일에 저장됩니다. 이를 다시 봇으로 [가져오면](#import-a-conversation-designer-bot) 봇을 나중에 이 상태로 복원할 수 있습니다.

## <a name="import-a-conversation-designer-bot"></a>대화 디자이너 봇 가져오기

가져오기를 사용하면 대화 디자이너 봇을 이전 상태로 복원할 수 있습니다. 가져오기를 수행하면 현재 봇을 가져온 봇으로 바뀝니다. 새로운 현재 봇에 있는 항목을 손실되지 않도록 하려면 가져오기 작업을 수행하기 전에 현재 봇을 [내보냅니다](#export-a-conversation-designer-bot).

대화 디자이너 봇을 가져오려면 다음을 수행합니다.
1. 탐색 패널의 오른쪽 위 모서리에서 줄임표(**...**)를 클릭합니다.
2. **가져오기**를 클릭합니다. 
3. 현재 봇에서 작업 내용을 저장하려면 **백업 및 가져오기** 옵션을 선택합니다. 이 옵션은 로컬 컴퓨터에 현재 봇을 저장한 다음, 가져올 .zip 파일의 위치를 묻는 메시지가 표시됩니다. 그렇지 않으면 **백업하지 않고 가져오기**를 선택합니다.

이제 봇을 가져옵니다.

> [!NOTE]
> 대화 디자이너에서 내보낸 봇만 가져올 수 있습니다.

## <a name="import-a-luis-app-into-a-conversation-designer-bot"></a>대화 디자이너 봇으로 LUIS 앱 가져오기

LUIS 앱이 있고 대화 디자이너 봇에서 사용하려는 경우 대화 디자이너 봇에 LUIS 앱을 가져올 수 있습니다. 개념적으로 이 프로세스를 수행하려면 LUIS 앱 및 대화 디자이너 봇을 내보낸 다음, 봇의 **luis.model** 파일 콘텐츠를 **luis.json** 파일로 바꾸어야 합니다. 그런 다음, 변경 내용을 다시 대화 디자이너 봇으로 가져옵니다. 기본적으로 봇의 LUIS 의도를 LUIS 앱의 의도로 바꿉니다. 이로 인해 봇의 LUIS 의도를 사용자 지정하기 전에 이 가져오기 작업을 수행하는 것이 좋습니다. 그렇지 않은 경우 모든 작업이 이 가져오기 작업으로 덮어쓰기 됩니다.

> [!NOTE]
> 봇에 연결된 [LUIS](https://luis.ai) 앱을 변경하는 경우(각 대화 디자이너 봇에 해당 LUIS 앱이 있음) 이러한 단계를 수행할 필요가 없습니다. 대신 대화 디자이너 봇으로 이동한 다음, [**저장**](conversation-designer-save-bot.md)을 클릭하기만 하면 됩니다.

대화 디자이너 봇으로 LUIS 앱을 가져오려면 다음을 수행합니다.

1. [LUIS.ai](https://luis.ai)에서 LUIS 앱을 연 다음, **설정**을 클릭합니다.
2. 사용할 앱의 **버전**을 선택하고 **{ }** 작업 아이콘을 클릭합니다. 그러면 **luis.json** 파일이 로컬 컴퓨터에 다운로드됩니다. 
3. 대화 디자이너에서 [새 봇을 만들거나](conversation-designer-create-bot.md#create-a-conversation-designer-bot) 기존 봇을 엽니다.
4. 봇을 [내보냅니다](#export-a-conversation-designer-bot). 그러면 봇이 .zip 파일로 사용자 컴퓨터에 내보내기됩니다.
5. 내보낸 .zip 파일로 이동하고 추출합니다.
6. 텍스트 편집기에서 **luis.model** 파일을 열고 이 파일의 콘텐츠를 **luis.json** 파일의 콘텐츠로 바꿉니다. 파일을 저장합니다.
7. 대화 디자이너 봇의 폴더를 압축합니다.
8. 봇을 다시 대화 디자이너 봇으로 [가져옵니다](#import-a-conversation-designer-bot).

이제 봇이 방금 가져온 새 LUIS 의도를 사용할 수 있습니다.

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [작업](conversation-designer-tasks.md)
