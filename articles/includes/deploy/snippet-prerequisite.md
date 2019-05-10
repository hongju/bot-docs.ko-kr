---
ms.openlocfilehash: 7b065c8adf97b83454ee3dedf25a4bc3623c1db4
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035682"
---
- Azure 구독이 아직 없는 경우 시작하기 전에 [체험 계정](https://azure.microsoft.com/free/)을 만듭니다.
- 최신 버전의 [Azure CLI 도구](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)를 설치합니다.
- 최신 버전의 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 도구를 설치합니다.
- 최근에 출시된 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) 버전을 설치합니다.
- [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)를 설치 및 구성합니다. 
- [bot 리소스 관리](~/v4sdk/bot-file-basics.md)에 대한 지식.

msbot 4.3.2 이상에서는 Azure CLI 버전 2.0.54 이상이 필요합니다. botservice 확장을 설치한 경우 이 명령을 사용하여 제거하세요.

```cmd
az extension remove --name botservice
```
