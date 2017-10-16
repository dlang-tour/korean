# Vibe.d 프레임워크 기본과 비동기 I/O (Basics & Asynchronous I/O)

기본 상태에서, vibe.d 애플리케이션에서의 실질적 `main()` 함수 역할은 `shared static this()` 라는
특별한 함수가 합니다. 이 함수는 모듈 구성함수(module constructor)로써 표면상의 `main()` 역할을 하게 됩니다.

```d
    import vibe.d;
    shared static this() {
        // Vibe.d 코드가 이 안에 작성됩니다
    }
```

모듈 구성함수가 먼저 실행된 후에 `main()` 이 호출됩니다. vibe.d의 `main()` 함수는 자동으로 제공되며,
`main()` 내에는 프레임워크 구동과 이벤트 루프(event loop) 등에 필요한 코드가 포함되어있습니다.

vibe.d 는 논블로킹(non-blocking)이라고도 불리는 비동기 I/O 구현을 위해 *파이버(fiber)* 를 사용합니다.
보통 네트워크에 연결된 소켓을 다루는 작업은  응답이 도착할 때까지 기다리는 블로킹(blocking) 방법으로 동작하는데,
파이버를 사용하지 않으면 데이터가 올때까지 무한정 기다려야하는 문제가 있습니다.
하지만 vibe.d는 파이버를 사용하고 있기에, 데이터를 기다려야하면 `yield` 로 다른 작업을 할 수 있도록 양보했다가
데이터가 준비되면 다시 원래 하던 작업으로 돌아와 효율을 높여줍니다.

```d
    // 블로킹이 일어나지만 특별한 추가작업이 필요하진 않습니다. (transparent layer)
    // 읽기 작업 시 데이터가 없으면 다른 작업을 할 것이며, 준비되면 이곳으로
    // 돌아옵니다.
    line = connection.readLine();
    // 위의 경우와 마찬가지입니다.
    connection.write(line);
```

vibe.d는 여기서 다시금 파이버의 장점을 활용합니다. 최종적으로 작성된 코드는 *동기식(synchronous)* 형태로 작성되어
블로킹 방식으로 동작할 것 같지만, 실제로 동작할 때는 자동으로 비동기 방식으로 동작하게 됩니다. 덕분에 간결한 코드로도
하나의 CPU 코어로 수천개의 네트워크 연결을 처리할 수 있는 성능을 보일 수 있게 됩니다.

모든 vibe.d 의 기능들은 파이버와 함께 비동기 소켓 I/O 방식을 사용하고 있습니다.
그래서 예를 들어 MongoDB 연결 하나가 느려지더라도, 전체 애플리케이션이 느려지는 일을 걱정할 필요가 없습니다.

실제로 동작하는 예제를 보는 게 좋습니다. 꼭 아래의 TCP 에코(echo) 서버 예제를 살펴보시기 바랍니다.

## {SourceCode:disabled}

```d
import vibe.d;

shared static this()
{
    // TCP 8080 포트로 오는 연결을 기다립니다.
    // 새 연결이 들어올 때마다 델리게이트에 TCP 연결을
    // 담은 객체가 전달됩니다.
    // 이 연결은 클라이언트로부터 온 연결입니다.
    listenTCP(8080, (TCPConnection conn) {
        string line;
        conn.write("ECHO 서버의 인사입니다!\r\n");
        conn.write("반가워요!");
        conn.write("'quit' 을 입력해 마칩니다.\r\n");
        while (line != "quit") {
            line = cast(string) conn.readLine();
            conn.write("ECHO: " ~ line
              ~ "\r\n");
        }

        // TCP 연결을 직접 닫지 않아도,
        // 이 델리게이트 코드의 끝에 도달한 순간
        // 자동으로 닫힙니다.
    });

    // 참고로 이 예제는
    // 이 애플리케이션을 가동한 뒤에
    // telnet 클라이언트를 사용해 접속하시면
    // 테스트할 수 있습니다.
}
```
