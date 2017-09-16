# 트레이트(Traits)

앞선 섹션들 중에서 **컴파일 시점 반환값 예측(compile-time function evaluation)** 이 D 언어의 특장점 중 하나로 소개를 했는데, 이걸 **내부 구조 확인(introspection)** 기능을 함께 사용해 제네릭(generic)을 사용한 코드로부터 더욱 강력한 결과물을 얻을 수 있습니다.

D 언어 컴파일러는 함수 호출 형태로부터 적용 가능한 트레이트를 찾고, 해당 트레이트에 맞춰 기계어로 컴파일 해줍니다.

## 명시적 약속(Explicit contracts)

트레이트를 함수 선언부에 함께 작성하여 어떤 특성을 가진 입력값을 받을지 명시할 수 있습니다. `splitIntoWords` 를 예로 들면, 이 함수는 어느 형식의 문자열 타입이든 문자열이라고 간주할 수 있는 타입 `S` 에 대해 모두 적용됩니다.

```d
S[] splitIntoWord(S)(S input)
if (isSomeString!S)
```

이런 기법은 템플릿 파라미터에도 적용되며, 아래 예제의 `myWrapper` 는 주어진 `f` 가 호출 가능한 특성을 가질 것을 트레이트로 명시하고 있습니다.

```d
void myWrapper(alias f)
if (isCallable!f)
```

