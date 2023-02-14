## *05/09/22*

## Базовое

### Что изучаем

- Картинка `c-vs-cpp.pdf`
    - Есть реализованный в компиляторах C++.
    Компиляторов много разных.
    - Есть стандартный C++.
    - Есть стандартный Си. Это другой язык, который возник раньше и 
послужил основой. Но с тех пор разошлись.
    - Есть олимпиадный C++. Он маленький.
    - А изучать мы будем что-то другое
- Сравните объёмы:
    - 
[https://en.cppreference.com/w/cpp/language](https://en.cppreference.com/w/cpp/language) 
против 
[https://en.cppreference.com/w/c/language](https://en.cppreference.com/w/c/language) 
(это только язык, без библиотеки)
    [https://en.cppreference.com/w/cpp](https://en.cppreference.com/w/cpp) 
против [https://en.cppreference.com/w/c](https://en.cppreference.com/w/c) 
(с библиотеками)
- Есть протекающие абстракции: 
[https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/)
    - C++ старый, но хочет совместимость с Си
    - Могут возникнуть беды со строчками: `"foo" + "bar"` и `"foo" + 4` 
делает чушь
    - Может возникнуть "препроцессор", "арифметика указателей", но мы это 
попробуем отложить до второго семестра

Все проблемы идут из концепции "программист всегда прав, даже если он 
неправ".
А из противоречия математически выводится абсолютно что угодно, любое 
поведение.

### Стандарты С++

У C++ есть несколько версий с разными фичами: C++98, C++03, C++11 (C++0x 
по историческим причинам), C++14 (C++1y), C++17 (C++1z), C++20, сейчас в 
работе C++23.
Почти обратно совместимы.

Мы используем C++17, но я буду делать отсылки к старым и новым.
На работе может встретиться любая версия в зависимости от.
Потому что мигрировать между версиями и компиляторами конкретно в C++ 
очень сложно: 
[https://www.youtube.com/watch?v=kJSXhZzEyZU](https://www.youtube.com/watch?v=kJSXhZzEyZU)

### Зоопарк компиляторов

[https://notes.algoprog.ru/cpp/basic_info.html#id5](https://notes.algoprog.ru/cpp/basic_info.html#id5)

Компилятор — программа, которая преобразует *исходный код* в *исполняемый 
файл* (бинарник, экзешник). Для этого ей обычно нужна *стандартная 
библиотека C++* и что-то ещё.

Упомянуть GCC (GNU Compiler Collection), Clang (LLVM, форк от Apple), 
Visual Studio, Intel C++ Compiler.
Есть отдельно компилятор и отдельно стандартная библиотека (обычно они 
тесно связаны, но бывают варианты).

Стандарт: попытка договориться, иногда получается: 
[https://en.cppreference.com/w/cpp/compiler_support](https://en.cppreference.com/w/cpp/compiler_support). 
Документ формальный. Из интересного: as-if rule. По факту компиляторов 
зоопарк, а ещё сам стандарт позволяет очень вольные трактовки. Их даже 
описывать необязательно.

Бывают расширения компиляторов вроде: 
[https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Extensions.html](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Extensions.html). 
Их иногда утаскивают друг у друга (Clang у GCC для совместимости), иногда 
чуть меняется синтаксис. А иногда общее: `#pragma pack` (не было на 
лекции).

Есть онлайн-компиляторы (wandbox, godbolt, cppinsights, ideone), но их вам 
не хватит.

### Комбинации компиляторов

В домашках должно работать под почти всеми комбинациями:

- Ubuntu
    - gcc + GNU libstdc++ (стандартная комбинация)
    - LLVM clang + GNU libstdc++
    - LLVM clang + LLVM libc++
    - Intel C++ Compiler Classic + GNU libstdc++
- Windows
    - Visual C++ + Microsoft STL (стандартная комбинация)
    - Много сборок gcc/libstdc++ с названиями mingw и mingw64: 
[https://www.mingw-w64.org/downloads/](https://www.mingw-w64.org/downloads/)
    - Много сборок LLVM clang
- macOS
    - Apple Clang + Apple libc++ (похожи на LLVM, но не совсем)
    - Есть сборки gcc и LLVM clang

Лучшей нет. Выбираете ту, что "принято" использовать вокруг вас.

Они обычно одинаковые, но чуть-чуть отличаются. И компилятор про это не 
расскажет:

- [https://t.me/experimentalchill/96](https://t.me/experimentalchill/96)
- Portability is Reliability: 
[http://evan.nemerson.com/2021/05/04/portability-is-reliability/](http://evan.nemerson.com/2021/05/04/portability-is-reliability/)

Как жить: выбираете один компилятор, под ним работает (где удобнее), 
регулярно проверяете код на других компиляторах и ОС. В домашках это будет 
автоматические, но медленно, редко и с плохой обратной связью, лучше 
локально.

### Среды разработки

Удобно иметь подсветку синтаксиса, подсказки "что писать дальше", 
мгновенную перемотку к функции, запуск компилятора, предупреждения о 
возможных ошибках и так далее.
Этим занимаются IDE: бывают специализированные (CLion, Visual Studio), 
бывают накручены поверх редакторов.
Вопрос вкуса и удобства: 
[https://notes.algoprog.ru/cpp/ide.html](https://notes.algoprog.ru/cpp/ide.html)

Я умею: либо текстовый редактор и компилировать из консоли, либо Visual 
Studio, либо CLion. Люблю текстовый редактор. Практики могут иметь другие 
предпочтения.

Проект — это набор настроек "как компилировать".

### Системы сборки

Так как IDE на вкус и цвет, а компилировать сложно, обычно делают через 
"систему сборки". Чтобы можно было одним конкретным консольным 
приложением, которое везде есть, компилировать. А дальше IDE могут проект 
системы сборки превратить в свой.

Системы сборки в C++ обычно прибиты гвоздями к компилятору: проект Visual 
Studio, файл `Makefile`, система Ninja...

Поэтому придумали ещё слой: CMake, проект - это файл `CMakeLists.txt`, а 
CMake умеет сгенерировать проект под любую систему сборки. А CLion читает 
`CMakeLists.txt` напрямую. Поэтому можно считать, что `CMakeLists.txt` - 
это такой "проект на C++", если мы живём в хоть сколько-то хорошем мире 
(но нет).

## Основы С++

### “Hello world” на С++

Функция `int main` возвращает число, *код возврата (exit code)*. 0 — 
успешное завершение.

```cpp
#include <iostream>  // Akin to `import` in Python/Java, details later

// Top-level: no code, only functions/global variables/#include/etc

int main() {  // entry point, the `main` function
    std::cout << "Hello World\n";  // printing to the "standard output" of 
the program
                                   // only double quotes, escaping with \ 
works
    return 0;  // any application has an "exit code", 0 is "success" by 
convention,
               // everything else is typically "runtime error"
}
```

Только у `main` можно не писать код возврата, сгенерируется автоматически.

```cpp
int main() {
    std::cout << "Hello World\n";
    // In `main` (and `main` only) `return 0` is automatically appended.
    // Hooray! Hello World in 4 lines. Maybe in 2.
}
```

### a + b

a + b — считали два числа, вывели их сумму. При объявлении переменной 
нужно указывать ее тип.

`std::endl` — перевод строки, сбрасывающий буффер ОС. Медленно.

```cpp
/* variable declaration: type of a variable is always fixed
   type is a pair of (possible values, possible operations) */
int a /* comment here */, b;
a = 5;  // ok
std::cin >> a >> b;
std::cout << a + b << std::endl;  // like "\n", but flushes buffer, much 
slower

// Alternative
int s = a + b;  // declare variable right before it's needed please!
std::cout << s << std::endl;
```

### Ошибка компиляции (сompilation error)

В С++ компилятор сначала читает программу, преобразует в испорлняемый 
файл, потом запускает. Поэтому можно ловить ошибки на этапе компиляции.

```cpp
int a, b;
std::cin >> a >> b;
a = "hello";  // compilation error: cannot assign string (`const char*` 
because legacy) to an `int` variable
std::cout << a + b << std::endl;
```

### Undefined behavior (UB)

*UB — undefined behavior* — неопределенное поведение. Программа 
компилируется, но возможно все, что угодно. В данном случае читаем 
значение из неинициализированной переменной.

```cpp
int x;
x = x * 0;
```

### Области видимости

Добавление фигурных скобок —>  добавление областей видимости в программу. 
Можно заводить переменные с одинаковыми именами в разных областях 
видимости.

```cpp
int a = 10, b = 20;

int main() {
    std::cout << a + b << "\n";  // 30
    {  // new scope
        std::cout << a << " " << b << "\n";  // 10 20
        int a = 5;  // "shadows" the global variable, I dislike this style
        std::cout << a << " " << b << "\n";  // 5 20
        // int a = 100;  // cannot make another variable in the same scope
        {
            int a = 6;  // "shadows" the local variable
            std::cout << a << "\n";  // 6
        }
        std::cout << a << "\n";  // 5
    }
    std::cout << a << "\n";  // 10
}
```

 

### Namespace

`namespace` — есть какие-то куски программы, где могут пересекаться 

В `std` живет стандартная библиотека. Можно явно указывать `std::…`. Можно 
писать `using namespace std` . Во втором случае могут быть проблемы: 
например, пересекаются названия функций из стандартной библиотеки и 
написанный собственными руками.

```cpp
// To the compiler: when unable to find X, check for std::X as well.
// Typically each library has its own "namespace", `std` is for the 
standard C++ library.
// Bad style: lots of stuff in `std`, and more gets added with each 
standard.
using namespace std;

int main() {
    int a, b;
    // Helps in olympiads: less to type, do not care about other compilers 
or newer standards.
    cin >> a >> b;
    cout << a << b;
}
```

---
## 12/09/22

## Типы

### **Int**

Целочисленные значения. Размер типа = размеру переменной. На 32/64-битных 
система)х гарантированно, что не меньше 32 бит, но бывает по-разному. 
Числа от $[-2^{31};2^{31}-1]$ . При переполнении — UB.

```cpp
int a = 10;
std::cout << a << "\n";
std::cout << sizeof(a) << " " << sizeof(int)
          << "\n";  // size in bytes, typically 4.
a = -10;
std::cout << a << "\n";  // negative numbers work

a = 2'000'000'000;
std::cout << a << "\n";  // 2e9 works

a = a * 2;  // UB: signed overflow.
            // 4e9 does not fit inside 4-byte `int` ([-2**31; 2**31-1])
            // My compiler wraps around here, but it's not necessary.
std::cout << a << "\n";
if (a < 0) {  // TODO: NEVER DO THAT, SEE NEXT FILE
    std::cout
        << "Overflow detected. But it's actually UB already, no use.\n";
}
```

**Пример UB на переполнение**

Ожидание: когда-то случится переполнение и число станет отрицательным.

Реальность: компилятор может оптимизировать, ведь при умножении 
положительного числа на положительное не может получится отрицательное. 

`-fsanitize=undefined` может ловить некоторые виды UB.

```cpp
int i = 1;
while (true) {
    i = i * 3;  // UB is typically caught by -fsanitize=undefined, 
execution
                // continues
    std::cout << i << "\n";
    if (i < 0) {  // Typically optimized out by the compiler.
        std::cout << "Overflow detected\n";
        return 0;
    }
}
```

### **Модификаторы `unsigned`, `signed` (signedness modifiers)**

`unsigned` можно писать почти к любому целочисленному типу.

```cpp
unsigned int a = 10;  // or just `unsigned`
std::cout << sizeof(a) << " " << sizeof(int)
          << "\n";  // size in bytes, typically 4.
```

Если попытаться записать отрицательное, то оно возьмется по модулю 
$2^{32}$.  Значения из диапазона $[0;2^{32}]$.

```cpp
a = -10;  // not UB: always modulo 2**32 (depending on size), i.e. 2**32 -
          // 1
a = 4'000'000'000;
a = a * 2;               // not UB: modulo 2**32
if (a < 0) {  // always false
    std::cout << "Always non-negative number is negative??\n";
}
```

Сравнивать с нулем бесполезно — всегда `false`.

При сравнении с $-1$ типы приводятся к общему типу, в данном случае, оба 
становятся беззнаковыми —> $-1$ становится $2^{32}-1$.

```cpp
a = 4'000'000'000;
if (a < 0) {  // always false
    std::cout << "Always non-negative number is negative??\n";
}
```

```cpp
// Not ok: `unsigned > signed` is `unsigned > unsigned`, `-1` is
// taken modulo 2**32. See "Usual arithmetic conversions" in
// https://en.cppreference.com/w/c/language/conversion
if (a > -1) {
    std::cout << "OK\n";
} else {
    std::cout << "Always non-negative number is not greater than -1??\n";
}
```

### **Модификаторы `long` и `short` (size modifiers)**

Модификатор `long` — гарантирует, что размер перменной будет хотя бы 32 
бита. Так тип `long long` занимает порядка 8 байт. Помещаются числа из 
диапазона $[-2^{31};2^{31}-1]$. 

```cpp
long long a = 9'000'000'000'000'000'000;
```

Любое значение типа `int` поместится в `long long`, произойдет неявное 
преобразование. При обратной операции не UB, но потеряется часть 
информации — *unspecified behaviour*. Может быть запишет последние 32 
бита, может запишет 0. Гарантий нет.

```cpp
long long y = x;  // ok, implicit conversion
        y = y * 1'000'000'000;
        int z = y;  // ok, implicit conversion, possible loss of 
information,
                    // unspecified (not undefined!) behavior
```

Также существует тип `unsigned long long`. Чтобы записать большую 
константу, стоит дописать `U`.

```cpp
unsigned long long a =
            18'000'000'000'000'000'000U;  // need the 'U' suffix for big
                                          // unsigned decimal literals
```

Помимо этого есть модификатор `short` (гарантирует, что размер переменной 
будет не меньше 16 бит) и `long long` (гарантирует, что размер переменной 
будет не меньше 64 бит).

Можно писать в любом порядке — `unsigned long long int` и `long int 
unsigned long` — одно и то же.

### **Литералы**

`123` — decimal.

Можно добавлять апострофы в любое число в любое место (с С++14).

```cpp
std::cout << 1'2345 << "\n";  // C++14
a = 123;                                                    // decimal
std::cout << a << " " << 100 * 1 + 10 * 2 + 1 * 3 << "\n";  // 123
```

`0x2F` — hexadecimal.

```cpp
a = 0x2F;  // hexadecimal: 0x or 0X
a = 0x2f;
a = 0xDEADBEEF;
```

`0123` — octal.

```cpp
a = 0123;  // octal
```

`0b101` — binary (c++14).

```cpp
a = 0b1101;                                  // binary (C++14)
```

*integer suffix* — может быть один из двух или оба сразу:

- `u` или `U` — *unsigned-suffix*

```cpp
unsigned long long l1 = 18446744073709550592ull; // C++11
```

- about size:
    - `l` или `L` — *long-suffix*
    
    ```cpp
    long l2 = 18'446'744'073l; // C++14
    ```
    
    - `ll` или `LL` — *long-long-suffix*
    
    ```cpp
    unsigned long long l3 = 1844'6744'0737'0955'0592uLL; // C++14
    ```
    
    - `z` или `Z` — *size-suffix*

### **Неявные преобразования (implicit conversions)**

При записи из разных типов происходит неявное преобразование.

```cpp
int n = 1L; // expression 1L has type long, int is expected
n = 2.1; // expression 2.1 has type double, int is expected
char *p = malloc(10); // expression malloc(10) has type void*, char* is 
expected
```

В выражениях: сначала вычисляется от int’ов `a * a`, результат которого, 
по мнению компилятора, int. 

```cpp
int a = 1'000'000'000;
long long b =
    a * a;  // UB: int * int is calculated inside int (signed overflow,
            // UB), and is then converted to long long
// Check with -fsanitize=undefined
```

Решение: явно преобразовать к long long. Тогда выражение вычисляется в 
long long.

(*C++)-style*  — `static_cast<long long>a`

```cpp
// Explicit conversion to `long long`.
long long b = static_cast<long long>(a) * a;  // long long * int
// See "Usual arithmetic conversions" in
```

*Олимпиадный стиль* — `1LL * a`.

```cpp
// Olympic style.
long long b = 1LL * a * a;  // long long literal * int * int
```

*C-style* — `(long long)a` (лучше не надо).

```cpp
// C-style cast, do not use.
long long b = (long long)a * a;  // long long * int
```

[Implicit 
conversions](https://www.notion.so/Implicit-conversions-57b1dae2f03a4190a9f2940f939c837a)

### **Auto**

При объявлении переменной можно вместе типа писать `auto`. Компилятор 
посмотрит на то, чем инициализируют, и сам определит тип.

```cpp
auto x = 1'000'000'000;   // int
auto y = 1'000'000'000U;  // unsigned int
auto x2 = 2 * x;          // int * int, int
auto y2 = 2 * y;          // int * unsigned int, unsigned int
```

В одной строке не может быть противоречий между предполагаемыми типами 
переменных.

```cpp
// auto x3 = 2 * x, y3 = 2 * y;  // CE: inconsistent deduction for 'auto'

auto z2 = 0x123LL;

// `auto` works with other types as well, but it is always deduced from 
the
// initialization.
```

Обязательно нужно проинициализировать перменные типа `auto`.

```cpp
// Does not work:
auto tmp; tmp = 12;
```

### **Другие целочисленные типы**

`short` — обычно занимает 2 байта.

```cpp
std::cout << "short: " << sizeof(short) << "\n";  // Typically 2 bytes
std::cout << "unsigned short: " << sizeof(unsigned short) << "\n";
```

`long` — то 4, то 8 байт.

```cpp
std::cout << "long: " << sizeof(long)
          << "\n";  // Typically 4 or 8 bytes (see msys2 vs Linux)
```

`std::size_t` (в библиотеке `<cstddef>`).  Иногда 4, иногда 8 байт. 
Беззнаковый. В нем хранят размеры объектов. Количество элементов в 
массиве, например.

```cpp
#include <cstddef>

std::cout << "std::size_t: " << sizeof(std::size_t)
          << "\n";  // Typically 4 or 8 bytes

std::size_t x =
    -1;  // std::size_t is unsigned, used for sizes of arrays/variables...
std::cout << x << "\n";
```

### **Bool**

1 байт, 2 значения — `true` или `false`.

```cpp
std::cout
        << "bool: " << sizeof(bool)
        << "\n";  // Typically 1 byte, but can only hold false (0) and 
true (1)
```

Если записать в `bool` что угодно, кроме 0, то это true. 

```cpp
b = 55;  // not UB, changed to 1
```

Можно складывать/умножать на `bool`. Где-то удобно.

```cpp
std::cout << b + 10 << "\n";  // bool + int, int, 1 + 10
```

## Арифметика

### **Присвоение**

Можно присваивать несколько переменных одновременно — `x = y = 20`. 

```cpp
int x = 0, y = 0;
x = y = 20;
```

Присваивание на самом деле возвращает присваиваемое значение.

```cpp
std::cout << (x = 15) << "\n";  // 15, but what for?
std::cout << x << "\n";         // 15
```

Ошибка, но не совсем — x = 5 вернет 5 —> условие `true`.

```cpp
if (x = 5) {  // modern compiler should emit warning
    std::cout << "x = " << x << "\n";
}
```

### **Операция +=**

`x += 10` <—> `x = x + 10`

```cpp
x = x + 10;
x += 10;
```

`x++` — сначала возвращается значение x, только потом увеличивается.

`++x` — сначала увеличивается значение x, потом возвращается увеличенное 
значение.

Аналогично `x--` и `--x`.

```cpp
x++;
++x;
```

Присвоение выполняется справа налево.

```cpp
int x = 10;
int y = x += 20;  // x += 20; y = x;
```

`x++ + --x` — UB. 

```cpp
int z3 = x++ + --x;  // UB: variable is modified twice
```

### **Деление**

Только оператор “`/`”. int / int —> int.

```cpp
std::cout << 7 / 3 << " " << 7 % 3 << "\n";  // 2 1, int / int is int
// Invariant: (a / b) * b + (a % b) == a
```

С какого-то стандарта при делении отрицательного на положительное (или 
положительного на отрицательное) всегда округляется к 0.

```cpp
// -2 -1 or -3 2, was implementation-defined behavior (sic!):
// 
https://gcc.gnu.org/onlinedocs/gcc/Integers-implementation.html#Integers-implementation
// 
https://stackoverflow.com/questions/2397984/undefined-unspecified-and-implementation-defined-behavior
// Since N2757 https://cplusplus.github.io/CWG/issues/614.html it is 
always
// -2 -1
std::cout << -7 / 3 << " " << -7 % 3 << "\n";

// -2 1 since N2757
std::cout << 7 / -3 << " " << 7 % -3 << "\n";

// 2 -1 since N2757
std::cout << -7 / -3 << " " << -7 % -3 << "\n";

```

Деление на 0 — UB.

```cpp
int x = 0;
std::cout << 1 / x << "\n";  // UB
std::cout << "OK 1\n";
std::cout << 1 % x << "\n";  // UB
std::cout << "OK 2\n";
```

Некоторый олимпиадный трюк, чтобы остаток был неотрицательным.

```cpp
// Olympic trick
int a = -10, b = 3;
std::cout << a % b << " " << (a % b + b) % b << "\n";
```

### **Побитовые операции**

Операции независимо делаются с каждым битом — `&` (*AND*), `|` (*OR*), `^` 
(*XOR*), `~` (*NOT*).

```cpp
// 3 = 0b011
// 5 = 0b101
std::cout << (3 & 5) << "\n";  // bitwise AND: 1
std::cout << (3 | 5) << "\n";  // bitwise OR: 7
std::cout << (3 ^ 5) << "\n";  // bitwise XOR: 6
std::cout << ~3U << "\n";      // bitwise NOT: 2**32 - 1 - 3
```

Побитовые сдвиги — `<< i` (умножение на $2^i$), `>> i` (деление на $2^i$).

```cpp
int a = 10;
std::cout << (a << 3) << "\n";  // *2**3, *8
std::cout << (a >> 2) << "\n";  // /2**2, /4
```

Примеры UB:

```cpp
std::cout << "===== shifts overflow =====\n";
std::cout << (a << 30) << "\n";  // UB: overflow
std::cout << (a << 31) << "\n";  // UB: overflow

unsigned a_u = 10;
std::cout << (a_u << 30) << "\n";  // OK: unsigned overflow

std::cout << "===== shifts UB =====\n";
a = -10;

// Implementation-defined in C++17 and prior, division in C++20
// 
https://gcc.gnu.org/onlinedocs/gcc/Integers-implementation.html#Integers-implementation
std::cout << (a >> 2) << "\n";

// UB in C++17 and prior, multiplication in C++20
std::cout << (a << 3) << "\n";

int b = 32;
std::cout << (0 << b) << "\n";  // UB: shifting by 8*sizeof() bits
std::cout << (0 >> b) << "\n";  // UB: shifting by 8*sizeof() bits

b = -1;
std::cout << (0 << b) << "\n";  // UB: shifting by negative
std::cout << (0 >> b) << "\n";  // UB: shifting by negative
```

Иногда побитовые операции используют, чтобы оптимизировать код. Чаще всего 
компилятор умеет делать это самостоятельно, но без гарантий.

```cpp

    // Bit shifts and operations are very fast: https://www.nandgame.com/
// https://godbolt.org/z/Y5hez7WMG
int a;
std::cin >> a;
int b = a * 33;  // probably optimized to (a + (a << 5))
std::cout << b << "\n";

int c = a / 33;
std::cout << c << "\n";

int d = a / 32;
std::cout << d << "\n";

int e = a % 32;  // a is signed, so we have to handle `a < 0` and return
                 // a negative result
std::cout << e << "\n";

int f = a & 0b11111;  // always non-negative
std::cout << f << "\n";

int g = static_cast<unsigned>(a) % 32;  // can be optimized to &
std::cout << g << "\n";
```

## Циклы

### **Оператор if**

База:

```cpp
// Shorter
if (n < 10)
    std::cout << "less than 10\n";

// Not so short
if (n < 10)
    std::cout << "less than 10\n";
else
    std::cout << "greater or equal than 10\n";
```

else всегда относится к самому вложенному if. 

```cpp
// Ambiguity
if (n < 10)
    if (n < 20)
        std::cout << "a\n";
    else  // always for the most nested if
        std::cout << "b\n";

// Harder to screw up
if (n == 10) {
    std::cout << "n is \n";
    std::cout << "equal to ten\n";
} else {
    std::cout << "not equal to ";
    std::cout << "ten\n";
}
```

Есть конструкция `else if`.

```cpp
if (n < 10) {
    std::cout << "n < 10\n";
} else if (n > 10) {
    std::cout << "n > 10\n";
} else {
    std::cout << "n == 10\n";
}
```

### **Логические операции**

`&&` (логическое умножение), `||` (логическое сложение), `!`. Приоритет 
`||` ниже, чем у `&&`.

```cpp
if (n % 3 == 0 && n % 5 == 0) {
    std::cout << "FizzBuzz\n";
} else if (n % 3 == 0 || n % 5 == 0) {
    std::cout << "Fizz or Buzz\n";
}
// Operator precedence: && is like *, || is like +:
// https://en.cppreference.com/w/cpp/language/operator_precedence
// a && b || c
// a || b && c
```

Аккуратно с `!`:

```cpp
// Just to demonstrate negation, if-else would be better
// if (!n % 3 == 0 || n % 5 == 0) {   // oops: !n
if (!(n % 3 == 0 || n % 5 == 0)) {
    std::cout << "Neither Fizz nor Buzz\n";
}
```

В стандарте С++ есть слова `and`, `or` и `not`.

```cpp
// Alternative operators are banned for us:
// https://en.cppreference.com/w/cpp/language/operator_alternative
if (not(n % 3 == 0 or n % 5 == 0)) {
    std::cout << "Neither Fizz nor Buzz\n";
}
```

Fun fact: пустая строка считается как `true`.

```cpp
if ("") {  // works, but does something weird
    std::cout << "Why is empty string truthful? We do not know yet\n";
}
```

### **Цикл for**

База:

```cpp
// for (init; cond; update)
for (int i = 0; i < n; i++) {
    std::cout << "i=" << i << "\n";
    if (i == 3) {
        std::cout << "  break\n";
        break;  // stop immediately
    }
    if (i % 2 == 1) {
        std::cout << "  continue\n";
        continue;  // continue to the next iteration (with update)
    }
    std::cout << "  end of iteration\n";
}
```

Переменная, объявленная внутри цикла, видна только внутри цикла.

```cpp
// std::cout << i << "\n";  // compilation error: `i` is dead
```

`++i` быстрее, чем `i++` (не нужно запоминать старое значение), но 
оптимизатор обысно справляется с этим.

```cpp
for (int i = 0; i < n; ++i) {  // ++i may be faster than i++, but 
optimizer
                               // for int is good enough.
    std::cout << "i=" << i << "\n";
}
```

Некоторые части цикла можно опускать:

```cpp
int j = 5;
for (; j < 10; j += 2)  // you can omit any part
    std::cout << "j=" << j << "\n";
std::cout << "loop done: " << j << "\n";
```

`for (;;)` — бесконечный цикл. Опущенное условие равносильно `true`.

```cpp
for (;;) {  // omitted condition is always true
}
```

### **Цикл while**

База:

```cpp
while (n-- > 0) {  // while (n --> 0)
    std::cout << n << "\n";
}
```

`while (true)` — бесконечный цикл. При этом совершенное бесплатно 
объявлять переменную внутри цикла, а не снаружи. Тогда жизнь перменной 
ограничена одной итерацией цикла и нет необходимости следить за ее 
значением до/после его выполнения. Чем меньше зависимостей между 
итерациями, тем лучше.

```cpp
while (true) {
    int v;  // Please do declare variables inside the loop, it's free!
    // 
    if (!(std::cin >> v)) {
        break;
    }
    std::cout << "v=" << v << "\n";
}
```

`do {`

`} while (…)` — проверяет условие не в начале, а в конце.

```cpp
do {
    std::cout << "infinite loop\n";
} while (true);
```

### **Оператор `,`**

Вычисляется слева направо. Сначала вычисляется то, что слева, потом то, 
что справа, и возвращает это значение. Удобно, например, использовать в 
циклах, когда нужно объявить несколько перменных.

```cpp
int a = 0, b = 0, c = 0;
std::cout << (a = 1, b = 2, c = 3) << "\n";  // 3
for (int i = 0, j = 10 /* usual variable declaration */; i <= j;
     i++, j-- /* comma operator */) {
    std::cout << i << " " << j << "\n";
}
```

## Массив

### База

`std::vector<int> v` — динамический массив. По умолчанию вектор пуст. 

```cpp
std::vector<int> v1, v2;  // empty dynamic arrays
```

Можно создавать определенного размера.

```cpp
std::vector<int> v3(n);        // 10 zeros
```

Можно задать определенные значения.

```cpp
std::vector<int> v4{1, 2, 3};  // 3 elements: 1, 2, 3
```

Можно сделать одного размера из одинаковых элементов.

```cpp
std::vector<int> v5(n, 239);   // 10 of 239
```

`v.size()` — размер вектора типа `std::size_t`.

```cpp
for (int i = 0; i < static_cast<int>(v1.size()); i++) {
    std::cout << " " << v1[i];

for (std::size_t i = 0; i < v2.size(); i++) {
    std::cout << " " << v2[i];
}
```

### Range-based-for

*range-based-for* — `for (int value : v)` — пробежаться по элементам 
массива. Можно вместо типа писать `auto`.  При этом значения будут копиями 
(если не дописать `&`).

```cpp
for (int value : v3) {  // range-based-for
    std::cout << " " << value;
    // please do not change size of v3, it will be UB
}

for (auto value : v5) {
    // value is a copy, not element of v5
    value += 10;
    std::cout << " " << value;
}
```

Выход за пределы массива — UB. Переполнение.

*Adress Sanitizer* — поиск UB, связанного с обращением по неправильному 
адресу.

```cpp
std::vector<int> vec{1, 2, 3};
// vec[3];  // UB
// vec[100000];  // UB
for (int value = 0; value < 10; value++) {
    bool found = false;
    for (std::size_t i = 0; /*i < vec.size()*/; i++) {
        if (vec[i] == value) {
            found = true;
            // std::cout << i << "\n";
            break;
        }
    }
    // Always found? Enabling cout above changes behavior.
    // Try with Address Sanitizer. But without optimizations.
    std::cout << "value=" << value << " found=" << found << "\n";
}
```

Аналогичный выход за пределы массива  — UB.

```cpp
std::cout << v[-1] << "\n";  // UB, not v[2], unlike Python
```

### **Цикл в обратную сторону**

Аккуратно с беззнаковыми типами!  В данном примере у `i` тип `unsigned` 
из-за `v.size()`. Поэтому компилятор может соптимизировать и не проверять 
условие `i >= 0` —> в какой-то момент `i` станет равным $2^{31}-1$ —> UB.

```cpp
for (auto i = v.size() - 1; i >= 0; i--) {
    std::cout << v[i] << "\n";
}
```

UB для пустого вектора — обращение к -1 элементу.

```cpp
for (auto i = v.size() - 1;; i--) {
    std::cout << v[i] << "\n";
    if (i == 0) {
        break;
    }
}
```

Рабочий вариант.

```cpp
for (auto i = v.size(); i > 0; i--) {
    std::cout << v[i - 1] << "\n";
}
```

Рабочие, но странные варианты.

```cpp
for (auto i = v.size(); i-- > 0;) {
    std::cout << v[i] << "\n";
}   

// for (int i = v.size() - 1; i >= 0; i--) {  // UB if v.empty()
for (int i = static_cast<int>(v.size()) - 1; i >= 0; i--) {
    std::cout << v[i] << "\n";
}
```

## Что-то с практики

### Clang format

```cpp
clang-format -i 01-clang-format.cpp
```

### Compiler warning

```cpp
-Wall -Wextra -Werror
```

### Underfined sanitizer

```cpp
-fsanitize=undefined
```

### Address sanitizer

```cpp
-fsanitize=address
```

### Valgrind

```cpp
valgrind ./main.
```

### Clang-tidy

```cpp
clang-tidy 21-clang-tidy.cpp
```

### Cppcheck

```cpp
cppcheck --language=c++ -DSOME_DEFINE_TO_FIX_CONFIG --enable=all 
--error-exitcode=1 --inline-suppr 22-cppcheck.cpp
```

---
## *19/09/22*

## Массивы

### **Изменение размера вектора**

Можно менять размер динамически при помощи методов `push_back()` и 
`emplace_baсk()` (второй принимает другие аргументы, то, из чего можно 
сконструировать объект. иногда это выгодно, иногда — нет (к примеру, при 
передачи строки сразу сконструируется новый объект, а не будет создавать 
временный, который потом мувнется в вектор, как сделал бы `push_back`, 
подробнее — https://abseil.io/tips/112)).

```cpp
v.push_back(10);  // O(1) amortized
```

Удалить объект из конца при помощи метода `pop_back()`.

```cpp
v.push_back(10);  // O(1) amortize
```

Можно изменять размер вектора при помощи метода `resize()`. Если элементов 
больше, то будут нулями. Если меньше, то обрежутся. Можно также 
передавать, чему сделать равными элементы (при  этом лишь новые элементы 
будут равны этому значению).

```cpp
std::vector<int> v{10};
v.resize(5); // 10 0 0 0 0
v.resize(10, 2);  // 10 0 0 0 0 2 2 2 2 2
v.resize(3, 123);  // 10 0 0
```

Можно очищать вектор при помощи `clear()`.

```cpp
v.clear(); // size = 0
```

### **Ссылки**

В range-based-for можно передавать ссылки на элементы. Тогда они не 
копируюся + их можно менять.

```cpp
for (auto x : v) {
    x += 10;  // modifies a copy
}

// for (int &x : v) {  // also ok
for (auto &x : v) {  // & is 'reference' when attached to a type
    x += 10;
}
```

Ссылку можно сделать константной (`const auto &x`).

```cpp
// for (const int &x : v)  // also ok
for (const auto &x : v) {  // 'const-reference', cannot be modified
    // x += 10;  // compilation error
    std::cout << ' ' << x;
}
```

### **Двумерные вектора**

Вектор векторов (с С++11). Размер — в вектор положить элемент 
`std::vector<int>(3, 123)`.

```cpp
// Before C++11: > > because it looks too much like bitwise shift.
std::vector<std::vector<int>> v(5, std::vector<int>(3, 123));
```

Обращение к элементу `v[i][j]`, **не** через `v[i, j]`.

```cpp
std::cout << v[i][j];  // not v[i, j]
```

Все еще можно использовать range-based-for. Лучше по `&`. Обычно еще и 
`const` (потому что копировать вектор дорого).

```cpp
for (const auto &row : v) {  // no copies! copy allocates memory, which is 
slow
		...
}
```

Аналогично можно делать 3-, 4-, n-мерные вектора…

### **Сравнение векторов**

Можно сравнивать с помощью `<`, `>`, `=`. Сравниваются лексикографически.

```cpp
std::vector v1{1, 2, 3}, v2{4, 5};
std::cout << (v1 == v2) << (v1 < v2) << (v1 > v2) << '\n';  // 010: 
lexicographical comparison: 1 < 4
```

Можно присваивать векторы друг в друга.

```cpp
v1 = v2;  // O(n + m), copies all elements, v1 and v2 are still 
independent
```

### **Assert**

Assert думает, что три аргумента (разделенные запятой). Скобки решают 
(нужна закрывающая).

Макрос из 70х, ура.

```cpp
std::vector v{1, 2, 3};
assert(v == std::vector{1, 2, 3});  // Compilation error because assert is 
a naive macro
assert((v == std::vector{1, 2, 3}));  // OK
assert(v == (std::vector{1, 2, 3}));  // OK
```

### Устройство вектора

Нет срезов. Все элементы одного типа. 

```cpp
Unlike Python:

* No slices `a[1:10:2]`
* Arrays are homogenous: all elements have the same type, known in advance
```

### **Вектор bool**

Сделали сжатие, поэтому хранятся как биты, но так хранить тяжело, поэтому 
много костылей.

`for (auto x : v)` —> меняет вектор!!

```cpp
std::vector<bool> v{false, true, false};
for ([[maybe_unused]] bool x : v) {  // copy as expected
    x = true;
}
for (auto x : v) {  // copy?
    x = true;
}
```

### **Массивы в стили С**

Много тонкостей. Элементы не инициализируются. Нет большинства операций 
вектора. Нельзя присваивать друг в друга. Не константный размер не 
является стандартным —> есть только специальное расширение компилятора 
*variable-length-array (VLA*).

```cpp
int n;
std::cin >> n;

int v1[3], v2[3];  // uninitialized memory
// no push_back, pop_back, resize
v2 = v1;  // compilation error

[[maybe_unused]] int v3[n];  // non-standard extension: 
variable-length-array (VLA)

// Do not use!
```

## Строки

### **Char**

1 символ, арифметический тип, размер обычно 1 байт. Несмотря на символьный 
литерал (`‘’`), легко превращается в число. Можно делать любую арифметику. 

`char - int = int`

`char - char = int`

Любая арифметика типов меньше int конвертируется в int.

```cpp
std::cout << "char: " << sizeof(char)
          << "\n";  // Typically 1 byte, but is treated as "character" for
                    // I/O purposes. Still does arithmetics.
[[maybe_unused]] int wtf = '0';
char c = '0';  // Typically 48 (ASCII table; please do not use non-ASCII),
               // note single quotes.
               // 
https://en.cppreference.com/w/cpp/language/character_literal
std::cout << c << " " << c - 1 /* char - int = int */ << " " << 
static_cast<char>(c - 1) << "\n";
std::cout << static_cast<char>(48) << "\n";
std::cout << '5' - '2' << "\n";  // promoted to `int`. `int - int`

std::cout << '\'' << '\n';  // escaping of special characters: still a 
single int
std::cout << "\"quotes\"\n";  // escaping works inside strings as well
```

Ввод — cin пропускает все пробельные символы и считывает один символ.

```cpp
std::cin >> c;  // Skips whitespaces, reads a single char
std::cout << "char=" << c << " " << static_cast<int>(c) << "\n";
std::cin >> c;  // Skips whitespaces, reads a single char
std::cout << "char=" << c << " " << static_cast<int>(c) << "\n";
// Alternative: std::cin.get(), will be later
```

**Смешное**

Можно `char`’ом задать размер вектора и проинциализировать `int`-овый 
вектор :)

```cpp
std::vector<int> vec1('a', 'b');  // vec2(97, 98);
std::vector<int> vec2{'a', 'b'};
```

### **Signed char**

`signed char`, `char` и `unsigned char` — разные типы. `char` может быть 
отрицательным!

```cpp
char c1 = -10;  // can be either signed or signed, typically signed
unsigned char c2 = c1;  // always unsigned
signed char c3 = c1;  // always signed

std::vector<int> vec(256);
vec[c1] = 5;  // oops, UB
vec[static_cast<unsigned char>(c1)] = 5;  // ok
```

Можно ловить UB на функциях: `isspace` принимает беззнаковые.

```cpp
std::cout << std::isspace(c1) << '\n';  // oops, UB: expects `unsigned 
char` or `int`
```

### **String**

`std::string` лежат в `<string>`. Можно выводить, можно менять 
произвольный символ, можно прибавлять строку.

```cpp
std::string a = "hello";
a += "world";
std::cout << a << '\n';

a[2] = 'X';  // unlike Java
```

Складывать нельзя, но можно добавлять `char`. При этом `a += ‘x’` за O(1), 
a `a = a + ‘x’` — за O(n).

```cpp
// compilation error: legacy, string literal is not really a string
// `auto` here is not `std::string`, legacy
[[maybe_unused]] auto a_no_auto = "hello";

// https://en.cppreference.com/w/cpp/language/string_literal
std::string a_bad = "hello" + "world";
std::string a_good = std::string("hello") + "world";

// ok, but why? Literals only
[[maybe_unused]] std::string a_wtf = "hello" "world";
[[maybe_unused]] std::string a_wtf_ce = "hello" a;  // compilation error
```

Похожа на вектор: есть размер (`size()`, `length()`), можно делать 
`push_back()`, `pop_back()`, `append()`.

```cpp
// Very like vector<char>
a.push_back('Z');
a.pop_back();
std::cout << a.size() << " " << a.length() << "\n";

for (int i = 0; i < 1'000'000; i++) {
    a += 'x';  // push_back, O(1) amortized
}

for (int i = 0; i < 1'000'000; i++) {
    a = a + 'x';  // new temporary string, O(n), clang-tidy warns
}
```

### **Stringstream**

Лежит в `<sstream>`. Можно добавлять туда строки, числа, можно попросить 
вернуть строку `s.str()` (то, что записали).

```cpp
int x = 456;
s << 123 << ' ' << x;
std::string str = s.str();
```

Аналогично можно считывать из него в переменные.

```cpp
std::stringstream s(" -10 -20 ");
int y, z;
s >> y >> z;
```

### **Преобразование в строку**

`std::to_string(10)` — из числа делает строку. `static_cast<std::string>` 
не умеет. 

```cpp
std::string s = std::to_string(10);
// std::string s2 = static_cast<std::string>(10);  // compilation error
```

Из числа строку — `std::stoi("123")`. Также есть `std::stoll(”100”)` (из 
`long long`).

```cpp
int x = std::stoi("123");  // string-to-int, stoll
```

От 0 — UB (причина в legacy, совместимости с языком С).

```cpp
std::string s3 = static_cast<std::string>(0);  // UB, because C++ and 
legacy
```

### **Считывание строк**

`std::cin` —  не считывает пробельные символы, считывает строку из 
непробельных символов.

```cpp
std::cin >> s1;  // skip whitespaces, read non-whitespaces
```

`std::getline(std::cin, s)` — считывает до конца строки.

```cpp
std::getline(std::cin, s2);  // read until end of line
```

### **Считывание по символам**

`std::cin.get()` — считывает один символ (любой), возвращает `int`.

`std::char_traits<char>::eof()` — конец ввода. -1 не всегда.

Сопоставление символу числа зависит от кодировки (какая последовательность 
байт соответствует какому числу).

```cpp
while (true) {
        int c = std::cin.get();
        if (c == std::char_traits<char>::eof()) {  // theoretically can be 
not -1
            break;
        }
        std::cout << c << '\n';
    }
```

### **Считывание до конца ввода**

Неправильно!

```cpp
while (!std::cin.eof()) {  // Never do that: only checks whether EOF is 
already known.
    int x;
    std::cin >> x;  // Reads until the number is ended, not further.
    std::cout << x << "\n";
}
```

Некоторые символы могут интерпретироваться как `eof` (FF —> -1).

```cpp
while (true) {
    char c = std::cin.get();
    if (c == std::char_traits<char>::eof()) {  // Some `char`'s values may 
coincide with `eof()`
        break;
    }
    std::cout << c << '\n';
}
```

Правильно!

```cpp
while (true) {
    int x;
    if (!(std::cin >> x)) {
        break;
    }
    std::cout << x << "\n";
}
```

## Функции

### Базовое

Функция sum возвращает `int`, вызывается от двух параметров `a` и `b` типа 
`int`, которые ведут себя как локальные переменные. Передача параметра по 
значению — `int a`. При этом он копируется внутрь функции.

*Параметр* — локальная переменная функции.

*Аргумент* — то, что указалось в месте вызова — sum(1, 2).

```cpp
int sum(int a, int b) {  // each parameter is a copy of an arguments; 
pass-by-value
    a += b;  // does not modifies arguments
    std::cout << "new a=" << a << "\n";
    return a;
}
```

Если не вернуть значения, то будет UB, но такое ловится санитайзерами.

```cpp
int sum(int a, int b) {
    a += b;
    if (a == 25) {
        return 10;
    }
}

int main() {
    sum(x, 21);  // UB, caught by UBSan   
}
```

### **Void функции**

Можно ничего не возвращать — `void f()`. Можно писать `return`, но это 
необязательно.

```cpp
void print(int a) {
    std::cout << a << '\n';
    if (a == 5) {
        return;
    }
    std::cout << "a is not 5\n";
    return;  // optional
}
```

### **Копирование**

Копирование — это медленно. 

```cpp
int sz(std::string s) {  // pass-by-value
    return static_cast<int>(s.size());
}

int main() {
    const int LEN = 1'000'000;

    std::string s(LEN, 'x');

    for (int i = 0; i < 1'000'000; i++) {
        assert(s.size() == LEN);
    }

    for (int i = 0; i < 1'000'000; i++) { // works but too long
        assert(sz(s) == LEN);
    }
}
```

Решение: передавать по константной ссылке.

```cpp
int sz(const std::string &s) {  // very popular in C++, 
pass-by-const-reference
    // s[0] = 'x';
    return static_cast<int>(s.size());
}
```

При попытке передать временный объект в функцию, принимающей по 
неконстантной ссылке, будет ошибка компиляции. Для константной ссылки все 
ок.

```cpp
void append(std::string &s) {  // pass-by-reference
    s += 'x';
}

void print_appended(const std::string &s) {  // pass-by-const-reference
    std::cout << s << 'x';
}

int main() {
    std::string s = "hello";
    append(s);

    append(s + "world");  // compilation error: temporary object
    append(std::string("world"));  // compilation error: temporary object

    print_appended(std::string("world"));  // ok
}
```

Возвращение значения — тоже копирование.

```cpp
std::string BIG(1'000'000, 'x');

std::string create_string() {
    return BIG;  // return copies; O(n)
}
```

## Что-то с практики

### Redirecting

Запустите приложение с перенаправлением ввода из файла `01-redirect-io.in` 
и вывода в файл `01-redirect-io.out`:

```bash
./01-redirect-io <01-redirect-io.in >01-redirect-io.out
```

Проверьте, что файлы `01-redirect-io.sol` (с правильным ответом) и 
`01-redirect-io.sol` (с выводом программы) не совпали (в первом пять 
пробелов, во втором шесть):

```bash
diff 01-redirect-io.sol 01-redirect-io.out
```

### Подавление предупреждений

Подавить предупреждение Clang-Tidy:

```bash
// NOLINTNEXTLINE(bugprone-branch-clone,имя-второго-предупреждения)
```

Подавить предупреждение cppcheck:

```bash
// cppcheck-suppress [someWarning,anotherWarning]
```

---

## 26/09/22

## Атрибуты

### **[[мaybe unused]]**

`[[maybe_unused]]` — атрибут (пишется в двойных [[]]) с С++17, который 
ставится перед переменными, чтобы обозначить, что они могут не 
использоваться. Зачем? Компилятор может сообразить, когда переменная 
объявлена, но не используется. Тогда при компиляции с предупреждениями 
будет ошибка компиляции. А с `[[maybe_unused]]` не будет.

А иногда хочется скомпилировать программу с предупреждениями, но с 
неиспользуемой переменной. 

Можно также помечать параметры функции. Или куски кода, запускаемые не 
всегда (например, только под windows/linux).

```cpp
/*[[maybe_unused]]*/ int a = 10, b = 20;
a++;
// (b++) = 10;
```

([https://en.cppreference.com/w/cpp/language/attributes/maybe_unused](https://en.cppreference.com/w/cpp/language/attributes/maybe_unused))

### **[[nodiscard]]**

Еще один атрибут, пишется перед функциями, чтобы получить предупреждение 
об неиспользовании результата функции. Зачем? Полезно, если функции 
бессмысленно вызывать и не использовать значение. 

Пример: функция ReadInt, считывающая число. Иногда нужно пропустить число 
из ввода.

```cpp
[[nodiscard]] int sum(int a, int b) {
    return a + b;
}

/*[[nodiscard]]*/ int readInt() {
    int x;
    std::cin >> x;
    assert(std::cin);
    return x;
}

int main() {
    std::cout << sum(2, 3) << "\n";  // makes sense
    sum(5, 6);  // totally useless, we want a warning from the compiler

    readInt();  // skip the number. Kind of useless, [[nodiscard]] is a 
matter of taste
    std::ignore = readInt();  // explicit ignore, non-standard use of 
std::ignore

    sum(readInt(), readInt());  // totally useless, just readInt(), 
readInt()
}
```

([https://en.cppreference.com/w/cpp/language/attributes/nodiscard](https://en.cppreference.com/w/cpp/language/attributes/nodiscard))

## Структуры и классы

### Базовое про классы

Заводим свой собственный тип, состоящий из названия и членов (в `{}`). 

Члены бывают полями (*member*), которые содержатся в каждом экземпляре 
класса, и функциями (методы, *member functions*), которые вызываются 
только на экзепляре класса.

Обращение к полям — через `.`. Поля инициализируются как обычные 
переменные (в данном случае `int` и `int`). Вызов метода аналогично через 
`.`.

```cpp
Point p;
// std::cout << p.x << " " << p.y << "\n";  // UB: not initialized
p.x = 10;
std::cout << p.dist2() << "\n";
```

### Перегрузка операторов (operator overload)

Можно перегружать операторы (*operator overload*). Работает только для 
любых встроенных в С++, операторов (новые создавать нельзя).

```cpp
struct Point {
    // members
    int x, y;  // default initialization (e.g. nothing for `int`, empty 
`vector`)

    // member function
    int dist2() {
        return x * x + y * y;
    }

    void operator+=(const Point &other) {  // operator overload
        x += other.x;
        y += other.y;
    }
};  // important semicolon!

void foo([[maybe_unused]] Point p) {
}
```

Синтаксис: `void operator+=(Point other)`. Левый операнд — то, на ком 
вызывается, а правый передается как аргумент.

Результат: `p += p1` (что тоже самое, что и `p.operator+=(p1)`).

```cpp
p += p2;
// p.operator+=(p2);
```

### **Агрегаты (aggregates)**

“Простая” структурка — есть поля, может какие-то методы, нет всяких 
хитростей.

```cpp
struct Point {
    int x, y;

    int dist2() {
        return x * x + y * y;
    }
};
```

Агрегаты можно инициализировать прямо при объявлении фигурными скобками. 
При этом все поля инициализируются гарантированно. А для `Point p;` — так 
не работает, при чтении переменных структуры будет UB.

```cpp
// aggregate initialization
// Works with "aggregates", no details from me, changes between standards.
// "Simple class": public members.
[[maybe_unused]] Point p1{};  // implicit init: x=0, y=0
[[maybe_unused]] Point p2{3};  // x = 3, warning about missing 'y' 
initializer
[[maybe_unused]] Point p3{3, 4};  // x = 3, y = 4
```

### Structured binding

Можно разобрать простую структуру на составляющие — `auto [x, y] = p` — 
*structured binding* (названия переменных не важны, гланое, чтобы 
количество совпадало). Работает с парами, массивами, кортежами и так 
далее.

```cpp
// Can disassemble some "simple classes" (not aggregates) as well.
// It is called "structured binding".
auto [x, yyy] = p3;
```

### **Константность (**const qualified**)**

Функция print принимает константую ссылку (чтобы не копировать). Не можем 
менять поля, а также, оказывается, не можем вызывать методы, непомеченные 
`const` (*const qualified*) —> будет ошибка компиляции. Даже если они 
ничего не меняют, так как по умолчанию предполагается, что любой метод 
может менять структуру.

Вывод: если можно пометить метод `const` — помечать.

```cpp
int dist2() const {  // Add `const` everywhere when possible
    // x++;
    return x * x + y * y;
}
```

Почему С-style это плохо: можно сделать так `Point &p2 = (Point&)(p);` и 
потом вызвать *неконстантный* метод. Можно так делать, но только в том 
случае, если исходный объект не был сам константным —> UB.

При этом `Point &p3 = static_cast<Point&>(p);` не скомпилировалось бы. 
Есть специальный `const_cast<..>`, но по нему видно, что он отбрасывает 
константность.

```cpp
void print(const Point &p) {
    std::cout << p.x << " " << p.y << "\n";
    std::cout << p.dist2() << "\n";
}

int main() {
    Point p;
    p.x = 10;
    p.y = 20;
    print(p);  // OK

    const Point const_p{30, 40};
    print(const_p);  // OK
}
```

### И**нициализации полей (field initialization)**

Способ 1: при объявлении поля сказать, чему оно будет равно по умолчанию.

```cpp
struct ratio {
    int num = 0;  // C++11: member default initialization
    int denom = 1;
};

ratio r;
ratio r2{3, 4};
```

Способ 2: создать конструктор. Название совпадает с названием структуры.

Конструктор по умолчанию (*default constructor*) не имеет аргументов. 
Вызывается, например, когда создали переменную или написали `()`/`{}`.

```cpp
ratio() {  // default constructor
    num = 0;
    denom = 1;
}

ratio r;  // default initialization
ratio r2{};  // default initialization/list initialization
```

Конструктор с аргументами  — `ratio(a)` (*direct initialization)* и 
`ratio{a}` (*direct list initialization)*.

```cpp
ratio(int value) {
    num = value;
    denom = 1;
}

ratio r(10);  // direct initialization
ratio r2{10};  // direct list initialization
```

Что раньше: оператор `<<` или конструктор? С С++17 все определено.

```cpp
// Who is completed first: first << or constructor?
// C++17: behavior is defined;
// Before C++17: behavior is implementation-defined(???)
std::cout << "a>" << ratio{10}.num << "<\n";
```

Способ 3: *member initialization list*. Тогда компилятор не будет 
инициализировать переменные 0 и 1, а сразу вызовет конструктор.

```cpp
// Better to initialize correctly right away than to reassign.
ratio() {}
ratio(int value) : num(value) {}
ratio(int value, int value_denom) : num(value), denom(value_denom) {}
```

Если есть хотя бы один конструктор, то нельзя писать `ratio r` — нет 
конструктора по умолчанию.

```cpp
struct ratio {
    int num = 0;
    int denom = 1;
    // no default constructor is autogenerated
    ratio(int value) : num(value) {}
};
```

### **Public**

`public` написан в начале каждой структуры — кто угодно может обращаться к 
внутренним полям и методам. Но у структуры может быь какой-то инвариант, 
который пользователь может нечаянно сломать, как быть? 

Решение —> поменять уровень доступа с помощью слов `public` и `private`.  
Внутри могут быть как поля, так и методы.

```cpp
struct ratio {
private:
    ...
public:
		...
};
```

### **Private**

`private` — доступ к полям есть только у методов структуры.

Чтобы считать поля пишут геттеры — специальные методы, возвращающие 
значения приватных полей.

```cpp
// getter
int numerator() const {
    return num;
}
int denominator() const {
    return denom;
}
// setter: no need actually
void numerator(int new_value) {
    num = new_value;
}
void denominator(int new_value) {
    assert(new_value != 0);
    denom = new_value;
}*/
```

### **Friend**

Можно объявить функцию `friend`’ом. Тогда функция получает доступ к 
приватным полям. При этом неважно, какая функция (`private`/`public`).

```cpp
private:  // private, public, does not matter, visible to all
    friend bool eq(const Foo &a, const Foo &b) {
        return a.field == b.field;
    }
```

### **Разница между struct и class**

Разница в том, что у `class` по умолчанию написано `private`,  а у 
`struct` — `private`.

```cpp
struct Foo {  // public:
    int x = 10;
};

class Bar {  // private:
// public:
    int x = 10;
};

// Different teams have different conventions.
```

## Преобразования (conversions)

### Неявные преобразования (implicit conversion)

Если есть конструктор, то подключаются неявные преобразования (*implicit 
conversion*):

Копирующая инициализации, где создаются объекты типа ratio и вызывается 
нужный конструктор.

```cpp
struct ratio {
    int num = 0;
    int denom = 1;
    ratio() {
        std::cout << "Default constructor\n";
    }
    ratio(int value) : num(value) {
        std::cout << "ratio(int)\n";
    }
};

void println(const ratio &r) {
    std::cout << r.num << " " << r.denom << "\n";
}

ratio generate_ratio() {
    return 123;  // 3. copy initialization: a new 'ratio' is created
}

int main() {
    ratio r(10);  // direct initialization
    ratio r1 = 10;  // 1(a). copy initialization, it has '='
    ratio r2 = {10};  // 1(b). list copy initialization, it has '='.

    println(r);  // no new objects
    println(ratio(10));  // direct initialization of temporary
    println(ratio{10});  // direct list initialization of temporary
    println(10);  // 2(a). copy initialization: a temporary is created
    println({10});  // 2(b). list copy initialization

    println(generate_ratio());

    int x = 20;
    ratio r3 = x;  // copy initialization
    r3 = 40;
    println(r3);
}
```

## **Когда преобразования — это плохо**

Возможен вызов функции, где `int` неявно преобразуется в vector — ???

```cpp
struct my_vector {
    my_vector() {}
    my_vector(int) {}
};

void print_vector([[maybe_unused]] const my_vector &vec) {
}

int main() {
    [[maybe_unused]] my_vector v1(10);
    [[maybe_unused]] my_vector v2 = 10;  // copy initialization, should 
not work
    print_vector(10);  // copy initialization, should not work
}
```

### Явные преобразования (e**xplicit conversions)**

`explicit` — запрещает неявные образования в копирующей инициализации, но 
разрешает прямую.

```cpp
explicit my_vector(int) {}
```

*direct initialization* —> вызывает и explicit, и implicit конструкторы.
*copy initialization* —> вызывает только implicit конструкторы.

```cpp
struct my_vector {
    my_vector() {}
    // explicit constructors do not participate in copy initialization.
    // direct initialization ~ explicit + implicit
    // copy initialization ~ implicit
    explicit my_vector(int) {}
};

void print_vector([[maybe_unused]] const my_vector &vec) {
}

int main() {
    [[maybe_unused]] my_vector v1(10);

    // [[maybe_unused]] my_vector v2 = 10;
    // print_vector(10);

    // cast is explicit + copy initialization from vector
    [[maybe_unused]] my_vector v2 = static_cast<my_vector>(10);
    // cast is explicit + copy initialization
    print_vector(static_cast<my_vector>(10));
    // direct initialization of temporary
    print_vector(my_vector(10));
}
```

## Операторы (operators)

### **Операторы преобразования**

Чтобы можно было неявно преобразовывать из нашего типа к стандартному, 
нужно написать соответвующий метод (оператор преобразования): `operator 
double(){ ..}`. Лучше сделать `const`. 

```cpp
operator double() const {
    std::cout << "operator double()\n";
    return num * 1.0 / denom;
}

ratio r{3, 4};
double x = r;  // implicit conversion
double x1 = r.operator double();
double x2 = static_cast<double>(r);  // explicit conversion
std::cout << r << "\n";  // implicit conversion to double
```

Такие операторы также можно делать явными, дописав к ним `explicit`. Тогда 
нужен будет `static_cast<..>`.

```cpp
explicit operator double() {
    std::cout << "operator double()\n";
    return num * 1.0 / denom;
}

ratio r{3, 4};
// double x0 = r;  // compilation error
double x = static_cast<double>(r);
double x2(r);
double x3 = double(r);
std::cout << x << "\n";
std::cout << static_cast<double>(r) << "\n";
```

А что если все смешать? В данном случае компилятор скажет, что 
преобразование неоднозначно.

```cpp
// Does not compile because of 'reasons'
struct Bar;

struct Foo {
    operator Bar();
};

struct Bar {
    Bar() {}
    Bar(Foo /*arg*/) {}
};

Foo::operator Bar() {
    return Bar{};
}

int main() {
    Foo f;
    Bar b = f;  // ambiguous
}
```

### Оператор сравнения (operator==)

`b == 4` компилируется, а `4 == b` — нет.

Правило: если оператор написан не симметрично (`bool operator==(const 
bigint &other)`), то у левого аргумента преобразований нет.

```cpp
struct bigint {
private:
    std::vector<int> digits;

public:
    bigint() {}
    bigint(int digit) : digits{digit} {
        assert(0 <= digit && digit < 10);
    }
    // Fun fact: this is wrong algorithmically. Guess how?

    bool operator==(const bigint &other) const {
        return digits == other.digits;
    }
};

int main() {
    bigint a = 4, b = 5;
    std::cout << (a == b) << "\n";
    std::cout << a.operator==(b) << "\n";
    std::cout << (a == 6) << "\n";  // 6 is implicitly converted
    std::cout << (a == 4) << "\n";  // 4 is implicitly converted
    std::cout << (4 == b) << "\n";  // does not convert 4 to bigint
}
```

Решение: сделать оператор `friend`’ом, синтаксис немного меняется — 
`friend bool operator==(const bigint &lhs, const bigint &rhs)`.

Не самое оптимальное — нужно на каждое сравнение нужно создавать новый 
вектор, но что поделать.

```cpp
#include <cassert>
#include <iostream>
#include <vector>

struct bigint {
private:
    std::vector<int> digits;

public:
    bigint() {}
    bigint(int digit) : digits{digit} {
        assert(0 <= digit && digit < 10);
    }

    // left-hand side, right-hand side
    friend bool operator==(const bigint &lhs, const bigint &rhs) {
        return lhs.digits == rhs.digits;
    }
};

int main() {
    bigint a = 4, b = 5;
    std::cout << (a == b) << "\n";
    std::cout << operator==(a, b) << "\n";
    std::cout << (a == 6) << "\n";  // 6 is implicitly converted
    std::cout << (a == 4) << "\n";  // 4 is implicitly converted
    std::cout << (4 == b) << "\n";  // 4 is implicitly converted
}
```

### **Оператор += (operator+=)**

`bigint &operator+=(const bigint &other)` — по конвеции выглядит так. 
Возвращает ссылку на текущий объект (`this`).

```cpp
bigint &operator+=(const bigint &other) {
    digits[0] += other.digits[0];
    return *this;  // should return reference to 'this' object,
                   // `*this` is magic at the moment.
}
```

### **Оператор ++ (operator++())**

`bigint &operator++()` — префиксный (без аргументов), возвращает новое 
значение. `++a` это тоже самое, что и `a.operator++()`.

`bigint operator++(int)` — постфиксный (с аргументами), возвращает старое 
значение. `a++` это тоже самое, что и `a.operator++(0)`.

```cpp
bigint &operator++() {  // returns the new value, this object
    digits[0]++;
    return *this;
}

bigint operator++(int) {  // returns old value, should be a copy
    bigint old = *this;
    digits[0]++;
    return old;
}

++a;  // a.operator++()
a++;  // a.operator++(0)
```

### **Выражение операторов друг через друга**

```cpp
bigint &operator+=(const bigint &other) {
    return *this = *this + other;
}

bigint operator+(const bigint &other) const {
    return bigint(*this) += other;  // make a copy, alter it
}

bigint &operator-=(bigint &lhs, const bigint &rhs) {
		return lhs = lhs - rhs;
}

bigint operator-(bigint lhs, const bigint &rhs) {
		return lhs -= rhs;  // lhs is a copy already
}
```

### Оператор вывода (operator<<)

`friend std::ostream &operator<<(std::ostream &os, const bigint &b)` — 
оператор вывода. Возвращает поток, который ему передали.

```cpp
// std::ostream& is generalization of std::stringstream and std::cout,
// see 'inheritance' later.
friend std::ostream &operator<<(std::ostream &os, const bigint &b) {
    os << b.digits[0];
    // Less naive, not required in lab: 
https://en.cppreference.com/w/cpp/named_req/FormattedOutputFunction
    return os;
}

std::cout << a << "\n";
(std::cout << a) << "\n";
// (  std::cout.operator<<(a) ).operator<<("\n");
// operator<<(  operator<<(std::cout, a)  , "\n");
// operator<<(             std::cout      , "\n");
```

### Оператор ввода (operator>>)

`friend std::istream &operator>>(std::istream &is, bigint &b)` — оператор 
ввода.

```cpp
friend std::istream &operator>>(std::istream &is, bigint &b) {
    is >> b.digits[0];
    // Less naive, not required in lab: 
https://en.cppreference.com/w/cpp/named_req/FormattedInputFunction
    return is;
}

std::cin >> a >> b;
(std::cin >> a) >> b;
```

---
## 03/10/22

## Define-declare

### **Идентификаторы**

Идентификатор — название для структуры, перменной, чего угодно.

 `_t` — зарезервированные имена, может использоваться POSIX’ом.

Что угодно, начинающееся с `_` — UB.

Что-то с названием, являющимся ключевым словом, не компилируется. Что-то, 
начинающееся с цифры, тоже.

Нельзя использовать имена, содержащие `__` или `_[заглавная буква]`.

```cpp
// https://stackoverflow.com/a/228797/767632
// https://codeforces.com/blog/entry/17747

// int true = 10;  // No keywords
// int 10fff = 20;  // Not start with a digit

int _;  // UB
int _z;  // UB
int z_;  // ok
int x__a = 30;  // UB
int _Xx;  // UB

struct foo_t {  // '_t', not UB, but reserved by POSIX.
};

int main() {
    int _, _x;  // OK, but no.
    int x__b = 30;  // UB
    int _Xy;  // UB
}
```

Компилятор читает файл сверху вниз.

Если функция используется до своего определения, то можно ее объявить 
(вместо тела `{}` пишется `;`), а потом ее определить.

Не только для циклических функций, но и для объявления функций в других 
файлах.

```cpp
void bar(int n);  // declaration, объявление

void foo(int n) {  // definition, определение
    std::cout << "foo " << n << "\n";
    bar(n - 1);
}

void bar(int n) {  // definition, определение
    std::cout << "bar " << n << "\n";
    if (n == 0) {
        return;
    }
    foo(n - 1);
}
```

Внутри классов все хорошо, методы видят друг друга (компилятор сначала 
читает определение класса, а потом только разбирается, как что работает).

```cpp
struct Foo {
    void foo() {
        bar();
    }

    void bar() {
        foo();
    }
};
```

### **Incomplete type and forward declaration**

*forward declaration* — объявить структуру до ее определения. При этом 
нельзя класть ее как поле (неясно, какой размер у неполного типа; 
неизвестно, какие конструкторы, какие методы у структуры —> ошибка 
компиляции).

```cpp
// https://en.cppreference.com/w/cpp/language/type
struct Bar;  // forward declaration

struct Foo {
    // Bar b;  // compilation error: incomplete type
    std::vector<Bar> bs;

    void doit() {
        bs.push_back(Bar());
        bs.resize(10);
        Bar b2;
        b2.bar_do();
    }
    friend void foo_friend() {
        Bar b3;
    }
};

struct Bar {
    Foo f;
};
```

Можно объявить метод внутри структуры, а реализовать его после определения 
структуры —> внутри — declaration, снаружи — definition с синтаксисом 
`Foo::doit` (приписать, к какому классу относится метод). Аналогично можно 
с конструкторами, операторами и `friend` функиями (которые выносятся без 
слова `friend` и без `Foo::`).

```cpp
// https://en.cppreference.com/w/cpp/language/type
struct Bar;  // forward declaration

struct Foo {
    std::vector<Bar> bs;

    void doit();  // declaration
    friend void foo_friend();  // declaration
};

struct Bar {
    Foo f;
};

void Foo::doit() {  // definition: `Foo::` is important
    bs.push_back(Bar());
    bs.resize(10);
    Bar b2;
    bs[0].bar_do();
}

// friend functions are usual global functions with special handling
// two classes cannot have the same friend function
void foo_friend() {  // definition: `Foo::` for friends is not needed
    Bar b3;
}
```

Рекурсия: объявляем структуру, объявляем оператор, определяем оператор 
потом.

```cpp
struct Bar;  // incomplete type

struct Foo {
    operator Bar();
};

struct Bar {
    Bar() {}
    Bar(Foo /*arg*/) {}
};

Foo::operator Bar() {
    return Bar{};
}

int main() {
    Foo f;
    Bar b = f;  // ambiguous
}
```

### The most vexing parse

Запрещает писать круглые скобки в некоторых местах. Например, у int’а не 
будет вызова конструктора по умолчанию (похоже на объявление функции). 
Аналогично у vector’а. Результат: `f.x = 10` не скомпилируется; 
полагается, что `v3` — функция.

```cpp
// The most vexing parse
struct Foo {
    int x{};  // ok: default initialization, i.e. 0
    int y();  // the most vexing parse: function?????
};

int main() {
    Foo f;
    f.x = 10;
    f.y = 10;

    std::vector<int> v1{};
    std::vector<int> v2(10);
    std::vector<int> v3(); // is it fwd of function?
    std::vector<int> v4;
}
```

Грустнее: `Foo f(Bar())` — компилятор подумал, что это функция, 
принимающая функцию…

Как бороться: заменить какие-то `()` на `{}` (можно вообще все по 
максимуму инициализировать ими) или `(Bar())`.

```cpp
struct Bar {
    explicit Bar() {}
};
struct Foo {
    explicit Foo(Bar) {}
};

struct Botva {
    Foo f(Bar());  // oops, function
    // TODO

    void foo() {
        f = f;
    }
};

int main() {
    Foo f(Bar());  // the most vexing parse: Foo f(Bar (*arg) ())
    // Foo f{Bar()};
    // Foo f(Bar{});
    // Foo f{Bar{}};
    // Foo f((Bar()));  // C++03 and before
    // Foo f(   (Bar())  );  // C++03 and before
    f = f;
}

// Foo f( (int x) ) {}
```

## Пространства имен (namespaces)

### **Базовое**

Хочется, чтобы в разных кусках программы можно было называть функции 
одинаково. Помогают namespace’ы —> namespace ns1 { } // namespace ns1

Изначально находимся в глобальном namespace’е.

`::foo()` (явно указали, где искать фукцию) или `foo()` (объяснение 
дальше) — вызов функции из глобального namespace.

`ns1::foo()` — вызов функции из namespace ns1.

В стандартной библиотеке все функции завернуты в namespace std, поэтому 
при ихвызове надо дописывать `std::`.

```cpp
void foo() {
    std::cout << "foo global\n";
}

namespace ns1 {
int x;

void foo() {
    std::cout << "foo1\n";
}
}  // namespace ns1

namespace ns2 {
void foo() {
    std::cout << "foo2\n";
}
}  // namespace ns2

namespace ns3 {}

int main() {
    foo();  // unqualified name lookup
    ns1::foo();  // unqualified name lookup + qualified
    ns2::foo();
    ns1::x = 10;
    // ns3::foo();  // compilation error
}
```

### **Правило разрешения имен (name resolution)**

*name resolution* — поиск имен функций. Если не указано, в каком namespace 
искать, то сначала смотрим в текущем, а потом в родительском.

```cpp
void foo() {
    std::cout << "foo global\n";
}

void some_global() {
    std::cout << "some_global\n";
}

namespace ns1 {
void bar() {
    std::cout << "bar\n";
}

void foo() {
    std::cout << "foo1\n";
    bar();  // unqualified name lookup for 'bar': look at ns1, ::
    some_global();  // unqualified name lookup: look at ns1, ::

    // same, becase 'ns1' is only in the global namespace
    ns1::bar();  // unqualified name lookup for 'ns1': look as ns1, :: 
(found).
                 // qualified name lookup for 'bar' inside 'ns1': found
    ::ns1::bar();  // qualified name lookup for 'ns1' inside ::
                   // qualified name lookup for 'bar' inside 'ns1': found

    // different, because 'foo' exists both in 'ns1' and global
    foo();  // unqualified name lookup: look at ns1 (found, infinite 
recursion)
    ::foo();  // qualified name lookup with global namespace
}
}  // namespace ns1

int main() {
    foo();
    ::foo();
    ns1::foo();
    ::ns1::foo();
}
```

### **Вложенные namespace’ы**

*unqualified name lookup* — если не указано, в каком namespace лежит.

*fully qualified lookup* — ::ns1::ns2::bar();  

```cpp
#include <iostream>

void foo() {
    std::cout << "foo global\n";
}

void some_global() {
    std::cout << "some_global\n";
}

namespace ns1 {
void bar() {
    std::cout << "ns1::bar()\n";
}

namespace ns2 {
void bar() {
    std::cout << "ns1::ns2::bar\n";
}
void botva_ns2() {
    std::cout << "ns1::ns2::botva_ns2()\n";
}
}  // namespace ns2

namespace ns3 {
void botva_ns3() {
    some_global();  // ok
    ns2::botva_ns2();  // ok, find 'ns2' first
    // botva_ns2();  // only looks up in ns1::ns3, ns1::, ::, not ns1::ns2
    std::cout << "ns1::ns3::botva_ns3()\n";
}
}  // namespace ns3

namespace ns3::ns4 {
namespace ns1 {  // ns1::ns3::ns4::ns1
}

void baz() {  // ns1::ns3::ns4
    botva_ns3();  // unqualified name lookup for 'botva_ns3'
    ns2::bar();  // unqualified name lookup for 'ns2', qualified name 
lookup bar()

    foo();  // ok, global
    // ns2::foo();  // compilation error: no 'foo' inside 'ns2'

    ::ns1::ns2::bar();  // fully qualified lookup
    // ns1::ns2::bar();  // thinks that 'ns1' is 'ns1::ns3::ns4::ns1', 
'n2' not found
}
}  // namespace ns3::ns4
}  // namespace ns1

int main() {
    ns1::ns3::ns4::baz();
    ::ns1::ns3::ns4::baz();

    ::ns1::ns2::bar();  // ok
    ns1::ns2::bar();  // ok here, but not in ns1::ns3::ns4::baz()
}
```

### **Argument-Dependent Lookup aka Koenig Lookup (ADL)**

Иногда можно вызвать функцию, не указывая явно, из какого namespace она 
вызывается. При вызове от параметров компилятор переберет их все, 
посмотрит на их типы и заглянет в namespace’ы этих типов.

Зачем? Чтобы писать `f == f` вместо `ns_parent::ns::operator==(f, f)`.

```cpp
// Argument-Dependent Lookup aka Koenig Lookup
#include <algorithm>
#include <vector>

namespace ns_parent {
namespace ns {
struct Foo {};

void do_something() {}
void do_something(Foo) {}
bool operator==(const Foo&, const Foo&) { return true; }
};

void do_other(ns::Foo) {}
}

int main() {
    // do_something();
    ns_parent::ns::do_something();

    // Foo f;
    ns_parent::ns::Foo f;

    f == f;
    operator==(f, f);  // ok: unqualified lookup looks in argument's 
type's namespaces as well (ns_parent::ns)
    ns_parent::ns::operator==(f, f);

    ns_parent::ns::do_something(f);  // ok
    do_something(f);  // unqualified name lookup, ADL enabled

    ns_parent::do_other(f);  // ok
    // do_other(f);  // do_other is in another namespace, no ADL

    // Example:
    // getline(std::cin, str)
    // Better: std::getline

    std::vector<int> v{1, 2, 3};
    sort(v.begin(), v.end());  // v.begin() ~ std::vector<int>::iterator 
~(?) int*
    // Better: std::sort
}
```

## Программы из нескольких файлов

### Translation units

Программа состоит из нескольких единиц трансляции (в быту — файлов с 
раширением `.cpp`).

Зачем:

- Не такие длинные файлы.
- Компиляция параллелится.
- Перекомпиляция ускоряется: меньше файлов перечитывать.
- Можно собрать несколько программ из разных кусочков и переиспользовать 
результаты компиляции.
    - Например, отдельно код, отдельно тесты к коду.

Дальше можно делать статические библиотеки (набор `.o`, собранный вместе) 
и динамические. Статические вкомпилированы в исполянемый файл, 
динамические подгружаются ОС в процессе выполнения (и тогда их можно иметь 
общие между разными программами).

Файлы:

- main.cpp
    
    ```cpp
    // Translation unit 1.
    
    // No <iostream>: we do not use it in main() at all.
    void print_hello_please();
    
    int main() {
        print_hello_please();
    }
    ```
    
- print_hello.cpp
    
    ```cpp
    // Translation unit 2.
    
    #include <iostream>
    
    void print_hello_please() {
        std::cout << "Hello World-2!\n";
    }
    ```
    

Компилирование:

```cpp
g++ main.cpp print_hello.cpp -o main-01
```

Можно компилировать по отдельности, а потом слинковать вместе:

```cpp
g++ -c main.cpp -O2 -DDEBUG -std=c++17
g++ -c print_hello.cpp -O2 -DDEBUG -std=c++17
g++ main.o print_hello.o -O2 -std=c++17 -o main-02
```

Удобно, если меняется только один файл. Можно даже хранить по отдельности 
— компилятор считает единицы трансляции независимыми.

! gcc и g++ не отличаются в плане компиляции, но отличаются в плане 
линковки (gcc подгружает только стандартную библиотеку c).

### **Структуры в нескольких файлах**

Определение структур доджно быть идентичным во всех файлах, иначе — *IFNDR 
(Ill-Formed, No Diagnostic Required)*.

Файлы:

- main.cpp
    
    ```cpp
    #include <vector>
    #include <iostream>
    
    // Should have definition. Method are supposedly defined outside of 
the struct.
    struct Foo {
        int a = 10;  // Should be exactly 10, IFNDR otherwise.
        std::vector<int> v;
    
        void method();
        void method_unused();
    };
    
    Foo get_foo();
    
    int main() {
        get_foo().method();
    
        Foo f = get_foo();
        f.method();
    
        Foo f2;
        f2.method();
    }
    ```
    
- foo.cpp
    
    ```cpp
    #include <vector>
    #include <iostream>
    
    struct Foo {
        int a = 10;
        std::vector<int> v;
    
        void method();
        void method_unused();
    };
    
    Foo get_foo() {
        return Foo{};
    }
    
    void Foo::method() {
        std::cout << "method() called " << a << "\n";
    }
    
    void Foo::method_unused() {
        std::cout << "method_unused() called " << a << "\n";
    }
    ```
    

Можно определять методы внутри, но опять же надо делать это идентично. 
Если снаружи, то метод должен быть написан в одном файле!

Файлы:

- main.cpp
    
    ```cpp
    #include <iostream>
    #include <vector>
    
    // Should have definition. Method are supposedly defined outside of 
the struct.
    struct Foo {
        int a = 10;
        std::vector<int> v;
    
        void method() {
            std::cout << "method() called " << a << "\n";
        }
    };
    
    Foo get_foo();
    
    int main() {
        get_foo().method();
    
        Foo f = get_foo();
        f.method();
    }
    ```
    
- foo.cpp
    
    ```cpp
    #include <vector>
    #include <iostream>
    
    struct Foo {
        int a = 10;
        std::vector<int> v;
    
        void method() {  // Definition inside struct: not ODR-violation, 
if the same everywhere.
            std::cout << "method() called " << a << "\n";
        }
    };
    
    Foo get_foo() {
        return Foo{};
    }
    ```
    

### Name mangling

В одной файле функция возвращает double, а в другом — float.

- about
    
    ```cpp
    API (Application Programming Interface) void foo(); foo();
    
    Gets converted by the compiler to:
    
    ABI (Application Binary Interface) '_Z3foov' --> (name mangling)
    
    But return value type is not mangled.
    
    Different compilers/flags => different ABIs => UB. ABIs are 
incompatible and not checked. E.g.: size of std::size_t, int*, int, fields 
inside std::vector's implementation... long vs int
    ```
    

Файлы:

- get.cpp
    
    ```cpp
    float get() {
        return 1'234'567'890;
    }
    ```
    
- main.cpp
    
    ```cpp
    #include <iostream>
    
    double get();  // UB: return type differs from get.cpp
    
    // IFNDR: Ill Formed, No Diagnostic Required
    
    int main() {
        std::cout << get() << "\n";
    }
    ```
    

Исторически компилятор не париться по поводу возвращемого значения (в С 
функция могла быть определена одним образом, по ее имнеии можно было 
понять, что это за функция).

В ABI не учитывается возвращаемое значение.

### **One Definition Rule (ODR)**

*ODR:* любой программе на С++ должно быть ровно одно определение каждой 
сущности.

Также относится к именам из стандартной библиотеки.

Пример: все хорошо компилируется, но если добавить ключ `-static` 
(затаскивает всю стандартную библиотеку), то перестанет компилироваться 
(есть функция такая под linux). Как защититься? А без понятия…  

- example
    
    ```cpp
    #include <iostream>
    
    int write;  // how to solve: namespaces
    
    int main() {
        std::ios_base::sync_with_stdio(false);
        std::cout << 10;
    }
    ```
    

## Include

### Header structure

Чтобы избежать многочисленного копирования, можно создать заголовочный 
файл с расширением `.hpp`, где будет лежать опеределения структуры. И 
вместо копирования можно писать  `#include “... .hpp”`.

Файлы:

- foo.cpp
    
    ```cpp
    #include <vector>
    #include <iostream>
    #include "foo.hpp"
    
    Foo get_foo() {
        return Foo{};
    }
    
    void Foo::method() {
        std::cout << "method() called " << a << "\n";
    }
    
    void Foo::method_unused() {
        std::cout << "method_unused() called " << a << "\n";
    }
    ```
    
- foo.hpp
    
    ```cpp
    struct Foo {
        int a = 10;
        std::vector<int> v;
    
        void method();
        void method_unused();
    };
    
    Foo get_foo();
    ```
    
- main.cpp
    
    ```cpp
    #include <vector>
    #include <iostream>
    #include "foo.hpp"
    
    int main() {
        get_foo().method();
    
        Foo f = get_foo();
        f.method();
    
        Foo f2;
        f2.method();
    }
    ```
    

*Include guards* — файл можно добавлять несколько раз, но он добавится 
только один, в самый первый раз (чтобы избежать многочисленного 
определения).

---

## 10/10/22

### ODR

Если функции имеют разный набор аргументов (*перегрузки функций*), то ODR 
не нарушается.

```cpp
// These are two independent overloads.
void foo(int);
void foo() {
}

int main() {
    foo();
    foo(10);
}
```

### Линковка со структурами

Если структуру, определенную в другой единице трансляции, необходимо 
использовать, то придется копировать ее определение (один в один! нельзя 
удалить даже неиспользуемый метод) из одного файла в другой.

Структура может быть определена сколько угодно раз, но все определения 
должны быть одинаковыми.

Если в структуре определены какие-то методы, то их определение тоже 
придется копировать.

(!) если поменять местами поля, то все плохо (например, если метод 
реализован в другой единице трансляции, то будет попытка считать не те 
байты структуры и вместо vector считается int или наоборот или удалить не 
те поля).

```cpp
struct Foo {
    int a = 10;  // Should be exactly 10, IFNDR otherwise.
    std::vector<int> v;

    void method();
    void method_unused();
};
```

### Incomplete type

Вместо реализации структуры можно определить неполный тип.

Но нельзя копировать (вдруг структуру нельзя копировать), нельзя 
определить метод на ссылке (вдруг его нет), нельзя определить переменную 
данного типа (неясно, сколько байт выделять).

Если от структуры нужна только передача по ссылке, то это идеальное 
решение.

```cpp
Foo& get_foo();
void call_method(Foo &f);
```

### #pragma once

Аналогично include guars, header включится один раз.

### Цикл из заголовков

Циклических зависимостей не бывает (из-за определения один раз), поэтому 
нужно смотреть, как друг на друга ссылаются файлы, возможно, где-то хватит 
неполного типа.

### Forward include

Определение неполных типов из соответвующих hpp-шников.

```cpp
#ifndef BAR_FWD_HPP_
#define BAR_FWD_HPP_

struct Bar;

#endif  // BAR_FWD_HPP_
```

Пример из стандартной библиотеки:

```cpp
#ifndef FOO_H_
#define FOO_H_

#include <iosfwd>  // smaller than <iostream>, only forward declarations

struct Foo {};

std::ostream& operator<<(std::ostream&, const Foo&);

#endif  // FOO_H
```

### ODR  в заголовках

Если включить заголовок в файл с реализацией какой-то фукнцией, тип которй 
отличается от определния, то вместо UB будет ошибка компиляции.

```cpp
#include "get.hpp" // --> double get();

float get() {
    return 1'234'567'890;
}
```

### Static Initialization Order Fiasko (SIOF)

Инициализация глобальных переменных в разных файлах не гарантирована! 
Поэтому плохо, если программа завязана на порядке инициализации 
переменных.

На самом деле, даже cout может оказаться не проинициализированным (для 
него решение — объект std::ios_base::Init, который гарантирует, что cout 
будет проинициализирован).

### static

Проинициализируется при первом проходе по файлу.

```cpp
Foo &get_foo() {  // construct-on-first-use
    static Foo f;  // like global variable, but initialized at first 
execution of this line
    return f;
}
```

### Unnamed namespace

Если в разных файлах есть функция с одинаковым названием. Особенно, если 
какая-то функция является внутренней функцией файла. Тогда при линковки не 
возникнет проблем, ничего наружу торчать не будет.

Поэтому при написании программ из нескольких файлов лучше по максимуму 
оборачивать все в unnamed namespace.

```cpp
// "unnamed namespace"
namespace {
// internal linkage
void foo() {
    std::cout << "other foo\n";
}
}  // namespace
```

---

## 17/10/22

## Вычисление с вещественными числами

### Вещественные числа

Два типа — `float` и `double` (8 bytes) (*floating point numbers*). Хранят только двоичные дроби, остальное — только приближенно.

При выводе компилятор выводится с какой-то точностью (около 1e6).

```cpp
// floating-point numbers, IEEE-754
double x = 1.2;  // mantissa * 2^{exponent}
```

Стандартная байка — 0.1 + 0.2 == 0.3.

```cpp
std::cout << (0.1 + 0.2 == 0.3) << "\n"; // false
```

`std::cout.precision` — точность вывода.

```cpp
std::cout.precision(100);
std::cout << x << "\n";
```

—> с вещественными числами нужно понимать, что вычислениями получаются приближенными.

`double` обладает большей точностью (могут быть проблемы в с кастом из `double` во `float`). Еще большей точностью обладает `long double` (10 bytes).

```cpp
auto y = 1.2e100;  // double    std::cout << static_cast<float>(y) << "\n";

double z = 1.23456789;
std::cout << static_cast<float>(z) << "\n";

long double lz1 = 1.23456789;  // double literal
long double lz2 = 1.23456789L;  // long double literal
std::cout << static_cast<long double>(z) << "\n";
```

### **Накопление ошибки**

Пихаем вещественные числа там, где хватило бы десятичных —> ошибка может накапливаться.

При касте к инту происходит округление вниз.

```cpp
// Does not work for me locally.
// Use <cmath> and std::abs or fabs.
std::cout << abs(-23.5) << "\n";
std::cout << fabs(-23.5) << "\n";
std::cout << std::abs(-23.5) << "\n";

// towards 0
std::cout << static_cast<int>(-2.8) << " " << static_cast<int>(-2.2) << " "
          << static_cast<int>(2.2) << " " << static_cast<int>(2.8) << "\n";
// towards -inf
std::cout << std::floor(-2.8) << " " << std::floor(-2.2) << " "
          << std::floor(2.2) << " " << std::floor(2.8) << "\n";
// towards +inf
std::cout << std::ceil(-2.8) << " " << std::ceil(-2.2) << " "
          << std::ceil(2.2) << " " << std::ceil(2.8) << "\n";
// towards nearest
std::cout << std::round(-2.8) << " " << std::round(-2.2) << " "
          << std::round(2.2) << " " << std::round(2.8) << "\n";

std::cout << ".5 is towards +inf/-inf\n";
std::cout << std::round(-2.5) << " " << std::round(2.5) << "\n";
std::cout << std::round(-1.5) << " " << std::round(1.5) << "\n";
std::cout << std::round(-0.5) << " " << std::round(0.5) << "\n";
std::cout << std::round(-2.4999999999) << " " << std::round(2.499999999) << "\n";
}
```

`std::fixed` — вывести как есть. `double` хранит числа с каким-то шагом.

```cpp
long long x = 123'456'789'123'456'789;
std::cout << x << "\n";
std::cout << std::fixed << static_cast<double>(x) << "\n";
```

## Порядок вычислений в С++

### Порядок вызова функций

Нет гарантий. Порядок вызова функций зависит от стандарта —> лучше не завязываться на порядок вычислений.

```cpp
// https://notes.algoprog.ru/cpp/additional.html#id4
int a = 0;

int next_a() {
    a++;
    return a;
}

int subtract(int x, int y) {
    return x - y;
}

cout << next_a() << next_a() << endl;  // Может вывести как 12, так и 21.
cout << subtract(next_a(), next_a()) << endl;  // Может вывести как 3-4 == -1, так 4-3 == 1.
cout << subtract(next_a(), next_a()) << ' ' << next_a() << endl;  // Есть шесть вариантов выполнения.
```

*Interleaving* — под разными компиляторами поведение различно. Стандартом не гарантируется!

```cpp
bool a = false, b = false;

int f(bool, int x) {
    std::cout << "f(" << a << b << ", " << x << ")\n";
    return true;
}

// GCC: f(10, 1), f(11, 2)
// MSVC: f(11, 2), f(11, 1)
std::cout << f(a = true, 1) + f(b = true, 2) << "\n";
```

### **Short-circuit вычисления (короткое замыкание)**

Гарантии на тему, что проиходит при вычислении выражений типа  `a && b`, `a || b`.

`a && b` — если a == false, то вторая не вычислиться.

`a || b` — если a == true, то вторая не вычислиться.

Полезно, например, при проверке размера вектора.

Работает только для не перегруженных операторов.

```cpp
bool f(bool x) {
    std::cout << "f(" << x << ")\n";
    return x;
}

bool g(bool x) {
    std::cout << "g(" << x << ")\n";
    return x;
}
// short circuit, lazy evaluation
f(false) && g(false);
std::cout << "====\n";

f(false) && g(true);
std::cout << "====\n";

f(true) && g(false);
std::cout << "====\n";

f(true) && g(true);
std::cout << "====\n";

std::vector<int> v;
if (v.size() >= 10 && v[9] == 123) {  // safe!
    std::cout << "something\n";
}
if (v[9] == 123 && v.size() >= 10) {  // UB!
    std::cout << "something\n";
}

// similarly for ||
// only for non-overloaded && and ||
// does not work for `... operator&&(.....)`
```

### **UB до С++17**

До С++17 не было гарантии, что сначала вычислялась правая, а только потом — левая часть выражения.

```cpp
std::vector<int> vec;

int foo() {
    vec.push_back(20);
    return 123;
}

vec.push_back(10);
vec[0] = foo();  // Safe since C++17: = computes rights first, left second.
```

## Контейнеры

### **Set**

Хранит элементы в отсортированном порядке без повторений.

Все работает за $\mathcal{O}(\log n)$.

```cpp
std::set<int> s{3, 1, 2, 4, 2};
std::cout << s.size() << "\n"; // 4

// All operations are O(log)
s.insert(10);
s.erase(2);

std::cout << s.count(3) << "\n";
std::cout << s.count(33) << "\n";
if (s.count(10)) { /* ... */ }
if (s.count(10) != 0) { /* ... */ }
```

### **Multiset**

Хранит элементы в отсортированном порядке с повторениями. Операции работают дольше, чем в set’е.

`ms.count()` работает за линию.

```cpp
std::multiset<int> ms{3, 1, 2, 4, 2};
std::cout << ms.size() << "\n"; // 5

ms.insert(10);
ms.erase(2);  // O(log) + count(2)

std::cout << ms.count(3) << "\n";  // O(log) + count(3)
```

### **Итераторы**

Специальный тип, который называется `typename std::multiset<int>::iterator`. Умеет указывать на элемент в контейнере и бегать вперед-назад.

```cpp
std::multiset<int> ms{3, 1, 2, 4, 2};
typename std::multiset<int>::iterator it = ms.begin();
```

`s.begin()` — указывает на первый элемент.

`s.end()` — указывает на элемент за последним.

`++it` работает быстрее, чем `it++` (не нужно запоминать старое значение).

`*it` — получить значение по итератору.

Операции с итераторами работают за константное время амортизированно.

```cpp
// range-based for gets expanded into:
[[maybe_unused]] auto it2 = ms.begin();
//   1    2   2   3   4
//   ^    ^   ^   ^   ^    ^
// begin                  end
for (; it != ms.end(); ++it) {  // ++it faster than it++
    std::cout << " " << *it;
}
// All operations with iterators are O(1) amortized.
```

`ms.find(elem) == ms.end()` — элемента нет.

`erase(ms.end())` — UB!

```cpp
ms.erase(ms.find(2));
ms.erase(ms.find(2));

// Элемента не существует: find() == end()
// erase(end()) - UB
ms.erase(ms.find(2));  // UB
```

### **Structered binding**

`auto [x, y] = p` — разобрать пару по значениям.

`auto  &[x, y] = p` — ссылки на элементы пары, при изменении x, y значения в паре возникнут.

```cpp
auto [x, y] = p;
// auto x = p.first;
// auto y = p.second;

auto &[x, y] = p;
// auto &x = p.first;
// auto &y = p.second;
```

`std::minmax(elem1, elem2)` — возвращает почти пару. На самом деле, возвращает пару ссылок, а так как переданные значения временные, то потом они стираются.

```cpp
// ok
std::pair<int, int> p = std::minmax(30, 20);
std::cout << p.first << " " << p.second << "\n";  // Ok!

// UB
auto [x, y] = std::minmax(30, 20);
std::cout << x << " " << y << "\n";  // UB
}
```

### **Map**

Похоже на set из пар {константный ключ, неконстантное значение}.

`std::map<int, double> m` — создание.

```cpp
// ~ set<pair<const int, double>>
    std::map<int, double> m{
        std::pair<int, double>{10, 12.5},
        /*std::pair<int, double>*/{20, 2.5},
        /*std::pair<int, double>*/{30, 3},
    };
    m.insert(std::pair{25, 40.1});
    m.insert({35, 40.1});
    m.emplace(44, 1.1);
```

Итерирование происходит по парам ключ-значение. Можно использоваться structered binding. Можно и с &. Можно и с const &.

Особенности `operator[]`: если вызвать несуществующий элемент, то он будет 0.

```cpp
for (std::pair</*const*/int, double> p : m) {
    p.first += 100;  // WTF?
}

for (auto [key, value] : m) {
    // key += 100;  // key is const
    value += 100;  // does not affect map
}

for (auto &[key, value] : m) {
    // key *= 2;
    value *= 2;
}

for (const auto &[key, value] : m) {
    // value *= 2;
}
```

Нельзя использовать `operator[]`, если есть константная ссылка на map (`operator[]` не константный, он должен уметь добавлять элементы). Можно: `m.find(elem)->second`.

```cpp
std::map<int, double> m{
    std::pair<int, double>{10, 12.5},
    /*std::pair<int, double>*/{20, 2.5},
    /*std::pair<int, double>*/{30, 3},
};

std::cout << m[10] << "\n";  // 25
m[10]++;
std::cout << m[10] << "\n";  // 26
m[123] = 566;  // Создаётся новый элемент
std::cout << m[123] << "\n";  // 566
std::cout << m[124] << "\n";  // 0 

const auto &const_m = m;
// const_m.insert({12345, 67.89});
// std::cout << const_m[123] << "\n";

std::cout << (const_m.find(843579875) == const_m.end()) << "\n";  // not UB
*const_m.find(843579875);  // UB, please compare with .end() first
```

### **Отличия итераторов**

Для списка нет operator+= (оператор должен уметь работать за константное время).

В `<iterator>` — `std::next(l.begin, 2)` (может работать за линию для списка).

*random access iterator* (у вектора) — оператор с произвольным доступом.

*bidirectional iterator* (у списка) — умеет ходить вперед-назад единицу.

```cpp
std::vector v{10, 20, 30};
std::list l{10, 20, 30};

auto vit = v.begin();
vit += 2;  // random access iterator
std::cout << *vit << "\n";
std::cout << vit - v.begin() << "\n";  // 2

auto lit = l.begin();
// lit += 2;
++lit;  // bidirectional iterator
--lit;
++lit;
++lit;
std::cout << *lit << "\n";
std::cout << *std::next(l.begin(), 2) << "\n";  // #include <iterator>
std::cout << std::distance(l.begin(), lit) << "\n";  // linear time for non-random access
```

## Алгоритмы

Все алгоритмы заточены на итераторы.

### Sort

Сортирует, принимает полуинтервал [начало; конец + 1).

```cpp
std::vector v{1, 30, 2, 10, 3, 20, 4, 5};
//            0   1  2   3  4   5  6  7  end()
//                  [                )
std::sort(v.begin() + 2, v.end() - 1);
```

Можно передавать компаратор (оператор строго меньше) — как сравнивать между собой элементы. Передается функция, но достаточно лямбды.

При этом можно брать перменную, объявленную в main — захват по  `[&]`.

```cpp
std::sort(v.begin(), v.end(), []([[maybe_unused]] int a, [[maybe_unused]] int b) {
    // operator<, total order (полный линейный порядок)
    return std::abs(a - 7) < std::abs(b - 7);
//        return false;  // Ok: all elements are equal
//        return true;  // UB: irreflexivity is violated
//        return std::abs(a - 7) <= std::abs(b - 7);  // UB: irreflexivity is violated
});
```

Важно, что оператор должен быть корректным. То есть если, например, компаратор всегда возвращает всегда true, то это UB (элемент меньше сам себя ??). Не должна быть нарушена нерефлексивность.

### Lower bound

`std::binary_search` — true или false, найдет ли элемент.

`std::lower_bound` — первый элемент, *больший или равный* переданного. Если переданный больше максимального, то вернется `end`.

```cpp
std::vector v{1, 1, 1, 2, 2, 4, 5};
//            0  1  2  3  4  5  6  7
//          begin                  end
//            [                    )
std::cout << std::binary_search(v.begin(), v.end(), 3) << "\n"; 

typename std::vector<int>::iterator it = std::lower_bound(v.begin(), v.end(), 2);
// *it >= 2, *(--it) < 2

it = std::lower_bound(v.begin(), v.end(), 3);
```

### Upper bound

`std::upper_bound` — первый элемент, *больший* переданного.

```cpp
std::vector v{1, 1, 1, 2, 2, 4, 5};
//            0  1  2  3  4  5  6  7
//          begin                  end
//            [                    )
typename std::vector<int>::iterator it = std::upper_bound(v.begin(), v.end(), 2);
// *it > 2, *(--it) <= 2
std::cout << it - v.begin() << "\n";  // 5

it = std::upper_bound(v.begin(), v.end(), 3);
// *it > 3, *(--it) <= 3
std::cout << it - v.begin() << "\n";  // 5
```

### Удаление элементов

Если удалять элементы во время прохода, то все ломается.

```cpp
std::set<int> s{3, 1, 2, 4, 2};//, 100, 101, 102, 103, 104, 105, 106, 107};
for (int x : s) {
    if (x % 2 == 0) {
        s.erase(x);
        // UB при переходе к следующему элементу
    }
}
```

Правильно:

```cpp
{
    std::set<int> s{3, 1, 2, 4, 2};//, 100, 101, 102, 103, 104, 105, 106, 107};
    for (auto it = s.begin(); it != s.end(); ) {
        int x = *it;
        ++it;

        // 1 2 3 4
        //     ^
        if (x % 2 == 0) {
            s.erase(x);
        }
        // 1   3 4
        //     ^
    }
}
{
    std::multiset<int> s{3, 1, 2, 4, 2};//, 100, 101, 102, 103, 104, 105, 106, 107};
    for (auto it = s.begin(); it != s.end(); ) {
        if (*it % 2 == 0) {
            auto it_next = it;  // Или std::next()
            ++it_next;
            s.erase(it);
            it = it_next;
        } else {
            ++it;
        }
    }
    std::cout << s.size() << "\n";
}
{
    std::set<int> s{3, 1, 2, 4, 2};//, 100, 101, 102, 103, 104, 105, 106, 107};
    for (auto it = s.begin(); it != s.end(); ) {
        if (*it % 2 == 0) {
            it = s.erase(it);
        } else {
            it++;
        }
    }
    std::cout << s.size() << "\n";
}
// C++20: ranges
}
```

---

## 07/11/22

### Внутренние классы (inner classes)

Внутри структур и классах можно объявлять структуры и классы.

Имеют доступ ко всем приватным полям. Не привязаны к конкретному экземпляру Foo.

`Foo::Bar` — доступ к внутренней структуре. Похоже на обращение к `namespace`.

Слова `private` и `public` влияют на структуры.

```cpp
#include <iostream>
#include <map>

struct Foo {
private:
    int x = 0;

    void foo() {
        x++;
    }

public:
    struct Bar {  // has access to all privates of Foo
        int y = 0;
        void foo() {
            // x++;
            Foo f;
            f.x++;
            f.foo();
            y++;
        }
    };

private:
    struct Baz {};
};

int main() {
    std::map<int, int>::iterator it;
    Foo::Bar b;
    b.foo();

    // Foo::Baz z;
}
```

### Статические функции (static functions)

Похожи на `friend` функции. Объявляются внутри класса, принадлежат классу. Чем-то похоже на метод, но не привязана к объекту.

`Foo::baz` — вызов функции.

Имеют доступ к приватным полям.

Возможность вызова `static` функции корректируется словами `private/public/protected`.

```cpp
struct Foo {
private:
    int x;

private:
    friend void foo(Foo &f);  // 1(a). not affected by private/public
    static void bar(Foo &f);  // 1(b). affected by private/public

public:
    static void baz(Foo &f);
};

void foo(Foo &) {
}

void Foo::bar(Foo &f) {
    f.x = 10;
}

void Foo::baz(Foo &f) {
    bar(f);  // can access private
}

int main() {
    Foo f;

    // 2(a). Called as a usual function.
    foo(f);

    // 2(b). Called as a member function.
    // Foo::bar(f);  // cannot access private
    Foo::baz(f);
}
```

### Чистые указатели (raw pointers)

Чем-то похожи на ссылки. Встроенный в язык способ ссылаться на другие объекты.

`*prev` —  указатель. `*` пишется перед каждым именем переменной.

Можно объявлять где угодно. Можно делать локальными/глобальными. Можно изменять.

`nullptr` — указатель, указывает в никуда (в отличии от ссылок, которые обязательно куда-то указывают).

`&a` — узнать адрес и присвоить в указатель.

`*b` — разыменование указателя, извлечение значенния.  `a->` означает то же самое.

Разыменование `nullptr` — UB.

`a++` — арифметика указателей. Внутри массива ОК, дальше — сложнее.

```cpp
#include <cassert>
#include <string>
#include <iostream>

struct node_indexed {
    std::string data;
    int prev = -1, next = -1;
};

struct node_ptr {
    std::string data;
    node_ptr *prev = nullptr, *next = nullptr;  // Note that '*' is next to name
    // nullptr is better than NULL or 0.
};

int main() {
    {
        node_indexed nodes[3]{
            {"hello", /*prev*/ -1, /*next*/ 1},
            {"world", /*prev*/ 0, /*next*/ 2},
            {"wow", /*prev*/ 1, /*next*/ -1}
        };
        int head = 0;
        std::cout << nodes[head].data << "\n";  // hello
        std::cout << nodes[nodes[head].next].data << "\n";  // world
        std::cout << nodes[nodes[nodes[head].next].next].data << "\n";  // wow
    }
    std::cout << "\n";
    {
        node_ptr a{"hello"}, b{"world"}, c{"wow"};

        // Unlike reference:
        // 1. Can be nullptr.
        assert(a.prev == nullptr);
        assert(c.next == nullptr);
        // 2. Can be changed.
        a.next = &b; b.prev = &a;  // `&` means 'take address of'
        b.next = &c; c.prev = &b;

        // `*` means 'dereference', like with iterators
        std::cout <<   b      .data << "\n";  // world
        std::cout << (*a.next).data << "\n";  // world
        std::cout <<   a.next->data << "\n";  // world

        a.next = &c;
        std::cout << (*a.next).data << "\n";  // wow
        std::cout <<   a.next->data << "\n";  // wow

        *a.prev;  // UB: dereference of nullptr
        a.next++;  // Possible, 'pointer arithemtics' ~ 'iterators', out of scope for now
    }
}
```

`this` — указатель на текущий объект (например, внутри структуры).

```cpp
struct Foo {
    int x = 20;

    void foo() {
        Foo *f = this;
    }
};
```

### Умные указатели (unique pointers)

Оператор `new`  — находит в куче (heap) место памяти, инициализирует его, вызывает конструктор и возвращают указатель на это место —> `dynamic storage duration`.

```cpp
Foo *f = new Foo;  // Dynamic storage duration for 'Foo'.
                       // "На куче" (heap)
```

Оператор `delete` — удаляет все данные, очищает поля. Необходимо вызвать **ровно** 1 раз (иначе *double-free +* UB). Работает только в паре с `new` (иначе — UB).

```cpp
delete f;  // Leak otherwise
// delete f;  // Double-free
```

Если не вызвать `delete`, то UB не будет, но будет утечка памяти (*leak otherwise*) — объект будет жить до конца программы. Для `nullptr` `delete` делать можно.

—> лучше не использовать `new`-`del`.

```cpp
f = nullptr;
delete f;  // ok
```

**Unique pointer**

Умные указатели (*unique pointer*) умеют вызывать `delete` в нужный момент, но функционал немного ограниченнее, чем у чистых указателей.

```cpp
#include <memory>

// smart pointer / умный указатель
std::unique_ptr<Foo> f;
```

`std::unique_ptr<Foo> ptr` — объявление указателя. Лежит в `<memory>`.

Может быть равен  `nullptr`, можно разыменовывать, есть ->

```cpp
f = nullptr; // ok
f->v[4]; // ok
```

**Инвариант** — он *единственный* хранит то, что было выделено при помощи new.

`std::make_unique` — с С++14, функция, гарантирующая, что не возникнут чистые указатели.

```cpp
f = std::unique_ptr<Foo>(new Foo());
// f = std::make_unique<Foo>();  // C++14
```

Удаление одного указателя, второй указывает на несуществующий объект. Поэтому для `unique_ptr` запрещено копирование (кто-когда-что будет удалять?).

```cpp
auto f = std::make_unique<Foo>();
std::unique_ptr<Foo> b;

b = f;  // copy is prohibited.
b = std::unique_ptr<Foo>(f.get());  // copy, but will do double free
```

### **Move-семантинка (move-semantic)**

Операции перемещения для `unique_ptr` — передача владения от одного `unique_ptr` к другому (работает и с векторами, и со строками…).

Передать вектору результат функции — ок, через промежуточный объект — не ок.

`std::move` — передача значения. Лежит в `<utility>`.

Писать `std::move` в `return` можно, но зачем?..

```cpp
#include <utility>

struct Foo {
    std::vector<int> v;

    Foo() : v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10} {}
};

std::unique_ptr<Foo> create_foo() {
    auto f = std::make_unique<Foo>();
    assert(f->v[2] == 3);
    return f;  // Can return local variables
}

auto global_f = std::make_unique<Foo>();
std::unique_ptr<Foo> create_foo_bad() {
    return std::move(global_f);  // But why?
    // return std::make_unique<Foo>(*global_f);  // deep copy
}

int main() {
    std::vector<std::unique_ptr<Foo>> vec;
    {
        vec.emplace_back(create_foo());  // No std::move needed

        auto f = create_foo();
        vec.emplace_back(std::move(f));
    }
}
```

Пригождается в конструкторах.

`*.get()` — возвращает чистый указатель из `unique_ptr`.

```cpp
struct Node {
    int value;
    std::unique_ptr<Node> next;

    Node(int value_, std::unique_ptr<Node> next_)
        : value(value_)
        , next(std::move(next_)) {
    }
};

int main() {
    auto a = std::make_unique<Node>(10, std::make_unique<Node>(20, nullptr));
    std::cout << a->value << " " << a->next->value << "\n";

    auto c = std::make_unique<Node>(30, std::move(a));
    std::cout << (a == nullptr) << "\n";
    std::cout << c->value << " " << c->next->value << "\n";
}
```

Пример UB

Внутри функции `unique_ptr` удалит указатель, поэтому адрес будет неверным —> UB.

```cpp
#include <cassert>
#include <iostream>
#include <memory>
#include <vector>

struct Foo {
    std::vector<int> v;

    Foo() : v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10} {}
};

Foo *bad() {
    std::unique_ptr<Foo> f = std::make_unique<Foo>();
    Foo *ptr = f.get();
    assert(ptr->v[3] == 4);
    return ptr;
}

int main() {
    Foo *f = bad();
    *f;  // UB, because unique_ptr `delete`d the object
    std::cout << f->v[3] << "\n";
}
```

Можно использовать `std::move` с контейнерами, но нет гарантий, что будет лежать в старом контейнере.

```cpp
std::string s = "hi";
std::string s2 = std::move(s);
// `s` is not guaranteed to be empty
```

## Наследование (inheritance)

### **Базовое наследование**

- warning
  - "Наследование" много где пытаются переплести с "объектно-ориентированным программированием"
    - Говорят, мол, "наследование, инкапсуляция, полиморфизм - три кита ООП!"
    - Я не умею определять эти штуки в терминах ООП так, чтобы было понятно, о чём речь.
    - Поэтому я эти слова покажу на некоторых примерах, но определять не буду.
  - На самом деле "наследование" - это просто один из механизмов языка C++, который позволяет делать некоторые новые вещи. В других языках иногда наследования вообще нет (Haskell, Rust), но на них прекрасно пишут код, просто механизмы другие.
  - Если вы будете искать примеры "наследования", вы можете найти что-то вроде "яблоко/банан являются фруктом" или "квадрат является прямоугольником" или "компьютер является устройством" или "машина является средством передвижения"
    - Вроде [https://ravesli.com/urok-154-bazovoe-nasledovanie-v-c/](https://ravesli.com/urok-154-bazovoe-nasledovanie-v-c/)
    - Мне кажется, это не даёт понимания "зачем это реально может пригодиться в программах", но может дать ложное ощущение "всё очевидно"
    - Но на самом деле это всё неправда и надо смотреть не на "физический смысл", а на код, иначе придём к проблеме квадрата-прямоугольника (круга-эллипса), это будет на следующих лекциях.
  - В современном C++ классы/наследование используются не так часто, как в какой-нибудь Java.

*Наследование в базовой форме:* есть базовый (родительский/предок/надкласс/суперкласс) класс с какими-то полями. У него есть производный (дочерний) класс со всеми полями/методами базового класса + сови поля/методы.

Для компилятора: “возьми все поля/методы класса Base  и скопируй их в Derived”.

Наследник не может обращаться к `private` полям  базового класса.

`protected` — то, к чему может обращаться *класс и его наследники*.

Аналог: завести поле Base в классе Derived.

```cpp
struct Base {  // Базовый класс (base) в C++. Родительский/предок/надкласс/суперкласс (Python, Java).
    int x = 10;
    void foo() const {
        std::cout << "x=" << x << "\n";
    }

private:
    int very_secret = 20;  // for this class and its friends only

protected:
    int secret = 30;  // for this class and its derived classes
    // Please use 'private' still.
};

struct Derived : Base {  // Производный класс (derived). Дочерний/подкласс.
    // Base base;  // better alternative: композиция (composition)
    int y = 40;
    void bar() const {
        foo();
        std::cout << "x=" << x << ", y=" << y << ", secret=" << secret << "\n";
        // int z = very_secret;
    }
};

int main() {
		Derived d;
		d.x = 123;
		d.foo();
		d.bar();
		// int x = d.very_secret;
		// int y = d.secret;
}
```

**Пример, где наследование помогает:**

Можно взять *указатель* или *ссылку* базового класса на наследника —> можно писать код, одинаково с наследниками. Но нельзя вызывать методы наследников (логично, компилятор не знает, что это наследник).

Получается неявное преобразование от указателя на наследника к указателю на базовый класс — *basecast/upcast*.

Пример: `std::ostream &os` в `operator>>`.

Всегда корректно вызывать наследника вместо базового (основная причина существования наследников).

Derived — это Base, к которому дописали несколько полей, фактически, даже указатель не меняется.

```cpp
#include <iostream>

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

// operator<<(std::ostream &os, ....)
void foo_and_print(const Base &b) {
    b.foo();
    // b.bar();
    std::cout << "x=" << b.x << "\n";
}

int main() {
    {
        Derived d;
        std::cout << "Via reference\n";
        foo_and_print(d);

        Base &b = d;  // basecast (C++), upcast (другие языки).
        b.foo();
        // b.bar();
        b.x++;
        std::cout << "d.x=" << d.x << "\n";
    }
    {
        Derived d;
        std::cout << "Via pointer\n";

        Derived *dptr = &d;
        Base *bptr = dptr;
        bptr->foo();
        // bptr->bar();
        std::cout << "b.x=" << bptr->x << "\n";
        bptr->x++;
        std::cout << "d.x=" << d.x << "\n";

        std::cout << dptr << " " << bptr << "\n";
        std::cout << &d.x << " " << &d.y << "\n";
    }
    std::cout << sizeof(Base) << " " << sizeof(Derived) << "\n";
}
```

**Конструкторы у наследников**

Если у базового есть конструктор, то и у наследника должен быть конструктор —> явно прописываем конструктор по умолчанию.

`Derived() : Base(10) {}`

Конструкторы не наследуются (и не бывают виртуальными).

Нельзя непосредственно определять поля базового класса в списке инициализации.

```cpp
Derived() : Base(10) {}
//    Derived(int y_) : y(y_), Base(y) {}  // -Wreorder, UB: read from uninitialized field
```

Сперва вызываются конструкторы базового класса, затем полей в порядке объявления в классе. Удаление — в обратном порядке.

```cpp
struct Base {
    int x;
    Base(int x_) : x(x_) {}
};

struct Derived : Base {
    int y = 20;

    Derived() : Base(10) {}
//    Derived(int y_) : y(y_), Base(y) {}  // -Wreorder, UB: read from uninitialized field
//    Derived(int y_) : Base(y), y(y_) {}  // UB is apparent
    Derived(int y_) : Base(y_), y(y_) {}  // OK
};

struct SubDerived : Derived {
    SubDerived(int) {}  // : Derived() is implied.
};

struct DerivedNaive : Base {
};
```

**Методы у наследников**

Пусть мы реализуем одинаковые методы в базовом классе и в наследнике. Компилятор решает, какой из них вызвать, в зависимости не от реального типа объекта, а в зависимости от типа указателя/ссылки.

```cpp
struct Base {
    int x = 10;

    void print() const {
        std::cout << "x = " << x << "\n";
    }
};

struct Derived : Base {
    int y = 20;

    void print() const {
        std::cout << "x = " << x << ", " << "y = " << y << "\n";
    }
};

void print(const Base &db) {
    db.print();
}

int main() {
    Base b;
    Derived d;
    b.print(); // called base method
    d.print(); // called derived method

		pritn(d); // called base method !!
}
```

Чиниться пометить функцию как `virtual`, чтобы можно было вызвать реальный метод объекта (`virtual` добавляет дополнительную информацию).

По-хорошему у наследника нужно писать `override` (с С++11) — проверить, что в родителе есть такая `virtual` функция.

```cpp
struct Base {
    int x = 10;

    virtual void print() const {
        std::cout << "x = " << x << "\n";
    }
};

struct Derived : Base {
    int y = 20;

    void print() const override /* C++11 */ {
        // override: проверить, что в родителе virtual есть.
        std::cout << "x = " << x << ", " << "y = " << y << "\n";
    }
};

void print(const Base &db) {
    db.print();
}

Derived d;

print(d); // called derived method
```

**Чисто виртуальные функции (pure virtual)**

Если у базового класса функция ничего не делает, а весь ее смысл — в наследниках, то нужно обозначать функцию как чисто виртуальную:

```cpp
virtual … = 0;
```

Класс называется абстрактным (*abstract*). Нельзя создать объект такого класса.

`virtual` всегда спускается до самого вложенного объекта.

```cpp
struct Printable {  // 'abstract' class because at least one 'pure virtual' function
    virtual void print() const = 0;  // pure virtual
};

struct Int : Printable {
    int x = 0;

    void print() const override {
        std::cout << x << "\n";
    }
};

struct String : Printable {
    std::string s;

    void print() const override {
        std::cout << s << "\n";
    }
};

void print(const Printable &p) {
    p.print();
}
```

---

## 14/11/22

## Время жизни (lifetime)

### Время жизни объектов (object lifetime)

Каждый объект в какой-то момент времени по определенному адресу создается, какое-то времени живет, потом уничтожается. Объект не может менять свой адрес! При копировании сначала создается новый объект, а потом туда копируется старый.

Узнать адрес структуры:

```cpp
this --> inside the struct
&f --> outside the struct
```

Из правил про время жизни:

1. начинается ровно один раз *при вызове конструктора.*
2. заканчивается ровно один раз *при вызове денструктора*, который автоматически очищает все поля, дополнительно указывать не надо.

   **rule of zero**

   Если ничего из специальных функций-членов не определено пользователем, то (с учетом переменных-членов) компилятор предоставит реализации по умолчанию для каждой из них. **Правило Нуля заключается в том, что тот сценарий, когда *не нужно* определять *ничего* из специальных функций-членов, должен быть *предпочтительным***.


Никаких новых объектов не создано — оператор присваивания вызывает оператор присваивания для всех полей. При этом в структурах он генерируется автоматически.

Жизнь `c` заканчивается в момент написания `}`.

```cpp
{
		Foo c(30);
		c = b;  // assignment operator (operator=), no new objects created
} 
```

При использовании ссылок в функциях новые объекты не создаются.

```cpp
void test(const Foo &f) {
}

test(a);  // reference binding, no new objects created
```

### **Копирующий конструктор (copy constructor)**

Новый объект создается (`f1`). Вызывался конструктор копирования, который генерируется автоматически. Сигнатура — `Foo(const Foo&)`.

```cpp
[[maybe_unused]] Foo f1 = a;  // copy constructor (Foo(const Foo&)), we did not modify it
// [[maybe_unused]] Foo f2(a);  // copy constructor (Foo(const Foo&)), we did not modify it
```

```cpp
void consume(Foo param) {
}

consume(a);  // copy constructor
consume(Foo(Foo(Foo(30))));  // new object, but copies are typically optimized
consume(40);  // new object
```

**Named Return Value Optimization (NRVO)**

При возвращении значения из функции может быть оптимизировано и сразу создастся объект в месте вызова функции, то есть в месте памяти, выделенной под f2 (без копирования переменной сначала во временный объект, а потом в локальный) — Named Return Value Optimization (NRVO).

```cpp
[[maybe_unused]] Foo f2 = create();  // named return value optimization (NRVO) is possible
```

Еще существует RVO для случаев, когда экземпляр возвращаемого класса создаётся прямо в операторе `return`.

```cpp
C f() { 
		return C(); 
}
```

- **about RVO/NRVO**

  RVO (Return Value Optimization) – оптимизация компилятора, позволяющая в некоторых случаях не создавать локальный объект, который будет использован как возвращаемое значение. Аналогично NRVO, разница лишь в том, что RVO применяется к prvalue объектами, а NRVO — lvalue.

  С С++17 RVO – это уже не оптимизация, а правило, которое должны выполнять компиляторы.

  Компилятор обязан применить RVO в функциях времени компиляции (`constexpr`) и при инициализации глобальных, статических и thread-local переменных (constant initialization).

  NRVO происходит следующим образом: на месте вызова функции, к которой применяется NRVO, вставляется инициализация объекта, в который будет присвоен результат этой функции. В аргументы функции добавляется указатель на этот объект. Все вычисления, относящиеся к возвращаемому объекту, теперь выполняются над объектом, переданным по указателю.

  **Необходимые условия для применения RVO/NRVO:**

  Необходимые условия для применения этой оптимизации:

  - Тип объекта, возвращаемого из функции согласно сигнатуре, должен совпадать с типом локального объекта или конструироваться неявно из типа локального объекта. Допустимо отличаться на константность.
  - Возвращаться должен именно локальный объект, а не ссылка на него или какая-то его часть.
  - В случае NRVO возвращаемый объект не должен быть volatile.

  Оптимизация не применяется в следующих случаях:

  - Есть несколько путей выхода из функции, которые возвращают разные локальные объекты.
  - Возвращаемый локальный объект ссылается на встроенный asm-блок.

### Automatic storage duration

*Automatic storage duration* — выделение “на стеке” памяти для переменных (например, локальных в функциях), очищается после выхода из области видимости `{}`.

```cpp
[[maybe_unused]] int x;  // Automatic storage duration.
                         // "На стэке".
Foo f;
```

Можем обратиться только к внутренней переменной `x`.

```cpp
[[maybe_unused]] int x;  

for (int x = 0; x < 10; x++) {  // shadowing (-Wshadow); independent x
    ...
}
// old 'x' is available again
```

При этом С-шные массивы выделяются на стеке, поэтому при попытке создать массив огромного размера происходит переполнение — *stack overflow*. Вектор же хранит свои данные не на стеке, а на куче.

```cpp
[[maybe_unused]] int data[100];
// [[maybe_unused]] int data2[100'000'000];  // stack overflow
```

### Висячие ссылки (d**angling references)**

Объект умер, но осталась висячая ссылка (`dangling`). Ее можно запомнить, но обращаться к ней нельзя.

```cpp
std::vector<int>& foo() {
    std::vector<int> vec{1, 2, 3};
    return vec;
}

std::vector<int> &vec1 = foo();  // Not UB
std::cout << vec1.size() << "\n";  // UB
```

Также нельзя присваивать.

```cpp
std::vector<int> vec2 = foo();  // UB, because copy constructor
```

### Инвалидация (invalidation)

Получить некорректную ссылку или указатель можно при использовании контейнеров.

**Set**

После добавления элементов в set ссылка и итератор на элемент остаются валидными. После удаления элемента из `set` они становятся некорректными.

```cpp
std::set<int> s{10};
const int &x = *s.begin();
std::set<int>::iterator x_it = s.begin();
s.insert(5);
s.insert(15); // x and x_it are still valid
s.erase(10);
// x and x_it are invalid.
```

**Vector**

Если же после добавления в `vector` большого количества элементов произошла релокация всех элементов, то все ссылки и итераторы станут некорректными.

```cpp
std::cout << "vector\n";
std::vector<int> vec{10, 20};
int &x = vec[0];
std::vector<int>::iterator x_it = vec.begin();
vec[1] = 100;
std::cout << &x << " " << &vec[0] << "\n";
vec.pop_back();
// If reallocated, all references and iterators are invalid
vec.push_back(20);
vec.push_back(30);
vec.push_back(40);
std::cout << &x << " " << &vec[0] << "\n";
std::cout << x << " " << *x_it << "\n"; // can be different
```

**Deque**

Инвалидируются все итераторы, а на ссылки нет гарантий.

В `deque` нет гарантии, что итераторы инвалидируются/нет.

```cpp
std::deque<int> d{10};
int &x = *d.begin();
std::deque<int>::iterator x_it = d.begin();
// Invalidates all iterators, but not references, e.g. in clang++ with Microsoft STL
for (int i = 0; i < 1000; i++) {
    d.push_back(0);
    d.push_front(0);
}
```

### Static storage duration

*Static storage duration —* память под переменные выделяется один раз в какой-то момент программы и живет все время, пока программа не завершится. Пример — глобальные переменные. При этом стандартные типы (например, int) гарантированно инициализируются.

```cpp
[[maybe_unused]] int global_x;  // Static storage duration. not default-initialized (reading is UB),
                                //                          but value-initialiezd (0).
[[maybe_unused]] Bar global_b;  // Static storage duration.
```

Слово `static` перед переменной — static storage duration внутри функций.

```cpp
int main() {
    int x = 10;  // Automatic, "на стэке".
    static int y = 20;  // Static, "в глобальной памяти".

    {
        static int data[1'000'000];
        // static int data[1'000'000] = {1, 2, 3};
        std::vector<int> v(10);
        std::cout << data[0] << "\n";
    }
}
```

**Static variables in functions**

Можно создать глоабльную переменную внутри функций. Никто, кроме этой функции, не может обратиться к этой переменную.

```cpp
int counter() {
    static int calls = 10;
    return ++calls;
}
```

Инициализируется такая переменная в момент прохода по этой строке.

Пример:

`vector` создастся в момент только во второй раз вызова (когда `condition = true`). При всех следующих вызовах размер вектора один и тот же. Поэтому не надо инициализировать статические объекты из аргументов функции — инициализация произойдет только в первый раз.

```cpp
void foo(bool condition, int n) {
    if (condition) {
        std::cout << "inside if\n";
        static std::vector<int> v(n);
    }
}
```

Способы обойти есть (но зачем тогда static?).

```cpp
/*static std::vector<int> v;
v.clear();
v.resize(n);*/

// OR: v.assign(n);
```

**Static variables in clasess**

Статические переменные внутри класса — как глобальные переменные, но видимость ограничена областью видимостью класса.

```cpp
struct Counter {
private:
    static int calls;  // Declaration

public:
    int calls_on_creation = calls;

    static int counter() {
        return ++calls;
    }
};

int Counter::calls = 10; // Definition

int main() {
    Counter c;
    std::cout << c.calls_on_creation << "\n";
    // std::cout << Counter::calls << "\n"; // banned
}
```

### Dynamic storage duration

*Dynamic storage duration* — управление выделением памяти при помощи `new` и `delete`. Объект должен быть создан и удален ровно один раз, но за этим следит программист.

У `new` много разных способов инициализации.

```cpp
#include <iostream>

struct Foo {
    Foo() {
        std::cout << "Foo()\n";
    }
    Foo(int x) {
        std::cout << "Foo(" << x << ")\n";
    }
};

// dynamic storage duration, "на куче"

Foo *f = new Foo;  // Foo f; // default initialization
delete f;

Foo *f = new Foo();  // default initialization
delete f;

Foo *f = new Foo{};  // default initialization
delete f;

Foo *f = new Foo(10);  // direct initialization
delete f;

Foo *f = new Foo{10};  // direct list initialization
delete f;

Foo *f_arr = new Foo[5];
delete[] f_arr;
// delete f_arr;  // UB

Foo *f_arr = new Foo[5]{1, 2, 3};  // copy initialization + value initialization
delete[] f_arr;
```

По факту * storage duration — разные области памяти. Можно пытаться угадывать, в какой области памяти лежит переменная, по ее адресу.

```cpp
int x1, x2, x3; // static

int main() {
    int y1, y2, y3; //automatic
    int *z = new int; // dinamic
}
```

Можно сравнивать адреса, но не гарантируется, что например, выделенные переменные лежат внутри памяти подряд. Сравнение же адресов переменных из разных областей памяти приводит к unspesified behaviour.

```cpp
std::cout << (&x1 < z) << "\n";
std::cout << (&x1 < &x2) << "\n";
```

С массивом все ок, сравнение будет работать корректно — все переменные лежат в памяти друг за другом.

```cpp
int arr[10];
int *a2 = &arr[2];
int *a5 = &arr[5];
assert(a2 < a5);  // Always true: both inside the same array.
```

## Время жизни временных объектов (temprorary object lifetime)

### Примеры

**Создание-удаление временных переменных**

Временный объект создается во время вычислений и потом удаляется в момент ближайшей “`;`”.

```cpp
Foo calc() {
    Foo x = Foo(20).value + Foo(30).value;
    return x;
}
```

**Висячие ссылки от временных объектов**

Попытка создать висячую ссылку. Не компилируется.

```cpp
int main() {
    std::vector<int> &vec = std::vector{1, 2, 3};
}
```

Можно создать константную ссылку к временному объекту. Объект умрет тогда же, когда умрет ссылка. Вектором можно спокойно пользоваться.

```cpp
void println(const std::vector<int> &) {
}

int main() {
    // https://herbsutter.com/2008/01/01/gotw-88-a-candidate-for-the-most-important-const/
    const std::vector<int> &vec = std::vector{1, 2, 3};
    std::cout << vec.size() << "\n"; // 3

    println(std::vector{1, 2, 3});
}
```

**Vector**

У `vector` есть отдельно его адрес вектора (automatic storage duration) и адрес его элементов. Также есть размер переменной, содержащей адрес вектора. Сами элементы — dynamic storage duration.

```cpp
int main() {
    std::vector<int> v{10, 20};
    std::cout << sizeof(v) << "\n";

    /*struct vector<int> {
        int *buffer;
        size_t len;
        size_t capacity;
    };*/
}
```

Пример UB — константная ссылка указывает на ссылку. Время жизни не продлевается. Получилась висячая ссылка.

```cpp
int id_val(int x) {
    return x;
}

const int &id_ref(const int &x) {
    return x;
}

int main() {
    // Rvalue lifetime disaster, Arno Schoedl.
    {
        const int &x = id_val(10);
        std::cout << x << "\n";
    }
    {
        const int &x = id_ref(10);
        std::cout << x << "\n";
    }
}
```

Проблемы возникают, когда стандартные функции возвращают ссылки, которые могут оказаться висячими, если объекты были временными.

```cpp
const auto &val = std::min(3, 4);
std::cout << val << "\n";

const auto &[min1, max1] = std::minmax(3, 4);
std::cout << min1 << " " << max1 << "\n";

// pair<const int &, const int &>
auto [min2, max2] = std::minmax(3, 4);
std::cout << min2 << " " << max2 << "\n";
```

### Derived cast

Можно преобразовать (`static_cast`) ссылку на базовый класс к константной ссылке на наследника. Работает только если переданная ссылка — ссылка на наследника.

```cpp
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

void foo(const Base &b) {
    std::cout << "foo(" << b.x << ")\n";
    const Derived &d = static_cast<const Derived&>(b);  // derivedcast (C++), downcast (others).
    std::cout << ".y=" << d.y << "\n";
}

Derived d;
foo(d);  // Not UB: d is really Derived.

Base b;
foo(b);  // UB!
```

Нельзя сделать `static_cast` для разных классов, даже не скомпилируется. `C-cast` же  скомпилирует… Поэтому его лучше не использовать.

```cpp
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

void foo(const Base &b) {
    const Derived &d = static_cast<const Derived&>(b);
//    const std::string &d = static_cast<const std::string&>(b);  // UB, but does not compile
//    const std::string &d = (const std::string&)b;  // UB, but compile;
}
```

- Пример реализации списка.

    ```cpp
    #include <iostream>
    
    struct node {
        node *prev = nullptr;
        node *next = nullptr;
    };
    
    struct node_with_data : node {
        int data;
    
        node_with_data(int data_) : data(data_) {}
    };
    
    void link(node &x, node &y) {
        x.next = &y;
        y.prev = &x;
    }
    
    int main() {
        node l;
        node_with_data a(10), b(20), c(30);
    
        link(l, a);
        link(a, b);
        link(b, c);
        link(c, l);
    
        for (node *it = l.next; it != &l; it = it->next) {
            auto it_data = static_cast<node_with_data*>(it);
            std::cout << it_data->data << "\n";
        }
        std::cout << "=====\n";
    
        node_with_data d(40);
        link(a, d);
        link(d, b);
    
        for (node *it = l.next; it != &l; it = it->next) {
            auto it_data = static_cast<node_with_data*>(it);
            std::cout << it_data->data << "\n";
        }
    }
    ```


## Run-Time-Type Information (RTTI)

Определение типа прямо во время исполнения программы. Работает только для полиморфных классов (классы с виртуальными функциями).

```cpp
// RTTI is for polymorphic classes only.
struct Base {
    virtual ~Base() {
    }
};

struct Derived : Base {};
```

Оператора `typeid` —> `const type_info &`:

- от типа —>  выдает тип закодировано, `boost::core::demangle` — расшифровывает

    ```cpp
    const std::type_info &info_base = typeid(Base);
    const std::type_info &info_derived = typeid(Derived);
    const std::type_info &info_int = typeid(int);
    std::cout << (info_base == info_derived) << "\n";
    
    std::cout << boost::core::demangle(info_base.name()) << "\n";
    ```

- от выражения (неполиморфное) —> тип выражения (не считает)

    ```cpp
    const std::type_info &info_int_expr =
            typeid(2 + 2 + foo());  // foo() is not called
    ```

- от ссылки —> тип объекта (наследника и прочее)

    ```cpp
    Base b;
    Derived d;
    std::cout << boost::core::demangle(typeid(b).name()) << "\n";
    std::cout << boost::core::demangle(typeid(d).name()) << "\n";
    ```


### typeid

Проблемы:

- typeid не знает про псевдонимы
- нельзя копировать
- странные имена

—> используется обертка `type_index`

```cpp
// RTTI is for polymorphic classes only.
struct Base {
    virtual ~Base() {
    }
};

struct Derived : Base {};

int main() {
    Base b;
    Derived d;

    const std::type_info &info_base = typeid(Base);
    const std::type_info &info_derived = typeid(Derived);
    using DerivedAlias = Derived;
    const std::type_info &info_derived_alias = typeid(DerivedAlias);

    const std::type_info &info_b = typeid(b);
    const std::type_info &info_d = typeid(d);

    std::cout << (info_base == info_b) << "\n";                 // 1
    std::cout << (info_base == info_d) << "\n";                 // 0
    std::cout << (info_derived == info_b) << "\n";              // 0
    std::cout << (info_derived == info_d) << "\n";              // 1
    std::cout << (info_derived == info_derived_alias) << "\n";  // 1

    std::cout << typeid(int).name() << "\n";  // 'i' (msys2) pr 'int' (VS)
    std::cout << info_b.name() << "\n";       // '4Base' or 'struct Base' (VS)
    std::cout << typeid(std::vector<int>).name()
              << "\n";  // 'St6vectorIiSaIiEE' or something else in VS

    // type_info is not copyable or <> comparable
    // std::type_info x = info_base;
    // Since C++11:
    std::set<std::type_index> types{info_base, info_derived, info_b,
                                    info_d};  // C++11
    for (const auto &t : types) {
        std::cout << "t=" << t.name() << "\n";
    }
}
```

### Dynamic cast

`dynamic_cast` как `static_cast`, но проверяет, что каст корректен.  Если не получилось, то вернет `nullptr`.

`-fno-rtti` — запрещается пользоваться `type_index` и `dynamic_cast` (слишком много памяти занимают).

```cpp
#include <iostream>

struct Base {
    virtual ~Base(){};
};

struct Derived1 : Base {};

struct SubDerived1 : Derived1 {
    int value = 123;
};

struct Derived2 : Base {
    int value = 456;
};

void f(const Base &b) {
    std::cout << "====\n";
    // dynamic_cast only compiles if Base is "polymorphic" <=> has at least one
    // virtual method. Typically it's a virtual destructor.
    const Derived1 *d1 = dynamic_cast<const Derived1 *>(&b);
    if (d1)
        std::cout << "Derived1 or SubDerived1\n";
    if (typeid(b) == typeid(Derived1))
        std::cout << "Exactly Derived1\n";
    // d1 is visible

    if (const SubDerived1 *sd1 =
            dynamic_cast<const SubDerived1 *>(&b)) {  // C++03
        std::cout << "SubDerived1 " << sd1->value << "\n";
    }
    // sd1 is not visible

    if (const Derived2 *d2 = dynamic_cast<const Derived2 *>(&b);
        d2 != nullptr && d2->value > 10) {  // C++17: if with init statement
        std::cout << "Derived2 " << d2->value << " > 10\n";
    }
    // d2 is not visible
}

int main() {
    // RTTI (Run-Time Type-Information) is needed, it's sometimes disabled with
    // `-fno-rtti`.
    SubDerived1 sd1;
    Derived2 d2;
    f(sd1);
    f(d2);

    Base *b = nullptr;
    std::cout << dynamic_cast<const Derived1 *>(b) << "\n";
}
```

С ссылками: если не получилось преобразовать, то программа падает.

```cpp
#include <iostream>

struct Base {
    virtual ~Base(){};
};

struct Derived1 : Base {};

struct SubDerived1 : Derived1 {
    int value = 123;
};

struct Derived2 : Base {
    int value = 456;
};

void f(const Base &b) {
    std::cout << "====\n";
    {
        [[maybe_unused]] const Derived1 &d1 = dynamic_cast<const Derived1 &>(b);
        std::cout << "Derived1\n";
    }
    {
        const SubDerived1 &md1 = dynamic_cast<const SubDerived1 &>(b);
        std::cout << "SubDerived1 " << md1.value << "\n";
    }
}

int main() {
    SubDerived1 sd1;
    Derived2 d2;
    f(sd1);
    f(d2);  // exception thrown
}
```

---

## 21/11/22

## Move

### Для чего сделан move

`std::swap` под С++11 работает быстро, под С++03 тоже может быстро отработать, например, для вектора.

```cpp
std::vector<int> v1(100000), v2;
for (int i = 0; i < 100000; i++) {
    using std::swap;
    swap(v1, v2);
}
```

Для структур под разными стандартами работает быстро (С++11) и медленно (С++03).

Почему: с С++11 используется `std::move`.

В зависимости от стандарта у кода может быть разная асимптотика.

```cpp
// C++03
void swap(T &a, T &b) {
    T tmp = a;
    a = b;
    b = tmp;
}

// C++11
void swap(T &a, &b) {
    T tmp = std::move(a);
    a = std::move(b);
    b = std::move(tmp);
}
```

### **Инициализация полей**

Разные варианты:

1. 1 копирование
2. 1 инициализация + 1 move + 1 вызов деструктора

Если в будущем необходимо проинициализировать поле значением, то лучше передавать значение и использовать move. Иначе — константная ссылка.

```cpp
// A person not necessarily have a name:
// https://github.com/kdeldycke/awesome-falsehood
// https://habr.com/ru/post/431866/
// https://t.me/c/1591098747/3404 https://www.youtube.com/watch?v=Z8SHvJnGUCM&ab_channel=AndreyGein

struct PersonCpp03 {
    std::string name;
    PersonCpp03(const std::string &name_) : name(name_) {}  // 1 copy
};

struct PersonCpp11 {
    std::string name;
    PersonCpp11(std::string name_) : name(std::move(name_)) {}  // 1 initialization name_ + 1 move + 1 destruct of empty
    void say(const std::string &message) {  // better than `std::string message`
        std::cout << name << " says " << message << "\n";
    }
};

std::string create_name() {
    std::string s = "hello world";
    return s;  // no std::move needed
}

int main() {
    {
        std::string x = "Egor";
        [[maybe_unused]] PersonCpp03 p1(x);  // x is copied into p1.name: 1 copy
        [[maybe_unused]] PersonCpp03 p2("Egor");  // temporary is copied: 1 init, 1 copy, 1 destruct
        [[maybe_unused]] PersonCpp03 p3(create_name());  // temporary is copied: 1 init inside create_name(), 1 copy, 1 destruct
    }
    {
        std::string x = "Egor";
        [[maybe_unused]] PersonCpp11 p1(x);  // 1 copy + 1 move + 1 destruct of empty
        [[maybe_unused]] PersonCpp11 p2("Egor");  // 1 init, 1-2 move, 1-2 destruct of empty
        [[maybe_unused]] PersonCpp11 p3(create_name());  // 1 init inside create_name(), 1-3 move, 1-3 destruct of empty
    }
}
```

### **Moved from state**

*Moved from state* — состояние, в котором оказывается объект, из которого сделали `move`.

Для ptr гарантированно, что будет `nullptr`.

```cpp
std::unique_ptr<int> ptr(new int(200));
std::move(ptr);  // Does nothing.
auto ptr2 = std::move(ptr);
// Guaranteed: ptr.get() == nullptr
```

Для всех остальных типов гарантий нет. Не UB.

```cpp
std::string s1 = "hello";
std::string s2 = std::move(s1);
// s1 is 'moved-from': valid, but unspecified.
// May be empty, may be "hello", may be in a random state.
```

Почему так? Потому что типы можно реализовывать разными способами —> *small string optimization* или (в общем случае) *small object optimization*.

```cpp
// small string optimization (small object optimization)
struct string {
    char buf_small[10];
    char *buf_big = nullptr;
    void move_from(string &other) {
        if (other.buf_big) {
            buf_big = other.buf_big;
            other.buf_big = nullptr;
        } else {
            buf_small[0] = other.buf_small[0];
            buf_small[1] = other.buf_small[1];
            // ...
            other.len = 0;  // optional
        }
    }
};
```

### Friend make unique

Хотим сделать приватный конструктор, но чтобы можно было создавать `unique_ptr`.

Проблема — `make_unique` не может вызывать приватный конструктор Foo.

Технически можно было бы сделать фукнцию `friend`. Потому что такая функция может быть реализована через другие, которые уже не смогут вызвать приватный конструктор.

```cpp
static std::unique_ptr<Foo> make() {
    return std::make_unique<Foo>();  // bad
    // return std::unique_ptr<Foo>(new Foo());  // good
}

// auto p1 = std::make_unique<Foo>();  // hence, this is bad
auto p2 = Foo::make();  // this is good
```
## Final

### **Final struct**

`final` запрещает наследоваться от структуры.

Стратегии использования:

1. Можно помечать все структуры, от которых нет наследников —> нельзя случайно отнаследоваться.
2. Вообще не использовать слово. Если программист захочет, то он отнаследуется, поменяв код. + иногда отнаследоваться очень нужно. Пример:  *moke objects*.

```cpp
struct Base {};

struct Derived1 final : Base {
    int value = 123;
};

// struct SubDerived : Derived1 {};  // impossible

// Two alternative strategies:
// 1. `final` all structs, unless you are sure there will be derived classes.
// "Safer" because you cannot inherit from someone who did not think about it.
// 2. `final` nothing, because you do not want to prohibit some "safe"
// inheritance where needed (see "mock objects").

struct Derived2 final : Base {
    int value = 456;
};
```

### **Final methods**

Возникает только для виртуальных функций. Означает, что нельзя переписывать дальше. `override` следует из `final` автоматически. Лучше писать `final`.

Опять разные концепции, нужно или не нужно…

```cpp
struct Base {
    virtual void foo() = 0;
    virtual void bar() = 0;
};

struct Derived : Base {
    void foo() final {  // final needs 'virtual'
    }

    // 'override' is redundant:
    // void fooo() final {}  // CE, which is fine.
    // virtual void fooo() final {}  // Not CE, which is not fine.

    void bar() override {
    }
};

struct SubDerived : Derived {
    // 'override' is not important, will not compile either way.
    // void foo() {}

    void bar() override {
    }
};
```

## Cmake

### Cmake

*Cmake* — приложение, которое позволяет генерировать скрипты для сборки программ на С++.  CMake — кроссплатформенная автоматизированная система сборки проектов. Непосредственно сборкой она не занимается, а только генерирует Makefile, который потом будет выполнен утилитой make.

Под разные компиляторы необходимо писать разные команды и разные флаги.

*Cmake:*

```cpp
cmake_minimum_required(VERSION 3.22)

project(lectures-cmake CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

include_directories(include)  # Should be before add_executable
add_executable(main src/main.cpp src/foo.cpp)
```

*Консоль:*

```cpp
g++ src/main.cpp src/foo.cpp -Iinclude -o main
```

**Подробнее:**

```go
cmake_minimum_required(VERSION 2.6) // set version

project(visualization) // project name

set(VARIABLE The variable's value) // set variable named VARIABLE which containes 
																	 // data "The variable's value"

add_definitions(-DSOME_IMPORTANT_DEFINITION) // set comands for compiler

include_directories("headers/" "more_headers/") // add directories with headers

find_package(Boost COMPONENTS chrono filesystem REQUIRED) // add packages

target_link_libraries(test ${Boost_LIBRARIES}) // find libraries and link them to projects
```

**Cmake без внешних библиотек**

`cmake .` —> `make —version`

В директории может быть записано не более 1 генератора.

## Внешние библиотеки

- about

    ```cpp
    Libraries consist of two parts:
    
    1. **Headers**. Source code in C++ and C.
       Function declarations and some other stuff.
       Some functions _can be_ defined in headers, we just don't know the syntax (it's easy, though).
    2. **Binaries** (optional). Compiled source code, to be linked with your program.
    
    Header-only libraries: e.g. `doctest`. Are very easy to use.
    
    Binaries are different for different compilers and settings, require *ABI compatibility*.
    
    Types of binaries:
    
    * Static library. `lib****.a` with GCC/Clang or `****.lib` with MSVC.
      An archive of multiple `.o` files, just a separate translation unit.
    
    * Dynamic library. `*.dll` on Windows, `lib*.so` on Linux, `*.dylib` on macOS.
      A separate executable with its own global variables, heap and dependencies.
      Can be (un)loaded on demand.
      * Can be loaded on start, e.g. `libstdc++.dll` in msys2.
        There is a corresponding static library.
      * Can be loaded on demand. Examples: editor plugins, game mods.
        In Java/Python it's free.
    
    https://github.com/fffaraz/awesome-cpp
    
    How to configure:
    
    1. Set up include path. Either works or `-I/some/path` for GCC.
    2. Set up library binaries path. Either works or `-L/some/path` for GCC.
    3. Tell linker which libraries to use: `-lfoo` means `libfoo.a` or `foo.lib` depending on the compiler ("decorated")
    4. (optional) Put dynamic libraries next to the app or add them to the `PATH` so it can be found on startup.
    ```


### **Библиотека boost**

Если библиотека не настроена, то необходимо подключать вручную: `g++ … -I` —> работает для *header-only libraries*.

```cpp
#include <boost/core/demangle.hpp>
#include <iostream>
#include <vector>

int main() {
    std::cout << boost::core::demangle(typeid(std::vector<int>).name());
}
```

Иногда не работает (не хватает реализации) —> добавить: `g++ … -l`

Компилятор сам дописывает lib*.a и ищет в нужной папке.

`-L` — поиск библиотеки в этой папке.

Обычно библиотеки состоят из заголовков и реализации —> в каких именно, выяснется методом проб и ошибок. Так ставится почти любая библиотека.

```cpp
#include <boost/filesystem/operations.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    std::cout << "Current directory is: " << fs::current_path() << "\n";
    std::cout << "Content is:\n";
    for (const auto &f : fs::directory_iterator(fs::current_path())) {
        std::cout << f.path().filename() << "\n";
    }
}
```

### Пример использования других библиотек

**SMFL**

```cpp
#include <SFML/Graphics.hpp>

int main()
{
    sf::RenderWindow window(sf::VideoMode(200, 200), "SFML works!");
    sf::CircleShape shape(100.f);
    shape.setFillColor(sf::Color::Green);

    while (window.isOpen())
    {
        sf::Event event;
        while (window.pollEvent(event))
        {
            if (event.type == sf::Event::Closed)
                window.close();
        }

        window.clear();
        window.draw(shape);
        window.display();
    }

    return 0;
}
```

**Nana**

```cpp
#include <nana/gui.hpp>
#include <nana/gui/widgets/label.hpp>
#include <nana/gui/widgets/button.hpp>

int main()
{
    using namespace nana;

    //Define a form.
    form fm;

    //Define a label and display a text.
    label lab{fm, "Hello, <bold blue size=16>Nana C++ Library</>"};
    lab.format(true);

    //Define a button and answer the click event.
    button btn{fm, "Quit"};
    btn.events().click([&fm]{
        fm.close();
    });

    //Layout management
    fm.div("vert <><<><weight=80% text><>><><weight=24<><button><>><>");
    fm["text"]<<lab;
    fm["button"] << btn;
    fm.collocate();
	
    //Show the form
    fm.show();

    //Start to event loop process, it blocks until the form is closed.
    exec();
}
```

**Сборка через Cmake**

- include
  - foo.hpp

      ```cpp
      #include "foo.hpp"
      #include <boost/filesystem/operations.hpp>
      #include <iostream>
      
      namespace fs = boost::filesystem;
      
      void foo() {
          std::cout << "Hello World\n";
          std::cout << "Current directory is: " << fs::current_path() << "\n";
      }
      ```

- src
  - foo.cpp

      ```cpp
      #include "foo.hpp"
      #include <boost/filesystem/operations.hpp>
      #include <iostream>
      
      namespace fs = boost::filesystem;
      
      void foo() {
          std::cout << "Hello World\n";
          std::cout << "Current directory is: " << fs::current_path() << "\n";
      }
      ```

  - main.cpp

      ```cpp
      #include "foo.hpp"
      
      int main() {
          foo();
      }
      ```

- cmakelists

    ```cpp
    cmake_minimum_required(VERSION 3.22)
    
    project(lectures-cmake CXX)
    
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED True)
    
    find_package(Boost 1.74 REQUIRED filesystem)
    include_directories(${Boost_INCLUDE_DIRS})  # -I
    link_directories(${Boost_LIBRARY_DIRS})  # -L
    
    include_directories(include)  # Should be before add_executable
    add_executable(main src/main.cpp src/foo.cpp)
    target_link_libraries(main ${Boost_LIBRARIES})  # -l
    ```


### Разделяемые библиотеки (shared libraries)

- about

    ```cpp
    # What is a library
    The answer is heavily OS- and compiler- dependent.
    Exact flags, naming conventions are not unified as well.
    
    Typically everything comes from C, not C++.
    Other languages have their own mechanisms (e.g. `import`s in Java/Python).
    
    ## Static library
    A bunch of translation units embedded into your executable.
    E.g. parts of Standard C++ Library.
    
    E.g. `libsqlite3.a` (for msys2 and on Linux) or `sqlite3.lib` (for Visual Studio).
    
    ## Dynamic library, linked at compiled time
    A bunch of translation units linked together with your executable.
    
    E.g. `libstdc++-6.dll` for msys2 weigs 2MB and contains parts of C++ Standard Library.
    It's necessary to run the executable.
    It may have its own dependencies (e.g. C Standard Library or Microsoft C++ Runtime).
    
    It may be possible to "link dynamic library statically", i.e. embed the library into the executable.
    
    How to tell linker about the DLL is a separate issue.
    Typically there is a helper static library like `libsqlite3.dll.a`.
    
    ## Dynamic library, loaded at run time
    Very complex topic, see the "Linkers and Loaders" book by John R. Levine (not sure if outdated, I did not read it).
    E.g. there are export/import tables, different sections, symbol visibility, weak symbols...
    Not for us.
    
    See example with `./main` and `./plugins/*.dll` (only a subset of plugins is available at any given moment).
    ```


Набор каких-то единиц трансляции, которые встраиваются в exe-шник.

Зачем? Можно вынести кусок стандартной библиотеки, чтобы другие приложения использовали его.

Динамические библиотеки можно подгружать динамически во время исполнения программы —> так называемые плагины (скомпилированный кусок кода, который можно подгрузить к программе).

- src
  - CmakeLists.txt

      ```cpp
      cmake_minimum_required(VERSION 3.10)
      
      project(shared-library CXX)
      
      set(CMAKE_CXX_STANDARD 17)
      set(CMAKE_CXX_STANDARD_REQUIRED True)
      
      set(CMAKE_CXX_VISIBILITY_PRESET hidden)  # Recommended by Boost.DLL's manual
      
      find_package(Boost 1.71 REQUIRED filesystem)  # Required by Boost.DLL
      include_directories(${Boost_INCLUDE_DIRS})
      link_directories(${Boost_LIBRARY_DIRS})
      
      # Windows: libmultiply.dll (msys2), multiply.dll (VS)
      # Linux: libmultiply.so
      # macOS: libmultiply.dylib
      add_library(multiply SHARED plugin_multiply.cpp)
      add_library(sum SHARED plugin_sum.cpp)
      
      add_executable(main main.cpp)
      target_link_libraries(main ${Boost_LIBRARIES} ${CMAKE_DL_LIBS})
      ```

  - abstract_plugin.hpp

      ```cpp
      #ifndef ABSTRACT_PLUGIN_HPP_
      #define ABSTRACT_PLUGIN_HPP_
      
      #include <boost/config.hpp>
      #include <string>
      
      namespace plugins_demo {
      class BOOST_SYMBOL_VISIBLE abstract_plugin {
      public:
          virtual const std::string &name() const = 0;
          virtual float calculate(float x, float y) = 0;
          virtual ~abstract_plugin() = default;
      };
      }  // namespace plugins_demo
      
      #endif  // ABSTRACT_PLUGIN_HPP_
      ```

  - main.cpp

      ```cpp
      #include <boost/dll/import.hpp>
      #include <boost/filesystem/path.hpp>
      #include <boost/version.hpp>
      #include <iostream>
      #include <vector>
      #include "abstract_plugin.hpp"
      
      #if BOOST_VERSION >= 107600
      #define boost_dll_import_symbol ::boost::dll::import_symbol
      #else
      #define boost_dll_import_symbol ::boost::dll::import
      #endif
      
      namespace plugins_demo {
      int main() {
          namespace fs = boost::filesystem;
      
          std::vector<boost::shared_ptr<abstract_plugin>> plugins;
          for (auto &f : fs::directory_iterator(fs::path("plugins/"))) {
              std::cout << "Loading from " << f << "\n";
              plugins.emplace_back(
                  boost_dll_import_symbol<abstract_plugin>(f.path(), "plugin"));
              // TODO: dll::load_mode::append_decorations
          }
      
          for (auto &plugin : plugins) {
              std::cout << plugin->name()
                        << "->calculate(1.5, 1.5) == " << plugin->calculate(1.5, 1.5)
                        << std::endl;
          }
          return 0;
      }
      }  // namespace plugins_demo
      
      int main() {
          return plugins_demo::main();
      }
      ```

  - plugin_multipy.cpp

      ```cpp
      #include <boost/config.hpp>
      #include <iostream>
      #include "abstract_plugin.hpp"
      
      void foo();  // Not implemented anywhere
      
      namespace plugin_multiply {
      class plugin_multiply : public plugins_demo::abstract_plugin {
      public:
          plugin_multiply() {
              std::cout << "Constructing plugin_multiply\n";
              // The following line will cause 'undefined reference' error.
              // foo();
              // Windows: linker error during DLL compilation.
              // Linux: loader error during .so loading in `main.cpp`, very similar to the linker. You may implement foo() in main.cpp.
              // macOS: linker error during .dylib compilation.
          }
      
          const std::string &name() const override {
              static std::string name = "multiply";
              return name;
          }
      
          float calculate(float x, float y) override {
              return x * y;
          }
      
          ~plugin_multiply() {
              std::cout << "Destructing plugin_multiply\n";
          }
      };
      
      extern "C" BOOST_SYMBOL_EXPORT plugin_multiply plugin;
      plugin_multiply plugin;
      }  // namespace plugin_multiply
      ```

  - plugin_sum.cpp

      ```cpp
      #include <boost/config.hpp>
      #include <iostream>
      #include "abstract_plugin.hpp"
      
      namespace plugin_sum {
      class plugin_sum : public plugins_demo::abstract_plugin {
      public:
          plugin_sum() {
              std::cout << "Constructing plugin_sum\n";
          }
      
          const std::string &name() const override {
              static std::string name = "sum";
              return name;
          }
      
          float calculate(float x, float y) override {
              return x + y;
          }
      
          ~plugin_sum() {
              std::cout << "Destructing plugin_sum\n";
          }
      };
      
      extern "C" BOOST_SYMBOL_EXPORT plugin_sum plugin;
      plugin_sum plugin;
      }  // namespace plugin_sum
      ```


## Что-то из практики

### Ручная настройка компилятора

Этот раздел требуется только если всё не сработало автоматически.

После установки вам требуется найти две папки:

- `BOOST_ROOT`: в ней лежит папка `boost`, в которой лежат заголовки вроде `version.hpp`.
- `BOOST_LIBRARY_DIRS`: в ней лежат скомпилированные файлы с именами вроде `libboost_filesystem.a` или `boost_filesystem.lib` или более сложными. Обычно `BOOST_LIBRARY_DIRS` лежит в `BOOST_ROOT` с названием `lib`.

Дальше надо добавить в компилятор две настройки:

1. Путь поиска заголовочных файлов (здесь `/foo/bar/baz` — полный путь к `BOOST_ROOT`):
  - GCC/Clang: `I/foo/bar/baz`
  - [Visual Studio](https://docs.microsoft.com/en-us/cpp/build/reference/i-additional-include-directories?view=msvc-160): `/I /foo/bar/baz` (консоль) или добавить `/foo/bar/baz` в Additional Include Directories (графический интерфейс).
2. Путь поиска статических библиотек (здесь `/foo/bar/baz` — полный путь к `BOOST_LIBRARY_DIRS`):
  - GCC/Clang: `L/foo/bar/baz`
  - [Visual Studio](https://docs.microsoft.com/en-us/cpp/build/reference/libpath-additional-libpath?view=msvc-160): `/libpath:/foo/bar/baz` после [разделителя `/link`](https://docs.microsoft.com/en-us/cpp/build/reference/compiler-command-line-syntax?view=msvc-160) (регистр важен) или добавить `/foo/bar/baz` в Additional Library Directories (графический интерфейс).

Примеры:

- `g++-10 main.cpp -I/foo/bar/baz -L/foo/bar/baz/lib остальные-параметры`
- `cl main.cpp /I /foo/bar/baz параметры-компилятора /link /libpath:/foo/bar/baz/lib параметры-линковщика`

### Линковка со статической библиотекой

Это требуется для GCC/Clang независимо от способа установки и не требуется для Visual Studio.

Вам требуется явно написать, какие статические библиотеки подключать при помощи ключа `-lLIBRARY`. Чтобы узнать строчку `LIBRARY`:

1. Найдите в папке `BOOST_LIBRARY_DIRS` (см. выше) файл с подстрокой `boost_filesystem` в названии.
2. Если файлы называются по-другому, попросите помощи.
3. Если есть несколько файлов, то при наличии выбора выбирайте:
  - Без `.dll` или `.so` в названии (чтобы была статическая линковка)
  - С `mt` в названии (чтобы работало в многопоточных приложениях, нам это пригодится в конце семестра)
4. Файл должен начинаться с `lib` и заканчивается на `.a`, это название статической библиотеки в стиле Linux. Отбросьте префикс и суффикс, результат — строчка `LIBRARY`. Например: `boost_filesystem`, `boost_filesystem-mt`.

**Важно**: конкретно GCC (точнее, его линковщику LD) [важен порядок](https://stackoverflow.com/questions/45135/why-does-the-order-in-which-libraries-are-linked-sometimes-cause-errors-in-gcc), в котором вы передаёте исходные файлы и библиотеки: `-lboost_filesystem a.cpp` не скомпилируется с undefined reference, а вот `a.cpp -lboost_filesystem` скомпилируется. Надо, чтобы сначала шли единицы трансляции, которые используют какие-то функции, а строго после них — статические библиотеки, в которых эти функции определяются.

---

## 28/11/22

### Виртуальный деструктор

Деструкторы сгенерируются автоматически и будут корректно работать.

Но если взять на себя управление выделением/освобождением памяти, то могут возникнуть проблемы. В данном случае int и string удаляются по-разному, поэтому вызывать delete на объектах типа Printable* некорректно: смотрим на родителя, вызываем его никак не помеченный деструктор —> поля наследников остались нетронутыми, утечка/UB.

Такое хорошо ловиться санитайзером.

```cpp
struct Printable {  // 'abstract' class because at least one 'pure virtual' function
    virtual void print() const = 0;  // pure virtual
};

struct Int : Printable {
    int x = 0;

    void print() const override {
        std::cout << x << "\n";
    }
};

struct String : Printable {
    std::string s;

    void print() const override {
        std::cout << s << "\n";
    }
};

void print(const Printable &p) {
    p.print();
}

std::vector<Printable*> create_vector() {
    std::vector<Printable*> v;
    Int *i = new Int;
    i->x = 10;
    v.push_back(i);

    String *s = new String;
    s->s = "hello";
    v.push_back(s);
    return v;
}

int main() {
    std::vector<Printable*> v = create_vector();
    for (auto &el : v) {
        print(*el);
    }
    for (auto el : v) {
        delete el;  // UB because incorrect destructor is called; Address Sanitizer, compiler warning
    }
}
```

Решение: делаем деструктор виртуальным: `virtual ~Printable() {};`. Тогда компилятор вызовет нужный деструктор.

```cpp
struct Printable {
    virtual void print() const = 0;
    virtual ~Printable() {};  // !!! VIRTUAL DESTRUCTOR
};

int main() {
    std::vector<Printable*> v = create_vector();
    for (auto &el : v) {
        print(*el);
    }
    for (auto el : v) {
        delete el;  // No UB because virtual destructor is called
    }
}
```

Аналогичная проблема для unique_ptr, только деструктор вызовем не мы.

```cpp
std::vector<std::unique_ptr<Printable>> create_vector() {
    std::vector<std::unique_ptr<Printable>> v;
    std::unique_ptr<Int> i = std::make_unique<Int>();
    i->x = 10;
    v.push_back(std::move(i));

    std::unique_ptr<String> s = std::make_unique<String>();
    s->s = "hello";
    v.push_back(std::move(s));
    return v;
}

int main() {
    std::vector<std::unique_ptr<Printable>> v = create_vector();
    for (auto &el : v) {
        print(*el);
    }
    // UB because incorrect destructor is called; Address Sanitizer, gcc does not warn, clang does warn
}
```

Решение все то же — виртуальный деструктор.

## Многопоточность

### Базовое

На компьютерах может выполняться больше одной программы одновременно. Каждая запущенная программа называется *процессом*. Внутри каждого процесса бывает один или несколько *потоков выполнения* — кусочков программы, выполняющихся одновременно с другими кусочками программы.

Чтобы использовать потоки в С++, подключается заголовок `<thread>`.

Создание потока: `std::thread t([&]() { ... });`

Потоки могут получать доступ к переменным друг друга.

`t.join()` — ждет завершения потока и потом очищает ресурсы. Если не вызвать UB.

В конце потока вызывается деструктор, который проверяет, был ли вызван join (join вызывается руками, так как может быть заморожен на время).

```cpp
#include <thread>

int main() {
    int data = 1234;
    std::thread t([&]() {
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
        std::cout << "Hello from thread! data=" << data << "\n";
        data += 10;
    });
    std::cout << "Waiting for it...\n";
    t.join();  // Подождать поток t, освобождает ресурсы потока.
    std::cout << "data is " << data << "\n";
    // Вызывается ~thread(), к этому моменту обязательно сделать join().
}
```

Можно запускать потоки с обычной функцией (без лямбды), можно передавать с параметрами. При этом ссылки передаются по-особенному: `std::ref(a)` (иначе траблы, вдруг переменная была локальной или что-то такое).

```cpp
void worker(int a, int &b) {
    std::cout << "Thread: " << a << " " << &b << "\n";
}

int main() {
    int a = 10, b = 20;
    std::thread(worker, a, std::ref(b)).join();
}
```

В С++ можно делать `t.detach()` — обещаем больше не делать `join()`, теперь ОС отвечает за сборку мусора в потоке. Возникает тогда, когда нам ничего от потока не нужно, то есть  не важно, что делает поток и когда он завершится.

Но нужно гарантировать, что программа завершится после завершения всех потоков, иначе — UB.

```cpp
int main() {
    int data = 1234;
    std::thread t([&]() {
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
        std::cout << "Hello from thread! data=" << data << "\n";
        data += 10;
    });
    std::cout << "Detaching...\n";
    t.detach();  // Обещаем больше не делать join(), теперь ОС отвечает за сборку мусора в потоке.
                 // Возникает только если нам вообще ничего от потока `t` никогда не будет нужно.
                 // Единственный разумный пример: мы создаём сетевой сервер, который в бесконечном цикле плодит потоки для клиентов.
    std::cout << "data is " << data << "\n";
    // После завершения main() завершается программа, но поток всё ещё работает — UB (обращения к умершим глобальным переменным).
    return 0;
}
```

Пример параллельного подсчета:

```cpp
std::vector<unsigned> results;

void worker(int begin, int end, int t) {
    unsigned sum = 0;
    for (int i = begin; i < end; i++) {
        sum += i * i / 2;
    }
    results[t] = sum;
}

int main() {
    const int N = 1'000'000'000;
    const int K = 3;

    auto start = std::chrono::steady_clock::now();

    std::vector<std::thread> ts;
    results.resize(K);
    for (int i = 0; i < K; i++) {
        std::thread t(worker, N * i / K, N * (i + 1) / K, i);
        ts.emplace_back(std::move(t));  // note: std::thread cannot be copied
    }
    for (auto &t : ts) {
        t.join();
    }

    unsigned result = 0;
    for (auto r : results) {
        result += r;
    }
    std::cout << result << "\n";
    std::cout << "Elapsed "
              << std::chrono::duration_cast<std::chrono::milliseconds>(
                     std::chrono::steady_clock::now() - start
                 )
                     .count()
              << "ms\n";
}
```

### Взаимодействие потоков

Работает.

```cpp
void writeln(const std::string &s) {
    for (std::size_t i = 0; i < s.size(); i++) {
        std::cout << s[i];
    }
    std::cout << '\n';
}

int main() {
    for (;;) {
        writeln("Hello from the main thread");
    }
}
```

Плохо работает, символы выводятся рандомно.

```cpp
void writeln(const std::string &s) {
    for (std::size_t i = 0; i < s.size(); i++) {
        std::cout << s[i];
    }
    std::cout << '\n';
    // std::cout << s << "\n";
}

int main() {
    std::thread t([]() {
        for (;;) {
            writeln("Hello from the second thread");
        }
    });
    for (;;) {
        writeln("Hello from the main thread");
    }
}
```

То есть все проблемы возникают из-за того, что потоки работают одновременно. Ровно как и бонусы :)

### Mutex

Методы борьбы — *примитивы синхронизации*.

`std::mutex` — *MUTual EXclusion* (лежит в `<mutex>`)—> две операции, `lock()` и `unlock()`. Поток может захватывать только кто-то один одновременно.

Например, можно (вернее даже сказать “нужно”) сделать глобальный mutex, который будет захватывать вывод на экран.

`std::yeild` —> костыль, уступка кому-нибудь другому.

```cpp
#include <mutex>

std::mutex m;
void writeln(const std::string &s) {
    m.lock();
    for (std::size_t i = 0; i < s.size(); i++) {
        std::cout << s[i];
    }
    std::cout << '\n';
    m.unlock();
}

int main() {
    std::thread t([]() {
        for (;;) {
            writeln("Hello from the second thread");
            std::this_thread::yield();  // Костыль!
        }
    });
    for (;;) {
        writeln("Hello from the main thread");
        std::this_thread::yield();
    }
}
```

Как ошибиться — сделать локальный `mutex`.

`lock()` для mutex  делается один раз, для второго будет ожидание освобождения.

```cpp
void writeln(const std::string &s) {
    std::mutex m;
    m.lock();
    for (std::size_t i = 0; i < s.size(); i++) {
        std::cout << s[i];
    }
    std::cout << '\n';
    m.unlock();
}
```

### Unique_lock

Если `unlock()` забыли или поток упал, то все упало, все потоки подвисли... —>

`std::unique_lock m1{m}` —> автоматически сделал и `lock()`, и `unlock()`.

```cpp
std::mutex m;
void writeln(const std::string &s) {
    std::unique_lock l{m};
    for (std::size_t i = 0; i < s.size(); i++) {
        std::cout << s[i];
    }
    std::cout << '\n';
}
```

### Байки про многопоточность

- src

    ```cpp
    #include <iostream>
    #include <thread>
    
    #pragma GCC optimize ("O0")
    
    const int N = 500'000'000;
    const int M = 1000;
    
    int main() {
        int data = 0;
        auto worker = [&]() {
            for (int i = 0; i < N; i++) {
                data++;
            }
        };
        std::thread t(worker);
        for (int i = 0; i < M; i++) {
            std::cout << "data is " << data << " (in progress)\n";
        }
        t.join();
        std::cout << "data is " << data << "\n";
        return 0;
    }
    ```


Выводим только четные… Или нет? —> сначала проверяется условие, а потом выводится уже измененное значение.

- src

    ```cpp
    #include <iostream>
    #include <thread>
    
    #pragma GCC optimize ("O0")
    
    const int N = 500'000'000;
    const int M = 1000;
    
    int main() {
        int data = 0;
        auto worker = [&]() {
            for (int i = 0; i < N; i++) {
                data++;
            }
        };
        std::thread t(worker);
        for (int i = 0; i < M; i++) {
            if (data % 2 == 0) {
                std::cout << "data is " << data << " (in progress)\n";
            }
        }
        t.join();
        std::cout << "data is " << data << "\n";
        return 0;
    }
    ```


Можно считывать в локальную переменную.

- src

    ```cpp
    #include <iostream>
    #include <thread>
    
    #pragma GCC optimize ("O0")
    
    const int N = 500'000'000;
    const int M = 1000;
    
    int main() {
        int data = 0;
        auto worker = [&]() {
            for (int i = 0; i < N; i++) {
                data++;
            }
        };
        std::thread t(worker);
        for (int i = 0; i < M; i++) {
            int data_snapshot = data;
            if (data_snapshot % 2 == 0) {
                std::cout << "data is " << data_snapshot << " (in progress)\n";
            }
        }
        t.join();
        std::cout << "data is " << data << "\n";
        return 0;
    }
    ```


Два потока —> data++ — это три операции: считать, увеличить, записать.

- src

    ```cpp
    #include <iostream>
    #include <thread>
    
    #pragma GCC optimize ("O0")
    
    const int N = 500'000'000;
    const int M = 10'000;
    
    int main() {
        int data = 0;
        auto worker = [&]() {
            for (int i = 0; i < N; i++) {
                data++;
            }
        };
        std::thread t1(worker);
        std::thread t2(worker);
        for (int i = 0; i < M; i++) {
            int data_snapshot = data;
            if (data_snapshot % 2 == 0) {
                std::cout << "data is " << data_snapshot << " (in progress)\n";
            }
        }
        t2.join();
        t1.join();
        std::cout << "data is " << data << "\n";
        return 0;
    }
    ```


Можно захватывать значение mutex. Но все еще есть UB — нельзя одновременно делать что-то с переменной одновременно. Даже читать.

- src

    ```cpp
    #include <iostream>
    #include <mutex>
    #include <thread>
    
    #pragma GCC optimize ("O0")
    
    const int N = 5'000'000;
    const int M = 10'000;
    
    int main() {
        std::mutex m;
        int data = 0;
        auto worker = [&]() {
            for (int i = 0; i < N; i++) {
                std::unique_lock l{m};
                data++;
            }
        };
        std::thread t1(worker);
        std::thread t2(worker);
        for (int i = 0; i < M; i++) {
            int data_snapshot = data;
            // todo: example: copy vector
            if (data_snapshot % 2 == 0) {
                std::cout << "data is " << data_snapshot << " (in progress)\n";
            }
        }
        t2.join();
        t1.join();
        std::cout << "data is " << data << "\n";
        return 0;
    }
    ```


Нужно любой доступ к переменной делать по mutex’у.

- src

    ```cpp
    #include <iostream>
    #include <mutex>
    #include <thread>
    
    #pragma GCC optimize ("O0")
    
    const int N = 5000000;
    const int M = 10000;
    
    // Дополнительное чтение: у clang есть статический анализ: https://clang.llvm.org/docs/ThreadSafetyAnalysis.html
    // Можно пометить int data GUARDED_BY(m); и он много чего проверит.
    
    int main() {
        std::mutex m;
        int data = 0;
        auto worker = [&]() {
            for (int i = 0; i < N; i++) {
                std::unique_lock l{m};
                data++;
            }
        };
        std::thread t1(worker);
        std::thread t2(worker);
        for (int i = 0; i < M; i++) {
            std::unique_lock l(m);
            int data_snapshot = data;
            l.unlock();  // Не m.unlock()! Иначе unique_lock сделает unlock() ещё раз, это UB.
    
            if (data_snapshot % 2 == 0) {
                std::cout << "data is " << data_snapshot << " (in progress)\n";
            }
        }
        t2.join();
        t1.join();
        std::cout << "data is " << data << "\n";
        return 0;
    }
    ```


### Atomic

`std::atomic<int> data` (в `<atomic>`) — атомарные переменные, с ними можно делать какие-то операции без `mutex`’а.

Атомарная операция — такая операция, что не может случиться ситуации, когда мы "вклинились" в середину её выполнения. Считаем, что ничего не атомарно, если нам явно не сказали обратное.
Например, утверждается, что записать символ в `std::cout` — атомарная операция и мы безопасно можем это сделать.

Но тут все еще плохо.

- src

    ```cpp
    #include <atomic>
    #include <iostream>
    #include <thread>
    
    #pragma GCC optimize ("O0")
    
    const int N = 50'000'000;
    const int M = 10'000;
    
    int main() {
        std::atomic<int> data = 0;
        auto worker = [&]() {
            for (int i = 0; i < N; i++) {
                data = data + 1;
            }
        };
        std::thread t1(worker);
        std::thread t2(worker);
        for (int i = 0; i < M; i++) {
            if (data % 2 == 0) {
                std::cout << "data is " << data << " (in progress)\n";
            }
        }
        t2.join();
        t1.join();
        std::cout << "data is " << data << "\n";
        return 0;
    }
    ```


Можно делать data++, но snapshot все еще нужен.

- src

    ```cpp
    #include <atomic>
    #include <iostream>
    #include <thread>
    
    #pragma GCC optimize ("O0")
    
    const int N = 50'000'000;
    const int M = 10'000;
    
    int main() {
        std::atomic<int> data = 0;
        auto worker = [&]() {
            for (int i = 0; i < N; i++) {
                data++;
            }
        };
        std::thread t1(worker);
        std::thread t2(worker);
        for (int i = 0; i < M; i++) {
            if (data % 2 == 0) {
                std::cout << "data is " << data << " (in progress)\n";
            }
        }
        t2.join();
        t1.join();
        std::cout << "data is " << data << "\n";
        return 0;
    }
    ```


Лучше не использовать atomic..

- src

    ```cpp
    #include <atomic>
    #include <iostream>
    #include <thread>
    
    struct list_node {
        std::atomic<int> value = 0;
        std::atomic<list_node*> next = nullptr;
    };
    
    void insert_after(list_node *n, int new_value) {
        list_node *n2 = new list_node;
        n2->value = new_value;
    
        #if 0
        n2->next = n->next;
        #else
        list_node *n_next = n->next;
        n2->next = n_next;
        #endif
    
        n->next = n2;
    
        // Further reading:
        // 1. Lock-free single linked list.
        // 2. CAS operations
        // 3. ABA problem
    }
    
    int main() {
        list_node head;
        const int N = 1'000'000;
        auto worker = [&]() {
            for (int i = 0; i < N; i++) {
                insert_after(&head, i);
            }
        };
        std::thread t1(worker);
        std::thread t2(worker);
        t1.join();
        t2.join();
        int len = 0;
        for (list_node *p = &head; p; p = p->next)
            len++;
        std::cout << len << "\n";
    }
    ```


Теряются элементы списка: нужно одновременно прочитать, записать, поменять с тремя атомиками одновременно ???

- src

    ```cpp
    
    ```


### Смешные UB

Программа зависает, data = 0 в конце. Оптимизатор считает, что оба while = бесконечный цикл (stop проверяется один раз, data не меняется вне потока).

- src

    ```cpp
    #include <cassert>
    #include <chrono>
    #include <iostream>
    #include <thread>
    
    #pragma GCC optimize("-O2")
    
    int main() {
        int data = 0;
        bool stop = false;
    
        std::thread t([&]() {
            while (!stop) {  // Hmm: stop is always the same => loop is either infinite or never starts.
                data++;
            }
        });
    
        while (data < 100) {}  // Hmm: data is always the same => loop is either infinite or never starts.
        assert(data >= 100);
        std::cout << "done " << data << "\n";
        stop = true;
    
        // Hmm: optimizations like this are important. The alternative is to always re-read from memory, memory is slow.
    
        t.join();
    }
    ```


Компилятор может поменять строки местами и data окажется непроинициазированной.

- src

    ```cpp
    #include <chrono>
    #include <iostream>
    #include <thread>
    
    #pragma GCC optimize("-O0")
    
    int main() {
        int data = 0;
        bool finished = false;
    
        std::thread t([&]() {
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
            // Hmm: we can run following two lines in any order, it does not matter.
            data = 123;
            finished = true;
        });
    
        while (!finished) {}
        // Hmm: we assume that `finished = true` is ran after `data = 123`.
        std::cout << data << "\n";
    
        t.join();
    }
    ```


### Volatile

`volatile` (ключевое слово языков C/C++, которое информирует компилятор о том, что значение переменной может меняться из вне и что компилятор не будет оптимизировать эту переменную.) — для многоточности вообще не помогает. На оптимизации процессора не влияет.

- src

    ```cpp
    #include <chrono>
    #include <iostream>
    #include <thread>
    
    #pragma GCC optimize("-O2")
    
    int main() {
        volatile int data = 0;
        volatile bool finished = false;
    
        std::thread t([&]() {
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
            // Hmm: compiler cannot reorder two writes below. But CPU still can!
            data = 123;
            finished = true;
        });
    
        while (!finished) {}
        // Hmm?
        std::cout << data << "\n";
    
        t.join();
    }
    ```


## Параллельные вычисления

### Базовое

```cpp
Появилось только в 1978 году, первые две страницы: https://lamport.azurewebsites.net/pubs/time-clocks.pdf

# Модель параллельных вычислений
Мотивация: есть оптимизации компилятора/процесора и кэши.
Хотим решить, по каким правилам играть.

Бывают даже слабые модели памяти, когда нельзя считать, что "потоки выполняются по очереди": https://habr.com/ru/company/JetBrains-education/blog/523298/

Решение: заводим между событиями отношение happens-before (произошло-до).
Это свойство конкретного выполнения программы(!), а не программы целиком.
Если между А и Б есть отношение, то всё ок, если нет — они произошли "одновременно" независимо от того, что показывают часы на стене (wall clock time).

Базово happens-before возникает:

1. Внутри потока: если `A; B`, то A happened-before B. Плюс ещё некоторые "sequenced-before": https://en.cppreference.com/w/cpp/language/eval_order
2. Создание потока.
3. Между потоками: "отпустили мьютекс M" happens-before "взяли мьютекс M".
4. У atomic'ов: "записали в A" не совсем happens-before "прочитали из A" (есть тонкости с "код должен зависеть от прочитанного значения", но этим почти никто не пользуется).

# Пример хорошего happens-before
```c++
// Thread 1
started = true;
m.lock();
finished = true;
m.unlock();

// Thread 2
m.lock();
//std::cout << started;
if (finished) {
    assert(started);    // Верно
}
m.unlock();
```

Здесь при любом выполнении между записью `finished = true` и чтением `finished`
всегда есть happens-before либо в одну сторону, либо в другую.

Аналогично с записью/чтением `started`.
И мы даже можем делать выводы про совместные значения `finished`/`started`.

# Пример отсутствия happens-before
```c++
// Thread 1
started = true;
m.lock(); data++; m.unlock();
finished1 = true;
finished2 = true;

// Thread 2
m.lock(); m.unlock();
if (finished2) { // UB
    assert(finished1);  // Непонятно, UB/reordering
    assert(data > 0);   // Непонятно, UB
    assert(started);    // Непонятно, UB
}
```

Если сначала выполнился thread 1, а потом thread 2,
то есть запись `started = true` happened-before чтения `started`, что хорошо.
Но нет никакого happens-before между записью `finished2 = true;` и чтением `finished2`.
То есть при чтении `finished2` будет UB и программа дальше UB.

Более того, ни в одном выполнении happens-before для `finished2` не получится,
то есть это UB вообще всегда.

# Мораль
* Не помогает даже `sleep_for`, он не создаёт никакого happens-before!
  Только маскирует проблемы.
* Мьютексы помогают хорошо: если у нас доступ к данным всегда защищён мьютексом, то в любом выполнении можем найти happens-before в одну из сторон.
* Можно делать хитрые мьютексы (`shared_mutex`): разрешать либо одного "писателя", либо несколько "читателей" одновременно.
  Как в Rust.
* Рекомендация: делайте поменьше общих ресурсов у потоков.
```

---

## 05/12/22

## Массивы в C

### Базовое про массивы

Есть массив `arr[N]` N - константа на этапе компиляции (кроме некоторых исключений).

```cpp
const int N = 5;

int main() {
// "C-arrays", "plain arrays"
[[maybe_unused]] int var1, arr[2 * N], var2;
// 10 uninitialized ints one after another.
// Automatic storage duration
// Size should be a compile-time expression, strictly greater than 0
```

`sizeof(arr) / sizeof(arr[0])` — размер массива.

```cpp
arr[2] = 123;
assert(std::size(arr) == 10);  // C++-style
assert(sizeof(arr) == 10 * sizeof(int));  // C style
assert(sizeof(arr) / sizeof(arr[0]) == 10);  // C style
```

Обращение к несуществующему элементу — UB.

```cpp
// std::cout << arr[10];  // UB
```

Нельзя копировать. Совсем.

```cpp
// No push_back/pop_back/insert/operator==/copying/...
// int arr2[2 * N] = arr;  // Just does not compile
// int arr3[2 * N];
// arr3 = arr;
```

Можно проходиться по элементам при помощи range-based-for.

```cpp
// range-based-for is ok, though
for (int &x : arr) {
    x = 10;
}
```

### **Variable Length Array (VLA)**

Можно создать массив неконстантной длины с помощью расширения GCC и Clang.

Отрубается функция `std::size` (sizeof работает).

```cpp
// GCC/Clang extension in C++ (available in C99): Variable Length Array (VLA).
// Not available in other compilers.
// https://gcc.gnu.org/onlinedocs/gcc/Variable-Length.html
int n;
std::cin >> n;
int vla[n];  // Allocated "on stack"
std::cout << sizeof(vla) / sizeof(vla[0]) << "\n";
// std::cout << std::size(vla) << "\n";  // Does not compile because there is no exact "type" for VLA to instantiate std::size with.
for (int i = 0; i < n; i++) {
    vla[i] = i * i;
}
```

### **Инициализация**

Изначально ничем не проинициализирован.

`a[10] = {}` — автоматически инициализируется 0.

```cpp
[[maybe_unused]] int a1[10];  // 10 default-initialized ints: uninitialized
[[maybe_unused]] std::string a2b[10];  // constructors are still called
[[maybe_unused]] int a2[10] = {};  // value-initialize all elements: 0.
[[maybe_unused]] int a3[10]{};  // same

[[maybe_unused]] int a4[10] = {1, 2, 3};  // value-initialize all elements
                                          // except first three (copy-initialized)
int a5[10]{1, 2, 3};  // same, copy-initialized

[[maybe_unused]] int a6[10] = { 0 };  // 0, 0, 0, 0, ...
[[maybe_unused]] int a7[10] = { 1 };  // 1, 0, 0, 0, ...
[[maybe_unused]] int a8[] = {1, 2, 3};  // size == 3
```

### **Арифметика указателей**

`int *dpr = data` — *array-to-pointer decay* — преобразование массива в адрес на свой первый элемент.

```cpp
int data[] = {1, 2, 3, 4, 5};
int *dptr = data;  // array-to-pointer decay, implicit
```

`*(dpr + 1)` — арифметика, указывает на data[1].

Оператор [] — `dpr[-1] == *(dpr - 1) == -1[dpr]`.

```cpp
&data[0], &data, dptr;  // All the same
*dptr, *(dptr + 2); 
*(dptr - 1), *(dptr + (-1)), dptr[-1], (-1)[dptr];  // 2
```

Можно воспринимать data как итератор — `sort(data, data + 5)`

```cpp
std::sort(std::begin(data), std::end(data));
std::sort(&data[0], &data[4] + 1);
std::sort(data, data + 5);
```

При этом выход зв пределы все еще будет  UB.

```cpp
// std::cout << *(dptr - 3) << "\n";  // UB, out of bounds
// std::cout << *(dptr + 3) << "\n";  // UB, out of bounds
```

### **Разыменование**

Нельзя разыменовать указатели в переменные.

```cpp
void print_all(int *beg, int *en) {
		// Just like iterators
		for (int *it = beg; it != en; it++) {
		    std::cout << *it << "\n";
		}
}

int data[]{40, 20, 10, 30};

int *one_past_end = data + 4;
print_all(data, one_past_end);
// int x = *one_past_end;  // dereference is UB.

int *ub1 = data - 1;
int *ub2 = data - 1 + 1;
int *ub3 = data + 5;
int *ub4 = data + 5 - 5;
```

### **Массивы в структурах**

Можно делать массивы полями в структурах. Копировать в структурах можно.

```cpp
const int MAX_N = 10;
struct Points {
    int n;
    int xs[MAX_N];  // Same as if 10 fields are defined.
    int ys[MAX_N];
};

Points a, b;

a.n = 1;
a.xs[0] = 10;
a.ys[0] = 20;

b = a;  // Can reassign structs with arrays inside.
```

### **Выделение массива на куче**

Если нужен массив большого размера — использовать оператор `new[]`  и `delete[]`.

```cpp
std::string *strs = new std::string[n];  // Array of 'n' std::string's,
// Heap-allocated, dynamic storage duration.
// Also possible: std::unique_ptr<std::string[]>, shared_ptr<std::string[]>
// Note [] so they call delete[] instead of delete.
// Better, although adds level of indirection: shared_ptr<vector<string>>

strs[0] = "hello world, this is the first string in the array";
strs[1] = std::string(1000, 'x');
for (int i = 0; i < n; i++) {
    std::cout << i << ": " << strs[i] << "\n";
}
delete[] strs;  // delete[], not just delete!

// Inferior to std::vector, please never use.
```

### **Получение С-массива из вектора**

```cpp
void print_all(int *beg, int *en) {
    // Just like iterators
    for (int *it = beg; it != en; it++) {
        std::cout << *it << "\n";
    }
}

std::vector<int> vec(10);
// Requires non-empty vec
print_all(&vec[0], &vec[0] + vec.size());
print_all(&*vec.begin(), &*vec.begin() + vec.size());
// Works even with empty vec
print_all(vec.data(), vec.data() + vec.size());

std::set<int> s{1, 2, 3};

std::set<int>::iterator it_1 = s.begin();
it_1++;  // ok

// UB:
print_all(&*s.begin(), &*s.begin() + 3);
int *s_it2 = &*s.begin();
s_it2++;  // UB
```

### **Передача массива в качестве аргумента**

`int *arr` — может быть указатель на int, а может быть и массивом. Решительно неясно.

Лучше, когда в функции передают указатель на начало и размер массива!

```cpp
// void foo(int arr[]) {  // Actually void foo(int *arr)
// void foo(int arr[5]) {  // Actually void foo(int *arr)
void foo(int arr[15]) {  // Actually void foo(int *arr)
// void foo(int *arr) {
    // Careful: sizeof pointer
    std::cout << "arr[0] = " << arr[0] << ", sizeof = " << sizeof(arr) << "\n";
}

void foo_good(int arr[], int arr_len) {
    std::cout << "arr[0] = " << arr[0] << ", len = " << arr_len << "\n";
    // foo_good(arr, sizeof arr / sizeof arr[0]);  // bad as well
    foo_good(arr, arr_len);  // ok
}
```

## Строки в C

### C-style string / null-terminated string / ASCIIZ string

Строка = массив символов `char str[]`, в котором последний символ имеет код 0 (`0` или `‘\0’`).

```cpp
// C-style string / строки в стиле си / сишные строки
// null-terminated string
// ASCIIZ string

/*const*/ char str[] = "hello";  // const is optional: we're copying "hello" into a local array with automatic storage duration.
// char str[] = {'h', 'e', 'l', 'l', 'o', 0};
```

`std::size(str)` и `sizeof(str) / str[0]` —> длина строки + 1.

```cpp
std::strlen(str);  // 5
std::size(str);  // 6
sizeof(str) / sizeof(str[0]);  // 6
```

`char *` в С++ трактуется именно как С-строка, а не массив char’ов —> могут быть проблемы, когда нужен именно массив.

Все еще есть преобразование из строки-массива в указатель.

`const char *` — указатель на начало строки, не строка.

```cpp
const char *str_ptr = str;  // array-to-pointer decay
// std::cout << std::size(str_ptr) << "\n";  // does not compile

std::cout << static_cast<const void*>(str_ptr) << "\n";
std::cout << str << "\n";  // decay

const char *str_ptr2 = &str[2];
std::cout << str_ptr2 << "\n";

const char strs[] = "hello\0world\0botva";
std::cout << std::size(strs) << "\n";  // 18
std::cout << strs << "\n";
std::cout << &strs[1] << "\n";
std::cout << &strs[6] << "\n";
std::cout << &strs[12] << "\n";

std::string cpp_str_1 = "hello\0world";  // initialization from C string
std::cout << cpp_str_1 << " " << cpp_str_1.size() << "\n";

std::string cpp_str_2 = "hello";
cpp_str_2 += '\0';
cpp_str_2 += "world";
std::cout << cpp_str_2 << " " << cpp_str_2.size() << "\n";
```

UB <— не найден 0 в конце.

```cpp
char a = 'x';
char *a_str_bad = &a;
char b[2] = "hi";

std::cout << a_str_bad << "\n";  // ub: out of bounds
std::cout << b << "\n";  // ub: out of bounds
```

### **Сравнение строк**

Можно сравнивать с помощь ‘<’, если указатели из одного массива. Если из разных, то это unspecified behaviour.

Если положить в конец 0, то неясно, какой это 0.

Правильное сравнение — через `for` или через `strcmp()` —> возвращает или 0, или число больше 0, или число меньше 0, не +-1.

```cpp
{
    const char *s = "hello";
    const char *a = s;  // hello
    const char *b = s + 1;  // ello
    std::cout << (a < b) << "\n";  // Not UB: 'a' and 'b' are in the same array.
}
{
    const char *a = "hello";
    const char *b = "ello";
    std::cout << (a < b) << "\n";  // unspecified behavior, may be inconsistent.
}
{
    const char *a = "hello";
    const char *b = "hello\0";
    std::cout << (a == b) << "\n";  // Unknown result, not UB.
}
{
    const char *a = "hello";
    const char *b = "hello\0";
    bool eq = true;
    for (int i = 0; a[i] || b[i]; i++)
        if (a[i] != b[i]) {
            eq = false;
            break;
        }
    std::cout << eq << "\n";
}
{
    const char *a = "hello";
    const char *b = "world";
    // not necessarily +-1. May be +12437.
    // mnemonics: sign(a - b)
    std::cout << std::strcmp(a, b) << "\n";
    std::cout << std::strcmp(a, "hello\0botva") << "\n";  // Always 0.
}
}
```

### **Длина строки**

За линию. Функция `strlen()` тоже за линию.

```cpp
int botva[10];

bool has_a(const char *str) {
//    for (std::size_t i = 0; i < std::strlen(str); i++) {
//    for (std::size_t i = 0, len = std::strlen(str); i < len; i++) {
    for (std::size_t i = 0; str[i]; i++) {
        if (str[i] == 'a') {
            return true;
        }
        botva[0] = 123;
    }
    return false;
}

int main() {
    {
        const char *a = "hello";
        int len = 0;
        while (a[len]) {
            len++;
        }
        std::cout << len << "\n";
        std::cout << std::strlen(a) << "\n";
    }
}
```

### Выделение памяти для строк

Можно писать функции, выделяющие память, которые в конце необходимо будет очистить память самостоятельно —> сложно отслеживать.

```cpp
const char *strcat_alloc(const char *a, const char *b) {  // It's not like std::strcat, the latter does not allocate memory at all.
    char *res = new char[std::strlen(a) + std::strlen(b) + 1];

    int res_len = 0;
    for (int i = 0; a[i]; i++)
        res[res_len++] = a[i];
    for (int i = 0; b[i]; i++)
        res[res_len++] = b[i];
    res[res_len] = '\0';
    return res;
}

int main() {
    const char *x = "xyz";
    const char *y = strcat_alloc("xy", "z") + 1;
    // ......
    delete[] (y - 1);
}
```

### **Считывание строк**

Функция `readWord`. Не проверяет, не вышли ли мы за границы массива. Небезопасно + нужно следить за размером буфера.

```cpp
#include "optimization.h"

int main() {
    const int MAX_LEN = 10;
    char buf[MAX_LEN + 1];
    bool is_password_correct = false;
    readWord(buf);  // Careful: do not overflow the buffer
                    // UB otherwise, see https://en.wikipedia.org/wiki/Buffer_overflow
                    // Solution: do not use 'readWord', use 'fgets' instead of 'gets'
                    // Also see https://ulearn.me/Course/Hackerdom
}
```

Функция gets. UB для длинных строк. Поэтому использовать не рекомендуется. Функции, не принимающие размер массива, — подозрительные…

```cpp
int main() {
    const int MAX_LEN = 10;
    char buf[MAX_LEN + 1];
    gets(buf);  // DANGEROUS
    std::cout << "buf=|" << buf << "|\n";
    // Top-1 vulnerability: https://ulearn.me/course/hackerdom/Perepolnenie_steka_3bda1c2c-c2a1-4fb0-9146-fccc47daf93b
}
```

Если более аккуратно, то `fgets()`.

```cpp

    printf("> ");

    char command[10];
    // Мы за безопасный gets!
    fgets(command, sizeof command, stdin);  // Always null-terminated string, truncated.

    // C++'s std::string never truncates, but may exhaust whole memory.

    printf("command is |%s|\n", command);
```

### **String dangling**

`c_str()` — указатель на внутренний буффер string. После удаления s, s_ptr указывает на очищенную память —> UB, которое стреляет не всегда (например, из-за small string optimization).

```cpp
const char *s_ptr;
{
    // Small string optimization
    // std::string s = "hello";
    std::string s = "hello 1234567890654345678986434567898643456789097654";

    s_ptr = s.c_str();
    // s_ptr = &s[0];
}
{
    char x[] = "botva1234565436543654654387658746";
    std::cout << x << "\n";
}
```

## Библиотеки в С

### Базовое

- about

    ```cpp
    C APIs may seem complicated. It is because:
    
    1. They use C and fully manual resource management.
       E.g. no methods grouped for an object, functions are intermingled in docs, lots of void*, lack of `const`.
       E.g. a buffer is user-allocated and is two arguments: the pointer and the length.
    2. They rarely interoperate with each other.
       E.g. there may be a "dynamic string/array" library and a separate
       "file reading"/"network sockets" library which only knows about C
       with fixed-size buffers.
    3. They are very low-level for the domain, although there may be "convenience wrappers".
       Lots of options, all cases of the domain are covered.
       Any interface may be wrong, convoluted or both.
       E.g.: do not use these functions: https://www.openssl.org/docs/man1.1.1/man3/SHA256.html
    4. There may be lots of "flags", bit-fiddling, data packing, little type checks.
       Not a beautiful `enum`, but just `int`/`long` (be careful) and a bunch of
       consts which you have to find in the code.
       E.g. `fopen("a.txt", "r")`, `printf("hello=%d", 123)`
       E.g. https://curl.se/libcurl/c/curl_easy_setopt.html
    5. Anything can err, should be handled.
       In C++ we just don't think about it as much => surprising exceptions or silent failures (like with `iostream`).
    
    How to use:
    
    0. Try find library in C++, otherwise:
    1. Try to find a "convenience wrapper" or "simple interface".
    2. Try to find a full official example: https://curl.se/libcurl/c/example.html
    3. DO NOT try to call random functions without reading docs/manpages.
    
    Convention:
    
    * (Almost) everything can be either zero-initialized with `memset` or have a dedicated initialization function.
    * (Almost) everything can be copied/moved bytewise with `memcpy`.
    * Memory and buffers are either:
      * Fully (de)allocated by the user (strings)
      * Fully (de)allocated by the library (see next example).
    ```


### **Void ***

Передача произвольных данных функций (например, потому что нет перегрузок,  а данные могут быть различны).

`void *` — базовый указатель на все на свете, указатель куда-то в память.

Арифметика по стандарту не компилируется, но gcc и clang сделали расширение — на 1 байт.

```cpp
#include <cassert>

// "Library"
void *user_data;

//todo: like setopt
void set_data(void *data) {
    user_data = data;
}

void *get_data() {
    return user_data;
}

// Somewhat like std::any

// Application
int main() {
    int data[]{1, 2, 3, 4};

    set_data(data);  // implicit: int[4] --> int* --> void*
    int *data_ptr = static_cast<int*>(get_data());
    assert(data_ptr[2] == 3);
}
```

### Const cast

Какая-то древняя ничего не меняющая функция принимает char *, а нужно передать const char *, что делать? —> `const_char<char *>(s)`. Если попытать после этого поменять s, то будет UB.

```cpp
int len(char *s) {
    int l = 0;
    while (s[l] != '\0') {
        l++;
    }
    return l;
}

void update(char *s) {
    s[0] = 'h';
}

int main() {
    const char *s = "hello";
    // assert(len(s) == 5);  // compilation error
    assert(len(const_cast<char*>(s)) == 5);  // ok

    update(const_cast<char*>(s));  // ub
}
```

### **Непрозрачные указатели (opaque pointer)**

`FILE *` — не лезь, убьет..

```cpp
#include <stdlib.h>
#include <stdio.h>

int main() {
    // "opaque" pointer
    FILE *f = fopen("02-opaque-pointers.c", "r");  // "constructor", resources are allocated by the library.

    // Never try to access `FILE`'s fields directly.
    char buf[20];
    fscanf(f, "%20s", buf);  // "method"
    printf("buf=|%s|\n", buf);

    fclose(f);  // "destructor"
}
```

---

## 12/12/22

## Функции

### Перегрузки функций

Функции в одном `namespace` можно перегружать, одно имя — разные параметры.

Компилятор начинает выбор перегрузки, разрешение перегрузки.

Иногда непонятно, какую функцию выбрать — не компилируется.

```cpp
void println(int x) {
    std::cout << x << "\n";
}

void println(double x) {
    std::cout << x << "\n";
}

void println(const std::vector<int> &vec) {
    std::cout << "{";
    bool first = true;
    for (const auto &item : vec) {
        if (!first) {
            std::cout << ", ";
        }
        first = false;
        std::cout << item;
    }
    std::cout << "}\n";
}

println(10);
println(1.2);
println(std::vector{1, 2, 3});
//    println(123456789123456);
println(1LL);
```

### Неоднозначности перегрузок

**Проблема с NULL**

`NULL` (канонично — указатель в никуда) — проблема при перегрузке функции, некоторые компиляторы считают `int`’ом.

```cpp
struct node {};

void foo(int) {
    std::cout << "foo(int)\n";
}

void foo(node*) {
    std::cout << "foo(node*)\n";
}

int main() {
    foo(0);
    foo(NULL);
    foo(nullptr);  // C++11
}
```

**Проблема с функциями из библиотек**

Компилятор выбрал стандартную функцию, которая вызывается от `int`’ов.

```cpp
using namespace std;

int gcd(int a, int b) {
    // Ternary operator.
    // Bad stuff happens when two branches are of different types.
    // (some complicate type deduction)
    return b ? gcd(b, a % b) : a;

    /* if (b != 0) {
        return gcd(....);
    } else { 
        return a;
    } */
}

// Since C++17: int std::lcm(int a, int b)
// https://codeforces.com/blog/entry/94896
long long lcm(long long a, long long b) {
    return a / gcd(a, b) * b;
}

int main() {
    std::cout << lcm(1'000'000'000, 1'000'000'001) << "\n";
}
```

### Неименованный аргумент

1. использовать [[maybe_unused]] — `f([[maybe_unused]] int x)`
2. не писать имя аргумента — `f(int)`

```cpp
// [[]] is an 'attribute', since C++17
void foo([[maybe_unused]] int x) {
    std::cout << "foo(int)\n";
}

void foo(int x, int) {
    std::cout << "foo(int x, int) x=" << x << "\n";
}

void foo(int x, int, int z) {
    std::cout << "foo(int x, int, int z) x=" << x << ", z=" << z << "\n";
}
```

### Тэги (type tags)

Костыль. Структура-тэг. Для пометок перегрузок значениями нужных типов. Обычно заканчиваются на `*_tag`.

```cpp
struct comma_separated_tag {};  // Not _t, it's reserved.
struct element_per_line_tag {};  // Not _t, it's reserved.

void println(const std::vector<int> &vec, comma_separated_tag) {
    bool first = true;
    for (const auto &item : vec) {
        if (!first) {
            std::cout << ", ";
        }
        first = false;
        std::cout << item;
    }
    std::cout << "\n";
}

void println(const std::vector<int> &vec, element_per_line_tag) {
    for (const auto &item : vec) {
        std::cout << item << "\n";
    }
}

int main() {
    std::vector<int> v{1, 2, 3};
    // type tags
    println(v, comma_separated_tag{});
    println(v, element_per_line_tag{});
}
```

### Параметры по умолчанию

Можно у функций можно указать у каких-то *последних* аргументов дефолтные значения.

При вызове просто не пишутся аргменты.

```cpp
// K _last_ parameters can be defaulted.
void foo(int x, int y = 10, int z = 20) {
    std::cout << "x = " << x << "\n";
    std::cout << "y = " << y << "\n";
    std::cout << "z = " << z << "\n";
}

void bar(std::vector<int> vec = {1, 2, 3}) {
    vec.push_back(100);
    std::cout << "vec.size() = " << vec.size() << "\n";
}

foo(1);
foo(2, 3);
// foo(2, z=3);  // Python-only, not C++
foo(4, 5, 6);
bar();
```

**Пример перегрузки дефолтных значений**

```cpp
#include <iostream>

void foo(int, int = 10) {
    std::cout << "1\n";
}

void foo(int) {
    std::cout << "2\n";
}

int main() {
    foo(10, 20);
    foo(30);
}
```

## Функтор/функциональный объект

### Базовое

Структура, у которой перегружен `operator()`.

Аккуратно с вызовом и конструктора, и оператора()!!!

```cpp
struct Greater {  // Functor (jargon)/special case of "functional object"
    bool operator()(int a, int b) const {
        return a > b;
    }

    int operator()(int a, int b, int c) const {  // overloads are possible
        return a + b + c;
    }
};

int main() {
    std::vector<int> v{1, 2, 3, 4, 5, 6, 7, 8};
    const Greater g;  // 'const' is optional, just to demonstrate const-qualifier
    assert(g.operator()(5, 4));
    assert(!g.operator()(4, 5));
    assert(g(5, 4));
    assert(!g(4, 5));
    assert(Greater().operator()(4, 5));
    assert(Greater()(4, 5));
}
```

Некоторые функции принимают в качестве аргументов функторы. Например, для `sort` оператор должен быть `const`.

```cpp
sort(v.begin(), v.end(), g /* Greater() */);
```

### Использование функторов

**Пример компаратора**

```cpp
struct CloserTo {
    int center;

    bool operator()(int a, int b) const {
        return std::abs(a - center) < std::abs(b - center);
    }
};

sort(v.begin(), v.end(), CloserTo{3});
```

**Пример с set**

Можно передавать компаратор в `set`, которому он нужен, чтобы уметь сравнивать элементы. Лямбды здесь не работают. `set`'у нужен именно тип.

```cpp
struct Greater {
    bool operator()(int a, int b) const {
        return a > b;
    }
};

int main() {
    std::set<int, Greater> v{1, 2, 3, 4, 5, 6, 7, 8};  // Creates Greater() with default ctor.
}
```

Поменять нельзя, только создать новый с новым компаратором и переприсвоить.

```cpp
struct CloserTo {
    int center;

    bool operator()(int a, int b) const {  // 'const' is important
        return std::abs(a - center) < std::abs(b - center);
    }
};

int main() {
    std::set<int, CloserTo> v1({1, 2, 3, 4, 5, 6, 7, 8}, CloserTo{3});  // Copies CloserTo() inside
    std::set<int, CloserTo> v2({1, 2, 3, 4, 5, 6, 7, 8}, CloserTo{7});
    // v1 = v2;
    // v1 = std::set(v1.begin(), v1.end(), CloserTo{5});
}
```

### Указатель на функцию

`(*)` — указатель на функцию. Так можно передавать любые функции, но не лямбды.

```cpp
bool compare_gt(int a, int b) {
    return a > b;
}

int main() {
    std::set<int, bool(*)(int, int)> v({1, 2, 3, 4, 5, 6, 7, 8}, compare_gt);
}
```

`std::function` — не совсем указатель на функцию, можно считать базовым классом для всех существующих функциональных обхектов. Можно класть что угодно, даже лямбды. Возвращаемый тип не пишется, работает как `auto`.

Выделяет на куче место для функтора, внутри себя хранит указатель + какие-то виртуальные функции. Похож на `unique_ptr`.

*Type Erasure* — стирание типов.

```cpp
struct CloserTo {
    int center;

    bool operator()(int a, int b) const {
        return std::abs(a - center) < std::abs(b - center);
    }
};

struct BigFunctor {
    long long a, b, c, d, e, f, g, h, i;

    bool operator()(int lhs, int rhs) const {
        return lhs < rhs;
    }
};

int main() {
    [[maybe_unused]] std::function<bool(int, int)> cmp1 = CloserTo();
    std::cout << sizeof(CloserTo) << " " << sizeof(cmp1) << "\n";

    [[maybe_unused]] std::function<bool(int, int)> cmp2 = BigFunctor();
    std::cout << sizeof(BigFunctor) << " " << sizeof(cmp1) << "\n";

    int x = 10;
    [[maybe_unused]] std::function<bool(int, int)> cmp3 = [&](int a, int b) {
        return a + x < b + x;
    };
    std::cout << sizeof(cmp3) << "\n";
    
    // Type Erasure (need basic templates): https://www.youtube.com/watch?v=tbUCHifyT24
}
```

### Передача функторов в качестве аргументов

STL предполагает, что копировать можно как угодно.

Функтор `c` скопировалась внутрь `for_each`, сам `c.index` не поменялся. Принимает по значению.

`for_each` возвращает функтор — так можно узнать, какое состояние у функтора послу выполнение функции. Враппер внутри себя реализует `operator()`.

```cpp
struct Counter {
    int index = 10;
    void operator()(int value) {
        std::cout << index << " " << value << "\n";
        index++;
    }
};

int main() {
    {
        std::vector<int> v{1, 2, 3, 4};
        Counter c;
        std::for_each(v.begin(), v.end(), c);
        std::cout << "c.index = " << c.index << "\n";  // May be 10
    }
    {
        std::vector<int> v{1, 2, 3, 4};
        Counter c;
        c = std::for_each(v.begin(), v.end(), c);
        std::cout << "c.index = " << c.index << "\n";  // Should be 14
    }
}
```

### Ref-wrapper

`std::reference_wrapper<…>` — ссылка на объект, но ее можно скопировать.

`std::ref(..)` — другой способ копирования.

Можно как угодно вызвать ссылку или объект — вызов враппера равносилен вызову функтора с круглыми скобками.

—> можно передать в функция reference wrapper, а не функтор, тогда функтор поменяется, копироваться не будет.

В отличии от обычной ссылки, враппер можно копировать и менять.

```cpp
struct Counter {
    int index = 10;
    void operator()(int value) {
        std::cout << index << " " << value << "\n";
        index++;
    }
};

int main() {
    {
        Counter c;
        Counter c2 = c;  // copy
        std::reference_wrapper<Counter> cwrap = c;  // copyable reference to c OR raw pointers with operator()
        assert(&cwrap.get() == &c);
        auto cwrap2 = cwrap;  // reference to c
        auto cwrap3 = std::ref(c);  // std::reference_wrapper<Counter>
        auto cwrap4 = cwrap3;

        c(1);  // 10 1
        c(2);  // 11 2
        c2(3);  // 10 3
        c2(4);  // 11 4
        c2(5);  // 12 5
        // std::reference_wrapper implements operator() only in addition to get()
        cwrap(6);  // 12 6
        cwrap(7);  // 13 7
        cwrap2(8);  // 14 8
        cwrap2(9);  // 15 9
        cwrap3(10);  // 16 10
        cwrap3(11);  // 17 11
        cwrap4(12);  // 18 12
        cwrap4(13);  // 19 13

        // Also: std::reference_wrapper<const Counter> and std::cref
    }
    std::cout << "=====\n";
    {
        std::vector<int> v{1, 2, 3, 4};
        Counter c;
        std::for_each(v.begin(), v.end(), std::ref(c));
        std::cout << "c.index = " << c.index << "\n";  // Should be 14
    }
}
```

## Лямбды

### Базовое

Лямбда функция — это синтаксический сахар поверх `struct CloserTo`.

```cpp
sort(v.begin(), v.end(), [=](int a, int b) {  // Can also be: [&], [city_position]
    return std::abs(a - city_position) < std::abs(b - city_position);
});
```

### **Виды захватов**

Каждая лямбда создает свой тип, со своими полями, которые она захватывает (локальные переменные, которые она захватывает), и operator().

В квадратных скобках пишется

`[ ]` — *ничего* не захватывает.

Имеет доступ ко всем своим полям и к глобальным переменным. Размер маленький, нет внутренних полей.

`[=]` (`[this, local_var]` — явное перечисление) — захват всех используемых переменных *по значению*. Они все копируются внутрь лямбды, что может быть долго. Поля лямбды не умеют захватывать поля, только `this` целиком.

Лямбда запомнит значение в момент создания, после изменения переменных значение внутри лямбды не изменится. Но при принимании по значению нельзя менять локальные перменные, по умолчанию `operator()` помечен  `const`.

Указатель `this` поменять нельзя, а `*this` можно поменять.

`mutable` —  по умолчанию `operator()` у лямбды помечен `const`: она не может менять свои поля (то есть захваченные значения). Можно добавить слово `mutable`, тогда этот `const` уберётся. Так делают довольно редко, обычно лямбды делают без состояния.****

`[&]` — захват всего по ссылке (кроме `this`).  `operator()` все еще константная. Теперь переменные меняются вне лямбд. Если лямбду вызвать после того, как умерли ее захваты, получится висячая ссылка.

`[a = b], [a = a]`  (С++14) — создание новой переменной `a`внутри лямбды, инициализация выражением `b`. Имя `a` может совпадат с именем локальной переменной.

`[&a = b]` (С++14) — создание новой ссылки `a`внутри лямбды, привязанной к выражению `b`. Имя `a` может совпадат с именем локальной переменной.

Поля захватываются только вместе с `this`/`*this` (если только не написать через `x = x`). Из `[=]` и `[&]`следует `[this]` (если он используется).****

`[&local_var - std::as_const(local_var)]` — захват по константной ссылке.

Также можно комбинировать несколько захватов, для каждой используемой переменной выберется наиболее специфичный: `[&, a, &b = c, *this]`

```cpp
struct Foo {
    int member_var = 10;
    static inline int static_member_var = 20;
    int arr[10'000];

    void method() {
        int local_var = 30;
        static int static_local_var = 40;
        [[maybe_unused]] long long local_big[100]{};

        // 1
        auto lambda1 = []() {
            // Not captured.
            // [[maybe_unused]] int x = member_var;
            // [[maybe_unused]] int y = local_var;
            // Never captured, always available.
            static_member_var++;
            static_local_var++;
        };
        lambda1();
        std::cout << "sizeof(lambda1) == " << sizeof(lambda1) << "\n";
        assert(static_member_var == 21);
        assert(static_local_var == 41);

        // 2
        std::cout << "before lambda 2: " << &member_var << " " << &local_var << "\n";
        auto lambda2 = [=]() {  // [this, local_var]
            std::cout << "in lambda 2    : " << &member_var << " " << &local_var << "\n";
            assert(member_var == 10); // this->member_var: reference!
            assert(local_var == 30);
            // assert(local_big[0] == 0);  // Not captured, as unused.
            // operator() is const by default.
            member_var++;  // this is const, but *this is not.
            // local_var++;
        };
        local_var++;  // Do not affect lambda2().
        lambda2();
        local_var--;
        std::cout << "sizeof(lambda2) == " << sizeof(lambda2) << "\n";

        // 3
        auto lambda3 = [=]() mutable {  // [this, local_var]
            // operator() is non-const now.
            member_var++;
            local_var++;
            std::cout << "in lambda 3: local_var new = " << local_var << "\n";
        };
        lambda3();
        std::cout << "sizeof(lamda3) == " << sizeof(lambda3) << "\n";
        assert(member_var == 12);  // Changed.
        assert(local_var == 30);  // Unchanged.

        // 4
        [&]() {  // [this, &local_var]
            // operator() is still const, but these two work.
            member_var++;  // this->member_var, this is const.
            local_var++;  // local_var is a reference, the reference itself is always const.
        }();
        assert(member_var == 13);
        assert(local_var == 31);

        // 5
        std::cout << "before lambda 5: " << &member_var << " " << &local_var << "\n";
        auto lambda5 = [=, *this]() mutable {  // C++17: [*this, local_var]: full copy of '*this', including unused 'arr'.
            std::cout << "in lambda 5    : " << &member_var << " " << &local_var << "\n";
            assert(member_var == 13); // 'this' is fully copied.
            assert(local_var == 31);
            member_var++;
            local_var++;
        };
        lambda5();
        std::cout << "sizeof(lambda5) == " << sizeof(lambda5) << "\n";
        assert(member_var == 13);  // Unchanged.
        assert(local_var == 31);  // Unchanged.

        // 6
        [wtf = local_var + member_var]() {  // C++14
            assert(wtf == 44);
        }();
        [local_var = local_var * 2]() {
            assert(local_var == 62);
        }();
        assert(local_var == 31);

        // 7
        [&local_var = local_var]() {  // Shortcut: [&local_var] (even in C++11)
            assert(local_var == 31);
            local_var++;
        }();
        assert(local_var == 32);
        [&local_var = std::as_const(local_var)]() {
            assert(local_var == 32);
            // local_var++;  // const int &local_var;
        }();
        assert(local_var == 32);
    }
};
```

### Тип лямбд

У всех лямб свой тип. Даже если они одинаковы, типы будут разными. Если нужны лямбды содного типа, то придется писать свою структуру с `operator()`.

```cpp
auto l1 = [](int a, int b) { return a > b; };
auto l2 = [](int a, int b) { return a > b; };
std::set<int, decltype(l1)> s1(l1);
// std::set<int, decltype(l1)> s2(l2);
```

### Рекурсия лямбд

Рекурсии нет. Внутри лямбды нельзя назвать лямбду.

```cpp
auto fib = [&](int n) {
    if (n <= 1) return n;
    // Compilation error: type of lambda depends on its captures => cannot capture fib.
    return fib(n - 1) + fib(n - 2);
};
```

Можно написать `std::function<int(int)>`. Но это вызов виртуальной функции (очень дорого).

```cpp
std::function<int(int)> fib = [&](int n) {
    if (n <= 1) return n;
    // Careful: calls to virtual functions!
    return fib(n - 1) + fib(n - 2);
};
```

Дешевле — писать `operator()`.

```cpp
int operator()(int n) {
    if (n <= 1) return n;
    return operator()(n - 1) + operator()(n - 2);
    // return (*this)(n - 1) + (*this)(n - 2);
}
```

## Extra

### Псевдонимы типов

Два способа переопределить имя типа — `typedef` и `using`. Разница только на этапе компиляции. Еще можно `#define` (категорически против, плевать на namespace, зависимости и прочее).

```cpp
typedef std::vector<int> vi;
using pii = std::pair<int, int>;
// Please do not #define: it does not respect namespaces/private/public
// https://stackoverflow.com/a/1666375/767632
```

### Наследование с виртуальными функциями, подхвохи

Вызывать виртуальные функции в конструкторах — плохо. Вызывается сначала функция из родителя, потом из наследника, то есть вызывается не только самая вложенная функция. У базового класса вызовется только его функция. Аналогично для деструктора.

```cpp
#include <iostream>

struct Base {
    int value = 123;

    virtual void foo() {
        std::cout << "foo(" << value << ")\n";
    };

    Base() {
        foo();
    }

    ~Base() {
        foo();
    }
};

struct Derived : Base {
    int value2 = 456;

    void foo() override {
        std::cout << "foo(" << value << ", " << value2 << ")\n";
    }

    Derived() : Base(), value2(100) {
        foo();
    }

    ~Derived() {
        foo();
    }
};

int main() {
    Derived d;
    std::cout << "== constructed ==\n";
    d.foo();
    static_cast<Base &>(d).foo();
    std::cout << "== foo called ==\n";
}
```

Не использовать в конструкторах…

```cpp
#include <iostream>

struct Base {
    int value = 123;

    virtual void printTo() = 0;

    Base() {
        printTo();
    }
};

/*void Base::printTo() {  // BUT WHY
    std::cout << "Not so pure, eh?\n";
}*/

struct Derived : Base {
    void printTo() override {
    }
};

int main() {
    // Base b;  // class is still abstract.
    Derived d;
}
```

### Парадокс круга-эллипса.

Что от кого наследовать? Одно является частным случаем другого:

1. **Эллипс —> круг**

Если у элипса есть ширина-высота, то понятно, как считать тоже самое у круга (+ добавиться еще и радиус).

```cpp
// https://isocpp.org/wiki/faq/proper-inheritance

// OK
struct Ellipse {
    virtual int width();
    virtual int height();
};

struct Circle : Ellipse {  // 'is-a' relationship
    int width() override {
        return 2 * radius();
    }
    int height() override {
        return 2 * radius();
    }
    virtual int radius();
};
```

В обратную сторону — неясно.

```cpp
// WTF
struct Circle {
    virtual int radius();
};

struct Ellipse : Circle {
    int radius() override {
        ? ? ? ? ? ? ? ? ? ? ? ? ?
    }
    virtual int width();
    virtual int height();
};
```

Попробуем так отнаследовать. Добавим эллипсу методы для установления высоты и установления ширины. Но тогда `foo()` для круга работать не будет (меняется ширина и высота)!

```cpp
// WTF
struct Ellipse {
    virtual int width();
    virtual int height();
    virtual void setWidth(int w);
    virtual void setHeight(int h);
};

void foo(Ellipse &e) {
    // LSP violation:
    // https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%B8%D0%BD%D1%86%D0%B8%D0%BF_%D0%BF%D0%BE%D0%B4%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B8_%D0%91%D0%B0%D1%80%D0%B1%D0%B0%D1%80%D1%8B_%D0%9B%D0%B8%D1%81%D0%BA%D0%BE%D0%B2
    e.setWidth(10);
    e.setHeight(20);
    assert(e.width() == 10);
    assert(e.height() == 20);
}

struct Circle : Ellipse {
    void setWidth(int w) override {
        setRadius(w / 2);  // ???
    }
    virtual int radius();
    virtual void setRadius(int r);
};
```

1. **Круг —> эллипс**

…тогда удобно сделать так:

```cpp
// OK
struct Circle {
    virtual void setRadius(int r);
};

struct Ellipse : Circle {
    void setRadius(int r) override {
        setWidth(2 * r);
        setHeight(2 * r);
    }
    virtual void setWidth(int w);
    virtual void setHeight(int h);
};
```

Получилось нарушение *принципа подстановки Барбары Лисков*: если что-то работает для эллипса, то должно работать и для круга-наследника.

Вывод: если мы только читаем, то выгодно одним способом. Если пишем — другим способом.

> *Поэтому неважно, что происходит в жизни, важно, что происходит в коде.*
>

---