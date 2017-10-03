# 메시지 패싱(Message Passing)

쓰레드를 사용하며, 쓰레드 사이의 동기화에 고민하는 대신 *메시지 패싱(message passing)* 을 사용하여 다중 CPU 코어를 충분히 활용해보십시오.

이 방법을 사용하면, 쓰레드 사이에서 *메시지* 로 통신을 하게 되는데, 이 메시지에는 D 언어에서 다룰 수 있는 임의의 값이 담겨있게 됩니다. 이를 이용해 의미 있는 값을 전달하여 작업을 분배하거나, 쓰레드 사이의 동작 순서 등을 결정할 수 있습니다.

D 언어의 쓰레드는 다중 쓰레드 프로그래밍을 할 때 발생할 수 있는 여러 문제를 회피하기 위해, 가능한 변수 등을 공유하지 않도록 설계되어있습니다.

메시지 패싱에 사용되는 모든 표준 함수들은 [`std.concurrency`](https://dlang.org/phobos/std_concurrency.html) 표준 라이브러리 패키지에 정의되어있습니다. `spwan` 으로 새 *쓰레드* 를 만들 때, 쓰레드 내에서 어떤 함수를 실행할지 프로그래머가 지정해주어야합니다.

```d
    auto threadId = spawn(&foo, thisTid);
```

`thisTid` 는 `std.concurrency` 가 기본으로 제공해주는, 메시지를 받길 기다리는 쓰레드에 대한 참조입니다.

`spawn` 은 첫번째 입력값으로 함수를 받으며, 이후에 첫번째 입력값에 제공된 함수를 실행하는데 필요한 입력값들을 매개변수로 받습니다.

```d
    void foo(Tid parentTid) {
        receive(
          (int i) { writeln("An ", i, " was sent!"); }
        );

        send(parentTid, "Done");
    }
```

`receive` 라는 함수가 보이는데, 이 함수는 `switch`-`case` 처럼 사용할 수 있는 함수입니다.

이 함수는 다른 쓰레드에서 전달된 값을 처리하는 `델리게이트(delegate)` 로써, 메시지를 보내는 쓰레드가 몇 개의 값을 보내느냐, 각 값의 타입은 어떻게 되는지에 따라 실행여부가 결정됩니다.

특정 쓰레드에게 메시지를 전달하려면 `send` 라는 함수를 사용하고, 메시지를 받을 쓰레드 아이디를 함께 입력값으로 제공해야합니다.

```d
    send(threadId, 42);
```

`receiveOnly` 는 지정된 한 타입의 한 가지 값만 받을 때 사용됩니다.

```d
    string text = receiveOnly!string();
    assert(text == "Done");
```

`receive` 와 `receive` 로 시작되는 메시지 패싱 관련 함수들은, 원하는 메시지가 도착할 때까지 쓰레드를 일시 정지시키는 효과를 가지고 있습니다.


### 더 살펴보기

- [Exchanging Messages between Threads](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=5)
- [Messaging passing concurrency](http://ddili.org/ders/d.en/concurrency.html)
- [Pattern Matching with receive](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=6)
- [Multi-threaded file copying](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=7)
- [Thread Termination](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=8)
- [Out-of-Band Communication](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=9)
- [Mailbox crowding](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=10)

## {SourceCode}

```d
import std.stdio : writeln;
import std.concurrency : receive, receiveOnly,
    send, spawn, thisTid, Tid;

/*
쓰레드의 메시지로 사용될 struct를 만듭니다. 숫자 하나를 전송할 수 있습니다.
*/
struct NumberMessage {
    int number;
    this(int i) {
        this.number = i;
    }
}

/*
취소 메시지를 보내기 위해 선언한 struct 입니다
*/
struct CancelMessage {
}

/// 취소 메시지를 받았다고 응답(Ack)하기 위한 struct 입니다
struct CancelAckMessage {
}

/*
어느 쓰레드가 새 쓰레드를 시작시켰는지, 쓰레드 아이디를 전달 받는 함수입니다.
이 함수를 spawn 에 전달해 쓰레드 구동에 쓰겠습니다.
*/
void worker(Tid parentId)
{
    bool canceled = false;
    writeln("Starting ", thisTid, "...");

    while (!canceled) {
      receive(
        (NumberMessage m) {
          writeln("Received int: ", m.number);
        },
        (string text) {
          writeln("Received string: ", text);
        },
        (CancelMessage m) {
          writeln("Stopping ", thisTid, "...");
          send(parentId, CancelAckMessage());
          canceled = true;
        }
      );
    }
}

void main()
{
    Tid threads[];
    // 10개의 작업 쓰레드를 만듭니다
    for (size_t i = 0; i < 10; ++i) {
        threads ~= spawn(&worker, thisTid);
    }

    // 홀수번째 인덱스의 쓰레드는 숫자를,
    // 짝수번째 인덱스는 문자열을 메시지로 받습니다
    foreach(int idx, ref tid; threads) {
        import std.string : format;
        if (idx  % 2)
            send(tid, NumberMessage(idx));
        else
            send(tid, format("T=%d", idx));
    }

    // 그리고 모든 쓰레드에, 임의로 정의된 취소 메시지를 보냅니다
    foreach(ref tid; threads) {
        send(tid, CancelMessage());
    }

    // 모든 쓰레드로부터 CancelAckMessage를 받을 때까지 메인 쓰레드에서
    // 기다립니다
    foreach(ref tid; threads) {
        receiveOnly!CancelAckMessage;
        writeln("Received CancelAckMessage!");
    }
}
```
