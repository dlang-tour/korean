# 연관 배열(Associative Arrays)

D 언어에는 다른 언어에서 해시맵(HashMap)이나 사전(Dictionary)로 알려진 *연관 배열* 을 제공합니다.

*연관 배열* 에서는 값을 찾기 위한 '키(key)' 와 그에 해당하는 '값(value)'이 일대일로 이어집니다. 예를 들어 `string` 을 키로 사용하고 이에 대응하는 값이 `int` 인 연관 배열을 아래와 같이 작성합니다.

```d
    int[string] arr;
```

그리고 값을 설정하거나 읽는 건 이렇게 하면 됩니다.

```d
    arr["key1"] = 10;
    writeln(arr["key1"]);
```

찾으려는 키가 연 관배열에 있는지 먼저 체크하는 게 중요합니다. 이걸 확인하기 위해서 `in` 이라는 연산자를 사용합니다.

```d
    if ("key1" in arr)
        writeln("있어요!");
```

`in` 연산자는 단순히 키가 있는지 없는지 확인해주는 것만 하는 게 아니라, 키에 대응하는 값의 주소를 담고 있는 포인터(참조)를 반환합니다. 그리고 만약 키가 없다면 `null` 포인터를 반환하여 값이 없다는 걸 알려줍니다. 위에서 확인한 `in` 의 용법을 응용하여, 키가 존재하는지 체크하고 키가 존재할 때 값을 바꾸는 조건부 작업을 간편히 수행할 수 있습니다.

```d
    // key1 이 존재한다면, auto 타입은 int형 값에 대한 레퍼런스로 자동 추론됩니다
    if (auto val = "key1" in arr)
        *val = 20;
```

만약 없는 키를 연관 배열에서 값을 찾기 위해 사용한다면 `RangeError` 의 원인이 되며, 이 에러로 인해 프로그램 동작이 종료됩니다. 따라서 연관 배열에 값이 없을 경우를 고려해 안전하게 값을 확인하려면 `get(키, 키가 없을 때 기본값)` 함수를 사용하는 게 좋습니다.

연관 배열은 일반 배열과 마찬가지로 `.length` 프로퍼티를 가지고 있어 몇 개의 키가 들어있는지 확인할 수 있습니다. 또한 `.remove(key)` 내부 함수를 통해 키와 키에 해당하는 값을 연관 배열에서 제거할 수 있습니다. `.byKey` 와 `.byValue` 라는 프로퍼티도 있는데, 직접 소개하기 보다는 이어질 실습을 통해 직접 확인하시기 바랍니다.

### 더 살펴보기

- [Associative arrays in _Programming in D_](http://ddili.org/ders/d.en/aa.html)
- [Associative arrays specification](https://dlang.org/spec/hash-map.html)
- [std.array.byPair](http://dlang.org/phobos/std_array.html#.byPair)

## {SourceCode}

```d
import std.array : assocArray;
import std.algorithm.iteration: each, group,
    splitter, sum;
import std.string: toLower;
import std.stdio : writefln, writeln;

void main()
{
    string text = "Rock D with D";

    // 각 단어가 문장에서 몇번 나타내는지 셉니다.
    int[string] words;
    text.toLower()
        .splitter(" ")
        .each!(w => words[w]++);

    foreach (key, value; words)
        writefln("key: %s, value: %d",
                       key, value);

    // `.keys` 와 .values` 는
    // 각각 키들의 배열, 값들의 배열을 반환합니다
    writeln("Words:", words.keys);

    // `.byKey`, `.byValue` 그리고 `.byKeyValue` 는
    // `.keys` 나 `.values` 와는 다르게
    // 최종적으로 세 함수에서 반환되는 값이
    // 필요해질 때만 동작하는 지연 평가(lazy evaluation) 기법을
    // 사용하고 있습니다. 호출의 결과물은 지연 평가로 만들어지는 배열입니다.
    writeln("# Words: ", words.byValue.sum);

    // `assocArray` 에 키와 값이 담긴 튜플(tuple)을
    // 전달해주면 연관 배열을 만들 수 있습니다.
    auto array = ['a', 'a', 'a', 'b', 'b',
                  'c', 'd', 'e', 'e'];

    // `.group` 은 배열에 담긴 값들이 몇번 반복되는지
    // 자동으로 계산하여 튜플(tuple)을 만듭니다.
    // 이 튜플로부터 연관 배열을 만들 수 있습니다.
    auto keyValue = array.group;
    writeln("Key/Value range: ", keyValue);
    writeln("Associative array: ",
             keyValue.assocArray);
}
```
