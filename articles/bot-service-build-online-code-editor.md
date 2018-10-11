---
title: Azure 온라인 코드 편집기를 사용하여 봇 빌드 | Microsoft Docs
description: Bot Service의 온라인 코드 편집기를 사용하여 봇을 빌드하는 방법에 대해 알아봅니다.
keywords: 온라인 코드 편집기, Azure Portal, 함수 봇
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/21/2018
ms.openlocfilehash: 00342b624c43b8eea5bbc2bdd828703eafa5ef63
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707859"
---
# <a name="edit-a-bot-with-online-code-editor"></a>온라인 코드 편집기를 사용하여 봇 편집

온라인 코드 편집기를 사용하여 IDE 없이 봇을 빌드할 수 있습니다. 이 항목에서는 온라인 코드 편집기에서 봇 코드를 여는 방법을 보여 줍니다. 

## <a name="edit-bot-source-code-in-online-code-editor"></a>온라인 코드 편집기에서 봇 소스 코드 편집

온라인 코드 편집기에서 봇의 소스 코드를 편집하려면 사용 중인 봇의 유형에 맞게 다음을 수행합니다.

### <a name="web-app-bot"></a>Web App 봇
1. [Azure Portal](http://portal.azure.com)에 로그인하고 봇용 블레이드를 엽니다.
2. **봇 관리** 섹션에서 **빌드**를 클릭합니다.
3. **온라인 코드 편집기 열기**를 클릭합니다. 이렇게 하면 봇의 코드가 새 브라우저 창에서 열립니다. 

   ![온라인 코드 편집기 열기](~/media/azure-bot-build/open-online-code-editor.png)

   봇의 언어에 따라 **WWWRoot** 디렉터리 아래의 파일 구조가 달라집니다. 예를 들어, C# 봇이 있는 경우 **WWWRoot**는 다음과 같을 수 있습니다.

   ![C# 파일 구조](~/media/azure-bot-build/cs-wwwroot-structure.png)

   Node.js C# 봇이 있는 경우 **WWWRoot**는 다음과 같을 수 있습니다.

   ![Node.js 파일 구조](~/media/azure-bot-build/node-wwwroot-structure.png)

4. 코드를 변경합니다. 예를 들어 C# 봇의 경우 .cs 파일로 시작할 수 있습니다. Node.js 봇의 경우 App.js 파일로 시작할 수 있습니다.

   > [!NOTE]
   > 프로젝트의 현재 원본 파일에 대한 코드를 변경할 수 있지만 온라인 코드 편집기를 사용하여 새 원본 파일을 만들 수는 없습니다. 봇에 새 원본 파일을 추가하려면 [소스 프로젝트를 다운로드](bot-service-build-download-source-code.md)하고 파일을 추가한 후 변경 내용을 Azure에 다시 게시해야 합니다.

5. **소비 계획**에 있는 C# 봇 및 모든 Node.js 봇의 경우 봇은 자동으로 저장됩니다. 

6. **App Service** 계획에 있는 C# 봇의 경우 **콘솔** 블레이드를 열고 **build.cmd** 명령을 보냅니다. 

   ![콘솔 블레이드에서 프로젝트 빌드](~/media/azure-bot-build/cs-console-build-cmd.png)
 
   > [!NOTE]
   > 이 명령이 빌드에 실패하면 봇의 App Service를 다시 시작하고 빌드를 다시 시도합니다. App Service를 다시 시작하려면 봇의 블레이드에서 **모든 App Service 설정**을 클릭한 다음, **다시 시작** 단추를 클릭합니다.
   > ![웹앱 다시 시작](~/media/azure-bot-build/open-online-code-editor-restart-appservice.png)

7. Azure Portal로 다시 전환하고 **웹 채팅 테스트**를 클릭하여 변경 내용을 테스트합니다. 이미 이 봇에 대한 웹 채팅이 열려 있는 경우 **다시 시작**을 클릭하여 새 변경 내용을 확인합니다.

### <a name="functions-bot"></a>함수 봇

1. [Azure Portal](http://portal.azure.com)에 로그인하고 봇용 블레이드를 엽니다.
2. **봇 관리** 섹션에서 **빌드**를 클릭합니다.
3. **Azure Functions에서 이 봇 열기**를 클릭합니다. 그러면 <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> UI가 포함된 봇이 열립니다. 
4. 코드를 변경합니다. 예를 들어 함수의 메시지 코드를 업데이트합니다. 아래의 스크린샷은 Node.js 함수 봇의 메시지 코드를 보여 줍니다.

   ![함수 봇 메시지 코드 편집기](~/media/azure-bot-build/functions-messages-code.png)

5. 코드 변경 내용을 저장합니다.
6. Azure Portal로 다시 전환하고 **웹 채팅 테스트**를 클릭하여 변경 내용을 테스트합니다. 이미 이 봇에 대한 웹 채팅이 열려 있는 경우 **다시 시작**을 클릭하여 새 변경 내용을 확인합니다.

## <a name="next-steps"></a>다음 단계
온라인 코드 편집기를 사용하여 봇 코드를 편집하는 방법을 알게 되었으므로, 이제 즐겨 찾는 IDE를 사용하여 로컬로 봇을 빌드할 수도 있습니다.

> [!div class="nextstepaction"]
> [봇 소스 코드 다운로드](bot-service-build-download-source-code.md)
