# 델리게이트(Delegates)

델리게이트는 '대행'의 의미를 담고 있으며, 함수 등이 자신 내에서 직접 처리하는 게 아니라 대신 해줄 함수나 클래스에게 입력값에 대한 처리를 맡기는 패턴입니다. 이런 역할을 수행하는 값/객체를 델리게이터(delegator)라고 부르고, `대행자` 로 번역하기도 합니다.

### 함수를 델리게이트로 쓰기(Functions as arguments)

실행하고자 하는 함수를, 다른 함수의 입력값으로 함께 전달할 수 있습니다.


```d
    void doSomething(int function(int, int) doer) {
        // doer 라는 이름으로 전달된 함수를 실행합니다.
        // doer 라는 이름으로 전달될 함수는 int 타입의 값을 결과로 반환하고,
        // 실행을 위해 int 타입의 값 두 개를 입력받는 함수여야 합니다.
        doer(5,5);
    }

    doSomething(add); // `add` 라는 함수를 전달합니다.
                      // doSomething은 add 를 실행합니다.
```

`doSomething` 내에 전달된 `doer` 는 `doSomething` 내에서 여느 함수처럼 자유롭게 사용할 수 있습니다.

### 함수 속의 함수(Local functions with context)

이전의 `doSomething` 예제에서는 전역 함수(소스코드 전체에서 사용할 수 있는 함수)를 가르키는 참조 타입인 `function` 을 사용하는 예를 보았습니다. 그러나 모든 함수를 이렇게 입력값으로 전달할 수는 없습니다. 클래스의 내부 함수나 함수 속에 선언된 함수를 참조할 때는 `delegate` 구조를 반드시 사용해야 합니다. `delegate` 는 `function` 과 비슷하게 함수를 가르키는 참조 타입이지만, 어떤 상태와 환경에서 실행 중인지 알려주는 문맥 정보를 담고 있는 컨텍스트(context)를 함께 담고 있다는 차이점이 있습니다.

이렇게 컨텍스트가 함께 담긴 상태를 *엔클로저(Enclosure; 컨텍스트 보존형 함수)* 라고 부릅니다. 자바스크립트와 같은 타 언어에서는 *클로저(Closure)* 라고 부르기도 합니다. 만약 클래스로부터 생성된 객체(object)의 내부 함수를 `delegate` 로 전달하게 되면 객체의 내부의 상태도 함께 전달됩니다. 함수 속에 함수가 선언된 로컬 함수(Local functions)의 경우에는 로컬 함수가 바깥 함수의 컨텍스트를 같이 갖고 가게 됩니다.

`delegate` 의 컨텍스트 보존 기능으로 인해 원본 컨텍스트가 함께 전달되어, 델리게이트가 원본을 변형시킬 우려도 있습니다. 그러나 D 언어 컴파일러는 이런 상황이 발생할 것 같으면 미리 컨텍스트를 힙(heap)에 하나 더 복사해놓고, 그 컨텍스트 내에서 델리게이트가 동작하도록 합니다. 다만 항상 그런 것은 아닙니다.

```d
    void foo() {
        // foo 내에 local 이라는 로컬 함수를 선언했습니다.
        // local은 foo 밖에선 존재하는지 알 수 없는 존재입니다.
        // local의 컨텍스트는 foo와 공유되기 때문에, foo에 미리 선언해둔 변수를
        // local 이 사용할 수 있습니다.
        void local() {
            writeln("local");
        }
        auto f = &local; // 이때 f의 타입은 delegate() 로 추론됩니다
    }
```

`doSomething` 을 `delegate` 를 사용할 수 있게 바꾸려면 아래와 같이 수정하면 됩니다.

```d
    void doSomething(int delegate(int,int) doer);
```


`delegate` 와 `function` 은 비슷한 역할을 하지만, 둘은 동시에 사용될 수 없습니다. 즉 `function` 이면서 `delegate` 일 수는 없다는 것입니다. 대신 필요하다면, `function` 을 `delegate` 로 변환해주는 표준 라이브러리 함수인  [`std.functional.toDelegate`](https://dlang.org/phobos/std_functional.html#.toDelegate) 를 쓰는 걸 고려해볼 수 있습니다.

### 람다, 이름 없는 익명 함수(Anonymous functions & Lambdas)

함수가 다른 함수의 입력값으로 전달되는 게 꽤 흔한 일이라, 매번 그런 함수에 이름을 붙이는 건 번거로운 일이 될 것입니다. D 언어는 그래서 아예 이름 없는 함수를 지원하면서, 더불어 한 줄로 선언되는 함수인 _람다(lambda)_ 를 만들 수 있도록 했습니다.

```d
    auto f = (int lhs, int rhs) {
        return lhs + rhs;
    };
    auto f = (int lhs, int rhs) => lhs + rhs; // 이것이 람다입니다. 위의 번거로운 익명 함수 선언을 간단하게 만들었습니다.
```

이것 마저도 귀찮다면 단순히 문자열로 작성된 식을 전달해 `folding` 연산을 수행하는 아래의 예제를 참고하십시오. 단순히 문자열을 템플릿 함수의 입력값처럼 전달했는데, 마치 a와 b가 입력값으로 선언되고 함수처럼 `+` 가 동작하는 모습을 볼 수 있습니다.

```d
    // reduce의 연산 순서는 아래와 같습니다.
    // 1    2    3
    // |____|    |
    //    +      |
    //    |______|
    //        +
    //        = 6
    [1, 2, 3].reduce!`a + b`; // 6
```

모든 경우에 이렇게 처리되는 것은 아닙니다. 익명 함수나 람다로 바꾸었을 때, _입력 값이 하나 혹은 둘 일때만 가능_ 합니다. 그리고 첫번째 입력값을 `a`, 두번째 입력값을 `b` 라고 부르는 것 또한 D 언어와 프로그래머 사이의 약속입니다. 지켜주셔야합니다.

### 더 살펴보기

- [Delegate specification](https://dlang.org/spec/function.html#closures)

## {SourceCode}

```d
import std.stdio : writeln;

enum IntOps {
    add = 0,
    sub = 1,
    mul = 2,
    div = 3
}

/**
사칙연산 기능을 구현합시다
입력값:
    op = 사칙연산자 중 하나를 고릅니다
반환값:
    숫자만 넣으면 op 으로 지정해둔 연산을 대신 수행해주는 델리게이터를 반환합니다.
*/
auto getMathOperation(IntOps op)
{
    // 사칙연산 각가에 대해 한 줄 함수인 람다를 선언해둡시다
    auto add = (int lhs, int rhs) => lhs + rhs;
    auto sub = (int lhs, int rhs) => lhs - rhs;
    auto mul = (int lhs, int rhs) => lhs * rhs;
    auto div = (int lhs, int rhs) => lhs / rhs;

    // 각 경우에 맞춰 알맞은 람다를 반환합시다
    final switch (op) {
        case IntOps.add:
            return add;
        case IntOps.sub:
            return sub;
        case IntOps.mul:
            return mul;
        case IntOps.div:
            return div;
    }
}

void main()
{
    int a = 10;
    int b = 5;

    auto func = getMathOperation(IntOps.add);
    writeln("func 의 타입은  ",
        typeof(func).stringof, "이네요!");

    // 이제 델리게이터를 통해 원하는 최종 결과물을 얻습니다
    writeln("result: ", func(a, b));
}
```
