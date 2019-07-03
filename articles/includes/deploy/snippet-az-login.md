---
ms.openlocfilehash: f8aad539a2d1e415833609f66cd5b398c88206f1
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252671"
---
명령 프롬프트를 열어 Azure Portal에 로그인합니다.

```cmd
az login
```

로그인할 수 있는 브라우저 창이 열립니다.

### <a name="set-the-subscription"></a>구독 설정

사용할 기본 구독을 설정합니다.

```cmd
az account set --subscription "<azure-subscription>"
```

봇을 배포하는 데 사용할 구독이 확실하지 않은 경우 `az account list` 명령을 사용하여 계정에 대한 `subscriptions`의 목록을 볼 수 있습니다.

봇 폴더로 이동합니다.

```cmd
cd <local-bot-folder>
```