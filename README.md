# neko-signalr-chat
- Asp.net core 3.1 + SignalR, Chatting sample
- Step 01 : Create Asp.net core 3.1 + Web Application  
- Step 02 : SignalR js 클라이언트 라이브러리 추가
```
솔루션 탐색기에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 추가 > 클라이언트 쪽 라이브러리를 선택합니다.

클라이언트 쪽 라이브러리 추가 대화 상자에서 공급자로 unpkg를 선택합니다.

라이브러리인 경우 @microsoft/signalr@latest를 입력합니다.

특정 파일 선택을 선택하고 dist/browser 폴더를 확장한 다음 signalr.js 및 signalr.min.js를 선택합니다.

대상 위치를 wwwroot/js/signalr/ 로 설정하고 설치를 선택합니다.
```
- Step 03 : SignalR Hub 만들기 
```
[Hubs/ChatHub.cs]
namespace SignalRChat.Hubs
{
    public class ChatHub : Hub
    {
        public async Task SendMessage(string user, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }
    }
}
```
- Step 04 : 서버구성 (설정)
```
[Startup.cs]
using SignalRChat.Hubs;

public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR();
    services.AddCors(options =>
    {
        options.AddPolicy("CorsPolicy", builder => builder.AllowAnyMethod().AllowAnyHeader().AllowCredentials().SetIsOriginAllowed((host) => true).Build());
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseCors("CorsPolicy");  //C2
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chathub");
    });
}
```
- Step 05 : 클라이언트 코드 (html + js)
```
[/Pages/index.cshtml]
@page
    <input type="text" id="userInput" />
    <input type="text" id="messageInput" />
    <input type="button" id="sendButton" value="SEnd Message" />

    <ul id="messagesList"></ul>

<script src="~/js/signalr/dist/browser/signalr.js"></script>
<script src="~/js/chat.js"></script>
```

```
[/wwwroot/js/chat.js]
'use strict';

//커넥션
var connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();

//센드버튼 비활성화
document.getElementById('sendButton').disabled = true;

//메시지받으면 목록에 추가
connection.on('ReceiveMessage', function (user, message) {
    var msg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
    var encodedMsg = user + " says " + msg;
    var li = document.createElement("li");
    li.textContent = encodedMsg;
    document.getElementById("messagesList").appendChild(li);
});

//커넥션 시작 + 센드버튼 활성화
connection.start().then(function () {
    document.getElementById("sendButton").disabled = false;
}).catch(function (err) {
    return console.error(err.toString());
});

//센드버튼 클릭시 서버로 메시지 전송
document.getElementById("sendButton").addEventListener("click", function (event) {
    var user = document.getElementById("userInput").value;
    var message = document.getElementById("messageInput").value;
    connection.invoke("sendMessage", user, message).catch(function(err) {
        return console.log(err.toString());
    });
    event.preventDefault();
});
```
- Step 06 : Run (F5)




