# 범위 탐색(Ranges)

`foreach` 를 이전 섹션에서 간편하게 이용할 수 있다는 걸 확인했습니다. 이런 형태였습니다.

```d
foreach (element; range)
{
    // Loop body...
}
```

그러나 실제로 `foreach` 는 컴파일 과정에서 다음과 같은 형태로 바뀌게 됩니다. 다시 말하면, `범위 탐색` 기능을 제공하려면 필요한 몇가지 프로퍼티가 존재한다는 것입니다. 특히, `popFront()` 함수의 동작 때문에 `__rangeCopy`에 올 타입에 주의해야합니다.

```d
for (auto __rangeCopy = range;
     !__rangeCopy.empty;
     __rangeCopy.popFront())
 {
    auto element = __rangeCopy.front;
    // Loop body...
}
```

`popFront()` 함수로 인해 `class` 와 같은 참조 타입을 `범위 탐색` 의 대상으로 사용하게 되면 첫번째 탐색은 잘 되지만 두번째 탐색과 그 이후의 모든 탐색은 실패합니다. 왜냐면 참조 타입의 원본 값이 변형되기 때문입니다. `popFront` 가 동작함에 따라 갖고 있는 값이 하나씩 사라지고 나중엔 텅 빈 상태가 되는 것입니다. (`break` 로 도중에 중단하게 되면 멈춘 지점부터는 남아있습니다) `struct` 와 같은 비참조 타입은 변수에 할당할 때 '복사(copy)' 가 일어나게 되는데 이 덕분에 원본은 무사하고, 다음 `범위 탐색` 에서도 이전과 똑같이 동작하게 됩니다. 참조 타입에서 일아났던 특성을 `destructive` 라고 부르고, 비참조 타입에서 일어났던 특성을 `non-destructive` 라고 부르니 참고하십시오.

대부분 `Phobos` 표준 라이브러리 내의 함수들은 `foreach` 를 안전하게 실행할 수 있도록 되어있습니다만은, 100% 보장된 것은 아닙니다. 만약 절대적으로 원본이 유지되어야한다면 **forward range(예지형 범위 탐색)** 기법을 제공하는 알고리즘을 선택해야합니다.

그렇다면 어떤 타입이 범위 탐색 기능을 제공할 수 있는지 살펴보겠습니다. 아래와 같은 **범위 인터페이스** 를 제공하기만 한다면 그 어떤 타입이든 이용할 수 있습니다.


```d
    struct Range
    {
        T front() const @property;
        bool empty() const @property;
        void popFront();
    }
 ```

 `empty` 와 `front` 는 `const` 함수로 정의되어 있기에 값 내부를 바꾸지 않는다고 약속하고 있습니다. 반드시 `const` 로 선언해야하는 것은 아닙니다.

표준 라이브러리의 [`std.range`](http://dlang.org/phobos/std_range.html) 와
[`std.algorithm`](http://dlang.org/phobos/std_algorithm.html) 에서 제공하는 함수들을 통해 `범위 탐색` 을 쉽게 이용할 수 있습니다. 또한 구현하는 입장에서, 범위 탐색  중 값 하나 하나를 제공하기 위한 복잡한 과정들을 단순한 인터페이스로 감추는 것과 동시에, 제공될 값들을 한번에 모두 생성하는 게 아니라 하나의 값이 필요할 때 한번의 연산만 수행할 수 있는 **지연(lazy) 생성** 기법을 적용하기 쉽습니다. 쉽게 설명하면, `popFront()` 가 이루어질 때마다 하나씩 만들어주면 된다는 것입니다.

기본 알고리즘 외에 특별한 알고리즘들은 [D 언어 심화 강좌](gems/range-algorithms) 에서 살펴볼 수 있습니다.

### 실습

아래 소스코드를 완성하여 `FibonacciRange` 범위 탐색을 구현하고, [피보나치 수열](https://en.wikipedia.org/wiki/Fibonacci_number) 이 순서대로 생성되어 하나씩 값이 나올 수 있도록 만들어 봅시다.

`assert` 는 올바르게 코드가 작성되었는지 검증하기 위해 미리 작성되어있습니다. `assert` 가 거슬린다고 다 지워버리는 어리석은 행동을 자제해주십시오.

### 더 살펴보기

- [`std.algorithm`](http://dlang.org/phobos/std_algorithm.html)
- [`std.range`](http://dlang.org/phobos/std_range.html)

## {SourceCode:incomplete}

```d
import std.stdio : writeln;

struct FibonacciRange
{
    bool empty() const @property
    {
        // 코드를 작성해주세요
        // 고려 사항1: 피보나치 수열이 언제 끝나야할까요?
        // 고려 사항2: D 언어의 정수 타입에는 한계가 있지 않을까요?
    }

    void popFront()
    {
        // 코드를 작성해주세요
    }

    int front() const @property
    {
        // 코드를 작성해주세요
    }
}

void main()
{
    import std.range : take;
    import std.array : array;

    FibonacciRange fib;

    // `take` 는 최대 N 개까지 값을 범위 탐색에서
    // 추출해냅니다. 값이 필요할 때마다 `popFront` 가 호출됩니다.
    auto fib10 = take(fib, 10);

    // 때론 범위 탐색으로 다루는 것보다
    // 배열로 다루는 게 훨씬 편할지도 모릅니다.
    // 아래와 같이 변환할 수 있습니다.
    int[] the10Fibs = array(fib10);

    writeln("첫 10개의 피보나치 수 : ",
        the10Fibs);
    assert(the10Fibs ==
        [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]);
}
```
