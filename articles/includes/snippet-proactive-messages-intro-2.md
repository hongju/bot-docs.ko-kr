**임시 자동 관리 메시지**는 가장 간단한 형식의 자동 관리 메시지입니다.
봇은 메시지가 트리거될 때마다 사용자가 현재 봇과 다른 주제의 대화에 참여하고 있으며 대화를 변경하지 않으려고 하는지 여부에 관계없이 메시지를 간단히 대화에 삽입합니다.

알림을 더 원활하게 처리하려면 대화 상태에 플래그를 설정하거나 알림을 큐에 추가하는 것처럼 알림을 대화 흐름에 통합하는 다른 방법을 사용하는 것이 좋습니다.

<!--Snip
A **dialog-based proactive message** is more complex than an ad hoc proactive message. 
Before it can inject this type of proactive message into the conversation, 
the bot must identify the context of the existing conversation and decide how (or if)
it will resume that conversation after the message interrupts. 

For example, consider a bot that needs to initiate a survey at a given point in time. 
When that time arrives, the bot stops the existing conversation with the user and 
redirects the user to a `SurveyDialog`. 
The `SurveyDialog` is added to the top of the dialog stack and takes control of the conversation. 
When the user finishes all required tasks at the `SurveyDialog`, the `SurveyDialog` closes,
 returning control to the previous dialog, where the user can continue with the prior topic of conversation.

A dialog-based proactive message is more than just simple notification. 
In sending the notification, the bot changes the topic of the existing conversation. 
It then must decide whether to resume that conversation later, or to abandon that conversation altogether by resetting the dialog stack. 
/Snip-->