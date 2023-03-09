## *10/01/23*

## Move-semantic

То как под капотом компилятор отличает `move` от копирования, что 
вызывается, когда мы хортим скопировать или перместить объект.

### Value category

```cpp
Each expression has:
1. Non-reference(!) type.
2. Value category: 
┌────────────────────────────┐
│         glvalue            │
│      (Generalized Left)    │
│              
┌─────────────┼───────────────────┐
│              │             │      rvalue       │
│              │             │      (Right)      │
│ ┌──────┐     │ ┌─────────┐ │   
┌────────────┐  │
│ │lvalue│     │ │ xvalue  │ │   │  prvalue   │  │
│ │(Left)│     │ │(eXpired)│ │   │(Pure Right)│  │
│ └──────┘     │ └─────────┘ │   
└────────────┘  │
└──────────────┴─────────────┴───────────────────┘
glvalue: "has name", can be polymorphic, has address.
rvalue: "can be moved from"
Special case: `void()`
```

*Категория значения* — некий формализм, который подсказывает компилятору, 
когда нужно копировать а когда перемещать. Возникает когда есть выражение 
— вызов функции, вызов оператора +, вызов еще чего-нибудь.

1) у каждого выражения есть тип (vector, int, str, …)

Строго говоря тип ≠ ссылке!

2) value category

- **lvalue** (может быть слева от знака =)
- **xvalue** (то из чего можно замувать)
- **prvalue** (справа от знака = + из него *нельзя* мувать)
- **gvalue** (lvalue + xvalue) — что угодно, у чего есть имя
- **rvalue** (xvalue + prvalue) — (то из чего можно замувать ) временные 
или то, к чему применили `std::move`

Специальный случай: `void()` — есть тип void, но нет value category.

***Примеры:***

```cpp
int get_value() { // value
    return 10;
}

int &get_ref() { // reference
    static int x;
    return x;
}

int &&get_rvalue_ref() { // rvalue-reference
    static int x;
    return std::move(x);
}

const int &get_cref() { // const reference
    static int x;
    return x;
}

int *get_ptr() { // pointer
    static int x;
    return &x;
}

int main() {
    int a = 10;
    int arr[5]{};
    struct { int x, y; } s;
    std::vector<int> vec(10);

    // lvalue: "has name", "cannot be moved from because lives long 
enough"
    std::cin;
    a;
    get_ref();
    get_cref();  // type: const int
    *get_ptr();
    arr;
    arr[4];  // `arr` should be lvalue
    s.x;  // `s` should be lvalue

    // prvalue: does not have name, "temporary" object
    10;
    get_value();
    get_value() + 20;
    &a;
    static_cast<int>(10.5);
    // `this` is also a prvalue.
    []() { return 23; };

    // xvalue: will expire soon, but has name
    std::move(vec);
    std::move(vec)[9];
    std::move(s).x;
    static_cast<std::vector<int>&&>(vec);  // std::move is almost exactly 
that.
    get_rvalue_ref();
}
```

### std::move

`std::move` — возвращает `&&` — это фактически 
`static_cast<std::vector<int>&&>(vec)&`

То есть это просто конструкуция, которая переводит выражение из одной 
категории в другую. Самим перемещением занимается кто-то другой.

### References

**Lvalue references**

Ссылку можно привязать только к lvalue.

Ссылку нельзя привязать к prvalue.

Ссылку нельзя привязать к std::move(x).

```cpp
int& foo() {
    static int z = 0;
    return z;
}

int bar() {
    static int z = 0;
    return z;
}

int& bad_foo() {
    int x = 10;
    return x;  // no lifetime extension! dangling reference.
}

int main() {
    int x = /* prvalue */ 10;
    x;  // lvalue
    std::move(x);  // xvalue

// lvalue references, can only bind to lvalue
int &x0 = 10;
int &x1 = /* should be lvalue */ ((((x))));
int &x2 = /* should be lvalue */ foo();

// int &x3 = /* prvalue, CE */ bar();
// int &x4 = /* prvalue */ (/* lvalue */ x + /* prvalue */ 10 + /* prvalue 
*/ bar());  // CE
// int &x5 = /* xvalue is rvalue, CE */ std::move(x);
```

**Rvalue references**

Можно привязать только к rvalue.

Lifetime extension —> объект будет жить, пока жива ссылка.

! std::move(/*prvalue*/) —> бессмысленно.

```cpp
// rvalue references, can only bind to rvalue (xvalue + prvalue)
int &&y1 = /* prvalue */ 10;
int &&y2 = /* prvalue */ bar();  // lifetime extension: lifetime of the 
temporary is extended to lifetime of y2
int &&y2b = /* xvalue */ std::move(/* prvalue */ bar());  // no lifetime 
extension, accessing y2b is UB.
    // One should never move(prvalue).
int &&y3 = /* xvalue */ std::move(x);
// int &&y4 = /* lvalue, CE */ x;
```

**Const references**

Константные ссылки могут привзятаь к чему-угодно по историческим причинам. 

```cpp
// const lvalue references, can bind anywhere for historical reasons.
// "Rvalue lifetime disaster": https://www.youtube.com/watch?v=zzkpTbJiFPM
const int &z1 = /* lvalue */ x;
const int &z2 = /* prvalue */ 10;  // lifetime extension
const int &z3 = /* xvalue */ std::move(x);  // move(x) == 
static_cast<int&&>(x)
const int &z4 = bar();  // lifetime extension
```

Может быть такое. А ссылки можно использовать где угодно.

```cpp
// Also possible, but mostly useless.
const int &&zz1 = 10;
// const int &&zz2 = x1;
int (&(zzz)) = x;  // Just weird declaration syntax.
((x)) = 20;
```

### Special methods

Зачем: пишем свой класс, который должен копироваться/муваться необычным 
способом, то нужно реализовывать пять специальных методов.

***Правило трех*** —> устарело.

***Правило нуля*** —> не надо заниматься реализацией методов, которые 
описаны внизу (потому что нужно их аккуратно реализовывать..).

***Правило пяти*** —> если начали реализовывать хотя бы один из методов, 
то придется реализовать все пять (например, если копировать нужно хитро, 
то и перемещать скорее всего тоже).

Специальные методы:

1) **Copy constructor** —> not rvalue 

```cpp
Foo(const Foo &other) {  // copy constructor: direct init, copy init...
    std::cout << "Foo(const Foo&) " << this << " <- " << &other << "\n";
    /* lvalue */ other;
}
```

2) **Move constructor** —> rvalue

```cpp
Foo(Foo &&other) {  // move constructor: same, but initialized from rvalue
    std::cout << "Foo(Foo&&) " << this << " <- " << &other << "\n";
    /* lvalue */ other;
}
```

3) **Копирующий** **оператор присваивания** —> not rvalue (copy)

```cpp
Foo& operator=(const Foo &other) {
        std::cout << "operator=(const Foo&) " << this << " = " << &other 
<< "\n";
        return *this;
    }
```

4) **Оператор присваивания** —> rvalue (move)

Move или copy происходит в соответвующем конструкторе. std::move лишь 
помогает выбрать соответвующую перегрузку.

```cpp
Foo& operator=(Foo &&other) {
        std::cout << "operator=(Foo&&) " << this << " = " << &other << 
"\n";
        return *this;
    }
```

5) **Деструктор**

```cpp
~Foo() {
        std::cout << "~Foo() " << this << "\n";
    }
```

Примеры:

**&&** — ссылка на временный объект. При этом можно менять объект по этой 
ссылке.

```cpp
void call_by_value(Foo f) {
    std::cout << "call_by_value " << &f << "\n";
}

Foo create_foo() {
    Foo f;
    std::cout << "create_foo " << &f << "\n";
    return f;  // `f` is local variable => considered rvalue, please do 
not std::move it, details later.
}

int main() {
    Foo a;
    Foo b(/* lvalue */ a);  // Foo b = a
    Foo c(/* xvalue */ std::move(a));  // Foo c = std::move(a);
    Foo d;

    b = /* lvalue */ d;
    b = /* xvalue */ std::move(d);
    b = /* xvalue */ static_cast<Foo&&>(d);
    b = /* prvalue*/ Foo{};

    std::cout << "===== call 1 =====\n";
    call_by_value(/* lvalue */ a); // copy constructor

    std::cout << "===== call 2 =====\n";
    call_by_value(/* xvalue */ std::move(a)); // move constructor

    // Copy elision: copies/moves may be optimized out even if there are 
side effects, details are for later.
    std::cout << "===== call 3 =====\n";
    call_by_value(/* prvalue */ create_foo()); // optimization !

    std::cout << "===== dtors =====\n";
}
```

На самом деле, можно использовать замуванные объекты, потому что возможно 
определить, в каком состоянии они останутся после std::move. Просто в 
стандарте определили только для unique_ptr.

**optimization — copy elision:** место создания временных переменных 
компилятор оптимизирует и f локальная переменная создается в месте, где 
потом будет аргумент  call_by_value. Лучше делать return f, а не return 
std::move(f), чтобы компилятор мог оптимизировать.

## Templates

В С использовался `#define` (макрос), что хорошо ломается (компилятор не 
видит ошибок, просто ctrl-c, ctrl-v).

```cpp
#include <iostream>

#define DECLARE_OPTION(T) \
struct Optional ## T { \
private: \
    T value; \
    bool exists; \
public: \
};

DECLARE_OPTION(int)
DECLARE_OPTION(std::string)

struct OptionalString {
private:
    std::string value;
    bool exists;
public:

};

int main() {
}
```

### Class template

Улучшение С++ — шаблоны класса. Но у структуры есть параметры.

```cpp
#include <iostream>

// class template
template<typename/* class */ T, char default_value>
struct templ_foo {
    T value;
    bool exists;
};

int main() {
    [[maybe_unused]] templ_foo<int, 10> x;
    [[maybe_unused]] templ_foo<int, 10> y;
    [[maybe_unused]] templ_foo<int, 11> z;
    [[maybe_unused]] templ_foo<std::vector<int>, 10> botva;
    x = y;  // OK
    x = z;  // CE: different types

    std::cout << sizeof(x) << " " << sizeof(z) << " " << sizeof(botva) << 
"\n";

    struct Foo {};
    struct Bar {};
    Foo x1;
    Bar y1;
    x1 = y1;
}
```

### Instantiation

Подстановка конкретных типов в шаблоны, при которой происходит проверка 
корректности полей. Методы же не проверяются, они компилируются в момент 
*инстанцирования* (первого обращения) к этому методу. То есть методы могут 
быть корректными в зависимости от того, от какого типа вызываются шаблоны.

```cpp
#include <iostream>
#include <vector>
#include <set>

// class template
template<typename/* class */ C>  // C++20: concepts
struct templ_foo {
    typename C::iterator value;  // checked on class template 
instantiation
    // std::vector<int>::iterator

    // methods are checked on method instantiation only
    static void static_method() {
        C::foobarbaz();
    }

    void foo(C &c) {
        c.push_back(10);
    }

    void wtf() {
        C::hregfiuhtrghtiughgihtrg(10, 'h', "hello");
    }
};

int main() {
    {
        [[maybe_unused]] templ_foo<std::vector<int>> x;
        std::vector<int> v;
        x.value = v.begin();
        x.foo(v);

        // templ_foo<std::vector<int>>::static_method();
    }

    {
        [[maybe_unused]] templ_foo<std::set<int>> x;
        std::set<int> s;
        x.value = s.begin();
//        x.foo(s);
    }
//    [[maybe_unused]] templ_foo<int> y;
}
```

### Shortcut

Внутри класса можно опускать <T> и писать my_ptr вместо my_ptr<T>.

```cpp
my_ptr/*<T>*/() : data(new T()) {}
my_ptr(T value) : data(new T(std::move(value))) {}
```

### Function template

Функция, принимающая что-то типа Т.

```cpp
// function template
template<typename T>
void swap(T &a, T &b) {
    T tmp = std::move(a);
    a = std::move(b);
    b = std::move(tmp);
}

int main() {
    int x, y;
    [[maybe_unused]] short z;
    swap<int>(x, y);
    swap<int>(x, z);  // compilation error: cannot bind int& to short
}
```

### Argument deduction

Компилятор может догадываться сам, какие типы у аргументов.

(ссылки не конвертируются!)

```cpp
// function template
template<typename T>
void swap(T &a, T &b) {
    T tmp = std::move(a);
    a = std::move(b);
    b = std::move(tmp);
}

int x, y;
[[maybe_unused]] short z;
// template argument deduction
swap(x, y);  // Arg1=int, Arg2=int => T = int
// swap(x, z);  // compilation error: deduced conflicting types for T
```

Чтобы вызвать функцию от двух разных параметров, можно создать два 
независимых шаблона:

```cpp
template<typename T, typename U>
void foo(const std::vector<T> &, const std::vector<U> &) {
}

foo(std::vector<int>{}, std::vector<short>{});  // T=int, U=short
foo(std::vector<int>{}, std::vector<int>{});  // T=int, U=int
```

