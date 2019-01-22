---
ms.openlocfilehash: 867e65b25878f810e3247eb3cace95f4d31e11db
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360908"
---
현재 프로젝트 디렉터리에 포함되지 않는 임시 디렉터리를 사용합니다. 

이 명령은 save-path 아래에 하위 디렉터리를 만드는데 지정된 경로가 이미 있어야 합니다.

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| 옵션 | 설명 |
|:---|:---|
| --name | Azure에서 봇의 이름입니다. |
| --resource-group | 봇이 속하는 리소스 그룹의 이름입니다. |
| --save-path | 봇 코드를 다운로드할 기존 디렉터리입니다. |