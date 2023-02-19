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