## <a name="application-settings-and-messaging-endpoint"></a>응용 프로그램 설정 및 메시징 엔드포인트

### <a name="verify-application-settings"></a>응용 프로그램 설정 확인

봇이 클라우드에서 제대로 작동하려면 응용 프로그램 설정이 정확한지 확인해야 합니다. **appID** 및 **암호**가 있으면 배포 프로세스의 일부로 응용 프로그램 구성 설정에서 `Microsoft AppId` 및 `Microsoft App Password` 값을 업데이트합니다. 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

> [!TIP]
> [!INCLUDE [Application configuration settings](~/includes/snippet-tip-bot-config-settings.md)]

봇을 Bot Framework에 등록하지 않은 경우(**appID** 및 **암호** 없음) 이 설정에 대한 임시 자리 표시자 값으로 봇을 배포할 수 있습니다.
그런 다음, [봇을 등록](~/bot-service-quickstart-registration.md)한 뒤 배포된 응용 프로그램 설정을 등록 중에 봇에 대해 생성된 **appID** 및 **암호** 값으로 업데이트합니다.

### <a id="messagingEndpoint"></a> 메시징 엔드포인트 확인

배포된 봇에는 Bot Framework Connector Service로부터 메시지를 수신할 수 있는 **메시징 엔드포인트**가 있어야 합니다.

> [!NOTE]
> Azure에 봇을 배포하면 응용 프로그램에 대한 SSL이 자동으로 구성되므로 Bot Framework에서 요구하는 **메시징 엔드포인트**를 사용할 수 있습니다.
> 다른 클라우드 서비스에 배포할 경우에는 봇에 **메시징 엔드포인트**가 포함되도록 응용 프로그램에 대한 SSL이 구성되어 있는지 확인해야 합니다.
