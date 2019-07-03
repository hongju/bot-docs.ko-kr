---
title: 결제 요청 | Microsoft Docs
description: .NET용 Bot Framework SDK를 사용하여 결제 요청을 보내는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1b27cbc9a901318a4e1b050720fb9f8b230f9e75
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496654"
---
# <a name="request-payment"></a>결제 요청

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-request-payment.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-request-payment.md)

봇을 통해 사용자가 항목을 구매할 수 있는 경우 [서식 있는 카드](bot-builder-dotnet-add-rich-card-attachments.md) 내 특수한 단추 유형을 포함시키면 결제를 요청할 수 있습니다. 이 문서에서는 .NET용 Bot Framework SDK를 사용하여 결제 요청을 보내는 방법을 설명합니다.

## <a name="prerequisites"></a>필수 조건

.NET용 Bot Framework SDK를 사용하여 결제 요청을 보내기 전에 이러한 필수 구성 요소 작업을 완료해야 합니다.

### <a name="update-webconfig"></a>Web.config 업데이트

`MicrosoftAppId` 및 `MicrosoftAppPassword`에 대한 봇의 **Web.config** 파일을 [등록](~/bot-service-quickstart-registration.md) 프로세스 동안 봇에 대해 생성된 앱 ID 및 암호 값으로 업데이트합니다. 

> [!NOTE]
> 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

### <a name="create-and-configure-merchant-account"></a>가맹점 계정 만들기 및 구성

1. <a href="https://dashboard.stripe.com/register" target="_blank">Stripe 계정이 아직 없는 경우 하나 만들고 활성화합니다.</a>

2. <a href="https://seller.microsoft.com/dashboard/registration/seller/?accountprogram=botframework" target="_blank">Microsoft 계정을 사용하여 판매자 센터에 로그인합니다.</a>

3. 판매자 센터 내에서 Stripe를 사용하여 계정에 연결합니다.

4. 판매자 센터 내에서 대시보드로 이동하고 **MerchantID**의 값을 복사합니다.

5. 봇의 **Web.config** 파일을 업데이트하여 `MerchantId`를 판매자 센터 대시보드에서 복사한 값으로 설정합니다. 

[!INCLUDE [Payment process overview](../includes/snippet-payment-process-overview.md)]

## <a name="payment-bot-sample"></a>결제 봇 샘플

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">결제 봇</a> 샘플은 .NET을 사용하여 결제 요청을 전송하는 봇의 예제를 제공합니다. 작업에서 이 샘플 봇을 확인하려면 <a href="https://webchat.botframework.com/embed/paymentsample?s=d39Bk7JOMzQ.cwA.Rig.dumLki9bs3uqfWFMjXPn5PFnQVmT2VAVR1Zl1iPi07k" target="_blank">웹 채팅에서 시도</a>하거나, <a href="https://join.skype.com/bot/9fbc0f17-43eb-40fe-bf3b-af151e6ce45e" target="_blank">Skype 연락처로 추가</a>하거나, 결제 봇 샘플을 다운로드하고 Bot Framework Emulator를 사용하여 로컬로 실행합니다. 

> [!NOTE]
> 웹 채팅 또는 Skype에서 **결제 봇** 샘플을 사용하여 엔드투엔드 결제 프로세스를 완료하려면 Microsoft 계정 내에서 유효한 신용 카드 또는 직불 카드를 지정해야 합니다(예: 미국 카드 발급자로부터 유효한 카드). **결제 봇** 샘플은 테스트 모드에서 실행되므로(즉, `LiveMode`가 **Web.config**에서 `false`로 설정되어 있음) 카드에 요금이 청구되지 않으며, 카드의 CVV를 확인하지 않습니다.

이 문서의 다음 몇 개 섹션에서는 **결제 봇** 샘플과 관련하여 결제 프로세스의 세 가지 부분에 대해 설명합니다.

## <a id="request-payment"></a> 결제 요청

봇은 “결제”의 `CardAction.Type`을 지정하는 단추가 있는 [서식 있는 카드](bot-builder-dotnet-add-rich-card-attachments.md)를 포함하는 메시지를 보내 사용자의 결제를 요청합니다. **결제 봇** 샘플의 이 코드 조각은 사용자가 클릭(또는 탭)하여 결제 프로세스를 시작할 수 있는 **구입** 단추가 있는 Hero 카드를 포함하는 메시지를 만듭니다. 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#requestPayment)]

이 예제에서 단추의 형식은 `PaymentRequest.PaymentActionType`으로 지정됩니다. Bot Builder 라이브러리는 “지불”로 정의합니다. 단추의 값은 `BuildPaymentRequest` 메서드로 채워지며, 이는 지원되는 결제 방법, 세부 정보 및 옵션에 대한 정보를 포함하는 `PaymentRequest` 개체를 반환합니다. 구현 세부 정보에 대한 자세한 내용은 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">결제 봇</a> 샘플 내 **Dialogs/RootDialog.cs**를 참조하세요.

이 스크린샷은 위의 코드 조각에서 생성된 Hero 카드(**구입** 단추 포함)를 보여 줍니다. 
 
![지불 샘플 봇](../media/payments-bot-buy.png) 

> [!IMPORTANT]
> **구입** 단추에 대한 액세스 권한이 있는 사용자는 이를 통해 결제 프로세스를 시작할 수 있습니다. 그룹 대화의 컨텍스트 내에서 특정 사용자만 사용하도록 단추를 지정할 수는 없습니다. 

## <a id="user-experience"></a> 사용자 환경

사용자가 **구입** 단추를 클릭하면 Microsoft 계정을 통해 모든 필수 결제, 배송 및 연락처 정보를 제공하는 결제 웹 환경으로 이동합니다. 

