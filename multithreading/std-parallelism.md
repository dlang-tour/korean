# 표준 병렬 처리 라이브러리(std.parallelism)

`std.parallelism` 패키지에서는 데이터 병렬처리를 위한, 복잡한 조건이 필요없는 고수준의 기본 도구들을 제공하고 있습니다.

### parallel

[`std.parallelism.parallel`](http://dlang.org/phobos/std_parallelism.html#.parallel) 는 `foreach` 순회시 자동으로 각 작업을 새 쓰레드에 분배하여 처리량을 늘립니다.

```d
    // 병렬 실행으로 제곱수를 구합니다
    auto arr = iota(1,100).array;
    foreach(ref i; parallel(arr)) {
        i = i*i;
    }
```

`parallel` 함수는 내부적으로 `opApply` 연산자를 사용하여 동작합니다. 그리고 이 함수는 `taskPool.parallel` 의 단축 함수로, 전체 가용 CPU 중 하나만을 지정하여 쓰레드들을 돌리기 위한 `TaskPool(작업 풀)` 을 구성하도록 이미 설정되어있습니다.

따라서 병렬 처리 수준을 제어하기 위해서는 `parallel` 을 사용하기 보다는 직접 작업 풀을 구성하는 게 좋습니다.

주의할 점이 있다면, `parallel` 과 함께 동작하는 코드에서는, 현재 쓰레드가 값을 바꿀 것을 제외한 다른 것들을 건들지 않도록 해야합니다.

함수 호출시 `workingUnitSize` 입력값을 지정하면 작업 쓰레드 당 `foreach` 로부터 받은 값을 몇개나 처리할지 지정할 수 있습니다.

### reduce

[`std.algorithm.iteration.reduce`](http://dlang.org/phobos/std_algorithm_iteration.html#reduce) 는 함수형 프로그래밍 언어에서 *누적 연산(accumulate)* 또는 *foldl* 로 알려진 기능을 제공합니다. 이 함수는 `fun(acc, x)` 형태로 구성된 함수/람다 등을 입력으로 받아 값을 처리하되, `acc` 에는 항상 이전 처리에서 얻은 값을 전달됩니다.

```d
    // reduce 연산에는 항상 두 개의 입력값이 필요합니다.
    // 하나는 이전까지 계산된 값이며, 다른 하나는 앞으로 처리할 값입니다.
    // 초기에는 무엇이 오는지 모르기 때문에, 0을 초기값으로써 줍니다.
    auto sum = reduce!"a + b"(0, elements);
```

이 `reduce` 는 기본적으로 병렬처리를 지원하지 않습니다. 하지만  [`Taskpool.reduce`](http://dlang.org/phobos/std_parallelism.html#.TaskPool.reduce) 를 사용함으로써 병렬처리를 할 때 얻는 이득을 얻을 수 있습니다.

```d
    // 범위 탐색이 가능한 타입에 담긴 숫자의 합을 병렬처리로 구합니다.
    // taskPool.reduce 는 자동으로 첫번째 값을 초기값으로 가져옵니다.
    auto sum = taskPool.reduce!"a + b"(nums);
```

`TaskPool.reduce` 는 범위 탐색 타입의 자료를 다시 작은 범위로 나누어 병렬로 처리될 수 있도록 합니다.

각 작은 범위의 결과값들이 계산되면, 이 결과값들을 다시 전체의 결과로 환원(reduce)시켜 최종 결과를 얻습니다.

### `task()`

[`task`](http://dlang.org/phobos/std_parallelism.html#.task) 는 반드시 별도의 쓰레드에서 실행되어야할 작업을 처리하거나, 실행에 긴 시간이 필요한 작업을 처리하는데 사용됩니다. `task` 는 쓰레드와 마찬가지로 작업 풀에 넣어 관리할 수 있습니다.

```d
    auto t = task!read("foo.txt");
    taskPool.put(t);
```

혹은 작업 풀을 구성하지 않고 직접 시작시켜도 됩니다. 이때 별도의 쓰레드가 새로 구성됩니다.

```d
    t.executeInNewThread();
```

`task` 의 실행결과를 받으려면 `yieldForce` 를 호출하여 결과를 확인하면 됩니다.

단 `yieldForce` 는 결과가 나올때까지 호출한 쓰레드가 다른 작업을 못하도록 기다리게 함을 명심하시기 바랍니다.

```d
    auto fileData = t.yieldForce;
```

### 더 살펴보기

- [Parallelism in _Programming in D_](http://ddili.org/ders/d.en/parallelism.html)
- [std.parallelism](http://dlang.org/phobos/std_parallelism.html)

## {SourceCode}

```d
import std.parallelism : task,
    taskPool, TaskPool;
import std.array : array;
import std.stdio : writeln;
import std.range : iota;

string theTask()
{
    import core.thread : dur, Thread;
    Thread.sleep( dur!("seconds")(1) );
    return "Hello World";
}

void main()
{
    // 2개 쓰레드를 가진 작업 풀을 구성합니다
    auto myTaskPool = new TaskPool(2);
    // 함수나 프로그램이 종료되기 전, 작업 풀의 작업이 모두 종료되었는지
    // 확인하는 건 중요합니다.
    scope(exit) myTaskPool.stop();

    // 충분히 긴 작업을 시작해봅시다
    auto task = task!theTask;
    myTaskPool.put(task);

    auto arr = iota(1, 10).array;
    foreach(ref i; myTaskPool.parallel(arr)) {
        i = i*i;
    }

    writeln(arr);

    import std.algorithm.iteration : map;

    // 병렬처리를 사용해 제곱수의 합을 구합니다
    auto result = taskPool.reduce!"a+b"(
        0.0, iota(100).map!"a*a");
    writeln("제곱수의 합: ", result);

    // 별도의 task를 생성하여 처리했던 작업의 결과물이
    // 반환될때까지 기다립니다
    // 결과가 준비되면 화면에 출력합니다
    writeln(task.yieldForce);
}
```
