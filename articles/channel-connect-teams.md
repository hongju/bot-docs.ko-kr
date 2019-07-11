---
title: Teams에 봇 연결 | Microsoft Docs
description: Team을 통해 액세스할 수 있도록 봇을 구성하는 방법을 알아봅니다.
keywords: Teams, 봇 채널, Teams 구성
author: kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
ms.openlocfilehash: d2609e4294416691e156ba3dbd09eabc8e0d3423
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/05/2019
ms.locfileid: "67592232"
---
# <a name="connect-a-bot-to-teams"></a>Teams에 봇 연결

Microsoft Teams 채널을 추가하려면 [Azure Portal](https://portal.azure.com)에서 봇을 열고 **채널** 블레이드를 클릭한 다음, **Teams**를 클릭합니다.

![Teams 채널 추가](media/teams/connect-teams-channel.png)

그런 다음, **저장**을 클릭합니다.

![Teams 채널 저장](media/teams/save-teams-channel.png)

Teams 채널을 추가한 후 **채널** 페이지로 이동하여 **봇 embed 태그 가져오기**를 클릭합니다.

![embed 태그 가져오기](media/teams/get-embed-code.png)

- **봇 embed 태그 가져오기** 대화 상자에 표시되는 코드의 _https_ 부분을 복사합니다. 예: `https://teams.microsoft.com/l/chat/0/0?users=28:b8a22302e-9303-4e54-b348-343232` 

- 브라우저에서 이 주소를 붙여넣은 다음, Teams에 봇을 추가하는 데 사용하는 Microsoft Teams 앱(클라이언트 또는 웹)을 선택합니다. Microsoft Teams에서 메시지를 보내고 받을 수 있는 연락처로 나열된 봇을 볼 수 있어야 합니다. 

## <a name="additional-information"></a>추가 정보
Microsoft Teams별 정보는 Teams [설명서](https://docs.microsoft.com/en-us/microsoftteams/platform/overview)를 참조하세요. 
