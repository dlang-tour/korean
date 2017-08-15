# 구조체(Structs)

D 언어에서 `struct` 타입을 이용해 하나 이상의 타입+값을 보관할 수 있는 복합적 타입을 선언할 수 있습니다. 구조체라고 부르기도 합니다. 구조체 속의 변수들은 멤버(member) 라고 부릅니다.

```d
    struct Person {
        int age;
        int height;
        float ageXHeight;
    }
```

`struct` 타입을 변수로 사용하게 되면 함수 호출 스택(stack)에 메모리 공간이 할당되고, 다른 변수에 할당하거나 타 함수를 호출할 때 전달하게 되면 **변수 전체가 복사** 되어 전달됩니다. 프로그램 실행 중에 자유롭게 크기가 줄어들고 늘어나는 힙(heap) 공간에 할당 받으려면 `new` 연산자를 사용해야합니다.

```d
    auto p = Person(30, 180, 3.1415);
    // t에 p가 갖고 있는 값이 복사됩니다. t의 내용이 바뀌어도 p의 내용은 바뀌지 않습니다.
    auto t = p; 
```

`struct` 타입으로 새 변수를 만든 경우, 구조체 내의 값들은 `struct` 변수 선언시에 지정한 기본값으로 초기화 됩니다. 변수 생성시 추가적인 작업이 필요하다면, D 언어의 기본 생성자(constructor) 대신에 프로그래머가 직접 작성한 생성자를 쓸 수 있습니다. 이때 선언되는 생성자는 구조체 내부의 함수이고, 구조체 내에 선언되며 함수 이름은 `this` 입니다. `this` 로 전달되는 입력값 변수명과 구조체 내의 변수명이 충돌되지 않도록, 내부 변수를 지칭할 때는 `this.` 를 앞에 붙여 지칭할 수 있습니다.

정리하면, `this` 는 생성자 이름으로 쓰이기도 하고 구조체 내부에서 자신의 변수를 가르킬 때 쓰입니다.

```d
    struct Person {
        this(int age, int height) {
            this.age = age;
            this.height = height;
            this.ageXHeight = cast(float)age * height;
        }
            ...

    Person p(30, 180); // 변수 초기화
    p = Person(30, 180);  // 새로운 값을 변수에 할당합니다
```

`struct` 는 `this()` 외에도 다른 내부 함수를 포함할 수 있습니다. 이렇게 추가된 함수들은 `public` 접근 영역을 가지고 있어 구조체 밖에서도, 해당 구조체 타입을 가진 변수를 통해 실행이 가능합니다. 만약에 내부 함수 중 외부에 노출시키고 싶지 않은 함수가 있다면 `private` 키워드를 앞에 붙여 선언하면 됩니다. `private` 함수는 구조체 내에 선언된 함수 내에서는 호출 가능하지만, 외부에서는 호출할 수 없습니다.

```d
    struct Person {
        void doStuff() {
            ...
        private void privateStuff() {
            ...

    p.doStuff(); // doStuff라는 public 함수를 p를 통해 호출합니다.
    p.privateStuff(); // private 이기 때문에 p를 통하더라도 호출이 불가능합니다.
```

### 불변 함수(Const member functions)

구조체 내부 함수는 내부 변수를 바꿀 수 있는 능력을 가지고 있지만, 이 함수를 `const` 키워드를 붙여 선언하게 되면 구조체 내부의 값을 변경할 수 없게 됩니다.

이런 제약조건은 컴파일 시점에 점검되며, 이렇게 선언된 함수는 해당 구조체 변수 선언시 `const` 나 `immutable` 로 선언되어도 호출될 수 있습니다. `const` 가 없는 함수는 내부 값을 변경할 수 있기 때문에, D 언어는 그런 함수들이 실행되는 걸 막습니다. `const` 는 프로그래머에게 이 함수를 호출해도 내부 값이 바뀌지 않는다는 든든한 믿음을 주는 강점도 있습니다.

```d
    struct Student {
        string name = "default";
        string nickname;

        const void nogoodFunction() {
            this.name = "fireman"; // const 함수가 내부 값을 바꾸려 했으므로, 컴파일에 실패합니다.
        }
    }
```

### 정적 함수(Static member functions)

내부 함수 중 `static` 으로 선언된 함수는 변수 선언하지 않고 구조체 타입의 이름을 적는 것만으로도 호출이 가능합니다. 예를 들면 `Person.myStatic()` 처럼 말입니다.

