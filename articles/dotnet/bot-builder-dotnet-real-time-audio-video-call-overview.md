---
redirect_url: https://aka.ms/realTimeMediaCalling-repo
ms.openlocfilehash: 9251c3a8ea75376b0891fc173975470a66ccaec8
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032958"
---
<a name="--"></a><!--
---
제목: Skype에 대한 실시간 미디어 봇 빌드 | Microsoft Docs 설명: .NET용 Bot Framework SDK 및 .NET용 Bot Builder-RealTimeMediaCalling SDK를 사용하여 Skype와의 실시간 오디오/비디오 통화를 수행하는 봇을 빌드하는 방법을 살펴봅니다.
작성자: MalarGit ms.author: malarch manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 12/13/17 monikerRange: 'azure-bot-service-3.0'
---

# <a name="build-a-real-time-media-bot-for-skype"></a>Skype용 실시간 미디어 봇 빌드

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

봇용 실시간 미디어 플랫폼은 봇이 프레임 단위로 음성 및 비디오를 주고받을 수 있게 하는 고급 기능입니다. 봇은 음성, 비디오, 화면 공유 실시간 형식에 대한 "원시" 액세스를 갖습니다. 이 문서에서는 오디오/비디오 호출 봇을 빌드하고 실시간 형식에 액세스하는 개요를 제공합니다.

이 문서에서는 봇이 Azure Cloud Service에서 웹 역할이나 작업자 역할 자체 서명 ASP.NET Web API 프레임워크로 실행됩니다.

> [!IMPORTANT]
> 이 문서의 콘텐츠는 예비 상태입니다. 샘플 실시간 미디어 봇의 전체 코드는 GitHub 리포지토리 <a href="https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling">BotBuilder RealTimeMediaCalling</a>의 Samples 폴더를 참조하세요.

## <a name="configure-the-service-hosting-the-real-time-media-bot"></a>실시간 미디어 봇을 호스팅하는 서비스 구성

실시간 미디어 플랫폼을 사용하려면 다음 서비스 구성이 필요합니다.

* 봇은 서비스의 FQDN(Fully-Qualified Domain Name)을 알고 있어야 합니다. 이것은 Azure RoleEnvironment API에서 제공되는 것이 아닙니다. 그 대신 FQDN은 봇의 클라우드 서비스 구성에 저장되고 서비스 시작 중에 읽어야 합니다.

* Bot Service에는 인식할 수 있는 인증 기관에서 발급한 인증서가 있어야 합니다. 인증서의 지문은 봇의 클라우드 서비스 구성에 저장되고 서비스 시작 중에 읽어야 합니다.

* 공용 <a href="/azure/cloud-services/cloud-services-enable-communication-role-instances#instance-input-endpoint">인스턴스 입력 엔드포인트</a>를 프로비전해야 합니다. 이렇게 하면 봇 서비스의 각 VM(가상 머신) 인스턴스에 고유한 공공 포트가 할당됩니다. 이 포트는 실시간 미디어 플랫폼이 Skype Calling Cloud와 통신하는 데 사용됩니다.
  ```xml
  <InstanceInputEndpoint name="InstanceMediaControlEndpoint" protocol="tcp" localPort="20100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="20200" min="20101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

  콜백 및 알림과 관련한 호출에 대해 다른 인스턴스 입력 엔드포인트를 만들 때도 유용합니다. 인스턴스 입력 엔드포인트를 사용하면 콜백 및 알림이 호출에 대한 실시간 미디어 세션을 호스팅하는 서비스 배포에서 동일한 VM 인스턴스에 전달되게 할 수 있습니다.
  ```xml
  <InstanceInputEndpoint name="InstanceCallControlEndpoint" protocol="tcp" localPort="10100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="10200" min="10101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

* 각 VM 인스턴스에는 ILPIP(인스턴스 수준 공용 IP 주소)가 있어야 합니다. 시작 중에 봇은 각 서비스 인스턴스에 할당된 ILPIP 주소를 검색해야 합니다. ILPIP 획득 및 구성에 대한 자세한 내용은 <a href="/azure/virtual-network/virtual-networks-instance-level-public-ip">ILPIP</a>를 참조하세요.
  ```xml
  <NetworkConfiguration>
  <AddressAssignments>
    <InstanceAddress roleName="WorkerRole">
    <PublicIPs>
        <PublicIP name="InstancePublicIP" domainNameLabel="InstancePublicIP" />
    </PublicIPs>
    </InstanceAddress>
  </AddressAssignments>
  </NetworkConfiguration>
  ```

