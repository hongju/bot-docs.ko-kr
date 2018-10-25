---
title: Azure Cosmos DB를 사용하여 사용자 지정 상태 데이터 관리 | Microsoft Docs
description: Node.js용 Bot Builder SDK와 함께 Azure Cosmos DB를 사용하여 상태 데이터를 저장 및 검색하는 방법을 알아봅니다.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0c0d91a7ec9fd1d72c7c51c042b0f52e28798778
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998120"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-nodejs"></a>Node.js용 Azure Cosmos DB를 사용하여 사용자 지정 상태 데이터 관리

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

이 문서에서는 Cosmos DB 저장소를 구현하여 봇의 상태 데이터를 저장 및 관리합니다. 봇에서 사용되는 기본 커넥터 상태 서비스는 프로덕션 환경에서 사용할 수 없습니다. GitHub에서 제공되는 [Azure 확장](https://www.npmjs.com/package/botbuilder-azure)을 사용하거나 선택한 데이터 저장소 플랫폼을 사용하여 사용자 지정 상태 클라이언트를 구현해야 합니다. 사용자 지정 상태 저장소를 사용하는 몇 가지 이유는 다음과 같습니다.

- 상태 API 처리량 증가(성능에 대한 제어 강화)
- 지역 복제에 대한 대기 시간 감소
- 데이터가 저장되는 위치 제어(예: 미국 서부 및 미국 동부)
- 실제 상태 데이터에 대한 액세스
- 상태 데이터 DB가 다른 봇과 공유되지 않음
- 32kb 초과 저장소

## <a name="prerequisites"></a>필수 조건

- [Node.js](https://nodejs.org/en/).
- [Bot Framework Emulator](~/bot-service-debug-emulator.md)
- Node.js 봇이 있어야 합니다. 아직 없는 경우 [봇을 하나 만듭니다](bot-builder-nodejs-quickstart.md). 

## <a name="create-azure-account"></a>Azure 계정 만들기
Azure 계정이 없으면 [여기](https://azure.microsoft.com/en-us/free/)를 클릭하여 체험 계정으로 등록하세요.

## <a name="set-up-the-azure-cosmos-db-database"></a>Azure Cosmos DB 데이터베이스 설정
1. Azure Portal에 로그인한 후 **새로 만들기**를 클릭하여 새 *Azure Cosmos DB* 데이터베이스를 만듭니다. 
2. **데이터베이스**를 클릭합니다. 
3. **Azure Cosmos DB**를 찾고 **만들기**를 클릭합니다.
4. 필드를 입력합니다. **API** 필드에 **SQL(DocumentDB)** 을 선택합니다. 모든 필드를 다 입력했으면 화면 아래쪽에 있는 **만들기** 단추를 클릭하여 새 데이터베이스를 배포합니다. 
5. 새 데이터베이스가 배포된 후 새 데이터베이스로 이동합니다. **액세스 키**를 클릭하여 키 및 연결 문자열을 찾습니다. 봇은 이 정보를 사용하여 상태 데이터를 저장하기 위한 저장소 서비스를 호출합니다.

## <a name="install-botbuilder-azure-module"></a>botbuilder-azure 모듈 설치

명령 프롬프트에서 `botbuilder-azure` 모듈을 설치하려면 봇의 디렉터리로 이동한 후 다음 npm 명령을 실행합니다.

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>봇 코드 수정

**Azure Cosmos DB** 데이터베이스를 사용하려면 다음 코드 줄을 봇의 **app.js** 파일에 추가합니다.

1. 새로 설치된 모듈이 필요합니다.

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. Azure에 연결하기 위해 연결 설정을 구성합니다.
   ```javascript
   var documentDbOptions = {
       host: 'Your-Azure-DocumentDB-URI', 
       masterKey: 'Your-Azure-DocumentDB-Key', 
       database: 'botdocs',   
       collection: 'botdata'
   };
   ```
   `host` 및 `masterKey` 값은 테이블의 **키** 메뉴에서 찾을 수 있습니다. `database` 및 `collection` 항목이 Azure 데이터베이스에 존재하지 않으면 새로 만듭니다.

3. `botbuilder-azure` 모듈을 사용하여 Azure 데이터베이스에 연결할 두 개의 개체를 새로 만듭니다. 먼저 연결 구성 설정을 전달하는 `DocumentDBClient`의 인스턴스를 만듭니다(위에서 `documentDbOptions`로 정의됨). 다음으로, `DocumentDBClient` 개체를 전달하는 `AzureBotStorage`의 인스턴스를 만듭니다. 예: 
   ```javascript
   var docDbClient = new azure.DocumentDbClient(documentDbOptions);

   var cosmosStorage = new azure.AzureBotStorage({ gzipData: false }, docDbClient);
   ```

4. 메모리 내 저장소 대신 사용자 지정 데이터베이스를 사용하도록 지정합니다. 예: 

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...
   })
   .set('storage', cosmosStorage);
   ```

이제 에뮬레이터를 사용하여 봇을 테스트할 준비가 완료되었습니다.

## <a name="run-your-bot-app"></a>봇 앱 실행

명령 프롬프트에서 봇의 디렉터리로 이동한 후 다음 명령을 사용하여 봇을 실행합니다.

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>에뮬레이터에 봇 연결

이 시점에서 봇이 로컬로 실행됩니다. 에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터의 주소 표시줄에 <strong>http://localhost:port-number/api/messages</strong>를 입력합니다. 여기서 port-number는 응용 프로그램이 실행 중인 브라우저에 표시되는 포트 번호와 일치합니다. 지금은 <strong>Microsoft 앱 ID</strong> 및 <strong>Microsoft 앱 암호</strong> 필드를 비워 둘 수 있습니다. 나중에 [봇을 등록](~/bot-service-quickstart-registration.md)할 때 이 정보를 가져올 수 있습니다.
2. **Connect**를 클릭합니다.
3. 봇에 메시지를 전송하여 봇을 테스트합니다. 평소와 같이 봇과 상호 작용합니다. 완료되면 **Storage 탐색기**로 이동한 후 저장된 상태 데이터를 확인합니다.

## <a name="view-state-data-on-azure-portal"></a>Azure Portal에서 상태 데이터 보기

상태 데이터를 보려면 Azure Portal에 로그인하고 데이터베이스로 이동합니다. **데이터 탐색기(미리 보기)** 를 클릭하여 봇의 상태 정보가 저장되고 있는지 확인합니다.

## <a name="next-step"></a>다음 단계

봇의 상태 데이터를 완전히 제어할 수 있으므로 이 상태 데이터를 사용하여 대화 흐름을 보다 잘 관리하는 방법을 알아보겠습니다.

> [!div class="nextstepaction"]
> [대화 흐름 관리](bot-builder-nodejs-dialog-manage-conversation-flow.md)