Корректность шаблона все еще нельзя проверить наверняка, она очень зависит 
от типа. Например, swap не будет работать для типов, которые нельзя 
перемещать. Какие-то требования к типу можно добавить только c C++20 :c

```cpp
#include <vector>
#include <utility>

// function template
template<typename T>
void swap(T &a, T &b) {
    T tmp = std::move(a);
    a = std::move(b);
    b = std::move(tmp);
}

struct unmovable {
    unmovable() = default;
    unmovable(const unmovable &) = delete;
    unmovable(unmovable &&) = delete;
    unmovable &operator=(const unmovable &) = delete;
    unmovable &operator=(unmovable &&) = delete;
    ~unmovable() = default;
};

int main() {
    int x, y;
    swap(x, y);

    unmovable a, b;
    swap(a, b);
}
```

### Полиморфизм во время компиляции

Static polymorphism —> *templates*

Dynamic polymorphism —> *inheritance*

На самом деле почти все, что использует шаблоны, можно писать через 
наследование. Но обычно выбирают шаблоны.

+ —> все хорошо оптимизируется, нет расплаты за виртуальные вызовы, код 
получается короче.

- —> если что-то не скомпилировалось, то будет больно отлаживать код..

```cpp
#include <memory>
#include <list>
#include <vector>
#include <iostream>

// 'Static'/'compile-time' polymorphism
template<typename C>
bool is_empty(const C &c) {
    return c.empty();  // Which .empty() to call is determined in compile 
time
}

// 'Dynamic'/'run-time' polymorphism
struct AbstractContainer { virtual bool empty() = 0; };
struct VectorInt : AbstractContainer {
    std::vector<int> vec;
    bool empty() override {
        return vec.empty();
    };
};
struct ListInt : AbstractContainer {
    std::list<int> lst;
    bool empty() override {
        return lst.empty();
    };
};

bool is_empty(AbstractContainer &c) {
    return c.empty();  // Which .foo() to call is determined in run time
}

std::unique_ptr<AbstractContainer> create_container() {
    int type;
    std::cin >> type;
    if (type == 0) {
        return std::make_unique<VectorInt>();
    } else {
        return std::make_unique<ListInt>();
    }
}

int main() {
    std::vector<int> vec;
    std::list<int> lst;
    // vec's and lst's types are known
    std::cout << is_empty(vec) << "\n";
    std::cout << is_empty(lst) << "\n";

    std::unique_ptr<AbstractContainer> cont = create_container();
    // `cont` can be anything depending on create_container()
    std::cout << is_empty(*cont) << "\n";
}
```

## Functors

### Как принимать и вызывать функторы в функциях

Через шаблоны.

```cpp
#include <iostream>

template<typename F>
void apply(F operation) {
    std::cout << "Calling with 10\n";
    operation(10);
}

struct Print {
    void operator()(int x) const {
        std::cout << x << "\n";
    }
};

struct PrintTwice {
    void operator()(int x) const {
        std::cout << x << ", " << x << "\n";
    }
};

void no_op(int) {}
// void no_op(double) {}

int main() {
    apply/*<void(*)int>*/(no_op);
    apply/*<Print>*/(Print{});
    apply/*<Print>*/(Print{});
    apply/*<PrintTwice>*/(PrintTwice{});
    // apply(12);
}
```

### Поменять состояние

`apply(std::ref(p))` —> тогда изменится p.

```cpp
#include <iostream>
#include <functional>

template<typename F>
void apply(F operation) {
    std::cout << "Calling with 10\n";
    operation(10);
}

struct Print {
    int printed = 0;
    // std::ref

    void operator()(int x) {
        std::cout << x << "\n";
        printed++;
    }
};

int main() {
    Print p;
    std::cout << "printed=" << p.printed << "\n";
    apply(p);
    // apply(std::ref(p));
    std::cout << "printed=" << p.printed << "\n";
}
```

### Хранение

Опять шаблоны.

```cpp
#include <iostream>

template<typename F, typename T>
struct Caller {
    F operation;
    T value;

    void operator()() {
        operation(value);
    }
};

struct Printer {
    void operator()(int a) const {
        std::cout << a << "\n";
    }
};

int main() {
    Caller<Printer, int> c{Printer{}, 10};
    c();
    c();
}
```

## Операторы

### Operator*()

Унарный, возвращает ссылку на объект.

```cpp
struct int_wrapper {
    int data = 10;
    int &operator*() {
        return data;
    }
};
```

### Operator->()

Бинарный. Возвращает то, к чему можно применить ->.

```cpp
struct point {
    int x = 0, y = 0;
};

struct point_wrapper {
    point p;
    point &operator*() {
        return p;
    }

    point *operator->() {
        return &p;
    }
};

std::cout << p->x << "\n";  // (p.operator->())->x
```

---

## 30/01/23

## Препроцессор

Базовое

Препроцессору все равно на синтаксис С/С++, то есть на этом этапе не 
возникнет ошибок даже если код неправильный. ошибки вылезут на этапе 
компиляции.

В препроцессинге всегда есть фаза трансляции — порядок, в котором 
компилятор что-то делает. Всего 9 фаз. Первые 6 — препроцессор. Компиляция 
происходит только начиная с 7 фазы. На 8 происходит раскрытие шаблонов, а 
на 9 — линковка.

```cpp
Run `g++ -xc++ -E 01-preprocessor-not-related-to-cpp.txt`

This file is not a valid C++ code. However, the preprocessor still works:
Phases 1-6: https://en.cppreference.com/w/cpp/language/translation_phases

#include "included-text.txt"
#include "included-text.txt"
```

### Странности препроцессора

**Триграфы**

Препроцессор умеет заменять символы на другие символы, триграфы (до С++17, 
так как раньше на некоторых компьютерах не было []):

??( —> [

?? —> ]

<: :> —> []

??/ —> \

\ —> убрать перевод строки

Комментарии убираются препроцессором!

**Контактенация строковых литералов**

Препроцессор умеет объединять два идущих подряд литерала.

```cpp
std::string s1 = "hello" "wor\
ld";
[[maybe_unused]] std::string s2 = "he\n\t\xFF" R"foo(Hello World)foo";

```

### #define и #ifdef

`#define` определяет макрос, который во что-то раскрывает. SOME_VAR — 
перемеенная препроцессора

```cpp
#define SOME_VAR
```

Можно также передать переменную на этапе компиляции:

```cpp
// CMakeLists.txt: target_compile_definitions(main PUBLIC -DSOME_VAR)
// g++ -DSOME_VAR
```

Теперь с помощью этих перменных и `#if` `#ifdef` можно контролировать 
поведение программы

```cpp
#define VALUE 123

#ifdef SOME_VAR
std::cout << "SOME_VAR was defined\n";
#else
std::cout << "SOME_VAR was not defined\n";
#endif

std::cout << VALUE << "\n";
// std::cout << SOME_VAR << "\n";  // SOME_VAR is replaced with nothing

#if defined(SOME_VAR) && 1 >= 2 && VALUE == 120 + 3
std::cout << "x\n";
#endif
```

Также есть предопределенные макросы (`__cplusplus` — номер стандарта):

```cpp
// There are "predefined macros", see https://stackoverflow.com/a/2224357
#ifdef __cplusplus
std::cout << "We are C++\n";
#else
std::cout << "We are not C++???\n";  // TODO: trigraphs?
// Preprocessor warning/error commands: 
https://en.cppreference.com/w/cpp/preprocessor/error
// #error Compiling with C is not supported  // Supported
// #warning Compiling with C is badly supported  // Supported, official 
standard since C++23
#endif
```

### **Пример применения макросов:**

1) Определение версии:

```cpp
std::cout << "Detecting compiler... ";
#if defined(__clang__)
    std::cout << "LLVM/Apple clang " << __clang_version__ << "\n";
#elif defined(__INTEL_LLVM_COMPILER)
    std::cout << "Intel clang-based compiler: " << __INTEL_LLVM_COMPILER 
<< " " << __VERSION__ << "\n";
#elif defined(__INTEL_COMPILER)
    std::cout << "Intel C++ Compiler Classic: " << __INTEL_COMPILER << " " 
<< __VERSION__ << "\n";
#elif defined(__GNUC__)
    std::cout << "GNU C " << __GNUC__ << "." << __GNUC_MINOR__ << "." << 
__GNUC_PATCHLEVEL__ << "\n";
#endif
#ifdef _MSC_VER
    std::cout << "Microsoft Visual C++ " << _MSC_VER << "\n";
#endif
```

2) Можно заглушать предупреждения на определенных компиляторах.

`#pragma` — команда компилятору.

```cpp
int main() {
#ifdef __GNUC__  // Everything is very compiler-specific, see doctest.h 
for an example
    // `#pragma` is a compiler command, not preprocessor
    #pragma GCC diagnostic push  // save current warnings state to a stack
    #pragma GCC diagnostic ignored "-Wunused-variable"
#endif
    /* [[maybe_unused]] */ int x = 10;
#ifdef __GNUC__
    #pragma GCC diagnostic pop  // restore warnings state
#endif
}
```

### Особенность макроса

Препроцессору плевать на синтаксис!

```cpp
#define THREE 1 + 2
#define ZERO 1 - 1

std::cout << THREE * 3 << "\n"; // 7

#if ZERO
std::cout << "Zero is the truth\n"; // true
#else
std::cout << "Zero is the lie\n";
#endif
```

### UB

#define любого ключевого слова С++ — UB.

```cpp
// #define private public  // UB
// #include <vector>

// #define true false // UB

// #define int long long  // UB

// Define of any keyword is UB

// https://github.com/menahishayan/rickroll.h  // Probably not UB, but 
macros are weird. Condolenses to IDEs.

#define __FOO__  // UB, double underscore
#define _Abotva  // UB, starts with underscore
```

## Макро-функции

### Проблемы функций

Способ, использовавшийся в С, для написания простых функций. Очень легко 
огрести.

```cpp
#include <iostream>

#define max(a, b) a < b ? a : b
#define mul(a, b) a * b

int foo() {
    std::cout << "foo() evaluated\n";
    return 10;
}

int main() {
    std::cout << max(2, 1) << "\n";  // No brackets => ((std::cout << 2) < 
1) ? 2 : 1 << "\n";
		// need --> (a < b ? a : b)
    std::cout << mul(2 + 1, 2) << "\n";  // No brackets => 2 + 1 * 2
		// need ((a) < (b) ? (a) : (b))

    int x = 10;
    std::cout << max(x++, 12) << "\n";  // Macro => double change of 'x' 
=> UB.
    std::cout << max(foo(), 12) << "\n";  // Macro => double evaluation of 
foo().
}
```

```cpp
#define print_twice(x) std::cout << x; std::cout << x;

int main() {
    print_twice(500);
    if (2 * 2 == 5)
        print_twice(123); // 123 printed once !
}
```

—> решение

```cpp
// 
https://stackoverflow.com/questions/923822/whats-the-use-of-do-while0-when-we-define-a-macro
#define print_twice(x) do { std::cout << x; std::cout << x; } while (0)
```

### Assert

`#expr` —> превратилось в строковой литерал.

`__FILE__` —> предопределенный макрос имя файла.

`__LINE__` —> предопределенный макрос номер строки вызова.

```cpp
#include <cassert>
#include <iostream>

void my_assert_1(bool expr) {
    if (!expr) {
        std::cout << "Assertion failed: ???????\n";
    };
}

#define my_assert_2(expr) \
    do { \
        if (!(expr)) { \
            std::cout << "Assertion failed: " #expr " at " __FILE__ ":" << 
__LINE__ << "\n"; \
        } \
    } while(0)

int main() {
    my_assert_1(2 * 2 == 5);
    my_assert_2(2 * 2 == 5);
    assert(2 * 2 == 5);
}
```

### Склейка в макросах

`##` позволяют взять два токена и склеить их.

```cpp
#define FOO(a, b) st##a##b
#define NEW_VAR(id) int x##id

int main() {
    FOO(d::c, out) << "Hello World\n";
    NEW_VAR(123) = 10;
    std::cout << x123 << "\n";
}
```

## Трюки макросов

### Вариадические функции

Функции, принимающие что угодно. Но, опять же, аккуратно.

```cpp
// Compile with `g++ -E` to see preprocessed output.

// variadic macro
#define foo(a, b, ...) bar(b, a, __VA_ARGS__)

int main() {
    foo(1, 2, 3, 4);
    foo(1, 2, 3);

    // Oopses:
    foo({1, 2, 3, 4});
    foo(({1, 2, 3, 4}), 5, 6);
    foo(1, 2);  // Can be fixed with C++20's __VA_OPT__ or GCC's extension 
of ##.
    foo(std::map<int, int>(), std::map<int, int>());
    foo((std::map<int, int>()), (std::map<int, int>()));

    // TODO: Compilation error
    foo((std::map<int, int>()));
}
```

### Реккурсия

Нет реккурсии. Можно проэмулировать чернез жесткие костыли (Boost::PP).

