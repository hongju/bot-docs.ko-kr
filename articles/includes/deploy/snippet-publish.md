---
ms.openlocfilehash: bb7520e8e99ad6326d7d00d8190dae306bf11afa
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252593"
---
Azure에 로컬 봇을 게시하세요. 이 단계는 시간이 약간 걸릴 수 있습니다.

```cmd
az bot publish --name <bot-resource-name> --proj-file-path "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| 옵션 | 설명 |
|:---|:---|
| --name | Azure에서 봇의 리소스 이름입니다. |
| --proj-file-path | C#인 경우 게시해야 하는 시작 프로젝트 파일 이름(.csproj 제외)입니다. 예: `EnterpriseBot` Node.js인 경우 봇의 주 진입점을 사용합니다. 예: `index.js` |
| --resource-group | 리소스 그룹의 이름입니다. |
| --code-dir | 업로드할 봇 코드가 있는 디렉터리입니다. |

"배포 성공!" 메시지와 함께 이 작업이 완료되면 봇이 Azure에 배포됩니다.
