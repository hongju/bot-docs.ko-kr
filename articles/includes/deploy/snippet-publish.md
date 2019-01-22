---
ms.openlocfilehash: 88732d2d5490962d7a899d936767e7dd148e94c5
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360897"
---
Azure에 로컬 봇을 게시하세요. 이 단계는 시간이 약간 걸릴 수 있습니다.

```cmd
az bot publish --name <bot-resource-name> --proj-name "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| 옵션 | 설명 |
|:---|:---|
| --name | Azure에서 봇의 리소스 이름입니다. |
| --proj-name | C#인 경우 게시해야 하는 시작 프로젝트 파일 이름(.csproj 제외)입니다. 예: `EnterpriseBot` Node.js인 경우 봇의 주 진입점을 사용합니다. 예: `index.js` |
| --resource-group | 리소스 그룹의 이름입니다. |
| --code-dir | 업로드할 봇 코드가 있는 디렉터리입니다. |

"배포 성공!" 메시지와 함께 이 작업이 완료되면 봇이 Azure에 배포됩니다.