```cpp
#include <iostream>

#define FOO(x) if (x > 0) { std::cout << x; FOO(x - 1) }

// Extra reading: Boost::PP (preprocessor).

int main() {
    FOO(5);
}
```

### Макросы, раскрывающиеся в другие макросы

```cpp
#include <iostream>

enum E { Option1, Option2, Option3 };
#define LIST_E(f) f(Option1) f(Option2) f(Option3)

int main() {
    E e = Option2;
    switch (e) {
    case Option1: std::cout << "Option1\n"; break;
    case Option2: std::cout << "Option2\n"; break;
    case Option3: std::cout << "Option3\n"; break;
    }

    switch (e) {
    #define PRINT_OPTION(e) case e: std::cout << #e "\n"; break;
    LIST_E(PRINT_OPTION)
    #undef PRINT_OPTION  // undefine, remove macro
    }
}
```

### Ограничение выполняемости кода

Выполнение только в режиме DEBUG.

```cpp
#include <iostream>

#ifdef DEBUG
#define debug_only
#else
#define debug_only if (false)
#endif

int main() {
    std::cout << "1\n";
    debug_only {
        std::cout << "2\n";
        std::cout << "3\n";
    }
    std::cout << "4\n";
}
```

### Пример из логгирования

```cpp
#include <iostream>

int foo() {
    std::cout << "Works a lot!\n";
    return 10;
}

bool logging_enabled = true;

#define LOG !logging_enabled ? std::cout : std::cout

int main() {
    if (logging_enabled) {
        std::cout << foo() << "\n";
    }
    LOG << foo() << "\n";
}
// TODO: show why not func
```

Почитать про макросы —> 
[http://jhnet.co.uk/articles/cpp_magic](http://jhnet.co.uk/articles/cpp_magic)

---
## ***06/02/23***
## Идеология обработки ошибок
http://joeduffyblog.com/2016/02/07/the-error-model/

John Lakos "Defensive Programming Done Right": https://www.youtube.com/watch?v=1QhtXRMp3Hg https://www.youtube.com/watch?v=tz2khnjnUx8

think-cell's error reporting: https://youtu.be/Cmud1jO__VA?t=2318

Любая обработка ошибок — это часть алгоритма, контракта и инвариантов программы! Нет никакого способа добавить 
обработку ошибок в общем случае, все инструменты языка (те же исключения) — это лишь инструменты. Если обработка 
ошибок важна, её тоже надо продумывать и тестировать. Нельзя просто "сообщить об ошибке в момент возникновения и 
обработать где удобно"; это так же неточно, как "решить задачу циклами и ифами".

### Ошибки программирования

Любой UB, нарушение инварианта (двоичный поиск, состояние класса), нарушения контракта.

```c++
void foo(const char *s) {
    printf("%s\n", s);  // Некорректный s.
}
```

Лучше выявлять на стадии тестирования.

Обычно видны не сразу, а через какое-то время.

Можно пробовать проверять и детектировать (например, vector::at вместо vector::operator[]), но не задетектирует всё 
(потому что баги) и неясно, что делать после обнаружения (мало в чём можно быть уверенным).

Если случилось, то никаких гарантий, all bets are off. Баг в программе точно есть, причём неизвестный, могло упасть где угодно.
На самом деле какие-то гарантии могут быть. John Lakos называет нарушения контракта "soft undefined behavior" (мягкое неопределённое поведение):
всё ещё можно логически вывести поведение программы. Но оно легко приводит к "hard undefined behavior" (жёсткое неопределённое поведение): тут уже вообще ничего сделать нельзя.

Принцип кувалды: https://thephd.dev/c-undefined-behavior-and-the-sledgehammer-guideline
### Ошибки окружения/предсказуемые ошибки
Некорректный ввод пользователя, файл не найден, оборвалось сетевое соединение...

- Могут произойти независимо от программы.

- Можно предсказать при написании кода.

- Лучше обрабатывать.

- Например, клиент Telegram должен быстро работать, даже если сервер внезапно отвалился. 
Переподключиться? Держать ещё одно соединение в запасе? Надо думать.

- Не хватило прав доступа — может, наоборот, не сообщать пользователю, зато сообщить администратору.
  Бывают пограничные ситуации в разных контекстах.

### Надо думать 
На хороших контестах гарантируется, что ввод корректен. В сетевых приложениях нас могут хотеть взломать.

- А что если кончился стэк/память/место на диске?
    - Место на диске: можно забить, можно почистить свой собственный кэш. Это чаще не обрабатывают почему-то.
    - Память: а если при обработке потребуется ещё память?
    - Не удалась запись в файл: могут быть разные требования к чат-клиенту (который может всё скачать с сервера) и самому серверу (которому надо как-то восстанавливаться и централизованно сообщать "жёсткий диск сломался").
- Нарушен инвариант структуры — проблема. А вот если мы её перед этим считывали по сети, то, может, это попытка взлома и надо залогировать и пропустить запрос. Мораль: отделяем внешний ввод от внутренних данных программы.

Про каждую ошибку в программе надо думать:

1) Что случится, когда произойдёт? (не "если")
2) Можно ли с ней сделать что-то разумное с точки зрения _пользователя_?
3) Если да, то как эта обработка ошибки встраивается в алгоритм и архитектуру?

### Обработка ошибок
Почти всегда можно залогировать ошибку для разработчика.

1) Уронить процесс целиком. Самое безопасное при ошибках программирования. Хотя тоже может нарушить инварианты (например, файл не до конца записали). Можно ещё перед этим позвонить домой и отправить багрепорт. А потом немедленно упасть, даже если кажется, что всё хорошо.
2) Уронить кусок процесса. Только если не ошибка программирования (хотя в Java/Python с изоляцией получше). Например, мы сервер и порвалось соединение с клиентом.
3) Как-то обработать: например, попробовать другой адрес для подключения к серверу.
4) Сообщить пользователю и попросить что-нибудь сделать: "файл не найден, выберите другой".
    - Важно понимать, что пользователь далеко не всегда что-то может сделать. О каких ошибках и как сообщать пользователю — отдельный вопрос дизайна user experience.

Проще и надёжнее всего уронить программу целиком.

Нет серебрянной пули для обработки ошибок: это такая же часть спецификации программы, над которой надо хорошо думать заранее и отдельно реализовывать.

### Подавление ошибок 

Популярная раньше идея (wide contracts):

```c++
int max(const std::vector<int> &v) {
    if (v.empty()) return -1;
    .....
}
```

Чем плохо:

1) Если что-то пошло не так (вектор пуст), программа продолжает выполнение с каким-то значением. Может быть hard undefined behavior дальше.
2) Ошибки дальше может и не случиться => сложнее найти баг.

Чем хорошо: _иллюзия_, что всё хорошо.

Даже если включить в контракт, то пользователи его не прочитают и будут не ожидать.

Как обычно делают:
```c++
int max(const std::vector<int> &v) {
    assert(!v.empty());
    _ASSERT(!v.empty());  // think-cell
    BSLS_ASSERT(!v.empty());  // Bloomberg
    .....
}
```
## Механизмы обработки ошибо
###Флаги
У объекта можно ставить какой-нибудь флаг, описывающий его состояние, которое можно регулярно проверять.

Пример: чтение из файла с помощью класса `ifstream`. Если файл не открылся (что нужно проверить самостоятельно
ручками), то можно что-то сделать (в библиотеке не прописано, что именно, просто `f.is_open() = false`). Ошибки 
записываются в `f.rdstate()` (хранит не последнюю ошибку, а состояние файла в целом) :
1) `std::ios_base::badbit` – невосстановимая ошибка, например, жесткий диск отвалился.
2) `std::ios_base::failbit` – восстановимая ошибка, например, формат нарушился и считали число вместо строки.
3) `std::ios_base::eofbit` – верно ли, что наткнулись на `EOF`.

Если у файла установлен хотя бы один такой бит, то он считается некоррректным (`!f` проверяет, установлен ли хотя какой-то бит).

```c++
#include <fstream>

void check_file(const std::string &filename) {
    std::ifstream f(filename);
    std::cout << "Reading from " << filename << "\n";
    // Easy to forget to check.
    std::cout << "    is_open: " << f.is_open() << "\n";
    for (;;) {
        // https://en.cppreference.com/w/cpp/io/ios_base/iostate
        // State, not result of operation
        std::cout << "    status: " << f.rdstate() << "; "
                  << std::ios_base::badbit << " "  // irrecoverable error
                  << std::ios_base::failbit << " "  // format/extract error
                  << std::ios_base::eofbit << "\n";  // EOF reached
        if (!f) {
            break;
        }
        int x = -239;
        // Careful handling is actually required: https://en.cppreference.com/w/cpp/named_req/FormattedInputFunction
        f >> x;
        std::cout << "    x = " << x;
    }
}
```

### Глобальные переменные с ошибкой
После открытия файла в `errno` лежит код ш=ошибки (последней ошибки, если открыть другой файл, то он поменяется).

```c++
#include <cerrno>

void check_file(const std::string &filename) {
    FILE *f = std::fopen(filename.c_str(), "r");
    std::cout << "Reading from " << filename << "\n";
    std::cout << "    f: " << f << ", errno: " << errno << " " << std::strerror(errno) << "\n";  // POSIX (not C/C++) requires that errno macro is set.
    // Problem: have to check immediately after each function. Does not propagate up.
    if (f == nullptr) {
        return;
    }
    for (;;) {
        int x = -239;
        int read = std::fscanf(f, "%d", &x);  // Still easy to forget to check.
        std::cout << "    x = " << x << ", read = " << read << ", errno = " << errno << " " << std::strerror(errno) << "\n";
        if (read != 1) {
            break;
        }
    }
    std::fclose(f);
}

int main() {
    check_file("numbers-01-simple.in");
    check_file("numbers-02-immediate-eof.in");
    check_file("does-not-exist.in");
}
```

Еще пример: С-шная библиотека `<sqlite3.h>`.  Функция `sqlite3_open` принимает ссылку на базу данных и возвращает `int`,
если удалось/не удалось открыть. Можно потом сравнивать с глобальными перменными, например, `SQLITE_OK` или `SQLITE_CANTOPEN`. 
```c++
#include <sqlite3.h>
#include <iostream>

// https://www.sqlite.org/c3ref/open.html
// int sqlite3_open(
//   const char *filename,   /* Database filename (UTF-8) */
//   sqlite3 **ppDb          /* OUT: SQLite db handle */
// );

int main() {
    sqlite3 *db;
    if (int err = sqlite3_open("does-not-exist/some-db.sqlite3", &db); err != SQLITE_OK) {  // Still easy to forget to check, but syntax is ugly.
        std::cout << "err = " << err << " " << sqlite3_errstr(err) << "\n";
        if (err == SQLITE_CANTOPEN) {
            std::cout << "Unable to open DB file\n";
        }
        return 1;
    }
    sqlite3_close(db);
    return 0;
}
```

### Пример из языка rust
Функция `File::open` возвращает или файл, или ошибку, а дальше можно будет это проверить (популярно в функциональном программировании).
```c++
// https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html
use std::fs::File;
use std::io::Read;

fn main() {
    let mut f = match File::open("hello.txt") {  // We have to check.
        Ok(file) => file,
        Err(error) => {
            println!("Problem opening the file: {:?}", error);
            return
        },
    };
    let mut s = String::new();
    match f.read_to_string(&mut s) {
        Ok(_) => println!("File read: {:?}", s),
        Err(e) => {
            println!("Problem reading the file: {:?}", e);
            return
        }
    }
}
```

Даже добавили оператор `?`, заменяющий конструкцию `if failure, return Err()`. То есть если функция может выбросить ошибку, то она пробрасывается
на уровень нашей функции. По сути те же коды возврата, но проверка сводится к одному оператору `?`.
```c++
// https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html
use std::fs::File;
use std::io::Error;
use std::io::Read;

fn main() -> Result<(), Error> {
    let mut f = File::open("hello.txt")?;  // '?' is a shortcut for "if failure, return Err()".
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    println!("File read: {:?}", s);
    Ok(())
}
```

### Не обрабатыватьошибкаи :)
...просто assert понатыкать можно.

```c++
#include <cassert>
#include <cstdlib>
#include <iostream>
#include <fstream>
#include <vector>
#include <string>

std::vector<int> read_vector(const std::string &filename) {
    std::ifstream f(filename);
    assert(f);  // Why no handling?
    int n;
    f >> n;
    assert(f);
    std::vector<int> vec(n);
    for (int &v : vec) {
        f >> v;
        assert(f);
    }
    return vec;
}
```

### Обрабатывать на месте
Сразу при возникновении обрабатывать возникшую ошибку. С одной стороны, эта концепция позволяет достаточно точно обработать ошибку, а с другой – иногда
она роняет программу.
```c++
#include <cassert>
#include <cstdlib>
#include <iostream>
#include <fstream>
#include <vector>
#include <string>

std::vector<int> read_vector(const std::string &filename) {
    std::ifstream f(filename);
    assert(f);  // Why no handling?
    int n;
    f >> n;
    assert(f);
    std::vector<int> vec(n);
    for (int &v : vec) {
        f >> v;
        assert(f);
    }
    return vec;
}
```

