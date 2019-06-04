# 루프(Loops)

반복되는 작업을 코드로 옮기기 위해, D 언어는 4가지 방법을 통해 루프를 구현할 수 있게 되어있습니다.

### 1) `while`

`while` 은 주어진 조건에 해당하는 상황 동안에 코드 블록(중괄호 사이의 코드들)을 반복합니다.

```d
    while (condition)
    {
        foo();
    }
```

### 2) `do ... while`

`do .. while` 은 `whilte` 과 비슷하지만, 반복하고 싶은 _코드 블록_ 이 한번 실행된 다음에 반복 조건을 확인합니다.

```d
    do
    {
        foo();
    } while (condition);
```

### 3) 일반적인 `for` loop

C/C++ 이나 Java에서 알려진 _초기식_, _조건식_, _증감식_ 을 입력하는 방법의 `for` 루프입니다.

```d
    for (int i = 0; i < arr.length; i++)
    {
        ...
```

### 4) `foreach`

[`foreach` 루프](basics/foreach) 에 대해서는 다음 섹션에서 다룰 예정입니다. 간단히 정리하면, 일반적 `for` 루프처럼 복잡한 제어를 하지 않아도 배열이나 리스트의 모든 원소를 순회할 수 있는 루프입니다.

```d
    foreach (el; arr)
    {
        ...
    }
```

#### 특별한 예약어와 루프 레이블(Special keywords and labels)

`break` 는 현재 루프를 종료하기 위한 예약어 입니다. 하지만 `break` 단독으로는 현재 루프만을 종료시키기 때문에, 여러 루프 안에서 2단계 이상 거슬러 빠져 나가려면 _레이블_ 을 적어놓고 `break` 와 함께 나갈 지점의 레이블을 지정해야합니다.

```d
    outer: for (int i = 0; i < 10; ++i)
    {
        for (int j = 0; j < 5; ++j)
        {
            ...
            break outer;
```

`continue` 라는 예약어는 현재 루프를 `continue` 지점에서 끝내되, 다음 루프 반복을 할 수 있게 합니다. 다음 루프를 반복할 수 없다면 루프문은 종료됩니다.

### 더 살펴보기

- `for` 루프 [_관련 D 언어 글_](http://ddili.org/ders/d.en/for.html), [specification](https://dlang.org/spec/statement.html#ForStatement)
- `while` 루프 [_관련 D 언어 글_](http://ddili.org/ders/d.en/while.html), [specification](https://dlang.org/spec/statement.html#WhileStatement)
- `do-while` 루프 [_관련 D 언어 글_](http://ddili.org/ders/d.en/do_while.html), [specification](https://dlang.org/spec/statement.html#do-statement)

## {SourceCode}

```d
import std.stdio : writeln;

/*
배열 속에 담긴 숫자의 평균을 구합시다.
*/
double average(int[] array)
{
    immutable initialLength = array.length;
    double accumulator = 0.0;
    while (array.length)
    {
        // 첫번째 값을 얻기 위해 [0]를 쓰는 대신,
        // .front 를 이용해서 해결할 수 도 있습니다.
        // 해당 함수는 import std.array : front; 문으로
        // 가져올 수 있습니다
        accumulator += array[0];
        array = array[1 .. $];
    }

    return accumulator / initialLength;
}

void main()
{
    auto testers = [ [5, 15], // 20
          [2, 3, 2, 3], // 10
          [3, 6, 2, 9] ]; // 20

    for (auto i = 0; i < testers.length; ++i)
    {
      writeln("평균 ", testers[i],
        " = ", average(testers[i]));
    }
}
```
