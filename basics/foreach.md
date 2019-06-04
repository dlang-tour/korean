# 간편 순회 루프(Foreach)

{{#img-right}}dman-teacher-foreach.jpg{{/img-right}}

D 언어는 `foreach` 라는 루프 문법을 제공하는데, 일반적 `for` 루프와는 달리 증감식, 조건식 등을 제어할 필요가 없기 때문에 오류 발생할 가능성이 줄어들고 보기 좋은 코드를 작성할 수 있습니다.

### 배열 순회(Element iteration)

주어진 배열 `arr` 이 `int[]` 타입이라면, `foreach` 루프를 통하여 모든 값을 순회할 수 있습니다. 이런 게 가능한 타입을 `iterable` 이라고 표현하기도 합니다. `반복 가능한` 으로 이해할 수 있지만, 혼란을 줄이기 위해 가능한 `iterable` 이라는 단어를 직접 사용하도록 하겠습니다.

```d
    foreach (int e; arr) {
        writeln(e);
    }
```

`foreach` 구문의 소괄호 내에 첫번째로 명시된 건 `iterable` 한 타입 내의 값들을 어떤 타입과 어떤 이름으로 받아 순회할 건지 정해주는 것입니다. 타입은 명시하지 않더라도, 자동으로 예측됩니다.

```d
    foreach (e; arr) {
        // 이전 설명에서 arr 이 int[] 타입이라고 했던 것을 명심하십시오.
        // arr 내의 값들을 하나씩 받아온다면, 그 값들의 타입은 int 타입입니다.
        writeln(e);
    }
```

소괄호 내의 세미콜론 우측에 있는 입력 값은 반드시 배열이거나, 혹은 **범위 탐색(range)** 를 지원하는 `iterable` 한 값을 입력값으로 제공할 수도 있습니다. `range` 에 대해서는 [range 섹션](basics/ranges) 에서 다룹니다.

### 값 대신 값을 가르키는 주소로 접근하기(Access by reference)

이전 예제는 배열에 담겨 있는 `int` 값을 그대로 복사해서 이용하며 순회했습니다. `int`, `char` 등 기본 타입에 대해서는 큰 문제가 없겠지만 `struct` 와 같이 크고 복잡한 자료를 담고 있는 타입이라면 복사에 많은 시간이 소요됩니다. 또한 배열의 값을 바꾼다고, 복사한 값을 바꾸는 *in-place mutation* 과 같은 실수를 범할 우려가 큽니다. 따라서 원래 배열에 있던 값은 그대로 두고, 그 값을 가르키는 주소만 가져다가 순회하는 방법도 있습니다. 이를 참조(reference)라고 부릅니다. 여기에서 따온 `ref` 라는 예약어를 통해 참조만 가져올 수 있습니다.

이후 강좌에서는 참조와 레퍼런스라는 용어를 혼용해서 사용할 수도 있습니다. 여러분의 이해를 부탁드립니다.

```d
    foreach (ref e; arr) {
        e = 10; // 이전 예제와는 달리, 배열의 해당 값이 바뀌게 됩니다.
    }
```

### 정해진 회수만큼 반복하기(Iterate `n` times)

D 언어는 정확히 정해진 회수만큼만 반복되도록 지정하는 간단한 방법을 가지고 있습니다. `..` 문법을 이용하는 것인데, 아래 예제를 참고해주십시오.

```d
    foreach (i; 0 .. 3) {
        // 0 .. 3 을 통해 0부터 2까지의 숫자를 만들어 냅니다.
        // 숫자가 만들어질 동안 코드 블럭은 계속 실행됩니다.
        // 만들어진 숫자를 이용하지 않고 반복만 할 수도 있지만, 받아서 무언가 의미있는 작업을 할 수 있습니다.
        writeln(i);
    }
    // 0 1 2 이 출력됩니다.
    // 마지막 숫자는 포함되지 않습니다.
```

마지막 숫자는 범위에서 제외되기 때문에, 위의 `foreach` 루프는 3번 실행되고 종료됩니다.

### 인덱스와 함께 순회하기(Iteration with index counter)

이전의 `foreach` 는 인덱스를 직접 관리하지 않아 여러 장점이 있지만, 때로는 인덱스를 다뤄야할 필요가 있습니다. 그래서 D 언어에서는 배열에 대해서만 인덱스와 값을 함께 받아올 수 있는 방법을 제공합니다.

```d
    foreach (i, e; [4, 5, 6]) {
        writeln(i, ":", e);
    }
    // 0:4 1:5 2:6
```

### `foreach_reverse` 로 거꾸로 순회하기(Reverse iteration with `foreach_reverse`)

역순으로 가는 방법도 있습니다. `foreach_reverse` 를 이용하는 것입니다.

```d
    foreach_reverse (e; [1, 2, 3]) {
        writeln(e);
    }
    // 3 2 1
```

### 더 살펴보기

- [`foreach` in _Programming in D_](http://ddili.org/ders/d.en/foreach.html)
- [`foreach` with Structs and Classes  _Programming in D_](http://ddili.org/ders/d.en/foreach_opapply.html)
- [`foreach` specification](https://dlang.org/spec/statement.html#ForeachStatement)

## {SourceCode}

```d
import std.stdio : writefln;

void main() {
    auto arr = [
        [5, 15],      // 더해서 20이 나옵니다.
        [2, 3, 2, 3], // 더해서 10이 나옵니다.
        [3, 6, 2, 9], // 더해서 20이 나옵니다.
    ];

    foreach (i, row; arr)
    {
        double total = 0.0;
        foreach (e; row)
            total += e;

        auto avg = total / row.length;
        writefln("평균값 [%d행]: %.2f", i, avg);
    }
}
```