### Пробрасывание ошибки наверх
Самая популярная концепция. Функция `read_vector` возвращает код ошибки, что пошло не так. Да, функция стала длинее раза в 3, но безопаснее,
можно будет обработать возникшие ошибки.

Заметим, что считанные вектор не сразу записыается в `result`. То есть при возникновении ошибки он не поменяется и туда не запишется что-то частичное.
```c++
enum class read_vector_error { OK, CANNOT_OPEN, CANNOT_READ_LENGTH, CANNOT_READ_NUMBER };

/*[[nodiscard]]*/ read_vector_error read_vector(const std::string &filename, std::vector<int> &result) {
    std::ifstream f(filename);
    if (!f) {
        return read_vector_error::CANNOT_OPEN;
    }
    
    int n;
    if (!(f >> n)) {
        return read_vector_error::CANNOT_READ_LENGTH;
    }
    
    std::vector<int> vec(n);  // WARNING: TODO
    for (int &v : vec) {
        if (!(f >> v)) {
            return read_vector_error::CANNOT_READ_NUMBER;
        }
    }
    result = std::move(vec);
    return read_vector_error::OK;
}
```
## Исключения
Исключения создавались для исклбчительных ситуаций. А дальше – дисскусия лет на 40, какая ситуация является исключительной :)
### Базовое
Если блок `try` успешно отработает, то `catch` опустится. Если возникает ошибка типа `invalid_vector_format`, то зайдем в блок `catch`.
```c++
try {
    solve();  // if exception is thrown, it's propagated
} catch (const invalid_vector_format &err) {  // looking for the first catch
    std::cout << "Invalid vector format\n";
    // Print good error message is hard:
    // 1. How exactly the format is invalid? We could add it to the exception.
    // 2. We don't have much context here: what exactly failed? A or B?
    // 3. We don't have that context in read_vector() either.
    // Consequence: this is probably not a great error handling.
    //              Not because of exceptions per se, but because we did not think _in advance_.
}
```

Если во время считывания что-то пошло не так, то вы `throw` ошибку. `throw` ожидание како-нибудь значение любого типа. Чаще всего 
пишут какие-нибудь конструкторы.

Причем при выбросе исключения локальные перменные удаляется, никаких утечек.

`throw` в `read_vector` —> перебрасывается в `solve`, там никак не обработать —> летит дальше и ловится в `main`. 

То есть не проверяя ошибку в solve, мы вытащили ошибку наверх. СОбственно, для этого и нужны исключения – вытащить ошибку с более глубоких уровней
так, чтобы она не затронула промежуточные.
```c++
struct invalid_vector_format {};

std::vector<int> read_vector() {
    int n;
    if (!(std::cin >> n)) {
        throw invalid_vector_format();
    }
    std::vector<int> result(n);
    for (int i = 0; i < n; i++) {
        if (!(std::cin >> result[i])) {
            throw invalid_vector_format();
        }
    }
    return result;
}

void write_answer(const std::vector<int> &vec) {
for (std::size_t i = 0; i < vec.size(); i++) {
if (i) std::cout << " ";
std::cout << vec[i];
}
std::cout << "\n";
}

void solve() {
std::vector<int> a = read_vector();  // if exception is thrown, it's propagated
std::vector<int> b = read_vector();  // if exception is thrown, it's propagated; local variables are cleaned up (stack unwind, раскрутка стека).

// Тут какие-то вычисления
std::vector<int> answer = a;
answer.insert(answer.end(), b.begin(), b.end());

write_answer(answer);
}
```

При этом наше сообщение об ошибки не очень: как именно вектор invalid format? Какой вектор сломался? A или B? 

Можно добавить еще один параметр, имя вектора, но это усложение кода для обработки ошибок. 

### Про утечки и деструкторы
Локальные переменные при возникновении ошибки уничтожаться, деструктор вызовется при любом завершении функции/скобок 
(ошибка или удачное завершение). Все будет хорошо, никаких утечек. Называется stack unwind (раскрутка стека). 

`catch(...)` – поймай любое исключение.
```c++
#include <iostream>

struct err1 {};
struct err2 {};

struct with_destructor {
    int key;
    with_destructor(int key_) : key(key_) {}
    ~with_destructor() {
        std::cerr << "Destructed " << key << "\n";
    }
};

void bar() {
    with_destructor wd31(31);  // Implementation-defined stack unwind.
    try {
        with_destructor wd40(40);
        std::cerr << "3\n";
        throw err1();
        std::cerr << "x3\n";
    } catch (const err1 &e) {
        std::cerr << "4\n";
        throw err2();
        std::cerr << "x4\n";
    }
    std::cerr << "bar end\n";
}

void foo() {
    with_destructor wd20(20);  // Implementation-defined stack unwind.
    try {
        with_destructor wd30(30);  // Implementation-defined stack unwind.
        std::cerr << "2\n";
        bar();
        std::cerr << "x2\n";
    } catch (const err1 &e) {
        std::cerr << "z\n";
    }
    std::cerr << "foo end\n";
}

int main() {
    std::cerr << "1\n";
    {
        with_destructor wd10(10);
    }
    try {
        foo();
    } catch (...) {
        std::cerr << "caught!\n";
    }
std::cerr << "main end\n";
}
```
### Несколько catch
Можно писать несколько `catch`. При этом UB и `assert` не поймаются.
```c++
try {
    // throw err1();
    // throw err2();
    std::vector<int> vec(-1);
} catch (const err1 &) {
    std::cout << "1\n";
} catch (const err2 &) {
    std::cout << "2\n";
} catch (...) {  // Exceptions only. Not UB, not assertions!
    std::cout << "3\n";
}
```

### Наследование исключений
Можно сделать базовый класс и несколько наследников. Тогда можно будет ловить исключения по ссылке на базовый класс.

Если в первом `try`-`catch` ловить ошибку по значению, то произойдет слайсинг и поймается `Base`, а не `Derived` (грубо говоря, Derived обрежется до Base).

При этом если структуру `Magic` можно преобразовать к `const Base&`, то при выкидывании исключения такого типа во втором блоке 
оно не поймается – такие исключения работают только с наследованием, а не с преобразованиями.

Благодаря этому можно делать целую иерархию ошибок. 
```c++
#include <iostream>

struct Base {
    virtual const char *who() const {
        return "Base";
    }
};

struct Derived : Base {
    const char *who() const override {
        return "Derived";
    }
};

struct Magic {
    operator Base() { return {}; }
};

int main() {
    try {
        throw Derived();
    } catch (const Base &e) {
    //    } catch (Base e) {
    std::cout << "Caught: " << e.who() << "\n";
}

    try {
        [[maybe_unused]] const Base &b = Magic();
        throw Magic();
    } catch (const Base &e) {  // Does not work, needs ~type match or inheritance.
        std::cout << "Caught Base\n";
    }
}
```

### Вложение try-блоки
Острожно - `catch` относится к своему блоку `try`, а не к ближацшему сверху `try`! Можно перебрасывать наверх, если нужно.
```c++
#include <iostream>

struct err1 {};
struct err2 {};

void bar() {
    try {
        try {
            std::cerr << "3\n";
            throw err1();
            std::cerr << "x3\n";
        } catch (const err1 &e) {
            std::cerr << "4\n";
            throw err2();
            std::cerr << "x4\n";
        } catch (const err2 &e) {
            std::cerr << "x5\n";
        }
    } catch (int e) {
        std::cerr << "caught int???\n";
    } catch (const err2 &e) {
        std::cerr << "caught err2!\n";
        throw e;
    }
    std::cerr << "bar end\n";
}

void foo() {
    try {
        try {
            std::cerr << "2\n";
            bar();  // std::cerr << "3\n4\n"; throw err2();
            std::cerr << "x2\n";
        } catch (const err1 &e) {
            std::cerr << "z\n";
        }
        std::cerr << "x21\n";
    } catch (int e) {
        std::cerr << "x22\n";
    }
    std::cerr << "foo end\n";
}

int main() {
    try {
        std::cerr << "1\n";
        foo();
        std::cerr << "xxx\n";
    } catch (const err2 &) {
        std::cerr << "5\n";
    }
    std::cerr << "main continues\n";
}
```

### Непойманное исключение
Implementation-defined stack unwind – если исключение не поймано, то зависит от компилятора. Он может раскрутить стек,
а может и не раскручивать. Может уронить программу и не вызвать деструкторы.
```c++
#include <iostream>

struct err1 {};
struct err2 {};

struct with_destructor {
    int key;
    with_destructor(int key_) : key(key_) {}
    ~with_destructor() {
        std::cout << "Destructed " << key << "\n";
    }
};

void bar() {
    with_destructor wd31(31);  // Implementation-defined stack unwind.
    try {
        with_destructor wd40(40);
        std::cout << "3\n";
        throw err1();
        std::cout << "x3\n";
    } catch (const err1 &e) {
        std::cout << "4\n";
        throw err2();
        std::cout << "x4\n";
    }
    std::cout << "bar end\n";
}

void foo() {
    with_destructor wd20(20);  // Implementation-defined stack unwind.
    try {
        with_destructor wd30(30);  // Implementation-defined stack unwind.
        std::cout << "2\n";
        bar();
        std::cout << "x2\n";
    } catch (const err1 &e) {
        std::cout << "z\n";
    }
    std::cout << "foo end\n";
}

int main() {
    std::cout << "1\n";
    {
        with_destructor wd10(10);
    }
    try {
        foo();
    } catch (...) {
//        throw;
    }
    std::cout << "main end\n";
}
```

### Исключения стандартной библиотеки
В STL есть целая иерархия исключений: `std::runtime_error`, `std::logic_error` или не относятся ни к чему. В свою очередь они наследуются от некоторого
базового типа `std::exception`, у которого есть метод what, соедержащий информацию об ошибке.
```c++
#include <boost/core/demangle.hpp>
#include <exception>
#include <iostream>
#include <vector>
#include <string>

struct Base { virtual ~Base() = default; };
struct Derived1 : Base {};
struct Derived2 : Base {};

int main() {
    try {
        // Some are std::runtime_error, some are std::logic_error, some are neither.
        // std::vector<int> vec(-1);  // std::bad_alloc
        /*{
            Derived1 d1;
            Base &b = d1;
            [[maybe_unused]] Derived2 *d2a = dynamic_cast<Derived2*>(&b);  // nullptr
            std::cout << d2a << "\n";
            [[maybe_unused]] Derived2 &d2b = dynamic_cast<Derived2&>(b);  // std::bad_cast
        }*/
        // std::cout << std::stoi("123") << "\n";
        // std::cout << std::stoi("foo") << "\n";  // std::invalid_argument
        // std::cout << std::stoi("123foo") << "\n";  // no exceptions, ok
        // std::cout << std::vector<int>(10).at(15) << "\n";  // std::out_of_range
        // See Boost::Dll example with 'file not found'
    } catch (const std::exception &e) {
        std::cout << "Error type=" << boost::core::demangle(typeid(e).name()) << " what()=" << e.what() << "\n";
    }
}
```

### Наследование от `std::exception`
Свои исключения можно наследовать от стандартных. Можно напрямую от `std::exception`. Но лучше наследоваться от 
'std::runtime_error' или `std::logic_error` (в отличие от `std::exception` у них есть конструктор не от `char*`, а от `std::string`).

Потом можно будет ловить по ссылке на `std::exception`.
```c++
#include <exception>
#include <stdexcept>
#include <iostream>

struct my_exception : std::exception {
    const char *what() const noexcept override {
        return "my_exception";
    }
};

struct my_runtime_error : std::runtime_error {  // Choice between runtime_error and logic_error is not strict.
    my_runtime_error() : std::runtime_error("yay! " + std::to_string(123)) {}
};

// TODO: guess what is wrong with `class my_runtime_error : std::runtime_error`

int main() {
    try {
        throw my_exception();
    } catch (const std::exception &e) {
        std::cout << e.what() << "\n";
    }

    try {
        throw my_runtime_error();
    } catch (const std::exception &e) {
        std::cout << e.what() << "\n";
    }

    try {
        throw my_runtime_error();
    } catch (const std::runtime_error &e) {
        std::cout << e.what() << "\n";
    }
}
```

### Summary

Do not blindly use exceptions. Think about error handling first. Use exceptions only when it's easier than error codes. Especially do not use exceptions with programming errors. You can always easily go from error codes to exceptions, vice-versa is harder. Exceptions should be part of a contract.
---
## ***13/02/23***