* 서비스 인스턴스 시작 중에 `MediaPlatformStartupScript.bat` 스크립트(Nuget 패키지의 일부로 제공)를 상승된 권한을 통해 스타트업 작업으로 실행해야 합니다. 스크립트 실행은 플랫폼의 초기화 메서드를 호출하기 전에 완료되어야 합니다. 

```xml
<Startup>
<Task commandLine="MediaPlatformStartupScript.bat" executionContext="elevated" taskType="simple" />      
</Startup> 
```

## <a name="initialize-the-media-platform-on-service-startup"></a>서비스 시작 시 미디어 플랫폼 초기화

서비스 인스턴스 시작 중에 실시간 미디어 플랫폼을 초기화해야 합니다. 이 작업은 봇이 해당 인스턴스에서 오디오/비디오 Skype 호출을 수락할 수 있게 되기 전에 한 번만 수행합니다. 미디어 플랫폼 초기화를 위해서는 서비스의 FQDN, 공개 ILPIP 주소, 인스턴스 입력 엔드포인트 포트 값, 봇의 **Microsoft 앱 ID** 등을 포함한 여러 서비스 구성 설정을 제공해야 합니다.

> [!NOTE]
> 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

```cs
var mediaPlatformSettings = new MediaPlatformSettings()
{
    MediaPlatformInstanceSettings = new MediaPlatformInstanceSettings()
    {
        CertificateThumbprint = certificateThumbprint,
        InstanceInternalPort = instanceMediaControlEndpointInternalPort,
        InstancePublicIPAddress = instancePublicIPAddress,
        InstancePublicPort = instanceMediaControlEndpointPublicPort,
        ServiceFqdn = serviceFqdn
    },

    ApplicationId = MicrosoftAppId
};

MediaPlatform.Initialize(mediaPlatformSettings);            
```

## <a name="register-to-receive-incoming-call-requests"></a>걸려오는 호출 요청을 수신하도록 등록

`CallController` 클래스를 정의합니다. 이렇게 하면 Bot Service가 걸려오는 Skype 호출에 대해 등록하고 콜백 및 알림 요청을 적합한 `RealTimeMediaCall` 개체에 전달되게 할 수 있습니다.

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallController : ApiController
{
    static CallController()
    {
        RealTimeMediaCalling.RegisterRealTimeMediaCallingBot(
            c => { return new RealTimeMediaCall(c); },
            new RealTimeMediaCallingBotServiceSettings()
        );
    }

    [Route("call")]
    public async Task<HttpResponseMessage> OnIncomingCallAsync()
    {
        // forwards the incoming call to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.IncomingCall);
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> OnCallbackAsync()
    {
        // forwards the incoming callback to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.CallingEvent);
    }

    [Route("notification")]
    public async Task<HttpResponseMessage> OnNotificationAsync()
    {
        // forwards the incoming notification to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.NotificationEvent);
    }
}
```

`RealTimeMediaCallingBotServiceSettings`는 `IRealTimeMediaCallServiceSettings`를 구현하며 콜백 및 알림 이벤트에 대한 웹후크 링크를 제공합니다.

## <a name="register-for-incoming-events-for-the-call"></a>호출에 대해 걸려오는 이벤트 등록

`IRealTimeMediaCall`을 구현하는 `RealTimeMediaCall` 클래스를 정의합니다. 봇이 수신한 각각의 호출에 대해 Bot Framework가 `RealTimeMediaCall` 인스턴스를 만듭니다. 생성자에 전달된 `IRealTimeMediaCallService` 개체를 통해 봇은 실시간 미디어 호출에 연결된 이벤트를 처리하기 위해 이벤트에 대해 등록할 수 있습니다.

```cs
internal class RealTimeMediaCall : IRealTimeMediaCall
{
     public RealTimeMediaCall(IRealTimeMediaCallService callService)
     {
         if (callService == null)
             throw new ArgumentNullException(nameof(callService));

         CallService = callService;
         CorrelationId = callService.CorrelationId;
         CallId = CorrelationId + ":" + Guid.NewGuid().ToString();

         // Register for the call events
         CallService.OnIncomingCallReceived += OnIncomingCallReceived;
         CallService.OnAnswerAppHostedMediaCompleted += OnAnswerAppHostedMediaCompleted;
         CallService.OnCallStateChangeNotification += OnCallStateChangeNotification;
         CallService.OnRosterUpdateNotification += OnRosterUpdateNotification;
         CallService.OnCallCleanup += OnCallCleanup;
     }
}
```

## <a name="create-audio-and-video-sockets"></a>오디오 및 비디오 소켓 만들기
봇이 걸려 오는 오디오/비디오 Skype 호출을 수락하기 전에 먼저 실시간 미디어 송수신 지원을 위해 `AudioSocket` 및 `VideoSocket` 개체를 만들어야 합니다. 봇이 미디어를 지원하지 않으려면 AudioSocket만 만들어야 합니다.

봇은 지원하려는 형식을 결정하고 적합한 AudioSocket 및 VideoSocket 개체를 만들어야 합니다. 봇은 걸려오는 호출을 수락한 후에는 지원할 상태를 변경할 수 없습니다.

각 AudioSocket 및 VideoSocket에 대해 봇은 미디어 소켓이 미디어 송수신을 모두 지원할지 또는 송신이나 수신만 지원할지 지정합니다. 예를 들어 봇은 오디오는 송수신하고("Sendrecv") 비디오는 송신만("Sendonly")하고자 할 수 있습니다. 봇은 각 미디어 소켓에 지원할 미디어 형식도 지정해야 합니다. AudioSocket에 현재 지원되는 형식은 "Pcm16K": (서명) 16비트 PCM 인코딩 16KHz 샘플링 속도입니다. VideoSocket의 경우 송수신 미디어 형식을 별도로 지정합니다. 비디오 수신에는 "NV12" 형식만 지원되지만 송신에는 여러 다른 형식을 지원할 수 있습니다.

```cs
_audioSocket = new AudioSocket(new AudioSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    SupportedAudioFormat = AudioFormat.Pcm16K,
    CallId = correlationId
});