![Microsoft 결제](../media/microsoft-payment.png)

### <a name="http-callbacks"></a>HTTP 콜백

HTTP 콜백은 봇이 특정 작업을 수행해야 함을 나타내기 위해 봇에 전송됩니다. 각 콜백은 이러한 속성 값을 포함하는 [작업](bot-builder-dotnet-activities.md)입니다. 

| 자산 | 값 |
|----|----|
| `Activity.Type` | 호출 | 
| `Activity.Name` | 봇이 수행해야 하는 작업의 유형(예: 배송지 주소 업데이트, 배송 옵션 업데이트, 결제 완료). | 
| `Activity.Value` | JSON 형식의 요청 페이로드입니다. | 
| `Activity.RelatesTo` |  결제 요청과 연관된 채널 및 사용자를 설명합니다. | 

> [!NOTE]
> `invoke`는 Microsoft Bot Framework에서 사용되도록 예약된 특수한 작업 형식입니다. `invoke` 작업의 보낸 사람은 봇이 HTTP 응답을 전송하여 콜백을 승인하기를 대기합니다.

## <a id="process-callbacks"></a> 콜백 처리

[!INCLUDE [Process callbacks overview](../includes/snippet-payment-process-callbacks-overview.md)]

### <a name="shipping-address-update-and-shipping-option-update-callbacks"></a>배송지 주소 업데이트 및 배송 옵션 업데이트 콜백

[!INCLUDE [Process shipping address and shipping option callbacks](../includes/snippet-payment-process-callbacks-1.md)]

### <a name="payment-complete-callbacks"></a>결제 완료 콜백

결제 완료 콜백을 수신하면 봇은 `Activity.Value` 속성에서 결제 응답뿐만 아니라 초기에 수정되지 않은 결제 요청의 복사본과 함께 제공됩니다. 결제 응답 개체에는 결제 토큰과 함께 고객이 결정한 마지막 선택 항목이 포함됩니다. 봇은 초기 결제 요청 및 고객의 최종 선택을 기반으로 최종 결제 요청을 다시 계산해야 합니다. 봇은 고객의 선택 항목을 유효한 것으로 가정한 상태로, 결제 토큰 헤더의 금액 및 통화가 최종 결제 요청과 일치하는지 확인해야 합니다.  봇이 고객에게 요금을 청구하기로 결정한 경우 결제 토큰 헤더에 있는 금액만 청구되어야 합니다. 이는 고객이 확인한 금액이기 때문입니다. 봇이 예상한 값과 결제 완료 콜백에서 수신한 값이 일치하지 않는 경우 결제 필드를 `failure`로 설정하고 HTTP 상태 코드 `200 OK`를 보내면 결제 요청이 실패할 수 있습니다.   

결제 세부 정보를 확인하는 것 외에도 봇은 결제 프로세스가 시작되기 전에 주문이 이행될 수 있는지를 확인해야 합니다. 예를 들어 구매되는 항목이 재고에 있는지 확인할 수 있습니다. 값이 올바르고 결제 프로세서가 결제 토큰을 성공적으로 청구한 경우 봇은 결제 완료를 표시하는 결제 웹 환경을 위해 결과 필드를 `success`로 설정함과 동시에 HTTP 상태 코드 `200 OK`로 응답해야 합니다. 봇이 수신하는 결제 토큰은 요청한 가맹점에 의해 한 번만 사용할 수 있으며, Stripe로 제출되어야 합니다(Bot Framework가 현재 지원하는 결제 프로세서만 해당). `400` 또는 `500` 범위의 HTTP 상태 코드를 보내면 고객에게 일반 오류가 표시됩니다.

**결제 봇** 샘플의 `OnInvoke` 메서드는 봇이 수신하는 콜백을 처리합니다. 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#processCallback)]

이 예제에서 봇은 들어오는 작업의 `Name` 속성을 검사하여 수행해야 하는 작업의 형식을 파악한 다음, 콜백을 처리하는 적절한 메서드를 호출합니다. 구현 세부 정보에 대한 자세한 내용은 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">결제 봇</a> 샘플 내 **Controllers/MessagesControllers.cs**를 참조하세요.

## <a name="testing-a-payment-bot"></a>결제 봇 테스트

[!INCLUDE [Test a payment bot](../includes/snippet-payment-test-bot.md)]

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">결제 봇</a> 샘플에서 **Web.config**의 `LiveMode` 구성 설정은 결제 완료 콜백에 에뮬레이트된 결제 토큰 또는 실제 결제 토큰이 포함되는지 여부를 확인합니다. `LiveMode`가 `false`로 설정된 경우 헤더가 봇의 아웃바운드 결제 요청에 추가되어 봇이 테스트 모드임을 표시하며, 결제 완료 콜백에는 요금을 청구할 수 없는 에뮬레이트된 결제 토큰이 포함됩니다. `LiveMode`가 `true`로 설정된 경우에는 봇이 테스트 모드임을 나타내는 헤더가 봇의 아웃바운드 결제 요청에서 생략되며 결제 완료 콜백에는 결제 처리를 위해 봇이 Stripe에 제출하는 실제 결제 토큰이 포함됩니다. 이는 지정된 결제 방법으로 요금이 청구되는 실제 트랜잭션입니다. 

## <a name="additional-resources"></a>추가 리소스

- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">결제 봇 샘플</a>
- [작업 개요](bot-builder-dotnet-activities.md)
- [메시지에 서식 있는 카드 추가](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="http://www.w3.org/Payments/" target="_blank">W3C에서 웹 결제</a> 
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Framework SDK 참조</a>