## Про классы
### public, private, protected inheritance
Также, как и с полями – `private`, `protected` и `public`. Для классов наследование по умолчанию `private`, 
у структур – `public`. Влияет на то, кто понимает, что класс отнаследован от базового класса.
```cpp
struct Base {
    void foo() {
    }
};

struct Derived1 : public Base {};  // Default for 'struct', the most popular.
struct Derived2 : protected Base {};
struct Derived3 : private Base {  // Default for 'class'.
    void baz() {
        foo();
        Derived3 d3;
        [[maybe_unused]] const Base &b = d3;
    }
};
struct Derived22 : /* public */ Derived2 {
    void bar() {
        foo();
        Derived2 d2;
        [[maybe_unused]] const Base &b2 = d2;
    }
};

struct Derived33 : /* public */ Derived3 {
    void bar() {
    // foo();
    [[maybe_unused]] ::Base b;  // :: is important
    // [[maybe_unused]] const ::Base &b2 = *this;
    // [[maybe_unused]] const ::Base &b3 = static_cast<const ::Base &>(*this);
    [[maybe_unused]] const ::Base &b4 =
        (const ::Base
            &)*this;  // meh, C-style cast ignores access modifiers
    }
};
```
`private` – Derived3 знает, что он отнаследован от Base.

Derived33 не знает, что Derived3 наследован от Base --> нельзя вызывать методы Base, создавать поле
такого типа (лайфхак: ::Base сработает (не из-за наследования, а просто поле)), нельзя взять ссылку на себя как на Base.

`protected` – Derived2 знает, что он отнаследован от Base. Наследники не могут вызывать методы Base.
```c++
int main() {
    Base b;
    Derived1 d1;
    Derived2 d2;
    Derived3 d3;

    b.foo();
    d1.foo();
    // d2.foo();
    // d3.foo();

    [[maybe_unused]] const Base &b1 = d1;

    // [[maybe_unused]] const Base &b2x = d2;
    // [[maybe_unused]] const Base &b2y = static_cast<const Base &>(d2);
    [[maybe_unused]] const Base &b2z =
        (const Base &)d2;  // meh, C-style cast ignores access modifiers

    // [[maybe_unused]] const Base &b3x = d3;
    // [[maybe_unused]] const Base &b3y = static_cast<const Base &>(d3);
    [[maybe_unused]] const Base &b3z =
        (const Base &)d3;  // meh, C-style cast ignores access modifiers
}
```

! важно помнить, что у классов наследование приватное по умолчанию.

### Пример приватного наследования
Из boost:
```c++
struct noncopyable {  // boost::noncopyable
    noncopyable() = default;
    noncopyable(const noncopyable &) = delete;
    noncopyable(noncopyable &&) = delete;
    noncopyable &operator=(const noncopyable &) = delete;
    noncopyable &operator=(noncopyable &&) = delete;
};

struct Foo : private noncopyable {};
struct Bar : private noncopyable {};
struct Baz : private noncopyable {};

int main() {
    Foo f;
    // noncopyable &n = f;  // WTF, 'private' prevents that.

    // Foo f2 = f;
}
```

### Slicing
Есть класс и есть наследник.
```c++
struct Base {
    int x = 10;
    void foo() const {
        std::cout << "x=" << x << "\n";
    }
};

struct Derived : Base {
    int y = 20;
    void bar() const {
        foo();
        std::cout << "y=" << y << "\n";
    }
};

void bar(Base b) {  // Slicing: we create a new object
    std::cout << "bar(" << b.x << ")\n";
    const Derived &d = static_cast<const Derived&>(b);  // derivedcast, UB
    std::cout << ".y=" << d.y << "\n";  // UB
    &d.y;  // UB

```
Вызываем функцию, принимающую по значению, а не по ссылке. Тогда из Derived создастся объект типа Base и при попытке сделать
derivedcast возникнет UB.
```c++
{
    Derived d;
    d.x = 123;
    // Base(const Base &other) : x(other.x)
    bar(d);  // Always UB.
}
{
    Base b;
    bar(b);  // Always UB.
}
```

### Защита от slicing'а
Запретить в базовом классе копирование и перемещение. Тогда код не сломается (чаще всего), а при попытке принять Derived по значению
будет ошибка компиляции (нет копирования).
```c++
struct Base {
    int x = 10;
    void foo() const {
        std::cout << "x=" << x << "\n";
    }

    Base() {}
    Base(const Base &) = delete;
    Base(Base &&) = delete;
    Base &operator=(const Base &) = delete;
    Base &operator=(Base &&) = delete;
};
```
### Делегирующий конструктор
Конструктор вызывает собственный конструктор типа, а потом может еще что-то доделать, если надо.

```c++
bigint(const std::string &s) {
    // TODO: ...
    std::cout << "constructing from string " << s << "\n";
}
bigint(int x) : bigint(std::to_string(x)) {  // Delegating constructor
    // bigint(std::to_string(x));  // bad attempt :(
    std::cout << "constructing from int " << x << "\n";
}
bigint() : bigint(0) {}
```

## Странный синтаксис исключения
### rethrow
Модификация 1: наследуемся от , чтобы лучше работать со страндартными библиотеками + была бы возможность ловить в main 
какой-то exception.
```c++
struct invalid_vector_format : std::runtime_error {
    invalid_vector_format() : std::runtime_error("Invalid vector format") {}
};

...

int main() {
    try {
        solve();
    } catch (const std::exception &err) {  // !!! Modification 4
        std::cout << err.what() << "\n";
    }
}
```

Модификация 2: сначала ловим ошибку внутри функции, а потом перебрасываем, чтобы она поймалась в main.

throw e; – перебрасываем пойманное исключение, если знаем его тип.
throw; – смотрит, какое исключение поймано, и выбрасывает ровно его. Можно писать не только в catch.
```c++
void rethrow() {
    std::cout << "Rethrowing...\n";
    throw;
}

void solve() {
    std::vector<int> a;
    try {
    a    = read_vector();
    } catch (const invalid_vector_format &e) {
        std::cout << "Caught error while reading vector a, propagating...\n";
        // throw e;  // rethrow exception 'e', but what about other types?
        // throw;  // does the same
        rethrow();  // does the same, but indirectly
    }
    ...
}
```
Модификация 3: `catch(...)` – поймать что-нибудь.
```c++
void solve() {
    ...
    std::vector<int> b;
    try {
        b = read_vector();  
    } catch (...) {
        std::cout << "Caught error while reading vector b, propagating...\n";
        throw;  // works with `...`
    }
        ...
    }
    std::vector<int> answer = a;
    answer.insert(answer.end(), b.begin(), b.end());

    write_answer(answer);
}
```

### throw clicing
`throw a;` –  произойдет slicing (так как прилетает исключение по значению), тип выброшенного исключения будет `std::exception`.
`throw;` – так не потеряется тип сключения, лучше использовать, чтобы не терять динамический тип сключения.
```c++
#include <exception>
#include <iostream>
#include <stdexcept>

void foo() {
    try {
        throw std::runtime_error("Hello World");
    } catch (std::exception &a) {
        // throw a;  // oops, slicing
        throw;  // no slicing
    }
}
int main() {
    try {
        foo();
    } catch (std::exception &e) {
        std::cout << "caught in main(): " << e.what() << "\n";
    }
}
```

## Intermediate exceptions 
### Когда могут возникать исключения
Исключения вылетают только если их кто-то явно бросил (например, bad+alloc при нехватке памяти), при этом UB – это не исключение.
То есть в С++ исключения обрабатывает не все ошибки, а только те, о которых договорились.
```c++
int main() {
    int x = 2 * 2'000'000'000;  // UB
    int y = 2 / 0;  // UB
    assert(false);  // assertion failed, может быть удалён.

    std::vector<int> vec(10);  // Может быть std::bad_alloc, если памяти не хватило.
    vec[10] = 5;  // UB
    vec.at(10) = 5;  // std::out_of_bounds : std::exception, гарантируется.
}
```

### Гарантии икслючений (exception safety)
```c++
void do_something() {
    throw std::bad_alloc();  // Uncomment when running under Valgrind/ASan, it does not like not having enough memory and will close instead of bad_alloc.
    std::vector<int> vec(100'000'000'000);
    vec[vec.size() - 1] = 123;
}
```
Без гарантий: выделили data, при выбросе исключения будет утечка (не почистили data).
```c++
void no_guarantee() {
    std::cout << "2+2=";
    int *data = new int[10];
    do_something();
    delete[] data;
    std::cout << "4\n";
}
```
Базовая гарантия: если исключение произошло, то хотя бы нет UB и утечек памяти.
```c++
// Basic exception safety: no leaks, no UB
void basic_guarantee_1() {
    std::cout << "2+2=\n";
    std::vector<int> data(10);
    do_something();
    std::cout << "4\n";
}

void basic_guarantee_2() {
    std::cout << "2+2=4\n";
    std::vector<int> data(10);
    do_something();
}
```
Строгая (сильная) гарантия исключения или семантика отката назад: если произошло исключение, то в мире ничего не поменялось 
(или функция выполнилась полностью успешно, или выбросилось исключение и в мире ничего не поменялось (например, вывелось  2+2=4 полностью, но тут 
тоже зависит от нашей интерпретации того, что значит, что ничего не поменялось))
```c++
// Strong exception safety (rollback semantics): nothing unchanged
void strong_guarantee() {
    std::vector<int> data(10);
    do_something();
    // What is "nothing"? Can we output?
    std::cout << "2+2=4\n";
}
```
Нет исключений: следовательно, самая строгая гарантия – вообще без исключений)))
```c++
void noexcept_guarantee() {
    assert(std::cout.exceptions() == std::ios_base::goodbit);
    std::cout << "2+2=4\n";
}
```

### noexcept
`noexcept` функция не должна выкидывать исключений, а если и выкидывает, то вызовется `std::terminate` и убьет программу.
При этом никаких деструктором не вызывается.
```c++
void check_n(int n) {
    if (n < 0) {
        throw 0;
    }
}

void foo() noexcept {  // If exception: std::terminate(), no dtor calls
    std::cout << "foo() start\n";
    int n;
    std::cin >> n;
    check_n(n);
    std::cout << "foo() end\n";
}
```
При этом можно ловить и выкидывать исключения внутри, главное, чтобы снаружу не вылетело ничего.
```c++
void check_n(int n) {
    if (n < 0) {
        throw 0;
}
}

void foo() noexcept {  // If exception: std::terminate(), no dtor calls
    std::cout << "foo() start\n";
    int n;
    std::cin >> n;
    try {
        check_n(n);
    } catch (...) {
        std::cout << "Caught inside\n";
    }
    std::cout << "foo() end\n";
}
```
### Пример функции, помеченной noexcept почти всегда
Деструктор может вызываться в процессе обработки исключений. Поэтому если, например, в деструкторе вызвалось 100 исключений, то
С++ просто уронит программу.

Поэтому с С++11 почти всегда неявно дописывается `noexcept`.
```c++
#include <iostream>

void maybe_throw() {
    static int remaining = 2;
    if (!--remaining) { 
        throw 0;
    }
}

struct Foo {
    Foo() {
        std::cout << "Foo() " << this << "\n";
    }
    ~Foo() {
        std::cout << "~Foo() " << this << "\n";
    maybe_throw();
}
};

int main() {
    try {
        try {
            Foo a;
            Foo b;
            Foo c;
            std::cout << "before throw\n";
            // TODO: split in C++03 and C++11
            throw 1;
            std::cout << "after throw\n";
        } catch (int x) {
            std::cout << "caught " << x << "\n";
        } catch (...) {
            std::cout << "caught something\n";
        }
    } catch (...) {
        std::cout << "caught outside\n";
    }
    // C++03: two exceptions simultanously => std::terminate().
// C++11: dtors are noexcept, always std::terminate().
}
```

### Плюсы и минусы noexcept
Pros:
1) noexcept in function signature: better documentation
2) noexcept in function signature: can be checked in compile-time and adjusted for (e.g. algorithmically)
3) May be (or not) be faster: https://stackoverflow.com/questions/16104057/does-noexcept-improve-performance

Cons:
1) noexcept in function signature: compiler does not force anything, but happily terminates the program
2) Impossible to do "negative testing" that the function catches errors as expected: https://bloomberg.github.io/bde-resources/doxygen/bde_api_prod/group__bsls__asserttest.html#4.1

What we do in this course for training:

1) Specify noexcept everywhere where there are no exceptions including bad_alloc.
2) Strive for more noexcept.

### noexcept в структурах
Инициализация по умолчанию `noexcept` (нет никаких операций).

Инициализация с оператором `new` не `noexcept` (может быть `std::bad_alloc`). 

Копирование не `noexcept` (может быть сложная логика с вызовом оператора `new`).

Перемещение почти всегда `noexcept` (почти всегда это просто возня с указателями; обычно если он может быть не noexcept, то скорее 
всего вы делаете что-то не так).

Оператор присваивания копирующий не `noexcept` (может быть с копированием памяти).

Оператор присваивания перемещающий `noexcept` (внутри может быть вызов деструктора, но он noexcept, так что все ок).

