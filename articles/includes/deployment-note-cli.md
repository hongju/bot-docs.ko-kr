---
ms.openlocfilehash: cc7e656d7c8a61e7bf784db579d0065c3bff331a
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035769"
---
LUIS와 같은 서비스를 사용하는 경우 `luisAuthoringKey`도 전달해야 합니다. Azure에서 기존 리소스 그룹을 사용하려면 위의 명령에 `groupName` 인수를 사용합니다.

`verbose` 옵션을 사용하여 봇을 배포하는 동안 발생할 수 있는 문제를 해결하는 것이 좋습니다. `msbot clone services` 명령에 사용되는 추가 옵션은 다음과 같습니다.

| 인수    | 설명 |
|--------------|-------------|
| `folder`     | `bot.recipe` 파일의 위치입니다. recipe 파일은 기본적으로 `DeploymentsScript/MSBotClone`에 만들어집니다. 이 파일은 수정하지 마세요.|
| `location`   | 봇 서비스 리소스를 만드는 데 사용되는 지리적 위치입니다. 예를 들어 eastus, westus, westus2 등과 같습니다.|
| `proj-file`  | C# 봇의 경우 .csproj 파일입니다. JS 봇의 경우 로컬 봇의 시작 프로젝트 파일 이름(예: index.js)입니다.|
| `name`       | Azure에서 봇을 배포하는 데 사용되는 고유 이름입니다. 로컬 봇과 동일한 이름일 수도 있습니다. 이름에 공백 또는 밑줄을 넣지 마세요.|
| `luisAuthoringKey` | LUIS 리소스의 적절한 LUIS 제작 지역에 대한 제작 키입니다. |

Azure 리소스를 만들기 전에 인증을 완료하라는 메시지가 표시됩니다. 화면에 표시되는 지침에 따라 이 단계를 완료합니다.

위의 단계를 완료하는 데 _몇 초에서 몇 분_이 걸리고 Azure에 만들어진 리소스의 이름이 심하게 손상되어 있습니다. 이와 같은 이름의 손상에 대한 자세한 내용은 GitHub 리포지토리에서 [문제 # 796](https://github.com/Microsoft/botbuilder-tools/issues/796)을 참조하세요.
