---
title: Bot Framework Emulator를 사용한 봇 테스트 및 디버그 | Microsoft Docs
description: Bot Framework Emulator 데스크톱 애플리케이션을 사용하여 봇을 검사, 테스트 및 디버그하는 방법을 살펴봅니다.
keywords: 기록, msbot 도구, 언어 서비스, 음성 인식
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/26/2019
ms.openlocfilehash: 307a6bf697e274391336a0d216c64da85232616d
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033310"
---
# <a name="debug-with-the-emulator"></a>에뮬레이터를 사용하여 디버그

Bot Framework Emulator는 봇 개발자가 로컬 또는 원격으로 봇을 테스트 및 디버그할 수 있는 데스크톱 애플리케이션입니다. 에뮬레이터를 사용하면 봇과 채팅하고 봇이 보내고 받는 메시지를 검사할 수 있습니다. 에뮬레이터는 봇과 메시지를 교환할 때 메시지가 웹 채팅 UI에 표시되는 대로 메시지를 표시하며 JSON 요청과 응답을 기록합니다. 봇을 클라우드에 배포하기 전에 에뮬레이터를 사용하여 로컬로 실행하고 테스트합니다. 아직 Azure Bot Service를 사용하여 봇을 [만들지](./bot-service-quickstart.md) 않았거나 채널에서 실행되도록 구성하지 않은 경우에도 에뮬레이터를 사용하여 봇을 테스트할 수 있습니다.