Деструктор `noexcept` неявно, так что не помечаем.
```c++
// move constructor/assignment operator can (and should) almost always be noexcept
struct IntWrapper {
    int *data;

    IntWrapper() noexcept : data(nullptr) {
    }

    IntWrapper(int x) : data(new int(x)) {}

    IntWrapper(const IntWrapper &other) : data(other.data ? new int(*other.data) : nullptr) {
    }

    IntWrapper(IntWrapper &&other) noexcept : data(other.data) {
        other.data = nullptr;
    }

    IntWrapper& operator=(const IntWrapper &) {
        return *this;
    }

    IntWrapper& operator=(IntWrapper &&other) noexcept {
        if (!other.data) {
            delete data;  // noexcept
            data = nullptr;
        } else {
            if (data) {
                *data = *other.data;
            } else {
                data = other.data;
                other.data = nullptr;
            }
        }
        return *this;
    }

    ~IntWrapper() {
        delete data;
    }
};
```

### RAII
Resource Acquisition Is Initialization. Захват ресурса есть инициализация.
1) При захвате ресурсов конструктор должен устанавливать инвариант. При невозможности – выбрасывать исключение.
2) Деструктор должен освобождать все ресурсы.

RAII – свойство некоторого класса. Иногда говорят: в коде используются RAII-классы.
```c++
int main() {
    // RAII: Resource Acquisition Is Initialization
    {
        std::vector<int> v(1'000'000);
        // Invariant: v.size() == 1'000'000, memory allocation succeeded.
        v[999'999] = 123;
        // RAII part 1: constructor has to establish invariant and grab resources. Throw an exception if that's impossible. No exit codes.
    }  // RAII part 2: destructor has to free all resource.

    try {
        std::vector<int> v(100'000'000'000);
        // Invariant: memory allocation succeeded.
        v[99'999'999'999] = 123;
    } catch (std::bad_alloc &) {
        // Impossible to access 'v' with incorrect invariant!
        std::cout << "caught bad_alloc\n";
    }
    // Jargon: 'RAII' may only mean part 2: destructor cleans up everything.
    // Constructing object is less strict, see ifstream.
}
```

### Исключения в конструкторе
Нельзя поймать ошибки только в конструкторе.
(можно сделать вектор пустым, а потом resize (то есть разделить ), но это не RAII-чно)
```c++
void may_throw(std::vector<int> &) {
    // maybe throw exception
}

int main() {
    try {
        // try {
        std::vector<int> v(100'000'000'000);
        // } catch
        v[99'999'999'999] = 123;
        may_throw(v);
    } catch (std::bad_alloc &) {
        std::cout << "caught bad_alloc inside v ctor or from may_throw\n";
    }
}
```

Не очень красиво, но:
```c++
int main() {
    try {
        // try {
        std::vector<int> v(100'000'000'000);
        // } catch
        v[99'999'999'999] = 123;
        may_throw(v);
    } catch (std::bad_alloc &) {
        std::cout << "caught bad_alloc inside v ctor or from may_throw\n";
    }
}
```

### Исключения в разных единицах трансляции
В рамках одной программы все более менее хорошо, внутри библиоткек – спорно.
```c++
// a.cpp
#include <exception>
#include <iostream>

void foo();

int main() {
    try {
        foo();
    } catch (std::exception &e) {
        std::cout << "Exception: " << e.what() << "\n";
    }
}

// b.cpp
#include <iostream>

void bar(int i);

void foo() {
    for (int i = 0; i < 5; i++) {
        std::cout << "i=" << i << "\n";
            bar(i);
    }
}

// c.cpp
#include <exception>

struct my_error : std::exception {
    const char *what() const noexcept override {
        return "my_error";
    }
};

void bar(int i) {
    if (i == 3) {
    th  row my_error();
    }
}
```

### Скорость исключений
Теоретически компилятор мог бы соптимизировать функции, зная, что исключения не кидаются.
```c++
#include <chrono>
#include <iostream>

// There was no difference with/without noexcept with this example, but it may happen:
// 1. Say hello to no-noexcept-move and strong exception safety (will be later).
// 2. One can optimize code like `i++; my_noexcept(); i++;`
// 3. There is no need to prepare for stack unwind.
//
// -fno-exceptions did not make a difference for me as well.

bool foobar(int a, int b, bool please_throw) noexcept {
    /* if (please_throw) {
        if (a % b) {
            throw 0;
        }
        return true;
    }*/
    return a % b == 0;
}

const int STEPS = 100'000'000;

int main() {
    auto start = std::chrono::steady_clock::now();
    
    for (int i = 0; i < STEPS; i++) {
        //        try {
        foobar(10, 3, false);
        //        } catch (...) {
        //        }
}

auto duration = std::chrono::steady_clock::now() - start;
std::cout << std::chrono::duration_cast<std::chrono::milliseconds>(duration).count() << "ms\n";
}
```

Исключения тормозят программу примерно в 1000 раз больше, чем просто возврат из функции.
```c++
#include <chrono>
#include <iostream>

// Conclusion: exceptions are for exceptional cases only! Sad path is much, much slower.

bool foobar(int a, int b, bool please_throw) {
//    if (please_throw) {
        if (a % b) {
            //throw 0;  // todo: commenting out makes program faster
            return false;
        }
        return true;
//    }
    return a % b == 0;
}

const int STEPS = 100'000'000; // Note: much fewer steps.

int main() {
    auto start = std::chrono::steady_clock::now();

    for (int i = 0; i < STEPS; i++) {
//        try {
            foobar(12, 3, true);
//        } catch (...) {
//        }
    }

    auto duration = std::chrono::steady_clock::now() - start;
    std::cout << std::chrono::duration_cast<std::chrono::milliseconds>(duration).count() << "ms\n";
}
```

Мораль: исключения заоптимизированы так, что если вы их не кидаете, то вам почти бесплатно, а если кидаете, то обработка
занимает гораздо больше времени.

### Немного замечаний
Some projects disabled exceptions altogether due to:

1. Little to no support in a compiler
2. Slow happy path
3. Hard-to-maintain code

Example: Google.
Enabling exceptions back is hard due to the lack of exception safety. They did not.
Stereotype about hardware engineers: `-fno-exceptions`
---
## 20.02.23
## Потоки 
### promise и future
Задача: есть два потока consumer и producer (один что-то читает, а второй – обрабатывает или один генерирует ответы, а второй – что-то выводит и так далее).

Как перекидывать между ними значения?

Стандартное решение – `std::promise<...>` (то, что будем передавать), `std::future<...>` (то, что получаем).

std::promise – некая очередь длины один, кладем туда ("обещаем") с помощью set_value().

std::future получает обещанное значение с помощью get().

Помогает, когда нужно передавать одно значение. Если больше, то скорее всего придется писать
свою многопоточную очередь.

```c++
#include <chrono>
#include <future>
#include <iostream>
#include <string>
#include <thread>

int main() {
    std::promise<std::string> input_promise;
    std::future<std::string> input_future = input_promise.get_future();

    std::thread producer([&]() {
        std::string input;
        std::cin >> input;
        input_promise.set_value(std::move(input));
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        std::cout << "producer after set_value-1\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));
        std::cout << "producer after set_value-2\n";
    });

    std::thread consumer([&]() {
        std::cout << "Consumer started\n";
        std::string input = input_future.get();
        std::cout << "Consumed!\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(2000));
        std::cout << "Processed string: " << input << "\n";
    });

    consumer.join();
    producer.join();
}
```
По сути promise – это std::optional + std::mutex, get – взять mutex и вернуть значение. Но если значения нет,
то надо как-то уснуть, пока оно не появится. Но как..?

## conditional_variable
std::condition_variable – еще один примитив синхронизации. Должен быть жестко связан с каким-то mutex и 
каким-то условием, то есть позволяет быть уверенным, что условие true.

wait(l) –> жди, пока кто-нибудь не скажет notify_one(). wait пишется в while (особенность языка, что позволяет
разбудить случайный поток).

```c++
#include <chrono>
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <string>
#include <thread>

int main() {
    std::mutex m;
    std::string input;
    bool input_available = false;
    std::condition_variable cond;

    std::thread producer([&]() {
//        while (true) {
            std::cin >> input;
            std::unique_lock l(m);
            input_available = true;
            cond.notify_one();  // Разбудит один случайный поток. Важно, чтобы это было ему по делу.
//        }
    });

    std::thread consumer([&]() {
//        while (true) {
            std::unique_lock l(m);
            while (!input_available) {  // while, не if!
                cond.wait(l);
            }
            // Эквивалентная while конструкция, чтобы было сложнее набагать и были понятнее намерения.
            cond.wait(l, []() { return input_available; });  // "ждём выполнения предиката".
            std::string input_snapshot = input;
            input_available = false;
            l.unlock();

            std::this_thread::sleep_for(std::chrono::milliseconds(2000));
            std::cout << "Got string: " << input_snapshot << "\n";
//        }
    });

    consumer.join();
    producer.join();
}
```
---
## 27.02.23
## Потоки
### promise, future
Как можно передать значение?

Наивная реализация ожидания: сделать в producer флаг, помечающий пришедший input, в consumer взять mutex, проверять флаг,
продолжить, если ничего не пришло.

Чем решение плохо: бесконечно жрет процессор, даже когда ничего не происходит. Потому что
mutex не умеет рассказывать потокам, что ничего не произошло.
```c++
int main() {
    std::mutex m;
    std::string input;
    bool input_available = false;

    std::thread producer([&]() {
        while (true) {
            std::cin >> input;

            std::unique_lock l(m);
            input_available = true;
        }
    });

    std::thread consumer([&]() {
        while (true) {
            std::string input_snapshot;
            {
                std::unique_lock l(m);
                if (!input_available) {
                    continue;
                }
                input_snapshot = input;  // Не хотим, чтобы input изменился, пока мы ждём две секунды.
                input_available = false;
            }  // l.unlock();

            std::this_thread::sleep_for(std::chrono::milliseconds(2000));
            std::cout << "Got string: " << input_snapshot << "\n";
        }
    });

    consumer.join();
    producer.join();
}
```
### condition_variable
Ровно для таких случаев (когда есть какое-то условие) придумали `condition_variable`.

(можно вместо while (spurious wakeup) писать `cond.wait(l, []() { return input_available; });`)

notify_one() – разбудить они случайный поток.

notify_all() – разбудить все потоки.

Баги в многопоточных программах ищутся или с помощью санитайзеров, или при помощи доказательства,
что каждая переменная или используется ровно одним потоком, или используется разными
потоками всегда под mutex'ом, или ей все равно.

Сверху вниз:

* mutex – сам себя синхронизирует.
* std::cin >> input; – запись идет просто так, то есть producer может записывать когда угодно. Даже при чтении input из consumer. Фактически UB (чтобы поймать, надо чтобы cin записывал прямо одновременно с чтением из input, что очень маловероятно)

Пофиксить – поменять местами строки std::cin >> input; и std::unique_lock l(m);

* mutex разблокирован очень-очень мало времени, не успеваем переключиться на consumer и схватить.

Пофиксить (плохой вариант) – попросить систему потупить.
```c++
std::this_thread::yield();  // Просьба ОС что-то поделать
```
Костыльно, но никаких гарантий, сколько работает и что делает.

Пофиксить (плохой вариант-2) – задержка.
```c++
std::this_thread::sleep_for(std::chrono::milliseconds(1));
```

Но на самом деле, если программа работает только с задержками и/или не помогает yeild, то 
что-то в ней не так. Например, здесь mutex хватается на слишком малое время.

! чем меньше времени под mutex, тем лучше (чтобы не заставлять потоки голодать, когда одному досталось все , а другим ничего).

````c++
int main() {
    std::mutex m;
    std::string input;
    bool input_available = false;
    std::condition_variable cond;

    std::thread producer([&]() {
        while (true) {
            std::cin >> input;
            std::unique_lock l(m);
            input_available = true;
            cond.notify_one();  // Разбудит один случайный поток. Важно, чтобы это было ему по делу.
            // cond.notify_all();  // Разбудит все потоки.
        }
    });

    std::thread consumer([&]() {
        while (true) {
            std::unique_lock l(m);
            while (!input_available) {  // while, не if! spurious wakeup
                cond.wait(l);
            }
            // Эквивалентная while конструкция, чтобы было сложнее набагать и были понятнее намерения.
            // cond.wait(l, []() { return input_available; });  // "ждём выполнения предиката".
            std::string input_snapshot = input;
            input_available = false;
            l.unlock();

            std::this_thread::sleep_for(std::chrono::milliseconds(2000));
            std::cout << "Got string: " << input_snapshot << "\n";
        }
    });

    consumer.join();
    producer.join();
}
````

Решение:

Читать вне mutex в перменную, используемую в одном потоке, потом под mutex сделать
быстрый move, сбросить флаг и прочие быстрые операции.

Все еще не очень (очередь длины 1), но уже неплохо.

