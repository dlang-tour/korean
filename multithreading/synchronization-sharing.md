# 동기화와 공유(Synchronization & Sharing)

D 언어에서 권하는 다중 쓰레드 운영 방식은 `immutable` 로 선언하여 데이터가 변하지 않도록 하고, 메시지 패싱(message passing)을 사용해 실행 순서를 제어하는 것입니다. 다만, 기존의 락(lock)을 이용한 *동기화(synchronization)* 또한 D 언어에서 지원하고 있습니다.  여러 쓰레드에서 동시에 이용할 값이나 객체는 `shared` 라는 키워드로 표시해둘 수 있습니다.

`shared` 키워드는 여러 쓰레드에서 공유될 수 있도록 합니다.

```d
    shared(int)* p = new int;
    int* t = p; // 오류: shared는 shared 끼리만 처리되어야합니다.
```

`std.concurrency` 패키지의 `send` 함수는 `immutable` 또는 `shared` 로 표기된 값이나 객체를 복사하여 메시지로 전달합니다. `shared` 는 이행적(transitive) 성격을 가지고 있어, `class` 나 `struct` 가 `shared` 로 선언되면 그 밑의 내부 함수와 내부 변수 모두가 `shared` 상태가 됩니다.

다만 예외적으로 이전 섹션에서 배운 `static` 의 경우, *쓰레드 지역 변수(thread local storage, TLS)* 로 별도로 관리되기 때문에 해당하지 않습니다.

`synchronized` 로 동기화 블럭을 만들면 별도로 뮤텍스(mutex), 세마포어(semaphore) 등의 락(lock) 구현 없이, 오직 한 쓰레드만 실행되어야하는 임계 영역(critical section)을 손쉽게 지정할 수 있습니다. 이 메커니즘은 컴파일러에 의해 추가됩니다.

```d
    synchronized {
        importStuff();
    }
```

`class` 내부 함수에서는 어느 스코프(scope)의 값까지 `synchronized` 대상에 넣을지에 대한 논란이 있습니다. 이를 최소화하기 위해 내부 값/객체를 지정한 형태인 `synchronized(member1, member2)` 코드 형태로만 동기화 블럭을 만들 수 있습니다.

효율적이진 않지만, 필요하다면 클래스 전체를 `synchronized` 동기화 블럭으로도 만들 수 있습니다. 컴파일러는 이 클래스에 접근하려는 모든 경우에 대해 한번에 하나의 쓰레드만 접근하도록 통제된 코드를 만들 것입니다.

아토믹 연산(atomic operation)을 `shared` 로 선언된 변수에 적용하려면 `core.atomic.atomicOp` 이라는 보조 함수를 사용합니다. 아토믹 연산은, 원자(atom)가 더이상 쪼개지지 않는다는 점에서 나온 단어로, 다음 쓰레드가 접근하기 전에 현재 쓰레드에서 할 연산을 모두 수행함으로써 다중 쓰레드 환경에서 발생할 수 있는 값/상태 이상을 예방하는 기법입니다.

```d
    shared int test = 5;
    test.atomicOp!"+="(4);
```

### 더 살펴보기

- [Data Sharing Concurrency in _Programming in D_](http://ddili.org/ders/d.en/concurrency_shared.html)
- [`shared` type qualifier](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=11)
- [Lock-Based Synchronization with `synchronized`](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=13)
- [Deadlocks and `synchronized`](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=15)
- [`synchronized` specification](https://dlang.org/spec/statement.html#SynchronizedStatement)
- [Implicit conversions with `shared` data types](https://dlang.org/spec/const3.html#implicit_conversions)

## {SourceCode}

```d
import std.concurrency : receiveOnly, send,
    spawn, Tid, thisTid;
import core.atomic : atomicOp, atomicLoad;

/*
여러 쓰레드가 함께 이용해도 안전한 큐(Queue)를 만듭니다.
synchronized 키워드만으로 모든 접근은 한번에 하나씩만 접근 처리됩니다.
*/
synchronized class SafeQueue(T)
{
    // 주의: 클래스 내부 값들은 모두 private이 되어야만 합니다.
    // 클래스 외부로 노출된 변수의 동기화는 컴파일러가 잘 예측할 수 없습니다.
    // 만약 private이 아니라면 D 언어 컴파일러가 경고합니다.
    private T[] elements;

    void push(T value) {
        elements ~= value;
    }

    /// 큐가 비어있다면 T 타입의 기본 값을 반환합니다.
    T pop() {
        import std.array : empty;
        T value;
        if (elements.empty)
            return value;
        value = elements[0];
        elements = elements[1 .. $];
        return value;
    }
}

/*
여러 쓰레드에서 동시에 출력문을 호출해도 안전하게 출력되도록 만든
출력 함수입니다. 여기서 args는 가변 길이 입력값으로 제공되어있습니다.
전혀 입력값을 주지 않을 수도 있고,
원하는 만큼 다수의 입력값을 넣는 것도 가능합니다.
*/
void safePrint(T...)(T args)
{
    // 이 함수를 호출하는 동안 다른 쓰레드가 못쓰게 동기화합니다.
    synchronized {
        import std.stdio : writeln;
        writeln(args);
    }
}

void threadProducer(shared(SafeQueue!int) queue,
    shared(int)* queueCounter)
{
    import std.range : iota;
    // 1부터 11까지 숫자를 추가합니다
    foreach (i; iota(1,11)) {
        queue.push(i);
        safePrint("추가됨 ", i);
        atomicOp!"+="(*queueCounter, 1);
    }
}

void threadConsumer(Tid owner,
    shared(SafeQueue!int) queue,
    shared(int)* queueCounter)
{
    int popped = 0;
    while (popped != 10) {
        auto i = queue.pop();
        if (i == int.init)
            continue;
        ++popped;
        // atomicLoad를 사용해
        // shared로 선언된 queueCounter의 값을
        // 안전하게 읽어옵니다
        safePrint("추출됨 ", i,
            " (Consumer pushed ",
            atomicLoad(*queueCounter), ")");
    }

    // 끝났다고 다른 쓰레드에게 메시지를 보냅니다
    owner.send(true);
}

void main()
{
    auto queue = new shared(SafeQueue!int);
    shared int counter = 0;
    spawn(&threadProducer, queue, &counter);
    auto consumer = spawn(&threadConsumer,
                    thisTid, queue, &counter);
    auto stopped = receiveOnly!bool;
    assert(stopped);
}
```
