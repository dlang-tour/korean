# 연산자 대체(opDispatch & opApply)

D 언어는 [classes와 structs의 연산자 오버로딩](https://dlang.org/spec/operatoroverloading.html) 으로 `+`, `-` 같은 연산자나 함수 호출에 쓰이는 연산자 `()` 등을 본래 기능이 아닌 다른 기능을 수행하도록 대체하는 걸 허용합니다.

이 섹션에서 `opDispatch` 와 `opApply` 라는 특별한 함수를 작성해 본래 연산자와는 다른 동작을 수행하도록 하는 방법을 설명합니다.

### opDispatch 함수
`opDispatch` 는 `struct` 나 `class` 의 내부 함수로 선언됩니다. `struct` 나 `class` 에 대하여, 이 안에 미리 내부 함수로 정의되지 않은 함수를 호출하려고 시도하면 D 언어는 내부 함수 중 `opDispatch` 가 정의되어있는지 확인합니다.

이때 원래 호출하려던 함수의 이름은 `string` 타입으로 전달되고, 입력값 또한 템플릿 파라미터 형태로 전달됩니다. `opDispatch` 의 동작은 일종의 *블랙홀* 처럼, 미리 정의되지 않은 내부 함수 호출 시도를 모두 가져갑니다. 이 특성을 활용해 또다른 수준의 제네릭 프로그래밍(generic programming)을 **컴파일 시점** 에 완성할 수 있습니다.

```d
    struct C {
        void callA(int i, int j) { ... }
        void callB(string s) { ... }
    }
    struct CallLogger(C) {
        C content;
        void opDispatch(string name, T...)(T vals) {
            writeln("called ", name);
            mixin("content." ~ name)(vals);
        }
    }
    CallLogger!C l;
    l.callA(1, 2);
    l.callB("ABC");
```

### opApply 함수

*범위 탐색* 을 구현하지 않고 `foreach` 구문으로 탐색 가능한 타입을 구현하는 방법으로는 `opApply` 를 구현하는 방법이 있습니다.

`foreach` 로 타입 내부의 자료를 순회할 때, `opApply` 에 특별한 델리게이트(delegate)를 입력값으로 전달해줍니다. 이때 이 델리게이트를 처리해 `foreach` 를 구현할 수 있습니다.

```d
    class Tree {
        Tree lhs;
        Tree rhs;
        int opApply(int delegate(Tree) dg) {
            // 노드 하나에 최대 두 개의 서브노드가 존재하는
            // 이진트리(binary tree) 를 탐색합니다.
            // 왼쪽 -> 중앙 -> 오른쪽 순서대로 탐색하는 In-order 방식입니다.
            if (lhs && lhs.opApply(dg)) return 1;
            if (dg(this)) return 1;
            if (rhs && rhs.opApply(dg)) return 1;
            return 0;
        }
    }
    Tree tree = new Tree;

    // Tree 의 모든 원소를 왼쪽 -> 중앙 -> 오른쪽 순서로 탐색합니다
    foreach(node; tree) {
        ...
    }
```

컴파일러는 `foreach` 와 그에 묶인 코드 블럭을 특별한 델리게이트로 변환합니다. 그 델리게이트에는 현재 순회 중인 값 하나만 입력값으로 제공됩니다.

또한 그 델리게이트는 `int` 형의 값을 반환하도록 되어있는데, 이 값은 순회를 진행할지 멈출지 결정하는 마법의 숫자입니다. 매 값을 순회할 때마다 델리게이트의 반환값이 검사되며, 이 값이 `0` 이 아니라면 순회를 멈추도록 되어있습니다. 위의 예제에서는 트리의 터미널 노드(terminal node)에 도착하게 되면 1을 반환하여 탐색을 멈추게 하고 있습니다.

실제 그런지 확인해보려면, `opApply` 의 `return 0;` 을 `return 1;` 로 고쳐보십시오.

### 더 살펴보기

- [Operator overloading in _Programming in D_](http://ddili.org/ders/d.en/operator_overloading.html)
- [`opApply` in _Programming in D_](http://ddili.org/ders/d.en/foreach_opapply.html)
- [Operator overloading in D](https://dlang.org/spec/operatoroverloading.html)

## {SourceCode}

```d
/*
Variant 타입은 원래 지정된 타입 외에 다른 타입을 담을 수 있는
특별한 타입입니다. 아래 표준 라이브러리 문서를 참고하십시오:
https://dlang.org/phobos/std_variant.html
*/

import std.variant : Variant;

/*
아래의 struct var는 자바스크립트의 Object 변수처럼,
다양한 프로퍼티들을 설정하고 불러올 수 있습니다.
*/
struct var {
    private Variant[string] values;

    @property
    Variant opDispatch(string name)() const {
        return values[name];
    }

    @property
    void opDispatch(string name, T)(T val) {
        values[name] = val;
    }
}

void main() {
    import std.stdio : writeln;

    var test;
    test.foo = "test";
    test.bar = 50;
    writeln("test.foo = ", test.foo);
    writeln("test.bar = ", test.bar);
    test.foobar = 3.1415;
    writeln("test.foobar = ", test.foobar);
    // 아래 코드는 미리 지정되지 않은 값을 읽으려 시도했기 때문에 실패합니다.
    // writeln("test.notthere = ",
    //   test.notthere);
}
```
