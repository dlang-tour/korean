# 범위 탐색 알고리즘 모음(Range algorithms)

표준 라이브러리 `Phobos` 에서 제공하는 [std.range](http://dlang.org/phobos/std_range.html) 와 [std.algorithm](http://dlang.org/phobos/std_algorithm.html) 에는 코드 가독성은 높이면서도 *범위(Range)* 로 구성된 값들을 효율적으로 다룰 수 있게 해주는 함수들이 있습니다.

범위 탐색에 필요한 인터페이스를 구현해놓기만 한다면, 프로그래머가 직접 만든 타입으로도 표준 라이브러리에서 제공하는 모든 알고리즘을 이용할 수 있기 때문에 꽤나 편리하다고 할 수 있겠습니다.

### std.algorithm

`filter` - 람다(lambda) 식을 템플릿 값으로 전달하여 범위 탐색으로 값을 걸러냅니다. 값이 두 개 이하일 때는 첫번째 값을 `a` 로, 두번째 값을 `b` 로 놓은 문자열을 람다 함수 대신 전달할 수 있다는 걸 기본 강좌의 델리게이트(delegate) 섹션에서 설명했습니다.

```d
    filter!"a > 20"(range);
    filter!(a => a > 20)(range);
```

`map` - 템플릿 값으로 전달된 람다함수를 *범위 탐색* 의 모든 값에 적용 후, 그 결과값을 반환합니다

```d
    [1, 2, 3].map!(x => to!string(x));
```

`each` - `foreach` 를 단순하게 이용할 수 있습니다.

```d
    [1, 2, 3].each!(a => writeln(a));
```

### std.range
`take` - 범위 탐색을 하여 최초 *N* 개의 자료만 가져옵니다.

```d
    theBigBigRange.take(10);
```

`zip` - 입력값으로 범위 탐색이 가능한 값 두 개를 받아, 각각의 순서에 맞춰 두 값을 하나의 튜플(tuple)로 묶어줍니다.

```d
    assert(zip([1,2], ["hello","world"]).front
      == tuple(1, "hello"));
```

`generate` - 입력값으로 값을 반환할 수 있는 함수를 받아, 범위 탐색시 매번 그 함수를 호출해 값을 생성하는 함수를 만듭니다.

```d
    alias RandomRange = generate!(x => uniform(1, 1000));
```

`cycle` - 입력된 범위 탐색 타입을 계속 돌아가며 탐색할 수 있게 만들어줍니다.

    auto c = cycle([1]);
    // 보통의 범위 탐색 타입은 empty 상태가 되어 끝나지만,
    // cycle을 통과하면 그렇게 될 수 없습니다. 끝나지 않습니다.
    assert(!c.empty);

### 표준 라이브러리 문서를 더 살펴보시는 걸 권합니다


### 더 살펴보기

- [Ranges in _Programming in D_](http://ddili.org/ders/d.en/ranges.html)
- [More Ranges in _Programming in D_](http://ddili.org/ders/d.en/ranges_more.html)

## {SourceCode}

```d
// 쓸 수 있는 알고리즘은 다 가져와봅시다
import std.algorithm : canFind, map,
  filter, sort, uniq, joiner, chunkBy, splitter;
import std.array : array, empty;
import std.range : zip;
import std.stdio : writeln;
import std.string : format;

void main()
{
    string text = q{This tour will give you an
overview of this powerful and expressive systems
programming language which compiles directly
to efficient, *native* machine code.};

    // 위의 문장을 단어별로 나눠줄 구분자를 찾는 람다를 작성합니다
    alias pred = c => canFind(" ,.\n", c);
    // 추가적으로 메모리 공간을 할당받지 않고도
    // 필요할 때마다 한 단어씩 게으르게(lazily) 나눕니다!
    auto words = text.splitter!pred
      .filter!(a => !a.empty);

    auto wordCharCounts = words
      .map!"a.count";

    // 각 단어의 글자수를 세어, 길이가 짧은 것부터 모아서 정리합니다
    // 단어 길이별로 나오는 단어를 정리해 보여주는 코드입니다
    // 이 문법이 잘 이해되지 않는다면,
    // 연쇄 함수 호출 문법(Uniform Function Call Syntax (UFCS)) 섹션을 다시 살펴보십시오
    zip(wordCharCounts, words)
      // 정렬 알고리즘을 적용하기 위해 배열로 변환합니다
      .array()
      .sort()
      // 중복된 원소를 걸러냅니다
      // 길이별로 나오는 단어가 무엇인지만 확인하면 되기 때문입니다
      .uniq()
      // 같은 글자수를 가진 단어를 한 곳에 몰아넣습니다.
      // chunkBy 함수는 범위 탐색 타입을 주어진 조건에 맞는 것끼리 묶어주는 역할을 합니다
      // 이 경우에는 a[0], 즉 wordCharCounts를 기준으로 묶어주게 됩니다
      .chunkBy!(a => a[0])
      // 결과물이 한 행의 문자열로 합쳐지게 합니다
      .map!(chunk => format("%d -> %s",
          chunk[0],
          chunk[1]
            .map!(a => a[1])
            .joiner(", ")))
      // joiner 를 사용해 각 행 사이사이에 \n 을 삽입합니다.
      // 이로써 결과물이 한 줄에 하나씩 표시될 준비를 마쳤습니다
      .joiner("\n")
      // 마지막으로 결과물을 출력합니다
      .writeln();
}
```