좀 더 복합적인 예제를 살펴보기 위해 표준 라이브러리 `std.algorithm.searching` 에 포함된 [`commonPrefix`](https://dlang.org/phobos/std_algorithm_searching.html#.commonPrefix) 함수를 살펴보겠습니다. 이 함수는 *범위 탐색* 이 가능한 두 입력값에 대하여 공통되는 시작 부분을 찾습니다.

```d
auto commonPrefix(alias pred = "a == b", R1, R2)(R1 r1, R2 r2)
if (isForwardRange!R1
    isInputRange!R2 &&
    is(typeof(binaryFun!pred(r1.front, r2.front)))) &&
    !isNarrowString!R1)
```

이 함수는 선언될 때 여러 트레이트로, 해당 트레이트를 지원하는 입력값이 제공되길 기다립니다. 이 함수를 호출하기 위해 지켜줘야할 특성들을 다음과 같습니다.

- `r1` 은 *범위 탐색* 을 몇번 처음부터 끝까지 반복해도 항상 같은 값들을 탐색할 수 있어야합니다. (`isForwardRange` 트레이트에 의해 제약됨)
- `r2` 는 *순회 가능* 특성을 가지고 있어야 합니다. (`isInputRange` 트레이트에 의해 제약됨)
- `pred` 는 `r1` 과 `r2` 가 갖고 있는 원소를 입력값으로 받았을 때 동작해야합니다.
- `r1` 은 D 언어에서 보장하는 유니코드 문자열 타입이어야만 합니다. (`char[]`, `string`, `wchar` 또는 `wstring`) 만약 아니라면 디코드를 해서 유니코드로 바꿔줄 필요가 있습니다.

### 특화된 처리(Specialization)

API를 작성할 때 다용도로 동작할 수 있게 만들길 원하지만, 이걸 위해 프로그램 실행 시간에 타입을 검증하는 것처럼 추가적 실행과정이 더 들어가는 건 원하지 않을 것입니다.

**내부 구조 확인(introspection)** 기법과 CTFE를 이용하여 컴파일 시점에 특화된 함수들을 미리 만들어 놓고 곧바로 실행하여 최적의 성능을 얻을 수 있습니다.

이러한 특화된 처리가 적용되면 좋은 경우로는 리스트(list)나 스트림(stream)처럼 길이를 확인하려면 전체 개수를 세어봐야하는 타입과, 선언한 시점부터 길이를 알 수 있는 배열(array) 타입 모두에서 길이를 구해야하는 함수를 작성하는 경우를 들 수 있습니다.

*순회 가능* 한 타입이라면 `std.range` 의 `walkLength` 와 배열이 가지고 있는 `.length` 프로퍼티 두 가지만으로 모든 입력값의 경우를 일반화할 수 있습니다.

```d
static if (hasMember!(r, "length"))
    return r.length; // r에 length 프로퍼티가 있다면 바로 구합니다. O(1)의 복잡도를 나타냅니다
else
    return r.walkLength; // r에 해당 프로퍼티가 없다면 하나씩 세어 몇개인지 확인합니다. O(n)의 복잡도를 갖습니다
```

#### `commonPrefix`

Phobos 표준 라이브러리 내에 컴파일 시점의 내부 구조 탐색을 적용한 부분은 곳곳에 숨어있습니다. `commonPrefix` 는 `RandomAccessRange` 타입이 들어올 때와 일반적인 *범위 탐색* 타입이 들어올 때 동작이 다릅니다. `RandomAccessRange` 쪽이 좀 더 빠르게 동작하는데, 왜냐면 순차적으로 가지 않고 임의로 다른 곳으로 이동하여 탐색할 수 있어 알고리즘이 더 빠르게 동작할 수 있게 해주기 때문입니다.

#### CTFE를 좀 더 체험하기(More CTFE magic)

[std.traits](https://dlang.org/phobos/std_traits.html) 에는 D 언어에서 쓰일 수 있는 대부분의 [트레이트](https://dlang.org/spec/traits.html) 가 들어있습니다. 단 `compiles` 처럼, 해당 코드가 컴파일 되는지 확인해주는 트레이트들은 제외됩니다.

```d
__traits(compiles, obvious error - $%42); // false
```

#### 특별한 키워드(Special keywords)

편리한 디버깅을 위해 D 언어는 몇가지 특별한 키워드를 제공하고 있습니다.

```d
void test(string file = __FILE__, size_t line = __LINE__, string mod = __MODULE__,
          string func = __FUNCTION__, string pretty = __PRETTY_FUNCTION__)
{
    writefln("file: '%s', line: '%s', module: '%s',\nfunction: '%s', pretty function: '%s'",
             file, line, mod, func, pretty);
}
```

이 특별한 키워드 중 하나를 사용하면, 시스템 시간을 확인하기 위해 `time` 명령어를 셸에 입력하지 않아도 됩니다.

```d
rdmd --force --eval='pragma(msg, __TIMESTAMP__);'
```

## 더 살펴보기

- [std.range.primitives](https://dlang.org/phobos/std_range_primitives.html)
- [std.traits](https://dlang.org/phobos/std_traits.html)
- [std.meta](https://dlang.org/phobos/std_meta.html)
- [Specification on Traits in D](https://dlang.org/spec/traits.html)

## {SourceCode}

```d
import std.functional : binaryFun;
import std.range.primitives : empty, front,
    popFront,
    isInputRange,
    isForwardRange,
    isRandomAccessRange,
    hasSlicing,
    hasLength;
import std.stdio : writeln;
import std.traits : isNarrowString;

/**
자동 인코딩 변환없이, 공통된 앞부분을 찾습니다

Params:
    pred = 같은지 판별해줄 판별 함수
    r1 = 예측형 범위 탐색을 지원하는 타입
    r2 = 범위 탐색을 지원하는 타입

Returns:
공통된 앞부분을 r1 으로부터 슬라이스하여 반환
 */
auto commonPrefix(alias pred = "a == b", R1, R2)
                 (R1 r1, R2 r2)
if (isForwardRange!R1 && isInputRange!R2 &&
    !isNarrowString!R1 &&
    is(typeof(binaryFun!pred(r1.front,
                             r2.front))))
{
    import std.algorithm.comparison : min;
    static if (isRandomAccessRange!R1 &&
               isRandomAccessRange!R2 &&
               hasLength!R1 && hasLength!R2 &&
               hasSlicing!R1)
    {
        immutable limit = min(r1.length,
                              r2.length);
        foreach (i; 0 .. limit)
        {
            if (!binaryFun!pred(r1[i], r2[i]))
            {
                return r1[0 .. i];
            }
        }
        return r1[0 .. limit];
    }
    else
    {
        import std.range : takeExactly;
        auto result = r1.save;
        size_t i = 0;
        for (;
             !r1.empty && !r2.empty &&
             binaryFun!pred(r1.front, r2.front);
             ++i, r1.popFront(), r2.popFront())
        {}
        return takeExactly(result, i);
    }
}

void main()
{
    // "hello, " 가 출력됩니다
    writeln(commonPrefix("hello, world"d,
                         "hello, there"d));
}
```
