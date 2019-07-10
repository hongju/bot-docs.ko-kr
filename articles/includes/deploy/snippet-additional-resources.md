---
ms.openlocfilehash: 2c06c67099f44fe1df2eb0099a514a697ef0d1c9
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405588"
---
봇이 배포되면 일반적으로 Azure Portal에 만들어지는 리소스는 다음과 같습니다.

| 리소스      | 설명 |
|----------------|-------------|
| Web App 봇 | Azure App Service에 배포되는 Azure Bot Service 봇입니다.|
| [App Service](https://docs.microsoft.com/azure/app-service/)| 웹 애플리케이션을 구축하고 호스팅할 수 있습니다.|
| [App Service 계획](https://docs.microsoft.com/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| 실행할 웹앱에 대한 컴퓨팅 리소스 세트를 정의합니다.|

Azure Portal을 통해 봇을 만드는 경우 [원격 분석용 Application Insights](~/v4sdk/bot-builder-telemetry.md)와 같은 추가 리소스를 프로비저닝할 수 있습니다.

`az bot` 명령에 대한 설명서를 보려면 [참조](https://docs.microsoft.com/cli/azure/bot?view=azure-cli-latest) 항목을 참조하세요.

Azure 리소스 그룹에 익숙하지 않은 경우 이 [용어](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#terminology) 항목을 참조하세요.