이 함수는 변수를 선언하지 않기 때문에, 선언했을 때야 접근 가능한 내부 변수들에 대해선 접근할 수 없습니다. 따라서, `static` 함수가 어떤 내부 변수를 이용하고자하면 내부 변수 또한 `static` 으로 선언되어야합니다.

`static` 이 주로 쓰이는 경우를 보면, 변수 선언을 하지 않고 해당 구조체 타입과 연관된 어떤 작업을 수행해야할 때가 많습니다. 그 일례로 싱글턴 디자인 패턴(Singleton design pattern)에서 프로그램 전체에서 유일무이한 단 하나의 객체(instance)를 만드는 사례를 들 수 있습니다.

### 구조체 상속 관계(Inheritance)

`struct` 는 다른 `struct` 타입으로부터 선언된 구조를 가져오는, 상속 행위를 할 수 없습니다. 대신 `class` 를 사용할 수 있지만, 현재 이 강좌에서 다루지는 않겠습니다. 그렇지만 `struct` 만 이용하더라도 `alias this` 나 `mixins` 같은 걸 사용해 Java 등의 객체지향 언어에서 나타나는 다형성 상속(polymorphic inheritance)를 흉내낼 수 있습니다. `mixins` 는 이 D 언어 강좌에서 다루지 않습니다. 궁금하시다면 [D 언어 웹사이트](https://dlang.org/mixin.html) 를 확인해주십시오.

### 더 살펴보기

- [Structs in _Programming in D_](http://ddili.org/ders/d.en/struct.html)
- [Struct specification](https://dlang.org/spec/struct.html)

### 실습

`struct Vector3` 는 x, y, z 축 좌표를 담는 3차원 벡터를 표현한 자료구조입니다. 여기에는 미처 구현되지 않은 내부 함수들이 있습니다. 예제 프로그램이 정상적으로 동작할 수 있도록 비어있는 부분을 직접 구현해보십시오.

* `length()` - 벡터의 '길이'를 반환합니다.
* `dot(Vector3)` - 두 벡터 사이의 내적(inner product, dot product)을 구합니다.
* `toString()` - 사람이 봤을 때 적절한 형태로 Vector3를 문자로 표현해주십시오.
  여기에 사용된 [`std.string.format`](https://dlang.org/phobos/std_format.html) 함수는
  `printf` 를 사용하는 것과 비슷하지만 대신 결과물로 문자열을 내놓습니다.
  `format("MyInt = %d", 13)` 을 호출하면 `"MyInt = 13"` 이라는 문자열이 나옵니다.
  `std.string` 모듈에 대해서는 이후 관련 섹션에서 다루겠습니다.

## {SourceCode:incomplete}

```d
struct Vector3 {
    double x;
    double y;
    double z;

    double length() const {
        import std.math : sqrt;
        // TODO: 벡터의 길이를 구합니다. 구현이 필요합니다.
        return 0.0;
    }

    // rhs will be copied
    double dot(Vector3 rhs) const {
        // TODO: 벡터의 내적을 구합니다. 구현이 필요합니다.
        return 0.0;
    }
}

void main() {
    auto vec1 = Vector3(10, 0, 0);
    Vector3 vec2;
    vec2.x = 0;
    vec2.y = 20;
    vec2.z = 0;

    // 호출하려는 내부 함수에 별도의 입력값이 필요하지 않다면,
    // 마치 프로퍼티(property)를 이용하듯 소괄호를 생략할 수 있습니다.
    assert(vec1.length == 10);
    assert(vec2.length == 20);

    // 내적 함수의 동작을 검증합니다.
    assert(vec1.dot(vec2) == 0);

    // 내적은 벡터내의 각 요소의 값을 곱해 더합니다.
    // 아마도 (1,2,3) 벡터와 (1,1,1) 벡터의 내적은
    // 1 * 1 + 2 * 1 + 3 * 1 처럼 계산될 겁니다. (힌트)
    auto vec3 = Vector3(1, 2, 3);
    assert(vec3.dot(Vector3(1, 1, 1)) == 6);

    // 위와 마찬가지입니다.
    // 1 * 3 + 2 * 2 + 3 * 1
    assert(vec3.dot(Vector3(3, 2, 1)) == 10);
}
```
