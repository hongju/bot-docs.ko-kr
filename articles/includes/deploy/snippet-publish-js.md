---
ms.openlocfilehash: 0891b9652154f8ed086cc45ce6018aa0be1a67b8
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230570"
---
로컬 JavaScript 봇을 Azure에 다시 게시하려면 먼저 봇을 로컬로 빌드하고 실행하는 데 사용되는 모든 파일이 포함된 단일 압축 파일을 수동으로 만들어야 합니다. 여기에는 `node_modules` 폴더에 다운로드된 모든 npm 라이브러리가 포함됩니다. 이 zip 파일을 만드는 경우 _사용하는 루트 디렉터리가 index.js 파일이 있는 디렉터리와 동일해야 합니다_.

봇의 모든 소스 코드가 포함된 zip 파일이 만들어지면 명령 프롬프트 창을 열고 다음 _Az cli_ 명령을 실행합니다. 

이 단계는 시간이 약간 걸릴 수 있습니다.

```cmd
az webapp deployment source config-zip --resource-group <resource-group-name> --name <bot-resource-name> --src <directory-path>
```

| 옵션 | 설명 |
|:---|:---|
| --resource-group | Azure의 리소스 그룹 이름입니다. |
| --name | Azure에서 봇의 리소스 이름입니다. |
| --src | 업로드할 압축된 봇 코드의 원본에 대한 전체 디렉터리 경로입니다. 예: `c:\my-local-repository\this-app-folder\my-zipped-code.zip` |

이 작업이 성공적으로 완료되면 봇이 Azure에 배포됩니다.