## <a name="prerequisites"></a>필수 조건
- [에뮬레이터](https://aka.ms/Emulator-wiki-getting-started) 설치

## <a name="connect-to-a-bot-running-on-localhost"></a>로컬 호스트에서 실행되는 봇에 연결

![에뮬레이터 UI](media/emulator-v4/emulator-welcome.png)

로컬로 실행되는 봇에 연결하려면 **봇 열기**를 클릭하거나 미리 구성한 구성 파일(.bot 파일)을 선택합니다. 봇 연결에 구성 파일이 필요한 것은 아니지만 구성 파일이 있다면 에뮬레이터가 해당 파일에서 작동합니다. 봇이 MSA(Microsoft 계정) 자격 증명으로 실행 중이라면 이 자격 증명도 입력합니다.

![에뮬레이터 UI](media/emulator-v4/emulator-open-bot.png)

## <a name="view-detailed-message-activity-with-the-inspector"></a>검사기를 사용 하여 자세한 메시지 활동 보기

봇에 메시지를 보내면 봇이 응답합니다. 대화 창에서 메시지 풍선을 클릭하고 창의 오른쪽에 있는 **검사기** 기능을 사용하여 원시 JSON 활동을 검사할 수 있습니다. 메시지 풍선을 선택하면 해당 풍선이 노란색으로 바뀌며 활동 JSO 개체가 채팅 창 왼쪽에 표시됩니다. JSON 정보에는 채널 ID, 활동 유형, 대화 ID, 텍스트 메시지, 엔드포인트 URL 등과 같은 주요 메타데이터가 포함됩니다. 사용자가 보낸 활동과 봇이 응답한 활동을 모두 살펴볼 수 있습니다. 

![에뮬레이터 메시지 활동](media/emulator-v4/emulator-view-message-activity-03.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>봇 기록으로 대화 저장 및 로드

에뮬레이터의 활동은 기록으로 저장할 수 있습니다. 열려 있는 라이브 채팅 창에서 기록 파일에 **기록으로 저장**을 선택합니다. 언제든 **다시 시작** 단추를 사용하여 대화를 지우고 봇 연결을 다시 시작할 수 있습니다.  

![에뮬레이터 - 기록 저장 ](media/emulator-v4/emulator-save-transcript.png)

기록을 로드하려면 **파일 > 기록 파일 열기**를 선택하고 기록을 선택하면 됩니다. 새 기록 창이 열리고 메시지 활동을 출력 창에 렌더링합니다. 

![에뮬레이터 - 기록 로드](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>서비스 추가 

에뮬레이터에서 직접 LUIS 앱, QnA 기술 자료 또는 디스패치 모델을 봇에 직접 등록할 수 있습니다. 봇이 로드되면 에뮬레이터 창 맨 왼쪽의 서비스 단추를 선택합니다. **서비스** 메뉴 아래에 LUIS, QnA Maker, 디스패치를 추가하는 옵션이 표시됩니다. 

서비스 앱을 추가하려면 **+** 단추를 클릭하고 추가하려는 서비스를 선택합니다. 봇 파일에 서비스를 추가하려면 Azure Portal에 로그인하라는 메시지가 표시되면 서비스를 봇 애플리케이션에 연결합니다. 

> [!IMPORTANT]
> 서비스 추가는 `.bot` 구성 파일을 사용 중인 경우에만 작동합니다. 서비스는 따로 추가해야 합니다. 자세한 정보는 [봇 리소스 관리](v4sdk/bot-file-basics.md) 또는 추가하려는 서비스에 대한 개별 방법 문서를 참조하세요.
>
> `.bot` 파일을 사용하지 않는 경우 왼쪽 창에 서비스가 나열되지 않으며(봇이 서비스를 사용하더라도) *서비스를 사용할 수 없음*이 표시됩니다.

![LUIS 연결](media/emulator-v4/emulator-connect-luis-btn.png)

서비스가 연결되면 라이브 채팅 창으로 돌아가 서비스가 연결되어 작동 중임을 확인할 수 있습니다. 

![QnA 연결됨](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-services"></a>서비스 검사

새 v4 에뮬레이터를 사용하여 LUIS 및 QnA에서 JSON 응답도 검사할 수 있습니다. 연결된 언어 서비스가 있는 봇을 사용하여 LOG 창의 오른쪽 아래에서 **추적**을 선택할 수 있습니다. 이 새로운 도구는 에뮬레이터에서 직접 언어 서비스를 업데이트하는 기능도 제공합니다. 

![LUIS 검사기](media/emulator-v4/emulator-luis-inspector.png)

연결된 LUIS 서비스를 통해 추적 링크가 **Luis 추적**을 지정하는 것을 볼 수 있습니다. 선택하면 의도, 엔터티, 지정된 점수를 포함하는 LUIS 서비스로부터의 원시 응답이 표시됩니다. 사용자 표현에 대한 의도를 다시 할당하는 옵션도 있습니다. 

![QnA 검사기](media/emulator-v4/emulator-qna-inspector.png)

연결된 QnA 서비스를 통해 로그는 **QnA 추적**을 표시하며 선택한 경우 해당 활동과 관련한 질문 및 답변 쌍을 신뢰도 점수와 함께 미리 볼 수 있습니다. 여기에서 답변에 대한 대체 질문 문구를 추가할 수 있습니다.

<!--## Configure ngrok

If you are using Windows and you are running the Bot Framework Emulator behind a firewall or other network boundary and want to connect to a bot that is hosted remotely, you must install and configure **ngrok** tunneling software. The Bot Framework Emulator integrates tightly with ngrok tunnelling software (developed by [inconshreveable][inconshreveable]), and can launch it automatically when it is needed.

Open the **Emulator Settings**, enter the path to ngrok, select whether or not to bypass ngrok for local addresses, and click **Save**.

![ngrok path](media/emulator-v4/emulator-ngrok-path.png)
-->

## <a name="login-to-azure"></a>Azure에 로그인

Azure 계정에 로그인하는 데 에뮬레이터를 사용할 수 있습니다. 특히 봇이 종속된 서비스를 추가 및 관리할 때 유용합니다. 에뮬레이터로 관리할 수 있는 서비스에 대한 자세한 정보는 [위](#add-services)를 참조하세요.

### <a name="to-login"></a>로그인하기

![Azure 로그인](media/emulator-v4/emulator-azure-login.png)

로그인하기
- 파일 -> Azure 로그인을 클릭할 수 있음
- 시작 화면에서 Azure 로그인 계정을 클릭합니다. 에뮬레이터 애플리케이션을 다시 시작해도 로그인 상태를 유지하도록 선택할 수 있습니다.

![Azure 로그인](media/emulator-v4/emulator-azure-login-success.png)

## <a name="disabling-data-collection"></a>데이터 수집 비활성화

에뮬레이터의 사용 데이터 수집을 중지하게 하려면 다음 단계에 따라 데이터 수집을 간단히 해제할 수 있습니다.

1. 왼쪽의 탐색 모음에서 설정 버튼(기어 아이콘)을 클릭하여 에뮬레이터의 설정 페이지로 이동합니다.

    ![데이터 수집 사용 안 함](media/emulator-v4/emulator-disable-data-1.png)

2. **데이터 수집** 아래에 이름이 *사용 데이터 수집을 허용하여 에뮬레이터 개선 참여*인 확인란을 선택 취소합니다.

    ![데이터 수집 사용 안 함](media/emulator-v4/emulator-disable-data-2.png)

3. "저장" 단추를 클릭합니다.

    ![데이터 수집 사용 안 함](media/emulator-v4/emulator-disable-data-3.png)
    
상황이 바뀌면 언제든 확인란을 다시 선택하여 사용할 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

Bot Framework Emulator는 오픈 소스입니다. [버그 및 제안을 제출][EmulatorGithubBugs]하여 개발에 [기여][EmulatorGithubContribute]할 수 있습니다.

문제를 해결하려면 [일반 문제 해결](bot-service-troubleshoot-bot-configuration.md) 및 해당 섹션의 다른 문제 해결 문서를 참조하세요.

## <a name="next-steps"></a>다음 단계

대화를 대본 파일에 저장하면 신속하게 초안을 만들고 디버깅을 위해 특정 반복 집합을 재생할 수 있습니다.

> [!div class="nextstepaction"]
> [대본 파일을 사용하여 봇 디버그](~/v4sdk/bot-builder-debug-transcript.md)

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
