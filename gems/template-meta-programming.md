# 템플릿 메타 프로그래밍(Template meta programming)

C++을 먼저 해본 경험이 있어 *템플릿 메타 프로그래밍* 을 해봤다면, D 언어에도 비슷한 요소가 있기에 마음이 놓일 것입니다.

템플릿 메타 프로그래밍은 타입을 템플릿을 통해 전달해, 실행에 필요한 요소를 미리 결정하여 좀 더 일반화된 동작을 수행할 수 있는 코드를 작성하게 해줍니다.

### `static if` 와 `is` (`static if` & `is`)

`if` 는 프로그램 실행 중에 어느 코드를 실행할지 결정할 때 쓰이지만, `static if` 는 어느 코드를 컴파일해야 할지 결정하는데 쓰입니다.

`static if` 에서 컴파일될 코드는 오직 컴파일 시점에만 결정되며, 프로그램 실행 중에 결정되는 건 아닙니다.

```d
    static if(is(T == int))
        writeln("T 는 int 타입입니다");
    static if (is(typeof(x) :  int))
        writeln("변수 x는 자동적으로 int로 변환되는 타입의 값입니다.");
```

[`is` 표현식](http://wiki.dlang.org/Is_expression) 을 이용해 컴파일 시점에 타입 제네릭(type generic) 조건을 확인합니다.

```d
    static if(is(T == int)) { // T는 템플릿 파라미터로, 생략된 코드에 정의되어 있습니다
        int x = 10;
    }
```

일반적으로 `{ }` 코드 블럭은 새 스코프(scope)를 만들지만, `static if` 는 조건문이 참인 경우 단순히 `static if` 에 딸려오는 중괄호만을 제거합니다.

만약 `static if` 를 사용하면서도 새 스코프(scope)를 만들고자하면 `{ {` 로 열고 `} }` 로 닫아주셔야합니다.

`static if` 는 함수나 타입 선언부 등 그 어디서든 어디에서든 이용이 가능합니다.

### 믹스인 템플릿(`mixin template`)

자주 반복되는 *상용구 코드(boiler plate)* 를 다듬을 때 `mixin template` 을 사용하는 걸 권합니다.

```d
    mixin template Foo(T) {
        T foo;
    }
    ...
    mixin Foo!int; // { 와 }가 없어지고, int foo 변수가 선언되어 이용 가능해집니다.
```

`mixin template` 으로 `if` 나 반복문이 포함되는 복잡한 표현도 삽입하실 수 있습니다. C를 사용했던 분이라면, 이제 전처리기(pre-processor)와 이별하고 D 언어를 만나기 딱 좋은 타이밍입니다.

### 템플릿 제약사항 지정(Template constraints)

템플릿을 작성할 때, 타입이 어떤 속성들을 갖춰야하는지에 대해 여러가지 제약사항을 걸 수 있습니다.

```d
    void foo(T)(T value)
      if (is(T : int)) { // foo!T 는 T가 int 타입으로
                         // 변환될 수 있을 때만 유효합니다
    }
```

제약사항을 작성할 때 참 또는 거짓으로 결과가 나오는 여러 조건식을 조합할 수 있으며, 컴파일 시점에 결정될 수 있는 트레이트(trait)도 조건식에서 이용할 수 있습니다.

일례로, `std.range.primitives.isRandomAccessRange` 를 들 수 있습니다. 임의 접근(random access)를 지원하는 범위 탐색 타입인지 확인하는 트레이트인데, 이 임의 접근은 소스 코드 상에서 `[]` 연산자로 표현됩니다.

### 더 살펴보기

### 기본 참고문서(Basics references)

- [Tutorial to D Templates](https://github.com/PhilippeSigaud/D-templates-tutorial)
- [Conditional compilation](http://ddili.org/ders/d.en/cond_comp.html)
- [std.traits](https://dlang.org/phobos/std_traits.html)
- [More templates  _Programming in D_](http://ddili.org/ders/d.en/templates_more.html)
- [Mixins in  _Programming in D_](http://ddili.org/ders/d.en/mixin.html)

### 심도 있는 참고문서(Advanced references)

- [Conditional compilation](https://dlang.org/spec/version.html)
- [Traits](https://dlang.org/spec/traits.html)
- [Mixin templates](https://dlang.org/spec/template-mixin.html)
- [D Templates spec](https://dlang.org/spec/template.html)

## {SourceCode}

```d
import std.traits : isFloatingPoint;
import std.uni : toUpper;
import std.string : format;
import std.stdio : writeln;

/*
정수든 부동소수점이든 가리지 않고 동작하는 3차원 벡터를
표현하는 struct 를 작성합니다.
*/
struct Vector3(T)
  if (is(T: real))
{
private:
    T x,y,z;

    /*
    getter (값 반환 함수)와 setter (값 설정 함수)를
    만들어주는 믹스인 템플릿입니다
    같은 기능을 하는 코드를 여러번 반복하면 짜증나기 때문입니다

    var -> T getVAR() 와 void setVAR(T) 를 만듭니다
    */
    mixin template GetterSetter(string var) {
        // 믹스인을 사용해 적절한 getter/setter 함수명을 만듭니다
        mixin("T get%s() const { return %s; }"
          .format(var.toUpper, var));

        mixin("void set%s(T v) { %s = v; }"
          .format(var.toUpper, var));
    }

    /*
    위에 만든 함수 생성을 위한 믹스인 템플릿으로
    getX/setX, getY/setY,
    getZ/setZ 함수를 만들어 벡터를 다룹시다
    */
    mixin GetterSetter!"x";
    mixin GetterSetter!"y";
    mixin GetterSetter!"z";

public:
    /*
    dot 함수는 부동소수점 값을 담고 있는 벡터에게만 유효하다고
    정의합니다
    */
    static if (isFloatingPoint!T) {
        T dot(Vector3!T rhs) {
            return x * rhs.x + y * rhs.y +
                z * rhs.z;
        }
    }
}

void main()
{
    auto vec = Vector3!double(3,3,3);
    // Vector3 은 템플릿 제약조건으로 인해
    // 실수 범위 내에서 다룰 수 없는 값이라면
    // 컴파일에 실패합니다.
    // 아래 주석을 해제하여 테스트해보십시오.
    // Vector3!string illegal;

    auto vec2 = Vector3!double(4,4,4);
    writeln("vec 와 vec2의 내적 = ", vec.dot(vec2));

    auto vecInt = Vector3!int(1,2,3);
    // vecInt에 대해서는 dot 함수가 정의되지 않습니다.
    // Vector3 struct에 그렇게 정의해두었기 때문입니다.
    // 아래 코드의 주석을 해제해 오류가 발생하는 걸 확인하십시오.
    // vecInt.dot(Vector3!int(0,0,0));

    // 믹스인 템플릿을 사용해 만들어진 setter들을 이용해봅시다
    vecInt.setX(3);
    vecInt.setZ(1);
    writeln(vecInt.getX, ",",
      vecInt.getY, ",", vecInt.getZ);
}
```
