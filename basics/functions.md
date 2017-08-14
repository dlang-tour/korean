# 함수(Functions)

함수에 대해 다루기에 앞서, 이미 이전 강좌에서 한 함수를 보았습니다. 바로 모든 D 언어 프로그램의 실행 시작점인 `main()` 입니다.

함수는, 몇가지 값을 매개변수(argument)로 입력 받고 함수 내에서 일련의 작업을 수행한 다음 결과값을 반환합니다. `void` 를 결과값의 타입으로 지정한 경우, 아무것도 반환하지 않습니다. 

```d
    int add(int lhs, int rhs) {
        return lhs + rhs;
    }

    void welcome() {
        writeln("Welcome!");
        // 여기에서 아무 것도 반환하지 않고 함수가 끝났습니다.
    }
```

### 결과값 타입 자동 추론 ( `auto` return types )

함수의 결과값 타입을 `auto` 로 지정하면, D 언어 컴파일러가 적절한 타입이 무엇인지 예상해서 지정합니다. `return` 문이 여러개가 있고, 여러 타입이 반환되는 경우 그 모든 타입들을 아우를 수 있는 어떠한 타입으로 추론하게 됩니다.


```d
    auto add(int lhs, int rhs) { // int와 int의 덧셈이므로 명백히 `int` 타입이 추론됩니다.
        return lhs + rhs;
    }

    auto lessOrEqual(int lhs, int rhs) { // int와 double 중 더 큰 표현범위를 가진 `double` 로 추론합니다.
        if (lhs <= rhs)
            return 0;
        else
            return 1.0;
    }

### 기본값(Default arguments)

함수를 작성할 때 특정한 입력값에 대해 기본값을 미리 지정해놓고, 실행될 때 참조되도록 할 수도 있습니다.

```d
    void plot(string msg, string color = "blue") {
        ...
    }
    plot("D 언어 최고");
    plot("D 언어 최고", "blue");
```

함수 입력값에 기본값을 지정할 때 한 가지 주의할 사항이 있습니다. 한번 기본값을 지정한 뒤에는, 이후에 오는 모든 입력값에 대해서 기본값을 지정해야한다는 것입니다.

```d
    // 괜찮습니다
    void plot_good(string msg, string color = "blue", string linetype="solid") {
        ...
    }

    // 오류가 발생합니다
    void plot_error(string msg, string color = "blue", string linetype) {
        ...
    }

```

### 함수 내 함수(Local functions)

함수는 함수 내에서 선언될 수 있습니다. 이를 로컬 함수라고도 부릅니다. 이때 안에 선언된 함수는 외부에 노출되지 않아, 외부에서 직접 호출할 수 없습니다. 안에 선언된 함수는 바깥 함수에 선언된 변수들을 이용할 수 있습니다.

```d
    void fun() {
        int local = 10;
        int fun_secret() {
            local++; // 잘 동작하며, fun() 내에서 fun_secret()이 실행되면 바깥의 local이 증가합니다.
        }
        ...
```

이러한 내재된 함수들을 델리게이트(delegate)라고 부릅니다. [해당 강좌](basics/delegates) 에서 좀 더 자세히 다루겠습니다.

### 더 살펴보기

- [Functions in _Programming in D_](http://ddili.org/ders/d.en/functions.html)
- [Function parameters in _Programming in D_](http://ddili.org/ders/d.en/function_parameters.html)
- [Function specification](https://dlang.org/spec/function.html)

## {SourceCode}

```d
import std.stdio : writeln;
import std.random : uniform;

void randomCalculator()
{
    // 사칙연산을 위한 4개의 로컬 함수를 작성합니다
    auto add(int lhs, int rhs) {
        return lhs + rhs;
    }
    auto sub(int lhs, int rhs) {
        return lhs - rhs;
    }
    auto mul(int lhs, int rhs) {
        return lhs * rhs;
    }
    auto div(int lhs, int rhs) {
        return lhs / rhs;
    }

    int a = 10;
    int b = 5;

    // uniform(START, END) 함수는 START <= x < END 를 만족하는 정수 x 를 랜덤하게 생성합니다.
    // uniform이 반환하는 값에 따라 랜덤한 동작을 보여줍니다.
    switch (uniform(0, 4)) {
        case 0:
            writeln(add(a, b));
            break;
        case 1:
            writeln(sub(a, b));
            break;
        case 2:
            writeln(mul(a, b));
            break;
        case 3:
            writeln(div(a, b));
            break;
        default:
            // '도달할 수 없는 코드 위치'를 표시하는 약속입니다.
            // 기본 타입(Basic types) 강좌에서 살짝 언급되었습니다.
            assert(0);
    }
}

void main()
{
    randomCalculator();
    // add(), sub(), mul() and div() 는 randomCalculator 밖에서는 알 수 없습니다.
    // 이를 assert 구문으로 검증합니다.
    static assert(!__traits(compiles,
                            add(1, 2)));
}

```
