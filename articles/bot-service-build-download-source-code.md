---
title: 봇 소스 코드 다운로드 및 재배포 | Microsoft Docs
description: Bot Service를 다운로드하고 게시하는 방법을 알아봅니다.
keywords: 원본 코드 다운로드, 재배포, 배포, zip 파일, 게시
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: b77e096d28f51f605db9c49d36e796553f9293ef
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756713"
---
# <a name="download-and-redeploy-bot-source-code"></a>봇 소스 코드 다운로드 및 재배포

Bot Service를 사용하면 봇에 대한 전체 원본 프로젝트를 다운로드할 수 있습니다. 이를 통해 선택한 IDE를 사용하여 로컬에서 봇 관련 작업을 할 수 있습니다. 변경을 완료하면 변경 내용을 다시 Azure에 게시할 수 있습니다. 

이 항목에서는 봇의 원본 코드를 다운로드하고 변경 내용을 다시 Azure에 게시하는 방법을 보여 줍니다. 

## <a name="download-bot-source-code"></a>봇 원본 코드 다운로드

봇을 로컬에서 개발하려면 다음을 수행합니다.

1. Azure Portal에서 봇에 대한 블레이드를 엽니다.
2. **봇 관리** 섹션에서 **빌드**를 클릭합니다.
3. **Zip 파일 다운로드**를 클릭합니다. 

   ![원본 코드 다운로드](~/media/azure-bot-build/download-zip-file.png)

4. .zip 파일을 로컬 디렉터리에 추출합니다.
5. 추출된 폴더로 이동하고 선호하는 IDE에서 원본 파일을 엽니다.
6. 원본 내용을 변경합니다. 기본 원본 파일을 편집하거나 새 원본 파일을 프로젝트에 추가합니다.

준비가 완료되면 원본을 다시 Azure에 게시할 수 있습니다.

## <a name="publish-node-bot-source-code-to-azure"></a>Azure에 Node 봇 원본 코드 게시

이러한 패키지를 설치하려면 명령 프롬프트에서 프로젝트 디렉터리로 이동하고 다음 NPM 명령을 실행합니다.

**참고:** 이러한 패키지는 한 번만 추가해야 합니다.

```console
npm install --save fs
npm install --save path
npm install --save request
npm install --save zip-folder
```

이제 프로젝트를 Microsoft Azure에 게시할 준비가 완료되었습니다. 프로젝트를 Microsoft Azure에 게시하려면 cmd 프롬프트에서 다음 NPM 명령을 실행합니다.

```console
npm run azure-publish
```

> [!NOTE]
> 이 NPM 명령 후에 오류가 발생하면 `"scripts": {"azure-publish": "node publish.js"}`를 `package.json` 파일에 추가하고 다시 실행해야 할 수 있습니다.

## <a name="publish-c-bot-source-code-to-azure"></a>Azure에 C# 봇 원본 코드 게시

Visual Studio를 사용하여 C# 코드를 Azure에 게시하는 작업은 2단계 프로세스입니다. 먼저 게시 설정을 구성해야 합니다. 그런 다음, 변경 내용을 게시할 수 있습니다.

Visual Studio에서 게시를 구성하려면 다음을 수행합니다.

1. Visual Studio에서 **솔루션 탐색기**를 클릭합니다.
2. 프로젝트를 마우스 오른쪽 단추로 클릭하고 **게시...** 를 클릭합니다. **게시** 창이 열립니다.
3. **새 프로필 만들기**를 클릭하고, **프로필 가져오기**를 클릭하고, **확인**을 클릭합니다.
4. 프로젝트 폴더로 이동한 다음, **PostDeployScripts** 폴더로 이동하고, **.PublishSettings**로 끝나는 파일을 선택합니다. **열기**를 클릭합니다.

이제 프로젝트는 변경 내용을 Azure에 게시하도록 구성되어 있습니다.

프로젝트가 구성된 후 다음을 수행하여 봇 원본 코드를 다시 Azure에 게시할 수 있습니다.

1. Visual Studio에서 **솔루션 탐색기**를 클릭합니다.
2. 프로젝트를 마우스 오른쪽 단추로 클릭하고 **게시...** 를 클릭합니다.
3. **게시** 단추를 클릭하여 변경 내용은 Azure에 게시합니다.

## <a name="next-steps"></a>다음 단계
이제 로컬에서 봇을 빌드하는 방법을 알았으므로 봇에 대한 지속적인 배포를 설정할 수 있습니다.

> [!div class="nextstepaction"]
> [지속적인 배포 설정](bot-service-build-continuous-deployment.md)
