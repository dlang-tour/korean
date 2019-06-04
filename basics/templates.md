# 템플릿(Templates)

D 언어는 C++ 이나 자바의 **제너릭(Generic)** 지원에 쓰이는 템플릿 함수(templated functions)를 제공합니다. 어떤 타입이 템플릿 함수에 지정되면, 컴파일 시점에 지정된 타입을 처리할 수 있는 함수를 **D 언어 컴파일러**가 자동 생성하게 됩니다. 예를 들어 앞으로 들어올 미지의 타입을 `T` 로 표시해놓은 템플릿 함수를 작성해두고, 이후에 이것을 `int` 로 처리해달라고 지정하게 되면 `int` 에 맞는 함수를 알아서 생성합니다.

```d
    auto add(T)(T lhs, T rhs) {
        return lhs + rhs;
    }
```

함수 입력값(parameter) 앞에 소괄호로 감싸둔 `T` 가 보이는데, 이게 컴파일 시점에 채워질 어떤 네모칸 같은 겁니다. 이것을 템플릿 파라미터(template parameter)라고 부릅니다. `T` 는 `!` 연산자로 타입을 지정해줄 때 *지정된 타입으로 대체*됩니다.


```d
    add!int(5, 10);
    add!float(5.0f, 10.0f);

    // 주의해야할 것은, 위의 add 함수는 + 연산이 가능한 타입에 대해서만 실행이 가능하다는 것입니다.
    // 클래스(Classes) 섹션에 나왔던 Animal 클래스는, + 연산자를 지원하지 않기 때문에
    // 아래 코드는 템플릿 함수 처리 후에 컴파일 오류가 발생하게 됩니다.
    add!Animal(dog, cat);
```

### 템플릿 파라미터 생략(Implicit Template Parameters)

함수가 동작할 때 보통 두가지 시점에서 타입을 검토하게 됩니다. 컴파일 시점과 프로그램 실행 시점인데, 템플릿 함수도 마찬가지 입니다. 만약 `!` 연산자로 템플릿 파라미터를 제공하지 않는 경우, 컴파일 시점에 컴파일러가 입력값 타입 중 가장 적절한 것을 추론하려고 시도합니다. 프로그래머가 의도하는대로 추론되지 않는 경우도 있습니다.


```d
    int a = 5; int b = 10;

    // `!` 연산자는 사용되지 않았지만 a와 b가 모두 int이기 때문에 int로 추론합니다
    // 즉 add!int(a, b) 입니다
    add(a, b);

    float c = 5.0;

    // `!` 연산자는 사용되지 않았지만 a와 c 중 표현 범위가 더 넓은 float로 추론합니다
    // 즉 add!float(a, c) 입니다
    add(a, c);
```

### 템플릿 속성(Template properties)

템플릿 속성은 `func!(T1, T2, ...)` 처럼 2개 이상 사용하는 것도 가능합니다. 템플릿 파리미터에는 `string` 과 부동소수점 계통 타입(`float`, `double`) 등 모든 D 언어 기본 타입이 들어올 수 있습니다. 기본 타입만 허용하는 이유는, 컴파일 시점에 함께 전달되어야하기 때문입니다.

D 언어는 컴파일 시점에만 템플릿 함수를 처리합니다. 그래서 자바의 제너릭(Generic) 지원과는 다르며, 컴파일 시점에 모든 게 결정되기 때문에 최적의 성능을 나타내는 기계어로 컴파일된 결과물을 얻을 수 있습니다.

`struct` 나 `class`, `interface` 도 템플릿 속성 넣어 선언할 수 있습니다.

```d
    struct S(T) {
        // ...
    }
```

### 더 살펴보기

- [Tutorial to D Templates](https://github.com/PhilippeSigaud/D-templates-tutorial)
- [Templates in _Programming in D_](http://ddili.org/ders/d.en/templates.html)

#### 깊게 살펴보기

- [D Templates spec](https://dlang.org/spec/template.html)
- [Templates Revisited](http://dlang.org/templates-revisited.html):  Walter Bright writes about how D improves upon C++ templates.
- [Variadic templates](http://dlang.org/variadic-function-templates.html): Articles about the D idiom of implementing variadic functions with variadic templates

## {SourceCode}

```d
import std.stdio : writeln;

/**
동물을 클래스로 표현하기 위한 템플릿 클래스입니다
템플릿 속성 입력값:
    noise = 울음소리
*/
class Animal(string noise) {
    void makeNoise() {
        writeln(noise ~ "!");
    }
}

class Dog: Animal!("멍") {
}

class Cat: Animal!("냥") {
}

/**
makeNoise 내부 함수를 갖고 있는 어떤 타입이라도 T에 넣을 수 있는
함수를 만듭니다.
입력값:
    animal = makeNoise 가 구현된 값/객체
    n = 몇번이나 출력할지 지정
*/
void multipleNoise(T)(T animal, int n) {
    for (int i = 0; i < n; ++i) {
        animal.makeNoise();
    }
}

void main() {
    auto dog = new Dog;
    auto cat = new Cat;
    multipleNoise(dog, 5);
    multipleNoise(cat, 5);
}
```