_videoSocket = new VideoSocket(new VideoSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    ReceiveColorFormat = VideoColorFormat.NV12,
    SupportedSendVideoFormats = new VideoFormat[]
    {
        VideoFormat.Yuy2_1280x720_30Fps,
        VideoFormat.Yuy2_720x1280_30Fps,
    },
    CallId = correlationId
});

_audioSocket.AudioMediaReceived += OnAudioMediaReceived;
_audioSocket.AudioSendStatusChanged += OnAudioSendStatusChanged;
_audioSocket.DominantSpeakerChanged += OnDominantSpeakerChanged;
_videoSocket.VideoMediaReceived += OnVideoMediaReceived;
_videoSocket.VideoSendStatusChanged += OnVideoSendStatusChanged;
```                             

## <a name="create-a-mediaconfiguration"></a>MediaConfiguration 만들기
미디어 소켓을 만든 후에는 봇이 걸려오는 오디오/비디오 Skype 호출과 미디어 소켓을 연결하는 데 필요한 `MediaConfiguration` 개체를 만들어야 합니다.

```cs
var mediaConfiguration = MediaPlatform.CreateMediaConfiguration(_audioSocket, _videoSocket);
```

##  <a name="answer-an-incoming-audiovideo-call"></a>걸려오는 오디오/비디오 호출에 응답
`OnIncomingCallReceived` 이벤트는 걸려오는 Skype 오디오/비디오 호출을 봇이 수락할 수 있게 하기 위해 호출됩니다. 이를 위해 봇은 `MediaConfiguration` 개체가 있는 `AnswerAppHostedMedia` 개체를 만듭니다. 봇이 이 Skype 호출과 관련된 알림에 대해 등록합니다.

```cs
private Task OnIncomingCallReceived(RealTimeMediaIncomingCallEvent incomingCallEvent)
{
    // ... create Audio/VideoSocket objects and MediaConfiguration ...

    incomingCallEvent.RealTimeMediaWorkflow.Actions = new ActionBase[]
    {
        new AnswerAppHostedMedia
        {
            MediaConfiguration = mediaConfiguration,
            OperationId = Guid.NewGuid().ToString()
        }
    };

    // subscribe for roster and call state changes
    incomingCallEvent.RealTimeMediaWorkflow.NotificationSubscriptions = new NotificationType[]
    {
        NotificationType.CallStateChange,
        NotificationType.RosterUpdate
    };
}
```

## <a name="outcome-of-the-call"></a>호출의 결과
`AnswerAppHostedMedia` 작업이 완료되면 `OnAnswerAppHostedMediaCompleted`가 발생합니다. `AnswerAppHostedMediaOutcomeEvent`의 `Outcome` 속성은 성공 또는 실패를 나타냅니다. 호출을 설정할 수 없으면 봇이 해당 호출을 위해 만든 AudioSocket 및 VideoSocket 개체를 삭제해야 합니다.

## <a name="receive-audio-media"></a>오디오 미디어 수신
오디오 수신 기능을 통해 `AudioSocket`을 만든 경우 오디오 프레임을 수신할 때마다 `AudioMediaReceived` 이벤트가 호출됩니다. 봇은 오디오 콘텐츠를 소싱하고 있는 피어에 관계없이 초당 약 50회 이러한 이벤트를 처리합니다(오디오가 피어로부터 수신되지 않으면 쾌적 노이즈 버퍼가 로컬로 생성되기 때문). 오디오 콘텐츠의 각 패킷은 `AudioMediaBuffer` 개체로 배달됩니다. 이 개체는 디코딩된 오디오 콘텐츠를 포함하는 기본 힙 할당 메모리 버퍼에 대한 포인터를 포함합니다. 

```cs
void OnAudioMediaReceived(
            object sender,
            AudioMediaReceivedEventArgs args)
{
   var buffer = args.Buffer;

   // native heap-allocated memory containing decoded content
   IntPtr rawData = buffer.Data;            
}
```

이벤트 처리기는 신속하게 반환해야 합니다. 애플리케이션은 `AudioMediaBuffer`을 비동기로 처리되도록 큐에 넣는 것이 좋습니다. `OnAudioMediaReceived` 이벤트는 실시간 미디어 플랫폼으로 직렬화됩니다(즉 다음 이벤트는 현재 이벤트가 반환될 때까지 발생하지 않음). `AudioMediaBuffer`를 사용한 후에는 미디어 플랫폼에서 관리되지 않는 기본 메모리를 재청구할 수 있게 애플리케이션이 버퍼의 Dispose 메서드를 호출해야 합니다. 

```cs
   // release/dispose buffer when done 
   buffer.Dispose();