```c++
#include <chrono>
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <string>
#include <thread>

int main() {
    std::mutex m;
    std::string input;
    bool input_available = false;
    std::condition_variable cond;

    std::thread producer([&]() {
        while (true) {
            std::string input_buf;
            std::cin >> input_buf;  // Не держим мьютекс на долгих операциях.

            std::unique_lock l(m);
            input = std::move(input_buf);
            input_available = true;
            cond.notify_one();
        }
    });

    std::thread consumer([&]() {
        while (true) {
            std::unique_lock l(m);
            cond.wait(l, [&]() { return input_available; });
            std::string input_snapshot = std::move(input);  // Тут тоже можно соптимизировать и добавить move.
            input_available = false;
            l.unlock();

            std::this_thread::sleep_for(std::chrono::milliseconds(2000));
            std::cout << "Got string: " << input_snapshot << "\n";
        }
    });

    consumer.join();
    producer.join();
}
```
### Важное про condition variable
* `wait()` — это оптимизация. Если её заменить на "отпустили мьютекс, подождали, взяли мьютекс обратно", то всё ещё должно работать.
* Следствие: condition variable никогда не используются без какого-то отслеживаемого условия или мьютекса.

### Мораль
* Никогда не модифицируйте ничего без мьютекса!
  Даже если сейчас не стреляет, ловить потом будет сложно.
* Не надо брать мьютекс надолго.
* Обычно проще писать не через condvar'ы, а через future или каналы (потокобезопасная очередь сообщений).
    * Но ещё лучше вообще писать без потоков или хотя бы не синхронизировать ничего руками.

### async
Берет лямбду и запускает лямбду в каком-то потоке (новом, зарезервированном, существующем, неясно).

С одной стороны, удобно, написать и поехали. С другой, мы ничего не можем сделать с потоком, кроме как получить значение.
А еще нет гарантий, что в новом потоке запустится. Так что создает ложно впечатление, что вы обо всм подумали...

А еще считается плохим стилем!
```c++
#include <chrono>
#include <future>
#include <iostream>
#include <string>
#include <thread>

int main() {
    // Может запустить код как в этом(!) потоке, так и в каком-нибудь соседнем. Без гарантий.
    std::future<std::string> input_future = std::async([]() {
        std::string input;
        std::cin >> input;
        return input;
    });
    // Похоже на команду go в языке Go, что может быть не очень хорошим стилем,
    // даже если накрутить всяких .then()/.map()/.flatMap(); это слишком низкоуровнево.
    // Например, отмену вычислений самому писать.
    // https://vorpus.org/blog/notes-on-structured-concurrency-or-go-statement-considered-harmful/
    // https://ericniebler.com/2020/11/08/structured-concurrency/

    std::thread consumer([&]() {
        std::cout << "Consumer started\n";
        std::string input = input_future.get();
        std::this_thread::sleep_for(std::chrono::milliseconds(2000));
        std::cout << "Got string: " << input << "\n";
    });

    consumer.join();
}
```
### mutable

Пишется рядом с полями класса. Означает, что можно менять даже в const-qualified методах.

Применения:
1. для mutex'а в каком-то классе, чтобы использовать его в методах (например, для get).

```c++
#include <mutex>

struct atomic_int {
    int get() const {
        std::unique_lock l(m);
        return value;
    }

    void set(int new_value) {
        std::unique_lock l(m);
        value = new_value;
    }

private:
    mutable std::mutex m;  // Can be changed even in const-qualified methods.
    int value;
};

int main() {
}
```

2. чтобы что-то закешировать посчитанное значение.
```c++
struct matrix {
    int determinant() const {
    }

private:
    int n;
    std::vector<int> data;
    mutable std::optional<int> last_determinant;
};
```
## Сеть
We need TCP/IP.
### IP (Internet Protocol)
* Each device has zero or more IP addresses 
  * IPv4: 4 bytes, typically in decimal, point-separated. Only 4B addresses. 
  * IPv4: 127.0.0.1 is "local host", means "this device"
  * IPv6: 16 bytes, not covered, e.g. [::1] or [2607:f8b0:4004:c1b::65]
* Domain Name System (DNS) is a special system which translates "host name" like hse.ru to IP addresses (and other stuff)
  * Not demonstrated: nslookup tool 
  * Translation is not unique 
* Similar IP addresses for physically connected devices form a network 
  * E.g. my Wi-Fi router is 192.168.10.254, my computer is 192.168.10.20, they see each other
  * My computer also has a separate IP address and a separate network for WSL
* IP can send small packages (~2-64 KiB), one-way only. 
* If you would like to go to an IP far away, you may need to hop through multiple devices (routers)
  * Demonstrated: tracert -w 100 mirror.yandex.ru (traceroute on Linux/macOS)
  * Note how there are multiple "local" IP addresses: see Network Address Translation (NAT) for TCP 
  * Note that this is one-way road: you cannot reach my computer at 192.168.10.20 if you're not in my network 
  * Global IPs are typically reachable from anywhere with "Internet Access"
### TCP
New term: _connection_.

  client (initiates)           server (accepts)
  ┌───────────────────┐  bytes  ┌────────────────┐
  │      ip:port      ├────────►│     ip:port    │
  │95.161.246.38:65124│◄────────┤186.2.163.228:80│
  └───────────────────┘  bytes  └────────────────┘ 
* TCP port is an integer between 1 and 65535 
* TCP socket is a (ip, port) pair 
* Bytes reliably flow in both directions (no duplicates, no reordering, no misses).
* (source ip, source port, destination ip, destination port) is a unique connection identifier

  server                       clients
  ┌─────────95.161.246.48:65124
  ▼
  186.2.163.228:443◄────────95.161.246.48:61536
  ▲
  └─────────217.66.156.214:53612
* Source IP address and source port are typically automatically chosen (e.g. port is random, IP is the best to connect to the destination)
* One has to know destination IP address and the destination port 
  * There are some standard TCP ports for servers: https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers

### Command line tools
See exercises

* Netcat 
  * Start server which waits for a single client: nc -l -p 10000 
  * Connect client: nc localhost 10000 (127.0.0.1 is better so localhost is not accidentally IPv6)
  * Reads/writes data line-by-line just like your typical C++ program
* Telnet 
  * Client only: telnet localhost 10000 
  * Sends key strokes immediately, does not display them on the screen.
* Web browser 
  * Client only: http://localhost:10000 (not https, it's encrytped)

### Сервер
Работа с сетью с помощью стандартного С++ невоможна, пока только через сторонние библиотеки. 
На курсе пользуемся библиотекой ```<boost/asio.hpp>```. Библиотека мощно, но будем использовать в простом, блокирующем режиме (поток
блокируется до поступления байт). Есть еще асинхронный режим.

`boost::asio::io_context io_context` – один раз создать, которая занимается обработкой взаимодействия по сети и не только в случае boost.

`tcp::acceptor acceptor(
io_context, tcp::endpoint(tcp::v4(), std::atoi(argv[1]))` – подключение, принимает контекст, где слушать (`tcp::()` везде)) и кого слушать. 

`.local_endpoint()` – на каком порту и на каком IP-адресе принимаются подключения.

`.accept` – блокирует выполнение потока, пока кто-нибудь не подключился. После подключения возвращает сокет (подключение).

`tcp::iostream` – превращение сокета в что-то похожее на cin/cout.

Такой сервер умеет обрабатывать только одного клиента (из-за блокирующего `.accept()`) –>
вылезла многопоточность, чтобы создавать поток на клиента.

```c++
#include <boost/asio.hpp>
#include <cassert>
#include <cstdlib>
#include <exception>
#include <iostream>
#include <sstream>
#include <utility>

using boost::asio::ip::tcp;

int main(int argc, char *argv[]) {
    assert(argc == 2);

    boost::asio::io_context io_context;  // TODO: once per thread or per app?
    tcp::acceptor acceptor(
        io_context, tcp::endpoint(tcp::v4(), std::atoi(argv[1]))
    );
    std::cout << "Listening at " << acceptor.local_endpoint() << "\n";

    tcp::iostream client([&]() {
        tcp::socket s = acceptor.accept();
        std::cout << "Connected " << s.remote_endpoint() << " --> "
                  << s.local_endpoint() << "\n";
        return s;
    }());

    while (client) {
        std::string s;
        client >> s;
        client << "string: " << s << "\n";
    }
    std::cout << "Completed\n";
}
```
### Клиент
Похоже на север, но вместо создания асептора создаем подключение:
```c++
auto create_connection = [&]() {
tcp::socket s(io_context);
boost::asio::connect(
s, tcp::resolver(io_context).resolve(argv[1], argv[2])
);
return tcp::iostream(std::move(s));
}
```

`tcp::resolver(io_context).resolve(host, port)` – преобразование в IP-адрес host и port.

`.local_endpoint()` – откуда, `.remote_endpoint()` – куда подсоединились.

`.shutdown()` – метод и у клиента, и у сервера, говорящий о том, что больше не будут отсылаться пакеты.

`.get()` – прочитать байты от сервера.

```c++
#include <boost/asio.hpp>
#include <cassert>
#include <exception>
#include <iostream>
#include <sstream>
#include <utility>

using boost::asio::ip::tcp;

int main(int argc, char *argv[]) {
    assert(argc == 3);

    boost::asio::io_context io_context;

    auto create_connection = [&]() {
        tcp::socket s(io_context);
        boost::asio::connect(
            s, tcp::resolver(io_context).resolve(argv[1], argv[2])
        );
        return tcp::iostream(std::move(s));
    };
    tcp::iostream conn = create_connection();
    std::cout << "Connected " << conn.socket().local_endpoint() << " --> "
              << conn.socket().remote_endpoint() << "\n";

    conn << "hello world 123\n";
    conn.socket().shutdown(tcp::socket::shutdown_send);
    std::cout << "Shut down\n";

    int c;
    while ((c = conn.get()) != std::char_traits<char>::eof()) {
        std::cout << static_cast<char>(c);
    }

    std::cout << "Completed\n";
    return 0;
}
```

### Передача данных
Передача данных – это передача БАЙТОВ, при этом нет гарантий, что байты приходят заявленными группами/сообщениями. 

```c++
for (;;) {
    std::vector<char> buf(50);
    std::size_t read = client.receive(buffer(buf), 0);  // reads _something_
    std::string s(buf.begin(), buf.begin() + read);
    std::cout << "string of length " << read << ": " << s << "\n";
}
```

`.send()` отправляет сколько-то байт, `.receive()` – принимает сколько-то байт. 

! В TCP нет понятия сообщения, поэтому нужно специально договариваться/обрабатывать, что является корректным вводом.
```c++
for (int i = 0; i < 20; i++) {
        s.send(buffer("hello\n"));  // bad
        // write(s, buffer("hello\n"));  // good, but does not matter
    }
```

### Баг в GCC
Boost так написан, что (client << "x\n") выкидывает исключение при завершении ввода, но оно вызывается как-то хитро, что в итоге программа
падает по std::terminate.
```c++
try {
    try {
        boost::asio::io_context io_context;
        tcp::acceptor acceptor(io_context, tcp::endpoint(tcp::v4(), 10000));
        std::cout << "Listening at " << acceptor.local_endpoint() << "\n";

        tcp::iostream client(acceptor.accept());
        client.exceptions(
            std::ios_base::failbit | std::ios_base::badbit |
            std::ios_base::eofbit
        );  // (1)
        while (client << "x\n") {
        }
        std::cout << "Completed\n";
    } catch (std::exception &e) {
        std::cout << "Exception: " << e.what() << "\n";
    }
} catch (...) {
    std::cout << "Unknown exception\n";
}
```
---
## 06.03.23
## Сети
### NAT (Network Address Translation)
```
  client (initiates)            router local     router internet              server (accepts)
┌───────────────────┐         ┌──────────────┬───────────────────┐         ┌────────────────┐
│      ip:port      ├────────►│      ip      │      ip:port      ├────────►│     ip:port    │
│192.168.10.20:65124│◄────────┤192.168.10.254│95.161.246.38:23768│◄────────┤186.2.163.228:80│
└───────────────────┘         └──────────────┴───────────────────┘         └────────────────┘
     other client
┌───────────────────┐
│      ip:port      │
│192.168.10.21:44234│
└───────────────────┘
```

1. Router intercepts the connection request from the client.
2. Initiates a new one "on behalf" of the client.
3. Remembers that data flowing to `:23768` should go to the local network to `192.168.10.20:65124`.

Now:

* Client thinks it talks to the server.
* Server thinks it talks to `95.161.246.38:23768`.

But you cannot initiate connection from the outside.

### Port forwarding
If you want to overcome NAT, do like in old local network games:

1. Easiest way: buy(!) a public IP address as a part of VDS/VPS (Virtual Dedicated/Private Server).
   I may set up an instance for your projects, if you wish.
   SSH reverse tunnel (GUI clients should support it as well):
* https://habr.com/ru/company/ruvds/blog/676596/
* https://timeweb.cloud/tutorials/network-security/ssh-tunnels#obratnyj-tunnel
2. Use Hamachi or other VPN (Virtual Private Network) _for local network gaming_, not for "private browsing".
3. For HTTP(S) protocol only (subset of TCP): ngrok and alternatives: https://github.com/anderspitman/awesome-tunneling
4. Configure your router with "port forwarding"

Different styles of writing networking (not only) code: https://stackoverflow.com/a/31298006/767632
Actually, any code with two participants, e.g. your program and the networking library.

Typically APIs only support one style, sometimes all.

