# 함수형 프로그래밍(Functional programming)

D 언어에서 강조할만한 특징 중 하나는 *함수형 프로그래밍(functional programming)* 지원이고, 1급 함수(first-class function)를 제공하여 함수형 프로그래밍을 할 수 있게 해줍니다.

여기서 1급 함수(first-class function)는 별도의 가공 과정 없이 함수 이름 자체를 함수의 입력값으로 쓸 수 있다는 걸 의미합니다. 자세한 내용은 위키백과의 [일급 객체](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B8%89_%EA%B0%9D%EC%B2%B4) 항목을 참고하시기 바랍니다.

함수형 프로그래밍에서 중요하게 여기는 특징은 **순수 함수(pure function)** 입니다. 여기서 순수 함수는 수학적인 의미의 함수를 의미하며, 풀어 쓰면 같은 입력에 대해 언제나 같은 값을 출력해야한다는 약속입니다.

D 언어에서 이런 순수 함수로 다루고 싶은 함수의 선언부에는 `pure` 라는 키워드를 추가해주면 됩니다. `pure` 로 표시된 함수는 프로그램 실행 중 바꿀 수 있는 요소를 접근하거나 변경할 수 없습니다. 따라서 `pure` 로 선언된 함수는 자신 안에서 `pure` 로 선언된 함수만 실행할 수 있습니다.

```d
    static auto globalNumber = 1;

    int add(int lhs, int rhs) pure {
        // pure로 표시된 함수는 절대 프로그램의 상태를 변경해선 안됩니다.
        // 아래 코드의 주석을 해제하게 되면 컴파일시 오류가 발생합니다.
        // globalNumber += 1;

        return lhs + rhs;
    }
```

위와 같은 유형의 `add` 함수를 **엄격한 순수 함수(strongly pure function)** 라고 부릅니다. 왜냐면 입력값의 변화에 따른 출력값의 변화만 발생하지, 이 함수를 호출한다고 입력값을 포함한 다른 값이 바뀌진 않았기 때문입니다. D 언어는 여기에 더불어 **느슨한 순수 함수(weakly pure functions)** 를 정의하는 것도 가능합니다. 느슨한 순수 함수에는 값을 바꿀 수 있는 입력값이 포함됩니다.

```d
    void add(ref int result, int lhs, int rhs) pure {
        // result 에 결과값을 저장합니다
        result = lhs + rhs;
    }
```

위의 함수도 제한된 범위에서만 값이 바뀌는 걸 허용하기에 순수 함수라고 간주하고 있습니다.

`pure` 키워드를 함수에 선언에 포함할 때 나타나는 여러 제약사항들이 불편할 수 있으나, 이 덕분에 한 변수의 값을 여러 쓰레드(thread)가 동시에 수정할 때 발생하는 데이터 레이스(data race)가 *별다른 장치 없이* 예방됩니다. 그리고 프로그램 전체의 상태를 바꾸는 일이 없기에, 컴파일러가 함수의 결과값을 캐시(cache)하는 등 다양한 각도에서 최적화를 시도하게 됩니다.

`pure` 키워드를 명시하지 않더라도 템플릿 함수, `auto` 를 반환값 타입으로 쓰는 함수 중에 해당하는 함수가 있다면 컴파일러가 알아서 `pure` 키워드를 자동으로 달아줍니다. 이는 `@safe`, `nothrow`, `@nogc` 키워드가 달린 함수에서도 마찬가지입니다.

### 더 살펴보기

- [Functional DLang Garden](https://garden.dlang.io/)

## {SourceCode}

```d
import std.bigint : BigInt;

/**
 * 임의의 양의 정수 x의 n승을 구합니다
 *
 * Returns:
 *     양의 정수 x의 n승을 반환합니다
 */
BigInt bigPow(uint base, uint power) pure
{
    BigInt result = 1;

    foreach (_; 0 .. power)
        result *= base;

    return result;
}

void main()
{
    import std.datetime : benchmark, to;
    import std.functional : memoize,
        reverseArgs;
    import std.stdio : writefln, writeln;

    // 메모이즈(memoize)는 제공된 함수의,
    // 입력값에 따른 결과값을 기억해두는 템플릿(template)입니다.
    // 순수 함수는 같은 입력값에 대해 항상 같은 값을 반환하기 때문에
    // 메모이제이션(memoization) 기법을 적용하기에 적절한 함수입니다.
    alias fastBigPow = memoize!(bigPow);

    void test()
    {
        writefln(".uintLength() = %s ",
               fastBigPow(5, 10000).uintLength);
    }

    foreach (i; 0 .. 10)
        benchmark!test(1)[0]
            .to!("msecs", double)
            .reverseArgs!writefln
                (" 실행에 %.2f 밀리초 소요");
}
```
