# WebServer, WAS 와 Apache, Nginx

Web Server 와 WAS 에 대해 본격적으로 알아보기 이전에 Web 에 대한 기초적인 용어를 알고 가면 좋습니다.

## 📖 WEB, Client 와 Server

Internet 을 통해 우리는 여러 정보, Web Resource 를 공유합니다.

> Internet : 여러 네트워크들이 연결되어 통신하는것, 또는 이를 서비스화 한 것 입니다. TCP/IP 프로토콜 위에서 동작합니다.
>
> Web Resource : URI 라는 고유한 문자열을 이용해 리소스를 식별합니다
>
> URI : URI 는 해당 리소스를 어디서 가져와야하는지에 대한 정보를 담은 URL 과 어떤 리소스를 가져와야하는지에 대한 정보를 담은 URN 으로 구성됩니다



웹 브라우저 (Client) 가 서버 (Server) 에 요청을 보내고, 응답을 받아 Web Resource 를 표시합니다.

> &#x20;Client-Server Architecture : 웹 리소스 요청하는 클라이언트와, 요청을 받아 처리하고 데이터를 응답하는 서버로 구분하는 아키텍쳐입니다.
>
> Protocol : 네트워크에서 통신하기 위한 규칙의 집합입니다.
>
> HTTP : 웹에서 데이터를 주고 받기 위한 Protocol 입니다



### ✏️ Web Server

Web Server 는 클라이언트의 요청에 따라 변경되지 않는 <mark style="background-color:orange;">정적 콘텐츠</mark> (HTML, CSS, JS, 이미지, 동영상 ...) 를 제공하는 역할을 합니다

웹 서버에는 Apache, NginX 가 있습니다.

### ✏️ Web Application Server

Web Application Server 는 동적인 웹 컨텐츠를 생성하고 실행하기 위한 서버입니다.

서버 측 스크립팅언어인 PHP, Java, Python 등 을 실행해서 <mark style="background-color:orange;">동적인 컨텐츠</mark>를 생성하고 클라이언트에 전송하는 역할을 수행합니다

웹 서버와 연동하여 클라이언트 요청에 따라 데이터베이스 조회, 비즈니스 로직 처리 등을 수행 한 후, 결과를 클라이언트에 전달합니다.



웹 애플리케이션은 클라이언트의 요청 처리 방식에 따라

> Client > Web Server > DB
>
> Client > WAS > DB
>
> Client > Web Server > WAS > DB

등 다양한 구조를 가질 수 있습니다

Web Application Server 는 데이터베이스 조회, 비즈니스 로직 처리 등 다양한 로직들을 처리하는데 집중되어있어,\
단순한 정적 컨텐츠를 Web Server 에 맡겨 기능을 분리시켜 서버 부하를 방지 할 수 있습니다.







