---
title: Azure에 봇 마이그레이션 | Microsoft Docs
description: 레거시 Bot Framework Portal의 봇을 Azure Portal의 봇 서비스에 마이그레이션하는 방법을 살펴봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 3/22/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c6e91ebfe54857f11772bd4a926e3b5b2776c8cc
ms.sourcegitcommit: 54a4382add4756346098b286695a9b4791db7139
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/29/2019
ms.locfileid: "58616939"
---
# <a name="migrate-your-bot-to-azure"></a>Azure에 봇 마이그레이션

[Bot Framework Portal](http://dev.botframework.com)의 모든 **Azure Bot Service(미리 보기)** 는 Azure의 새 Bot Service로 마이그레이션해야 합니다. 이 서비스는 2017년 12월에 일반 제공(GA)되었습니다. 

**Teams**, **Skype** 또는 **Cortana** 채널에만 연결된 등록 봇은 *마이그레이션할 필요가 없습니다*. 예를 들어, **Facebook** 및 **Skype**에 연결된 등록 봇은 마이그레이션이 *필요하지만* **Skype** 및 **Cortana**에 연결된 등록 봇은 마이그레이션이 *필요 하지 않습니다*.

> [!IMPORTANT]
> Node.js로 만든 Functions 봇을 마이그레이션하기 전에 **Azure Functions Pack**을 사용하여 **node_modules** 모듈을 함께 패키징해야 합니다. 이렇게 하면 마이그레이션 중의 성능과, 마이그레이션 후의 Functions 봇 실행이 향상됩니다. 모듈 패키징은 [Funcpack을 사용하여 Functions 봇 패키징](#package-a-functions-bot-with-funcpack)을 참조하세요.

봇을 마이그레이션하려면 다음을 수행합니다.

1. [Bot Framework Portal](http://dev.botframework.com)에 로그인하고 **내 봇**을 클릭합니다.
2. 마이그레이션할 봇에 대한 **마이그레이션** 단추를 클릭합니다.
3. **조건**에 동의하고 **마이그레이션**을 클릭하여 마이그레이션 프로세스를 시작하거나 **취소**를 클릭하여 이 작업을 취소합니다.

> [!IMPORTANT]
> 마이그레이션 작업을 진행하는 동안 페이지를 나가거나 새로 고치면 안 됩니다. 그러면 마이그레이션 작업이 미리 중단되고 다시 작업을 수행해야 합니다. 마이그레이션이 완료되도록 확인 메시지가 나타날 때까지 기다립니다.

마이그레이션 프로세스가 성공적으로 완료된 후에는 **마이그레이션 상태**에 해당 봇이 마이그레이션되었다고 표시되며 문제가 있는 경우 **마이그레이션을 롤백** 단추를 마이그레이션 날짜로부터 1주일 동안 사용할 수 있습니다.

마이그레이션된 봇의 이름을 클릭하면 [Azure Portal](http://portal.azure.com)에서 봇이 열립니다.

## <a name="package-a-functions-bot-with-funcpack"></a>Funcpack을 사용하여 Functions 봇 패키징

Node.js로 만든 Functions 봇은 마이그레이션하기 전에 [Funcpack](https://github.com/Azure/azure-functions-pack)을 사용하여 압축해야 합니다. 프로젝트를 Funcpack하려면 다음 단계를 따릅니다.

1.  아직 없으면 코드를 로컬로 [다운로드](bot-service-build-download-source-code.md)합니다.
2.  **packages.json**의 모든 npm 패키지를 최신 버전으로 업데이트한 다음, `npm install`을 실행합니다.
3.  **messages/index.js**를 열고 `module.exports = { default: connector.listen() }`을 `module.exports = connector.listen();`으로 변경합니다.
4.  npm: `npm install -g azure-functions-pack`을 통해 Funcpack을 설치합니다.
5.  **node_modules** 디렉터리를 패키징하려면 `funcpack pack ./` 명령을 실행합니다.
6.  Bot Framework Emulator를 통해 Functions 봇을 실행하여 로컬에서 테스트할 수 있습니다. *funcpack* 봇의 자세한 실행 방법은 [여기](https://github.com/Azure/azure-functions-pack#how-to-run)에 있습니다. 
7.  코드를 다시 Azure에 업로드합니다. `.funcpack` 디렉터리가 업로드되었는지 확인합니다. **node_modules** 디렉터리는 업로드하지 않아도 됩니다.
8. 원격 봇이 예상대로 응답하는지 테스트합니다.
9. 위의 단계를 사용하여 봇을 마이그레이션합니다.

## <a name="migration-under-the-hood"></a>내부에서 마이그레이션

마이그레이션하는 봇의 유형에 따라 아래 목록을 통해 내부에서 일어나는 일에 대한 이해를 높일 수 있습니다.

* **Web App 봇** 또는 **Functions 봇**: 이러한 유형의 봇에서는 소스 코드 프로젝트가 이전 봇에서 새 봇으로 복사됩니다. 봇의 저장소, Application Insights, LUIS 등과 같은 다른 리소스는 그대로 유지됩니다. 이러한 경우 새 봇은 기존 리소스에 대해 ID/키/암호 사본을 포함합니다. 
* **봇 채널 등록**: 이 유형의 봇에서는 마이그레이션 프로세스에서 단순히 새 **봇 채널 등록**을 만들고 이전 봇의 엔드포인트를 복사합니다. 
* 마이그레이션하려는 봇 유형에 관계없이 마이그레이션 프로세스에서 기존 봇의 상태가 어떤 식으로든 수정되지 않습니다. 이를 통해 롤백이 필요한 경우 안전하게 롤백할 수 있습니다.
* [지속적인 배포](bot-service-build-continuous-deployment.md)를 설정한 경우 원본 제어가 대신 새 봇에 연결할 수 있게 다시 설정해야 합니다.

## <a name="understanding-azure-resources-after-migration"></a>마이그레이션 후 Azure 리소스 이해
마이그레이션 완료되면 **리소스 그룹**에 봇 작동에 필요한 많은 Azure 리소스가 포함됩니다. 리소스 유형 및 수는 마이그레이션한 봇 유형에 따라 다릅니다. 자세한 정보는 다음 섹션을 참조하세요.

### <a name="registration-bot"></a>등록 봇

가장 간단한 유형입니다. Azure에서 리소스 그룹은 "봇 채널 등록"이라는 하나의 리소스 유형을 포함합니다. Bot Framework Developer Portal의 이전 봇 레코드에 해당하는 것입니다.

![Azure의 봇 채널 등록 봇 목록](~/media/bot-service-migrate-bot/channel-registration-bot.png)

### <a name="web-app-bot"></a>Web App 봇
Web App 봇 마이그레이션에서는 유형이 “Web App Bot”인 봇 서비스 리소스와 새 App Service Web App을 프로비전합니다(아래 스크린샷의 녹색). 이전 Azure Bot Service(미리 보기) 봇도 여전히 여기에 있고 삭제 가능합니다(아래 스크린샷의 빨강).

![Azure의 Web App 봇 목록](~/media/bot-service-migrate-bot/web-app-bot.png)

### <a name="functions-bot"></a>Functions 봇
Functions 봇 마이그레이션에서는 유형이 “Functions Bot”인 봇 서비스 리소스와 새 앱 서비스 Functions 앱을 프로비전합니다(아래 스크린샷의 녹색). 이전 Azure Bot Service(미리 보기) 봇도 여전히 여기에 있고 삭제 가능합니다(아래 스크린샷의 빨강).

![Azure의 Functions 봇 목록](~/media/bot-service-migrate-bot/functions-bot.png)


## <a name="roll-back-migration"></a>마이그레이션 롤백

마이그레이션 중에 또는 후에 봇이 무언가 잘못된 경우 **마이그레이션을 롤백**할 수 있습니다. 마이그레이션을 롤백하려면 다음을 수행합니다.

1. [Bot Framework Portal](http://dev.botframework.com)에 로그인하고 **내 봇**을 클릭합니다.
2. 롤백할 봇에 대해 **마이그레이션 롤백** 단추를 클릭합니다. 프롬프트가 표시됩니다.
3. 계속하려면 **예, 롤백**을, 롤백 작업을 취소하려면 **취소**를 클릭합니다.

> [!NOTE]
> 롤백 기능은 마이그레이션 후 1주일 동안 제공되며 마이그레이션 봇에 문제가 발생한 경우에만 사용해야 합니다.

## <a name="migration-troubleshootingknown-issues"></a>마이그레이션 문제 해결/알려진 문제
내 node.js/functions 봇이 성공적으로 마이그레이션되었으나 응답하지 않습니다.

* **엔드포인트 확인**
  * 봇 리소스에서 **설정** 블레이드로 이동하고 봇 엔드포인트에 값이 포함된 “code” 쿼리 문자열 매개 변수가 있는지 확인합니다. 없으면 추가해야 합니다.
* **백업을 위해 kudu에서 secrets 폴더 확인**
  * 일부 드문 경우 충돌을 야기하는 백업 비밀 파일이 있을 수 있습니다. Kudu에서 **home\data\Functions\secrets** 폴더로 이동하고 발견한 모든 **host.snapshot**(또는 **host.backup**) 파일을 삭제합니다. 한 **host.json**과 한 **messages.json**이 있어야 합니다. 마지막으로 App Service를 다시 시작하고 봇과 채팅해 봅니다.

그 밖의 문제는 Azure 고객 지원팀에 CRI를 제출하거나 [GitHub](https://github.com/MicrosoftDocs/bot-framework-docs/issues)에서 문제를 제기하세요.


## <a name="next-steps"></a>다음 단계

이제 봇이 마이그레이션되었으므로 Azure Portal에서 봇을 관리하는 방법을 살펴봅니다.

> [!div class="nextstepaction"]
> [봇 관리](bot-service-manage-overview.md)