```

> [!IMPORTANT]
> 봇은 버퍼 액세스를 마칠 때까지는 버퍼의 Dispose 메서드를 호출해서는 안 됩니다.

## <a name="receive-video-media"></a>비디오 미디어 수신
비디오 수신은 초당 버퍼 수가 프레임 속도 값에 따라 달라진다는 점을 제외하고는 오디오 수신과 유사합니다. `VideoMediaBuffer`에는 `VideoFormat` 및 `OriginalVideoFormat` 속성이 있습니다. `OriginalVideoFormat`은 소싱된 시점의 원래 버퍼 형식을 나타냅니다. `IVideoSocket.VideoMediaReceived` 이벤트 처리기를 통해 비디오 버퍼를 수신할 때만 사용할 수 있습니다. 버퍼를 전송하기 전에 크기를 조절한 경우 `OriginalVideoFormat` 속성은 원래의 너비와 높이를 갖지만 `VideoFormat`은 크기 조정 후 현재의 높이와 너비를 갖게 됩니다. `OriginalVideoFormat`의 너비와 높이 속성이 `VideoFormat` 속성과 다를 경우 `VideoMediaReceived` 이벤트에서 발생한 `VideoMediaBuffer`의 소비자는 `OriginalVideoFormat` 크기에 맞게 버퍼 크기를 조절해야 합니다. 현재 NV12 형식만 수신이 지원됩니다.

## <a name="send-audio-media"></a>오디오 미디어 보내기
`AudioSocket`이 미디어를 보내도록 구성된 경우 봇은 `AudioSocket`에서 `AudioSendStatusChanged` 이벤트 처리기에 대해 등록해야 보내기 상태 변경 내용에 관한 알림을 받을 수 있습니다. 봇은 보내기 상태가 활성으로 변경된 후에만 오디오 보내기를 시작할 수 있습니다.

```cs
void AudioSocket_OnSendStatusChanged(
             object sender,
             AudioSendStatusChangedEventArgs args)
{
    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

오디오 미디어를 보내려면 PCM 오디오 콘텐츠가 원시 힙 할당 버퍼 안에 포함되었다고 가정합니다. 봇은 `AudioMediaBuffer` 추상 클래스에서 파생되어야 합니다. 미디어는 AudioSocket의 `Send` 메서드를 통해 비동기로 보내지며 플랫폼은 보내기가 완료되면 `AudioMediaBuffer`에서 `Dispose`를 호출합니다. 봇은 `Send`가 반환할 때 관리되지 않은 리소스를 릴리스해서는 안됩니다(또는 풀 할당자 반환). `Dispose` 호출을 대기해야 합니다.

## <a name="send-video-media"></a>비디오 미디어 보내기
비디오 미디어 보내기는 오디오 미디어 보내기와 유사합니다. 봇은 `VideoSendStatusChanged` 이벤트에 대해 등록하고 `MediaSendStatus`의 `Active`를 대기해야 합니다. 봇은 `Dispose` 메서드가 호출될 때까지 버퍼의 관리되지 않는 리소스를 릴리스하거나 회수해서는 안 됩니다. RGB24, NV12 및 Yuy2 색 형식이 지원됩니다.

여러 `VideoFormat`이 비디오 보내기에 지원되지만 현재 선호되는 `VideoFormat`이 `VideoSendStatusChanged` 이벤트를 통해 봇에게 알립니다. 네트워크 대역폭 상황이나 비디오 창 크기를 조정하는 피어 클라이언트에 따라 비디오 보내기에 선호하는 `VideoFormat`은 변경될 수 있습니다.

```cs
void VideoSocket_OnSendStatusChanged(
            object sender,
            VideoSendStatusChangedEventArgs args)
{
    VideoFormat preferredVideoFormat;

    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        // bot is recommended to use this format for sourcing video content.
        preferredVideoFormat = args.PreferredVideoSourceFormat;
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

각 `VideoMediaBuffer`에는 특정 버퍼에 대한 비디오 콘텐츠 형식을 표시하는 VideoFormat 속성이 있습니다. `VideoFormat` 속성은 `VideoSendStatusChanged` 이벤트에 표시된 `PreferredVideoSourceFormat` 속성과 일치할 필요는 없지만 비디오 프레임 크기 조정에 소모되는 CPU 주기 낭비를 방지할 수 있게 표시된 `PreferredVideoSourceFormat`을 사용하는 것이 좋습니다.

## <a name="call-notifications"></a>호출 알림
### <a name="call-state-changes"></a>호출 상태 변경
봇은 `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow`의 `NotificationSubscriptions`에서 `NotificationType.CallStateChange`를 구독하여 호출 상태 변경 알림을 받을 수 있습니다.

```cs
private Task OnCallStateChangeNotification(CallStateChangeNotification callStateChangeNotification)
{
    if (callStateChangeNotification.CurrentState == CallState.Terminated)
    {   
        // stop sending media and dispose the media sockets
        _audioSocket.Dispose();
        _videoSocket.Dispose();
    }

    return Task.CompletedTask;
}
 ```

### <a name="roster-update"></a>명단 업데이트
봇이 회의에 추가되면 `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow`의 `NotificationSubscriptions`에서 `NotificationType.RosterUpdate`를 구독하여 회의 명단을 수신 대기할 수 있습니다. `RosterUpdateNotification`에는 모든 회의 참가자에 대한 세부 정보가 있습니다. 봇은 참가자가 비디오를 보냈다는 알림을 기다린 다음, `VideoSocket` 개체 중 하나에서 참가자의 비디오를 `Subscribe`하도록 선택할 수 있습니다.

```cs
private async Task OnRosterUpdateNotification(RosterUpdateNotification rosterUpdateNotification)
{
    // Find a video source in the conference to subscribe
    foreach (RosterParticipant participant in rosterUpdateNotification.Participants)
    {
        if (participant.MediaType == ModalityType.Video
            && (participant.MediaStreamDirection == "sendonly" || participant.MediaStreamDirection == "sendrecv")
            )
        {
            var videoSubscription = new VideoSubscription
            {
                ParticipantIdentity = participant.Identity,
                OperationId = Guid.NewGuid().ToString(),
                SocketId = _videoSocket.SocketId,
                VideoModality = ModalityType.Video,
                VideoResolution = ResolutionFormat.Hd720p
            };

            await CallService.Subscribe(videoSubscription).ConfigureAwait(false);
            break;
        }
    }  
}
```

## <a name="end-the-call"></a>통화 종료

### <a name="handle-call-termination-from-the-caller"></a>발신자의 통화 종료 처리
사용자가 피어 투 피어 대화에서 봇에 대한 호출을 종료하거나 봇이 회의에서 제거된 경우, 봇은 CallState가 Terminated로 되었다는 통화 상태 변경 알림을 받게 됩니다. 봇은 자신이 만든 모든 미디어 소켓을 삭제해야 합니다.

### <a name="terminate-the-call-from-the-bot"></a>봇에서 통화 종료
봇이 `IRealTimeMediaCallService`에서 `EndCall`을 호출하여 통화를 종료하도록 선택할 수 있습니다.

### <a name="handle-call-clean-up-by-the-bot-framework"></a>Bot Framework를 통한 호출 정리 처리
오류 상황(예: `AnswerAppHostedMediaOutcomeEvent`가 합당한 시간 내에 수신되지 않은 경우)에서 Bot Framework는 호출을 종료할 수 있습니다. 봇은 `OnCallCleanup` 이벤트에 대해 등록하고 미디어 소켓을 삭제해야 합니다. 

-->