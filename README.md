## Http(Header, Body)

인터넷에서 데이터를 주고받을 때 HTTP를 많이 사용한다.  
웹브라우저에서 서버에 연결하고 웹페이지를 받아와 화면에 보여줄 때 사용하는 것이 HTTP이기 때문에  
인터넷에서 웹페이지를 요청하고 응답으로 받을 때 HTTP를 사용한다고 이해하면 된다.  
HTTP는 앞서 공부한 Socket을 기반으로 동작하며 데이터를 주고 받을 때 HTTP표준 프로토콜 규격에 맞춘다.  

+ HTTP 요청과 응답  
  데이터를 주고받을 때의 포맷은 헤더(Header)와 바디(Body)로 구분되고  
  헤더에 어떤것이 들어갈 수 있는지가 규격으로 정해져 있다. 바디에는 보내고 받기위한 대상 데이터를 넣어둘 수 있다.

+ 요청 포맷  
  HTTP 요청 포맷을 보면, 첫 번째 줄이 기본적인 요청 정보를 포함하고 있다.  
  GET이나 POST와 같은 요청 방식(Method), 요청 패스 그리고 HTTP 버전 등이 들어가 있다.  
  헤더에 들어가 있는 각각의 줄은 하나의 속성을 나타내고 속성이름 + 콜론(:) + 속성값으로 구성된다.  
  각각의 줄은 \r\n 코드로 구분되므로 한 줄씩 보이게 되며, 헤더와 바디는 \r\n으로 구분되므로  
  헤더와 바디가 한 줄 더 띄워져 있는 모양이라고 생각하면 된다. 바디에는 전송하고자 하는 데이터를 넣을 수 있다.

+ 응답 포맷  
  응답 포맷은 요청 포맷과 크게 다르지 않다. 헤더와 바디로 구분되고 헤더에는 한 줄씩 속성이 들어간다.  
  응답의 첫 줄은 상태를 나타내며 HTTP 버전과 응답 코드, 응답 메시지 등으로 구성된다.
  
## Web - HttpConnection(코드 참조해서 보기)
앱에서 웹서버에 요청하는 방식은 표준 자바를 이용해 요청할 때와 크게 다르지 않다. 다만 Thread를 사용해야한다.  
웹으로 요청을하고 응답을 받으면 응답데이터를 화면에 보여줄 수 있다.  
그리고 그 응답데이터는 웹페이지일 수도 있고 일반데이터일 수도 있다.  
웹브라우저에서는 보통 웹페이지를 응답으로 받아 보여주지만,  
앱에서는 화면을 이미 보여주고 있으므로 데이터만 받아 화면의 뷰에 데이터를 뿌리는 경우가 많다.  

+ Http_URL_Connection
  표준자바에서 HTTP클라이언트를 만드는 가장 기본적인 방법은 HttpURLConnection 객체를 사용하는 것이다. 버튼을 추가하고 클릭이벤트로 요청스레드 객체를 하나 만들어 시작한다.    
  ```JAVA
    requestBtn.setOnClickListener(new OnClickListener() {
    public void onClick(View v) {
      String urlStr = input01.getText().toString();
	
      ConnectThread thread = new ConnectThread(urlStr);
      thread.start();
    }
  }); 
  ```
  위 스레드 안에서는 웹으로 요청을 보내고 응답을 받을 수 있다. 스레드 안에 만들어진 request 메소드를 보면 HttpURLConnection 객체를 사용하고 있다. 먼저 사용자가 입력한 URL 정보를 이용해 URL 객체를 만들고 openConnection 메소드를 호출하면 HttpURLConnection 객체가 반환된다. 이 객체에 몇 가지 속성을 설정하고 getResponseCode 메소드를 호출하면 웹서버에 연결하고 응답을 받아준다. 연결이 만들어진 객체의 getInputStream 메소드를 호출하면 InputStream 객체를 참조할 수 있으며 이 객체로부터 응답 데이터를 읽어 들인다. (BufferdReader 객체는 한 줄씩 읽어 들일 때 유용하게 사용)
  
  ```JAVA
  URL url = new URL(urlStr);
  HttpURLConnection conn = (HttpURLConnection)url.openConnection();
  if (conn != null) {
      conn.setConnectTimeout(10000);
      conn.setRequestMethod("GET");
      conn.setDoInput(true);
      conn.setDoOutput(true);

      int resCode = conn.getResponseCode();
      BufferedReader reader = new BufferedReader(
			  	new InputStreamReader(conn.getInputStream())) ;
  ```
+ 응답 데이터를 화면에 표시하기
  Request 메소드를 호출하여 응답 데이터를 읽어 들였다면 화면에 표시할 수 있다. 화면을 위한 XML 레이아웃에 TextView를 추가했었다면 이 텍스트뷰의 setText 메소드나 append 메소드를 호출하면 글자가 보이게 된다. 다만 스레드에서 응답을 받은 것이므로 핸들러를 반드시 이용해야 한다는 것을 명심하자.
  ```JAVA
  final String output = request(urlStr);
  handler.post(new Runnable() {
    public void run() {
        txtMsg.setText(output);
    }
  });
  ```
  
  ** volley 나 okhttp 등 라이브러리 사용하면 스레드 신경안써두됨 그래도 기본은 알아야지
