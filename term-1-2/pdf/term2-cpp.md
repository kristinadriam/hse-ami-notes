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