### Blocking
You call a function, it does something and returns the result.
May _block_ your thread for a while or indefinitely.
Sometimes you may ask it to do multiple things and/or not block more than _a timeout_.

```
x = s.readInt();
cout << "Read: " << x;

if (s.readInt(y, 100ms)) {
    cout << "Read: " << y;
} else {
    cout << "Timed out\n";
}
```

Easier to code, impossible to interrupt, needs a thread per action.

### Callbacks
Also called: ~non-blocking, ~event-based, ~asynchronous.

You ask the library to do something, it starts doing in the background.
Your code proceeds.
It "calls you back" when it's done or failed.
You can take the next action then.

The library decides everything: when to call and what to do next.
You don't have "the next statement".

Much harder to code, easy to intermix and interrupt, only a single thread is needed (but there may be more).

### In frameworks
```
class MySocketHandler : public SocketHandler {
    void onConnected() override {
        this->startReadingInt();
        cout << "Started reading int\n";
    }

    void intRead(int x) override {
        cout << "Read: " << x << "\n";
        if (x % 2 == 0) {
            this->startReadingInt();
        } else {
            this->shutdown();
        }
    }
};
```

In Qt you have to manually connect signals with slots.

### In libraries
```
int main() {
    s1.startReadingInt([&](int x) {
        cout << "Read from s1: " << x;
        s1.startReadingInt([&](int y) {
            cout << "Read from s1: " << x << " " << y;
        });
    });
    s2.startReadingInt([](int x) {
        cout << "Read from s2: " << x;
    });
    // ...
    framework.run();
}
```

### Unanswered questions
* What thread do lambda/callback runs in?
  Typically there is a single dedicated "event thread" or "UI thread" which handles all events (including mouse and keyboard), you should never do anything long there.
* What mutexes are held? What operations are permitted?
* Can the callback run before `startReadingInt` returns?

### Polling
Also called: ~non-blocking, ~event-based, ~synchronous

Almost never happens in frameworks, they use it under the hood.
See `select` and `epoll` in Linux, `WaitForMultipleObjects` in Windows.

For a project: use less libraries than more.
If you use Qt, it already has almost anything you need, including sockets.
Principles are the same, I don't recommend trying to set up both Boost and Qt. You can, though.

## Потоки
### Deadlock
Потоку нужно захватить оба mutex'а –> возможно зависание из-за образовавшегося deadlock'а: оба потока
выполнили по одной строке и ждут, пока другой отпустит mutex m1.
```c++
std::mutex m1, m2;
std::thread t1([&]() {
    for (int i = 0; i < N; i++) {
        std::unique_lock a(m1);
        std::unique_lock b(m2);
    }
});
std::thread t2([&]() {
    for (int i = 0; i < N; i++) {
        std::unique_lock b(m2);
        std::unique_lock a(m1);
    }
});
// N=10'000: уже deadlock: пусть сначала каждый поток схватил себе m1/m2, а потом ждёт второй. А он занят.
t2.join();
t1.join();
```

Решение костыльное: `scoped_lock(m1, m2)`.

Другой способ: программисту сделать глобальный порядок на mutex'ах.
```c++
std::mutex m1, m2;
std::thread t1([&]() {
    for (int i = 0; i < N; i++) {
        std::scoped_lock a(m1, m2);
        // Под капотом вызывает std::lock.
        // Он пытается избежать deadlock, например:
        // while (true) {
        //     m1.lock();
        //     if (m2.try_lock()) break;
        //     m1.unlock();
        // }
        // 
        // При этом &m1 < &m2 — UB (нельзя сравнивать указатели не из одного массива/объекта), но можно через std::less<std::mutex*>
    }
});
std::thread t2([&]() {
    for (int i = 0; i < N; i++) {
        std::scoped_lock b(m2, m1);
    }
});
t2.join();
t1.join();
```

### Атомарные операции
Внутрь double_foo может влезть какая-нибудь еще операция.
```c++
void foo(int x) {  // Атомарная операция, atomic.
    std::unique_lock l(m);
    std::cout << "foo(" << x << ")\n";
}

void double_foo(int x) {  // Неатомарная операция :(
    foo(x);
    foo(x + 1);
}
```

Можно было бы взять mutex два раза, но так поток заблокирует сам себя.
```c++
void foo(int x) {  // Атомарная операция, atomic.
    std::unique_lock l(m);
    std::cout << "foo(" << x << ")\n";
}

void double_foo(int x) {    // Неатомарная операция :(
    std::unique_lock l(m);  // Берём мьютекс первый раз.
    foo(x);  // Берём мьютекс второй раз, deadlock :( Можно было бы взять
             // recursive_mutex, но это обычно плохой стиль.
    foo(x + 1);
}
```

Можно взять два разных mutex, но операции все еще могут пересекаться между собой.
```c++
std::mutex m;
std::mutex m2;
    
void foo(int x) {  // Атомарная операция, atomic.
    std::unique_lock l(m);
    std::cout << "foo(" << x << ")\n";
}

void double_foo(int x) {  // Неатомарная операция :(
    std::unique_lock l(m2);  // Берём другой мьютекс, deadlock отсутствует.
    foo(x);
    foo(x + 1);
}
```


Как принято делать: выносить повторную часть в отдельный приватный метод, который предполагает, что
блокировка уже взята.

```c++
private:
    std::mutex m;

    void foo_lock_held(int x) {
        std::cout << "foo(" << x << ")\n";
    }

public:
    // Публичный интерфейс из атомарных операций.
    // Над ним надо очень хорошо думать, потому что комбинация двух атомарных
    // операций неатомарна.
    void foo(int x) {
        std::unique_lock l(m);
        foo_lock_held(x);
    }

    void double_foo(int x) {
        std::unique_lock l(m);
        foo_lock_held(x);
        foo_lock_held(x + 1);
    }
```

Проблема: возвращается ссылка на `v[i]`, но после снятия mutex могло оказаться, что 
в вектор добавили очень много элементов и он переехал в памяти.

```c++
template <typename T>
struct atomic_vector {
private:
    mutable std::mutex m;
    std::vector<T> v;

public:
    void push_back(T x) {
        std::unique_lock l(m);
        v.push_back(std::move(x));
    }

    // UB is imminent!
    const T &operator[](std::size_t i) const {
        std::unique_lock l(m);
        return v[i];
    }
};
```

### TIC-TOU (Time of Check, Time of Use)
Между проверкой баланса и снятием денег могло успеть произойти несколько операций....

Реальный пример: 1'000'000 запросов по возвращению денег на счет в секунду -> вернулось больше, чем было.

В потоках не млжет быть геттеров – как только значение вернулось, то оно уже устарело.
```c++
struct User {
private:
    mutable std::mutex m_;
    int balance_ = 1'000'000;

public:
    // TOC-TOU is imminent! Just by looking at the API.
    int balance() const {
        std::unique_lock l(m_);
        return balance_;
    }

    void decrease_balance(int decrease_by) {
        std::unique_lock l(m_);
        balance_ -= decrease_by;
    }

int main() {
    User u;
    std::thread t([&u]() {
        while (u.balance() >= 3) {  // time of check
            u.decrease_balance(3);  // time of use
        }
    });
    while (u.balance() >= 4) {  // time of check
        u.decrease_balance(4);  // time of use
    }
    t.join();
    std::cout << u.balance() << "\n";  // Should be >= 0, but can be < 0
}
```

Решение: использовать более сложные операции. 
```c++
struct User {
private:
    mutable std::mutex m_;
    int balance_ = 1'000'000;

public:
    // TOC-TOU is imminent! But the name suggests likewise.
    int approximate_balance() const {
        std::unique_lock l(m_);
        return balance_;
    }

    // No need to check before.
    bool decrease_balance(int decrease_by) {
        std::unique_lock l(m_);
        if (balance_ < decrease_by) {
            return false;
        }
        balance_ -= decrease_by;
        return true;
    }
};

int main() {
    User u;
    std::thread t([&u]() {
        while (u.decrease_balance(3))
            ;  // single operation
    });
    while (u.decrease_balance(4))
        ;  // single operation
    t.join();
    std::cout << u.approximate_balance() << "\n";
    assert(u.approximate_balance() >= 0);
}
```

### shared pointer
Можно безопасно копировать в разных потоках (счетчик количества копий увеличивает атомарно).
```c++
#include <iostream>
#include <memory>
#include <thread>

int main() {
    std::shared_ptr<int> p = std::make_unique<int>(10);

    // Each shared_ptr<int> p is actually three objects with different semantics:
    // 1. `p` itself is not thread-safe, but you can a) read-only; b) use mutex; or c) use std::atomic_* functions.
    // 2. Reference counter inside `p` is thread-safe (probably via atomic)
    // 3. `*p` is not thread-safe, but you can use mutex

    std::thread t1([p]() {
        for (int i = 0; i < 100'000; i++) {
            auto p2 = p;  // Thread-safe, even though it increases reference counter.
            ++*p;  // Non thread-safe.
        }
    });

    std::thread t2([p]() {
        for (int i = 0; i < 100'000; i++) {
            auto p2 = p;
            ++*p;
        }
    });

    t1.join();
    t2.join();

    std::cout << *p << std::endl;  // race-y.
    std::cout << p.use_count() << std::endl;  // non-race-y, always 1 here.
    p = nullptr;  // No leaks.
}
```

### thread_local
Переменная, которая создается один раз на весь поток. Инициализируется либо в момент начала потока,
либо в момент первого обращения. Новый тип storage duration.

```c++
#include <chrono>
#include <iostream>
#include <thread>

struct Foo {
    Foo(int id) {
        std::cout << "Foo(" << id << ")@" << std::this_thread::get_id() << "\n";
    }
    ~Foo() {
        std::cout << "~Foo()@" << std::this_thread::get_id() << "\n";
    }
};

thread_local Foo foo(10);  // new kind of storage duration.

int main() {
    std::cout << "T1 &foo=" << &foo << "\n";  // Компилятор может проинициализировать лениво, но может и в момент начала потока.
    std::thread t([&]() {
        thread_local Foo bar(20);  // Инициализирует при проходе через эту строчку.
        std::cout << "Before wait\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(3000));
        std::cout << "T2 &foo=" << &foo << "\n";  // Компилятор может проинициализировать лениво, но может и в момент начала потока.
        std::cout << "T2 &bar=" << &bar << "\n";  // Уже точно проинициализировано выше.
    });
    std::cout << "Waiting for it...\n";
    t.join();
    std::cout << "Joined\n";
    return 0;
}

/*
Зачем можно использовать: отслеживать стэк вызовов в каждом потоке.

TEST_CASE() {  // Запомнили в thread-local переменную текущий test case
    SUBCASE() {  // Запомнили текущий SUBCASE
       CHECK(....)  // Можем выводить красивое сообщение
    }
}
*/
```

### Немного о потоках и параллельности
* Процессоры с 2004 года не увеличивают принципиально частоту (Pentium 4 Prescott - 2.8 ГГц).
* С тех пор скорость работы повышается другими методами: размер кэша, скорость памяти, периферии.
* Уже уткнулись в ограничения размера процессора из-за скорости света.

Что делают для ускорения автоматически:

* Выполнять команды параллельно, если они независимы:
```с++
int x = a * b;
int y = a / b;
```
* В том числе команды, читающие из памяти.
* Многоканальная память.
* Переставить команды так, чтобы параллельности было побольше.

Что делают руками:

* Векторизация SIMD (Single-Instruction-Multiple-Data): обрабатываем 8 интов за раз.
  Можно жёстко эксплуатировать: https://habr.com/ru/company/ruvds/blog/551060/
* Учесть кэши и обращения к памяти.
* Несколько физических ядер процессора.
* Несколько процессоров.
* Несколько компьютеров.

Потоки можно применять не только для ускорения.
Можно для замедления, зато с упрощением кода (блокирующий ввод-вывод) или многозадачностью (много окон).

Параллельные алгоритмы:

* Параллелятся почти как угодно:
```с++
int sum = 0;
for (int x : values) sum += x;
```
* Не параллелятся вообще:
```с++
int steps = 0;
for (int x = 1; x != 0; x = f(x)) steps++;
```
* Параллелятся, но не очень: компиляция.
* Надо придумывать что-то хитрое, чтобы запустить на тысячах ядер:
  * Иногда выгоднее распараллелить перебор, потому что умный алгоритм не параллелится
  * Merge sort. https://compscicenter.ru/courses/video_cards_computation/2020-autumn/classes/


### Шаблонный метод у класса
Шаблонный метод нужно реализовывать внутри заголовочных файлов (так как инстанцирование
шаблона происходит в момент компиляции). При этом возможно объявление и в cpp-шнике, ODR не нарушится. 
```c++
struct Applier {
    int value;

    // We only know how to write it inside the class, no ODR violations here surprisingly.
    template<typename F>
    void apply(F operation) {
        std::cout << "Calling with " << value << "\n";
        operation(value);
    }
};

struct Print {
    void operator()(int x) {
        std::cout << x << "\n";
    }
};

int main() {
    Applier a{10};
    Print p;
    a.apply(p);
}
```