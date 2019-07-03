---
ms.openlocfilehash: eac6abae509d92ea4714bc01221f180ea575950f
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252628"
---
암호화 키를 가져옵니다.

1. [Azure Portal](http://portal.azure.com/)에 로그인합니다.
1. 봇의 Web App 봇 리소스를 엽니다.
1. 봇의 **애플리케이션 설정**을 엽니다.
1. **애플리케이션 설정** 창에서 **애플리케이션 설정**이 나올 때까지 아래로 스크롤합니다.
1. **botFileSecret**을 찾고 해당 값을 복사합니다.

.bot 파일의 암호를 해독합니다.

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| 옵션 | 설명 |
|:---|:---|
| --bot | 다운로드된 .bot 파일의 상대 경로입니다. |
| --secret | 암호화 키입니다. |

암호 해독된 `.bot` 파일을 로컬 봇 프로젝트가 포함된 디렉터리로 복사하고, 새 `.bot` 파일을 사용하도록 봇을 업데이트하고, 이전 `.bot` 파일을 제거합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**appsettings.json**에서, 로컬 디렉터리에 추가한 새 `.bot` 파일을 가리키도록 **botFilePath** 속성을 업데이트합니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**.env**에서, 로컬 디렉터리에 추가한 새 `.bot` 파일을 가리키도록 **botFilePath** 속성을 업데이트합니다.

---

봇이 업데이트되면 다운로드한 봇의 임시 디렉터리를 삭제합니다.