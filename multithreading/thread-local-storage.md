# 쓰레드 지역 변수(Thread local storage)

`static` 키워드는 여러 용도로 사용되지만, 대부분은 프로그램 실행시 단 한번 초기화되어 계속 사용되는 정적 타입 변수를 만들 때 사용됩니다. 한번 초기화가 완료되면, 이후로 초기화가 이루어지지 않습니다.

D 언어의 `static` 은 쓰레드와 함께 쓰이면 독특한 특성을 보여줍니다. Java나 C/C++에서는 `static` 으로 선언된 변수가 쓰레드 간에 공유되고, 이 때문에 2개 이상의 쓰레드가 동시에 변수의 상태를 바꾸려는 작업을 하려면 순차적으로 진행되도록 동기화(synchronization) 작업이 필연적으로 필요합니다.

하지만 D 언어의 `static` 은 *각 쓰레드마다 따로 관리* 되며, 각 쓰레드 내에서 `static` 특성은 유지하지만 다른 쓰레드와 변수가 공유되지 않습니다.

`static` 으로 선언된 변수에 값을 지정하려면, 컴파일 시점에 그 값이 무엇인지 알 수 있는 확정된 값을 지정해야합니다. 프로그램을 실행해야 알 수 있는 값은 쓰일 수 없습니다. 만약 프로그램 실행 후에 단순히 `static` 변수를 초기화하려면 `static this()` 라는 일회용 생성자(constructor)를 `struct`, `class`, 모듈 등에 사용하면 됩니다.

```d
    static int b = 42;
    // b는 42라는 값으로 초기화되었습니다.
    // 다른 쓰레드에서 b 의 값을 바꾸더라도, 새 쓰레드에서는
    // 다른 쓰레드의 상태와 상관없이 42라는 초기상태를 보게 됩니다.
```

혹시 D 언어의 `static` 변수 관리 방법이 마음에 들지 않거나, 과거의 고전적인 동작 방법을 이용할 필요가 있다면, C의 `static` 과 동일한 성격을 부여하는 `__gshared` 변수 특성을 부여하면 됩니다.

이름이 참 불편하게 되어있는 건, 가능한 쓰지 마라는 다정한 충고가 담겨 있는 것입니다.

```d
    __gshared int b = 50;
    // static 처럼 실행 후 단 한 번 초기화됩니다. 단!
    // 모든 쓰레드가 서로 같은 b 변수를 바라보고 있으며,
    // 한 쓰레드가 값을 바꾸게 되면 다른 쓰레드도 영향을 받습니다.
```

### 더 살펴보기

- [Thread-local storage on Wikipedia](https://en.wikipedia.org/wiki/Thread-local_storage)

## {SourceCode}

```d
import std.concurrency : spawn, thisTid;

void worker(bool firstTime)
{
    import std.stdio : writeln;
    // threadState 는 현재 쓰레드 내에서 유효한 상태를 유지합니다.
    // 다른 쓰레드가 읽는 값은 현재 쓰레드의 상태와 다릅니다.
    // 아래 행이 실행될 때, threadState는 단 한 번 초기화됩니다
    static int threadState = 0;
    writeln("쓰레드 번호 ", thisTid,
        ": 현재 상태 = ", threadState++);
    if (firstTime)
        worker(false);
}

void main()
{
    // worker(true, i) 를 실행하는 쓰레드 5개를 생성합니다
    for (size_t i = 0; i < 5; ++i) {
        spawn(&worker, true);
    }
}
```
