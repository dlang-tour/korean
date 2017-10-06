# 클래스(Classes)

D 언어는 Java나 C++에서 보던 클래스와 인터페이스 타입을 제공합니다.

모든 `class` 타입은 [`Object`](https://dlang.org/phobos/object.html) 타입을 명시하지 않더라도 자동적으로 `Object` 클래스를 확장하도록 구현됩니다. 이렇게 원래 있던 `class` 를 가져와 새로운 `class` 로 확장하는 걸 상속(inherit)이라고 부릅니다.

```d
    class Foo { } // 명시적 선언이 없어도 Object 를 상속합니다.
    class Bar : Foo { } // Foo 는 Bar를 확장하는 클래스입니다.
```

D 언어의 클래스를 객체(object)로써 사용하려면 일반적으로 `new` 연산자를 사용해 힙(heap) 공간에 생성 및 초기화를 해주어야합니다.

```d
    auto bar = new Bar;
```

클래스로부터 만들어진 객체는 언제나 참조(reference) 타입으로, 포인터를 통해 다루게 됩니다. 한편 과거에 배운 `struct` 는 값으로 다뤄진다는 걸 다시금 상기하십시오.

```d
    Bar bar = foo; // foo 객체가 bar로 복사되는 게 아니라, 같은 객체를 가르키는 주소(포인터)만 복사됩니다
```

가비지 컬렉터(garbage collector)가 더이상 사용되지 않는 객체를 정리하여 프로그램 실행에 필요한 메모리 공간을 확보합니다.

### 상속(Inheritance)

원래 있던 클래스를 확장하여 새로운 클래스를 만들 때, 원래 있던 내부 함수 대신 같은 이름의 새로운 내부 함수를 작성하려면 `override` 키워드를 사용해야합니다. `override` 없이 같은 이름으로 새 함수를 만들게 되면 오류가 발생하는 메커니즘을 통해, 실수로 다른 함수를 덮어쓰는 걸 방지합니다.


```d
    class Bar : Foo {
        override functionFromFoo() {}
    }
```

D 언어는 클래스의 다중 상속(Multiple Inheritance)을 허용하지 않습니다. 한번 상속 받을 때 단 하나의 클래스만 상속 받을 수 있습니다.

### final과 구현 필요 상태 표시(Final and abstract member functions)

- 클래스 내부 함수 작성시 `final` 로 표기하여, 이후 상속에서 같은 이름의 새 함수를 정의하는 걸 막을 수 있습니다.
- `abstract` 키워드로 함수를 선언하면 상속 후 직접 구현하지 않았을 때 컴파일이 되지 않습니다. 앞으로 구현되어야함을 나타낼 때 씁니다.
- `abstract` 키워드로 클래스를 선언하면 그 클래스로부터 직접 객체를 만들 수 없으며, `abstract` 가 아닌 클래스가 `abstract` 클래스를 상속 받아 부족한 구현을 채워야만 객체를 생성할 수 있습니다.
- `super(..)` 는 상속하려는 클래스의 원본 생성자(constructor)를 호출할 때 쓰입니다.

### 동일 여부 확인(Checking for identity)

클래스로부터 만들어진 객체들은 참조(reference) 형태로 접근되고, 값이 같은지 비교하는 연산자인 `==` 와 `!=` 는 참조가 가르키는 원본 객체의 구성상태를 비교하게 됩니다. 따라서 `null` 과 비교하는 건 `null` 의 원본 객체가 없기 때문에 비교할 수 없어 오류가 발생합니다. 그래서 해당 객체가 `null` 상태인지 아닌지 체크하려면 `is` 연산자를 사용해야합니다. `is` 는 비교하려는 객체가 해당 클래스로부터 만들어졌는지 확인할 때도 쓰입니다. 만약 반대의 경우를 확인하려면 `e1 !is e2` 꼴로 확인하십시오.

```d
MyClass c;
if (c == null)  // 주소가 null이기 때문에 원본 객체의 주소를 모릅니다. 오류가 발생합니다.
    ...
if (c is null)  // 해당 객체가 null 상태인지 확인합니다. 이 방법은 동작합니다.
    ...
```

한편 `struct` 는 동일함을 확인하기 위해 메모리상의 1비트 1비트를 모두 비교하여 일치하는지 검사합니다.

그 외의 모든 값의 타입에 대해서는 `is` 를 의식하지 않고 비교해도 됩니다. 이런 타입들의 비교에서는 동일(identity)과 동질(equality)의 차이가 발생하지 않습니다.

### 더 살펴보기

- [Classes in _Programming in D_](http://ddili.org/ders/d.en/class.html)
- [Inheritance in _Programming in D_](http://ddili.org/ders/d.en/inheritance.html)
- [Object class in _Programming in D_](http://ddili.org/ders/d.en/object.html)
- [Classes in D spec](https://dlang.org/spec/class.html)

## {SourceCode}

```d
import std.stdio : writeln;

/*
무엇이든 하기 위해 일단 Any 클래스를 만들어보았습니다.
*/
class Any {
    // protected 키워드로 선언된 변수는 자신 또는 자신을 상속한 클래스만
    // 직접 값을 읽거나 변경할 수 있습니다.
    protected string type;

    this(string type) {
        this.type = type;
    }

    // 별다른 선언이 없다면 기본은 public 입니다
    // 객체를 가져올 수 있다면 누구든 사용할 수 있습니다
    final string getType() {
        return type;
    }

    // abstract 로 선언되어있으니,
    // 이걸 상속하여 구현해줄 클래스를 만들어야합니다
    abstract string convertToString();
}

class Integer : Any {
    // Integer 클래스가 내부에 품을 값입니다
    private {
        int number;
    }

    // 객체 생성시 호출되는 생성자 함수입니다
    this(int number) {
        // 확장하려던 원본 클래스 Any의 생성자를 호출합니다.
        super("integer");
        this.number = number;
    }

    // 이것도 이후 함수들이 public 이라는 암묵적인 선언입니다.
    public:

    override string convertToString() {
        import std.conv : to;
        // 타입 변환을 위한 만능키 같은 문법입니다
        return to!string(number);
    }
}

class Float : Any {
    private float number;

    this(float number) {
        super("float");
        this.number = number;
    }

    override string convertToString() {
        import std.string : format;
        // 지저분한 모든 소수점을 보고 싶진 않을 겁니다
        // 소수 첫째 자리까지만 보이게 형식을 지정합시다
        return format("%.1f", number);
    }
}

void main()
{
    Any[] anys = [
        new Integer(10),
        new Float(3.1415f)
    ];

    foreach (any; anys) {
        writeln("any의 타입은 ", any.getType());
        writeln("내용물은 ",
            any.convertToString());
    }
}
```
