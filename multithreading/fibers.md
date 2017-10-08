# 파이버(Fibers)

파이버는 작업 단위 간의 *상호협동성* 을 극대화한 병행처리 기법이자 각 작업 단위를 가르키는 단어입니다.

D 언어에 구현된 이 기법은 [`core.thread`](https://dlang.org/phobos/core_thread.html) 모듈 안에 정의되어있습니다.

파이버가 *상호협동성* 을 가진 병행처리 기법이 된 배경에는, 프로그램에 제공될 입력을 무한정 기다릴 이유가 없다는 사실이 있습니다.

만약 어떤 입력을 기다려야한다면, 기다리는 동안에 현재 작업을 일시정지하고 `Fiber.yield()` 를 실행하여 입력이 준비될 동안 다른 작업을 수행하는 게 효율적일 것입니다.

`Fiber.yield()` 를 통해 입력값을 기다리던 상위 작업으로 돌아감과 동시에, 현재까지 실행 중이던 컨텍스트(함수 호출 스택에 저장되어있음)를 백업합니다. 그리고 상위 작업이 다시 `Fiber.yield()` 를 호출하여 프로그램 입력을 기다리던 작업에게 실행 순서를 반납해주면 일시정지된 작업이 다시 동작합니다.

```d
    void foo() {
        writeln("Hello");
        Fiber.yield();
        writeln("World");
    }
    // ...
    auto f = new Fiber(&foo);
    f.call(); // Hello 출력하고, 이 Fiber는 일시정지됨
    f.call(); // 다음 호출에서 재개되어 World 출력
```

파이버의 이런 실행 특성을 활용해 하나의 CPU 코어를 여러 파이버가 효율적으로 나눠쓸 수 있게 만들 수 있습니다.

쓰레드를 여러개 사용할 때와 파이버를 여러개 사용할 때를 비교해보면, 파이버는 컨텍스트 스위칭(context switching)이 필요하지 않아 시스템 자원 소모를 아낄 수 있는 장점이 있습니다. 반대로 쓰레드는 서로 다른 쓰레드가 동작할 때 지속적인 컨텍스트 스위칭이 발생합니다.

D 언어 라이브러리 중 파이버를 잘 활용하는 예로는 [vibe.d 프레임워크](http://vibed.org) 라는 비동기 I/O를 이용해 깔끔한 코드를 유지하면서도 높은 성능을 보여주는 웹 프레임워크가 있습니다.

### 더 살펴보기

- [Fibers in _Programming in D_](http://ddili.org/ders/d.en/fibers.html)
- [Documentation of core.thread.Fiber](https://dlang.org/library/core/thread/fiber.html)

## {SourceCode}

```d
import core.thread : Fiber;
import std.stdio : write;
import std.range : iota;

/**
범위 탐색이 가능한 타입을 입력으로 받고,
범위 내 값에 대해 `Fnc` 라는 함수를 각각 실행한 결과를
`result` 로 반환합니다.

매 결과값마다 이 파이버는 결과값을
squareResult 또는 cubeResult에 저장한 후
현재 작업을 일시 중단합니다.
*/
void fiberedRange(alias Fnc, R, T)(
    R range,
    ref T result)
{
    for(; !range.empty; range.popFront) {
        result = Fnc(range.front);
        Fiber.yield();
    }
}

void main()
{
    int squareResult, cubeResult;
    // 제곱수를 계산하는 델리게이트를 포함한
    // 파이버를 만듭니다.
    auto squareFiber = new Fiber({
        fiberedRange!(x => x*x)(
            iota(1,11), squareResult);
    });
    // 여기는 세제곱수를 구해봅시다.
    auto cubeFiber = new Fiber({
        fiberedRange!(x => x*x*x)(
            iota(1,9), cubeResult);
    });

    // 파이버에는 상태값이 있습니다.
    // 만약 TERM 이 현재 상태라면, 이 파이버는
    // 더이상 작업을 처리할 수 없습니다.
    squareFiber.call();
    cubeFiber.call();
    while (squareFiber.state
        != Fiber.State.TERM && cubeFiber.state
        != Fiber.State.TERM)
    {
        write(squareResult, " ", cubeResult);
        squareFiber.call();
        cubeFiber.call();
        write("\n");
    }

    // squareFiber는 cubeFiber보다 더 많은 범위를
    // 계산합니다.
    // 따라서 cubeFiber가 종료된 후에도 동작할 수 있습니다.
}
```
