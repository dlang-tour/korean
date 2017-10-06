# 타입 별칭과 문자열 (Alias & Strings)

지난 강좌들을 통하여 배열(arrays)과 슬라이스(slices)가 무엇인지 확인하고, `immutable` 속성을 기본 타입(basic types) 섹션에서 짧게 체험해보았습니다. 이제 배운 것들을 모두 섞은 새로운 무언가를 소개하겠습니다.

```d
    alias string = immutable(char)[];
```

`alias` 라는 선언문 뒤에 `string` 이라는 단어가 있습니다. 그렇다면 등호(=) 뒤의 내용이 `string` 으로 별칭이 지정된다는 의미로 볼 수 있겠습니다. 이때 등호 뒤의 내용을 보면 `immutable(char)` 의 슬라이스 타입으로 적혀있습니다. 이로써 이전에 무심코 사용하던 `string` 타입에 대해 알게 된 것입니다. `string` 타입의 변수는 한번 생성되면 `immutable` 이기 때문에 바뀌지 않습니다. 그리고 `char` 타입은 올바른 UTF-8로 인코딩된 문자 하나를 담습니다. 그렇습니다! UTF-8 `string` 을 사용하게 된 걸 다시 한번 환영합니다.

`string` 이 이렇게 불변(immutablility) 특성을 갖춘 덕분에, 여러 쓰레드(thread)에서 동시에 이용해도 전혀 문제가 없습니다. 그리고 `string` 은 슬라이스의 특성을 가지기 때문에 별도 메모리 공간을 받지 않고도 부분부분을 쉽게 참조할 수 있습니다. 가장 좋은 예로, 표준 라이브러리의 [`std.algorithm.splitter`](https://dlang.org/phobos/std_algorithm_iteration.html#.splitter) 함수는 주어진 문자열을 별도 메모리 할당 없이 줄단위로 나눠줍니다.

UTF-8 외에도 다른 인코딩을 쓰는 문자열을 쓸 수도 있습니다. 두 종류가 더 있습니다.

```d
    alias wstring = immutable(wchar)[]; // UTF-16
    alias dstring = immutable(dchar)[]; // UTF-32
```

`std.conv` 표준 라이브러리의 `to` 함수를 이용해 다른 인코딩을 쓰는 문자열로 쉽게 변환이 가능합니다.

```d
    dstring myDstring = to!dstring(myString);
    string myString   = to!string(myDstring);
```

### 유니코드(Unicode strings)

위의 `alias string = immutable(char)[];` 을 다시 상기해보면, `string` 은 8비트 유니코드 [코드 유닛](http://unicode.org/glossary/#code_unit) 들의 배열로 정의됩니다. 배열에 대한 모든 연산 특성은 문자열에도 적용될 수 있지만, 이때 연산은 각 글자 개별에 적용되는 게 아니라 유니코드 코드 유닛 수준을 적용하여 이루어집니다. D 언어 표준 라이브러리에 포함된 문자열 관련 알고리즘들은 `string`을 [코드 포인트](http://unicode.org/glossary/#code_point) 가 쭉 이어진 자료로 해석하지만, 필요하다면 [`std.uni.byGrapheme`](https://dlang.org/library/std/uni/by_grapheme.html) 같은 걸 이용해 [그래핌](http://unicode.org/glossary/#grapheme) 꼴로 해석하는 선택지도 존재합니다.

위 두 가지 해석 방법에 대한 간단한 예를 들어보겠습니다.


```d

    string s = "\u0041\u0308"; // Ä 라는 문자입니다

    writeln(s.length); // 3개 코드 유닛이 포함되어있습니다.

    import std.range : walkLength;
    writeln(s.walkLength); // 코드 포인트 수는 2입니다. (\uXXXX 의 개수)

    import std.uni : byGrapheme;
    writeln(s.byGrapheme.walkLength); // 그래핌으로 길이를 계산할 경우 1입니다. (화면에 표현될 때 한 글자)
```

`s` 의 바이트 배열 길이는 3입니다. 왜냐면 총 3개의 코드 포인트 `0x41`, `0x03`, `0x08` 을 담고 있기 때문입니다. 다음 예시에서 나타난 `2` 는 싱글 코드 포인트(combining diacritics character)가 무엇인지 알려주면서, [`walkLength`](https://dlang.org/library/std/range/primitives/walk_length.html)
(임의의 길이를 계산하기 위한 표준 라이브러리 함수) 가 싱글 코드 포인트를 세는 예제를 보여줍니다. 마지막으로 `byGrapheme` 함수 내부의 복잡한 과정을 거쳐 두 코드 포인트가 한 글자를 표현하고 있음을 인식한 끝에 한 글자 표시되는 걸 확인한 모습을 볼 수 있습니다.

여기까지의 설명도 처음 듣는 사람에겐 쉽지 않은 내용입니다. 유니코드 문자를 올바르게 처리하는 건 정말 어렵지만, D 언어를 설계하고 만든 사람들은 `string` 타입을 이용하는 것만으로도 유니코드 지원과 표준 라이브러리 이용을 간편하게 할 수 있도록 했습니다. `byGrapheme` 과 더불어, 코드 유닛들을 다뤄보고 싶다면 [`byCodeUnit`](http://dlang.org/phobos/std_utf.html#.byCodeUnit) 문서를 참고하시기 바랍니다.

자동 디코드라는 것도 있습니다. [Unicode gems chapter](gems/unicode) 를 참고하십시오.

### 여러 줄을 담은 문자열(Multi-line strings)

D 언어에서는 여러 줄을 담은 문자열도 작성할 수 있습니다.

```d
    string multiline = "
    이건
    긴
    글입니다.
    ";
```

만약 문자열 중간에 따옴표를 쓸 일이 있다면, 아래의 위지윅 문자열(Wysiwyg; What You See Is What You Get의 약자로 '보이는 그대로를 얻는다'는 뜻을 담고 있습니다) 설명 또는 [heredoc strings](http://dlang.org/spec/lex.html#delimited_strings) 글을 참고하십시오.

### 위지윅 문자열(Wysiwyg strings)

문자열 내에 들어와서는 안되는 문자를 이스케이프(escape) 처리하기 위해 공을 들이기 보다, 어쩌면 문자열을 입력한 그대로 표현해주는 어떤 것을 이용하는 게 편할 수 있습니다. 있는 그대로 표현하기 위해선 역 홑따옴표 ( `` ` ... ` `` ) 를 이용하거나 `raw` 문자열 상태임을 나타내기 위해 쌍따옴표 앞에 `r` 을 붙일 수 있습니다.

```d
    string raw  =  `raw "string"`; // raw "string"
    string raw2 = r"raw `string`"; // raw `string`
```

D 언어에는 다른 방법도 더 있으니 바쁘시더라도 [문자열을 나타내는 글자들](https://dlang.org/spec/lex.html#string_literals) 문서를 살펴보시기 바랍니다.

### 더 살펴보기

- [Unicode gem](gems/unicode)
- [Characters in _Programming in D_](http://ddili.org/ders/d.en/characters.html)
- [Strings in _Programming in D_](http://ddili.org/ders/d.en/strings.html)
- [std.utf](http://dlang.org/phobos/std_utf.html) - UTF 인코딩 종류에 대한 인코드/디코드 알고리즘 구현을 제공합니다.
- [std.uni](http://dlang.org/phobos/std_uni.html) - 유니코드 처리 알고리즘 구현을 제공합니다.
- [String Literals in the D spec](http://dlang.org/spec/lex.html#string_literals)

## {SourceCode}

```d
import std.stdio : writeln, writefln;
import std.range : walkLength;
import std.uni : byGrapheme;
import std.string : format;

void main() {
    // format을 통해 C에서 printf를 쓰듯 문자열을 만들 수 있습니다
    // 유니코드 처리는 D 언어에게 맡겨놓고 안심하고 코딩하세요.
    string str = format("%s %s", "Hellö",
        "Wörld");
    writeln("My string: ", str);
    writeln("Array length (code unit count)"
        ~ " of string: ", str.length);
    writeln("Range length (code point count)"
        ~ " of string: ", str.walkLength);
    writeln("Character length (grapheme count)"
        ~ " of string: ",
        str.byGrapheme.walkLength);

    // 문자열을 그냥 배열처럼 다뤄도 괜찮습니다.
    import std.array : replace;
    writeln(replace(str, "lö", "lo"));
    import std.algorithm : endsWith;
    writefln("Does %s end with 'rld'? %s",
        str, endsWith(str, "rld"));

    import std.conv : to;
    // UTF-32 인코딩으로 바꿉니다
    dstring dstr = to!dstring(str);
    // D 언어가 유니코드를 잘 처리해준 덕분에, 인코딩을 바꿔도 같은 문자열로 잘 보일 겁니다.
    writeln("My dstring: ", dstr);
}
```
