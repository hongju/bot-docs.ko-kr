---
title: Azure Table Storage를 사용하여 사용자 지정 상태 데이터 관리 | Microsoft Docs
description: Node.js용 Bot Framework SDK를 사용하여 Azure Table Storage에서 상태 데이터를 저장 및 검색하는 방법을 알아봅니다.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 34f2cb79d4dcef9ddb68c6de0333a94b4128b301
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404698"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-nodejs"></a>Node.js용 Azure Table Storage를 사용하여 사용자 지정 상태 데이터 관리

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

이 문서에서는 봇의 상태 데이터를 저장 및 관리하도록 Azure Table Storage를 구현합니다. 봇에서 사용되는 기본 커넥터 상태 서비스는 프로덕션 환경에서 사용할 수 없습니다. GitHub에서 제공되는 [Azure 확장](https://www.npmjs.com/package/botbuilder-azure)을 사용하거나 선택한 데이터 저장소 플랫폼을 사용하여 사용자 지정 상태 클라이언트를 구현해야 합니다. 사용자 지정 상태 저장소를 사용하는 몇 가지 이유는 다음과 같습니다.

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
- [Storage Explorer](http://storageexplorer.com/).

## <a name="create-azure-account"></a>Azure 계정 만들기
Azure 계정이 없으면 [여기](https://azure.microsoft.com/free/)를 클릭하여 체험 계정으로 등록하세요.

## <a name="set-up-the-azure-table-storage-service"></a>Azure Table Storage 서비스 설정
1. Azure Portal에 로그인한 후 **새로 만들기**를 클릭하여 새 Azure Table Storage 서비스를 만듭니다. 
2. Azure 테이블을 구현하는 **저장소 계정**을 검색합니다. **만들기**를 클릭하여 저장소 계정 만들기를 시작합니다. 
3. 모든 필드를 다 입력하면 화면 아래쪽에 있는 **만들기** 단추를 클릭하여 새 저장소 서비스를 배포합니다. 
4. 새 저장소 서비스가 배포된 후 방금 만든 저장소 계정으로 이동합니다. **저장소 계정** 블레이드에서 해당 서비스를 찾을 수 있습니다.
4. **액세스 키**를 선택하고 나중에 사용할 수 있게 키를 복사합니다. 봇은 **저장소 계정 이름** 및 **키**를 사용하여 상태 데이터를 저장하기 위한 저장소 서비스를 호출합니다.

## <a name="install-botbuilder-azure-module"></a>botbuilder-azure 모듈 설치

명령 프롬프트에서 `botbuilder-azure` 모듈을 설치하려면 봇의 디렉터리로 이동한 후 다음 npm 명령을 실행합니다.

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>봇 코드 수정

**Azure 테이블** 저장소를 사용하려면 봇의 다음 코드 줄을 **app.js** 파일에 추가합니다.

1. 새로 설치된 모듈이 필요합니다.

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. Azure에 연결하기 위해 연결 설정을 구성합니다.
   ```javascript
   // Table storage
   var tableName = "Table-Name"; // You define
   var storageName = "Table-Storage-Name"; // Obtain from Azure Portal
   var storageKey = "Azure-Table-Key"; // Obtain from Azure Portal
   ```
   `storageName` 및 `storageKay` 값은 Azure 테이블의 **액세스 키** 메뉴에서 찾을 수 있습니다. `tableName`이 Azure 테이블에 존재하지 않으면 만들어집니다.

3. `botbuilder-azure` 모듈을 사용하여 Azure 테이블에 연결할 두 개의 개체를 새로 만듭니다. 먼저 연결 구성 설정을 전달하는 `AzureTableClient`의 인스턴스를 만듭니다. 다음으로, `AzureTableClient` 개체를 전달하는 `AzureBotStorage`의 인스턴스를 만듭니다. 예:

   ```javascript
   var azureTableClient = new azure.AzureTableClient(tableName, storageName, storageKey);

   var tableStorage = new azure.AzureBotStorage({gzipData: false}, azureTableClient);
   ```

4. 메모리 내 저장소 대신 사용자 지정 데이터베이스를 사용하고 데이터베이스에 세션 정보를 추가하도록 지정합니다. 예:

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...

        // capture session user information
        session.userData = {"userId": session.message.user.id, "jobTitle": "Senior Developer"};

        // capture conversation information  
        session.conversationData[timestamp.toISOString().replace(/:/g,"-")] = session.message.text;

        // save data
        session.save();
   })
   .set('storage', tableStorage);
   ```
이제 에뮬레이터를 사용하여 봇을 테스트할 준비가 완료되었습니다.

## <a name="run-your-bot-app"></a>봇 앱 실행

명령 프롬프트에서 봇의 디렉터리로 이동한 후 다음 명령을 사용하여 봇을 실행합니다.

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>에뮬레이터에 봇 연결

이 시점에서 봇이 로컬로 실행됩니다. 에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터의 주소 표시줄에 <strong>http://localhost:port-number/api/messages</strong>를 입력합니다. 여기서 port-number는 애플리케이션이 실행 중인 브라우저에 표시되는 포트 번호와 일치합니다. 지금은 <strong>Microsoft 앱 ID</strong> 및 <strong>Microsoft 앱 암호</strong> 필드를 비워 둘 수 있습니다. 나중에 [봇을 등록](~/bot-service-quickstart-registration.md)할 때 이 정보를 가져올 수 있습니다.
2. **연결**을 클릭합니다.
3. 봇에 메시지를 전송하여 봇을 테스트합니다. 평소와 같이 봇과 상호 작용합니다. 완료되면 **Storage Explorer**로 이동한 후 저장된 상태 데이터를 확인합니다.

## <a name="view-data-in-storage-explorer"></a>Storage Explorer에서 데이터 보기

상태 데이터를 보려면 **Storage Explorer**를 열고 Azure Portal 자격 증명을 사용하여 Azure에 연결하거나 `storageName` 및 `storageKey`를 사용하여 테이블에 직접 연결한 후 `tableName`으로 이동합니다. 

![botdata 테이블 행이 표시된 Storage Explorer 스크린샷](~/media/bot-builder-nodejs-state-azure-table-storage/bot-builder-nodejs-state-azure-table-storage-query.png)

**data** 열에 나오는 대화의 레코드는 다음과 같습니다.

```JSON
{
    "2018-05-15T18-23-48.780Z": "I'm the second user",
    "2018-05-15T18-23-55.120Z": "Do you know what time it is?",
    "2018-05-15T18-24-12.214Z": "I'm looking for information about the new process."
}
```

## <a name="next-step"></a>다음 단계

봇의 상태 데이터를 완전히 제어할 수 있으므로 이 상태 데이터를 사용하여 대화 흐름을 보다 잘 관리하는 방법을 알아보겠습니다.

> [!div class="nextstepaction"]
> [대화 흐름 관리](bot-builder-nodejs-dialog-manage-conversation-flow.md)
